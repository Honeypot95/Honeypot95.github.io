---
layout: page
title: Archive
permalink: /archive/
---

Archive of all {{ site.posts.size }} posts.

<ul>
  {% for post in site.posts %}
    <li>
      <time datetime="{{ post.date | date: '%Y-%m-%d'}}">{{ post.date | date: "%Y-%m-%d"}}</time>
      <a href="{{ post.url }}">{{ post.title }}</a>
    </li>
  {% endfor %}
</ul>
