---
layout: page
title: Steamdeck
permalink: /blog/categories/steamdeck/
---

<h5> Posts by Category : {{ page.title }} </h5>

<div class="card">
{% for post in site.categories.steamdeck %}
 <li class="category-posts"><span>{{ post.date | date_to_string }}</span> &nbsp; <a href="{{ post.url | relative_url }}">{{ post.title }}</a></li>
{% endfor %}
</div>
