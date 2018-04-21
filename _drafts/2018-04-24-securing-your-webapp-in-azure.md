So you have deployed your web app to Azure. Now, how do you go about making it secure?

# The scenario
For the purposes of this, we will use a .Net Core ASP.Net app which uses Azure SQL and Blob storage.  
In practice I often use CosmosDb over SQL, but Azure SQL has many more security features than CosmosDb. It's not that Cosmos is a slouch, it's just that Azure SQL is pretty amazing security-wise.

I am going to list a lot of technologies in a short space of time. Most of these could take up a whole talk on their own, so I am aiming for a whirl-wind tour. There are certain additional layers I won't go into; the focus is on that simple web app and what protections you can wrap around it.

# Approach
- Prevent
- Detect
- Mitigate

I do not have a lot to say about the last one.

# Preventing
## Make your app code as safe as you can
-  Remove unnecessary headers
## Use SSL
- Let's Encrypt
- The built-in one in Azure
- cloudflare
## Encrypt data at rest
- Tickbox for SQL (Cosmos is automatic these days)
- Tickbox for Storage (only accounts created in the last couple of years)
## Encrypt extra sensitive data at the application layer
- SQL Always Encrypted
  - Alternatively use Dynamic Data Masking
- Storage SDK encryption
## Hide connection strings and secrets using KeyVault
## Use Managed Principal to access key vault etc
## Use Azure AD to lock down acess to SQL and Azure itself
## Set up a VNet and put your database and storage behind it
## WAF
Is debatable. But if you want one, just don't use Azure's.
## Security Center and SQL Security assessment thing

# Detecting
## Azure SQL Threat detection
## Application Insights
I like to use Serilog and then wire that up to App Insights. You don't need to, I mainly do it here because I am familiar with it.
## Log triggers
- Failed logins
- 404s and 403s
- Non-app users accessing the database