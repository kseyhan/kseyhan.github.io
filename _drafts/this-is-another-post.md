---
---

my content grows and grows

{% for person in site.data.people %}
<h1>{{ person.name}}, {{ person.occupation}} </h1>
{% endfor %}
