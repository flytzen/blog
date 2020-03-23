---
layout: post
title: Azure Private End Points, VPNs and multi-site
date: '2020-03-23'
author: Frans Lytzen
tags: azure
modified_time: '2020-03-23'
excerpt: Azure Private End Points, VPN clients and multi-region setups.
---
*NOTE: Right now this is not a finished post - I haven't yet figured out how to make this work. I have written it up here partially to be able to raise a support query.*

# This is what I am trying to achieve
<div style="width: 640px; height: 480px; margin: 10px; position: relative;"><iframe allowfullscreen frameborder="0" style="width:640px; height:480px" src="https://www.lucidchart.com/documents/embeddedchart/41848683-692d-4268-bc3f-e4d9ab764098" id="Pt~IQN7FkdqC"></iframe></div>

I have a database in the Europe North and another one in Australia East. I have users in both geographies and they need to be able to access both databases in order to write PowerBI reports. Ideally I want the users in both locations to be able to connect to a local VPN Gateway to avoid unnecessary roundtrips; at the distances we are talking about, latency becomes a real issue.

Until fairly recently the only way to do this was to open up the user's IP address on the SQL Firewall. Fortunately, with the advent of Private Link / Private Endpoint, you can now connect to Azure SQL via a VPN client.

The complication is that that a virtual network cannot span multiple data centres, so you have to connect them using either "virtual network peering" or "virtual network gateway connections". I have tried both and they come with different trade-offs.

