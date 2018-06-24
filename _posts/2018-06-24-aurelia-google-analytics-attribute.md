---
title: "Aurelia custom attribute for Google Analytics events"
date: 2018-06-24
tags:
  - ts
  - aurelia
excerpt:
  This custom attribute for Aurelia can be used to easily add event listeners
  to any element and send custom data to Google Analytics.
---

I wrote this Aurelia custom attribute to listen for DOM events on any element
and send custom event data to the Google Analytics API. It is pretty flexible
and worked well for the app I needed it for, which had a limited scope. For
larger apps, adding tracking code to every button and link may be a bit
cumbersome: I'd recommend using an [Aurelia Google Analytics plugin](https://github.com/miguelzakharia/aurelia-google-analytics)
for that.

Here's a simple use:

```html
<button type="button" analytics="PrimarySalesButton">Order now</button>
```

This will send `{"action": "PrimarySalesButton"}` to analytics when the button
gives off a `click` event. A more complex example might look like this:

```html
<form analytics="on.bind: 'submit'; event.bind: 'ContactFormSubmitted'; category.bind: 'CustomerContact'; label.bind: 'Send your question'">
    ...

    <input type="submit" value="Send your question">
</form>
```

This will send `{"action": "ContactFormSubmitted", "category": "CustomerContact", "label": "Send your question"}` when it receives a `submit` event from the form.

You can choose to include the attribute only in views that use it, but I found
it easier to add it as a [global resource](https://aurelia.io/docs/templating/html-behaviors-introduction#global-resources).

{% gist 1c43ca25858059ceed3723ca9e699eab %}

P.S.: this is of course assuming that you've already initialized the Analytics
tracking code before this is called. See the
[Google Analytics instructions for setting up tracking](https://support.google.com/analytics/answer/1008080).
