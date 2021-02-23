---
layout: post
title: "Threading Story"
description: "A place where I used threading to speed up a problem, and made it harder for myself."
category: [java]
tags: [java, threading]
---

{::options parse_block_html="true" /}
## Overview
I was on an engagement around 1999 where there was a program someone and written to transfer a db/2 mainframe-based 
data set into a different SQL db. 

It needed to run daily.

It took 26 hours to run once.

The program did 3 things:
* issue a dos batch command with a db key and wait for a text file to be created
* read the text file (xml for a row of big flat table) and translate into ORM object
* Write object to db

The program was single-threaded, and the first step took between 30 and 90 seconds. So the program was waiting for
I/O around 97.5% of the time.

## Threading to the rescue
I took the existing code and broke it up horizontally:
* first layer, run the dos batch command
* second layer, translate
* final-layer, write to db

Note, that was how the original code was written. I followed that original design and put threads on top of it.
I set up a set of queues between these layers and implemented the [producer/consumer algorithm](https://en.wikipedia.org/wiki/Producer%E2%80%93consumer_problem) 
between each.

The program eventually worked. As there were three stages, an input queue, queues between stages, I ended up
creating thread pools:
* pool 1, size 5, get a record from the db
* pool 2, size 5, transform
* pool 3, size 5, write to db

We ran parallel transactions because the keys were guaranteed unique given the source data.

## Results
This was early days in Java. The JVM didn't even have native threading. Even so, the time went from around 26 hours to 
16 minutes (that's where the 97.5% came from above).

It took me just over a week to do that. 3.5 days of that week was getting the program to properly shut down when it 
was done. The problem was trying to figure out when each queue was done versus empty. How does the consumer of a queue 
know that the producer is no longer going to publish. That is, it is done?

I'm not going to answer it other than to say [semaphores](https://en.wikipedia.org/wiki/Semaphore_(programming)), and
look at the algorithm mentioned above. It took 3.5 days to figure all of that out to avoid deadlock on shutdown. 
The program was running and translating for those days, and we used ctrl-c to kill it until I figured out how to get 
it to terminate cleanly. (Running in a test mode only, we didn't switch to it until it shut down properly.)

In retrospect, the three steps (extract, translate, load - ETL), is part of a single unit of work. No one part of it
is complete. So breaking it up by technical step might have seemed logical, but had I instead simply had one queue,
and multiple threads to handle each instance of the unit of work, it would have removed most of the development time.

That is, 1 thread pool:
* thread pool 1, size 5, get record, translate, write to db

{% include aside/start id="functional" title="Still do functional decomposition" %}
Even though we are putting an entire logical unit of work into a single thread, that doesn't mean the implementation
of the unit of work is all one large block of code.

I would still have different functions (and maybe even different classes, depending on the particular problem). 

That is, functional decomposition and decomposing work into threads have some things in common, but whereas more,
smaller, functions are often good, nearly the opposite is true for threads because they are not a light-weight tool.

Really, comparing the number of functions and number of threads is nonsensical, but then so was arbitrarily splitting
a logical unit of work across three threads, ostensibly, so I could independently scale each of the logical blocks.
In reality, because I love pipe and filter styles of programming and at that time had not yet separated the 
style of pipe and filter with their implementations (same process space/different process spaces), because most of
my use of that style was from unix command line use.

As it turned out, with DOS the limit was on number of batch files running. Five was stable, 6 was reliably unstable.
So having the capability to independently scale each of the steps of the process was entirely over-design that 
increased the effort at least 5 fold.
{% include aside/end %}

Splitting the problem by logical step (or layer) did not make the solution cleaner. It made it much worse. The logical
break lead to 2 additional queues, a total of six sets of semaphores (if I'm guessing, it was over 20 years ago), and 
it would have made that 3.5 days take probably a few hours.

In modern java, if you want something similar, use a ThreadExecutor. If you split the problem by complete unit
of work, then you can pre-load the queue, give it a number of threads, and it will complete the work and then easily
shut down. I did this work last century, just a few years after google existed, so we've gotten better.

If you are using Spring, you can use @EnableAsync on your @Configuration, and @Async on methods to have them do 
their work after an immediate return.

## Conclusion
Don't use threading.

If you do, keep it simple.

Use modern approaches over hand rolling solutions.

