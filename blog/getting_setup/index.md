---
Title: Getting my blog Setup
---
I've again renewed my effort to get started blogging. The subject of C++ came up a few times and I decided to 
see how much effort it would be to get some simple examples working. 

I took a look at [CppUTest](http://cpputest.github.io/), which is the tool I happen to use and liked the look of 
their github-hosted site, so I figured I'd git it a try. I'm successful for some definition of success, but I 
think I'm lacking some expected knowledge, because it took a bit of yak shaving to get to where I am.

I'm going to capture some notes so if you want to also get started, you might learn from what I have done. Or, if 
you are already there, you could tell me what I falied to understand or read or google that led me to what I'm doing.

I have both a mac and a Windows PC. I prefer Unix and I typically work in a VM anyway, so the host environment 
usually isn't a issue. In this case, I watned to get going quickly, so I fitured I'd try working in my Windows
host environment natively.

After messing around a bit in [git bash](https://git-scm.com/downloads), I switched to the Ubuntu subsystem in Windows 10. [I upgraded ruby to 2.5 using rbenv](https://gorails.com/setup/windows/10).

Next, I followed the steps for setting up [GitHub pages](https://guides.github.com/features/pages/). By using the repo schuchert.github.io, it became my default site. I had something else there before, but that was a 6 year old experiment I deleted.

Next, I wanted to figure out how to serve up my files locally. 

I had originally tried following the instructions [here](https://help.github.com/articles/setting-up-your-github-pages-site-locally-with-jekyll/). But I wasn't smart enough to udnerstand those steps. So I made a few minor changes.

1. I did follow [Step 1](https://help.github.com/articles/setting-up-your-github-pages-site-locally-with-jekyll/#step-1-create-a-local-repository-for-your-jekyll-site).
1. Rather than following [step 2](https://help.github.com/articles/setting-up-your-github-pages-site-locally-with-jekyll/#step-2-install-jekyll-using-bundler) I cloned [my github pages repo](https://github.com/schuchert/schuchert.github.io).
1. I created a sibling directory following using the normal jekyll approach:
~~~
jekyll new jekyl-site-schuchert.github.io
~~~
1. In the directory created in the previous example, I attempted to run jekyll locally:
~~~
bundle exec jekyll serve --source ../schuchert.github.io/
~~~
1. For this to work I needed to add three (if I remember them all) values in _config.yml of schuchert.github.io
~~~
title: Brett Schuchert's Collection Of Old And Outdated Ideas
description: >-
  Yet another attempt at a blog as well as porting of an old site
repository: schuchert/schuchert.github.io
~~~

Once I did that, I was able to preview my work at [localhost:4000](http://localhost:4000).

