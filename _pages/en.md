---
title: Post (English)
layout: collection
permalink: /en/
collection: en
sort_by: date
sort_order: reverse
---

{% assign sorted = site.resources | reversed %}
{% for item in sorted %}
  <h1>{{ item.name }}</h1>
  <p>{{ item.content }}</p>
{% endfor %}