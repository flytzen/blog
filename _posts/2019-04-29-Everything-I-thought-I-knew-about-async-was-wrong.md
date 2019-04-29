---
layout: post
title: Everything I thought I knew about Async in C# was wrong
date: '2019-04-29'
author: Frans Lytzen
tags: C#
modified_time: '2019-04-29'
excerpt: The way Async is being described, it sounds like it will make your code faster and more scalable, whilst solving all your problems and achieving world peace - all before lunch. Async certainly can help you do more I/O in parallel and may in some circumstances help you scale. But did you know Async code can sometimes also use more memory, make your code slower and can introduce subtle bugs that may only appear in production?
---
When I first agreed to do a [talk](/talks/csharp_async_deep_dive) about how Async in C# really works, I thought I was an expert. I had written lots of high-throughput, high-performance async code so it should be really easy to write this talk.

As soon as I began writing the code to prove the things I knew - it turned out that I knew nothing at all and all my assumptions were wrong. The result was a much better talk where I describe the fundamentals of how Async in C# works and explain a lot of the misconceptions people, such as myself, have. So far I have delivered the talk to my colleagues at work, at [Dotnet Oxford](https://www.meetup.com/dotnetoxford/) and at [DDD South West](https://dddsouthwest.com/) - thanks for having me!

The presentation and the code samples [are on GitHub](https://github.com/flytzen/Async.Presentation) and I have also recorded it as a video. For practical reasons, the video was recorded in five parts; you can see them all [in one playlist](https://www.youtube.com/watch?v=UzVMzBEpuJg&list=PLbwbf2ZiIT4N7G08GcCd1Z64N8tfgKfIn) or see the individual parts below.

## Part 1 - Introduction
There are lots of reasons to use async - some of them are true, others are not. In the introduction we have a quick look at these.

<iframe width="560" height="315" src="https://www.youtube.com/embed/UzVMzBEpuJg" frameborder="0" allow="accelerometer; autoplay; encrypted-media; gyroscope; picture-in-picture" allowfullscreen></iframe>

## Part 2 - Brain Teasers
A quick run through some code examples that may give you different results than you expect. It sets the stage for the next video about how it works.

<iframe width="560" height="315" src="https://www.youtube.com/embed/s4cvgyZ0kUM" frameborder="0" allow="accelerometer; autoplay; encrypted-media; gyroscope; picture-in-picture" allowfullscreen></iframe>

## Part 3 - How it works
The meat of the presentation and by far the longest video at nearly 20 minutes. It goes into detail about what really happens when you run async code and explains where threads come into play.

<iframe width="560" height="315" src="https://www.youtube.com/embed/RBFJoPbbvTk" frameborder="0" allow="accelerometer; autoplay; encrypted-media; gyroscope; picture-in-picture" allowfullscreen></iframe>

## Part 4 - Deadlocks
If you work with ASP.Net "old" (i.e. not Core) or you write Desktop apps then you have probably heard that using `.Wait` or `.Result` can result in deadlocks. And you may have experienced that some code will only deadlock sometimes. This video builds on part 3 to explain why the deadlocks occur. It does primarily focus on ASP.Net.

<iframe width="560" height="315" src="https://www.youtube.com/embed/eRuGnEAya8M" frameborder="0" allow="accelerometer; autoplay; encrypted-media; gyroscope; picture-in-picture" allowfullscreen></iframe>

## Part 5 - Tips
To sum up I have collected some tips that you can use, including some on how to utilise Async to easily parallelise network writes and other async work in a way that can speed up the throughput of your code very substantially. After the last time I gave this talk, [Tom Robinson](https://twitter.com/tjrobinson) kindly pointed me to [these comments](https://github.com/davidfowl/AspNetCoreDiagnosticScenarios/blob/master/AsyncGuidance.md#prefer-asyncawait-over-directly-returning-task) by [David Fowler](https://twitter.com/davidfowl) which put a bit more nuance around my tip to "just return the task"; consider your circumstances and make the choices that are right for you.

<iframe width="560" height="315" src="https://www.youtube.com/embed/UtF_0gfZ48Y" frameborder="0" allow="accelerometer; autoplay; encrypted-media; gyroscope; picture-in-picture" allowfullscreen></iframe>
