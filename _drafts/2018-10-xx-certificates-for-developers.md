---
layout: post
title: Certificate basics for developers
date: '2018-10-01'
author: Frans Lytzen
tags: Basics
modified_time: '2018-10-xx'
excerpt: Understand the basics of certificates for SSL/TLS, encryption, authentication and more
---
As developers we regularly have to deal with "certificates". Sometimes it's for https on a website, sometimes for encryption, sometimes for signing in to services, sometimes to manage single-sign-on etc.  
You may be given certificates in a variety of different file formats such pem, cer, pfx and several others. When you need a certificate you may find yourself wondering if you can just create one locally or if you need to go and buy one.  

Most articles you can find about certificates assume you already understand the basics of how they work. Truth is, most of us don't so this post is intended to cover the basics so you can go on to the more in-depth stuff as and when you need to.  

# It's all about the keys
Certificates come in different types for different purposes, but at the basis of it all it's just about using public/private key pairs for different purposes. In order to understand certificates, you need to understand public/private keys first.

A public/private key pair is a set of keys that have a relationship such that anything that is encrypted with the *public* key can only be 



## Performance of public/private key cryptography

# Signing

# Certificate authorities

# Signature chains

# File formats

## Open the files

## Certificate Signing Requests

# Key stores
windows/linux/mac/keyvault...