---
layout: post
draft: true
title: "Bamboo: Embodying Three Principles"
description: 'What planting bamboo and "knowing better than the grower" taught me'
category: [ self ]
tags: [ practices ]
---

> Note
> This is a revised version of [An Original Blog Post](https://blog.objectmentor.com/articles/2011/05/24/bamboo-reminds-me-of-a-time-when).
> Updated and expanded using some more recent writings and using Perplexity.io as an assistant.

In 2009, I planted an eight-foot by two-foot plot of bamboo and made a decision I would later regret. After digging down
two feet into the ground, a brutal thirteen-hour process that moved 32 cubic feet of dirt and clay, I surrounded most of
the bed with protective lining to contain the invasive species. But instead of lining the front completely, I left it
open. I told myself I wanted the bamboo to move into my yard but not into my neighbor's yard. It seemed reasonable at
the time. It seemed clever, even.

I was wrong.

When I finally addressed this three years later by digging a two-foot-deep trench to add the missing barrier, the
physical pain of the work taught me something far more valuable than bamboo containment. That work became a kinesthetic
education in three principles that apply equally to software development, to learning, and to any endeavor where we want
to improve.

## The Initial Mistake: Why Practices Exist

I knew the right thing to do. I had read about how to plant invasive bamboo. I had all the information. I had a
two-foot-deep hole practically begging to be fully lined. The practice was clear, tested, and established by people who
knew what they were doing.

But I thought I was more clever.

In software, we call this "Not Invented Here" syndrome. In life, it's just plain resistance to following guidance that
doesn't match our ego's interpretation of the problem. There's a famous story about a manager of a Coca-Cola plant whose
efficiency numbers far exceeded his peers. When asked what his secret was, he said simply: rather than taking a practice
and modifying it to fit what the plant did, he modified the plant to match the practice. His secret was not trying to be
too clever.

The cost of my cleverness was six hours of brutal digging, a two-inch bruise on the palm of my right hand (even through
gloves), and three years of procrastination about a problem that compound interest made exponentially harder.

Later, I learned this lesson in software development as well. When I first encountered Uncle Bob Martin's Transformation
Priority Premise (TPP) and what Gojko Adzic calls "TDD as if you meant it," I resisted. These practices weren't what I
had done in the past. I recognized the resistance, I could feel it, and I mentioned it to Bob directly. He confirmed
that
yes, he practiced TDD "as if you meant it." I decided to try it on a kata I knew well.

I ate crow that evening.

The kata moved smoothly in a way it never had before. It didn't require the planning-ahead thinking that I normally
relied on. The practice worked better than my modification of it would have.

The deeper lesson here is about what happens when you resist change before you have direct experience with a practice.
If you decide to modify a practice to match your local conditions before fully understanding both the practice and its
intentions, the very problems that led you to consider adopting the practice in the first place may be the same things
that cause the practice to lose its teeth.

This doesn't mean you never adapt. It means you first understand deeply, then you have direct experience with the
unmodified approach, and only then, armed with knowledge of both the practice's intentions and your actual challenges,
do you thoughtfully adapt.

How do you recognize when you're resisting change rather than thoughtfully rejecting an idea? I learned this from
practicing yoga. When you pause and bring awareness to the moment, you can notice the felt sense of resistance. This is
not intellectual disagreement; it's a bodily sensation. If you're unsure whether you're resisting or rejecting, assume
you're resisting and try it anyway. **The cost of being wrong is usually lower than the cost of missing something that
works.**

For those wanting to develop this
capacity, [Jerry Weinberg's Alternating Hand Grip Exercise](https://schuchert.github.io/wikispaces/pages/AlternatingHandGripExercise.html)
is a simple practice that trains your ability to notice resistance and work through it. Or try yoga. Or pause whenever
you find yourself objecting to something and ask yourself: am I resisting the idea or the change? If you're not sure,
try it.

## Leave a Little Slack

When I started the trench, I marked where I wanted the new edge to be and dug a shallow trench roughly a foot wide. This
extra width served a critical function: it gave me room to move around, to attack the dirt at different angles, and to
position my body in ways that wouldn't have been possible in a narrow trench.

That "extra" dirt became useful. I used it to fill a hole my dogs had dug. The broader trench also meant I could use
different tools as the work changed. As I dug deeper, the angle of my work naturally shifted the trench narrower anyway.
The width I started with meant I could still move, still adapt. When I hit clay, the hard part, those larger shovels
became unwieldy. The angle was wrong, the density of the material was too much for the leverage they provided. I
switched to a hand-shovel for gardening. The narrower profile worked better at the angle I needed. And crucially, I
still had room to use it because I had started wide.

In software, "leave a little slack" has several meanings, and they're all connected to this principle. In the
*Principles of Product Development Flow*, Donald Reinertsen describes how organizations that exploit a constraint at
100% utilization lose all ability to absorb variation, adapt, or respond to interruption. The purpose of Slack is not
laziness, it's the enabler of flow.

In the practice of [Trunk-Based Ping-Pong](https://kodyfintak.com/trunk-based-ping-pong/), you intentionally take small
steps, committing and pushing frequently. **Small commits create maneuvering room to pivot, just like the wide trench
let me switch tools mid-dig.** This creates slack in your work, you're never far from a stable state. The
work-in-progress is minimal. The implications: if you're interrupted, you lose little. If you need to change
direction, you can pivot easily. If you discover a better approach, reversibility is cheap.

The practice works because it honors reality: you *will* be interrupted. Your understanding will evolve. The
requirements will shift. Small steps with frequent commits don't add more time, they add resilience. They give you slack
to absorb what's actually going to happen rather than what you expected would happen.

The same applies to practice and learning. When you embrace incremental development, whether that's practicing stretches
in Science of Stretching methodology, learning a new language, or mastering software development, the slack built into
small steps is what allows you to notice what's actually happening, adjust, and keep moving forward.

## The Right Tool for the Job

By the time I was about twenty inches into the ground, I had used three different tools: a regular shovel, a gardening
shovel, and a standard spade. Each worked for a while. None of them worked well once I hit the clay and needed
precision. The last inch cost me an hour.

I took a break to buy supplies. I also found a four-inch trench digger.

Within two minutes of using it, I did not regret the money. In fifteen minutes, I had moved more soil than the previous
hour with inferior tools. The tool transformed the work from grueling to manageable. If I never used that trench digger
again, it was still worth buying.

My father grew up on a farm in Iowa, **the same one I eventually grew up on**. He frequently reminded me: use the right
tool for the job. It will go smoother. You might be able to use another tool, but if you use the right tool, the work
simply flows better.

This principle is subtle because it's easy to get wrong. We can mistake the apparent tool (the shovel) for the actual
tool we need (the right design and approach). In software testing, for instance, many teams buy UI scripting tools
thinking they've solved the testing problem. The apparent problem is "how do we test the UI?" The real problem is
often "our code is coupled in ways that prevent easy testing." A UI scripting tool doesn't solve that; it just automates
the pain. The solution is design-based testing that makes most of the logic unit-testable so that UI testing becomes a
small, focused concern rather than the entire testing pyramid.

But sometimes the tool really *is* just the right tool. My wife bought a nice pruning tool. In the past, I would just
use a shovel to try and break roots. This was hard work. The difficulty grows at least with the square of the root
diameter as you get into thicker roots. When I started the trench work, I used that pruning tool instead. It required
more effort to find it, carry it to the backyard, and put it away. But it was exactly the right thing for cutting the
roots I encountered.

Using the pruning tool forced me to notice something: the "extra" effort to get the right tool up front pays immense
dividends. This is true across domains. In **Code Whispering**, the practice of listening to code rather than projecting
design onto it, you must first execute the code to understand it. You must use the right practices (testing, iteration,
actually running what you write) to hear what the code is trying to tell you. The "extra" effort of practices like TDD
is not extra; it's the tool that lets you think clearly about what you're building.

## What This Means

The bamboo is now contained. The trench is dug, lined, filled, and the yard is reseeded. My hands are sore, my forearms
are bruised, and I have learned something about the cost of ignoring practices, the power of building slack into your
work, and the way the right tool transforms effort into flow.

But the deepest lesson is about integration. **These principles aren't separate, they're humility (or surender) in action**:
respect tested practices, build adaptability into your process, choose tools for reality (not ego), and let direct
experience, not just intellectual understanding, reshape your approach.

The question remains: when you work, do you leave a little slack? When you practice something (TDD, yoga, stretching,
any discipline), do you get the "pure" experience first before adapting, or do you adapt from the beginning? And most
importantly: are you using tools that solve your actual problem, or tools that solve an apparent problem while
introducing worse ones? 

Some of us become test-infected in 1997, and some of us dig a two-foot-deep trench before we understand what we're
doing. Either way, when physical pain or code that refuses to work despite your intentions collides with your ego, you
have a choice: defend your approach, or learn something.

**The bamboo taught me to choose the latter.** 
