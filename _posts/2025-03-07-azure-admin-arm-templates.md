---

layout: single
title:  "Azure ARM Templates"
categories: Cloud
tags: Azure
show_date: true
classes: wide
header:
  teaser: /assets/images/az104.png
author:
  name     : "Microsoft"
  avatar   : "/assets/images/az104.png"

sidebar:
  - title: "Blog"
   
    text: "Checkout other topics"
    nav: my-sidebar

---

# Azure ARM Templates

Declarative Automation for Azure

```json
/arm-templates ➜  cat vnet-deployment-template.json 
{
    "$schema": "https://schema.management.azure.com/schemas/2019-04-01/deploymentTemplate.json#",
    "contentVersion": "1.0.0.0",
    "parameters": {},
    "functions": [],
    "variables": {},
    "resources": [
        {
            "name": "virtualNetwork1",
            "type": "Microsoft.Network/virtualNetworks",
            "apiVersion": "2023-11-01",
            "location": "[resourceGroup().location]",
            "tags": {
                "displayName": "virtualNetwork1"
            },
            "properties": {
                "addressSpace": {
                    "addressPrefixes": [
                        "10.10.10.0/24"
                    ]
                }
            }
        }
    ],
    "outputs": {
    }
}
~/arm-templates ➜  
```

```json
~/arm-templates ➜  cat vnet-deployment-template.json 
{
    "$schema": "https://schema.management.azure.com/schemas/2019-04-01/deploymentTemplate.json#",
    "contentVersion": "1.0.0.0",
    "parameters": {},
    "functions": [],
    "variables": {},
    "resources": [
        {
            "name": "arm-vnet-test",
            "type": "Microsoft.Network/virtualNetworks",
            "apiVersion": "2023-11-01",
            "location": "[resourceGroup().location]",
            "tags": {
                "displayName": "virtualNetwork1",
                "Envionment": "Prod"
            },
            "properties": {
                "addressSpace": {
                    "addressPrefixes": [
                        "192.168.0.0/16"
                    ]
                }
            }
        }
    ],
    "outputs": {
    }
}

~/arm-templates ➜  
```



```json
~/arm-templates ➜  echo $RESOURCE_GROUP_NAME
pradeep_rg_1


~/arm-templates ➜  az deployment group create --resource-group $RESOURCE_GROUP_NAME --template-file vnet-deployment-template.json
{
  "id": "/subscriptions/my-azure-subscription/resourceGroups/pradeep_rg_1/providers/Microsoft.Resources/deployments/vnet-deployment-template",
  "location": null,
  "name": "vnet-deployment-template",
  "properties": {
    "correlationId": "c6ca418c-4602-420a-aa67-3b622f885ab3",
    "debugSetting": null,
    "dependencies": [],
    "duration": null,
    "error": null,
    "mode": "Incremental",
    "onErrorDeployment": null,
    "outputResources": [
      {
        "id": "/subscriptions/my-azure-subscription/resourceGroups/pradeep_rg_1/providers/Microsoft.Network/virtualNetworks/arm-vnet-test",
        "resourceGroup": "pradeep_rg_1"
      }
    ],
    "outputs": {},
    "parameters": {},
    "parametersLink": null,
    "providers": [
      {
        "id": null,
        "namespace": "Microsoft.Network",
        "providerAuthorizationConsentState": null,
        "registrationPolicy": null,
        "registrationState": null,
        "resourceTypes": [
          {
            "aliases": null,
            "apiProfiles": null,
            "apiVersions": null,
            "capabilities": null,
            "defaultApiVersion": null,
            "locationMappings": null,
            "locations": [
              "westus"
            ],
            "properties": null,
            "resourceType": "virtualNetworks",
            "zoneMappings": null
          }
        ]
      }
    ],
    "provisioningState": "Succeeded",
    "templateHash": "10832151437961360248",
    "templateLink": null,
    "timestamp": "2025-03-23T06:20:28.621006+00:00",
    "validatedResources": null
  },
  "resourceGroup": "pradeep_rg_1",
  "tags": null,
  "type": "Microsoft.Resources/deployments"
}

~/arm-templates ➜  
```

If you notice, there was a typo in the Tag name Environment. What happens if you fix the typo and re-deploy the ARM template?



