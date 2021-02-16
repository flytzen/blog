---
layout: post
title: Azure SQL Private End Points with Pulumi
date: '2021-02-16'
author: Frans Lytzen
tags: Azure Pulumi Network Security
modified_time: '2021-02-16'
excerpt: Configure Private End Points on Azure SQL Server with Pulumi
---
Private End Points are the right way to connect PaaS Services to vNets so they can be accessed from other resources on the vNet. There are other, older, ways but they are not as good. Unfortunately, Private End Points require understanding quite a few things, including DNS resolution, split-brain DNS and a bunch of weird abstractions. 
This [Microsoft post](https://docs.microsoft.com/en-us/azure/private-link/create-private-endpoint-cli) gives a good example of how to set it up with Azure SQL. This post does essentially the same thing, except it does not create a VM. Anything related to the VM is left out here.

This is part of a [bigger project](https://github.com/NewOrbit/AzureVnetPassSamples) I am working on to set up a complex vNet-and-PasS setup in Azure with multiple services, regions and VPN clients.


# Before we can add the private end point
This bit here sets up a network and a SQL Server so we can add the private end point.

```typescript
const resourceGroup = new azure.resources.latest.ResourceGroup("fl-vnettest", {
    location,
    resourceGroupName: "fl-vnettest"
})

const vnet = new azure.network.latest.VirtualNetwork("fl-vnettest-vn", {
    resourceGroupName: resourceGroup.name,
    virtualNetworkName: "fl-vnettest-vn",
    addressSpace: { addressPrefixes: ["10.0.0.0/16"] },
    location
});

const azureSqlSubnet = new azure.network.latest.Subnet("azure-sql", {
    subnetName: "azure-sql", 
    addressPrefix: "10.0.4.0/24",
    virtualNetworkName: vnet.name,
    resourceGroupName: resourceGroup.name,
    privateEndpointNetworkPolicies: "Disabled"
});

const passSqlServer = new azure.sql.v20200801preview.Server("fl-vnettest-ss",
    {
        serverName: "fl-vnettest-ss",
        administratorLogin,
        administratorLoginPassword,
        location,
        resourceGroupName: resourceGroup.name,
        tags,
        minimalTlsVersion: "1.2",
    });
```

# Setting up the private end point
*This is the important bit*

### First, add a Private DNS Zone and link it to the network
You need to do this *once* per vnet, no matter how many SQL Servers you have in that vNet. You will need to do this for each type of service, i.e you'd need something similar for Storage etc. This is required in order to make DNS lookups for the SQL Server return the *local* IP address, rather than the public one. The tricky bit is that an Azure SQL Server will always have a *public* IP Address, even if no public access is allowed. When you set up the DNS resolution just right, a DNS lookup on your private network will return the private IP address and when you look it up on the public internet, you will see the public IP address. There is a lot more detail to this, but that is for another post. What is described here is the easiest and most reliable way to make this work. There *are* other ways, but they require more knowledge and more work.

```typescript
// Sets up a private DNS Zone for SQL private link entries
const sqlPrivateDns = new azure.network.latest.PrivateZone("fl-vnettest-sqldns", {
    privateZoneName: "privatelink.database.windows.net",
    resourceGroupName: resourceGroup.name,
    location: "global" //https://github.com/Azure/azure-cli/issues/6052
})

// Links the private DNS Zone to the vnet
const dnsVnetLink = new azure.network.latest.VirtualNetworkLink("fl-vnettest-vnl", {
    privateZoneName: sqlPrivateDns.name,
    resourceGroupName: resourceGroup.name,
    virtualNetworkLinkName: "fl-vnettest-vnl",
    registrationEnabled: true,
    virtualNetwork: { id: vnet.id },
    location: "global"
})
```

### Set up a Private End Point for the SQL Server
You need to do this for each SQL Server in your vnet
```typescript
// Create the private end point for the SQL Server
const sqlPrivateEndPoint = new azure.network.latest.PrivateEndpoint("fl-vnettest-sqlpep", {
    privateEndpointName: "fl-vnettest-sqlpep",
    resourceGroupName: resourceGroup.name,
    location,
    subnet: { id: azureSqlSubnet.id },
    privateLinkServiceConnections: [
        {
            name: "sql",
            privateLinkServiceId: passSqlServer.id,
            groupIds: ["sqlServer"], // This particular group id is not documented by Azure that I can see - I found it by trying to do this manually in the portal
            privateLinkServiceConnectionState: {
                actionsRequired: "None",
                description: "Auto-approved",
                status: "Approved"
            }
        }
    ]
})

// Connects the private DNS Zone and the private end point (I think)
const sqlPrivateDnsZoneGroup = new azure.network.latest.PrivateDnsZoneGroup("fl-vnettest-sqldnsgroup", {
    privateDnsZoneGroupName: "fl-vnettest-sqldnsgroup",
    privateEndpointName: sqlPrivateEndPoint.name,
    resourceGroupName: resourceGroup.name,
    name: "fl-vnettest-sqldns",
    privateDnsZoneConfigs: [{
        name: sqlPrivateDns.name,
        privateDnsZoneId: sqlPrivateDns.id
    }]
})

```


# Connect a Web App
If you want to connect a web app to the vNet so it can talk to the SQL (not really the scope of this post, but hey) then do the following. Note, this is *outbound* from the Web App. Inbound traffic *to* the Web App is best handled by setting up a Private End Point for the Web App - though that currently requires a Premium App Service Plan. Service End Points is another option but they are much harder to use in my experience and I don't recommend them.

```typescript
// Set up a subnet dedicated as an entry point for Web Apps
const frontEndSubnet = new azure.network.latest.Subnet("front-end", {
    subnetName: "front-end",
    addressPrefix: "10.0.1.0/24",
    virtualNetworkName: vnet.name,
    resourceGroupName: resourceGroup.name,
    delegations: [
        {
            serviceName: "Microsoft.Web/serverfarms",
            name: "front-end-delegation"
        }
    ]
});

// Add an app service plan and a web app
const appServicePlan = new azure.web.latest.AppServicePlan("fl-vnettest-as", {
    name: "fl-vnettest-as",
    location,
    resourceGroupName: resourceGroup.name,
    sku: {
        family: "S",
        capacity: 1,
        size: "S1",
        name: "S1"
    }
});

const appService = new azure.web.latest.WebApp("fl-vnettest-wa",
    {
        name: "fl-vnettest-wa",
        location,
        resourceGroupName: resourceGroup.name,
        clientAffinityEnabled: false,
        httpsOnly: true,
        serverFarmId: appServicePlan.id,
        tags,
        siteConfig: {
            appSettings: [
                // This is critical to make DNS lookups return the internal IP Address. 
                // Without it, your web app will get the public IP address of the SQL Server and will be denied access. 
                // 168.63.129.16 is a magic constant provided by Microsoft and works with the setup above. 
                // You will only need something different if you use your own DNS servers.
                // See https://feedback.azure.com/forums/169385-web-apps/suggestions/38383642-web-app-and-private-dns-zone-support
                { name: "WEBSITE_DNS_SERVER", value: "168.63.129.16" }, 
                { name: "WEBSITE_VNET_ROUTE_ALL", value: "1" },
                { name: "hostname", value: passSqlServer.name.apply(n => n + ".database.windows.net") },
                {
                    name: "sql",
                    value: "Server=tcp:fl-vnettest-ss.database.windows.net,1433;Initial Catalog=fl-vnettest-db;Persist Security Info=False;User ID=" + administratorLogin + ";Password=" + administratorLoginPassword + ";MultipleActiveResultSets=False;Encrypt=True;TrustServerCertificate=False;Connection Timeout=30;"
                    // Note: This isn't ideal for many reasons, but I haven't got the brain power to nest apply calls :)
                }    
            ]
        },
    })

// Give the web app access to the vnet <- the thing you actually wanted to see :)
// Note that "Swift" is the "modern" way of doing this. Don't do what I do and spend ours trying to make WebAppVnetConnection work - it won't.
const webAppVNetConnection = new azure.web.latest.WebAppSwiftVirtualNetworkConnection("fl-vnettest-wa-vc", {
    name: appService.name,
    resourceGroupName: resourceGroup.name,
    subnetResourceId: frontEndSubnet.id
})

```

# References
- [Pulumi Azure Next Gen API Reference](https://www.pulumi.com/docs/reference/pkg/azure-nextgen/)
- [Quickstart: Create a Private Endpoint using Azure CLI](https://docs.microsoft.com/en-us/azure/private-link/create-private-endpoint-cli)
- [Unable to provision PrivateDnsZoneGroup](https://github.com/pulumi/pulumi-azure-nextgen/issues/227)
- [AzureVnetPassSamples repo (work in progress)](https://github.com/NewOrbit/AzureVnetPassSamples)
- [Web App and Private DNS zone support](https://feedback.azure.com/forums/169385-web-apps/suggestions/38383642-web-app-and-private-dns-zone-support)