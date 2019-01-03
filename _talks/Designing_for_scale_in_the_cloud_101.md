---
layout: talksummary
title: Designing for scale in the cloud 101
tags: Azure
excerpt: A quick overview of some of the core concepts you should consider when designing applications for the cloud
---
When thinking about developing an application for deployment in the cloud, be it Azure, AWS or Google, most newcomers fall into one of two camps;
- Just do what you normally do because the cloud will handle all scaling concerns.
- Re-think everything and learn all the latest buzz so your site can scale to a quintillion users.

For most people, most of the time, both of these statements are untrue. 99% of sites and web apps do not need to scale to thousands of concurrent users and rarely need the more complicated aspects of designing for large scale. At the same time, the cloud really *won't* just magically scale up in response to demand. 

In this talk I give an overview of a couple of different design principles you should adopt when you develop anything for the cloud. They are easy to incorporate into your development (they will actually make your code easier to deal with) by leveraging a few of the core infrastructure components provided by most cloud providers. I also give an overview of a number of real-world software projects that have used these principles to great effect.

The talk is language agnostic. I do use Azure for examples, but the principles apply equally to all the cloud providers.