```json
~/arm-templates ➜  az deployment group create --resource-group $RESOURCE_GROUP_NAME --template-file vnet-deployment-template.json
{
  "id": "/subscriptions/my-azure-subscription/resourceGroups/pradeep_rg_1/providers/Microsoft.Resources/deployments/vnet-deployment-template",
  "location": null,
  "name": "vnet-deployment-template",
  "properties": {
    "correlationId": "bdff7e39-383e-4000-8f67-e6c19a9b5c29",
    "debugSetting": null,
    "dependencies": [],
    "duration": null,
    "error": null,
    "mode": "Incremental",
    "onErrorDeployment": null,
    "outputResources": [
      {
        "id": "/subscriptions/my-azure-subscription/resourceGroups/pradeep_rg_1/providers/Microsoft.Network/virtualNetworks/arm-vnet-test",
        "resourceGroup": "pradeep_rg_1"
      }
    ],
    "outputs": {},
    "parameters": {},
    "parametersLink": null,
    "providers": [
      {
        "id": null,
        "namespace": "Microsoft.Network",
        "providerAuthorizationConsentState": null,
        "registrationPolicy": null,
        "registrationState": null,
        "resourceTypes": [
          {
            "aliases": null,
            "apiProfiles": null,
            "apiVersions": null,
            "capabilities": null,
            "defaultApiVersion": null,
            "locationMappings": null,
            "locations": [
              "westus"
            ],
            "properties": null,
            "resourceType": "virtualNetworks",
            "zoneMappings": null
          }
        ]
      }
    ],
    "provisioningState": "Succeeded",
    "templateHash": "8222263890642150289",
    "templateLink": null,
    "timestamp": "2025-03-23T06:25:08.498612+00:00",
    "validatedResources": null
  },
  "resourceGroup": "pradeep_rg_1",
  "tags": null,
  "type": "Microsoft.Resources/deployments"
}

~/arm-templates ➜  
```

## How to Deploy an ARM Template via Azure Portal

### Step 1: Log in to the Azure Portal

1. Open your web browser and navigate to [Azure Portal](https://portal.azure.com/).
2. Sign in using the Azure account credentials provided for the lab.

### Step 2: Access the "Deploy a Custom Template" Feature

1. In the Azure Portal, use the search bar at the top to type "Deploy a custom template."
2. Select **"Deploy a custom template"** from the search results.

### Step 3: Create Your Template

1. You’ll be directed to the **"Custom deployment"** page.
2. Click on **"Build your own template in the editor"**.
3. We have prepared a template for you named `deploy-custom-template.json` located at `/root/arm-templates` Please copy and paste your ARM template JSON code into the provided editor.
4. Click **"Save"** to proceed.

### Step 4: Set Up the Deployment

1. On the next screen, fill in the required details:
   - **Subscription**: Select the Azure subscription to use. This will be auto-populated for you.
   - **Resource Group**: Choose an existing resource group with a name starting with pradeep_rg_`.
   - **Region**: Select the region where your resources  will be deployed. The system will auto-suggest and fill in this field  for you, but make sure to verify it to avoid deployment failures.
2. If your template includes parameters, you will be prompted to  enter values. For instance, provide inputs like resource names,  locations, or SKU types.

**Note**: One parameter for the Storage Account is predefined.

- **StorageAccountName**: Choose a name for the storage account in the format `<anyRandomNumber>`.

### Step 5: Review and Validate Your Template

1. After completing all the required fields, click **"Review + create"**.
2. Azure will validate your template. If there are any issues or missing fields, you’ll receive a notification to make corrections.

### Step 6: Deploy Your Template

1. Once the validation is successful, a summary of your deployment settings will appear.
2. Click **"Create"** to initiate the deployment.
3. Azure will start deploying the resources as specified in your ARM template.

### Step 7: Monitor the Deployment

1. You will be taken to the deployment status page, where you can track the progress of your deployment.
2. After the deployment is complete, you’ll receive a notification, and the status will show as **"Succeeded"**.

### Step 8: Verify the Deployment

1. After the deployment, navigate to the resource group in the Azure Portal to ensure all resources have been deployed correctly.
2. Finally, click on the **"Check"** button to complete the process.

This streamlined process allows you to deploy ARM templates directly  through the Azure Portal, providing an easy way to manage and monitor  your deployments.







