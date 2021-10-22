---
layout: post
title: Volatile Blast From the Past
description: A memory from the past lead to my finally understanding the volatile keyword
tags: [java, threading]
draft: true
---
## Background

A group I'm coaching picked up work on a code base that needed some work. 
One thing it lacked was a post-deploy smoke test to make sure everything 
worked post deployment but pre release.

The team had already started capturing logs. They start tailing the logs,
publish a message on a topic, and wait for the dust to settle.
![](/assets/images/VolatileSystemDiagram.jpg)

The basic test starts after a build step has already deployed an instance
of the the application, a kafka listener, into PCF.

The first thing the test does (1a) is start pulling the logs. To do this it uses
a shell process and connects the output in to a log gobbler. The time this takes
to start varies (up to at least 10x at times, another issue there), so we 
check the contents of the logs grabbed until we see a particular message. 
That message is the indication that we can publish a message on a topic.

Once the message is published, the test then waits for another value. 
By wait, it checks until it finds it.

The overall test is limited to 60 seconds, but takes around 9, including everything
from starting gradle to verification of receipt of a processed message in the logs.

## Issues Abound
This test has a number of things going on at the same time.
1. It uses a process creator to create a process and then a thread to process 
 the results coming from the log gobbler.
2. It uses a thread to publish the message, though this really isn't necessary 
 and we'll probably remove that.
3. The main thread in which the test itself is running.

The challenge we have is that the startup time of the log gobbler varies by at least 10x
on the build box compared to a developer machine, but then only sometimes.

While the team initially though of using ```Thread.sleep()``` I pushed hard to 
make sure they didn't. This approach to solve timing issues makes them worse.
First, to make sure things work we have to sleep much longer than we think,
and as complexity increases, things slow down, we'll have to keep upping the 
sleep time.

Second, even if the test could finish in less time, it won't because the sleeps are 
hard-coded.

## What instead?
What I pushed for is polling with a long overall timeout. It might sound the same
but it is hugely different.

Let's look at both.

### Common (but highly dangerous and extremely not recommended) Approach

1. Start collecting the log
2. Sleep for 10 seconds (or some big number determined from trial and error)
3. Publish a message on a topic
4. Sleep for 10 seconds (or some big number determined from trial and error)
5. Assert the results

#### Why?

This **_is_** straightforward. It's easy to follow. It looks simple, but it 
is anything but any of those things. Looks are deceiving.

At the first sleep, how long is enough? If this is running on a non-virtual
build server, then you need to use a value that accommodates wide variances 
in execution time. You don't want inconsistent results, which are often
worse than no tests, so you make the sleep time much bigger than it needs to be.

I assume at least 10x execution variance, so if 3 seconds seems enough, then 
use 30 seconds. That means the test takes, assuming 30 seconds for both 
sleeps, 60 seconds of execution time. The actual value add time (actual
execution and assertion) is a second or two. 

Now multiply that number by the number of committers, mix in continuous
deployment, and understand that cost is done **_every_** day, multiple
times a day, and that adds up.

Now do that in 50 tests.

So what seems simple is:
* Unreliable, and error prone
  * This reduces trust in tests and leads to the death of your automated test suite
* Unnecessarily slow
  * I'll take slow and reliable over fast, but well-written tests are typically both fast **_and_** reliable

### My Preferred Alternative

We ultimately changed the test approach from tailing to getting recent logs.
This simplified the overall test, but an intermediate form that did not 
have any sleeps was both more reliable and faster.

The pithy principle is this
> Prefer Periodic Polling with long timeouts over sleeping

With this in mind, here's the same test updated:
0. Set an overall timeout for test execution (e.g., 120 seconds)
1. Start collecting the log
2. Look at the logs, waiting for a "signal" that indicates log gathering has started (we search for a particular message)
3. Publish a message on a topic
4. Look at the logs, waiting for a "signal" that indicates processing is complete
5. Assert the results

