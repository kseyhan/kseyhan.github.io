---
layout: page
title: Categories
permalink: /categories/
---

{% assign categories = site.categories | sort %}

{% for category in categories %}

<li class="post-list">
<a href="/categories/{{ category | first | slugize }}/">
{{ category | first }} ({{ category | last | size }})
</a>
</li>
{% endfor %}
