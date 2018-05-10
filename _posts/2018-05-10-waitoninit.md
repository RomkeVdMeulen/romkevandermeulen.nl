---
title: "@init / @waitOnInit TypeScript method decorators"
date: 2018-05-10
tags:
  - ts
excerpt:
  These TypeScript method decorators make lazy loading easy by ensuring that
  certain (async) functions are always called before others.
---

Imagine a very simple class that loads some data and has a couple of accessors
for that data:

```ts
class UserOverview {

    private users: User[];

    constructor() {
        this.loadUsers();
    }

    getUserById(id: number) {
        return this.users.find(user => user.id === id);
    }

    getAllUsers() {
        return this.users;
    }

    private async loadUsers() {
        this.users = await myUserStore.getUsers();
    }
}
```

Off course, this will automatically start loading users the moment an instance
is created, rather than when users are actually needed. And the constructor
can't block until the request is complete, so you just have to hope loading will
be done by the time any of the getters are called. We can do better. Let's add
some lazy loading, and make the accessors await the load request:

```ts
class UserOverview {

    private users: User[];

    async getUserById(id: number) {
        if (!this.users) {
            await this.loadUsers();
        }
        return this.users.find(user => user.id === id);
    }

    async getAllUsers() {
        if (!this.users) {
            await this.loadUsers();
        }
        return this.users;
    }

    private async loadUsers() {
        this.users = await myUserStore.getUsers();
    }
}
```

Great, but now we have duplicate code, and depending on timing the load request
may be fired more than once. It would be nice if we could abstract all this
wiring away with something like:

```ts
class UserOverview {

    private users: User[];

    @waitOnInit
    async getUserById(id: number) {
        return this.users.find(user => user.id === id);
    }

    @waitOnInit
    async getAllUsers() {
        return this.users;
    }

    @init
    private async loadUsers() {
        this.users = await myUserStore.getUsers();
    }
}
```

Well, now we can! Here's how to make these decorators work. You can add each
decorator to as many methods as you'd like. The unit tests directly below use
[Chai as Promised](https://github.com/domenic/chai-as-promised).

{%gist fb5fbbfd2f8154c6cb4590f058cdb728 %}
