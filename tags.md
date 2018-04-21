---
layout: page
title: Tags
permalink: /tags/
---

{% for tag in site.tags %}
  <div>
    {% capture tag_name %}{{ tag | first }}{% endcapture %}
    <div id="#{{ tag_name | slugize }}"></div>
        <a name="{{ tag_name | slugize }}"></a>
        <h3>{{ tag_name }}</h3>
        <ul>
            {% for post in site.tags[tag_name] %}
                <li>
                    <a href="{{ site.baseurl }}{{ post.url }}">{{post.title}}</a>
                </li>
            {% endfor %}
        </ul>
  </div>
{% endfor %}
