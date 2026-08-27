---
layout: post
draft: true
title: "Opportunistic Optimization: Shaving Five Minutes Off the Loop While Waiting on CI"
description: "How sneaking in side-branch experiments with an LLM during pipeline queues dropped test execution from six minutes to 58 seconds—and turned an emergency production fix into a casual afternoon non-event."
category: engineering
tags: [ systems thinking, developer experience, continuous integration, llm, testing, architecture, feedback loops ]
---

> *Note: This article is a co-creation developed in collaboration with Gemini. Interacting with the model
served both as the optimization pairing partner during the session described below and as a co-author in reflecting on
the systems dynamics.*

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

Within seconds, the LLM flagged two structural drags that had quietly accumulated in the codebase:

1. **Over-Aggressive Test Cleanup Annotations:** Several integration test hierarchies relied on heavy lifecycle teardown
   hooks that cleared caches, wiped database state, and reset containers after every single test method. In practice,
   the underlying tests were purely additive or read-only and did not mutate shared state. We were paying a massive
   teardown penalty for isolation we didn't actually require.
2. **Constrained Concurrency:** The local test runner was configured conservatively, leaving multi-core host resources
   mostly idle while waiting on blocking I/O and setup routines.

We stripped out the unneeded teardown hooks, tuned the test execution forks to better match local core capacity, and
fired off a run:

```
[INFO] Total time: 03:44 min
```

From **6:00 down to 3:44** on the first pass.

By the time the remote ADO PR pipeline finished its ten-minute run, I had already shaved over two minutes off the local
loop on a separate branch.

---

## Act II: The Security Intermission (3:44 → ~2:00)

We merged the feature branch into `main`. Almost immediately, the automated pipeline hit a tripwire: a Veracode security
scan flagged a new vulnerability in a shared internal library dependency that had been published earlier that day.

This wasn’t something I could solve inside our repository directly. I pinged the developer maintaining the upstream
dependency. He hopped on it, bumped the vulnerable transitive dependency, patched the code, and kicked off an upstream
build to publish a new version.

That handoff meant another ten-minute operational pause.

Rather than sitting on my hands waiting for the new package artifact to drop, I pulled the latest `main` into my
optimization branch and kept pairing with Claude.

We went after deeper configuration overhead:

* **Spring Context Churn:** Integration test classes with slight variations in configuration annotations were repeatedly
  forcing the framework to tear down and rebuild entire application contexts. By consolidating test configurations into
  shared, reusable base profiles, we eliminated multiple expensive context reloads.
* **Redundant Dependency Verification:** We pruned unnecessary validation plugins from the local development profile
  that belonged exclusively in outer CI/CD stages.

Just as the notification popped up that the upstream dependency had published its fix, we updated our internal version
pointer and triggered another local run.

Execution time dropped to **just over 2 minutes**.

In two remote waiting windows, we had carved four minutes out of every subsequent test cycle.

---

## Act III: Containers, Ghost Failures, and `DANGEROUS_FAST_BUILD` (2:00 → 58s)

With the baseline down to two minutes, the remaining bottleneck was obvious: integration container lifecycles.

Our integration tests relied on spinning up local background containers to simulate dependencies. Starting, verifying,
and stopping these containers across distinct test slices added a stubborn floor to total execution time.

We reorganized the harness to reuse running container instances across test suites during local development. A clean
build where the background container was not yet initialized clocked in between **77 and 97 seconds**.

If the container was already warm and running, an incremental build with full test execution took **58 seconds**.

```
[INFO] ------------------------------------------------------------------------
[INFO] BUILD SUCCESS
[INFO] ------------------------------------------------------------------------
[INFO] Total time:  58.124 s
[INFO] Finished at: 2026-08-26T14:22:18-05:00
[INFO] ------------------------------------------------------------------------
```

### The Ghost in the Concurrency

During this aggressive pass, a test failure flashed across my terminal. When I immediately re-ran the suite to diagnose
it, the test passed cleanly.

Ghost failures are the bane of developer velocity. If a test suite is flaky, developers stop trusting it, rendering
speed gains meaningless.

Investigating the collision revealed the cause: while I was running verification runs from the command line, the LLM
background tooling was simultaneously executing its own test passes against the same shared local ports and container
instances. Two full test suites running in parallel on the same host had stepped on each other's state.

Even though the harness itself was sound, introducing radical container reuse carryover risks edge-case test leakage. We
needed a way to capture the performance gain immediately without imposing risk on the rest of the engineering
organization.

We applied a foundational systems principle: **Decouple the mechanism from the default path.**

We wrapped the aggressive container reuse and extreme parallelism behind an explicit opt-in environment variable:

```bash
export DANGEROUS_FAST_BUILD=true
```

If the flag is set, you get the sub-minute feedback loop. If the flag is absent, the build falls back to the safer,
fully-isolated lifecycle—which, thanks to the earlier structural cleanups from Rounds 1 and 2, still ran in 104 seconds
(down from the original 360 seconds).

We committed the changes, opened a PR for the build harness improvements, and established a one-week observation window
to verify stability before considering making it the default.

## The Crucible: A 30-Minute Production Turnaround

The real test of an optimization is not the benchmark you screenshot; it is how the system behaves under pressure.

Shortly after landing the build updates, our outer automated testing suite in the Quality Assurance Testing (QAT)
environment flagged suspect data behavior. It wasn't an outright crash, but an edge-case data inconsistency that needed
immediate correction.

In the old reality (6-minute local runs, slow PR pipelines, 10-minute CI verification), a bug discovered mid-afternoon
in QAT was almost guaranteed to become tomorrow's problem:

1. Formulate a hypothesis.
1. Run local tests (wait 6 minutes).
1. Adjust code, re-run tests (wait 6 minutes).
1. Push branch, await PR build and scans (wait 15–20 minutes).
1. Merge, deploy, verify.

By the time you complete two local verification cycles and one remote pipeline, the workday is over.

With the new baseline:

* We cut a fix branch.
* Applied a surgical data-patching fix.
* Ran the entire test suite locally in 58 seconds to confirm zero regressions.
* Pushed the PR; the streamlined pipeline cleared in minutes.
* Merged to main and verified the fix live in QAT/Production in under 30 minutes from initial triage.

There was no rushing, no frantic corner-cutting, and no skipped tests. The team moved fast simply because the latency of
verifying correctness had collapsed.

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

| State                          | Execution Time | Primary Mechanism                                         | Impact                                           |
|--------------------------------|----------------|-----------------------------------------------------------|--------------------------------------------------|
| Initial Baseline               | 6m 00s (360s)  | Sequential execution heavy teardown hooks                 | High friction batched testing context drift      |
| Round 1 (During PR Wait)       | 3m 44s (224s)  | Pruned redundant lifecycle annotations opened concurrency | 38% reduction achieved during dead pipeline time |
| Round 2 (During Veracode Wait) | ~2m 00s (120s) | Consolidated Spring test contexts pruned plugins          | 66% cumulative reduction                         |
| Round 3 (DANGEROUS_FAST_BUILD) | 58s            | Reusable container lifecycles opt-in gating               | 84% reduction; sub-minute feedback loop          |
| Default (Safe Fallback)        | 1m 44s (104s)  | Structural cleanups without persistent containers         | 71% reduction for all developers out of the box  |

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