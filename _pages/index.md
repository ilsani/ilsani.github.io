---
title: "Blog posts"
permalink: /
layout: archive
---


{% assign postsByYear = site.posts | group_by_exp:"post", "post.date | date: '%Y'" %}
{% for year in postsByYear %}
  <h2>{{ year.name }}</h2>
  <hr />
    {% for post in year.items %}
      {% include archive-single.html %}
    {% endfor %}
{% endfor %}


