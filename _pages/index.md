---
title: "Blog posts"
permalink: /
layout: archive
---


{% assign postsByYear = site.posts | group_by_exp:"post", "post.date | date: '%Y'" %}
{% for year in postsByYear %}
  <h2>{{ year.name }}</h2>
    <ul>
      {% for post in year.items %}
        {% include archive-single.html %}
      {% endfor %}
    </ul>
{% endfor %}


