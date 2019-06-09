---
layout: page
title: Categories
permalink: /categories/
order: 1
---

{% assign categories = site.categories | sort %}

{% for category in categories %}

<li class="post-list" style="font-size: {{ category | last | size | times: 400 | divided_by: categories.size }}%">
<a href="/{{ category | first | slugize }}/">
{{ category | first }} ({{ category | last | size }})
</a>
</li>
{% endfor %}
