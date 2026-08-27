---
layout: post
draft: false
title: "Shrinking the Feedback Loop: From 6 Minutes to 58 Seconds with an LLM"
description: "How interleaving build optimizations during pipeline wait times collapsed our local test cycle, supercharged developer-LLM iteration, and turned a QAT data defect into a 30-minute non-event."
category: engineering
tags: [ systems thinking, developer experience, continuous integration, llm, testing, architecture ]
---

> Note: This article is a co-creation developed in collaboration with Gemini. Interacting with LLMs served both as the
> optimization catalyst during the session described below and as the co-author in structuring these reflections.
>
> There are two flavors of this post. I wrote an outline from memory and had Gemini draft it twice from deliberately
> different prompts, to see what the framing alone would change. This is the analytical version; the narrative one is
> [Opportunistic Optimization]({% post_url 2026-08-27-OpportunisticOptimization %}). Same afternoon, same numbers,
> different shape. A later pass with Claude checked both drafts against the actual commits and build files.

## Overview

In software development, cycle time dictates behavior. When a local test suite takes six minutes to execute, developers
adapt to the delay: they batch changes, hesitate to run the full suite before committing, context-switch to other tasks,
and rely on remote continuous integration (CI) pipelines to catch regressions.

Every increase in feedback latency introduces friction. It forces human working memory to swap out context, widens the
blast radius of small errors, and cripples the efficiency of pairing with Large Language Models (LLMs).

During a recent deployment cycle, rather than accepting remote build queues as dead time, we pointed an LLM directly at
our test configurations and build harness. Through three interleaved iterations, local execution dropped from 6 minutes
down to 58 seconds.

Collapsing this feedback loop didn't just save five minutes on a timer—it fundamentally altered our development cadence,
dramatically improved the efficacy of LLM pairing, and took a QAT defect from initial triage to a verified fix in
under 30 minutes on the same afternoon.

## The Economics of Feedback Latency

In The Principles of Product Development Flow, Donald Reinertsen emphasizes that queue time and batch size are the
primary drivers of cycle time and economic waste. When feedback loops are slow:

Batch sizes expand: Developers defer verification until they have accumulated multiple logical changes, making
root-cause analysis harder when a test fails.

Context drift accelerates: During multi-minute waits, human attention drifts away from the problem space, incurring a
high cognitive reload cost when returning.

LLM leverage degrades: Pairing with an LLM relies on tight, iterative probe-and-verify loops. If verification takes five
minutes, you cannot afford to take the tiny, exploratory steps where LLMs excel.

| Metric                             | Initial State         | Optimized State | Delta                   |
|------------------------------------|-----------------------|-----------------|-------------------------|
| Local incremental test run (warm)  | ~360s (6m 00s)        | 58s             | 84% reduction           |
| Local clean build (cold container) | ~360s (6m 00s)        | 77s – 97s       | 73–79% reduction        |
| Default build, no switches set     | ~360s (6m 00s)        | 104s            | 71% reduction           |
| Triage to fix verified in QAT      | Multi-hour / Next-day | ~30 minutes     | Massive throughput gain |

All figures are from one developer workstation, so read them as ratios rather than benchmarks. The 6-minute baseline and
the 77–97s cold figure both pay container startup; the 58s figure is a warm run against a container that is already up.

## The Three-Round Interleaved Optimization

The optimization work was not scheduled as a multi-sprint infrastructure initiative. Instead, it was interleaved
directly into the idle windows created by remote CI pipelines and dependency hand-offs.

```
[Main Track] -- Work Completed -> [PR Build (~10m)] -- Veracode Wait (~10m) -> [Merge to Main]
                                        |                     |
[LLM Pairing]                           +-> Round 1 (3m 44s)  +-> Round 2 (~2m) -> Round 3 (58s)
```

### Round 1: Profiling and Pruning Redundant Work (6:00 → 3:44)

While waiting roughly ten minutes for an Azure DevOps (ADO) PR build process to validate a completed branch, we fed the
local build configuration and test harness structure to Claude (running Opus-5).

The first finding was a sledgehammer where a scalpel would do. Fifteen test classes carried `@DirtiesContext` — mostly
`AFTER_CLASS`, one `AFTER_EACH_TEST_METHOD` — which tells Spring to discard the cached application context and build a
fresh one. That is the most expensive isolation mechanism the framework offers, and in these classes it was mostly there
to undo a couple of inserted rows.

