---
title: 'Archived Posts'
layout: page
published: true
---

<ul class="posts">
  {% for post in site.posts %}
    <li><span>{{ post.date | date: "%m/%d/%y" }}</span> &raquo; <a href="{{ post.url }}">{{ post.title }}</a></li>
  {% endfor %}
</ul>
