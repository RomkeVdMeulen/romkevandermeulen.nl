---
title: "Insert hard linebreaks into textareas with vanilla JavaScript"
date: 2016-06-30
tags:
  - javascript
  - html
---
While working on my little [checklist app]({{ site.baseurl }}/2016/06/30/romkes-checklists.html)
I needed a way to hard-wrap text entered into a `<textarea>` and preserve the
linebreaks when copy-pasting it somewhere else. HTML and CSS don't support this,
so I wrote some vanille JavaScript code to insert the linebreaks, trying to
break on words whenever possible.

{% gist b59ca36629559a4951256e4b59672580 %}
