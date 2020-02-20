---
layout: post
title: Why you should use the Azure CLI
date: '2019-03-06'
author: Frans Lytzen
tags: csharp
modified_time: '2019-03-06'
excerpt: TBD.
---
The Azure CLI (todo; link) takes effort to learn and it's not initially obvious why it's worth the effort when you can do things through the portal. Let me try to convince you why you should make yourself use it and learn it.

## Before I start
Everything I say about the Azure CLI applies equally to Azure PowerShell if you are that way inclined.  
I do think ARM templates and (that other thing) are slightly different, but I have not really used either so take my words on that with a pinch of salt. My understanding is that the CLI is *imperative* - i.e. you give it a list of instructions, whereas ARM etc are more *declarative* - i.e. you describe a desired end state. There's value in both, it's just different. And of course, you can mix and match; you can call ARM templates from the Azure CLI, for example.
ARM feels like a steep learning curve and it's very verbose; I am not sure how comfortable I am with hacking around with it manually. AZ CLI on the other hand is very straight forward to understand.

## Why *not* use it
- you won't remember the commands unless you do this every day, so you'll always be looking up commands. Easier to just do it in the portal.
- no point in keeping the script for how I set up a particular system because, well I did it, and I am not going to set it up again.

## Re-use
make yourself create scripts and save them with the code for the system. Before long you will have a collection of scripts to set things up and you'll be able to say "this one is a bit like that one, so I'll pinch the script and hack around with it for this one".


## Iterate
Realise you got your naming structure wrong or put something in the wrong location - and ten downstream things now depend on. Major pain to undo in the portal. In the script, just blow it all away and re-run the (fixed) script.
It's amazing how many times I find myself re-running scripts to make my setup "just so".

## Duplicate
When you get into setting up resilience etc or replicating Live in a UAT environment etc, it's so easy to just copy the script and hack around with it.

## It's faster
It's just a lot faster to run the AZ CLI commands than to go through the portal - as long as you know the commands. 