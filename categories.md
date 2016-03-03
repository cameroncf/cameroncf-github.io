---
layout: page
title: Categories
---

{% capture categories %}
  {% for category in site.categories %}
    {{ category[0] }}
  {% endfor %}
{% endcapture %}
{% assign sortedcategories = categories | split:' ' | sort %}

{% for category in sortedcategories %}
  <h3 id="{{ category }}">{{ category }}<!-- {{ site.categories[category].size }} --></h3>
  <ul>
  {% for post in site.categories[category] %}
    <li>
      {{ post.date | date: "%B %d, %Y" }}: <a href="{{ post.url }}">{{ post.title }}</a>
    </li>
  {% endfor %}
  </ul>
{% endfor %}