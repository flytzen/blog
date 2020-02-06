---
layout: post
title: AI will fundamentally change how we build software
date: '2019-08-14'
author: Frans Lytzen
tags: Technology AI Azure
modified_time: '2019-08-14'
excerpt: AI will change how we think about, design and build software. It's a revolution on par with the move to web.
---
Artificial Intelligence/Machine Learning/bots are set to fundamentally change how we design and build software - even how we think about the problems we create software to solve. 

I believe it's a revolution on par with the move to web and I think many people working in the industry are yet to fully realise this. If you are a business analyst, architect, developer or even a business process specialist, your world is about to be turned upside down and you will have to learn a completely new skill-set or face becoming increasingly irrelevant. 

Update: I turned this article into a 10-minute lightning talk and delivered it at .Net Oxford in December 2019:
<iframe width="560" height="315" src="https://www.youtube.com/embed/4HeRqsIf-jA" frameborder="0" allow="accelerometer; autoplay; encrypted-media; gyroscope; picture-in-picture" allowfullscreen></iframe>

You are quite likely to think I am being alarmist and you may want to dismiss what I'm saying. These could be some of the things you want to tell yourself:

- A business process is a business process.   
*No, it's not. AI allows many business process to be completely re-defined or eliminated.*  

- I still need to build - and know how to build - all the resilience, scale, micro services, databases and so on, all that AI stuff doesn't remove the need for that.  
*True - but it's no longer enough and you may find yourself being really good at solving the wrong problem.*

- Real people don't want to [make some change you can't imagine should change].  
*Except, people do change - and more and more rapidly*.


I like to think of this as more of an opportunity than a threat. But, it does require you to accept you are going to be a beginner again, for a while, allow yourself to learn and to challenge everything you already know. I am on this journey myself and I don't know the answers yet - but I am enjoying exploring.

Note: I am talking somewhat interchangeably about Artificial Intelligence, Machine Learning and bots. They are not the same and I am not even going to try to get the categories right, because that is not the point of this post; I am just going to talk about the impact "all of that stuff" could have on how we build software. 

# Automating routine tasks
The most obvious thing that springs to mind when you talk about things like Machine Learning is to automate routine tasks. The ubiquitous example is image recognition such as recognising hand writing or categorising pictures of animals into species. 

