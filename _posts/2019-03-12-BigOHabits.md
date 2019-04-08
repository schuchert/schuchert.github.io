---
title: O(1) habits
layout: post
description: >
  An incomplete idea I working on related to how people navigate code and other such things that seem to relate to DevOps thinking.
tagline: I wonder how this idea might impact your daily routine?
draft: true
---
## Background

Lately I've been mobbing or coaching mobbing with several people, sticking with a team for several weeks to months before moving on.

I notice one particular habit and I wonder if you've noticed it, or if you
do it, what is the underlying mental model.

To understand the habit let's use first pair programming. There's a [style
of pair programming](http://llewellynfalco.blogspot.com/2014/06/llewellyns-strong-style-pairing.html) where the person at the keyboard, the driver, is
doing, while the other person, the navigator, is giving direction.

So imagine we're working on a problem and you are the navigator. You want me
switch to the production code failing a test, and update the code to get the test green.
You tell me to go to the particular class. How do I get there?

Examples I've noticed:
* Scan the open tabs to see if it's up there. 
* Open up the project structure and start looking for the file.
* Use any of various shortcuts to get to the file

## How I Work

When trying to get to a particular class/file, assuming I don't 
have its location memorized (and even if I do), I'll use a direct access approach.

For example, if I'm using an IDE, I'll use its shortcut:

|Tool|Windows/Unix|Mac OS|
|----|------------|------|
|Idea|```ctrl-shift n```|```command-shift n```| 
|Eclipse|```ctrl-shift t```|```command-shift t```|
|Visual Studio Code|```ctrl p```|```command p```|
|Command Line|```vi `find . -name '*Name*.*'```|

These all work the same way. Go directly to a file, letting the tool 
figure out where it is.  I do not search tabs to see if a file is open. 
I do not search the project structure.

I work this way probaby because while my memory in the days timespan 
is generally pretty good, over time I forget where things are, or they 
could move (refactored into another package). So I happened across a habit 
that allows me to memorize less, get to a thing quickly, and relying on 
the computer to do things computers do well, find stuff.

It might stem from working at the command line for years before I had an IDE.

