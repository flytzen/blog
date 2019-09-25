---
layout: post
title: Can a SHA256 Hash have a "/" in it?
date: '2019-09-25'
author: Frans Lytzen
tags: C# Technology
modified_time: '2019-09-25'
excerpt: I was recently asked whether a SHA265 hash could have a "/" in it. If you know how hashes work, you probably know that the question doesn't make sense. But I thought it was a good reason to write a bit about how hashes work and, specifically, how they manifest in our coding. Personally, I've been doing this for a long time and some of the details were rather opaque to me until recently.
---
I was recently asked whether a SHA265 hash could have a "/" in it. If you know how hashes work, you probably know that the question doesn't make sense. But I thought it was a good reason to write a bit about how hashes work and, specifically, how they manifest in our coding. Personally, I've been doing this for a long time and some of the details were rather opaque to me until recently.

The short answer is that the "/" question comes about because we usually base64 encode hashes, because they are represented as byte arrays, because they are large numbers - meaning that the question really is "how do I avoid '/' in base64 encoding".

Let's peel back the layers of the onion.

# What is a hash and why do I want one?
The idea with a *hash* is that you can take some input (often text, but it doesn't have to be) and then calculate a *hash* from it. 
- The hash always has the same length (the length depends on which algorithm you use).
- If you hash the same input again, you will always get the same result.
- It is impossible to deduce the input from the hash.
- Good hashes have low ["collision" rates](https://blogs.msdn.microsoft.com/ericlippert/2010/03/22/socks-birthdays-and-hash-collisions/), meaning that the output hashes are widely spread across the possible values, making it unlikely that two different inputs will produce the same output (but not impossible as the hash is shorter than the input).
- A small change to the input should produce a completely different hash.


## Password hashing
You have probably heard about hashing passwords instead of storing them. The idea here is that when the user sets up a password, you hash it and then store the hash in the database. When the user tries to log in, you take the value they enter, hash it and check if the hash matches the stored hash. If they do, it is overwhelmingly likely that they entered the correct password. But, if an attacker steals your database, they won't be able to deduce the passwords from the hash, which is useful because people tend to re-use passwords across sites.   

*Note: This is a very short explanation and ignores such things as salting, brute force, iterations and rainbow tables. Do **not** implement password hashing based on what I write here, there is a lot more to it!*

## Message Authentication Codes (MAC or HMAC)
A very cool application of hashes is using it to make messages or other packages of data tamper proof.  
Let's imagine you need to put a message on a queue to process an order. The message could include a product ID and a price. What if someone could intercept that message in transit and change the price?  
What you can do is concatenate all the information in the message and then create a *hash* from it and include that hash in the message. The receiver can then do the same hashing and compare the resulting hash to make sure the message has not been modified in transit. This can be used in all sorts of situations, including when you round-trip data to a browser and want to make sure no-one is messing with it.
Of course, hash algorithms are public so you need to make sure the attacker can't just re-create the hash. There are some popular approaches, depending on your situation;

### Shared Secret
If you are sending data to your own system, i.e. on a message queue or round-tripping some data to a browser etc, you would usually add a *secret* (think password) to the concatenated data before you create the hash. An attacker won't know the secret, so can't re-create the hash.  
The recipient re-creates the hash in the same way you did, using the same secret, and compares the hashes to make sure they are the same.  

### Public/Private key cryptography
If you are sending data between two parties, it is more common to use public/private key cryptography to *sign* the message. In this case, you create a hash from the concatenated data and then you *encrypt* the hash with your *private* key and include the result with the message.  
The recipient will create the hash in the same way you did. They will then *decrypt* the hash you sent using your *public* key and check they match. 

# So what's with that string and the "/"?
When you generate a hash in code, you will get a *byte array* - not a string. In order to store the hash in a database and send it over HTTP etc, you will usually base64 encode it; If you aren't familiar with it, "base64" is a mechanism for turning binary data into strings for the purpose of transmission and storage.  

It turns out that, by default, base64 may produce "/" in it's output and that can be problematic in some scenarios, such as when including it in URLs. 

So, can a SHA265 Hash include a "/"? No - but the base64 representation of *any* hash might. See [Wikipedia](https://en.wikipedia.org/wiki/Base64) for a deeper discussion of base64 and see the "URL Applications" section for some tips on how to modify base64 to make it safe in URLs.

# Why is the hash a byte array anyway? 
When I think about a byte array I think about video files and other binary data so I struggled to understand why the hash algorithms insist on returning byte arrays. I mean, why can't it just give me a string?   

**It's because a hash algorithm doesn't actually produce a byte array at all.** 

Hash algorithms produce a very large *number*. For example the SHA1 algorithm produces a 160-bit number; That means it takes 160 0s and 1s to write it. In base 10 it would take about 48 digits to write it out. SHA256 produces a 256 bit number, SHA512 a 512 bit number and so on.  
Our programming languages don't generally have built-in types that can represent numbers that large, so the 256 bits are simply chopped into 8-bit chunks and presented to you as a byte array because there is no other easy way to do it - but you should really think about it as a single number with 256 binary digits.