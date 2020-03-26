---
layout: post
title: Azure Private Link and DNS
date: '2019-03-06'
author: Frans Lytzen
tags: azure
modified_time: '2019-03-06'
excerpt: TBD.
---



There's another whole set of problems about DNS names and SQL Server certificates.  
The SQL Server has a *public* IP address, but Private Link also gives it a *Private* IP address. In my example here, they are 10.0.1.4 and 10.1.1.4 respectively. When you connect to the SQL Server over the VPN, you need to use the private IP address. So you may think you can just type 10.0.1.4 into PowerBI and start connecting. You will unfortunately get an "Encryption Support" error saying "The server name provided doesn't match the server name on the SQL Server SSL certificate".  

This makes sense; The SQL Server has a certificate that has been issued to `mydatabase.database.windows.net` (just like a website would) and when you try to connect to `10.0.1.4`, you get an error - just like you would in a web browser in the same situation.

## Use the special "privatelink" DNS entry as you are told to
Microsoft are trying to help by offering you two DNS names for the database;
- `mydatabase.database.windows.net`
- `mydatabase.privatelink.database.windows.net`

Crucially, they will *both* point to the *public* IP address for your SQL Server! However, the suggestion is that you override `mydatabase.privatelink.database.windows.net` to point to the *private* IP Address in your local DNS server. I suspect the idea is that you can always try to connect to `mydatabase.privatelink.database.windows.net` and if you try to connect from a public network, it will just give you the public IP. I don't know for sure, this is just a guess. I believe there is also some magic happening if you try to connect from Azure connected systems, but I have yet to test that.

If you are in an office, then that makes sense as you will probably have a private DNS server. If you are working remotely, however, you have to edit your `hosts` file to put the `privatelink` DNS entry in there. Alternatively, you *may* be able to set up a private DNS server in Azure and configure your machine to use it - I am not sure.

*BUT - none of this will actually help you.* It turns out that the SQL server certificate is also not valid for `mydatabase.privatelink.database.windows.net` so you end up with exactly the same problem as if you just use the IP address. 

As far as I can tell, you have to change your local `hosts` file or local DNS server to point `mydatabase.database.windows.net` to the private IP address in order to get this to work. I think that Azure *should* use "Subject Alternative Name" - but it does not seem to do that.

*Note: I am actually investigating this further*


Does the private DNS Zone apply to VPN clients??