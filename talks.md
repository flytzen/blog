---
layout: page
title: Talks
permalink: /talks/
---
I give regular talks on a number of topics at various events. If you would like me to talk at an event, please contact me on [Twitter](https://www.twitter.com/flytzen) or [email](mailto:flytzen@neworbit.co.uk).

See below for [talk outlines](#talk-outlines), [speaker profile](#speaker-profiles) as well as [past and future talks](#past--future-talks).  
Video examples of talks are [here](https://youtu.be/0y-Xqsrr_kA) and [here](https://youtu.be/4HeRqsIf-jA).

## Talk Outlines

{% assign sorted_talks = site.talks | sort: 'sequence' %}
{% for talk in sorted_talks %}
  {% include card.html type="simple" title=talk.title url=talk.url excerpt=talk.excerpt cta="Read Outline" %}
{% endfor %}

## Speaker Profiles

### Short Profile
CTO and CISO at [NewOrbit](https://www.neworbit.co.uk); architecting and implementing large and small-scale software on Azure.

***

### Long Profile
Moving software to the cloud since 2006.

CTO and CISO at [NewOrbit](https://www.neworbit.co.uk); architecting and implementing large and small-scale software for our customers, all running on Microsoft Azure.

Helping other development organisations [make the most of Azure](https://neworbit.co.uk/azure/) through being an Azure Gold Partner and Reseller ("Direct CSP" in the parlance).

Obsessed with security, performance and scalability. Also, penny-pincher in the cloud.

[Download Headshot](/assets/franslytzenpicture.jpg){:.button}

## Past &amp; Future Talks
{% for talk in site.data.talkinstances %}
  {% include card.html type="simple" title=talk.title url=talk.link date-time=talk.date %}
{% endfor %}
