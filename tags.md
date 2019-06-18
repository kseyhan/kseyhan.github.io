---
layout: page
title: Tags
permalink: /tags/
---

{% assign tags = site.tags | sort %}

{% for tag in tags %}

<li class="post-list">
<a href="/tags/{{ tag | first | slugize }}/">
{{ tag | first }} ({{ tag | last | size }})
</a>
</li>
{% endfor %}
