{% comment %}
Generate summary content
{% endcomment %}

{% assign projectCount = site.data.projects | size %}
{% assign tools = '' | split: '' %}
{% for project in site.data.projects %}
  {% assign tools = tools | concat: project.technologies %}
{% endfor %}
{% assign toolsUniq = tools | uniq %}

Vish has worked on {{projectCount}} projects.
