---
layout: page
title: Tags
permalink: /tags/
order: 2
---

{% assign tags = site.tags | sort %}

{% for tag in tags %}

<li class="post-list" style="font-size: {{ tag | last | size | times: 400 | divided_by: tags.size }}%">
<a href="/{{ tag | first | slugize }}/">
{{ tag | first }} ({{ tag | last | size }})
</a>
</li>
{% endfor %}
