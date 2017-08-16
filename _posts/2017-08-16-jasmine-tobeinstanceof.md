---
title: "Jasmine custom matcher: toBeInstanceOf"
date: 2017-08-16
tags:
  - js
  - ts
  - jasmine
excerpt:
  Recently I found myself in need of a custom matcher for my Jasmine-based unit
  tests. I looked around and only found a few some outdated examples, so I wrote
  my own. Here it is so you don't have to do the same.
---

Recently I found myself in need of a custom matcher for my Jasmine-based unit
tests. I looked around and only found a few some outdated examples, so I wrote
my own. Here it is so you don't have to do the same. It's in TypeScript: if you
prefer plain JS you can simply take the types out and ignore the bottom
declaration.

{% gist a670e4cbb649edf36caa18e8859c2412 %}
