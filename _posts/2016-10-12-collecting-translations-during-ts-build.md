---
title: "Collecting translation keys during TypeScript build"
date: 2016-10-12
tags:
  - typescript
  - i18n
---

I found myself in the position of building a lightweight i18n mechanism for the
frontend of an enterprise app recently. I implemented a couple of decorators
that inject translations into properties. I also extended the Gulp build process
to gather all translation keys into a single file for me, which I could then use
on the backend to deliver only the required translations, rather than send over
the entire dictionary. Here's how.

The goal is to be able to decorate a property and have access to a translation
from JavaScript (injecting translations into templates is another story). It
would look like this:

{% highlight ts %}
import {translation} from "my-translation-service";

class MyFancyEditor {

	@translation("editor-error-toolong")
	private errorMessageTooLong: string;

	// ...

	validate() {
		if (this.content.length > 140) {
			this.errors.push(this.errorMessageTooLong);
		}
	}
}
{% endhighlight %}

I started off with stubbing the decorator:

{% highlight js %}
export function translation(textkey: string) {
	return (target: any, property: string) => void
}
{% endhighlight %}

Later on I also added another decorator: `@translationPromise`. The idea was
that while `@translation` injects the translation once it's loaded and doesn't
touch the property before then, the `@translationPromise` injects a Promise for
the translation immediately, so you can wait for it if need be.

Then I went into the Gulp build progress. Typescript was compiled with
[gulp-typescript](https://www.npmjs.com/package/gulp-typescript), based on
[the Aurelia skeleton project](https://github.com/aurelia/skeleton-navigation/tree/master/skeleton-typescript).

I used the TypeScript compiler to walk the source code Abstract Syntax Tree,
looking for my new decorators and extracting the text keys used. I put this in
a through-stream and plugged it into the Gulp build (line 28):

{% gist fee35daf1bad6ac8589188da636b4a03 %}

Now, whenever my TypeScript was compiled, the translation keys would be written
to `../build/META-INF/frontend-textkeys.conf`. Then I just read in this file in the
backend, which for this project was Java EE:

{% gist 0536d9d01dbf37deffe138d01db03e07 %}

Now I just need to implement a REST resource to serve the translations for
every available language. And boom, translations are served:

{% highlight js %}
{
  "editor-error-toolong": "Sorry, your message is longer than the maximum 140 characters!",
  // and many more...
}
{% endhighlight %}

Now I just create a frontend service to read this in, and I can implement the
last puzzle piece: the decorators, which all this started with.

{% gist a81a86d61f776c89c48770729ed65dd7 %}

I hope you've found this useful. The code for hooking into the TS build can be
used in other creative ways, I'm sure. The current implementation can't work
with dynamically composed text keys yet. If you find a clever solution for that,
be sure to let me know!
