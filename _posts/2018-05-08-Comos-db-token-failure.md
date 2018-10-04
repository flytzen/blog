---
layout: post
title: CosmosDB token has wrong time
date: '2018-05-08T12:00:00.000+01:00'
author: Frans Lytzen
tags: Azure CosmosDB
modified_time: '2018-05-08T12:00:00.000+01:00'
excerpt: CosmosDB error "The authorization token is not valid at the current time" and how to fix it.
---
Out of the blue, we started receiving the following error in our web app that uses CosmosDB:

```
The authorization token is not valid at the current time. Please create another token and retry (token start time: Tue, 08 May 2018 09:17:33 GMT, token expiry time: Tue, 08 May 2018 09:32:33 GMT, current server time: Tue, 08 May 2018 09:11:00 GMT
```

The important things to note here are `token start time: Tue, 08 May 2018 09:17:33 GMT` and `current server time: Tue, 08 May 2018 09:11:00 GMT`.  
The token has a start time *in the future*. As a consequence, any request to CosmosDB made with this token fails.

It took us a while to diagnose this because we just assumed that `current server time` referred to the time on the Web server, but it actually refers to the time on the CosmosDB server.  
We could see that the `current server time` was correct, in that it matched the known correct time on other servers.

What it boils down to is that the time on our webserver was wrong, specifically it was in the future, and the .Net client SDK was dutifully creating access tokens that started in the future.

If you are running your app on your own server or a VM, you can stop reading now and go reset the time on your server and all will be well. What was really weird for us was that we are running this in a Web App in Azure App Services. They are supposed to be time synchronised and it simply shouldn't be possible for their clock to drift. Except it did. We triangulated the problem by running the same code from another location and eventually from a web app deployed to a different App Service Plan.  

**If you are experiencing this problem in Azure Web App, the TL;DR; is to scale your site *up*, for example from an S1 to an S2 and then down again later. Details as to why are below.**

The thing to remember here is that an App Service Plan is what encapsulates the underlying hardware that actually hosts the App Services. We had an App Service Plan with a single node in it and *all* Web Apps deployed to this App Service Plan failed with the same error. Which is logical when you think about it, but easy to forget in the heat of battle.  
We confirmed the issue by going to the "Advanced Tools" and, using the Powershell console, ran `Get-Date`, which showed us that the web-server time was wrong.  

In our case we had just a single node in the App Service Plan. If you had multiple nodes and only one had time-drifted, you'd probably see the error intermittently and the `Get-Date` would just return the time from whatever server the console happened to be running on. If you suspect this situation, it may be worthwhile scaling your app service plan down to a single node, testing it and scaling out again.  

What we needed was for Azure to kill our faulty node so we would automatically roll on to another node with the correct time. Unfortunately there are no tools to do this (and you wouldn't expect to have to). However, there is a way you can do it and it was literally the process of writing this post that made me think about it, so I've just stopped writing to go fix the problem; Simply scale your site *up* and then down again. For example, if you are on an S1, scale to an S2 as this will force Azure to deploy your site on a new underlying machine. You can scale back down again when you are happy.
