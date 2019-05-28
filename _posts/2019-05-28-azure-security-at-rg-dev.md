---
layout: post
title: Azure Security Talk at rg-dev #29
date: '2019-05-28'
author: Frans Lytzen
tags: Azure
modified_time: '2019-05-28'
excerpt: I did a talk on "How to secure your web-app in Azure" at rg-dev #29
---
I had the pleasure of doing a presentation on [Securing Web Apps in Azure](/talks/Securing_web_apps_in_azure) at [rg-dev #29](https://www.meetup.com/rg-dev/events/253406007/) in Rzeszow, Poland on 24 May 2019. 
I was really impressed by how well organised the event was and just how dedicated and professional the audience were. We are talking about people giving up their Friday night to hear about security in Azure as well as [Piotr Stapp](https://twitter.com/ptrstpp950)'s talk about migrating [https://dotnetomaniak.pl/](https://dotnetomaniak.pl/) to Azure.  
Apparently this was the first ever talk done in English at rg-dev, but that didn't feel like a barrier at all.

I go to Rzeszow quite frequently these days as NewOrbit has opened a small, but growing, office there ([interested in joining us?](https://neworbit.pl/careers/)). I look forward to attending more rg-dev meetups, though I probably need to work a bit more on my Polish :) I am consistently impressed by the developers I meet in Rzeszow and look forward to working with more of them.

The slides from the talk can be downloaded in [PowerPoint format from GitHub](https://github.com/flytzen/SecurityTalk/blob/master/Secure%20Your%20Web%20App%20Presentation.pptx?raw=true) or you can see them (without animations) on Slideshare below:

<div style="text-align:center;">
<iframe src="//www.slideshare.net/slideshow/embed_code/key/vZ12nwRamiEwrr" width="595" height="485" frameborder="0" marginwidth="0" marginheight="0" scrolling="no" style="border:1px solid #CCC; border-width:1px; margin-bottom:5px; max-width: 100%;" allowfullscreen> </iframe>
</div>

The talk covers the things you can do to secure web apps and APIs hosted in Azure Web Apps and Functions.

## Outline
<div style="width: 480px; height: 360px; margin: 10px; position: relative;float:right;"><iframe allowfullscreen frameborder="0" style="width:480px; height:360px" src="https://www.lucidchart.com/documents/embeddedchart/f4062454-7eb7-4e7a-abca-2d244417011b" id="Gf4kpeh9VZr3"></iframe></div>
### Concepts
- Understand your exposure; how much at risk are you and how much work should you do to secure your app?
- Understand that you are probably more at risk from internal actors making mistakes with data than you are from external hackers.
- You should think about your security at three levels;
  - How to *prevent* someone from getting in
  - How to *detect* that someone is attacking you 
  - How to *mitigate* the attack

### Getting in
Azure Web Apps and Functions are already protected by a Firewall. It is impractical, if not impossible, to add your own firewall unless you use an App Service Environment. If you must, you can add a WAF and lock down the access to your web app to only the IP address of the WAF.  
SSL Certificates in Azure are harder than they should be, the talk covers a few different options and their pros and cons.

I strongly encourage you to use Azure ADB2C (and/or Azure AD) for authentication instead of having your own usernames and passwords.

### Secret Management
One of the biggest problem with both security and auditing is that the application needs passwords to access certain services; Those passwords need to be stored somewhere, which usually means they are exposed to various people. This leads to auditing and security issues.

With Azure Key Vault and Managed Identity it is surprisingly easy to eliminate this problem; there is a lot to learn before you can do it - but the actual implementation is really easy.

### Network Isolation
By default, all your back-end services such as Cosmos, SQL, storage and so on are open to other services running in Azure and some are open to the Internet. It is surprisingly easy to set up a private Virtual Network and set your back-end services so they can only be accessed from that virtual network - and then allow your web app to talk to that network. You can even set up micro-services hosted in Azure Web Apps or Functions in this way, completely blocking access from anything other than your web app.

The main challenge with the virtual networks is that 90% of the documentation on them has to do with features that are irrelevant when you are using Web Apps instead of VMs or Cloud Services, so it's mainly about knowing what to ignore.

### Encryption
Once the attacker is in - or the well-meaning employee is accessing data they probably shouldn't - it is extremely helpful that the data is encrypted. The obvious at-rest and transport encryption is there by default, but you can use Always Encrypted and Client-side encryption to easily encrypt data at the application level.

### Detection
One of the most overlooked areas of security is detecting when someone is attacking you. If you don't know someone is trying to hack you or that some well-meaning employee is accessing data they shouldn't, you can't do anything about it.
Azure has a rich set of features to help alert you when something isn't right. In the talk I focus on Application Insights, Threat Protection and Advanced Data Security.