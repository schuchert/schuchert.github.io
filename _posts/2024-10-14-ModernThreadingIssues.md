---
layout: post
draft: true
title: Modern Threading Issues
description: While most modern stacks handle multiple users well, especially when those users do not share data, there are still cases where knowledge of parallel/threading issues is still good to keep in mind.
category: [ software ]
tags: [ threading ]
---

{::options parse_block_html="true" /}

## Background
A few decades ago I contributed two chapters on Threading to Clean Code. There's an upcoming reprint of that book
for its 15th Anniversary, and it brought that subject to my mind. I have not directly worked on multithreaded code in 
well over a decade. However, many of the problems and issues still exist in modern systems, even ones that handle "all 
the multi-user stuff" for you.

## Problems
There are a number of issues to keep in mind when dealing with threaded implementations:
* Data Integrity
* Deadlock
* Livelock
* Starvation

We'll focus on data integrity here. The other three are well addressed by understanding a few fundamental algorithms:
* Producers / Consumers - deadlock
* Readers / Writers - starvation
* Dining Philosophers livelock

## Recent Data Integrity Issues
In the past few years we've come across a number of situations where a multi-user system had potential data integrity
issues. All of these share in common multiple simultaneous users (parallel), doing something that updated shared
data. 

All of these examples come from two different multi-user systems based in React-Native. All of them also use 
AWS Lambdas for a back-end, and storage in some data store.

There's nothing inherently problematic with these kinds of system. If two users do not share data, then there are
likely no data integrity issues. However, what happens when multiple users do depend on shared data?

Here are a number of examples that came up during development:
* Joining a Game
* Working on a shared Design
* Players working in parallel to create a deck of cards
* Double deleting of resources on re-render using a 2D Library in React-Native
* Loading Icon Resources

We'll look at each one of these in a bit more detail.

### Joining a Game
We have a system that allows players to log into a game. A typical game had a limit of 6 players, so after the
6th player joins, subsequent attempts to join the game are rejected. Each player is assigned a role, numbered 1 - 6.

The back-end system uses DynamoDb, which gives eventually consistent results.

For a number of months we've had an occasional kerfuffle where it seems we were losing players. It turned out to 
be an issue with when we decided to assign player roles. Just joining a game did not cause data integrity issues. 
However, in the original implementation, we combined joining a game, with assigning roles to players.

Occasionally, two players would log in close enough to each other such that they'd both be assigned the same role.

While it would have been possible rewrite how we interacted with DynamoDB, we had a desire to change how players
joined a particular game anyway, so we changed the when we assigned players roles.

The original issue was assigning roles to multiple users joining a game from multiple mobile clients, that is, in
parallel. To fix this, we first we expanded the domain. Rather than players joining a game directly, Players instead 
join a waiting room. Later, the administrator creates games and then issues a request to assign all waiting players to 
games.

This separated the multi-thread-safe part of the work, joining a game, from the not multi-thread-save part of the work, 
assigning user roles. Players instead joined a waiting room, which was thread/parallel-safe.

The administrator created empty games, then assigned players to games. Since there is only one administrator, this
took the not multi-thread-save part of the code, and ran it in one thread.

In this case, the desire to introduce a waiting room pre-dated the discovery of the defect. However, discovery of the
bug, raised the priority of moving to a waiting room.

### Working on a shared Design
Next up, a system allowing the shared-creation of a design for installation/updating of a commercial security system. 
For this system, we used a Relational Database.

Such a plan has dozens of images, each image might have several elements, that might be connected. How do you make sure 
that multiple people can work on the same design at the same time, without conflict?

For this particular domain, the number of users on a design is generally under 5, and the users tend to work on
different parts of an overall design. So while the possibility of conflict was there, given the typical use of 
the system, the likelihood of it happening was small.

Pre-existing in the system was the need to select an Element to edit them. This became a natural place to lock 
an element. We use a flag on the element for whether it is locked. Any attempt to lock when already locked fails and 
informs the user that the element is locked. Upon deselection, the element's lock is released.

