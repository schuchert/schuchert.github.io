---
layout: post
draft: false
title: "Opportunistic Optimization: Shaving Five Minutes Off the Loop While Waiting on CI"
description: "How sneaking in side-branch experiments with an LLM during pipeline queues dropped test execution from six minutes to 58 seconds—and turned an urgent QAT defect into a casual afternoon non-event."
category: engineering
tags: [ systems thinking, developer experience, continuous integration, llm, testing, architecture, feedback loops ]
---

> *Note: This article is a co-creation developed in collaboration with Gemini. Interacting with the model
served both as the optimization pairing partner during the session described below and as a co-author in reflecting on
the systems dynamics.*
>
> *This post comes in two flavors. I sketched an outline from memory, then asked Gemini to write it twice, changing the
> constraints between attempts. This is the story version; the report version is [Shrinking the Feedback
> Loop](https://schuchert.github.io/engineering/2026/08/26/Shrinking-the-FeedbackLoop-From6Minutesto58SecondsWithAnLLM.html).
> Neither one is the authoritative account, which is most of the point. Afterwards I had Claude go through both against
> the repository, because I wanted the feel to vary and the facts not to.*

## The Inevitable Queue

Every software engineer knows the dead space between `git push` and the green checkmark.

You finish a slice of work. The code is clean, local checks pass, and you push the branch to Azure DevOps (ADO) to
create a Pull Request. Then comes the wait. The remote pipeline spins up agents, pulls dependencies, runs static
analysis, compiles, and executes the suite.

On our team, that remote PR build was reliably taking around ten minutes.

Ten minutes is an awkward pocket of time. It is too short to load the mental context for a new user story, but far too
long to sit idly watching animated console spinners. Left unmanaged, it tempts you into low-value multitasking—checking
chat channels, opening news tabs, or getting pulled into side conversations that derail your flow.

We had already been discussing our local unit and integration test suite times. On a standard developer workstation,
running the full local build took roughly **six minutes**.

Six minutes meant running the full suite locally felt like a major commitment. You only ran it when you were fairly sure
you were done, leaning on the CI pipeline as an asynchronous crutch to catch anything you missed.

Knowing I had a ten-minute pipeline wait ahead of me, I decided not to switch tasks. Instead, I decided to sneak in a
side branch and point an LLM directly at our test harness to see how much latency we could shave off before the remote
build even finished.

---

## Act I: The Ten-Minute Window (6:00 → 3:44)

The initial premise was simple: treat the build harness itself as code under active refactoring.

I cut a temporary branch off `main`, captured our build configurations, test setup runners, and base integration test
classes, and handed them to Claude (running Opus-5).

The prompt was direct: *“Our local test suite takes six minutes. Review these configurations, identify redundant setup
or teardown lifecycle overhead, check our concurrency settings, and suggest concrete changes to cut execution time
without compromising isolation.”*

Within seconds, the LLM had found a sledgehammer where a scalpel would do.

Fifteen test classes carried `@DirtiesContext` — mostly `AFTER_CLASS`, one `AFTER_EACH_TEST_METHOD`. That annotation
tells Spring to throw away the cached application context and build a fresh one, which is the most expensive isolation
mechanism the framework offers. In these classes it was mostly there to undo a couple of inserted rows.

The interesting part is that deleting it was *not* the whole change. Three of those classes did need cleanup between
tests — so they got the cheap version:

```java
@AfterEach
void cleanup() {
    repository.deleteAll();
}
```

Same isolation, a tiny fraction of the cost. Rebuilding a Spring context to undo two `INSERT`s is not rigor, it is a
category error about which mechanism you reach for.

Fifteen classes, one substitution, and a run:

```
BUILD SUCCESSFUL in 3m 44s
```

From **6:00 down to 3:44** on the first pass.

One thing I deliberately left alone: the test runner was pinned to a single fork *on purpose*, because parallel forks had
previously caused embedded-Kafka shutdown races. There was obviously idle CPU on the table, but forking was the one
change that could reintroduce flake — so it waited for Act III, and for a flag.

By the time the remote ADO PR pipeline finished its ten-minute run, I had already shaved over two minutes off the local
loop on a separate branch.

---

## Act II: The Security Intermission (3:44 → ~2:00)

We merged the feature branch into `main`. Almost immediately, the automated pipeline hit a tripwire: a Veracode security
scan flagged a vulnerability that reached us through a shared internal library. Part of it we could fix in our own
dependency catalog with a Bouncy Castle bump; the rest needed a new build of the upstream library.

This wasn’t something I could solve inside our repository directly. I pinged the developer maintaining the upstream
dependency. He hopped on it, bumped the vulnerable transitive dependency, patched the code, and kicked off an upstream
build to publish a new version.

That handoff meant another ten-minute operational pause.

Rather than sitting on my hands waiting for the new package artifact to drop, I pulled the latest `main` into my
optimization branch and kept pairing with Claude.

This time I went looking somewhere the test report never points: **after the last test passes**.

Watch a run closely and you notice the suite goes green, and then the process just sits there. Forty-one seconds of
sitting there. Roughly twenty-seven of them turned out to belong to a politeness ritual: Reactor Netty will not drop its
event loops without first waiting out a two-second "quiet period" so in-flight requests can finish, and
`ReactorResourceFactory` performs that courtesy once per cached application context, one after another, from the
shutdown hook.

No test has an in-flight request. We had been waiting, over and over, for nobody.

Two lines:

```kotlin
systemProperty("spring.reactor.netty.shutdown-quiet-period", "0s")
systemProperty("server.shutdown", "immediate")
```

That is my favorite kind of waste: invisible in every test report, costing nothing to remove, and risk-free for a reason
you can state in one sentence.

The same instinct nearly cost us. A global embedded-Kafka property looked exactly like the residue that collects in
build files: it names no topics, and every Kafka test class declares its own anyway. I was ready to delete it.

We timed it first — **139.8s with it, 147.4s without** — and found it was quietly holding one broker open for six test
classes that would otherwise have started six of their own, at about 1.9 seconds each.

It stayed. So did both numbers, as a comment, so the next person who thinks it looks useless can be done thinking in
ten seconds.

Just as the notification popped up that the upstream library had published its fix, we updated our internal version
pointer and triggered another local run.

Execution time dropped to **just over 2 minutes**.

In two remote waiting windows, we had carved four minutes out of every subsequent test cycle.

---

## Act III: Containers, Ghost Failures, and `DANGEROUS_FAST_BUILD` (2:00 → 58s)

With the baseline down to two minutes, the remaining bottleneck was obvious: integration container lifecycles.

The full-text search tests need a real SQL Server, which we get from a container. Under emulation it costs roughly 25
seconds to start and seed, and we were paying that on every run.

There is a flag for this. Tell Testcontainers a container is reusable and it simply declines to tear the thing down, so
the next run walks up to a database that is already warm, already seeded, already listening. You pay the **77 to 97
seconds** once, when the container has to be built. After that:

```
BUILD SUCCESSFUL in 58s
```

Reuse hands you a bill, though. Ryuk — the reaper that normally cleans up after Testcontainers — is built to leave
reusable containers alone, because that is the only way they survive the JVM. Which makes the container my problem now.
Killing it when the suite ends would hand back exactly what I just bought, so the build starts an idle timer after every
`test` run instead. Test again and the timer resets. Walk away long enough and the container lets itself out.

### The Ghost in the Concurrency

Then, mid-pass, a failure went by in my terminal. `TrainingFilterIntegrationTest`, missing data. I re-ran it
immediately. Green.

This is the point where the story wants to become tidy, and I have all the ingredients for a satisfying one: four JVMs
now instead of one, a shared container, a suite that had been single-forked for a reason. Race condition. Chapter
closed.

I tried fifteen more times, some with the CPU pinned. Green, every time. I never found it, and at this point I do not
expect to.

Here is the part I do know, because I was there for it: two suites were hitting that host at once. I was running tests
from the command line while the model ran its own. I am certain that was happening — I was the one doing it. My strong
suspicion is that this is the entire story, and that the failure belonged to that afternoon rather than to the code. Two
suites, one container, one set of ports.

Strong suspicion is not knowing, and I have made peace with never knowing. What I can do is make the question stop
mattering. That is the move when a failure will not reproduce: **decouple the mechanism from the default path.** I did
not need the diagnosis to keep the speedup. I needed the speedup to be something a developer chooses, and a build agent
never does.

Which is also why the caution that follows is real and not ceremonial — a week of watching before either switch becomes
anyone's default. Not because I think it is broken. Because "I strongly suspect" is the honest confidence level, and a
week of waiting is cheap at that price.

Two independent switches, deliberately not one:

```bash
# In your shell — four test forks for the module that benefits
export DANGEROUS_FAST_BUILD=true

# In ~/.testcontainers.properties — keep the SQL Server container warm between runs
testcontainers.reuse.enable=true
```

They are separate on purpose, and they are separate in different ways. Forking is per-shell, so unset — everywhere except
a developer who exports it — reproduces the previous single-fork behavior *exactly*. Reuse is per-machine, and CI now
pins it off explicitly rather than staying safe by the accident of no agent happening to have that file.

With neither switch set, the structural cleanups from Acts I and II still put the build at **104 seconds**, down from
360. That is the number that matters for everyone who never reads this post.

We committed the changes, opened a PR for the build harness improvements, and established a one-week observation window
before considering either one a default.

## The Crucible: A 30-Minute QAT Turnaround

The real test of an optimization is not the benchmark you screenshot; it is how the system behaves under pressure.

Mid-afternoon, shortly after the build work landed, a query against the Quality Assurance Testing (QAT) environment
started answering with a 500.

Two rows. Sixty course offerings came back, and two of them had no title. The schema says `title: String!`, and GraphQL
does not shrug at a broken promise — it pushes the null up the tree until it finds something allowed to hold one, which
here was the top of the response. The client received `data: null`.

Two rows took out sixty.

In the old reality (6-minute local runs, slow PR pipelines, 10-minute CI verification), a bug discovered mid-afternoon
in QAT was almost guaranteed to become tomorrow's problem:

1. Formulate a hypothesis.
1. Run local tests (wait 6 minutes).
1. Adjust code, re-run tests (wait 6 minutes).
1. Push branch, await PR build and scans (wait 15–20 minutes).
1. Merge, deploy, verify.

By the time you complete two local verification cycles and one remote pipeline, the workday is over.

And the honest answer to "what is the fix" was: ask the people who own the data. Scrub it, constrain the view, or drop
those rows in the query — three real options, each carrying somebody else's consequences, none of them mine to pick
alone at four in the afternoon.

So we did not pick. We shipped a prop and labelled it a prop: a placeholder where rows turn into domain objects, a WARN
carrying the offending keys so the data can be characterized later, and a `TODO` listing all three candidates for
whoever gets to choose.

```java
.title(titleOrPlaceholder(row.get_Key(), row.getCourseTitle()))
```

With the new baseline:

* We cut a fix branch.
* Applied the placeholder at the two points where rows become domain objects.
* Ran the checks and the course-offering tests. Clean, in well under a minute.
* Pushed the PR; the streamlined pipeline cleared in minutes.
* Merged to main and verified the fix in QAT in under 30 minutes from initial triage.

And here is the part I want to be precise about, because it cuts against the obvious moral: **the fast loop did not
produce a test for the placeholder.** There deliberately isn't one. `"NO TITLE"` is not behavior we want to lock into a
test, because we intend to delete it — asserting on it would make the scaffolding harder to remove than it was to add.

The speed mattered for a different reason. It made it cheap to demonstrate that a two-line change to a mapping function
had disturbed nothing else. That is what lets a stop-gap ship as an honest stop-gap, with its expiry date attached,
instead of quietly growing into an afternoon of hedging and a test that enshrines the workaround.

There was no rushing and no frantic corner-cutting. The team moved fast simply because the latency of verifying
correctness had collapsed.

## Why Feedback Loops Dictate LLM Collaboration 
Working with an LLM fundamentally alters how software is authored. You move
away from typing syntax line-by-line and toward steering, reviewing, and verifying high-level intent.

In this paradigm, the cost of verification is the governing constraint.

```
[Prompt Step] ---> [LLM Generation] ---> [Verification Run]
      ^                                          |
      |                                          v
      +<------------- (Stack Trace / Feedback) --+
```

When local verification takes six minutes, you are forced to ask the LLM for large batches of work: “Implement this
entire service, generate three test classes, update the configuration, and wire the controllers.”

Large prompts create broad semantic drift. When something fails six minutes later, isolating whether the bug came from
the architecture, the prompt, or a subtle hallucination is excruciating.

When verification takes 58 seconds, your step size shrinks to the atomic level:

* Ask for a single refactoring step.
* Run the suite in under a minute.
* Feed the exact error or green output back into the context while the conversation state is pristine.

Small steps keep cognitive context fresh for both the developer and the LLM. Error rates drop, regressions are caught
immediately, and the compounding velocity of the pairing session surges.

### Summary Matrix

| State                         | Execution Time | Primary Mechanism                                               | Impact                                            |
|-------------------------------|----------------|-----------------------------------------------------------------|---------------------------------------------------|
| Initial baseline              | 6m 00s (360s)  | One fork, context rebuilt per class, cold container every run   | High-friction batched testing, context drift      |
| Act I (during PR wait)        | 3m 44s (224s)  | `@DirtiesContext` traded for targeted `@AfterEach` cleanup      | 38% reduction, achieved during dead pipeline time |
| Act II (during Veracode wait) | ~2m 00s (120s) | Netty/servlet shutdown tail removed; dead config measured, kept | 66% cumulative reduction                          |
| Act III (both switches on)    | 58s            | Reusable container plus four test forks, both opt-in            | 84% reduction; sub-minute feedback loop           |
| Default (nothing set)         | 1m 44s (104s)  | Structural cleanups only, single fork, no container reuse       | 71% reduction for every developer, out of the box |

One caveat on that table: the three acts are a narrative over two pull requests, not three. The `@DirtiesContext` work
merged on its own; everything in Acts II and III landed together. The per-act times are what I measured as I went, but
if you go looking for three commits you will find two.

## Systems Takeaways

* Exploit Queue Latency: Asynchronous pipeline waits are an inevitable reality of modern software engineering. Treating
  these pauses as exploratory optimization windows allows you to compound value without needing dedicated "refactoring
  sprints."

* Lowering Activation Energy Changes Behavior: Developers don't avoid running slow test suites because they are lazy;
  they avoid them because context-switching is cognitively expensive. When running the suite takes under a minute,
  running tests becomes continuous and instinctive.

* Safety Enables Speed; Speed Enables Safety: Fast feedback loops do not require cutting corners. By wrapping aggressive
  optimizations behind safe opt-in flags, you gain the benefits of extreme velocity immediately while giving the system
  time to prove its reliability.