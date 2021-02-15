---
layout: post
title: "Sed Breakdown dates and times"
description: >
  We often have data files that are not formatted in a convenient way. This is one of those examples, and how we used
  sed to fix it.
category: [Daily Tech]
tags: [unix, sed, regex]
---

# Overview

Sometimes we get input files in a format that doesn't work for us. We might ask the person to change their input format,
or we might just update it ourselves. In the example that follows, we decided to update the file ourselves. This is a
detailed breakdown of a problem, to give an idea of how to look at problems through the eye of a common command-line
tool in the U*nx world. In this case sed.

## Background

I got a request for help with sed on a date format problem. Here's something that worked. I'm going to break it
down, piece by piece.

```bash
sed -e "s/^\(.*\)\([0-9]\{4\}-[0-9]\{2\}-[0-9]\{2\}\) \([0-9]\{2\}:[0-9]\{2\}:[0-9]\{2\}[.][0-9]\{1,3\}\)\(.*\)$/\1\2T\3Z\4/" foo.txt
```

### sed - Stream EDitor

In this case, we are using sed to replace one date format with another in every line. sed applies a regular expression
to each line, and outputs the result of applying that regex, instead of the original line. In the case that the regex
does not match, sed outputs the original line unchanged. We capture the output to create a file with a date format that
another script knows how to handle.

Note: These examples take input and write to the terminal (stdout). So long as you have the original file available,
the "-i" option will update the file in-place.

#### Line-Oriented

Generally, unix tools work on lines from an input source, and produce lines to a destination, typically standard out.
Here is a simplification of sed (and the loop idea applies to most unix command line tools):

```bash
while there are more lines of input
    grab a line
    if the regular expression matches
        apply it to the input line and echo a different output line
    else
        echo the original input line
end
```

### Simple exmample
Here's a simple example input file:

### file: text.txt
```
hello
goodbye
```
And a sed command to perform a replacment:
```bash
sed -e "s/o/O/" text.txt
```
Which prints
```
hellO
gOodbye
```

We are replacing lower case o with capital O. Notice that only the first one was replaced? If we want to make a
substitution multiple times in the same line, we add g (for global to the end):
```bash
sed -e "s/o/O/g" text
```

The results chnage to:
```
hellO
gOOdbye
```

The original example above does not use the global option. It attempts to match the entire line, looks for one date/time
in the line and replaces a space between the date/time with a T and appends Z to the end of it.

### Values versus Shapes

In the trivial example above, we look for a specific value. The lower-case letter o. Somtimes you want to find something
specific like this (e.g., searching for all calls to a method named doNotUseThisMethod). However, this example is not
looking for *values*, so much as it is looking for *shapes*. We'll use a regular expression to describe the shape of the
thing we want to find. We need to do this becauase we use the value in the current line, which is unknown. We know we
want to do something with dates and times. We know for this actual problem, that a date is 4 digits, a dash, 2 digits, another dash, and 2 final
digits. That is, the shape is 9999-99-99. The particular value, however, is not known until we apply it to an input line. 
This example is not the exact problem, but it works with the same regex, so it is representative of the actual problem:
```
before 2020-08-21 22:15:13.03 after...
before 2020-09-22 22:15:13.3 after...
before 2020-10-23 22:15:13.983 after...
```
Here is what we want the new output to be:
```
before 2020-08-21T22:15:13.03Z after...
before 2020-09-22T22:15:13.3Z after...
before 2020-10-23T22:15:13.983Z after...
```

