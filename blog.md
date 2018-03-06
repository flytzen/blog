---
layout: default
title: Blog
permalink: /blog/
---

<div class="home">
    <h1 class="page-heading">Blog posts</h1>
    <div>
        <h2 class="post-list-heading">Posts</h2>
        <ul class="post-list">
        {% for post in site.posts %}
        <li>
            {% assign date_format = site.minima.date_format | default: "%b %-d, %Y" %}
            <span class="post-meta">{{ post.date | date: date_format }}</span>
            <h3>
            <a class="post-link" href="{{ post.url | relative_url }}">
                {{ post.title | escape }}
            </a>
            </h3>
            {{ post.excerpt | strip_html | truncatewords: 50 }}
        </li>
        {% endfor %}
        </ul>

        <p class="rss-subscribe">subscribe <a href="{{ "/feed.xml" | relative_url }}">via RSS</a></p>
    </div>
    <div>
        over to the right, should have the tags that a user can click on
    </div>
</div>
