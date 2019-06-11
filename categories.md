---
layout: page
title: Categories
permalink: /categories/
---

{% assign categories = site.categories | sort %}

{% for category in categories %}

<li class="post-list" style="font-size: {{ category | last | size | times: 400 | divided_by: categories.size }}%">
<a href="/categories/{{ category | first | slugize }}/">
{{ category | first }} ({{ category | last | size }})
</a>
</li>
{% endfor %}