The following approach works so long as...
* the pattern you are after is easily separable from the rest of the input line
* the pattern occurs exactly the same number of times (this is the approach, it's possible to allow for varying number
  of recurrences, but since we don't have to, this approach can be much simpler)
* in this specific example, it occurs exactly once. The example file above was my model for getting the regex right (
  took 15 tries give or take, probably more). 
  

### The From Part of the Regular Expression
I broke the line into 4 groups. The four (so-called capture) groups are:
* capture #1, everything before the date: `\(.*\)`
* capture #2, the date: `\([0-9]\{4\}-[0-9]\{2\}-[0-9]\{2\}\)`
* a single space (not a capture group, but the space is a necessary part of the overall regex)
* capture #3, the time: `\([0-9]\{2\}:[0-9]\{2\}:[0-9]\{2\}[.][0-9]\{1,3\}\)`
* capture #4, everything after the time: `\(.*\)` 
  
Here are the actual values, when looking at the first line of the input file:
* Group 1: `\ .*\)`  
  before 
* Group 2: `\([0-9]\{4\}-[0-9]\{2\}-[0-9]\{2\}\)`  
  2020-10-09 
* Group 3: `\([0-9]\{2\}:[0-9]\{2\}:[0-9]\{2\}[.][0-9]\{1,3\}\)`  
  22:13:35.112 
* Group 4: `\(.*\)`  
  after... 
  
There's a lot to unpack, so here goes. 
* `^` and `$`   
  Denote, respectively, the beginning of the line and the end of the line. If a
  regex does not match, nothing happens, and sed echos the original line. Adding ^ and $ says that this entire regex
  must consume the whole line or not match. 
* `\(` and `\)`  
  The beginning and ending of a capture group in sed. In some languages you use ( and ) (no starting back-slash) and if you want a literal (, you use \(. For basic sed, it is \(,
  but even that can change if you instead use another standard (command line option). 
* `.  
  A wild card. Match any 1 character
* `*`  
A modifier, match the previous thing 0 or more times. (Note, + means 1 or more, and sometimes you escape it with \ and
sometimes you don't - but for a given tool it is always one way or the other, never both.)
  
* `\(.\)`   
Capture 0 or more of any character. 

* `^\(.*\)`   
  Same thing, but the first character must be at the beginning of the line, that is, it must be directly after the logical start of the line, or just after `^`.
  
* `[` and `]`   
  Define a group of characters. For example, `[0-9]` means any digit in the set: `0 1 2 3 4 5 6 7 8 9` In some regex you have
to escape this, in others, like sed, you do not. If you want a literal `[`, you escape it: `\[`
  
* `[0-9]*`   
  zero or more digits 
  
* `\{` and `\}`   
  Range modifier. Like * or plus, but specifies a number or a range. Some examples:
  * `a\{4\}`  
    aaaa 
  * a{1,3}   
    a or aa or aaa (1 to 3 a)
  * `.\{2\}`  
    Any 2 characters. That's a wildcard, `.` (any character), followed by a count, 2.
* `-`   
  * Outside of `[]`   
    A literal dash, like the dashes in 2021-10-20
  * Inside of `[]`  
    Range, e.g., [a-z] a through z, [A-Z], A through Z
* ` ` (there's a space just before that open paren)  
  Means a literal space.
   
Putting it all together for the timestamp:
```bash
\([0-9]\{2\}:[0-9]\{2\}:[0-9]\{2\}[.][0-9]\{1,3\}\)
```
* Start a capture group: `\(`
* 2 numeric digits: `[0-9]\{2\}`
* literal `:`
* 2 numeric digits
* literal `:`
* 2 numeric digits
* literal `.` (I visuall preer `[.]` rather than `\.` they are equivalent, however.
* 1 to 3 numeric digits: `[0-9]\{1,3\}` 
  
That is, a timestamp in the format of hours, minutes, seconds, and up to 3 digits for milliseconds. Or a time. 
The original regular expression had 4 capture groups. The time group, #3, is the longest of 
the groups. The other three groups are: 
* Everything up to but not including a date
* The date
* Everything after the milliseconds in the timestamp. 
  
### To Part
Now on to "to" or what to replace. As a quick reminder, here's the "to" or what we want the replaced line to look like:
```
/\1\2T\3Z\4/
```

Here's a detailed breakdown of what the output line looks like, when the original regular expression matches:
* `\1`  
  back reference. Whatever was matched by the first capture group, put it first:   
  before 
* `\2`  
  back reference to the date:  
  2021-10-08 
* `T`  
  A literal capital T 
* `\3`  
  back reference to the time in hours, minutes, seconds, milliseconds  
  12:23:12.32
* `Z`  
  literal Z 
* `\4`  
  the rest of the line after the end of the date:  
  &nbsp; after (Notice space is in the last group, not with the time)

We take:
```
before 2021-10-08 12:23:12.32 after
```
And end up with:
```
before 2021-10-08T12:23:12.32Z after
```

Note: if the regular expression does not match, sed simply echos the original line.

# Conclusion
This is a breakdown of a single sed command. In this case, we needed to change a date/time format. That
date/time was embedded in a larger line. So we get the part before the date/time, and after. since 
we are putting something both between the date/time and after the time, we keep the date and time as 
separate groups to make that easy.

