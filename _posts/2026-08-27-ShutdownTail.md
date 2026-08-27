---
layout: post
draft: true
title: "The Shutdown Tail: 27 Seconds Your Spring Test Report Doesn't Show"
description: "Reactor Netty waits out a two-second quiet period when it disposes its event loops, and Spring pays it once per cached application context, serially, in the shutdown hook — after your last test has already passed. Two lines of test configuration remove it."
category: engineering
tags: [ spring, testing, gradle, performance, reactor netty, feedback loops ]
---

> Note: this one was drafted by Claude (Opus-5) from my notes and verified against the commit it describes. It is a
> short extraction from a longer pair of posts about the same afternoon:
> [Shrinking the Feedback Loop](https://schuchert.github.io/engineering/2026/08/26/Shrinking-the-FeedbackLoop-From6Minutesto58SecondsWithAnLLM.html)
> and [Opportunistic Optimization]({% post_url 2026-08-27-OpportunisticOptimization %}).

## The symptom

Your test suite goes green. The last test scrolls past. And then the process just sits there.

Not long enough to make you investigate. Long enough that you have noticed it, more than once, and filed it under
"Gradle being Gradle."

On our suite that interval was about 41 seconds, and roughly 27 of them had a single cause.

## What it is

Reactor Netty will not drop its event loops without first waiting out a two-second "quiet period." The purpose is
graceful shutdown: give in-flight requests a moment to finish before the sockets go away. In production that is
correct behavior and you want it.

Spring's `ReactorResourceFactory` pays that two seconds on context close. In a test JVM, contexts are *cached* — the
whole point of the Spring TestContext framework is that a context gets reused across test classes that share a
configuration — and every cached context is closed at the end of the run, in the JVM's shutdown hook, one after
another.

So the cost is not two seconds. It is two seconds times the number of distinct cached contexts, serially, *after* the
last assertion has already passed and the report already says green.

Twenty-seven seconds implies something like thirteen cached contexts. That is an unremarkable number for a Spring
codebase of any size.

And none of those contexts had an in-flight request to drain. Not one. No test does.

## Whether it is happening to you

Two conditions, both easy to check:

1. **Reactor Netty is on your test classpath.** You do not need to be a WebFlux application. A `WebClient`, a
   reactive health check, or a transitive dependency from an internal starter is enough.
2. **You cache more than one Spring context.** If your suite has exactly one, this costs you two seconds and you will
   never care. The cost scales with the number of distinct context configurations, which is the same thing that grows
   quietly as a codebase accumulates test slices.

The way you notice it is the gap between the last test line and your prompt coming back — not anything in the test
report, which stops counting when the tests stop.

## The fix

Two lines in the `Test` task configuration:

```kotlin
// Reactor Netty waits out a 2s "quiet period" when it disposes its event loops. Spring's
// ReactorResourceFactory does that on context close, so the test JVM pays a flat 2s for
// every cached Spring context, one after another, in the shutdown hook after the last test
// has already passed. That was ~27s of a ~41s tail. Nothing in a test needs the grace
// period: there are no in-flight requests to drain.
systemProperty("spring.reactor.netty.shutdown-quiet-period", "0s")

// Same idea for the servlet container: graceful shutdown exists to drain live traffic,
// which no test has.
systemProperty("server.shutdown", "immediate")
```

The second line is the same argument applied to the servlet container. Graceful shutdown exists to drain live traffic.
Tests have no live traffic.

I want to be precise about why this is safe, because "make shutdown faster" sounds like the kind of change that trades
correctness for speed. It doesn't, and the reason fits in one sentence: the quiet period protects requests that are
still in flight, and a test JVM at shutdown has none. This is not a tuned-down timeout. It is the removal of a wait
whose precondition is never met.

Scope it to your test tasks anyway. Nothing here belongs in a running service.

## Why this is my favorite kind of waste

It is invisible in every report you already look at. It costs nothing to remove. And the safety argument is one
sentence long, which means you can review it in about the time it takes to read it.

I found it while looking for something else — specifically, while looking for anything at all to do during a
ten-minute CI wait. The longer version of that afternoon, including a container-reuse change and a failure I never
managed to explain, is in the two posts linked at the top.

One caveat on the numbers: 41 seconds and 27 seconds are from one developer workstation, on one suite, with roughly
thirteen cached contexts. Treat them as the shape of the problem rather than a benchmark. The arithmetic is the part
that transfers — count your cached contexts, multiply by two seconds, and decide whether you care.
