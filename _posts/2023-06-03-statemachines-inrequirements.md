---
layout: post
title: State Machine Diagrams in \Requirements
date: '2023-06-03'
author: Frans Lytzen
tags: Requirements
excerpt: State Machine Diagrams can - sometimes - be helpful when working through requirements with clients
---
State Machines can be really useful when modelling certain behaviours in software. State Machine Diagrams tend to be technical and the documentation on how to write them can be quite theoretical.

So, can they be useful when working with end users to understand their requirements?

Honestly, not very often. In most cases you are better off using other tools. 

However, there are scenarios where a state machine diagram is the perfect tool. This is usually when you have a long-running process, especially ones where you can sometimes go back or you have various different kind of exceptions.  
The example that prompted this post was an absence process for a client. An employee reports an absence, then when they return they have to fill in a questionnaire and the manager then has to reviw this and close the absence. Sounds simple enough. But, often, the employee changes the expected end date, sometimes they forget to fill in the questionnarie, sometimes the manager forgets to to do their part, sometimes the absence is cancelled. So we need various chasers at different times and various kind of exception handling for edge cases. When you try to model this with a standard flowchart or a swim-lane diagram it quickly becomes a mess. Worse, those tools won't make it easy to spot scenarios you haven't thought about.

Flowcharts and swimlane diagrams generally try to model the *actions* users and systems take: A box on the digram generally describes what someone *does*. That's not terribly helpful in a long-running process where no-one does anything the majority of the time.

A State Machine Diagram works the other way: A box on a diagram generally represents a "resting state", such as "absence is open", "waiting for the employee to fill in their questionnaire" etc. It also shows clearly which actions are *legal* at any time and shows the transitions: For example, the action "fill in questionnaire" is only legal when the absence is currently "waiting for the employee to fill in their questionnaire" and when that action is taken, the absence moves to "waiting for manager to close". In addition, you can show what happens when you enter and exit certain states, such as "send an email to the manager when the absence is opened" etc. 

Another benefit is that the state machine diagram is mostly concerned to what happens to the absence, not who does it or how they do it. For example, in the absence example, several different actors are able to change the expected end date using several different processes. For the State Machine Diagram, it doesn't (usually) matter who does it or how - it's just one action. This strips away a lot of the complexity of flowcharts or swimlanes to allow you to see what really matters at the core of the process. Of course, you do need to capture the "how" somwhere else.

Many explanations of State Machine Diagrams are quite theoretical. You can take that with a pinch of salt - after all, you are just using it (here) to model a process so you can discuss it with users - you don't need to worry about adhering to the rules, just do what works.

## State Machine Basics
The most banal example is a door. It can either be in a "closed" state or in an "open" state. Two states. When the door is open, a person can close it and vice versa.

DIAGRAM

Now the user might say that, actually, you can lock the door - but only if it is already closed. You can then add that in:

DIAGRAM

Now the user wants us to notify a logging system when the door is opened or closed.

DIAGRAM

It turns out that the door will automatically lock at 22:00 and cannot be manually unlocked until 6:00 - so now we can add time based triggers and guard clauses.

DIAGRAM

... did you spot the problem if the door is not closed at 22:00?

There are only a few more things you can do with state machine diagrams, so you pretty much know the now. One useful thing is to have "sub diagrams". To somewhat stretch the example before, we may want to raise an alert every ten minutes if the door is left open. 

DIAGRAM

## References