---
layout: page
title: Lando
permalink: /blog/categories/lando/
---

<h5> Posts by Category : {{ page.title }} </h5>

<div class="card">
{% for post in site.categories.lando %}
 <li class="category-posts"><span>{{ post.date | date_to_string }}</span> &nbsp; <a href="{{ post.url | relative_url }}">{{ post.title }}</a></li>
{% endfor %}
</div>
