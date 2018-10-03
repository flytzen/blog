---
layout: post
title: Agile with fixed scope and budget projects
date: '2018-07-02'
author: Frans Lytzen
tags: "Project Management"
modified_time: '2018-07-02'
excerpt: TBD.
---

I think it's pretty clear that using an agile approach for your product management and development is the most cost-effective way to work with software. Of course, "agile" has been become a thoroughly abused term and many things now called agile are anything but. But if you go back to first principles and apply them, you will reap benefits. However, that is now what I want to talk about today. Instead I want to talk about how you can use some of the agile principles even in a fixed-scope/fixed-budget project. There are many reasons why a business wants to define a fixed outcome and budget for a software project up front. A few of the reasons are even good reasons. What and why those reasons are is a subject for another post - this post is about how you can "be as agile as you can" in that situation. At [NewOrbit](https://neworbit.co.uk) we work in this mode quite a lot and have continually been refining our approach over the years; I am looking to share our current approach in this post. I am writing this post from the point of view of an external consultancy, but the principles apply just the same to an internal team that are asked to commit to a fixed scope and budget.

TBD: cartoon from commitstrip.

# What does a fixed-scope project look like?
Given that the client wants a defined scope and a fixed price, we need to look at how to define both of those in a way that avoids us having to write a detailed specification up front. 

## Defining the scope
As far as the *scope* is concerned, the thing is to define this in terms of *outcomes*. I like the Actor/Goal list for this purpose; you list out the actors who need to interact with the system and list out their goals in terms of what they need to be able to achieve when you are done. This is a pretty good way to manage scope as it is usually pretty clear when a new requirement comes up as to whether it was on that list or not. What you *do* have to manage in another way is what we call Fidelity; i.e. how "nice" or how "well" can the user achieve their goal? Does it take one click or ten? That is harder to quantify. I recommend two parallel approaches;
1. Define the *business objectives* with the client separate from the actor/goal list. For example, if the business objective is to save money by automation, it tells you a lot about how manual a process in the system is allowed to be. If the objective is to wow external users with a slick experience then that gives you some constraints. The targeted volume also tells you a lot about how you need to develop the system; If you are dealing with 10 cases a day, for example, you can expect the human user to be able to remember things and reason about things, whereas if you are dealing with 10,000 per day then the system has to manage everything. And so on and so forth. As a side-note, it is often surprisingly hard to get the business objectives from the stakeholders; they often don't really know - and if they don't know, you can't know and then your project is guaranteed to fail as you can't reach a destination unless you know what it is. So, absolutely insist on getting the highest-level business objectives written down clearly. Go higher up the chain if needed.
2. Have a frank conversation about fidelity; Explain to your stakeholders how expensive high fidelity is and have frank conversations about what level they need and want. One of my favorite examples is to turn my laptop over and show them the grilles, screws and labels on the bottom of my run-of-the mill laptop and then explain that Apple spent millions to make the bottom of Mac laptops smooth. Talk about levels of fidelity and different users and what fidelity is required for different audiences. Make sure you share this with everyone on the team.

Another way of saying this, is; Define the *why* (the business objectives) and the *what* (the actor/goal list) up front, but leave as much of the *how* as possible to the project proper.

In addition to the idealised version above, you should of course sniff out any areas you think have *risk* around them and do more work on spec'ing them up front; There is nothing inherently wrong with doing detailed specifications on some things in order to reduce risk - just use this tool wisely. I *do* recommend insisting on simple POCs for any integration; it's very easy to get burnt by assuming an API you have to talk to is sensible when really it is a piece of crap that you have to spend weeks coding around. Insisting on spending a day or two writing a simple hello-world app to talk to the API will you a lot more about the risk than any glossy brochure or API documentation. 

## Defining the budget
Whatever you do, *don't* commit to a single price, instead insist on giving a *range*. At first, that doesn't seem to make a lot of sense; after all, if you give a fixed price and you come under, you get to keep the remainder, right? Whereas, with a range, the client gets to keep any saving. Talk about asymmetric risk.  
But... when you have a fixed price, any conversation with the client about whether something is really necessary or not will always end with "yes" - because the client will only ever pay a single amount, no matter what you have to deliver.  
When you have a *range*, it is in the client's interest to not do something - or do it with less fidelity - as this will reduce the final bill. Even when your counter-part doesn't directly care about the money they usually care about changes: Any fixed-scope project will have changes and edge-cases where you disagree about the extent of a certain requirement. When you and the client agree to not do something, it moves the cost down in the range, potentially freeing up money to do other things your counterpart at the client cares about (you should still manage these in as changes, of course, but it's a conversation you can have).



# What you can't do
..loses business agility. Gist..
..It also means that owning the budget is a necessary and inherent in everything you do, all the way down. Every person on the team needs to be aware of the budget item and need to own estimates, trying to do the work within the constraints and flagging up early if it's going to take longer. It's uncomfortable and most people just want to do the right thing and build a good product - but with a fixed budget, every person has to work to a budget all the time. Incidentally, in a git comore agile scenario this is generally dealt with by breaking things into much smaller deliverables, thus pushing the control back to the product owner/client.


..**throughput is now a primary concern** - as is budget control. "loweest level of fidelity possible"

# Sprint 0

# Requirements Maturity
..GEIGE. 
..you can deliver this in half the time or twice the time - manage this by having ownership all the way down.

# Velocity

# Make the iterations matter
..goals
..laps

# Developer ergonomics

..diff to technical debt

# Control/exception handling
..when the breakdown noticeably changes the initial estimates. But understand that restructuring is natural and the structure you use for initial project estimating and choosing will be different to how you want to actually deliver it.

# Tools
- zero bugs in the sprint is not optimal

# Notes
- Very high overhead managing small changes into a fixed project. Momenta example.
- Range quotes
- Change management