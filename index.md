---
layout: default
---
# Welcome to my randomness

Hi, I'm Frans.

I love technology and I enjoy <a href="/blog/">writing</a> and <a href="/talks/">speaking</a> about security, .Net, Azure and GDPR - not necessarily in that order.
When not doing that, I design and build systems at <a href="https://neworbit.co.uk">NewOrbit</a>.

***

## My next talk:

<a href="https://www.meetup.com/dotnetoxford/events/247607774/">how to secure your web apps in Azure at .Net Oxford on 24 April</a>

***

## Recent Blog Posts

<div class="homeboxes">
    <div class="homeblogbox">
            <ul class="post-list">
                {% for post in site.posts limit:4 %}
                <li>
                    <h3>
                        <a href="{{ post.url | relative_url }}">
                            {{ post.title | escape }}
                        </a>
                    </h3>
                    <div class="excerpt">
                        {{ post.excerpt | strip_html | truncatewords: 30 }}
                    </div>
                    <hr />
                </li>
                {% endfor %}
            </ul>

            <a href="/blog">More...</a>
    </div>
    <div class="homesidebar">
        <div>
            <h2><a href="/talks/">Speaking</a></h2>
            <h3><a href="https://www.meetup.com/dotnetoxford/events/247607774/">How to secure your web app in Azure</a></h3>
            <small>April 24, .Net Oxford</small>
        </div>
        <!-- <div>
            <a class="twitter-timeline" href="https://twitter.com/flytzen" data-height="300">Tweets by @flytzen</a>
            <script async src="https://platform.twitter.com/widgets.js" charset="utf-8"></script>
        </div> -->
    </div>
</div>
