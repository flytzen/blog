---
layout: post
title: Hierarchy IDs for Fun and Performance
date: '2018-06-21'
author: Frans Lytzen
tags: databases, performance, sql, cosmosdb
modified_time: '2018-06-21'
excerpt: Storing hierarchies in a database is easy - but applying hierarchical security and configuration can be very difficult and a significant performance problem. Hierarchy IDs can alleviate this, at the cost of a bit more complexity at write time.
---
In many of the systems we build, there is a *hierarchy* in the data. This may be an organisation hierarchy, or maybe a hierarchy caused by a multi-tenant system or a combination thereof.  
It's relatively easy to model a hierarchy in a relational or document database - it is much harder to effectively filter which part of the tree a given user can see. This post explores the use of Hierarchy IDs to make this filtering easy and performant. In addition, it has some pointers about how to set up hiearchical configuration using the same Hierarchy ID you are implementing anyway. Of course, you could just use a graph database and then it's a different story altogether.  

A multi-tenant hierarchy may look like this:

<div style="width: 480px; height: 360px; margin: 10px; position: relative;"><iframe allowfullscreen frameborder="0" style="width:480px; height:360px" src="https://www.lucidchart.com/documents/embeddedchart/dbfcc046-5651-4214-ac3f-00e74ac15477" id="eyBOsPLtpVDp"></iframe></div>

An infinite organisational hierarchy may look like this;
<div style="width: 480px; height: 360px; margin: 10px; position: relative;"><iframe allowfullscreen frameborder="0" style="width:480px; height:360px" src="https://www.lucidchart.com/documents/embeddedchart/9013e9ab-5de9-4702-8803-7cb90a31cb60" id="hzBOAr28wlCI"></iframe></div>

At [NewOrbit](https://neworbit.co.uk) we sometimes build multi-tenanted systems which has resellers, who have system customers, who in turn have organisational hierarchies so it quickly become a very deep hierarchy.  
Some hierarchies have a fixed depth and structure, others have infinite depth (though you'd often want some sense check on the depth). Yet others branch.

When you are creating your data structure, whether in a relational database such as Azure SQL or a document database such as Mongo or CosmosDb you will typically have a Parent ID on records in the hierarchy as illustrated above. This is easy at write time and makes it relatively easy to traverse the hiearchy both up and down. 
However, when you need to get access to a particular part of the tree or indeed multiple subsets of the tree then it becomes very hard to query efficiently and performantly.  

As an example with the infinite hierarchy, imagine in the illustration above that a given user is allowed to see the details of anybody who is Bob's hierachy; you'd need to write a recursive function to keep drilling down the layers until there are no more subordinates. That means running many different SQL queries or using UDFs (which in turn will run a recursive function).

Alternatively in the fixed hierarchy, imagine if a given user has access to see all the projects for three seperate branches and they want to see a list of all tasks across all projects they have access to. Or imagine that a Reseller user has access to see all the details for all their Customers, but not for any other Customer - and they want to see a list of all Projects (okay, maybe not the best example in the world, but you get the idea). In that scenario you'd probably write some code that JOINs all the way up the hierarchy and out to various permissions tables, such as "are they in the list that can see the project or in the list that can see the branch or in the list that can see the customer or in the list that can see the reseller". It may not be that hard to write the code, but it's easy to end up with a 20-table `JOIN`, which is expensive on a relational database - and impossible if you use CosmosSB which does not support `JOIN`s.

In this post I am primarily focusing on how to *filter* the data so a user can only see the data they are allowed in an efficient manner. There are other hierarchy scenarios, in particular around set based operations and there are other patterns that are better suited to those than what I am showing here. I highly recommend [Joe Celko's Trees and Hierarchies in SQL for Smarties](https://www.amazon.co.uk/Hierarchies-Smarties-Kaufmann-Management-Systems/dp/0123877334) for understanding more about this. In fact, the lessons that I am expounding on in this post are based on what I learned from that book many moons ago. Even if you use a document database, it's still a good read to understand the patterns.


# How to store it
As discussed above, you will typically have some kind of Parent ID on each node in the hierarchy (though it's usually called something more meaningful, such a CustomerID or ManagerID). 
The trick to efficient querying is to maintain a *Hierarchy ID* on each record that has all the parent IDs all the way up the tree.  
In a relational database, you would store it as a text string like this (for the Task in the fixed hierarchy example above);
10/20/57/2/1047.

In a Document Database you can do the same or use a slightly more efficient approach, which is to store an `array` of Ancestor IDs. If your documents use `GUID`s for IDs, then you can just put all the Ancestor IDs in the array. Otherwise (and only in the fixed hierarchy) you could prefix each ID with it's record type - but that starts making it too blurry in my opinion.

SQL Server has a native Hierarchy ID data type that gives you a few convenience functions, but it is not mapped in Entity Framework so if you use that you may prefer to just use a normal string.

# Querying
In order to query the database you need to 



..list of allowed ids..
  JOIN or Claims.
  Document database challenge



# Mutability of the hierarchy

# Ease of Writing vs ease of Reading
In the suggestions I am making here, you are making it more complex to *write* data; Whenever you add a node in the hierarchy you now have to update it's Hierarchy ID and not just set its Parent ID. If your tree is mutable, you also need to handle updating the Hierarchy IDs of all children whenever nodes are moved - something that can take considerable time and capacity if a high-level node is moved. Similarly, you may need to write code to maintain a list of "allowed hierarchy IDs" per user.  In other words, Hierachy IDs add noticeable extra complexity to your system.  
On the other hand, once Hierarchy IDs are in place, your data filtering/security code becomes much easier to write and your database queries will be much more performant.  

Whether Hierarchy IDs are right for a given solution will, as always, depend on the needs of that system. Some of the key indicators that you may need it are;  
- Deep hierarchies with access rights determined at multiple levels
- Large data sets in a hierarchy
- Infinite hierarchies with access rights set at arbitrary levels (think pretty much any organisational hierarchy with scope-of-control security).


# Hierarchical Configuration
