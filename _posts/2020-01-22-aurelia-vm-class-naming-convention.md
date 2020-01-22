---
title: "Changing the naming convention for Aurelia View Model class names"
date: 2020-01-22
tags:
  - ts
  - aurelia
---

I was working on a large Aurelia project and having trouble with clashing names
of JS classes. The convention for Aurelia View Models is that their class names
end in `Component`. But to prevent clashes, I wanted to ensure that View Models
attached to routes ended in `Route` (e.g. an `EmailRoute` might contain a
`EmailComponent`). I didn't find a built-in way to instruct the Aurelia loader
to interpret class names this way, but it turned out to be pretty easy to make
an override:

{% gist 1fdccff50ac11c47736374d7567b719d %}
