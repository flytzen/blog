---
layout: talksummary
title: C# Async Deep Dive
tags: C#
excerpt: Everything I thought I knew about async in C# was wrong
sequence: 30
---
This is not yet another session on how to use async/await. You are probably already using that, so we won't bore each other with the syntax. Instead, we shall be diving deep into how Async actually works and having a look at the benefits and pitfalls.  

The way Async is being described, it sounds like it will make your code faster and more scalable whilst solving all your problems and achieving world peace - all before lunch.  

Async certainly can help you do more I/O in parallel and may, in some circumstances, help you scale. But did you know Async code can sometimes also use more memory, make your code slower and can introduce subtle bugs that may only appear in production?  

Understanding how Async in C# works under the covers is crucial to be able to harness the benefits it can give you whilst avoiding the pitfalls. This session aims to give you that understanding.

Note that this talk will not show the generated IL; It focuses on the principles and uses pseudo-code to explain what is generated.

So far, I have delivered the talk at [Dotnet Oxford](https://www.meetup.com/dotnetoxford/) and [DDD South West](https://dddsouthwest.com/) - thanks for having me!

The presentation and the code samples [are on GitHub](https://github.com/flytzen/Async.Presentation) and I have also recorded it as a video - see my [related blogpost]({% post_url 2019-04-29-Everything-I-thought-I-knew-about-async-was-wrong %}).
