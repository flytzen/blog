---
layout: default
---
<h1 class="page-heading">Welcome to my randomness</h1>
<p>
Hi, I'm Frans.
I love technology and I enjoy <a href="/blog/">writing</a> and <a href="/talks/">speaking</a> about security, .Net, Azure and GDPR - not necessarily in that order. <br>
When not doing that, I design and build systems at <a href="https://neworbit.co.uk">NewOrbit</a>.<br>
My next talk will be on <a href="https://www.meetup.com/dotnetoxford/events/247607774/">how to secure your web apps in Azure at .Net Oxford on 24 April</a>
</p>

<div class="homebox">
    <h2><a href="/blog/">Blog</a></h2>
        <ul class="post-list">
            {% for post in site.posts limit:3 %}
            <li>
                <h3>
                    <a href="{{ post.url | relative_url }}">
                        {{ post.title | escape }}
                    </a>
                </h3>
                <div class="excerpt">
                    {{ post.excerpt | strip_html | truncatewords: 30 }}
                </div>
            </li>
            {% endfor %}
        </ul>

        <a href="/blog">More...</a>
</div>

