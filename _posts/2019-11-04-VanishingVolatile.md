---
title: Vanishing Volatile
layout: post
description: >
    Working on tests with code that is threaded lead to a situation where
    we needed to use the volatile keyword, until we didn't.
tagile: >
    If you need to use volatile, you might be (not) solving the wrong problem.
draft: true
---
## Overview

Recently I came across a need to use volatile. At the time my 
spider senses were tingling so I wasn't sure. Later, after confirming it 
was necessary, we changed the code to remove the need for the keyword. What 
follows is the same thing, a bit longer.

## The Beginning

A team I've been coaching was working on code that was multi-threaded.
More on that later, but in summary:
* One thread existed to capture log output from an application running in PCF
* Another thread waited on values in that log first wait to publish a message and then wait for a result.
* The group I was with didn't write the original solution

As I looked at the code collecting the log lines, it looked something like 
the following;
```java
public void track(String logLines) {
    this.logs += logLines + '\n';
}
```

This code is inefficient, to be sure, but when I saw that, and realized that
another thread was looking at the collected value, I was pretty certain
that we needed something like the following on the field:
```java
private volatile String logs = "";
```

The reason my spider senses were tingling was because I had read
[Concurrent Programming in Java](https://read.amazon.com/kp/embed?asin=B004V9OA84&preview=newtab&linkCode=kpe&ref_=cm_sw_r_kb_dp_51zWDbER7Z6XN) back when it was published.

We got the test passing and then "verified" that ```volatile``` was
needed by seeing the test fail without it and pass with it.

## Time, as usual, to the rescue

Later that evening I realized that the hunch I had was nearly correct. 
I was thinking in terms of values rather than refernces. ```String```s are constant.
{% include aside/start id="byRefByValue" title="By Referene, By Value" %}
My "main" language is C. I understand it the best. In C everything is 
```pass by value``. Don't argue with me. If you want the equivalent of pass 
by reference, you pass the **value** of the address of the thing. A pointer.

In Java, it's the same thing. But when you pass a non-primitive (instance
of a class), you pass a reference to the object. If you pass a primitive,
just like a reference, the value is copied. 

If two threads are looking at the same field and one changes it outside of
a synchronized block, the other might not see the change, thus ```volatile```.
I think what happens with references can happen with primitives, and
that's where the Atomic types added in Java 5 greatly reduced the need for
both the use of synchronized and situations where ```volatile``` are needed.
{% include aside/end %}

The Thread appending the logs did so by creating a new string and reassigning the field.

The ```volatile``` keyword informs java to not assume values 
are fixed. The thread waiting for the value, checking it every so often,
got a reference the first time it ran, but never checked the reference
again. The other thread changes the value (and reference) every chance it gets.
Take a look at the following:
![Snapshots in Time](/assets/images/VolatileThreads.png)

At Time 1, the "Log Gobbler" points to an empty string. As the main thread
starts, it picks up that reference, and caches it.

Also at Time 1, the gobbler thread starts and points to the exact same
object (a ```String```) in memory.

The main thread checks the log values, waiting for a signal to that it
OK to continue. However, since the thread assumes the underlying 
value of the field isn't changing, it keeps checking the original value.

Once logs start flowing, the gobbler thread picks up all the available
lines and appends them to its internal field. Since strings are 
immutable, the code actually creates a new string and sets the field
to the new string.

However, at T2, the main thread isn't paying attention because it is 
cached. In fact, as the code is written, that's what will always happen.

At Time 3 and going forward, nothing really changes.

So as written, the code requires ```volatile```.

{% include aside/start id="caching is evil" title="Caching is Evil" %}
This is a place where I usually go off on a rant about some thing
or another. The title says it all. Caching is evil. Much like 
we'll learn in the new Star War series, balance requires light and
dark. So sometimes the evil of caching (in terms of new failure 
modes) is outweighed by the value the caching brings.
{% include aside/end %}

## But do it better

String concatenation is inefficient. In this particular case efficiency 
doesn't matter. However, in addition to being inefficient, it makes 
things like threading a bit more of an issue.

Collecting strings in a StringBuffer is a better general solution.
As it turns out, it also resolves the need for the ```volatile``` keyword.

Now the field looks like this:
```java
private StringBuffer logs = new StringBuffer():
```

Since each thread uses the same reference, and also, since that reference
does not change, there's no need to mark this reference as ```volatile```.
This field could be ```final```, though I don't generally use ```final```.

Now both threads look at the same object, a StringBuilder, whose reference
never changes, but its internals change as one thread appends logs to it.
![Volatile Vanished](/assets/images/VolatileVanished.png)

## Conclusion

I don't have a conclusion so much as an a-ha. I was pleasantly surprised
that I guessed right for the need to use ```volatile```. I wasn't 
exactly on but I was in the right general area. 

I was more happy when we improved (and simplified) the implementation
and removed the need for ```volatile``` altogether.
