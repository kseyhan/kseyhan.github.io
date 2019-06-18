---
layout: page
title: Tags
permalink: /tag2/
---

{% assign tags = site.tags | sort %}

{% for tag in tags %}

<li class="post-list" style="font-size: {{ tag | last | size | times: 500 | divided_by: tags.size }}%">
<a href="/tags/{{ tag | first | slugize }}/">
{{ tag | first }} ({{ tag | last | size }})
</a>
</li>
{% endfor %}
