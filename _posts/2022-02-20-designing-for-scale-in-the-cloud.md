---
layout: post
title: Designing for scale in the cloud 101
date: '2022-02-20'
author: Frans Lytzen
tags: Azure
modified_time: '2022-02-20'
excerpt: 15 minutes to teach two tips on how to improve scale and resilience in the cloud.
---
Having spent more than a decade designing and building systems in the Cloud there are two basic things I almost always use queues and multiple "databases". This is very simple to do, yet can massively increase both scale and resilience.

I gave a 15-minute Lightning Talk at [.Net Oxford](https://www.dotnetoxford.com/posts/2022-02-lightning-talks) in February 2022 that explains this.

# Video  
<iframe width="560" height="315" src="https://www.youtube.com/embed/FUCAsAHvbFo" title="YouTube video player" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture" allowfullscreen></iframe>

# Slides
<iframe src="//www.slideshare.net/slideshow/embed_code/key/ay844F2Ttr6ovx" width="595" height="485" frameborder="0" marginwidth="0" marginheight="0" scrolling="no" style="border:1px solid #CCC; border-width:1px; margin-bottom:5px; max-width: 100%;" allowfullscreen> </iframe> <div style="margin-bottom:5px"> <strong> <a href="//www.slideshare.net/FransLytzen/designing-for-scale-and-resilience-in-the-cloud-101" title="Designing for scale and resilience in the cloud 101" target="_blank">Designing for scale and resilience in the cloud 101</a> </strong> from <strong><a href="//www.slideshare.net/FransLytzen" target="_blank">Frans Lytzen</a></strong> </div>

# Summary
When thinking about developing an application for deployment in the Cloud, be it Azure, AWS or Google, many people fall into one of two camps:

- Just do what you normally do because the Cloud will handle all scaling concerns.
- Re-think everything and learn all the latest buzz so your site can scale to a quintillion users.

For most people, most of the time, both of these statements are untrue. 99% of sites and web apps do not need to scale to thousands of concurrent users and rarely need the more complicated aspects of designing for large scale. At the same time, the Cloud really won’t just magically scale up in response to demand.

The talk is language agnostic. I do use Azure for examples, but the principles apply equally to all the cloud providers.

## Queues
Let's say users can register for an event on your site. When a user registers, you look up their geo-location via some external service and you then save the registration in a database. It is simple and you may choose to do it all in memory. This has several problems.

- If word gets around, you may get a surge of users all wanting to register at the same time. 
  - At first, your website doesn't scale out fast enough (see the video as to why) so your users start to see 503 errors after submission - and their data is lost.  
  - Once you scale out your webservers to handle the traffic, your database buckles under the load. Your website returns 500 errors to users - and you lose data until you manage to scale out the database.  
- If there is a bug in your code that causes an error - you lose the data.
- If the external service you are calling goes down, you may lose the data - depending on how you handle the error - and you will certainly have a tidy up task afterwards to find and update the records with missing data.
- If the database goes down - you lose the data.

Instead of doing all this in-process, just let the webserver grab the data and write it to a queue. This is a tiny amount of work and even a very small webserver can handle a very large amount of requests like that, so it is unlikely your webserver gets overwhelmed. Now use a different function, such as an Azure Function or an AWS Lambda to process the queue messages. If anything goes wrong, the messages will automatically be retried and eventually stored in a poison queue, giving you time to fix the problem and replay the messages.  
Almost by magic, you have enabled your website to scale to near infinity and have massively increased its reliability.  

But what if the queue goes down? This is a good question and one of the reasons this is a lot harder to do on-premise or when you do everything yourself. Cloud-provided queues, such as Azure's Storage Queues, are extremely resilient and scaleable to the point where you can assume for all practical purposes that they never go down. If you really, really need it, it is pretty simple to set up a fail-over queue in another data centre.

## Database types

Cloud providers offer many different ways to store your data. Not only are there managed versions of many "traditional" databases, you also get such things as Azure Blob Storage, Table Storage, BigTable and many more. These different "databases" have very different characteristics and a sensible mix of storage models can vastly increase the scalability of your solution without costing a fortune.

## Examples

In the video, I give some quick examples of where I have used the principles in real-world systems.

# Conclusion
[NewOrbit](https://www.neworbit.co.uk/azure) is an Azure Gold Partner and Azure Reseller ("Direct CSP") as well as development house. If you would like to buy your Azure from people who design and develop systems on Azure every day, give us a [shout](https://neworbit.co.uk/#contact) or ping me on [Twitter](https://twitter.com/flytzen). We usually give you a "trial", in the form of a Cost, Infrastructure or Security review so you can see if we can help you and if you like working with us.
