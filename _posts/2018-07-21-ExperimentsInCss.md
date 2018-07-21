---
title: Experiments in Css, making Asides collapsible 
draft: true
layout: post
description: >
  A live blog that is meant to demonstrated something I want to have.
tagline: Allowing for all the things while also appearing to make things shorter
---
{::options parse_block_html="true" /}

# What is this
I put too many things in when I write. I like sidebars. Right now I'm keeping those in, eventually I 
might convince myself that I don't need them.

I'm hoping to follow some examples I found about pure CSS collapsible sections and make them work
in Jekyll. Once I've done that, I want to make them easy to use in my blog.

{% include aside/start id="aside-1" title="Section 1" %}
A list of things might be split into:
* things to like
* things to not like
* anything that is not in the above list
{% include aside/end %}

{% include aside/start id="aside-2" title="Section 2" %}
A list of things might be split into:
* things to like
* things to not like
* anything that is not in the above list
{% include aside/end %}

{% include aside/start id='t3' title='The Section In Question' %}
When this blog is out of draft, this section will be here but collapsed by default. 
{% include aside/end %}

As the aside above says, when this blog is out of draft, the asside above will be expandable but collapsed
by default.
