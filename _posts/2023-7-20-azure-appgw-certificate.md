---		
title: 'Importing RBAC controlled certificate to Azure Application Gateway'
classes: wide
---
I have a scenario where I want to use certificates, stored within an RBAC controlled Azure Key Vault, in an Application Gateway.  Unfortunately, at this time, there is no real way to do this from within the portal.  The configuration has to be done from Azure Powershell.  

So, this post is mostly a reminder to myself on where to go and how to do it.  Specifically, the documentation to do this is found in [this article](https://learn.microsoft.com/en-us/azure/application-gateway/key-vault-certs#key-vault-azure-role-based-access-control-permission-model).  

I'll add the code example below:

```
# Get the Application Gateway we want to modify
$appgw = Get-AzApplicationGateway -Name MyApplicationGateway -ResourceGroupName MyResourceGroup
# Specify the resource id to the user assigned managed identity - This can be found by going to the properties of the managed identity
Set-AzApplicationGatewayIdentity -ApplicationGateway $appgw -UserAssignedIdentityId "/subscriptions/xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx/resourceGroups/MyResourceGroup/providers/Microsoft.ManagedIdentity/userAssignedIdentities/MyManagedIdentity"
# Get the secret ID from Key Vault
$secret = Get-AzKeyVaultSecret -VaultName "MyKeyVault" -Name "CertificateName"
$secretId = $secret.Id.Replace($secret.Version, "") # Remove the secret version so AppGW will use the latest version in future syncs
# Specify the secret ID from Key Vault 
Add-AzApplicationGatewaySslCertificate -KeyVaultSecretId $secretId -ApplicationGateway $appgw -Name $secret.Name
# Commit the changes to the Application Gateway
Set-AzApplicationGateway -ApplicationGateway $appgw
```