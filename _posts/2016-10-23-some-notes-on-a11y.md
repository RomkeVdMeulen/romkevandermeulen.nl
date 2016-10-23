---
title: "Some notes on Accessibility"
date: 2016-10-12
tags:
  - a11y
  - html
  - javascript
---

Here are some notes on the topic of
[accessibility](https://en.wikipedia.orgAccessibility) that I took
while researching the subject from the perspective of a developer.

When you first start looking into accessibility (or a11y to save on
keystrokes) you may think, as I did, that it is a very hard, vague
problem that you know you should probably deal with at some point, but
somehow there's always other stuff to do first. Like making the UI look
pretty and slick - that always seems to be so much more gratifying.

But as it turns out, a11y is not all that hard to think about, or to
take into account while you're developing. And if you do everything
right, it will not only make your app more pleasing to use for people
with disabilities, it will enhance the experience for everyone.

The very first thing I suggest you do is watch the
[Web Accessibility Perspectives videos on Youtube](https://www.youtube.com/playlist?list=PLhDEeYUfW02QndusXXtQtuMbMYhK7TMBT).
They were made by the W3C Web Accessibility Initiative and do a very
good job of conveying not just the challenges faced by people with
disabilities when navigating the web, but also how solving these
problems makes your webapp more pleasant to use even for people without
disabilities.

Also, reading the
[Stories of Web Users on the W3C WAI website](https://www.w3.org/WAI/intro/people-use-web/stories)
can be a good first step. These will further expand your understanding
of the kinds of disabilities people deal with, and how these affect how
people interact with webpages.

Now that you know your audience, all you need to know is what to do, and
what not to do, to make sure everyone can enjoy you app. It's really
quite simple when you think about it. There's two rules, one for input
and one for output:

* Make sure all input methods are fully supported. Users should be able
  to access every part of your application with just a keyboard without
  a mouse, with only touch and on-screen keyboards, etc..
* Make sure that all important information is conveyed as text. So no
  critical information in images without alt, no graphical structuring
  of your content unless that structure is also in the markup, etc..

Simple isn't it? And you thought a11y was too hard a topic...

I'll give a few more detailed notes on certain topics. After that, I
have a list of handy resources for you to use to do the good work of
making the entire web accessible.

## Keyboard navigation

Keyboard navigation is used by people who can't or won't use a mouse,
e.g. someone with a
[repetitive strain injury](https://en.wikipedia.org/wiki/Repetitive_strain_injury).
It's easy to forget to test your webapp with only keyboard navigation,
given how often we devs just click through the UI to test it.

One gotcha that I've fallen into a few times while developing an
interface with JavaScript: an `<a>` element without an `href` can't be
reached by pressing `Tab`. So either add `href="#"` or `tabindex="0"`
(tabindex higher than 0 breaks the element out of the natural tab order
and [is discouraged](http://webaim.org/techniques/keyboard/)).

> For best results: 
> Structure your underlying source code so that the reading/navigation
> order is correct. 
> Then, if necessary, use CSS to control the visual presentation of the
> elements on your page. 
> Do not use tabindex values of 1 or greater to change the default
> keyboard navigation order. 
>   -- [WebAIM: Keyboard Accessibility](http://webaim.org/techniques/keyboard/)

Further reading:

* [WebAIM: Keyboard Accessibility](http://webaim.org/techniques/keyboard/)
* [WebAIM: Links and Hypertext](http://webaim.org/techniques/hypertext/)
* [MDN: Keyboard-navigable JavaScript widgets](https://developer.mozilla.org/en-US/docs/Web/Accessibility/Keyboard-navigable_JavaScript_widgets)

## Semantic tags

HTML5 introduced a number of new 'semantic' tags like `<section>` and
`<main>` that carry more information about document structure than your
standard `<div>` or `<span>`. Assistive technology, like screenreaders,
can use this information to ease navigation. Plus, it's good for your
SEO :wink:

Not all assistive technology works consistently with all these new
elements, or even supports them yet. The Paciello Group publishes
[Aural UI of the Elements of HTML](https://thepaciellogroup.github.io/AT-browser-tests/)
which tells you what popular screenreader products do with every HTML tag
(though it's a work in progress).

Further reading:

 * [WebAIM: Semantic Structure](http://webaim.org/techniques/semanticstructure/)
 * [MDN: list of all HTML tags](https://developer.mozilla.org/en-US/docs/Web/HTML/Element)

## ARIA attributes

You may have heard of ARIA attributes, or seen them in example HTML code
on the web. The [ARIA spec](https://www.w3.org/TR/wai-aria/) was
published in 2014. ARIA attributes can help you to explain what kind of
UI-element a given HTML fulfils, if it's not obvious. So if you use a
`<button>` (you should) you're done, but if you insist on using a
`<div>` instead, you need to add a lot of additional information like
`role="button" aria-pressed="false"` to make sure everyone understands
what you're trying to do here (also you need to make sure your
JavaScript is listening for keyboard events, the element has a
`tabindex` - honestly just use a `<button>`).

There's some overlap between ARIA and the new elements presented by HTML5, but
[not everything ARIA does can be accomplished with HTML5](https://www.paciellogroup.com/blog/2010/04/html5-and-the-myth-of-wai-aria-redundance/).

Further reading:

 * [W3C: WAI ARIA Authoring Practices](https://www.w3.org/TR/wai-aria-practices/)
 * [W3C: The Roles Model](https://www.w3.org/TR/wai-aria/roles)
 * [WebAIM: ARIA](http://webaim.org/techniques/aria/)
 * [Heydon works: Practical ARIA examples](http://heydonworks.com/practical_aria_examples/)
 * [Terrill Thompson: ARIA Live Region Test](http://terrillthompson.com/tests/aria/live-scores.html)

## Text

Make sure information in images, videos and other non-text media has a
text-equivalent somewhere in your document. Usually this means writing
an `alt` attribute, or `aria-label`. Note however, that although the
`alt` attribute is required for images, it is sometimes better to leave
them empty. You should definitely read
[WebAIM's article on alt-text](http://webaim.org/techniques/alttext/) -
it's an eye-opener.

Further reading:

 * [WebAIM: Alternative Text](http://webaim.org/techniques/alttext/)
 * [WebAIM: Invisible content](http://webaim.org/techniques/css/invisiblecontent/)

## Color

Color is usually something the designer should concern themselves with.
In case that is you however, or you want to teach your designer about
a11y, here are some notes:

Don't convey important information only through color - not everybody
will be able to perceive it. Use some text, image or icon to convey the
same information.

Also check how your design looks to people with different forms of
color-blindness (that's right, there's more than one kind). You can
run screenshots or designs through
[Vischeck](http://www.vischeck.com/downloads/), but most modern image
editors like photoshop will also have an option somewhere to view the
image in the editor like a color-blind person would see it. Check the
`View` menu.

Also make sure you're always using plenty of contrast between text and
background, to maximize readability. Plenty of tools can check whether
you're using enough contrast, like e.g. the
[WebAIM contrastchecker](http://webaim.org/resources/contrastchecker/).

## Further Resources

 * [WebAIM Articles](http://webaim.org/articles/)
 * [The Paciello Group blog](https://www.paciellogroup.com/blog/archive/)
   * [Paciello Group: HTML5 Accessibility Chops: the placeholder attribute](https://www.paciellogroup.com/blog/2011/02/html5-accessibility-chops-the-placeholder-attribute/)
 * [Google: Introduction to Web Accessibility course](https://webaccessibility.withgoogle.com/course)
 * [W3C: Tips on Developing for Web Accessibility](https://www.w3.org/WAI/gettingstarted/tips/developing.html)
 * [WCAG 2.0 Quick Ref](https://www.w3.org/WAI/WCAG20/quickref/) - [principles](https://www.w3.org/WAI/intro/people-use-web/principles) - [At a glance](https://www.w3.org/WAI/WCAG20/glance/)

## Tools
 * [Accessibility Developer Tools: chrome plugin by Google](https://chrome.google.com/webstore/detail/accessibility-developer-t/fpkknkljclfencbdbgkenhalefipecmb)
 * [ChromeVox: a screenreader plugin for Chrome](https://chrome.google.com/webstore/detail/chromevox/kgejglhpjiefppelpmljglcjbhoiplfn)
 * [The A11Y Project: Web Accessibility Checklist](http://a11yproject.com/checklist.html)
