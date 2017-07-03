---
layout: page
title:  "Articles"
permalink: "/articles/"
---

{% for category in site.categories %}
  <h2>{{ category[0] }}</h2>
  <ul>
    {% for post in category[1] %}
      <li><a href="{{ post.url }}">{{ post.title }}</a>&nbsp;-&nbsp;<small><strong>{{ post.date | date: "%B %e, %Y" }}</strong></small></li>
    {% endfor %}
  </ul>
{% endfor %}
