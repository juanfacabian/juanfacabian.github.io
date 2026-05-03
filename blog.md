---
title: Blog
permalink: /blog/
---

# Blog

Short essays, project notes, and research-adjacent writing.

<ul class="post-list">
  {% for post in site.posts %}
    <li>
      <a class="post-title" href="{{ post.url | relative_url }}">{{ post.title }}</a><br>
      <span class="post-date">{{ post.date | date: "%B %-d, %Y" }}</span>
      {% if post.excerpt %}
        <p>{{ post.excerpt | strip_html | truncate: 180 }}</p>
      {% endif %}
    </li>
  {% endfor %}
</ul>
