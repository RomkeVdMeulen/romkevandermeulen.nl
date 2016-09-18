---
title: "Deferred - A TypeScript Promise wrapper"
date: 2016-09-18
tags:
  - shell
---

I love [Promises](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Promise),
and I love that they're becoming a core Javascript feature. However, the
standard implementation doesn't let you query the
[state or fate](https://github.com/domenic/promises-unwrapping/blob/master/docs/states-and-fates.md)
of a promise, which in some cases may be necessary.

While working on the [Vevida Aurelia datagrid](https://github.com/Vevida/datagrid-demo),
I found I needed to know whether a given promise had been fulfilled yet. I wrote
this TypeScript wrapper class, and named it 'Deferred' after Angular's
[Deferred API](https://docs.angularjs.org/api/ng/service/$q#the-deferred-api),
which they in turn based on [the Q library](http://documentup.com/kriskowal/q/).

{% gist 93588812a964f842a73631a22cd9b9ad %}
