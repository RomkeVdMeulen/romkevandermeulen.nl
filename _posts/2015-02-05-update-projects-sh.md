---
title: "Update your projects: up.sh"
date: 2015-02-05
tags:
  - bash
  - shell
  - svn
  - git
  - development
---
I have a number of frameworks, platforms and reference projects in my workspace
that I want to update daily. Rather than going to each of them and doing `svn up`
or `git pull` by hand, I've written a little Bash script to do it for me. I just
keep a list of all projects I want updated. Now I need only run `./up.sh` once a
day.

{% gist 5978f733f6d48542fa62 %}
