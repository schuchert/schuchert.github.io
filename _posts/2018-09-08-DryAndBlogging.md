---
title: DRY In Blogging
layout: post
description: >
  Making something more useful for subsequent uses could have lead to duplication, but I'm working on reducing that.
tagline: More usability, reducing duplication
---
## Background

I often write tutorials to cover what I learned because while I'll probably
forget the details, if I've written it, I'll remember that I've written
something on the subject, and I'll google for it.

<aside>
### Years Moved
As I write this, I'm still moving years of work from wikispaces.com to
github. Wikispaces closing down was the 
[foreign element](https://schuchert.github.io/wikispaces/pages/SatirChangeModel.html)
that got me to get back in to blogging.
</aside>

## Realization

On a recent coaching gig I was reminded that it's common for people working
in Java to not know how to create a project from scratch. I think this
is an important skill because being stopped or slowed down from 
trying to practice something like a 
[kata](https://schuchert.github.io/wikispaces/pages/SatirChangeModel.html)
could be just enough to stop someone from giving it a try.

I wrote [yet another example](https://schuchert.github.io/wikispaces/pages/java/project.from.scratch/using.gradle.html)
of how to do that using an approach that is somewhat close to the 
environment the group works in.

As I wrote this, I thought I'd like to be able to just grab the commands
and run them. Here's my thinking:
1. I remember I've written something
1. I can find it using google
1. What I find is written with details I don't need.
1. So it's a bit more work to use it later

I want a summary of the steps both for myself and for someone who 
basically gets it but might not remember the details.

If you have a look a [that example](https://schuchert.github.io/wikispaces/pages/java/project.from.scratch/using.gradle.html)
I think you'll see that I've got a good first cut. I can copy the summary
at the top, or I can go further into the details.

I don't like duplication. So I wondered how could I avoid duplication, 
or at least reduce it, as I update the details of that example.

Here were my givens:
1. I'm using Jekyll for github pages
1. I don't want to have to have the same material twice if I can avoid it
1. It would be nice to have something I could use in other places
1. I prefer using pure css

What follows is a collection of some of the things I did to get close.

### Extract File

First, I realized that I wanted similar or the same content with two different
views: a summary, and details. So I extracted that into its own file, which
I happened to call `using.gradle.steps.md`.

Next, I included that file twice, wrapping it with a div with a style:
``` ruby
<div class="show_bash">
{% raw %}
{% capture summary %}
    {% include_relative {{ using.gradle.steps.md }} %}
{% endcapture %}
{{ summary | markdownify }}
{% endraw %}
</div>
```

Now this was itself duplicated, notice the only difference is the class in 
the```div```.
``` ruby
<div class="show_terminal">
{% raw %}
{% capture summary %}
    {% include_relative {{ using.gradle.steps.md }} %}
{% endcapture %}
{{ summary | markdownify }}
{% endraw %}
</div>
```

### Pure CSS It Up
While I could have used jquery or raw Javascript, I wanted to stick to
pure css for the formatting. 

So I created css ([less](https://en.wikipedia.org/wiki/Less_(stylesheet_language) example shown) to get the effect I wanted (this is the relevant styling):

``` scss
div {
    &.show_terminal {
        div.language-bash {
            display: none;
        }
    }

    &.show_bash {
        p, ul, h1, h3, div.language-terminal  {
            display: none;
        }
    }
}
```

Later, I switched from using a```<div>``` to```<section>``` as I wanted
to represent the semantic breakdown of the page.

### More Duplication

While I was happy to have reduced much of the duplication, I wasn't happy
with the amount of fluff I needed to get the thing I included.

Noticing that there was a lot of duplication in the```div``` (later
```section```) definition, I extracted the common stuff and passed in
a parameter:
#### File Extracted: included_md_file
``` ruby
{% raw %}
{% capture summary %}
  {% include_relative {{ include.filename }} %}
{% endcapture %}
{{ summary | markdownify }}
{% endraw %}

```
#### Using it
``` ruby
{% raw %}
<section class="show_bash">
## Summary of Steps
{% include include_md_file filename="using.gradle.steps.md" %}
{% endraw %}
```

Notice, this does not introduce the```section``` or the css class. After 
some thinking, I wanted that include to stick to one thing. I also didn't
want the inclusion of content to be coupled to its semantic context, therefore 
* the first line introduces the semantic context
* Next, I give it a level 2 header for where it resides it the context of its use
* Finally, use include the file while passing in the filename.

### Lie Reveled

While I'm happy with the results, my desire to remove duplication between
the summary and details section is not yet perfect. Instead of having
the content typed twice, in differnt locations, I do still have it twice
but in the same file, and close together.

Here's an example:
``` markdown
{% raw %}
~~~ bash
gradle init --type java-application
~~~

~~~ terminal
vagrant@vagrant-ubuntu16:~/src/smoke$ gradle init --type java-application

BUILD SUCCESSFUL in 0s
2 actionable tasks: 1 executed, 1 up-to-date
~~~
{% endraw %}
```

The command that I display in the summary of a page is formatted using
the bash style. The command with a prompt and its output is formatted
with the terminal style. So I still have the duplication, I think 
this is better than what I would have otherwise done.

As I'm still new to using Jeykll, I'm getting the hang of using it.
If I could format the following line:
``` terminal
vagrant@vagrant-ubuntu16:~/src/smoke$ gradle init --type java-application
```

Something like this:
```
<span class="terminal">vagrant@vagrant-ubuntu16:~/src/smoke$</span><span class="bash"?gradle init --type java-application</span>
```

I would do it. But I have not yet figured out that magic. If I did all of the
formatting by hand I probably could do that. However, I'm using the code
formatting built into Jekyll.

Assuming I figure out how to get both, I won't duplicate that. Instead, 
I'll have some macro like:
```
{% raw %}
{$ include command_line 
  prompt="vagrant@vagrant-ubuntu16:~/src/smoke$"
  command="gradle init --type java-application" %}
{% endraw %}
```

But until then, I'm happy with what I've managed so far.

## The Rest of the Story

I could some of the previous steps, but as this is as much a series 
of notes to myself as a description of how I got to where I got, 
I'm going to keep it.

I'm using [Rogue](http://rouge.jneen.net/) with Jekyll. The following
example will show why some of my efforts were unnecessary.

### Example Markup
Notice that gone are the two copies of the command line. There's just one
version.  I keep the one marked `termianl`, however `bash` would have
worked as well.
``` markdown
{% raw %}
~~~ terminal
vagrant@vagran-ubuntu16:~/src$javac -version
javac 1.8.0_181
~~~
{% endraw %}
```

### Resulting HTML
Rogue parsing is smart enough to identify prompts versus commands and uses
a ```span``` to break up the one line. This is what I wanted to do and here
it is already done.
``` html
{% raw %}
<span class="gp">vagrant@vagran-ubuntu16:~/src$</span>javac <span class="nt">-version</span>
<span class="go">javac 1.8.0_181</span>
{% endraw %}
```

### How Each Are Rendered

After I looked at the rendered html in detail, I updated my CSS to take
advantage of this.

<section class="summary">
#### As Summary
~~~ terminal
vagrant@vagran-ubuntu16:~/src$javac -version
javac 1.8.0_181
~~~
</section>

#### As Details
<section class="details">
~~~ terminal
vagrant@vagran-ubuntu16:~/src$javac -version
javac 1.8.0_181
~~~
</section>

Now that I had what I wanted:
* Works out of the box in Jekyll
* Removed duplication of the command line summary and the detailed output
* Pure CSS solution

I then made a number of changes.

### Increase Semantic Level
First, the class names were about implementation rather than intent.
I updated both sections:

* The class ```show_bash``` became ```summary```
^
``` markdown
{% raw %}
<section class="summary">
## Summary of Steps
{% include include_md_file filename="using.gradle.steps.md" %}
{% endraw %}
```

* The class ```show_terminal``` became ```details```
^
``` markdown
{% raw %}
<section class="details">
## Step by Step Instructions with Example Output
{% include include_md_file filename="using.gradle.steps.md" %}
{% endraw %}
```

### Update the CSS Accordingly
``` scss
div.content {
  section {
    &.summary {
      p, h3 {
        display: none;
      }
      aside {
        p, h3 {
          display: block;
        }
      }
      code {
        span {
          &.gp, &.go {
            display: none;
          }
        }
      }
    }
  }
}
```

In fact, the ```details``` class has no special requirements regarding
hiding anything, but I still have it identified.

## Summary
I wanted to have a summary of detailed steps using pure css, Jekyll, and
avoding duplication.

Using the tools I had and a little bit of refactoring of the structure of
the files, I was able to do it and in retrospect, it was intuitive...
once I understood it.
