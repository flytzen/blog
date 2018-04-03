---
layout: default
---
# Welcome to my randomness

Hi, I'm Frans.

I love technology and I enjoy <a href="/blog/">writing</a> and <a href="/talks/">speaking</a> about security, .Net, Azure and GDPR - not necessarily in that order.
When not doing that, I design and build systems at <a href="https://neworbit.co.uk">NewOrbit</a>.

***

## My Next Talk

{% for talk in site.data.talkinstances limit:1 %}
  {% include card.html type="simple" title=talk.title url=talk.link date-time=talk.date %}
{% endfor %}

<div class="cta cta--right">
  <a href="" class="button">Talks</a>
</div>

***

## Recent Blog Posts

{% for post in site.posts limit:4 %}
  {% include card.html title=post.title url=post.url date-time=post.date excerpt=post.excerpt cta='Read Post' %}
{% endfor %}

<div class="cta cta--right">
  <a href="" class="button">Blog</a>
</div>
