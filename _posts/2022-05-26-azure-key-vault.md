---

layout: single
title:  "Exploring Azure Key Vault"
date:   2022-05-26 10:59:04 +0530
categories: Cloud
tags: Azure
show_date: true
classes: wide
header:
  teaser: /assets/images/ms_build_cloud_skills.svg
author:
  name     : "Microsoft"
  avatar   : "/assets/images/azure.png"

sidebar:
  - title: "Blog"
   
    text: "Checkout other topics"
    nav: my-sidebar

---
In this post, let us look at how Azure Key Vault can help us keep our apps more secure, and how to set and retrieve secrets by using the Azure CLI.

Azure Key Vault is a cloud service for securely storing and accessing secrets. A secret is anything that you want to tightly control access to, such as API keys, passwords, certificates, or cryptographic keys.

The Azure Key Vault service supports two types of containers: vaults and managed hardware security module(HSM) pools.

Vaults support storing software and HSM-backed keys, secrets, and certificates. 
Managed HSM pools only support HSM-backed keys.

Azure Key Vault has two service tiers: Standard, which encrypts with a software key, and a Premium tier, which includes hardware security module(HSM)-protected keys. 

Key benefits of using Azure Key Vault

Centralized application secrets
Securely store secrets and keys
Monitor access and use
Simplified administration of application secrets

To do any operations with Key Vault, you first need to authenticate to it.

There are three ways to authenticate to Key Vault:
Managed identities for Azure resources
Service principal and certificate
Service principal and secret


Azure Key Vault enforces Transport Layer Security (TLS) protocol to protect data when it’s traveling between Azure Key Vault and clients. 

Authentication with Key Vault works in conjunction with Azure Active Directory, which is responsible for authenticating the identity of any given security principal.

Let us use Azure CLI and Create a Key Vault, Add and retrieve a secret.

Let's set some variables for the CLI commands to use to reduce the amount of retyping.
The Key Vault name needs to be a globally unique name, and the script below generates a random string.

```sh
pradeep:~$myKeyVault=az204vault-$RANDOM
pradeep:~$myLocation=southindia
pradeep:~$
```
Create a resource group.

```sh
pradeep:~$az group create --name az204-vault-rg --location $myLocation
{
  "id": "/subscriptions/f3f98f91-9191-4dd5-a561-cc5b31d0c201/resourceGroups/az204-vault-rg",
  "location": "southindia",
  "managedBy": null,
  "name": "az204-vault-rg",
  "properties": {
    "provisioningState": "Succeeded"
  },
  "tags": null,
  "type": "Microsoft.Resources/resourceGroups"
}
pradeep:~$
```

Create a Key Vault by using the `az keyvault create` command.
```sh
pradeep:~$az keyvault create --name $myKeyVault --resource-group az204-vault-rg --location $myLocation
 / Running ..
```

After some time,
```sh
pradeep:~$az keyvault create --name $myKeyVault --resource-group az204-vault-rg --location $myLocation
{
  "id": "/subscriptions/f3f98f91-9191-4dd5-a561-cc5b31d0c201/resourceGroups/az204-vault-rg/providers/Microsoft.KeyVault/vaults/az204vault-8398",
  "location": "southindia",
  "name": "az204vault-8398",
  "properties": {
    "accessPolicies": [
      {
        "applicationId": null,
        "objectId": "279e475d-571c-4ecc-8947-c700ad100cbc",
        "permissions": {
          "certificates": [
            "all"
          ],
          "keys": [
            "all"
          ],
          "secrets": [
            "all"
          ],
          "storage": [
            "all"
          ]
        },
        "tenantId": "dc59789f-e59b-4d89-88eb-406a51421a63"
      }
    ],
    "createMode": null,
    "enablePurgeProtection": null,
    "enableRbacAuthorization": null,
    "enableSoftDelete": true,
    "enabledForDeployment": false,
    "enabledForDiskEncryption": null,
    "enabledForTemplateDeployment": null,
    "hsmPoolResourceId": null,
    "networkAcls": null,
    "privateEndpointConnections": null,
    "provisioningState": "Succeeded",
    "publicNetworkAccess": "Enabled",
    "sku": {
      "family": "A",
      "name": "standard"
    },
    "softDeleteRetentionInDays": 90,
    "tenantId": "dc59789f-e59b-4d89-88eb-406a51421a63",
    "vaultUri": "https://az204vault-8398.vault.azure.net/"
  },
  "resourceGroup": "az204-vault-rg",
  "systemData": {
    "createdAt": "2022-05-27T01:36:02.792000+00:00",
    "createdBy": "gaddepradeep@gmail.com",
    "createdByType": "User",
    "lastModifiedAt": "2022-05-27T01:36:02.792000+00:00",
    "lastModifiedBy": "gaddepradeep@gmail.com",
    "lastModifiedByType": "User"
  },
  "tags": {},
  "type": "Microsoft.KeyVault/vaults"
}
pradeep:~$
```

To add a secret to the vault, you just need to take a couple of additional steps.

Create a secret. Let's add a password that could be used by an app. The password will be called `DemoPassword` and will store the value of `pVHkk645JuEi` in it.

```sh
pradeep:~$az keyvault secret set --vault-name $myKeyVault --name "DemoPassword" --value "pVHkk645JuEi"
{
  "attributes": {
    "created": "2022-05-27T01:38:41+00:00",
    "enabled": true,
    "expires": null,
    "notBefore": null,
    "recoveryLevel": "Recoverable+Purgeable",
    "updated": "2022-05-27T01:38:41+00:00"
  },
  "contentType": null,
  "id": "https://az204vault-8398.vault.azure.net/secrets/DemoPassword/93d30392f1764c5b98646f9e18d42ac8",
  "kid": null,
  "managed": null,
  "name": "DemoPassword",
  "tags": {
    "file-encoding": "utf-8"
  },
  "value": "pVHkk645JuEi"
}
pradeep:~$
```

Use the `az keyvault secret show` command to retrieve the secret.
This command will return some JSON. The last line will contain the password in plain text.

```sh
pradeep:~$az keyvault secret show --name "DemoPassword" --vault-name $myKeyVault
{
  "attributes": {
    "created": "2022-05-27T01:38:41+00:00",
    "enabled": true,
    "expires": null,
    "notBefore": null,
    "recoveryLevel": "Recoverable+Purgeable",
    "updated": "2022-05-27T01:38:41+00:00"
  },
  "contentType": null,
  "id": "https://az204vault-8398.vault.azure.net/secrets/DemoPassword/93d30392f1764c5b98646f9e18d42ac8",
  "kid": null,
  "managed": null,
  "name": "DemoPassword",
  "tags": {
    "file-encoding": "utf-8"
  },
  "value": "pVHkk645JuEi"
}
pradeep:~$
```

So far, we have created a Key Vault, stored a secret, and retrieved it.

When you no longer need the resources in this exercise use the following command to delete the resource group and associated Key Vault.

```sh
pradeep:~$az group delete --name az204-vault-rg --no-wait
Are you sure you want to perform this operation? (y/n): y
pradeep:~$
```