# Peering
Virtual networks can be *peered*. The setup here is based on [this article](https://docs.microsoft.com/en-us/azure/vpn-gateway/vpn-gateway-peering-gateway-transit).
<div style="width: 640px; height: 480px; margin: 10px; position: relative;"><iframe allowfullscreen frameborder="0" style="width:640px; height:480px" src="https://www.lucidchart.com/documents/embeddedchart/2eb8492a-7b8b-4410-a4f8-9b386194dabc" id="vC~I0fkY7VUW"></iframe></div>

In this setup, I have created two peerings:
- One going from Europe to Australia which has "Allow Gateway Transit" set.
- One going from Australia to Europe which has "Use Remote Gateways" set.

## What works
I can connect to the Europe Gateway and connect to the database in both locations. That is a success.  

When I connect to the Azure VPN client, I can see in the `AzureVpnCxn.log` that I get both the 10.0.0.0/16 and the 10.1.0.0/16 routes.

```
[‎3‎/‎23‎/‎2020‎ ‎1‎:‎28‎:‎14‎ ‎PM] PId:[00025864] TId:[00024864] [momenta-live-eun] [{cd712a18-c0d7-482c-9f11-7404ad0c7b1b}] [Verbose] Received option: route 10.0.0.0 255.255.0.0
[‎3‎/‎23‎/‎2020‎ ‎1‎:‎28‎:‎14‎ ‎PM] PId:[00025864] TId:[00024864] [momenta-live-eun] [{cd712a18-c0d7-482c-9f11-7404ad0c7b1b}] [Verbose] Assigned Route: 10.0.0.0 / 255.255.0.0
[‎3‎/‎23‎/‎2020‎ ‎1‎:‎28‎:‎14‎ ‎PM] PId:[00025864] TId:[00024864] [momenta-live-eun] [{cd712a18-c0d7-482c-9f11-7404ad0c7b1b}] [Verbose] Received option: route 10.1.0.0 255.255.0.0
[‎3‎/‎23‎/‎2020‎ ‎1‎:‎28‎:‎14‎ ‎PM] PId:[00025864] TId:[00024864] [momenta-live-eun] [{cd712a18-c0d7-482c-9f11-7404ad0c7b1b}] [Verbose] Assigned Route: 10.1.0.0 / 255.255.0.0
[‎3‎/‎23‎/‎2020‎ ‎1‎:‎28‎:‎14‎ ‎PM] PId:[00025864] TId:[00024864] [momenta-live-eun] [{cd712a18-c0d7-482c-9f11-7404ad0c7b1b}] [Verbose] Received option: route-gateway 172.16.10.1
[‎3‎/‎23‎/‎2020‎ ‎1‎:‎28‎:‎14‎ ‎PM] PId:[00025864] TId:[00024864] [momenta-live-eun] [{cd712a18-c0d7-482c-9f11-7404ad0c7b1b}] [Verbose] Received option: topology subnet
[‎3‎/‎23‎/‎2020‎ ‎1‎:‎28‎:‎14‎ ‎PM] PId:[00025864] TId:[00024864] [momenta-live-eun] [{cd712a18-c0d7-482c-9f11-7404ad0c7b1b}] [Verbose] Received option: ifconfig 172.16.10.2 255.255.255.128
[‎3‎/‎23‎/‎2020‎ ‎1‎:‎28‎:‎14‎ ‎PM] PId:[00025864] TId:[00024864] [momenta-live-eun] [{cd712a18-c0d7-482c-9f11-7404ad0c7b1b}] [Verbose] Assigned IP address: 172.16.10.2 / 255.255.255.128
[‎3‎/‎23‎/‎2020‎ ‎1‎:‎28‎:‎14‎ ‎PM] PId:[00025864] TId:[00024864] [momenta-live-eun] [{cd712a18-c0d7-482c-9f11-7404ad0c7b1b}] [Verbose] Assigned Route: 172.16.10.0 / 255.255.255.128
[‎3‎/‎23‎/‎2020‎ ‎1‎:‎28‎:‎14‎ ‎PM] PId:[00025864] TId:[00024864] [momenta-live-eun] [{cd712a18-c0d7-482c-9f11-7404ad0c7b1b}] [Verbose] Received option: cipher AES-256-GCM
[‎3‎/‎23‎/‎2020‎ ‎1‎:‎28‎:‎14‎ ‎PM] PId:[00025864] TId:[00010832] [momenta-live-eun] [{cd712a18-c0d7-482c-9f11-7404ad0c7b1b}] [Verbose] Session established!
```

## What doesn't work
This setup is designed for a "hub and spoke" architecture; Setting the "Use Remote Gateways" setting explicitly disallows me from having a Gateway in Australia. In other words, my Australia users have to connect to the VPN Gateway in Europe, which means they'll suffer double latency when querying the Australia database.  
If you don't set the "Use Remote Gateways" options, you won't get the 10.1.0.0/16 route when you connect with the Azure VPN Client.

# Gateway Connection
In this setup, you use Gateway Connections between the two networks. 
<div style="width: 640px; height: 480px; margin: 10px; position: relative;"><iframe allowfullscreen frameborder="0" style="width:640px; height:480px" src="https://www.lucidchart.com/documents/embeddedchart/54b1279f-d5f1-4513-9e5b-a8346ab78cc6" id="QD~Ipb0I~VxK"></iframe></div>

## What works
I get the routes served to me, from the `AzureVpnCxn.log` I get this:
```
[‎3‎/‎20‎/‎2020‎ ‎4‎:‎51‎:‎21‎ ‎PM] PId:[00011592] TId:[00018256] [momenta-live-eun] [{7412cb8b-e91a-4757-8f1f-c32c4406248e}] [Verbose] Received option: route 10.0.0.0 255.255.0.0
[‎3‎/‎20‎/‎2020‎ ‎4‎:‎51‎:‎21‎ ‎PM] PId:[00011592] TId:[00018256] [momenta-live-eun] [{7412cb8b-e91a-4757-8f1f-c32c4406248e}] [Verbose] Assigned Route: 10.0.0.0 / 255.255.0.0
[‎3‎/‎20‎/‎2020‎ ‎4‎:‎51‎:‎21‎ ‎PM] PId:[00011592] TId:[00018256] [momenta-live-eun] [{7412cb8b-e91a-4757-8f1f-c32c4406248e}] [Verbose] Received option: route 10.1.0.0 255.255.0.0
[‎3‎/‎20‎/‎2020‎ ‎4‎:‎51‎:‎21‎ ‎PM] PId:[00011592] TId:[00018256] [momenta-live-eun] [{7412cb8b-e91a-4757-8f1f-c32c4406248e}] [Verbose] Assigned Route: 10.1.0.0 / 255.255.0.0
[‎3‎/‎20‎/‎2020‎ ‎4‎:‎51‎:‎21‎ ‎PM] PId:[00011592] TId:[00018256] [momenta-live-eun] [{7412cb8b-e91a-4757-8f1f-c32c4406248e}] [Verbose] Received option: route-gateway 172.16.10.1
[‎3‎/‎20‎/‎2020‎ ‎4‎:‎51‎:‎21‎ ‎PM] PId:[00011592] TId:[00018256] [momenta-live-eun] [{7412cb8b-e91a-4757-8f1f-c32c4406248e}] [Verbose] Received option: topology subnet
[‎3‎/‎20‎/‎2020‎ ‎4‎:‎51‎:‎21‎ ‎PM] PId:[00011592] TId:[00018256] [momenta-live-eun] [{7412cb8b-e91a-4757-8f1f-c32c4406248e}] [Verbose] Received option: ifconfig 172.16.10.3 255.255.255.128
[‎3‎/‎20‎/‎2020‎ ‎4‎:‎51‎:‎21‎ ‎PM] PId:[00011592] TId:[00018256] [momenta-live-eun] [{7412cb8b-e91a-4757-8f1f-c32c4406248e}] [Verbose] Assigned IP address: 172.16.10.3 / 255.255.255.128
[‎3‎/‎20‎/‎2020‎ ‎4‎:‎51‎:‎21‎ ‎PM] PId:[00011592] TId:[00018256] [momenta-live-eun] [{7412cb8b-e91a-4757-8f1f-c32c4406248e}] [Verbose] Assigned Route: 172.16.10.0 / 255.255.255.128
[‎3‎/‎20‎/‎2020‎ ‎4‎:‎51‎:‎21‎ ‎PM] PId:[00011592] TId:[00018256] [momenta-live-eun] [{7412cb8b-e91a-4757-8f1f-c32c4406248e}] [Verbose] Received option: cipher AES-256-GCM
[‎3‎/‎20‎/‎2020‎ ‎4‎:‎51‎:‎21‎ ‎PM] PId:[00011592] TId:[00012216] [momenta-live-eun] [{7412cb8b-e91a-4757-8f1f-c32c4406248e}] [Verbose] Session established!
```

**This is exactly the same as when I used peerings**

## What doesn't work
When I connect to the Europe Gateway, I am *not* able to connect to the Australia database; the connection simply times out. I suspect that the remote network is unable to respond, because:  
- The Europe -> Australia Gateway connection shows that it has data going *out* but 0 bytes data *in*.
- The Australia -> Europe Gateway connection shows that it has data coming *in* but 0 bytes data *out*.

## Notes
- I don't know why you have to create two connections, one for each direction.
-  You can enable `Border Gateway Protocol` (BGP) on the gateways. Interestingly, if you do that, you *don't* get the route for the remote network when you connect to the VPN Client. Still, you can't connect to the remote database.



# Naming and certificates
There's another whole set of problems about DNS names and SQL Server certificates.  
The SQL Server has a *public* IP address, but Private Link also gives it a *Private* IP address. In my example here, they are 10.0.1.4 and 10.1.1.4 respectively. When you connect to the SQL Server over the VPN, you need to use the private IP address. So you may think you can just type 10.0.1.4 into PowerBI and start connecting. You will unfortunately get an "Encryption Support" error saying "The server name provided doesn't match the server name on the SQL Server SSL certificate".  

This makes sense; The SQL Server has a certificate that has been issued to `mydatabase.database.windows.net` (just like a website would) and when you try to connect to `10.0.1.4`, you get an error - just like you would in a web browser in the same situation.... *TO BE FINISHED*