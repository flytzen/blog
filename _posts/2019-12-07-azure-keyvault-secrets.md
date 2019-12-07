---
layout: post
title: Azure Key Vault Secrets
date: '2019-12-07'
author: Frans Lytzen
tags: Azure Security
modified_time: '2019-12-07'
excerpt: Use Azure Key Vault to safely store secrets and passwords in Azure
---
[Azure Advent Calendar 2019](https://azureadventcalendar.com/) is a great initiative to generate and share a bunch of Azure content.
I was fortunate enough to be able to create a video about Azure Key Vault for the Advent Calendar. Specifically, this video focuses on storing *secrets* in Azure Key Vault.

<iframe width="560" height="315" src="https://www.youtube.com/embed/SIA3pleuqfQ" frameborder="0" allow="accelerometer; autoplay; encrypted-media; gyroscope; picture-in-picture" allowfullscreen></iframe>


Please check out all the [other great content and subscribe to the channel](https://www.youtube.com/channel/UCJL9wCcmeMBbah4J0uOWIPg).

# Content
The broad outline of the video is as follows:

## What is Azure Key Vault?
Azure Key Vault provides the following, quite separate, areas of functionality:

- Secrets: Safe place to store passwords and other secrets
- Keys / Cryptography: Public/Private key encryption
- Certificate Store: Safe place to store X509 certificates
- Storage Account key rotation: Essentially a wrapper around "Secrets" to help you manage and rotate you Azure Storage Account keys.

## Azure Key Vault Secrets
- Store and retrieve secrets
- Fine-grained access control
  - For example, write-only access to a user, read-only access to an app
- Auth is easy with Managed Identity
- Store multiple versions with stop/start dates etc
  - The dates are advisory and your code has to decide whether to read and use them
  - Uniquely reference specific versions


## Before you start with the examples
[Install the Azure CLI ](https://docs.microsoft.com/en-us/cli/azure/install-azure-cli?view=azure-cli-latest)  
*Note: I use the PowerShell line continuation character below - if you run this in bash you need to change that*

```
az login
az account set --subscription xxxx
az configure --defaults location=westeurope group=fl-test-keyvault
az group create --name fl-test-keyvault
```

## Create a Key Vault
```
az keyvault create --name fl-test
```

## Give read and write permissions to a user
```
az keyvault set-policy `
    --name fl-test `
    --upn "flytzen@neworbit.co.uk" `
    --secret-permissions get, list, set  
```
In the real world, a consuming application should not have "set" and an admin shouldn't have get

## Add a secret
*You can do this in the portal as well - and in code of course*
```
az keyvault secret set `
    --name supersecretpassword `
    --vault-name fl-test `
    --value ohmyworditssosecret `
    --not-before 2019-12-03T14:30:25z `
    --expires 2019-12-24T23:59:59z `
    --disabled false `
    --output table
```
Note that the "disabled" flag is enforced; most operations are blocked on a disabled secret.
However, the nbf and expiry are for information and it's up to you to read them and act on them
This is *different* for keys!

## Set up a console app to read the secret
*Note: New packages have just come out but they have [less intuitive auth](https://github.com/Azure/azure-sdk-for-net/issues/8934) - for now*
```
dotnet new console -n ShowSecrets 
cd ShowSecrets 
dotnet add package Microsoft.Azure.KeyVault
dotnet add package Microsoft.Azure.Services.AppAuthentication
code .
```

## Fields
Add the following to the top of the Main class:
```csharp
private static string keyVaultUrl = "https://fl-test.vault.azure.net";
private static string supersecretname = "supersecretpassword";
```

## Simple program to retrieve secret
```csharp
static async Task Main(string[] args)
{
    var tokenProvider = new AzureServiceTokenProvider();
    var keyVaultClient = new KeyVaultClient(
               new KeyVaultClient.AuthenticationCallback(tokenProvider.KeyVaultTokenCallback));
    var secretValue = await keyVaultClient.GetSecretAsync(keyVaultUrl, supersecretname);

    Console.WriteLine(secretValue.Value);
}

```

## Change the secret in the CLI
```
az keyvault secret set `
    --name supersecretpassword `
    --vault-name fl-test `
    --value OXFORD `
    --not-before 2019-12-24T14:30:25z `
    --expires 2019-12-31T23:59:59z `
    --disabled false
```


## See all the versions
```csharp
static async Task Main(string[] args)
{
    var tokenProvider = new AzureServiceTokenProvider();
    var keyVaultClient = new KeyVaultClient(
                 new KeyVaultClient.AuthenticationCallback(tokenProvider.KeyVaultTokenCallback));
    
    var secretVersions = await keyVaultClient.GetSecretVersionsAsync(keyVaultUrl, supersecretname);

    foreach(var secretVersion in secretVersions)
    {
        var secretVersionValue = await keyVaultClient.GetSecretAsync(secretVersion.Id);
        Console.WriteLine($"{secretVersion.Identifier} | {secretVersion.Attributes.Enabled} | {secretVersion.Attributes.NotBefore} | {secretVersion.Attributes.Expires} | {secretVersionValue.Value}" );
    }  
}
```

## Try it in ASP.Net Core
```
dotnet new mvc
dotnet add package Microsoft.Azure.KeyVault
dotnet add package Microsoft.Azure.Services.AppAuthentication
dotnet add package Microsoft.Extensions.Configuration.AzureKeyVault
```

In `program.cs` change:
```csharp
public static IHostBuilder CreateHostBuilder(string[] args) =>
    Host.CreateDefaultBuilder(args)
        .ConfigureWebHostDefaults(webBuilder =>
        {
            webBuilder.UseStartup<Startup>();
        })
        .ConfigureAppConfiguration((context, config) => {
            config.AddAzureKeyVault("https://fl-test.vault.azure.net",
                                    new KeyVaultClient(
                                        new KeyVaultClient.AuthenticationCallback(
                                            new AzureServiceTokenProvider().KeyVaultTokenCallback)),
                                    new DefaultKeyVaultSecretManager());
        });

```

In `Home/index.cshtml` add:
```
@using Microsoft.Extensions.Configuration
@inject IConfiguration Configuration
@{
   string myPassword = Configuration["supersecretpassword"];
}

<h2>Retrieved from Key Vault:</h2>
<h1>@myPassword</h1>
```
*Note: Directly injecting and using config in the view like this is a **bad** idea. Don't do this in the real world please.*

# Do it on Azure
This part is more about deploying to Azure and setting up Managed Identity

## Set up git on the repository to make it easy
```
git init 
echo "bin/" | out-file .gitignore -encoding utf8 
echo "obj/" | out-file .gitignore -encoding utf8 -append 
git add -A
git commit -m "Initial commit"
```

## Create a web app slot
*Note: this won't work in PowerShell because of the pipe symbol in "DOTNETCORE|3.0"! You need to run the `webapp create` command in a command prompt or bash*
```
az appservice plan create --name fl-test-keyvault-secrets --sku S1 --is-linux  -o json
az webapp create --name fl-test-keyvault-secrets --plan fl-test-keyvault-secrets --runtime "DOTNETCORE|3.0" --deployment-local-git -o json
```

## Enable Managed Identity and give it access to the key vault
```
az webapp identity assign --name fl-test-keyvault-secrets

az keyvault set-policy `
    --name fl-test `
    --object-id f0e46c32-3492-407c-b31f-3e137e378ea7 `
    --secret-permissions get, list  
```


## Add azure as a git remote so we can push to it
The first line gets the publishing username and password for this webapp, which you'll be prompted for when you `git push`.
```
az webapp deployment list-publishing-credentials --name fl-test-keyvault-secrets -o json
git remote add azure https://fl-test-keyvault-secrets.scm.azurewebsites.net/fl-test-keyvault-secrets.git
git push -u azure master
```

## Check the result
```
az webapp browse --name fl-test-keyvault-secrets
```


[![Azure Advent Calendar logo](/assets/Advent-Calendar7.jpg)](https://azureadventcalendar.com/)