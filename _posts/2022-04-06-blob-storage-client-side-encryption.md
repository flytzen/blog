---
layout: post
title: Blob Storage Client-Side Encryption
date: '2022-04-06'
author: Frans Lytzen
tags: Azure
excerpt: Transparently encrypt blobs in Azure Blob Storage at the Application level with client-side encryption.
---
# Blob encryption

Azure provides disk encryption and optional infrastructure encryption, so you can easily comply with "encryption at rest" - at least from a cursory compliance level. But, anyone who has access to your Storage Account can read the blobs.
Client-side encryption is built into the Azure Blob Storage SDK. When configured, this will transparently encrypt and decrypt the blob at the application level: You don't need to change any of the code that reads and writes blobs at all - and your application continues to work as before. But, if you log into the Azure portal and try to download the blob, you will just see encrypted data.

In this post, I will explain how this works and show you how to implement it in C# with Azure Key Vault. I am using the Storage SDK 12.x - for which the encryption [documentation](https://docs.microsoft.com/en-us/azure/storage/blobs/storage-encrypt-decrypt-blobs-key-vault?tabs=dotnet) is still not updated at the time of the writing - and the Key Vault SDK 4.3.x.

# How does this work

You *can* implement this without Key Vault, but that is hard to do securely, so I will only focus on the Key Vault method here.

Firstly, all of what I am about to describe is transparently handled by the Blob Storage SDK and Key Vault, once you have wired it up.  
Secondly, I need to introduce a couple of concepts before I can explain how the Storage SDK works its magic.

## Symmetric key encryption

Symmetric key encryption means that the same key is used to encrypt and decrypt the data. It is very fast, and it scales really well to large files. Content can be encrypted and decrypted as it is being streamed, without loading everything into memory, and there is only a negligible size overhead (less than 16 bytes).  
AES is an example of symmetric key encryption.

The problem with symmetric key encryption is that you need to keep that key somewhere safe. What's worse is that if you encrypt lots of different files with the same symmetric key, an attacker may have a chance of identifying the symmetric key used. The details of this are way beyond the scope of this article.

Ideally, you want a unique, randomly created symmetric key for *each* blob you store - but then where are you going to store all those keys?

## Asymmetric Encryption

RSA encryption is a kind of Asymmetric Encryption, is based on factors of large prime numbers and is very secure. It also has the benefit that you use the public key to encrypt the data and the private key to decrypt it. With Azure Key Vault, your code can read the Public key and use it to do encryption locally, but your code cannot read the private key and must send the encrypted content to Key Vault for decryption (I am not sure if the Blob SDK takes advantage of this or not, but you can roll it yourself for high-performance scenarios if needed).  

The problem with RSA is that it is very slow and doesn't scale - the C# implementation is limited to encrypting a maximum of 160 bytes. You do not want to encrypt whole files with RSA.

*There are other asymmetric encryption systems, but for the scope of this post I shall stick to RSA.*

## Envelope technique

When you save a Blob with Client-Side Encryption in Azure blob storage, the SDK will create a random, symmetric key (the Content Encryption Key or CEK) and will encrypt the content of the blob with that key.  
Next, the SDK will use Key Vault to encrypt the CEK, using an RSA key you have created in the Key Vault. Unless you have a specific reason to do otherwise, create an RSA 2048 key.

![Create a Key](/assets/createkey.png)

The SDK then stores the encrypted CEK in a Meta Tag field on the blob called `encryptiondata`. It also stores some additional data, most importantly the unique identifier for the RSA key that was used to encrypt the symmetric key.

```json
{
    "EncryptionMode":"FullBlob",
    "WrappedContentKey":
        {
            "KeyId":"https://xxx.vault.azure.net/keys/BlobEncryption/7e78c2c3164f458ba4884d3bed00925f",
            "EncryptedKey":"FgPb/2nELt5N3mzVIjD\u002BKGVmEgpO - XXXXXXX - 7mDINXvgaVw==",
            "Algorithm":"RSA-OAEP"
        },
    "EncryptionAgent":
        {
            "Protocol":"1.0",
            "EncryptionAlgorithm":"AES_CBC_256"
        },
    "ContentEncryptionIV":"ZA3An4jhVIV8uhRJhlPJKQ==",
    "KeyWrappingMetadata":
        {
            "EncryptionLibrary":"azsdk-net-Azure.Storage.Blobs/12.11.0\u002B594b8851940eb859860cff9aaaafbbb092df59d6 (.NET 5.0.15; Linux 5.10.16.3-microsoft-standard-WSL2 #1 SMP Fri Apr 2 22:23:49 UTC 2021)"
        }
}
```

When the SDK later *reads* the blob, it also retrieves the `encryptiondata` meta tag. Armed with that information, the SDK then asks Key Vault to decrypt the encrypted Content Encryption Key and then uses that symmetric key to decrypt the contents of the blob.

This is known as the "envelope technique". It is a tried and tested technique that combines the best features of AES and RSA encryption.

## Key Versioning or Rotation

