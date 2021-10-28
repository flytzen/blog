---
layout: talksummary
title: Real-world C# Performance
tags: C#
excerpt: How to adopt a set of habits that will dramatically improve the user-experienced performance of your C# code.
sequence: 15
---
The purpose of this talk is to give you the pointers, understanding and, above all, good habits to do *enough* performance optimisation in your daily work – not to highly tune a performance-critical system.

It is great fun to look at how you can shave a handful of micro-seconds of a critical operation on a hot path in C#. Sometimes you need that, but for most code-paths in most systems, it is not terribly important. User-experienced performance is usually determined by I/O - i.e. the amount of time your system spends talking to external services, file systems and databases.

For example, running a foreach loop over a 10k item Array&lt;T&gt; is about 40 micro-seconds faster than looping over a List&lt;T&gt;. Each and every database call you make is likely to take more than 10,000 microseconds (and usually significantly more than that) - so you have to change an awful lot of Lists to Arrays to save as much as time as eliminating a single database call. Even a call to a distributed cache is likely to take a few thousand micro-seconds.

In this talk we will look at;

- What do the performance numbers really mean.
- How to find out what external calls you are making.
- How to eliminate, optimise and parallelise external calls.
- Understand the common mistakes and patterns that cause your system to make - and wait for - many more external calls than it needs to.
- How to build good habits that stop you from making mistakes that will hurt user performance.

The focus in this talk is the performance experienced by users. This is distinct from the problem of massively scaling up a system or being able to deal with a very high volume of processing, where the aggregate compute time impacts scaling.  

For clarity, this talk is focused on C#, which includes web and API servers as well as desktop applications. But, it does not cover website optimisation in the browser itself.
