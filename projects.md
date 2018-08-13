---
layout: default
title: Vish's Projects
permalink: /projects/
---
{% include header.md %}

{% include liquid_summary.md %}

{% comment %}
Loop through each project for project details.
{% endcomment %}

**Project Details**

{% for post in site.posts %}
* [{{ post.title }}]({{ post.url }})
{% endfor %}


{% for project in site.data.projects %}

## {{project.summary}}

{% assign tools = project.technologies | join: ", " %}
{% assign sd = project.startdate | date: '%s' %}
{% assign statusStr = project.status %}
{% assign roleStr = project.roles | join: ', ' %}
{% if project.status == 'complete' %}
  {% assign ed = project.enddate | date: '%s' %}
  {% assign diffSeconds = ed | minus: sd %}
  {% assign diffMonths = diffSeconds | divided_by: 3600 | divided_by: 24 | divided_by: 30 | append: ' months' %}
  {% assign statusStr = statusStr | append: ' - ' | append: diffMonths %}
{% endif %}

* *Client: {{project.client}}*
* *Tools: {{tools}}*
* *Status: {{statusStr}}*
* *Roles: {{ roleStr }}*

**Key Activities**

{% for desc in project.description %}
* {{desc}}
{% endfor %}

{% endfor %}