With proper transaction isolation, this is a clean, safe solution to the particular problem.

### Players working in parallel to create a deck of cards
This is an example of an anti-case. We have another mobile game we use for our workshops where we allow small groups of
players to compete in creating a deck of cards. 

In this case, we have up to 4 people creating a deck of cards. Those players are likely remote, but working together.
They certainly might be in a room where they can communicate, but they might not be. 

What happens if 2 players create the same card?

Players attempt to create a deck with certain characteristics. If they happen to get duplicate cards, the instructor
rejects their deck. However, when this happens, the team can clean up their deck and resubmit it. 

In this case, there's a shared resource, the deck of cards. The addition of cards is done in parallel. There can
be a "data integrity" issue, duplicated work, but the end result is easy to recover from, all done through human 
interaction.

Early on we realized this was a potential issue. When we thought through solutions, we decided to play it by ear.
An instructor can run the game in different ways, with different rules. So this was a false positive.

### Double deleting of resources on re-render using a 2D Library in React-Native
We were using React-Native, with React-Native-Web, to produce a web and mobile application for a customer. We 
additionally used a 2D Graphics Library called Skia.

Skia is a powerful library, and it has its own rendering engine. This means there are at least two rendering 
engines running at the same time: React-Native, and Skia, and those two rendering engines are not synchronized. 
We did not notice an issue until we added asynchronous loading of designs.

When a user opens a design, we load it asynchronously while moving into the design view of the application. When 
we started doing this, we found that Skia was frequently crashing.

What ultimately caused this was rendering the Skia view before we had finished loading. In React-native,
this is normal. It re-renders the view. However, since Skia has its own rendering thread, when React-Native
re-rendered the view at an unexpected time, Skia would start to delete, then delete again the same resources. 
Deleting already-deleted memory killed Skia and required restarting the application. 

The solution for this was to not render the Skia view until loading was done. We had well-defined locations where 
we needed Skia to be rending and where it was not. We make sure to:
* Only render the Skia view when loading was complete, otherwise render an empty view
* Any time we issued a change that caused a re-render of the skia view, we first cleared a setting, which would cause the Skia view to stop being rendered
* Then we'd set a new value, causing everything to be rendered fresh

The end solution resulted in cleaner code overall.

### Loading Icon Resources
Another issue we encountered was indeterminate load ordering of resources. We had an issue with the order in which 
icons were loaded. It was possible to load a design into the system before the code to load the icons had started 
executing. When this happened, we'd have to re-load the design, or have several missing icons.

Since the resources were needed by other parts of the system, we moved them into a so-called provider in React-Native.
Then, any code that might cause a design to be loading always executed under that provider, that is, there was a 
parent-child relationship between the provider and anything using Icons. The parent initializes first, guaranteeing
the order of execution.

As with the previous example, this resulted in cleaner code. Things that were related moved closer together under
a single place, and we removed duplication.

## Sharing Data
In all of these examples, there's some issue with sharing data:

* Allowing players to join and get a role assigned in parallel, with an eventually consistent database
* Allowing editing of a shared design, by multiple users in parallel
* Allowing players to create cards in parallel
* Multiple, dependent rendering loops that are not coordinated
* Not controlling the execution / loading of resources explicitly in an asynchronous environment

We took different approaches for each of these problems:
* Isolate the non thread-safe code and execute it later in a single thread - isolate and separate
* Locking objects with a flag, using a relational database - optimistic locking
* Do nothing, a human can choose to reject or accept based on the current rules of the game
* Avoid using the second rendering thread until it's actually needed, and clear it before clearing everything else
* Make the loading part of a parent-child relationship, such that the child doesn't render until the parent is ready

Once you've discovered an issue with shared data, there are many approaches you might take to address it. We've seen
5 examples and 5 different approaches to specific problems. Are there any examples you've come across?
