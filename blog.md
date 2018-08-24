---
layout: default
title: About Vish
permalink: /blog/
---

# Blogs
<ul>
  {% for post in site.posts %}
    <li>
      {{post.date}} - <a href="{{ post.url }}">{{ post.title }}</a>
    </li>
  {% endfor %}
</ul>
