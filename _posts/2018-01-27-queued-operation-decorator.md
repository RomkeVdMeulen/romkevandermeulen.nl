---
title: "@queuedOperation TypeScript method decorator"
date: 2018-01-27
tags:
  - ts
excerpt:
  Here is a TypeScript method decorator used to queue calls to a number of async
  methods that work on shared state and need to be called one at a time to
  prevent race-conditions.
---

Here is a use-case: you have some class that offers a number of async methods.
Each of those methods works on some shared state, so you want prevent clients
from calling one of the other methods before the current operation is complete.
Rather than trusting your clients to be disciplined enough to await every
promise returned by your methods, you can queue the methods calls yourself. This
TypeScript method decorator even does it for you.

Of course, by the time a queued method call is actually executed, the
pre-conditions under which it was called may have changed, but that's your
caller's problem for starting new operations without waiting for old ones to
complete first.

{%gist f774324202d3fb8b710e7e2b1dcfdeb0 %}
