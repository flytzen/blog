---
layout: post
title: Find SELECT N+1 with Application Insights
date: '2019-05-20'
author: Frans Lytzen
tags: C#
modified_time: '2019-05-20'
excerpt: With ORMs it's very easy to write code that calls SQL many times to serve a single web requests, and it can be hard to find it. Application Insights can help.
---
ORMs like Entity Framework are good for many things - but they also make it easy to write "SELECT N+1 queries". This is when you thought you just did one SQL call but you make one call to retrieve the list of records and then (at least) one more call for each row in the resultset, because you loop over the data and inadvertently auto-expand a property. You rarely spot it in development and even in production it is sometimes hard to see; It may well be that a particular operation causes your code to call SQL 500 times, but that may take less than a second so the operation may not even show up at the top of your "performance offenders" list. But, it's still a lot of traffic to send to SQL and, combined with the fact that SELECT N+1 problems are usually very easy to fix, it's worthwhile hunting them down and sorting them out.

By default Application Insights log every Request made to your web/api server and log every *dependency call*, such as database and micro service call you do. It connects the two together by giving the dependency call a `operation_ParentId` that is the same as the `id` of the request.
In short, you can join the `requests` and `dependencies` to list each web/api request and show how many SQL calls it generated and how long they took to run in aggregate like this:

```
requests 
| project name, id, duration
| join kind = leftouter (
   dependencies 
   | where type  == 'SQL'
   | project sqlid = id, operation_ParentId , sqlduration = duration 
) on $left.id == $right.operation_ParentId 
| summarize sqlopscount = countif(isnotempty(sqlid)), sqlopsduration = sum(sqlduration) by name, id, duration
| order by sqlopscount  desc 
```

Note that this query will show each individual request made. You may want to roll that up further to see the *type* of call - but I find that quite often it is helpful to know the exact call to make it easier to reproduce it.  
I am currently working through a legacy app that has a lot of these kind of problems; I look at the result of this query and then set to work fixing them. It's a very easy way to find major causes of stress on the database.

### CosmosDb
You can use the same query, just change `type` to "Azure DocumentDB".

### What about other dependencies - or if my SQL calls are not logged?
When running ASP.Net (Core or otherwise) on Azure Web Apps, your SQL calls are automatically logged as dependencies. In other scenarios, this may not happen automatically. You may need to install certain extensions to pick them up - or you may have to write your own. Incidentally, writing your own can also be very helpful if you want to do the same statistics for something other than SQL.  
In short, you can write code to wrap dependency operations in calls to `TelemetryClient.StartOperation` and `TelemetryClient.StopOperation` - see the [Telemetry Client documentation](https://docs.microsoft.com/en-us/azure/azure-monitor/app/custom-operations-tracking). 
