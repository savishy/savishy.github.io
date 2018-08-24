---
layout: default
title: About Vish
permalink: /blog/
date: YYYY-MM-DD
---

# Blogs
<ul>
  {% for post in site.posts %}
    <li>
      {{post.date | date_to_long_string }} - <a href="{{ post.url }}">{{ post.title }}</a>
    </li>
  {% endfor %}
</ul>