[![Picure of photos classified into animal species](/assets/ClassifyPictures.png)](https://towardsdatascience.com/deep-learning-for-image-classification-why-its-challenging-where-we-ve-been-and-what-s-next-93b56948fcef)


Maybe more relevant to people likely to read this post is things like assessing the contents of online forms to decide on actions.  
For example, today you may have people submitting an annual health survey and have a health practitioner review these questionnaires to assess who is fine, who should receive a phone call, who should have a face-to-face visit and so on. You *can* automate that today using hard-coded rules, but it's probably a lot easier to analyse what the health practitioner actually decides and then train a Machine Learning model on that data.  

In practical teams, there are a few things you can do now to start this journey;
- Understand that it's not about getting to 100% accuracy. In most cases, Machine Learning models will spit out a **probability** so you can put a rule in to say that when that probability is less than, say, 90% then this thing is referred to a human being for assessment.   
This data can then be used to further refine the model, making it ever more accurate (you should also randomly refer some high-probability cases to a human on an ongoing basis to check and adjust the model over time).  

- Think about setting your software up so you can start gathering the data; without a training set, you can't train the model in the first place. So build the system so 100% of cases go to a human for now and design it so the *decision* is clearly recorded along with the original input.   
That thing about recording the decision clearly is very often overlooked when you are just focused on the process and you are not thinking about the future potential for using Machine Learning.
Once you have had some data through, you can start training a model - and the UI you built stays relevant for dealing with the edge cases. 

# Understanding what is normal
A much more subtle - and potentially pervasive - application is when a system starts to learn what's "normal" and start to react on its own when things are "not normal".  

One of the best examples at the moment is [Azure Application Insights](https://docs.microsoft.com/en-us/azure/azure-monitor/app/app-insights-overview). When you add App Insights to your system, it starts recording a lot of information, including requests, dependency calls, errors etc. You can, of course, view this in the traditional way but App Insights will also use Machine Learning to automatically build up a baseline of what's normal and alert you when something is not normal.  
For example, you may get an alert saying "between 10:03 and 10:07 5% of calls to [some dependency] failed - normally only 1% fail". Or similar alerts for duration or request volume etc. The key thing here is that you do not need to set up any special rules to get these alerts, the machine learning model figures it out on its own.
![Example alert from Application Insights](/assets/AppInsightsAutoDetection.png)

Azure also has similar things for security, threat detection and logins, where they establish a baseline for "normal" and alert you to unusual things.

You can use this approach in many different ways in your own software. For example, a known indicator of stress is if someone all of a sudden starts working a lot more hours. So you could use Machine Learning to automatically establish a baseline for "normal" hours *for each individual* in your company (assuming they record timesheets) and flag it up to their manager if all of a sudden their patterns change, giving you a chance to intervene early before a wider mental health issue develops. There are standard tools out there to help you with that, including [Azure's Anomaly Detector](https://azure.microsoft.com/en-gb/services/cognitive-services/anomaly-detector/).

When you are designing systems, these kind of technologies need to be in your mind. Do you rely on someone manually checking for deviations or do you have alerts or hard-coded rules to detect anomalies?

# Make suggestions and correct errors
When you receive a text message on your phone or an IM on LinkedIn, you will often be given some suggested answers; These are based on Machine Learning to analyse what the message sent to you was about and what people normally respond with in similar situations.

![Example of LinkedIn suggesting replies](/assets/LinkedInIM.jpg)

You may have similar things in a UI where, for example, the system learns over time what kind and level of expenses a person normally records and alerts them if they enter something that is unusual for them. These are not system-wide rules, simply knowing that it's normal for a particular individual to claim £50 for train tickets, but not £500. Or maybe that when you record mileage, more often than not it's to a particular location so it can auto-suggest the description and distance.  

The list of examples get longer the more you allow yourself to think about it, starting with detecting deviations from normal to understanding what you probably meant and suggesting that. [Google's Autodraw experiment](https://www.autodraw.com/) is an amazing example of this.

When you are thinking about both UX and business processes - these kind of technologies need to be in your mind. Are there places where users make mistakes and we can help them not to? Are there whole new categories of problems we simply haven't been able to solve before?
One thing's for sure, this is becoming the new normal so you need to incorporate it in your design thinking.

# Intent driven UI
I think this is probably the most challenging and different area. 
It's always been the case that good UX starts with *intent*. You need to know what the user wants to achieve and then design the UX to make it easy to get there. With things like bots, we now often use natural language processing to derive intent from normal speech. I see many/most software interfaces for *occasional* users needing to adopt this kind of conversational UI. But AI is not magic and making this work well is really hard.

Let me use an example to illustrate what I mean.

When you want to book a holiday you can probably describe it really easily to another person. Maybe "I just need to get away for a week, somewhere quiet where the kids have stuff to do. Just to get some rest and not have to think too much. It must not be too expensive, but it does need to be of a decent quality". If you said that to an experienced travel agent, they may ask you a few follow-up questions, and would then quickly be able to suggest a few options for you. How you respond to those options (too hot, too far) would help them to better understand your needs and they would refine their suggestions. In the end, they would probably give you a list of only a handful of suggestions, even if there may be 100 trips that match your criteria because [choice is overrated](https://www.amazon.com/Paradox-Choice-Barry-Schwartz/dp/0062449923/ref=cm_cr_arp_d_product_top?ie=UTF8).

Now imagine doing that same journey on most travel booking sites that exists today. You will be given a plethora of filters and tick-boxes so you can review every holiday option going. Except, you don't know what half the options really mean except you know that to go somewhere quiet you need to *not* select any of the ones that say "great night-life" or "18-30 special". It usually ends up being an exhaustive and often frustrating journey through a bunch of options, comparing and contrasting with a lot of swearing. 

![Example of a travel booking site](/assets/HolidayBooking.jpg)
(*I do like Crystal's site that I stole this picture from - I think it's one of the better ones out there, but it illustrates the point nicely. Hope they will forgive me.*)

With a more conversational UI and Machine Learning techniques, that journey could be much more like talking to an operator. Understand me right, building that system is **hard**; current Artificial Intelligence is not magic and cannot "think" in any meaningful way. Underneath the covers, that nice conversation with the user still has to be translated into those same tick-boxes, so you can show them filtered results. Natural Language Processing can *help* - but you need to identify all the intents yourself. This is not trivial but I truly believe this is where software design for occasional users is going. I believe we may see a (temporary) upsurge in slightly more hybrid solutions where artificial intelligence techniques are used but with operators being pulled in when the AI does not know what to do. However, the data gathered during those interactions will feed back into the AI model and the need for operator intervention will taper off.

Is the technology and the patterns there to build the dream travel booking site I describe here yet? Probably not. But it's coming and you should start thinking about how you can start gathering the data to train those models; what can you do today to capture information about how your users express their intent?

In this example, what about the travel agent - should they talk to an AI as well? I don't *think* so. After all, we are trying to model the AI on an experienced travel agent and it would probably be counter-productive to have the travel agent interfacing with an inferior emulation of themselves. In other words, there is very much still a place for complicated user interfaces with lots of options for the users who use your system every day - but casual users are going to expect something different.

In practice this isn't just for complicated things like travel agent bookings; it could be as simple as a customer wanting to buy a widget from your online store. At the same time, don't forget that sometimes a user does want to have all the information and not just a part of it, so model that intent into your considerations as well (okay, I am drifting towards general UX design here, sorry).

Building conversational UIs that actually work is really hard - but it is getting easier every day and it will start to become an expectation for more and more people. Just think about how we engage with our various digital assistants. Even the humble Google Search is a lot more than just a "search", you can ask it to translate things or do math amongst many other things.  

When will your customers begin to expect that they can just log in to your portal and ask a question like "when will my t-shirt arrive", rather than having to click on their account, list their orders, click on the right one and go through to "tracking"?


# Re-imagining problems fundamentally
All the above has really been about how we change existing types of software and problems to take advantage of AI to make it better or more efficient. For most of us, that is probably the primary thing we need to worry about because that **isn't going away anytime soon**. 

But there is also another whole class of problems that we can start to solve that simply wasn't possible before.  

I recently spoke to [Salad Money](https://www.saladmoney.co.uk/), a Social Enterprise that provide small, short-term loans. They use Open Banking and have fully automated their loan approval process, making it possible to provide these small, cheap loans cost-effectively and at scale. As they build up more data, they will be able to use Machine Learning to provide more tailored products to individuals, without having complicated or unfair assessment processes. 

This general ability to take what used to be quite complicated things that required expert involvement, automate it and deploy it at scale is something I, personally, am quite excited about and I think we will see a lot of innovation in that space.

# So what do I do now?
I am on the journey myself, I don't know the answers. 
I spent quite a long time over the last few years trying to get my head into how it all really works and I bought several books on the subject because that's how I have always approached learning new technical topics. But most of the books very quickly descend into deep math and most seem intended to help you design Machine Learning algorithms - which is probably not what you want.

Eventually I had a re-think and I decided on a smarter goal for my learning, at least for now:  

**I want to be able to recognise where AI technologies can potentially help or change how I design software**. 

That may sound obvious, but I don't think I have internalised AI enough that I can (yet) instinctively recognise where I should think about AI. 
That is obviously the first step, because once I identify these opportunities, I can then target my learning at something much more specific and/or hire help. 

The best way I can achieve that is by seeing as many examples as I can. That means going to events, reading blog posts and watching videos. I do like to see how these things are put together, but that's not the real purpose yet. I just want to load my brain up with examples so I can start to see the patterns of what is possible and start recognising the situations where I, too, should apply AI.


As a secondary goal, I also want to learn enough about the practical realities that I can at least do a proof-of-concept in order to show that there could be value.  
For example, I have identified some client scenarios that can benefit from automating classification as described above. I can use high-level tools, such as [ML.Nets Model Builder](https://marketplace.visualstudio.com/items?itemName=MLNET.07) to do a proof-of-concept without understanding the difference between a Bayes Classifier and a Logistic Regression algorithm, knowing how to do hyperparameter tuning or even clean my data that much. Would I put the result into production? Of course not. But if I can show a model that has 80% accuracy after tinkering for a couple of hours, I can create the business case for engaging a data scientist to build the real thing. And yes, I am learning how to use [Jupyter Notebooks](https://jupyter.org/) and all that as well, but the truth is that I don't actually need the grown-up tools yet.

At [NewOrbit](https://neworbit.co.uk) we are actively embracing this journey. We are working on training programs for everyone from sales, through business analysts to developers so they can help our customers take advantage of this revolution in software design. We are also building some of our own products in order to test our skills and showcase what's possible (and to have a bit of fun along the way).

Are you learning AI? Do you think this will affect software design as much as I do? Leave a comment with your views or contact me on [Twitter](https://twitter.com/flytzen) - I'd love to hear what you think.