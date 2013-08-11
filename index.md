---
layout: page
title: Tumblehome
tagline: ramblings, rants and other stuff
---

<ul class="posts">
  {% for post in site.posts %}
    <li><span>{{ post.date | date_to_string }}</span> &raquo; <a href="{{ BASE_PATH }}{{ post.url }}">{{ post.title }}</a> {{ post.excerpt }}</li>
  {% endfor %}
</ul>



