---
title: "Simple poller library in TypeScript"
date: 2017-04-08
tags:
  - typescript
  - javascript
---

Here is a very simple library of pollers written in TypeScript. It uses
async / await so if you want to transpile it to ES3 or ES5 you'll need
TypeScript 2.1 or higher.

I originally built this for [Aurelia](http://aurelia.io) but I pulled out
what little framework code there was to make it compatible with any
frontend framework. It assumes you have a separate class to manage
requests which exposes a `get` method to fetch with `GET` and which
returns a promise for a value. In this example that class is called
`BackendService` but you can change that to suit your needs.

I built it on an abstract base so you can have implementations of various
poller strategies. I implemented two: a basic poller that calls a callback
with each result and stops if that callback returns `false`, and a list
poller intended to incrementally gather a list of resources by some
identifier which stops when all requested resources have been received.
You should be able to easily add your own strategy. There's a top-level
service containing factories for each strategy, which you can use with any
DI mechanism you might be using.

If you have any questions about this code, or suggestions, I'd love to
hear from you.

{% gist 92d2127827098499762eceaed63eeb3a %}
