---
layout: post
draft: true
title: "Two Voices, Two Different Fabrications"
description: "I gave one model the same outline twice, changing exactly one paragraph of the prompt. It produced two posts with different voices — and different fabrications. Where the two drafts disagreed turned out to be where the model was making things up, which makes a paired draft a cheap fact-checking instrument."
category: engineering
tags: [ llm, writing, process, measurement, non-determinism ]
---

> Note: this post was drafted by Claude (Opus-5), which is also the model that performed the fact-check it describes,
> which makes it an artifact of its own subject. I asked for it, I edited it, and I am publishing it, but I did not
> write the sentences. The two posts under discussion are
> [Shrinking the Feedback Loop](https://schuchert.github.io/engineering/2026/08/26/Shrinking-the-FeedbackLoop-From6Minutesto58SecondsWithAnLLM.html)
> and [Opportunistic Optimization]({% post_url 2026-08-27-OpportunisticOptimization %}).

## The setup

I spent an afternoon making a test suite faster, and I wanted to write about it. So I did something slightly unusual.

I wrote an outline from memory — 800 words, all the timings, the order events happened in, what I thought had gone
wrong. Then I handed that outline to Gemini twice and asked for a blog post each time.

The two prompts were identical except for one paragraph. Same role, same task, same outline, same summary ideas, same
requested length, same output format. Only the `# Constraints` section differed.

The first said:

> I want the emphasis on this to be more of the story. I knew I was going to have to wait. We had been discussing unit
> test time, so I snuck in several side branches and side PRs to get faster feedback.

The second said:

> I want this to be in my style and have more of a general systems thinking flavor but not too jargon. There is already
> a lot in the outline, but it is fine to describe that less technically so long as the essence of multiple threads of
> work interleaving with feedback loops getting faster, leading to easier to make changes, and overall safety.

That is the whole experimental manipulation. One paragraph.

It worked, in the sense I expected. I got two genuinely different artifacts: one with Acts and beats and a
first-person narrator, one with Rounds and tables and a lessons section. At the sentence level they shared almost
nothing. Across 133 qualifying sentences the only near-identical pair was the boilerplate disclosure note both
carried at the top. I will come back to how that was measured, and to why the measurement is shakier than I would
like.

Then I published them and asked a different model — Claude — whether they were true.

## They were not, and they were untrue in different places

This is the part I did not anticipate, and it is the reason I am writing this post rather than the one I had planned.

I expected the errors to be shared. Same source outline, same model, same afternoon: surely the fabrications would
be common to both, and the accuracy pass would be one set of corrections applied twice. That is not what happened.
Each draft was wrong where the other was right.

| | Story arm | Systems arm |
|---|---|---|
| The concurrency observation | kept and expanded | kept, compressed to one clause |
| What actually caused the failure | right — "two full test suites running in parallel on the same host had stepped on each other's state" | wrong — "a race condition introduced by parallel runs" |
| Confidence level | wrong — "investigating the collision revealed the cause" | right — "the failure was **likely**…" |
| My one-week observation window | kept | dropped entirely |
| Fabricated console output | a full `[INFO] BUILD SUCCESS` block, Maven format, invented timestamp, in a repo that has never had a `pom.xml` | none |
| What the build flag gates | right — reuse *and* parallelism | incomplete — reuse only |

Read the second and third rows together, because that is the interesting bit. The story arm named the correct
mechanism and then overclaimed it as established fact. The systems arm hedged correctly and hedged the wrong thing —
it relocated "parallel" from *my run and the model's run colliding* to *a race condition in the code exposed by test
forking*. Those are different bugs with different fixes. The hedge was honest and the referent had drifted.

And the story arm — the one told to emphasize the story — is the only one that fabricated evidence. That is not
coincidence and it is not a defect of that particular sampling run. Narrative demands scene. Scene demands
concreteness. Concreteness with no source to draw on becomes invention, and a console block is the most
concrete-looking thing available. The systems arm was told to be less technical and drifted toward abstraction, which
is a failure mode too, just a less falsifiable one. **The constraint paragraph did not only choose the voice. It chose
the failure mode.**

The one error both arms shared is instructive for the opposite reason. Both moved the QA-environment data issue
toward production. When I went back to my own outline, I found why: I had written that the work sped up "the live to
production deployments" three sentences away from the paragraph where the actual data issue is correctly labelled
QAT. The ambiguity was mine. Both arms resolved it the same wrong way, because there was nothing in the source to
distinguish them.

So: six error sites. Five were disagreements between the two arms. The one shared error traces to an ambiguity in the
source rather than to the model.

## Which makes the paired draft a fact-checking instrument

I did not set out to build one. I set out to compare voices. But the arithmetic is hard to unsee: if you generate two
variants from one source and diff them, the disagreements localize the places where the model was interpolating rather
than reporting. Where it had something to go on, both arms say the same thing. Where it did not, the two arms invent
in different directions, and the diff points straight at it.

This is cheaper than fact-checking prose, which requires knowing the answer. A diff requires only two drafts.

It is not sufficient — the shared production error proves that, and it was the error a diff could never have caught.
But it is a filter that costs one extra generation and reads in about a minute.

## What none of it knew

There is a third layer worth naming, because it changes what the accuracy pass actually was.

The word counts: 800 in my outline, 1,399 and 1,862 in the two drafts, 2,336 and 2,825 in what I published. So the
fact-check added more words to each post than my outline contained in total.

Those words were not corrections. Almost none of the technical substance in the published posts appears in my outline
*or* in either draft. Reactor Netty's two-second quiet period. `@DirtiesContext`. Testcontainers reuse and why Ryuk
ignores reusable containers. The embedded-Kafka measurement — 147.4 seconds of test-class time with the broker off
against 139.8 with it on. The specific dependency that tripped the security scan. None of that was in any of the
first two layers, because none of the first two layers could see the repository.

Three sources, cleanly separated by accident:

* **My memory** had the arc, every timing, the correct hypothesis, and the caution.
* **Gemini** had the prose, and filled the gaps my memory left with plausible material.
* **The repository** had every mechanism, and nothing else had it.

## Then the accuracy pass ate the variation

Here is the second finding, and the one I originally thought was the whole post.

Correcting both posts made them into nearly the same post. Not in structure — one is still Rounds and the other is
still Acts. In sentences. Roughly fifteen new pairs of near-identical sentences appeared, clustered precisely in the passages
that had been corrected: the shutdown-tail mechanism, the container-reuse trade-off, the null-title defect. The most
technically substantial paragraphs in both posts had converged.

The measurement: split each post into sentences of eight words or more, drop the front matter and the fenced code
blocks, lowercase everything, compute Jaccard similarity on the word sets, and count the pairs above a threshold.

| State                            | pairs ≥ 0.60      | pairs ≥ 0.50 |
|----------------------------------|-------------------|--------------|
| The two original drafts          | 1                 | 3            |
| After the accuracy corrections   | ~15 *(see below)* | not measured |
| After a deliberate re-divergence | 1                 | 6            |

Two caveats on that table, both of which cost me the clean result I wanted.

**The middle row is not reproducible.** I never committed the accuracy-corrected drafts, so that file state no longer
exists. I have the count I took at the time and nothing to re-run it against. Worse, the script I used then reports 2
for the original drafts where the current one reports 1, so its boundary behaviour is soft and the middle figure should
be read as *on the order of fifteen*, not as sixteen exactly. In a post about the value of counting things, I failed to
preserve the artifact I counted.

**The re-divergence did not beat the baseline.** I claimed in an earlier version of this post that it landed below where
the drafts started. It did not. At 0.60 both states show exactly one pair, and in both cases that pair is the
disclosure note at the top, which is supposed to be identical. Loosen the threshold to 0.50 and the published versions
are *worse* than the originals — six pairs against three. The residue sits exactly where you would expect: the
shutdown-tail passage and the null-title passage, still at 0.56 and 0.57. Two sentences that say the same true thing in
different registers are still two sentences about the same narrow fact.

So the honest version of the finding is weaker than the version I first wrote. The accuracy pass caused real
convergence. The re-divergence removed the blatant part and left a measurable film of it behind.

One more thing the numbers show, which I like: the highest-scoring non-boilerplate pair in the *published* posts, at
0.54, is one Gemini produced on its own — "the primary constraint on this workflow is the cost of verification" against
"in this paradigm, the cost of verification is the governing constraint." It is present in the original drafts at the
same score. I did not cause it and I did not fix it. Some convergence is just two drafts reaching for the same idea.

**Voice has many degrees of freedom. Accuracy has few.** There are a hundred ways to convey how it feels to watch a
test suite sit there for forty-one seconds after going green. There is roughly one set of facts about why it does. When
a corrector replaces an invented sentence with a true one, the true one is heavily constrained — and the *feel* of it
is whatever the corrector's own register happens to be.

Worse, and more mundane: a single corrector writes each correction once and reuses it. That is not a property of
language models. That is what any careful editor does when handed two documents containing the same error, and it is
what I would have done myself.

Which means **a single reviewer across N variants is a convergence pressure.** The review does not have to be
homogenizing in intent to be homogenizing in effect. It just has to be efficient.

The repair was to correct each post again, separately, in its own register — same facts, written twice, deliberately.
The pronoun distribution came apart again too: the systems version runs nine
first-person plurals against a single singular, and that singular is inside quoted dialogue. The story version runs 36
singulars against 24 plurals — a mix, because "I" does things and "we" pairs with the model, which is a distinction
Gemini had drawn on its own and which I had no business flattening.

## What I would actually do differently

My first instinct was: fact-check the outline, not the drafts. Put the accuracy pass before the fan-out instead of
after it, so the variants are generated from an already-true base and are free to diverge as far as their prompts push
them.

That is wrong, and the provenance is what shows it. My outline was *already accurate on both points that went wrong* —
it named the concurrency collision and it named the one-week wait. Fact-checking it would have found nothing, and both
errors would have entered afterward anyway, because they were introduced during generation and introduced differently
by each constraint paragraph.

What survives is narrower and less satisfying:

* **Diff the variants before correcting either.** Disagreements are where the model was filling in. It is the cheapest
  signal available and it does not require knowing the answer.
* **Correct each variant in its own register.** Not because homogenization is a sin, but because if divergence was the
  point of generating N drafts, then a correction pass that collapses it has quietly undone the experiment.
* **Check whether it happened.** I did not notice the convergence by reading. I noticed because someone counted, and I
  would not have believed it from a subjective impression in either direction.
* **Suspect your own ambiguities.** The one error a diff could not catch was seeded by two sentences of mine sitting
  three lines apart.

## Limits

One trial, two posts, measured by the same model that caused the convergence and then repaired it. Treat it
accordingly.

Lexical overlap is a crude proxy for the thing I care about: Jaccard similarity on word sets measures word reuse, not
voice, and would miss a corrector who homogenized rhythm while varying vocabulary. And as the caveats above show, the
threshold does real work — the same states report differently at 0.50 and 0.60, and one of my two scripts disagreed
with the other on an input both could see. The direction of movement is what I trust. The magnitudes are soft.

And the differential-failure result is six error sites in one pair of drafts. It is the kind of finding that wants
replication before anyone builds a habit on it — though the habit it suggests costs one extra generation, which is
cheap enough to adopt on weak evidence.

What I am left with is not a law. It is a small correction to how I thought this worked. I assumed a model asked the
same question twice would be wrong the same way twice, and that accuracy and voice were independent knobs. Neither
held.
