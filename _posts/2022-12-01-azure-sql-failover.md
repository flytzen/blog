---
layout: post
title: Azure SQL Failover Setup
date: '2022-12-01'
author: Frans Lytzen
tags: Azure
excerpt: Configure a replica of an Azure SQL database in another region to protect you from failures.
---

# TL;DR;
Azure SQL Failover Groups is a managed version of Geo Replication which enables both automatic and manual failover.
1. Create a SQL Server in another region
2. Create a Failover Group for a database currently on the primary server
3. Check your authentication works on both servers
4. Change your connection string to point to `[failover-group-name].database.windows.net`

# Set up test server
If want to just try this out, use this bash script to create a server and database to experiment with.
Note; Even if you don't have the Azure CLI on your machine, you can run all of this from the [Azure Cloud Shell](https://learn.microsoft.com/en-us/azure/cloud-shell/overview) very easily.

```bash
group=fl-20221201-sqlfailover
location=northeurope
sqlServerName=failovertest-uat-eun-ss
sqlDbName=failovertest-uat-db
sqlUsername=[sql user name of your choice]
sqlPassword=[strong password]
sqlCapacity=10 #20 = S1, 10 = S0

az group create --name $group --location $location

az sql server create --name $sqlServerName --location $location --admin-user $sqlUsername --admin-password $sqlPassword -g $group

az sql db create --name $sqlDbName --server $sqlServerName --edition Standard --capacity $sqlCapacity -g $group
```

# Auth heads-up
Before you go any further it is important to stop for a minute and discuss authentication to avoid problems later.  
The failover is essentially just a bit of network routing magic: When you connect to the Failover connection string, it just routes you to the currently primary server. This means you need to take care that your applications credentials works correctly on both servers. Otherwise, everything will break when you fail over.

## Using server administrator and dbo
If your application just uses the "SQL Server Administrator" login and defaults to everything in SQL (i.e. using "dbo" at the database level) then it's relatively straight-forward: Just make sure you use the exact same SQL Admininistrator username and password on both servers.

\* You are not really supposed to do this! Your application should have a different user, with different permissions etc. In practice, though, many applications use Azure SQL exactly like this.

## Using Managed Identity
If you use Managed Identity (or other AD accounts) to access the database, it should "just work". When you do `CREATE USER [user] FROM EXTERNAL PROVIDER` then it is created as a "Contained User" in the database, meaning it doesn't create a "login" on the SQL Server itself.

That said - do test it, preferably in a test environment.

[Managed Identity with Azure SQL](https://learn.microsoft.com/en-us/azure/active-directory/managed-identities-azure-resources/tutorial-windows-vm-access-sql)

## Using your own SQL users
If you have created your own SQL User for your application (and, indeed, others) and you *didn't* create them as "Contained Users" then you need to do some more manual work. See [this Microsoft documentation](https://learn.microsoft.com/en-us/azure/azure-sql/database/active-geo-replication-security-configure?view=azuresql) for more details.

# set up fail over

```bash
group=fl-20221201-sqlfailover
failoverlocation=westeurope
existingSqlServerName=failovertest-uat-eun-ss
failoverSqlServerName=failovertest-uat-euw-ss
failoverGroupName=failovertest-uat-fog
sqlDbName=failovertest-uat-db # name of the *existing* database
sqlUsername=[sql user name of your choice] # See auth section above. May need to be the same as the primary
sqlPassword=[strong password] # See auth section above. May need to be the same as the primary

az sql server create --name $failoverSqlServerName --location $failoverlocation --admin-user $sqlUsername --admin-password $sqlPassword -g $group

az sql failover-group create --name $failoverGroupName --partner-server $failoverSqlServerName --server $existingSqlServerName --add-db $sqlDbName --failover-policy Automatic -g $group
```

**Remember to look at vNets and firewall rules and make sure they match!**

# Change connection string
Final step is to change the connection string in your application to be `[failover-group-name].database.windows.net`. This will connect your application to whichever database is currently primary and will automatically redirect if the servers fail over.

# Failover

## Automatic fail over
By default the *automatic* fail over will wait one hour before failing over.  
Azure will not automatically "fail back": Once the primary region is working again, you need to manually "fail over" again, if you wish to do so. If your application is running hot-hot in both data centres it doesn't matter, but otherwise you will incur a performance hit and ingress/egress costs by having your application and your database in different regions.

## Manual failover
You can manually fail over in the Azure portal by going to either of the *servers* (not database) and select "Failover groups" from the menu.  
Alternatively you can use the Azure CLI like this:

```bash
group=fl-20221201-sqlfailover
existingSqlServerName=failovertest-uat-eun-ss
failoverSqlServerName=failovertest-uat-euw-ss
failoverGroupName=failovertest-uat-fog

# See which server is currently primary
az sql failover-group list --server $existingSqlServerName -g $group

# Fail over
az sql failover-group set-primary --name $failoverGroupName --server $failoverSqlServerName -g $group

# Fail back
az sql failover-group set-primary --name $failoverGroupName --server $existingSqlServerName -g $group
```
