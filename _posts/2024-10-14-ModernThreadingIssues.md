---
layout: post
draft: true
title: Moder Threading Issues
description: While most modern stacks handle multiple users well, especially when those users do not share data, there are still cases where knowledge of parallel/threading issues is still good to keep in mind.
category: [ software ]
tags: [ threading ]
---

{::options parse_block_html="true" /}

## Background
A few decades ago I contributed two chapters on Threading to Clean Code. There's an upcoming reprint of that book
for its 15th Anniversary, and it brought that subject to my mind. I have not directly worked on multithreaded code in 
well over a decade, many of the problems and issues still exist in modern systems, even ones that handle all the
multi-user stuff for you.

## Problems
There are a number of issues to keep in mind when dealing with threaded implementations:
* Data Integrity
* Deadlock
* Livelock
* Starvation

We'll focus on data integrity here.

## Recently
Working on a multi-user system based in React-Native, with AWS Lambdas for a back-end, and storage in DynamoDB.

There's nothing inherently problematic with this kind of system. If two users do not share data, then there are
likely no data integrity issues. However, what happens when multiple users do share data?

Here are a number of examples:
* Joining a Room
* Multiple users working on the same "design"
* Players working in parallel to create a deck of cards
* Double deleting of resources on re-render using a 2D Library in React-Native
* Loading Icon Resources

We'll look at each one of these in a bit more detail.

### Joining a Room
We had a system that allowed players to log into a game. A typical game had a limit of 6 players, so after the
6th player joined, subsequent attempts to join the game were rejected.

The back-end system uses DynamoDb, which gives eventually consistent results.

For a number of months we've had an occasional kerfuffle where it seems we were losing players. It turned out to 
be an issue with when we decided to assign player roles.

Just joining a game did not cause data integrity issues. However, in the original implementation, we combined
joining a game, with assigning roles to players. Each of the first 6 players got a unique role, 1 - 6. 

Occasionally, two players would log in close enough to each other such that they'd both be assigned the same role.

While it would have been possible rewrite how we interacted with DynamoDB, we had a desire to change how players
joined a particular game anyway, so we split the work.

The original issue was assigning roles to multiple users joining a game from multiple mobile clients, that is, in
parallel.

Even joining a game was an issue, as it was possible to have enough players join at the same time, that the total 
count of joined players was also incorrect.

First, we expanded the domain. Rather than players joining a game, instead Players join a waiting room. At some
time later, they were assigned to games. 

This separated the multi-thread-safe part of the work, joining a game, from the not multi-thread-save part of the work, 
assigning user roles. Since two users could get the same role, rather than having it potentially triggered in parallel 
from the client app, we instead moved it to an administrator role.

The administrator created empty games, then assigned players to games. Since there is only one administrator, this
took the not multi-thread-save part of the code, and ran it in one thread.

In this case, the desire to introduce a waiting room pre-existed discovery of the bug. However, discovery of the
bug, raised the priority of moving to a waiting room.

### Multiple users working on the same "design"
A system allowing the shared-creation of a design for installation/updating of a commercial security system. For this
system, we used a Relational Database.

Such a plan has dozens of images, each image might have several elements, that might be connected.

How do you make sure that multiple people can work on the same design at the same time, without conflict?

For this particular domain, the number of users on a design is generally under 5, and the users tend to work on
different parts of an overall design. So while the possibility of conflict is there, given the typical use of 
the system, it's small.

Pre-existing in the system was the need to select an Elements edit them in any way. This became a natural place to lock 
an element. We use a flag on the element for whether it is locked. Any attempt to lock when already locked fails and 
informs the user that the element is locked.

Upon deselection, the element's lock is released.

With proper transaction isolation, this is a clean, safe solution to the particular problem.

### Players working in parallel to create a deck of cards
This is an example of an anti-case. We have another game we use for our workshops where we allow small groups of
players to compete in creating a deck of cards. 

In this case, we have up to 4 people creating a deck of cards. Those players are likely remote, but working together.
They certainly might be in a room where they can communicate, but they might not be. 

What happens if 2 players create the same card?

Players attempt to create a deck with certain characteristics. If they happen to get duplicate cards, the instructor
rejects their deck. However, either of the players can remove one of their cards. 

So in this case, there's a shared resource, the deck of cards. The addition of cards is done in parallel. There can
be a "data integrity" issue, duplicated work, but the end result is reject, fix, retry. All done through human 
interaction.

Early on we realized this was a potential issue. When we thought through solutions, we decided to play it by ear.
An instructor can run the game in different ways, with different rules. So this was a false positive.

### Double deleting of resources on re-render using a 2D Library in React-Native
We were using React-Native, with React-Native-Web to produce a web and mobile application for a customer. We 
additionally used a 2D Graphics Library called Skia.

Skia is a powerful library, but it has its own rendering engine, which means there are at least two rendering 
engines running at the same time: React-Native, and Skia. This was not an issue until we added in asynchronous 
loading of designs.

When a user selects a design, we load it asynchronously while moving into the design view of the application. When 
we started doing this, we found that Skia was frequently crashing. 

What ultimately caused this was that we were rendering the Skia view before we had finished loading. In React-native,
this is normal. It re-renders the view. However, since Skia has its own rendering thread, when React-Native
re-rendered the view that held Skia out of sync with Skia's own rendering, already-deleted memory was re-deleted.

The solution for this was to not render the Skia view until loading was done. We had well-defined locations where 
we needed Skia to be live and where it was not. We make sure to:
* Not render the Skia view until loading was complete
* Any time we issued a change that might cause a re-render of the skia view, we first cleared a setting, which would cause the Skia view to stop being rendered
* Then we'd set the a new value, causing everything to be rendered fresh

### Loading Icon Resources
In the same system mentioned in Double Deleting, we also had an issue with the order in which icons were loaded.
It was possible to load a design into the system before the code to load the icons had finished executing. When this 
happened, we'd have to re-load the design, or have several missing icons.

Since the resources were needed by other parts of the system, we moved them into a so-called provider in React-Native.
Then, any code that might cause a design to be loading always executed under that provider. The provider made sure
that the code to load resources had started, to guarantee icons would be available before it was possible to load
a design.

## Sharing Data
In all of these examples, there's some issue with sharing data:

* Allowing players to join and get a role assigned in parallel, with an eventually consistent database
* Allowing editing of a shared design my multiple users in parallel
* Allowing players to create cards in parallel
* Multiple, dependent rendering loops that are not coordinated
* Not controlling the execution / loading of resources explicitly

We took different approaches:
* Isolate the non thread-safe code and execute it later in a single thread
* Locking objects with a flag, using a relational database
* Allow it, but a human can choose to reject or accept based on the rules of the game
* Avoid using the second rendering thread until it's actually needed, and clear it before clearing everything else
* Make the loading part of a parent-child relationship, such that the child doesn't render until the parent is ready

Once you've discovered an issue with shared data, there are many approaches you might take to address it. We've seen
5 examples and 5 different approaches to specific problems. Are there any examples you've come acorss?
