---
layout: post
title: "Non-Printable Characters and How We Got Them"
description: >
  We do a bit of scripting. We often get input files that have non-printable characters. Sometimes we make them 
  ourselves. How can you identify when that's an issue and how can you fix it?
category: [Daily Tech]
tags: [unix, sed, regex, non-printable]
---
{::options parse_block_html="true" /}

## Overview
This follows [Sed Breakdown]({% post_url 2021-02-14-SedBreakdown %}). We start with a "text-only" csv file. But,
spoiler alert, but it's not text-only. When this happens, it's non-obvious. 

{% include aside/start id="smart-files" title="Smart Editors Considered Dangerous" %}
When creating files that are meant to be text, don't use fancy tools (anything really). If you do, you'll
likely get unprintable characters. That will cause scripts to break in obscure ways. This is a quick note on one way
to fix that.
{% include aside/end %}

Here is an example file:
Example.csv
```
,,,,,,
,AA03-123-12,F,2021-02-03T12:13:14.567z,,,
```

Notice I just created a new spreadsheet in Numbers and then exported as CSV. It added several extra commas and such, but
that isn't the issue we're going to solve... Once you save this file, try the following:
```
wc Example.csv 
  2   2   52 /Users/bschuch2/Documents/Example.csv
```

That's 2 lines, 2 words, 52 characters. We only care about the 52...

Now, we'll use sed to remove all non-printable characters directly in-place (updates file, does not echo updates lines 
to standard out):
```
sed -i 's/[^[:print:]]//' Example.csv
```
Has the file changed?
Example.csv
```
,,,,,,
,AA03-123-12,F,2021-02-03T12:13:14.567z,,,
```

While the file looks the same, what were those 2 characters? I think ASCII 0 (null) inserted at the beginning of the 
file to indicate it's a rich text file rather than an ASCII-based text file. 

Here's a breakdown of the sed command:

| Token | Description |
| -i | In-place. Update the file directly rather than output a new file. OK if you have the original. |
| s | Search |
| first /| Beginning of what to search for |
| [ | Start of group of characters |
| ^ | Not beginning of line, in [] it inverts the search (this means find a line that has non-printable characters) |
| [:print:] | Refer to the posix name for printable characters, e.g., letters, numbers, space, tab. |
| ] | End of group |
| / | What to replace it with, or the start of the "to" section |
| / | end of search, so replace non-printable characters with nothing. |


{% include aside/start id="vs-inline" title="Options to `-i`" %}
If you want to preserve the original file, you have options. The following (probably safer) option also works:
```
sed 's/[^[:print:]]//' < Example.csv  
```
This will print to the terminal.

Or you can redirect the output:

```
sed 's/[^[:print:]]//' < Example.csv > Example.csv.txt  
```

This also print to the terminal, but then we redirect the output to a file. I used .txt to indicate intent only. Unix
doesn't care. 
{% include aside/end %}
  
As written, this will find any line that has at least one non-printable character and replace the first one.
If the line has only printable characters, the regex doesn't match, and the original line remains unchanged.
If the line has a non-printable character, it is removed, but only the first one. That is fine, because the problem 
characters are at the beginning of the file, and apparently one per line. If you want to be more aggressive, add "g" after the final /. It will replace each non-printable character in the line. In this case, the results are the same.

## Conclusion

Don't trust "text" files from outside. Verify they don't have unwanted characters. Here's a quick way to check using
the commands from above:
```bash
wc -c < Example.csv && sed 's/[^[:print:]]//' < Example.csv | wc -c
52
50
```
While the formatting is different, you can see if anything would change by looking at the last value.

Using a bit more bash, you can compare the results numerically:
```bash
if [[ $(wc -c < e) -ne $(sed 's/[^[:print:]]//' < e | wc -c) ]]; then echo "different";fi
```
