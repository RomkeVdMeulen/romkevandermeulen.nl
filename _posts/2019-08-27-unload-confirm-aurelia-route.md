---
title: "@confirmUnload decorator for Aurelia route View Models"
date: 2019-08-27
tags:
  - ts
  - aurelia
excerpt:
  If a page has complicated forms, you may want to ask the user to confirm when
  they try to navigate away while there are still unsaved changes. Applying this
  decorator to Aurelia route View Models does that for you.
---

Your webapp may contain some complicated forms or user-interactions, and the
user can at times get confused about which changes they have entered that have
not yet been saved to the backend. To prevent frustration later on, you want to
make sure no hard work of the user is lost. Luckily the web platform offers a
simple solution: the [beforeunload event](https://developer.mozilla.org/en-US/docs/Web/API/Window/beforeunload_event).

The idea is that you register a listener for these events, and when you
determine that there are in fact unsaved changes, you call `preventDefault()` on
it. In response, the browser will show the user a built-in confirmation dialog,
asking them if they are sure they want to leave the page and lose the unsaved
changes. I'm sure you've seen some of these dialogs yourself.

```js
window.addEventListener('beforeunload', (event) => {
  if (hasUnsavedChanges()) {
    event.preventDefault();
    // See https://developer.mozilla.org/en-US/docs/Web/API/Window/beforeunload_event
    event.returnValue = '';
  }
});
```

But if you're using `aurelia-router` for frontend navigation, not every action
by the user will give you a `beforeunload` event &mdash; only full-page navigation or
refreshes. The Aurelia router has its own hook for preventing navigation:
`canDeactivate()`.

```js
class MyRoute {

  canDeactivate() {
    return !this.hasUnsavedChanges();
  }

  hasUnsavedChanges() {
    // [...]
  }
}
```

I wrote a simple decorator that can be applied to the View Model class of any
Aurelia route that will register a `beforeunload` listener and implement
`canDeactivate()`. Both will call `hasUnsavedChanges()` on the class, and if
true is returned, will ask the user to confirm. If the user declines, navigation
is canceled. Since the confirmation dialog triggered by `canDeactivate()`
doesn't rely on the browser default, you will have to give the confirm
message yourself with `getUnloadConfirmMessage()` (or you can edit this
decorator and hard-code it if you want). I use a simple function called
`decorateMethod()` to add methods to a class if they don't exist or wrap them if
they do. Let me know if you want the implementation for that, but it's pretty
straight-forward.

{% gist 214232ef7b61a2c514787d45d6f5aca4 %}
