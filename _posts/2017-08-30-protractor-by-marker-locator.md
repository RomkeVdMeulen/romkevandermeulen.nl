---
title: "Protractor custom locator: by.marker"
date: 2017-08-30
tags:
  - js
  - ts
  - protractor
---

Protractor makes it very easy to locate a specific element in your webapp for
testing by writing CSS selectors. However, by using this overmuch, you're
locking your integration tests to the current structure of your app, making the
tests fragile. Here's a more robust alternative.

I've written an extensive Protractor suite for one of my projects and ran into
this problem. At first I sought to solve this by using only classnames with a
given prefix to locate elements in Protractor (`e2e-something` in this case).
Later on I started using an attribute rather than a class, which seemed neater
to me since this information has nothing to do with CSS or scripting:
`<button e2e="my-submit-button">`. When I settled on this convention, I also
wrote a custom Protractor matcher so that I can locate elements in my Protractor
tests simply as `element(by.marker("my-submit-button"))`. Here it is, written in
TypeScript (to port to JS just remove the typing information). I've also added
a TS declaration for the use of this locator so you don't get an error when you
write `by.marker()`.

{% gist 1451cbb3866d4eaf501115cc2f9e6065 %}
