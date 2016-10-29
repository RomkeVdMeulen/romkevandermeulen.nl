---
title: "@keyListener: a TypeScript keyboard event filter"
date: 2016-10-12
tags:
  - typescript
  - javascript
excerpt:
  A decorator for KeyboardEvent listeners which filters out events from keys
  you're not interested in.
---

Alas, detecting which keyboard key was pressed through JavaScript is
[a bit of mess](http://www.quirksmode.org/js/keys.html). There's
[KeyboardEvent.key](https://developer.mozilla.org/en-US/docs/Web/API/KeyboardEvent/key),
which is supposed to clear all this up, but that's not going to do us a lot of
good now because
[it's not widely supported yet](http://caniuse.com/#feat=keyboardevent-key). So
until support improves (and even after that) it might be a good idea to tuck
away the filtering of keyboard events from keys we're not interested in and put
it somewhere where we don't have to worry about it. Like a pretty decorator for
methods on our classes. Here's my take.

{% gist e9468f694e5fe4d05430ee0cc51505f8 %}

I used `keyCode` which although deprecated is at least widely supported. For my
limited use case (space and enter keys) the behaviour seems to be consistent
across browsers. If you need to allow for more edge cases you can of course
extend this example further.
