---
title: "Instance-targeted TypeScript property decorators"
date: 2018-01-24
tags:
  - ts
excerpt:
  TypeScript property decorators are applied when a class is loaded, not when a
  new instance of the class is created. Here is a technique you can use to alter
  properties using decorators so that they still appear like unique, own
  properties of every instance of your class.
---

TypeScript has [property decorators](http://www.typescriptlang.org/docs/handbook/decorators.html#property-decorators)
that you can apply to your class's data properties like so:

```ts
class MyClass {
  @autocastValuesTo(MyPropType)
  private myProp: MyPropType;
}
```

However, these decorators do not work exactly as you might expect. To quote the
TypeScript handbook:

> ... there is currently no mechanism to describe an instance property when
> defining members of a prototype, and no way to observe or modify the
> initializer for a property.

What this means is that your decorator will be applied the moment the class is
loaded, and your decorator will only have access to the class itself and the
name of the property. By the time any instance of your class is created, you'll
have no connection to that instance from the decorator.

So what do you do if you want to change the behaviour of a property so that
every instance of the class is affected? I spent a few hours one afternoon
hacking at this problem.

The first solution I came up with was to alter the class prototype, so that each
instance inherits the altered behaviour. This will work fine for some cases. The
problem is that, since the property is now defined on the prototype and
inherited, it is no longer an own property of any instance. That means it won't
be iterated over in `for ... of` loops or listed with e.g. `Object.keys()` or
`Object.values()`. For private properties this is not a big problem, but if
you're decorating a public property of a data representation, then it is a big
drawback.

Here is a more involved solution that will ensure the altered behaviour is
installed as an own property of the instance itself: put a property on the
prototype with a getter and setter that will install a property of the same name
but with the right behaviour on the called instance automatically the first time
it is used. In other words: the prototype property overwrites itself during the
first call. 

Here is a gist that shows how that would work. The assumption here is that you
want to apply some kind of mapping to any value assigned to the decorated
property. It installs a setter on instances that maps values and stores them in
a map keyed to the called instance, and a getter that reads from that map. No
getter is defined for the prototype, so the instance property remains unknown
if it is read before it is initialized.

{%gist e45ee89ce848e7fda140635a4d29892b %}

Do note that since each object that this decorator is used on is added to the Map,
these objects won't go out of memory and won't be garbage collected. Thanks to
Daniel Schiavini for pointing this out in the comments.

P.S.: Ironically I came upon this [answer on StackOverflow](https://stackoverflow.com/questions/41397947/typescript-class-decorator-that-modifies-object-instance/41403157#41403157)
only a short while after I spent hours on this problem. Seems I wasn't the first
person to come up with this solution.
