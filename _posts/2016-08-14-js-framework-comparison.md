---
title: "Angular vs React vs Aurelia"
date: 2016-08-14
tags:
  - javascript
  - angular
  - react
  - aurelia
  - frontend
excerpt:
  In which I try out three JavaScript frameworks to see which is best suited for
  building individual frontend widgets.
---

## Or: that framework frameworks best which frameworks least.

The last couple of years have seen a big shift of UI code from the server to
the client. JavaScript is growing up, the browser is the new OS, and the major
browser vendors have finally learned to talk out their disagreements like
responsible adults. Well, mostly. Eventually.
[Given enough time](http://www.2ality.com/2015/08/web-component-status.html).

A while ago, at the company I work for, we decided to upgrade the design of
our venerable customer portal and move some of the more interactive UI
components from our webservers to the client. And since we were really, really
serious about it: no quagmire of jQuery plugins and assorted click-handlers.
This was the year of the JavaScript framework after all. It was time to build us
some maintainable, testable, enterprise-level JS components.

So, a JS framework we would have. But which one? I was tasked to take a look at
all the options and select the one most suitable to our needs. And we certainly
had a lot of options. It seems everybody and their dog are writing
JS application frameworks these days. Some are even maintained by such well-known
companies as Facebook and Google. Time to go out exploring, meet the locals, and
see the sights.

I started by making a list of candidates. Then I began building proofs of
concept, recreating a very limited example of our customer portal using the
ideas and idioms of each framework. I continued doing this for a couple of weeks.
After that, I took stock of the code I had written for each framework, and
what my experience had been writing it.

Here's what I learned.

## Introducing...

The most well-known JavaScript framework is Google's **[AngularJS](https://angularjs.org/)**.
We had previously used  Angular to build our customer checkout process.
Our experiences with the framework were mixed. Certainly Angular made good on the
promise of a JS application framework. We were able to turn out a complete
multi-step checkout process without having to dive into the nitty-gritty details
of DOM manipulation (often).

But Angular is a bit... particular in its conventions and interpretation of the
MVC pattern. It took a while to figure out which piece goes where, which isn't
ideal when several people are working on an application simultaneously. Besides,
Angular has long been plagued by performance issues. We had to do some tuning,
rethink some of our code to get it to start working quickly after page load.

For a while now, a large team of developers has been working on **[Angular 2](https://angular.io)**.
Don't let the name fool you: this is a completely new framework with new syntax
and new ideas. The first beta came out just as we started considering it as a
candidate (it has since been released). There's talk of a pipeline for completely
pre-compiling your templates, isomorphic app support, native app support - you
name it. Angular 2 seems all set to fulfil the promise developers have been
chasing after since time immemorial: code once, run *everywhere*.

In the blue corner, coming in at a mere 43KB minified, from Facebook, it's...
**[React](https://facebook.github.io/react/)**.
React is a view engine, not a complete application framework. You'll have
to add additional libraries for e.g. validation, or communicating with the backend.
React focuses on giving excellent performance and being easy to debug.
When it came out, it blew the old Angular, and most other frameworks, away
in benchmarks. Things have
[evened out a bit](https://auth0.com/blog/2016/01/11/updated-and-improved-more-benchmarks-virtual-dom-vs-angular-12-vs-mithril-js-vs-the-rest/)
since then but React is still among the fastest JS frameworks.

React came with a lot of ideas that have since been copied by many
other frameworks. For one, it popularized reactive programming in JS frameworks.
In React, this means that views are purely a mapping of your application state
to HTML elements. There's no support for two-way databinding. Every user interaction,
server-response and event must manipulate the application state in predictable
ways. After that React recomputes an in-memory representation
called the Virtual DOM and manipulates only those actual DOM elements that have
changed since the last update.

And then there's the most popular JavaScript framework you've never heard of:
**[Aurelia](http://aurelia.io/)**. It was created by experienced framework developer
Rob Eisenberg, who was part of the development team for Angular 2 for a few
months before they went their separate ways. It's built with the very newest
JavaScript standards, some of which haven't even been finalised yet. But it
can be transpiled to plain old ES5 JavaScript so simple that even your granny's
IE9 can run it.

Aurelia and Ember are also among the few JavaScript frameworks to be a core
product of their parent companies - Angular and React are not officially endorsed
by Google or Facebook. Which means you can be reasonably sure that the framework
developers won't introduce breaking changes without warning, and that the code you
write today will still be supported years from now.

I also had a go at an **[Ember](https://emberjs.com/)** proof of concept, but found it difficult to develop only small UI components
rather than a full client-side application. Ember is strongly opionated, and its
tools and tutorials assume you're working on a brand new application.
Our company was working with legacy code, a lot of it, and it will be a long time
before we're ready to move to a single-page application. Besides, by this time
I'd been working on this for a few weeks, and my conclusion was pretty clear.

## And the winner is...

**Aurelia**, by a mile. Sure, it's still rather new, sure it doesn't have as big a
community as some of the other frameworks (though the core developers are very
dedicated and respond quickly to questions). But there is one thing Aurelia
offers that no other framework does: when I'm writing my components,
I'm writing plain old JavaScript and plain old HTML. ...well,
[TypeScript](http://www.typescriptlang.org/) and custom HTML tags, but you get
the idea.

While developing with Angular 2, I was very clearly writing Angular code (the Angular template syntax
[isn't even standards-compliant HTML](http://eisenbergeffect.bluespire.com/on-angular-2-and-html/), strictly speaking).
React invented its own template language - JSX - which isn't supported by my current tooling and probable
won't be for the foreseeable future.

By contrast, take a look at this:

	export class MyComponent {
		constructor() {
			this.numberOfClicks = 0;
		}

		handleClick(e) {
			console.log('you clicked: ' + e.target);
			console.log('number of clicks: ' + ++this.numberOfClicks);
		}
	}

and the associated template:

	<template>
		<a click.trigger="handleClick($event)">
			This is a component. Watch your console when you click it.
		</a>
	</template>

If somebody had asked me to write out my ideal component code, it would probably
have looked a lot like this. Put your template in `my-component.html` and your
code in `my-component.js` and Aurelia will wire everything up, no config or glue-code required.
It works out of the box. Anywhere I put `<my-component></my-component>`, Aurelia will render my component.

This is portable code. I know it when I see it. If Aurelia folds tomorrow (which I doubt)
and I have to move everything I've built over to another framework, I know it will work.
I'll have to move some decorators around, change some calls to framework APIs,
but the UI and business logic that I've written are plain code and will work with any halfway decent framework.

More than that: it's elegant. In fact, the code I've written since we've decided on Aurelia
and started creating real UI components has been some of the most elegant code I've ever written.
I'd say it reads like Shakespeare, except I think Shakespeare will be harder for the average English-speaker
to understand than some of the code I've pushed to our repository over the last few weeks.

There are still some rough edges here and there. The documentation of some of the more advanced features still leaves a lot
to be desired. But I'm willing to forgive a lot. Aurelia's still young, after all. Besides:
I've had nothing but earnest and helpful feedback on any issue I've opened on Github
or question I've asked on StackOverflow. And I've learned more about new and upcoming JavaScript standards
in the last few weeks than in the past few years ([async functions](https://jakearchibald.com/2014/es7-async-functions/) FTW!).

So my advice to you, based on experience, is to ignore the big brand frameworks
and get yourself familiar with the framework that gets out of your way and lets you get on with it.
[Go try it out](http://aurelia.io/docs.html#/aurelia/framework/latest/doc/article/getting-started)
and let me know what you think: I'd love to hear about your experiences.

