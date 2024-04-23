---
layout: post
draft: false
title: This is one of those "May you Live in Interesting Times" days.
description: A field report from today on where the day went. It involved around 10 different technologies, because it was an interesting day.
category: [ software ]
tags: [ lambda, react native ]
---

{::options parse_block_html="true" /}

## How the Day Started
The morning started with a few minor UI improvements. We were making some simple layout changes, working in React Native.

That done, and pushed, and moved on to checking something in back end. We found and fixed an issue. 
That part of the system uses AWS Lambdas, and 10 other AWS systems.

Next, we realized we needed to change the key field of a new table. Switched to AWS CDT, found out you cannot change 
a key field. Finally fixed all of that.

Next, we started getting into what we really wanted to work on. We spend some time in the instructor portal, Angular JS, 
and verified a few things about how far we had gotten. Then we decided it was time to do a quick planning session.

## Plan Plan Plan
We spent maybe 30 minutes working on a decision tree for the new feature. We deferred several details for
later. We have an approach to maintain trunk-based development. We made several guesses, and then asked for
some confirmation. 

We had enough certain stuff to work on, so there was plenty of time to wait for those answers.

## The Build Failed

Having just finished a first pass at a decision tree, we noticed the last build had failed. We could not deploy to 
the Google Play Console, the Apple App store was fine (though we suspect that might change the next day).

Here’s the thread of events leading to that state:
* We have begun to fix a side issue related to devices logging in multiple times,  
* To help, we started asking for unique device IDs.
* The auto deployment was rejected because asking for a unique device IDs requires an updated policy document.
* That done, the builds continued to fail.
* The github action we were using, it appears, always pushed files into auto-review (test), but the responses to the policy document seem to disallow moving such deployments automatically into testing.
* Before we started asking for device id (the previous day), we had released the app to both the Apple and Google stores.

## Experiment and Learn Rapidly

It took googling, reading, guessing, and 4 tries:
![](/assets/images/TheFourCommits.png)

### hail marry, will this deploy?
* Tried adding a flag into the yaml using the existing plugin
* Failed, oops, that was for a different plugin, with the same purpose and a similar name

### hail marry 2
* Decided to that new plugin where that had the flag I needed to set
* Failed, new plugin uses camelCase, old uses kebab-case, otherwise most of their parameters are the same purpose and name

### hail marry 3
* Fixed the kebab-case to cameCase conversion, and did rename two fields
* Failed, typo and there was a missing file.

###  hail marry 4
*  The old plugin automatically unzipped files, the new plugin did not
*  Fixed by adding an unzip phase

## Started Getting Confirmation

That done, we were just about to get back into feature work, but one of our stakeholders both answered our questions,
and jumped in our zoom. We talked through some options, got some confirmation, and also got some confirmation on
a few near-term dates important to the project.

## Done?
This is just the latest in a series of events. Recently a number of things have happened:
* We tracked down a threading issue in our design, and we changed the order of things to avoid the problem
* Ran out of lambda space (bug in serverless framework, probably made worse by frequent commits to trunk)
* Switched to sso from access tokens
* An update to the Google Play Console required a different policy document review

We could go on, but while these hiccups are going to happen, it's nice to be working in a way where we can simply
resolve these issues as they occur.

