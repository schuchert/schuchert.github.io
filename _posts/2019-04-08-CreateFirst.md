---
title: Create First
layout: post
draft: true
description: >
  How can we clean up code in a legacy code base when continuous integration 
  is not happening? Maintain backwards compatibility by creating first.
tagline: Making the impossible, possible
---
{::options parse_block_html="true" /}

## Summary

Working in a legacy code base where multiple teams are making updates, and 
merging to master is infrequent, presents challenges. It turns out that
we can use similar thinking to that presented in 
[Refactoring Databases](https://www.amazon.com/Refactoring-Databases-Evolutionary-paperback-Addison-Wesley/dp/0321774515). 


In [Refactoring Databases](https://www.amazon.com/Refactoring-Databases-Evolutionary-paperback-Addison-Wesley/dp/0321774515), 
the recipes come in two varieties:
* Solutions where one client directly interacts with the database
* Solutions where multiple clients, on their own release cadence, interact with the same underlying database
