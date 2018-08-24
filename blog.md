---
layout: default
title: About Vish
permalink: /blog/
---

# Blogs
<ul>
  {% for post in site.posts %}
    <li>
      <a href="{{ post.url }}">{{ post.title }}</a><br/>
      <i>Published {{post.date | date_to_long_string }}</i>
    </li>
  {% endfor %}
</ul>
