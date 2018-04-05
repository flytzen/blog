---
layout: page
title: Talks
permalink: /talks/
---
I give regular talks on a number of topics at various events. If you would like me to talk at an event, please contact me on [Twitter](https://www.twitter.com/flytzen) or [email](mailto:flytzen@neworbit.co.uk).

See below for talk outlines, speaker profile as well as past and future talks.

{% for talk in site.talks %}
  {% include card.html type="simple" title=talk.title url=talk.link excerpt=talk.excerpt cta="Read Outline" %}
{% endfor %}

## Speaker profiles

### Short speaker profile
CTO and CISO at NewOrbit; architecting and implementing large and small-scale software for our customers, all running on Azure.

### Long speaker profile
Moving software to the cloud since 2006.

CTO and CISO at [NewOrbit](https://www.neworbit.co.uk); architecting and implementing large and small-scale software for our customers, all running on Microsoft Azure.

Helping other companies move to Azure as part of our Gold Cloud Partnership with Microsoft.

Obsessed with security and excited by how much easier Azure makes it to be GDPR compliant.

### Picture
!![Picture of frans](/assets/franslytzenpicture.jpg)


## Past and Future Talks
{% for talk in site.data.talkinstances %}
  {% include card.html type="simple" title=talk.title url=talk.link date-time=talk.date %}
{% endfor %}


<!-- {% assign date_format = site.minima.date_format | default: "%b %-d, %Y" %}
{% for ti in site.data.talkinstances %}| {{ ti.date | date: date_format }} | [{{ti.title}}]({{ti.link}})
{% endfor %} -->
