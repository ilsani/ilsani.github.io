---
permalink: /
title: Blog
layout: archive
---

{% capture written_year %}'None'{% endcapture %}
{% for post in site.posts %}
  {% include archive-single.html %}
{% endfor %}
