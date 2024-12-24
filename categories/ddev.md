---
layout: page
title: DDEV
permalink: /blog/categories/ddev/
---

<h5> Posts by Category : {{ page.title }} </h5>

<div class="card">
{% for post in site.categories.ddev %}
 <li class="category-posts"><span>{{ post.date | date_to_string }}</span> &nbsp; <a href="{{ post.url | relative_url }}">{{ post.title }}</a></li>
{% endfor %}
</div>
