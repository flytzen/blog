---
layout: post
title: Securing your web app in Azure
date: '2018-04-29T21:40:00.000+01:00'
author: Frans Lytzen
excerpt: A video overview of some of the Azure technologies that you can use to better protect your web applications in Azure - all depending on your required security level, of course. 
tags:
- Azure
- Security
modified_time: '2018-04-29T21:40:00.000+01:00'

---
So you have deployed your web app to Azure. Now, how do you go about making it secure?
I gave a talk on this topic at  [DotNet Oxford](https://www.meetup.com/dotnetoxford/) on 24 April 2018 and recorded it. You can view the video below.   

The video runs through a scenario using an ASP.Net Web App hosted on Azure App Service and covers a number of features you can use to improve your security - as well as a number of features that are not available for App Services.

The talk covers a lot of ground in an hour and everything is kept at a high level, but is nonetheless heavy on examples and code.  
Watching the video myself, I realised I say "Okay" and "So" way, way too much. Sorry...     
  
  ***

[James World](https://twitter.com/jamesw0rld) made this nice sketch note of the talk, reproduced with permission.  
![Sketch note]({{ "/assets/2018-04-29-security-talk-sketch-note.jpg" | absolute_url }})

***

<div style="text-align:center">
<iframe width="560" height="315" src="https://www.youtube.com/embed/2tR5sEk46v0" frameborder="0" allow="autoplay; encrypted-media" allowfullscreen></iframe>
</div>

## Some key timings  

|Use SSL|11:03|
|Virus scanning|20:01|
|WAF|21:00|
|Vnet|23:20|
|Azure Key Vault|26:10|
|Managed Service Identity|27:50|
|Use Key Vault and managed identify to store secrets|29:55|
|ASP.Net Core configuration with Key Vault|31:55|
|Connect to Azure SQL with Managed Identity (or not)|36:27|
|Encrypt data at rest|38:00|
|Require secure transport|40:30|
|SQL Always Encrypted|41:40|
|Storage client-side encryption (not shown)|52:00|
|Use Azure AD to access Azure|53:25|
|Use Azure AD to access Azure SQL|54:05|
|Supporting Security tools in Azure|56:50|
|Detection|57:45|