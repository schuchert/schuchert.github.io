---
title: Create First
layout: post
description: >
  How can we clean up code in a legacy code base when continuous integration 
  is not happening? Maintain backwards compatibility by creating first.
tagline: Making the impossible, possible
---
{::options parse_block_html="true" /}

## Summary

Working in a legacy code base where multiple teams are making updates, and 
merging to master is infrequent, presents challenges. It turns out that
we can ideas from  
[Refactoring Databases](https://www.amazon.com/Refactoring-Databases-Evolutionary-paperback-Addison-Wesley/dp/0321774515). 


In [Refactoring Databases](https://www.amazon.com/Refactoring-Databases-Evolutionary-paperback-Addison-Wesley/dp/0321774515), 
the recipes come in two varieties:
* Solutions where one client directly interacts with the database
* Solutions where multiple clients, on their own release cadence, interact with the same underlying database

{% include aside/start id="refactoring-defined" title="Refactoring Defined" %}
Refactoring defined
{% include aside/end %}

## Background
Imagine you have a mature code base that’s been in use for a decade. 
It is under active development, with multiple teams working in the 
code. While the code is does not have a full suite of regression 
checks, there are a number of automated checks that provide some 
idea of whether you've broken something.
{% include aside/start id="checking-versus-testing" title="Checking vs. Testing" %}
checking versus testing
{% include aside/end %}

Current development practices in this code base make continuous 
integration impossible, while this will likely change in the future, 
it is a reality now.
{% include aside/start id="continuous-integration-defined" title="Continuous Integration Defined" %}
What is CI - def and link
{% include aside/end %}

In this kind of situation, I've observed that developers will avoid 
refactoring because it’s likely that doing so will break the work of 
other people in other teams.

While we want to move to more frequent integration, and new code is 
better covered by automated checks, how can we safely change the 
structure of the code now, while we wait for the process changes to 
catch up?

## Legacy Refactoring
Michael Feathers defines legacy refactoring as "code without tests." Lets’ refine that a touch:

|----|----|
|tests|automated checks (be they micro, integration, regression, ...) |
|code|code that is in production (so whatever it is doing is "correct" unless we have explicit knowledge to the contrary)|

So applying these two changes we have:
> Code in production that does not have automated checks.

A typical first action with code lacking automated checks is to create 
characterization tests (checks), then refactor.

I will do this, but often I’ll make "safe" refactorings without automated 
checks. Safe in this context includes:
* A small number of refactorings, most of which are automated
* Refactorings that are semantically equivalent based on a solid understanding of the language in question
* Limited use of backwards incompatible changes, e.g., changing return type of a method, where the thing in question is "highly used."

### Example

Imagine you have a "large" (the box containing circles below) class that
is used in several places:

![](/assets/images/CreateFirst/InitialClass.jpg)

Let’s work through a few scenarios to see what options we have based on
what we’re trying to do.


#### Scenario 1: Move something with no external clients

Suppose you need to make a change to the code in the circle labeled F. 
Further, suppose that the containing class makes that difficult, e.g., 
the class uses a library that requires a license server. It cannot be 
instantiated on a developer machine as they lack a license (real thing 
from another project, not made up).

I might do what Michael Feathers calls “sprout class.” As there are no 
external users, and few places where the code is used internally, this 
is a quick change that is likely safe:

![](/assets/images/CreateFirst/SproutClassFromInternalOnlyMethod.jpg)


The code in the F circle becomes its own class and the one place where 
it was used in the original source now refers to this external class.

This allows quick improvement to F, simplifies the original class, and 
reduces the likelihood of merge conflicts.

This technique might, out of context, seem like the overall design is 
worse in that there are more classes. Also, this technique might make 
a class that is really just a function hiding in a class.

However, my definition of good enough is context dependent. Even if 
we made F its own class is worse design in an overall sense, if doing 
so makes it possible/easier to get automated checks on code I need to 
change, the “worse” is in fact “better.”

If that last paragraph is in any controversial, you can safely stop 
reading now.


#### Scenario 2: Moving something with external uses

Next, imagine you want to move the C circle instead. While in the original
example, there are four users of the code, 2 external, two internal. We can
externalize the code and maintain backwards compatibility. Doing this allows us
to increase the number of steps where the work is “stable.” By stable, I mean
still compiles, and existing automated checks still pass.  

The primary difference is what happens to the original method. In the first
scenario, the method is fully moved into its own class and code calling it, now
calls a method on the new class. In this next approach, the original method
still exists, but its implementation calls a method on the new class.

![](/assets/images/CreateFirst/SproutClassFromMethodWithExternalClients.jpg)

Notice that in this first version, we've created a new class that has the code
from C and other code that only C used.

Rather than remove the original method, we instead have it call into the new
class. This allows the code to be sprouted without having to change all
callers. It makes the change smaller, less likely to involve merge conflicts,
and I can safely do this kind of change in the absence of automated checks.

{% include aside/start id="pull-based-refactoring" title="Pull Based Refactoring" %}
I’m not going to do this without a reason. If this code does not need to be
changed to support new functionality, or fix a defect, then I’m not going to
change it. This is using pull, an actual need, rather than push, a possible
future need I’m taking care of just in case.

So in this situation, I’m sprouting this code because:
* I have a needed change
* The original class it was in is hard to instantiate
* My changes will be in C (or E or F)
* So I can now more easily get C under automated checks

So while I might start with what I’d call a “legacy refactoring,” it will continue with getting automated checks around C and/or the changes I make to C.
{% include aside/end %}

In this case, the next commit might be to update both clients and D to call out
to C.

![](/assets/images/CreateFirst/FullySproutClassFromMethodWithExternalClients.jpg)

However, those 4 changes could each be their own commit.

If we do all of those changes in one commit, the commit is more likely to
conflict with other changes simply because it is larger. So a larger number of
smaller commits can make merging go more smoothly.

More importantly, if you are working on a class that happens to be an
internally distributed library or used in other projects either as a binary
dependency or even a source level dependency, or is invoked reflective, it
might be the case that you don’t know all of the code that touches the original
method. 

Without a clear understanding of the code base, I can probably make the first
change, sprouting the class, safely. I might never make the next step, moving
all references to the internal version to the externalized version. 

However, irrespective of changing all the references, the first version with
the original method calling out gives enough wiggle room to get what I’m
touching now under automated checks. If I never make the global update of all
calls, it’s OK. 

## The First Step
The simple idea underlying this approach is that the first step is creational,
not destructive. Rather than killing the original method, which forces all
clients to change, keep it in place, but it now calls out.

This is protected variation. The original method remains in place, protecting
existing consumers from having to change. The underlying implementation
delegates to the new implementation, providing a single path of execution. If
there’s a need to eventually update all the client code, you can do it
incrementally. 

Since the required change can be done quickly, safely, and often with automated
refactorings, it can get merged into master sooner rather than later. This
allows us to make “big” changes in terms of getting work under automated
checks, and even significant redesign. However, we can do it in a way that
allows for quick turn around back to “green” that we can merge to master
frequently.

