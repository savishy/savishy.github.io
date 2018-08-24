---
layout: default
title: Vish's Posts on Technology
permalink: /blog/
---

# Posts on Technology

## Contents

<ul>
  {% for post in site.posts %}
    <li>
      <a href="{{ post.url }}">{{ post.title }}</a> - <i>{{post.date | date_to_long_string }}</i>
      {{ post.excerpt }}
    </li>
  {% endfor %}
</ul>


{% for post in site.posts %}

## <a href="{{ post.url }}">{{ post.title }}</a>
<i>Published {{post.date | date_to_long_string }}</i><br/>
<i>Categories {{post.categories | join: ', '}} </i><p/>

{{ post.excerpt }}

{% endfor %}