Deleting the annotation was not the whole change. Three of those classes did genuinely need cleanup, so they got the
cheap version instead:

```java
@AfterEach
void cleanup() {
    repository.deleteAll();
}
```

Same isolation, a tiny fraction of the cost. That one substitution, across fifteen classes, took local run time from 6
minutes to 3 minutes and 44 seconds — before the remote PR build had even finished.

Worth recording what we deliberately did *not* touch. The test runner was pinned to a single fork on purpose, because
parallel forks had previously caused embedded-Kafka shutdown races. More concurrency was obviously available, but it was
the one change that could reintroduce flake, so it waited for Round 3 and an opt-in flag.

### Round 2: The Security Scan Intermission (3:44 → ~2:00)

Following the PR merge, an automated Veracode scan flagged a vulnerability that reached us through a shared internal
library. Two things had to move: a Bouncy Castle bump in our own dependency catalog, and a fresh build of the upstream
internal security library, which meant coordinating with the developer who maintains it—an operational pause of about 10
minutes.

Instead of idling, we measured something the test report does not cover: the interval between the last assertion and
the process actually exiting. On this suite that interval was 41 seconds, and about 27 of them traced to a single
mechanism.

Disposing Reactor Netty's event loops involves a two-second quiet period, whose purpose is to let in-flight requests
finish. `ReactorResourceFactory` pays it once per cached application context, serially, from the JVM's shutdown hook.
Tests hold no in-flight requests, and the suite caches enough contexts for two seconds apiece to accumulate. Two lines
of test configuration remove the wait:

```kotlin
systemProperty("spring.reactor.netty.shutdown-quiet-period", "0s")
systemProperty("server.shutdown", "immediate")
```

The same pass produced a deletion candidate that survived measurement. A global embedded-Kafka property declares no
topics, and every Kafka test class names its own, which is what made it look like residue. Timing the suite both ways
gave 147.4 seconds without the property against 139.8 seconds with it: six test classes had been sharing a single
broker, and removing the property would have had each of them start one at roughly 1.9 seconds. The property stayed,
and both numbers went into the file beside it.

Local build and test execution dropped to approximately 2 minutes.

### Round 3: Container Lifecycles and Safe Opt-In (2:00 → 58s)

With the baseline down to two minutes, the remaining floor was the SQL Server container the full-text search tests
need. Under emulation it costs about 25 seconds to start and seed.

Testcontainers supports marking a container reusable, which leaves it running after the JVM exits. A later run attaches
to the warm container and skips both startup and seeding. That accounts for the gap between the 58-second incremental
figure and the 77-to-97-second cold figure, where the container still has to be created.

Reuse relocates the cleanup problem rather than removing it. Ryuk, the Testcontainers reaper, is designed to ignore
reusable containers, since that is what allows them to outlive the JVM, so disposal has to come from somewhere else.
Stopping the container when the suite ends would give back the cost it was introduced to avoid, so the build arms an
idle timer on each `test` invocation instead. Any later run resets it, and the container is removed only after an
interval with no test activity.

Then a test failed. `TrainingFilterIntegrationTest` reported missing data, and re-running it passed. Fifteen further
attempts, several under full CPU saturation, all passed. The cause was never established.

One property of the environment is not in doubt. Two suites were running against the same host concurrently: one driven
from the command line, one from the model's own verification passes. That is what was happening, not a reconstruction
after the fact. The most plausible reading is that two simultaneous suites contended for something — ports, the shared
container, database state — and that the failure belonged to that configuration rather than to the code. Four forks make
such a collision easier to reach, since which class lands in which JVM varies between runs, and reuse removes the
per-fork container that previously kept runs apart.

Plausible is where the evidence stops. That is enough to act on and not enough to close, which is the case the next
decision is built for: decouple deployment from activation. Capturing the speedup did not require explaining the
failure. It required the speedup to be something a developer opts into and the build agents do not.

So: two independent switches, deliberately not one.

```bash
# In your shell — four test forks for the module that benefits
export DANGEROUS_FAST_BUILD=true

# In ~/.testcontainers.properties — keep the SQL Server container warm between runs
testcontainers.reuse.enable=true
```

Unset — which is everywhere except a developer shell that exports it — reproduces the previous single-fork behavior
exactly, and CI pins reuse off explicitly rather than relying on no agent happening to have that file. Even with both
switches off, the structural cleanup had taken the default build to 104 seconds, still more than 3x faster than the
six-minute baseline. The PR went up with a one-week observation window before either becomes a default.

