---
layout: page
title: Index
tagline: ramblings, rants and other stuff
---

<ul class="posts">
  {% for post in site.posts %}
    <li><span>{{ post.date | date_to_string }}</span> &raquo; <a href="{{ BASE_PATH }}{{ post.url }}">{{ post.title }}</a> {{ post.excerpt }}</li>
  {% endfor %}
</ul>
<!-- i removed the link to the docs because I have not been working on gameex for a while; and I have some new ideas around the gamelib -->
<!-- Also here: [The GameEx Documentation](/gameex/html/) -->
