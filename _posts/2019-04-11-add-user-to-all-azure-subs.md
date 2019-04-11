---
layout: post
title: Add new user to all my Azure subscriptions
date: '2019-04-11'
author: Frans Lytzen
tags: DevOps
modified_time: '2019-04-11'
excerpt: Add a new user to all my Azure subscriptions using Azure CLI
---
I found myself needing to add the same user as an owner to all my Azure subscriptions. If I could ever retain Powershell syntax, I could write a clever script that would loop through all the subscriptions and do what was needed. Alas, that forever eludes me so I just opted for a few Azure CLI commands with a bit of editing in an editor. Still much faster than going throgh the UI repeatedly.

## Multiple tenants
For most people, this doesn't apply so feel free to skip this section if it is irrelevant.  
When you log in with Azure CLI, you can *see* all your subscriptions across tenants - but the command to add a user to a subscription only works for the tenant you are currently "logged in to". It can be confusing.  
To see the tenant IDs for all your subscriptions you can run this:
```
 az account list --query "[].{tenantId:tenantId, name:name} | sort_by([],&tenantId)"  --output table
```

In order to "login" to a specific tenant you need to :
```
az login --tenant <tenant id>
```
 Then do the stuff below for each tenant id

## Getting the list of subscription IDs
First, get the list of subscription ids for a given tenant id:
```
az account list --query "[] | [?tenantId == 'foo.onmicrosoft.com'].id" --output table
```
Note that the tenant id may be in this form or may be a guid - not sure why.  
Alternatively, if you only have a single tenant, just do:
```
az account list --query "[].id" --output table
```

## Add the user to all the subscriptions

Copy the list of subscription IDs into a text editor and modify each line to look like this (111... being the subscription id):
```
az role assignment create --role "Owner" --assignee foo@bar.com --scope /subscriptions/11111111-1111-1111-1111-111111111111
```

Copy those lines into your console and shortly after your new user will be an owner of all your subscriptions.