It is common practice to "rotate" encryption keys to reduce the impact of a key being leaked. If all your blobs were encrypted with the same key and that key was leaked, all your files would be vulnerable. But if you change the key once in a while, the attacker will only get access to a limited number of blobs.

Each blob has its own, individual symmetric encryption key, so there is nothing to "rotate" there. If you want to change the symmetric key for a particular blob for some reason, you need to read it in code and then re-save it.

But what about that RSA key? If an attacker gets hold of that, they can decrypt the symmetric key for each of your blobs and then read them. Fortunately, Key Vault has a concept of "versions" of a key, and even has a new "key rotation policy" in preview, which automatically creates a new version at a set interval.
Each "version" of a key is really a completely different key. But the mechanism means you can ask your Key Vault for the latest version of a specific key name and you will get a Key Identifier that includes a version number. That key can then be used to encrypt the data in the blob and the full Key Identifier, including the version number, is stored in the `encryptiondata` meta tag. 

When it comes time to read the blob, the Blob SDK reads the exact key version that was used to encrypt the content encryption key from the metadata. As a matter of fact, the Key Identifier includes the address of the Key Vault so if you need to use more than Key Vault for the same Blob Storage account, for example during a migration, you can.

![Key Versions](/assets/keyversions.png)

# Implementing this in C#

For this example I am using the following packages:
```
    <PackageReference Include="Azure.Identity" Version="1.5.0" />
    <PackageReference Include="Azure.Security.KeyVault.Keys" Version="4.3.0" />
    <PackageReference Include="Azure.Storage.Blobs" Version="12.11.0" />
```

Azure Identity is only included so I can use Managed Identity to connect to Key Vault and Blob Storage. You do not need to do that; a Connection String for Blob Storage and a Service Principal for Key Vault will work just as well.

## Permissions

In this example, I am using Managed Identity to access both Key Vault and Blob Storage. 
For Key Vault, if you are using the RBAC roles, the "user" needs at least the `Key Vault Crypto User` role.
In order to create and rotate keys, you need the `Key Vault Crypto Officer` role (do not give that to your application).

In Blob Storage SDK 12, most of what you do starts with getting a `BlobServiceClient`. The method below creates a `BlobServiceClient` that will transparently encrypt and decrypt data as blobs are written and read. 

You should probably set this up to be a Singleton in your DI. The only thing to be mindful of is that if you want to use Key Rotation (and you should) then you should re-create `BlobServiceClient` once in a while - on a timescale that is similar to your key rotation timescale. The reason is that the *encryption* key is only read during the creation of the `BlobServiceClient`. The old version of the key will continue to work even after a new version is created (unless you explicitly disable it in Key Vault), so you don't have to be very precise about the timings. But, you do obviously want to start using the new key version at some point, otherwise the whole rotation thing is pointless.

```csharp
private BlobServiceClient GetBlobServiceClient()
{
    var keyVaultUri = this.configuration.GetValue<string>("KeyVaultUri");
    var keyIdentifier = this.configuration.GetValue<string>("BlobEncryptionKeyName");
    var azureCredentials = new DefaultAzureCredential();

    // NOTE: This reads the current key version and uses that to encrypt Content Encryption Keys going forward
    // Problem is that when the key gets rotated, at some point you should update the key being used
    // So, you do need to refresh this occasionally!
    var keyClient = new KeyClient(new Uri(keyVaultUri), azureCredentials);
    var key = keyClient.GetKey(keyIdentifier);  // this gets the current version of the named key

    // The cryptoClient is used to encrypt the Content Encryption Key
    var cryptoClient = new CryptographyClient(key.Value.Id, azureCredentials);
    // The keyResolver is the thing that can be given a Key Identifier and get a reference to the RSA key so the CEK can be decrypted
    var keyResolver = new KeyResolver(new DefaultAzureCredential());

    // This and the next bit just sets up the configuration options to be used when creating the BlobServiceClient
    var encryptionOptions = new ClientSideEncryptionOptions(ClientSideEncryptionVersion.V1_0)
    {
        KeyEncryptionKey = cryptoClient,
        KeyResolver = keyResolver,
        KeyWrapAlgorithm = "RSA-OAEP"  // There are other options, use this for an RSA 2048 key
    };

    var blobClientOptions = new SpecializedBlobClientOptions()
    {
        ClientSideEncryption = encryptionOptions
    };

    // Finally, create the BlobServiceClient with the configuration options to use encryption.
    var blobServiceUrl = this.configuration.GetValue<string>("BlobServiceUri");
    return new BlobServiceClient(new Uri(blobServiceUrl), azureCredentials, blobClientOptions);
}

```


# Afterthought

[NewOrbit](https://www.neworbit.co.uk/azure) is an Azure Gold Partner and Azure Reseller ("Direct CSP") as well as development house. We help other development companies to get more out of Azure, with a particular focus on reducing costs and making systems more secure.
If you would like to buy your Azure from people who design and develop systems on Azure every day, give us a [shout](https://neworbit.co.uk/#contact) or ping me on [Twitter](https://twitter.com/flytzen). We usually give you a "trial", in the form of a Cost, Infrastructure or Security review so you can see if we can help you and if you like working with us.
