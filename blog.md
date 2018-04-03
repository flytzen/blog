---
layout: default
title: Blog
permalink: /blog/
---

# Blog

{% for post in site.posts %}
  {% include card.html title=post.title url=post.url date-time=post.date excerpt=post.excerpt tags=post.tags cta='Read Post' %}
{% endfor %}

<div class="cta--right">
  <a href="{{ "/feed.xml" | relative_url }}" class="button">Subscribe to RSS</a>
</div>
