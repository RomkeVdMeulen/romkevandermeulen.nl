---
title: "@autocast TypeScript property decorator"
date: 2018-01-31
tags:
  - ts
excerpt:
  This TypeScript property decorator makes sure that values assigned to that
  property are converted to the right type, provided you can do so in a
  predictable manner.
---

Last week I described [a way to change instance-specific properties using TypeScript property decorators]({{ site.baseurl }}/2018/01/24/typescript-property-decorators.html).
Here is a simple application of this: a decorator that you can apply to a
property to automatically cast values assigned to that property to the type you
want it to have (provided this can be done in a predictable fashion of course).

My use case for this was that I'd designed a bunch of related model classes. E.g.:

```ts
class Organization {
    companyName: string;
}
class Person {
    name: string;
    organization: Organization;
}
```

Now I wanted an easy way to convert a plain, nested object that I'd gotten from
a server response to these classes. Something like this:

```ts
function getPerson() {
    const json = fetchPersonFromServer();
    return Object.assign(new Person, json);
}
```

The problem here was that nested objects weren't converted, so the returned `Person`
instance had an `organization` property with a value that was not an instance of
`Organization`. Rather that convert all these things by hand, I wanted to be
able to do this:

```ts
class Person {
    @autocast(Organisation)
    organization: Organization;
}
```

(Of course, you could use TypeScript's `emitDecoratorMetadata` option and have
the decorator extract the required type, rather than passing it as a parameter,
but I'll leave that as an exercise to the reader.)

So here's my implementation of the `@autocast` decorator. Note that it makes
some assumption, such as that the type to cast to has a constructor that can be
called with no arguments, or that `undefined` should simply be passed through.
You may want to tune the implementation for your own use-cases, but the
framework can stay the same. I've also thrown in a decorator to apply to
properties that hold arrays of instances of some class: `@autocastArray`.

{%gist ee3397452be61f6b3e23a3ab506b7c0a %}
