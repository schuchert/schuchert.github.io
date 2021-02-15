---
layout: post
title: "Non-Printable Characters and How We Got Them"
description: >
  We do a bit of scripting. We often get input files that have non-printable characters. Sometimes we make them 
  ourselves. How can you identify when that's an issue and how can you fix it?
category: [Daily Tech]
tags: [unix, sed, regex, non-printable]
draft: true
---

warning this is raw text, mostly unfomatted. That's why it's draft.
## Overview
This follows [Sed Breakdown]({% post_url 2021-02-14-SedBreakdown %})
 example... We have a "text-only" csv file. BUT it's not text only.... And when this happens, it's it entirely non-obvious...
Quick suggestion. When creating files that are meant to be text, don't use fancy tools (anything really). Because you'll likely get unprintable characters. And that will cause scripts to break in obscure ways. To see a quick fix, look at the thread...




6 replies

Brett Schuchert  34 minutes ago
Here is an example file:
Example.csv
,,,,,,
,AA03-123-12,F,2021-02-03T12:13:14.567z,,,




Brett Schuchert  32 minutes ago
Notice I just created a new spreadsheet in Numbers and then exported as CSV. It added several extra commas and such, but that isn't the issue we're going to solve... Once you save this file, try the following:
wc Example.csv
2       2      52 /Users/bschuch2/Documents/Example.csv




Brett Schuchert  29 minutes ago
That's 2 lines, 2 words, 52 characters. We only care about the 52...
Now, we'll use sed to remove all non-printable characters directly in-place (updates file, does not echo updates lines to standard out):
sed -i 's/[^[:print:]]//' Example.csv
And has the file changed?
wc Example.csv
2       2      50 Example.csv

Brett Schuchert  29 minutes ago
But if you look at the file (attached after change):
Example.csv
,,,,,,
,AA03-123-12,F,2021-02-03T12:13:14.567z,,,




Brett Schuchert  28 minutes ago
The file looks the same. What were those 2 characters?
I think ASCII 0 (null) inserted at the beginning of the file to indicate it's a rich text file rather than an ASCII-based text file.
We don't want that, so this fixes that.

Brett Schuchert  20 minutes ago
Here's a breakdown of the sed command:
sed -i 's/[^[:print:]]//' Example.csv
-i in-place. Update the file directly rather than output a new file. OK if you have the original. The following (probably safer) option also works:
sed 's/[^[:print:]]//' < Example.csv # this will print to the terminal
sed 's/[^[:print:]]//' < Example.csv > Example.csv.txt # use .txt to indicate intent only
s -> search
first / -> beginning of what to search for
[ -> start of group of characters
^ -> NOT beginning of line, in [] it INVERTS the search (this means find a line that has non-printable characters - )
[:print:] -> how you use a posix name to name a group of characters -> printable characters, letters, numbers, space, tab.
] -> end of group
/ -> what to replace it with
/ -> end of search, so replace printable with non-printable...
As written, this will find any line that has at least one non-printable character and replace it.
If the line has only printable characters, the regex doesn't match, and the original line remains unchanged.
If the line has a non-printable character, it is removed. But only the first one. That is fine, because the problem characters are at the beginning of the file, and apparently one per line. If you want to be more aggressive, add "g" after the final /. It will replace each non-printable character in the line. In this case, the results are the same.
