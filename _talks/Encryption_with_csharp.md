---
layout: talksummary
title: Encryption with C#
tags: C# Azure
excerpt: How to encrypt data correctly in C# - and optionally use Azure Keyvault to do it even better
sequence: 20
---
> C# support AES encryption so I can just use that and all is well when I need to encrypt some data. Right? ... Right??

- So, do I use the Rijndael algorithm in C# or the AES one?
- Now it's asking for a encryption key - but why is it a byte array and not a string. How do I get one, how do I store it and can I just use one key to encrypt all my data?
- What's up with that IV it's asking me for? How do I get one of those and does it matter what it is?
- I also [read on the internet](https://blog.codinghorror.com/why-isnt-my-encryption-encrypting/) that there some settings like ECB, CBC and PaddingMode and more - which should I use again to be safe?
- What do you mean I should also create a MAC from my encrypted data with a different secret to get Authenticated Encryption and avoid the data being manipulated?
- What's the difference between Public/Private Key Encryption and Symmetric Key Encryption and why should I care when I just want to store some data safely?
- Can't I just hand this all over to an external Key Vault service in the Cloud?
- Do I even need to worry about this - doesn't the Cloud do it for me automatically?

As part of my role at [NewOrbit](https://neworbit.co.uk/azure/) I get to do [security reviews](https://neworbit.co.uk/azure/#need-help) for a lot of our Azure customers and one thing I see very often is that the way encryption works is very poorly understood. C# *tries* to make it easy, but unfortunately allows you access to all the knobs and dials, meaning you have to learn a lot more than you want to. 
Because of this, most attempts at using the encryption algorithms in C# that I see are wrong, usually leading to quite significant security holes that the developers are blisfully unaware off.

The focus of this talk is to enable you to use the encryption tools in C# safely and effectively. It's utterly pragmatic and will only do as much theory as is stricly required. It will also touch briefly on how some of the cloud can either replace or improve your own encryption.