## Why Feedback Loops Compound with LLMs

Pairing with an LLM shifts software development from manual syntax generation to high-speed hypothesis testing. The
primary constraint on this workflow is the cost of verification.

```
       +-------------------------------------------------+
       |                                                 |
       v                                                 |
[Prompt / Small Step] ---> [LLM Generation] ---> [Fast Test (58s)]
       |                                                 |
       +<------------- (Failure: Rapid Context Reset) ---+
```

### Controlling Context Drift

When an LLM produces code, subtle semantic drift can occur. If your feedback loop takes six minutes, you are
forced to ask the model for larger batches of work to justify the wait time. Larger batches increase the probability of
hidden errors and compound hallucinations.

When the feedback loop takes 58 seconds, you can operate in micro-steps:

* Ask for a surgical refactor.
* Watch the full suite finish inside a minute.
* Instantly feed any stack trace or regression back into the LLM context while the reasoning chain is still fresh.

### Lowering the Activation Energy of Testing

A six-minute test run creates mental hesitation. You ask yourself: “Do I really need to run the whole suite for this
minor change?”

A 58-second test run eliminates that hesitation. Verification becomes continuous rather than batched, catching
regressions at the exact point of introduction.

## The Real-World Payoff: A 30-Minute QAT Turnaround

The practical value of this optimization became clear shortly after completing Round 3.

A GraphQL query against the Quality Assurance Testing (QAT) environment began returning a 500. The mechanism is worth
stating precisely, because the blast radius is disproportionate to the defect. Two of sixty course-offering rows carried
a NULL title against a schema that declares `title: String!`. GraphQL resolves a non-null violation by propagating the
null to the nearest nullable ancestor, which in this connection is the response root. Two rows therefore returned
`data: null` for all sixty.

Under previous conditions this would have triggered a protracted cycle: reproduce locally, wait six minutes per
verification, push to CI, wait for long remote pipelines, and likely carry the fix into the next working day.

Three candidate fixes existed — clean the source data, add a constraint to the view, or exclude the rows in the query —
and choosing among them is a data-ownership decision rather than a code change. None was going to be settled that
afternoon. What shipped was labelled scaffolding: a placeholder substituted where rows become domain objects, a WARN log
carrying the offending keys so the data can be characterized later, and a `TODO` naming all three candidates.

```java
.title(titleOrPlaceholder(row.get_Key(), row.getCourseTitle()))
```

With sub-minute local verification and a faster PR build pipeline:

1. A patch branch was created.
1. Spotless, the compiler, PMD, and the course-offering tests all reported clean in well under a minute.
1. The PR was submitted, reviewed, built, and merged.
1. The deployment cleared the pipeline and was verified in QAT roughly 30 minutes after the first triage.

Note what the fast loop did *not* produce: a test for the placeholder. There deliberately isn't one, because `"NO
TITLE"` is not behavior we want to lock in — it is scaffolding with an expiry date and a `TODO` that names the three
candidate real fixes. Speed mattered for a different reason. It made it cheap to show that a two-line change to a
mapping function had disturbed nothing else, and that is what let a stop-gap ship as a stop-gap instead of expanding
into an afternoon of hedging.

There was no fire-drill atmosphere and no panic. The team moved quickly because the cost of verifying safety had
approached zero.

## Core Systems Lessons

* Exploit Idle Latency: Pipeline waits and upstream hand-offs are inevitable in distributed systems. Treating these
  pauses as exploratory optimization windows turns dead time into compounding leverage.
* Shrink the Batch Size: Large batch sizes are a symptom of slow feedback loops. Fix the loop, and smaller, safer batch
  sizes become the natural path of least resistance.
* Gate Aggressive Optimizations Safely: You do not need to explain every edge case to capture the gain. An opt-in
  switch (`DANGEROUS_FAST_BUILD=true`) lets the people who want speed have it now, while the default path and the build
  agents keep the behavior that was already known to work. The flag is worth most precisely when you *cannot* explain
  the failure yet.
* Speed Enables Safety: Slowness does not guarantee rigor; it incentivizes workarounds. When running a comprehensive
  test suite takes less than a minute, verification happens continuously, making the entire delivery pipeline
  fundamentally safer.