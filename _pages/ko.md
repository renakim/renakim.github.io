---
title: Post (한글)
layout: collection
permalink: /ko/
collection: ko
sort_by: date
sort_order: reverse
---

{% assign sorted = site.resources | reverse %}
{% for item in sorted %}
  <h1>{{ item.name }}</h1>
  <p>{{ item.content }}</p>
{% endfor %}