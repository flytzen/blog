---
layout: post
title: Spend less on Azure
date: '2022-03-24'
author: Frans Lytzen
tags: Azure
excerpt: It is estimated that 30% of cloud spend is wasted. How do you avoid wasting 30% of your Azure spend? This talk takes you through the approach NewOrbit takes to analyse and optimise Azure costs.
---
One of the things our clients often ask us to do is to review their Azure spend to see if they can save costs. It is common for us to identify potential cost savings of between 30% and 70%.

I gave a talk at [.Net Oxford](https://www.dotnetoxford.com/posts/2022-03-OptimiseYourAzureSpends) in March 2022 that goes through the methodology we use when doing [cost reviews](https://neworbit.co.uk/azure/#need-help).

# Video

<iframe width="560" height="315" src="https://www.youtube.com/embed/s0v-2lf3URs" title="YouTube video player" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture" allowfullscreen></iframe>

# Talk Summary

## Where is the money going

The built-in Azure Cost Analysis tool is a really good way to understand where your actual spend is. But, it does require some getting used to.  
We follow a 3-step approach.

For each step:

- Change the time period to the previous month.
- Change the Granularity to None
- Change the display to Column (Grouped)

### 1. Understand the big picture

Change "Group By" to "Service Family" to get an idea about the split between Compute, Database and so on. Then change it to "Service Type" to get a slightly more detailed view of the split.

The main thing here is to understand where most of your money is spent. There is little point in trying to reduce a £50 to a £40 spend, if you have another spend that is £5,000, which you can reduce to £4,000. Focus your energy on the areas that are most expensive.

### 2. Which resource is it?

Once you have identified which service types are most expensive, it's time to find out which instance it is. 

For that purpose, first add a filter for that particular service type and then Group By "Resource". "Resource" means "Resource Name", and will allow you to identify the specific instances that are most expensive.

### 3. What part of the service is it that is costing?

Most services in Azure have costs that are made up of multiple sub-parts, such as bandwidth, storage, etc. It is important to understand exactly which bit it is that is expensive - you will often be surprised, so don't make assumptions.

First, add a filter for the specific resource you are looking at. Then try grouping by Meter, Meter Category and Meter Subcategory. There seems to be little consistency that I can see to what hides under each of these, so try them all in turn. This often leads to surprises about what it *actually* is that is so expensive.

## Checklist

For each resource you want to reduce the cost for, this is the general checklist we go through (see the video for much more detail):

1. Is the service the right size? Have you over-provisioned? Are you using too high a service tier or redundancy level?
2. Is this the best service for what you are doing? Azure has many different services for each type of thing you want - and they often differ wildly in price. You can often save substantial sums by simply changing from one service type to another.
3. Do you use the correct scale? Could you use scheduled, automatic or manual scaling to reduce costs?
4. Could you share resources? There are many services in Azure where multiple service instances can share one set of compute and thus costs.
5. Are you using Infrastructure-as-a-Service rather than Platform-as-a-Service? IaaS is *usually* more expensive, not least because you usually need two instances to get the SLA, whereas with *most* PaaS you only need one instance.
6. Do you have services that are not actually in use? Demo or test systems that are kept running even when not used? Or services that were spun up and since abandoned?
7. Could you benefit from Reserved Instances to get a discount from Azure?
8. Are you in the right location? Some services are more expensive in some data centres.

## Architecture
We recommend four general principles for an architecture that reduces cost by design. The first two are the same that we recommend to achieve [scale and resilience in the cloud]({% post_url 2022-02-20-designing-for-scale-in-the-cloud %}), so you get two benefits for the price of one.

1. Use Queues. Without queues, you have to always provision enough capacity to deal with the highest potential spike in traffic. Queues give you a buffer so you can run with much lower provisioned capacity; spikes in traffic are soaked up in the queues.  
2. Use appropriate databases and storage options. Azure has many different ways to store data, including a wide variety of databases and things like Storage, Search and more. Moving some types of data to another storage type can sometimes reduce cost by 99% (yes, really).
3. Scale *out* rather than *up*. Azure inherently prefers many small workloads rather than a few large workloads and will make it easier for you if you follow that paradigm. It gives you much more granularity in terms of scaling and may also give you absolute cost savings.
4. Avoid Virtual Machines as far as possible. Use PaaS or containers instead.

## Chattiness

One of the most common problems we find when doing cost reviews is that the system is too chatty. Usually, this means that the application makes way too many calls to the database backend, though sometimes it is the web front-end that makes too many calls to the API server. In either case, this causes excessive load on both front and backend, which means higher costs. Most developers are surprised when we show them this - you may be surprised too.  

See our [post on how to detect this]({% post_url 2019-05-20-find-select-nplus1-with-app-insights %}) to get a simple Application Insights query you can run to check it. You might be able to save quite a bit on your database by tidying up a bit of code or introducing a tiny bit of memory cache.

# Conclusion

[NewOrbit](https://www.neworbit.co.uk/azure) is an Azure Gold Partner and Azure Reseller ("Direct CSP") as well as development house. If you would like to buy your Azure from people who design and develop systems on Azure every day, give us a [shout](https://neworbit.co.uk/#contact) or ping me on [Twitter](https://twitter.com/flytzen). We usually give you a "trial", in the form of a Cost, Infrastructure or Security review so you can see if we can help you and if you like working with us.
