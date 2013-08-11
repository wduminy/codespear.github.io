---
layout: page
title: Home Page
tagline: Codespear's pages
---
{% include JB/setup %}

# Posts
<ul class="posts">
  {% for post in site.posts %}
    <li><span>{{ post.date | date_to_string }}</span> &raquo; <a href="{{ BASE_PATH }}{{ post.url }}">{{ post.title }}</a>:{{ post.excerpt }}</li>
  {% endfor %}
</ul>



