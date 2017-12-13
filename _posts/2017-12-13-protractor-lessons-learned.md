---
title: "Protractor: lessons learned"
date: 2017-12-13
tags:
  - js
  - ts
  - protractor
---

I've been working with [Protractor](http://www.protractortest.org) a lot in recent monts,
writing automated tests for the web UI of our company's customer portal.
In this post, I thought I'd share some of the lessons I've learned along the way,
and give some practical advice.

Here's an outline to help you decide which parts of this article will be of use you:

* [browser.wait and ExpectedConditions](#browserwait-and-expectedconditions):
  if you're not sure your UI will update fast enough for your test to pass, use
  these Protractor features to delay the test until the update is done.
* [Marker attributes](#marker-attributes): relying on template HTML structure or
  CSS class names to find elements in your test couples your test to those
  structures, making them more likely to fail any time you change something in
  your UI styling. Instead, use special markers that don't change when your
  styling or layout does, like classes with a predefined prefix, or a custom
  attribute.
* [Protractor and the Promise Manager](#protractor-and-the-promise-manager):
  Protractor ships with the Promise Manager system enabled by default, but it's
  scheduled to be deprecated. If you start writing Protractor code now, disable
  this system manually so you won't have to migrate in the future. Use async /
  await instead.
* [Mock backend](#mock-backend): to test your app's behaviour with unexpected
  HTTP responses, you need to control those responses from the tests. Building
  a mock implementation of your backend may be a more realistic and flexible way
  to do this than overriding parts of the app directly and never sending the
  request in the first place. Fortunately, building a fake backend is pretty
  easy to do.
* [PO layer](#po-layer): it's a good idea to build an abstraction layer between
  the structure of your UI and your Protractor tests. The example given in the
  Protractor doc is outdated though: here I show the structure we came up with.
* [Conclusion](#conclusion): your UI tests are a first-class citizen of your
  code base. Put thought into its structure and design before writing your first
  tests, or you'll incur technical debt, like I did.

## browser.wait and ExpectedConditions

One of the most common problems I run into when using Protractor is its
sensitivity to timing issues. It's easy to ask Protractor to click a button
and then check that a certain element has appeared. But if some complex logic is
involved, or even a lengthy HTTP request, then it may take your UI some time to
update. If Protractor looks for the element to appear before it does, then it
rightly throws an error. And this error is unfortunately rather non-deterministic:
in some test runs your UI may update just in time, and no error is thrown.

Preventing such false positives requires you to put a lot of thought into your
tests. If you're not sure the UI will update in time for your tests to pass,
you may need to delay your tests until the update is complete. Fortunately,
Protractor gives you `browser.wait` and `ExpectedConditions` to do just that.
For example:

{% highlight js %}
const EC = ExpectedConditions;
element(by.tagName('button')).click();
browser.wait(EC.presenceOf(element(by.id("my-text")), 500);
{% endhighlight %}

This will test the same behavior as `expect(element(by.id("my-text")).isPresent()).toBe(true)`
would, but it gives your UI a 500ms window in which to update. If it doesn't,
a timeout error will be thrown from `browser.wait`.

## Marker attributes

Protractor gives you a lot of options to select elements in your web UI based
on tag names, CSS classes, or complicated selectors. However, by using these
methods of finding elements in your UI, your coupling your Protractor tests to
the structure of your HTML, which isn't completely stable. So every time you go
in and add a `<div>` or change a class name, you may inadvertently break your
Protractor tests. That's far from ideal.

One solution is to agree to a naming-convention for classes used to match UI
elements in your tests. Something like `<button class="e2e-submit">` for example.
The class prefix shows that this element is used in your end-to-end tests.
So you know not to use that class for styling purposes, or to change the button
without updating your tests.

My colleagues and I took this a step further. Seeing as classes are intended to
be used for styling, it seemed awkward to use them for tests. We introduced a
custom attribute for marking elements instead. Something like `<button e2e="submit">`.
I've even written a [custom Protractor locator for matching elements using a
predefined attribute]({{ site.baseurl }}/2017/08/30/protractor-by-marker-locator.html).

## Protractor and the Promise Manager

Protractor is built for asynchronous execution. So if you issue a command, like
`element(by.tagName('button')).click()`, the action will not necessarily be done
by the time the execution of that statement is complete. However, if you've
written any Protractor code before you may not have been aware of this.
The reason for that is that WebDriver, which Protractor is built on, ships with
a system called the `PromiseManager`. Roughly speaking, this system keeps track
of all the asynchronous commands issues through the WebDriver API, and makes
sure the last command has been completed before the next starts.

The Promise Manager is a convenient system, but it is
[scheduled to be deprecated](https://github.com/seleniumhq/selenium/issues/2969).
This system was added in a time before Promises and async / await were added
to the ECMA Script standard. The Promise Manager adds a lot of complexity,
making code harder to debug. With these new language features it becomes simple
enough to just manage asynchronous operations yourself. A Protractor test
built with async / await might look something like this:

{% highlight js %}
describe('The contact form', () => {

    it('ensures the subject field is filled out before submitting', async () => {
        await browser.get('/contact');

        const subjectfield = element(by.css('input[name="subject"]'));
        const error = element(by.id('subject-error'));

        await expect(subjectfield.getAttribute('value')).toBe('');
        await expect(error.isPresent()).toBe(false);

        await element(by.css('input[type="submit"]')).click();

        await expect(error.isPresent()).toBe(true);
        await expect(error.getText()).toBe('Please enter a subject for your message');
    });
});
{% endhighlight %}

If you decide to start adding Protractor code to your project, I advice you to
start writing async / await code right from the start so you won't have to
migrate later. Async / await should work natively starting with Node 8, but if
that doesn't work for you, you can always have it transpiled using e.g. Babel or
TypeScript. If you do decide to manage asynchronous timing yourself, don't
forget to disable the Promise Manager system (it is still enabled by default for
now, though that will change as part of the deprecation process). Check the
[instructions for using Protractor with async / await](https://github.com/angular/protractor/blob/master/docs/async-await.md).

## Mock backend

To completely test your UI's behaviour, you have to check what it does if an
AJAX resolves slowly, or fails altogether. To do that, you have to control the
responses to your UI's requests. There are several ways to do this. For instance,
you can route all requests in your application through one component, and then
override that component during tests.

However, I much prefer a different approach: overriding only the URL of requests,
and pointing them to a mock-implementation of the backend. By really doing the
HTTP request and processing the response, your tests will be far closer to actual
production behaviour.

Building a mock-implementation of your backend is not all that difficult. For my
last two projects I've built mock backends using [Express](https://expressjs.com/).
A simple mock backend resource in Express might look something like this:

{% highlight js %}
app.get("/backend/invoices", (req, res) => {
    if (req.query["load-failure"] === "true") {
        setTimeout(() => res.sendStatus(500), 500);
        return;
    }

    if (req.query["no-invoices"] === "true") {
        res.send([]);
        return;
    }

    setTimeout(() => res.send(DUMMY_INVOICE_DATA), 200);
});
{% endhighlight %}

As you can see, this backend supports testing of a couple of scenarios,
triggered by adding certain GET parameters. By default, the predefined dummy
invoice data will be sent, with a delay of 200ms to more realistically simulate
a network request (this backend runs on the same system that runs the tests,
and without this delay the response would be far too fast).  However, by adding
`load-failure=true` to the URL in some way, the app will be served a
`500 Internal Server Error` after 500ms. As you can see, this method gives you
a lot of flexibility to test all kinds of possible states of your production
backend and see how your app responds to them.

# PO layer

The Protractor documentation advises that you build an abstraction layer between
the structure of your UI and your tests. They call it a
[Page Object](http://www.protractortest.org/#/page-objects).
Here's an example from the Protractor doc:

{% highlight js %}
var AngularHomepage = function() {
  var nameInput = element(by.model('yourName'));
  var greeting = element(by.binding('yourName'));

  this.get = function() {
    browser.get('http://www.angularjs.org');
  };

  this.setName = function(name) {
    nameInput.sendKeys(name);
  };

  this.getGreeting = function() {
    return greeting.getText();
  };
};
{% endhighlight %}

Apart from the outdated ES5 syntax, this is a fine start to your abstraction
layer. As my colleagues and I found out, though, just because Protractor calls
this a Page Object, that doesn't mean it's a smart move to write exactly one
such class per page. I suggest writing your PO classes at roughly the same level
as your UI components.

After some experimenting, we came up with a more hierarchical structure that
groups properties and methods by UI element, using nested plain objects.
Here's an example (see the [Marker attributes](#marker-attributes) section
to see what `by.marker` does).

{% highlight js %}
class Addresses {

    get title() {
        return element(by.marker("title")).getText();
    }

    get description() {
        return element(by.marker("description")).getText();
    }

    get overview() {
        const overview = element(by.marker("address-overview"));
        const addresses = overview.all(by.marker("address"));
        return {
            get present() {
                return overview.isPresent();
            },

            get addressCount() {
                return addresses.count();
            },

            getAdres(number: number) {
                const address = addresses.get(number - 1);
                return {
                    get name() {
                        return address.element(by.marker("name")).getText();
                    },
                };
            },
        };
    }
}
{% endhighlight %}

Then you also add a support class that encapsulates every component included in
your test, plus some logic for loading different scenarios:

{% highlight js %}
class AddressesTest {

    constructor() {
        this.addresses = new Addresses();
    }

    load({noAddressData = false} = {}) {
        return browser.get(`/addressestest?no-address-data=${noAddressData}`);
    }
}
{% endhighlight %}

With these classes, the actual Protractor test can be written like this (using
async / await: see the section
[Protractor and the Promise Manager](#protractor-and-the-promise-manager)):

{% highlight js %}
describe('The Addresses component', () => {

    const test = new AddressesTest();
    const overview = test.addresses.overview;
    beforeEach(() => test.load());

    it('shows the title and description from the CMS', async () => {
        await expect(test.addresses.title).toBe("Your addresses");
        await expect(test.addresses.description).toBe("Manage your addresses.");
    });

    it('loads and shows an overview of addresses', async () => {
        await expect(overview.present).toBe(true);
        await expect(overview.addressCount).toBeGreaterThan(0);
        await expect(overview.getAddress(1).name).toBe("Address 1");
    });

    describe('if there are no addresses', () => {

        beforeEach(() => test.load({noAddressData: true}));

        it('doesn\'t show the overview', async () => {
            await expect(overview.present).toBe(false);
        });
    });
});
{% endhighlight %}

Nice and simple, no?  My colleagues and I have been using TypeScript for a while
now, and it makes importing these support classes and writing the actual test a
breeze.

Of course, you're free to organize the PO layer as you see fit. I do recommend
that you take some time, before you start writing tests, to think about how
you want this abstraction layer to be structured. We started out just
following the example from the Protractor documentation. It took us weeks
to reorganize the abstraction layer to what I've shown here.

## Conclusion

> The moral of this story is simple: Test code is just as important as production code.
> It is not a second class citizen. It requires thought, design and care.
> It must be kept as clean as production code.
>
> – Clean Code, Robert C. Martin

This is basically the lesson I learned by writing an extensive Protractor test
suite. Your testing code is a first-class part of the project. Adding tests
without thinking about your code design or architecture will land you in no less
of a mess than doing the same thing while writing production code.
By the time I realized this, it took me a few weeks to get rid of the technical
debt I myself had incurred. So next time, I will be putting a lot more thought
into the design of the entire code base, including the tests.
And I advise you to do the same. 😉
