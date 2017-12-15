---
permalink: /
title: Blog
---

{% capture written_year %}'None'{% endcapture %}
{% for post in site.posts %}
  {% include archive-single.html %}
{% endfor %}
