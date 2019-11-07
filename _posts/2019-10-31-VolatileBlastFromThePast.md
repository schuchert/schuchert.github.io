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

The team had already decided to capture the logs to check that after sending
a message on a topic, that the listener received and procesed it.
![](/assets/images/VolatileSystemDiagram.jpg)

The basic test start after a build step as already deployed an instance
of the listener into PCF.

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
1. It uses a process creator to create a process and then a thread to process the the results coming from the log gobbler.
2. It uses a thread to publish the message, though this really isn't necessary and we'll probably remove that.
3. The main thread in which the test itself is running.

The challenge we have is that the startup time of the log gobbler varies by at least 10x
on the build box compared to a developer machine, but then only sometimes.

While the team initially though of using ```Thread.sleep()``` I pushed hard to 
make sure they didn't. This approach to solve timing issues makes them worse.
First, to make sure things work we have to sleep much longer than we think,
and as complexity increases, things slow down, we'll have to keep upping the 
seelp time.

Second, even if the test could finish in less time, it won't because the sleeps are 
hard-coded.

## What instead?
What I pushed for is polling with a long overall timeout. It might sound the same
but it is quite different.
