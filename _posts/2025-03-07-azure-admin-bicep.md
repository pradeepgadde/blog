---

layout: single
title:  "Azure Bicep"
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

# Azure Bicep

### Schema Declaration

- **ARM Template**: Requires a $schema declaration to define the structure of the template 
- **Bicep Template** : Does not require a schema declaration - it automatically understands the structure

### Parameters Definition

- **ARM Template** : Parameters are defined in JSON with a type and optional default value.
  
- **Bicep Template** : Parameters are defined in a simpler, more concise syntax.

### **Resource Declaration**

- **ARM Template**: Resources are defined using JSON with explicit API versions, resource types, and properties.
- **Bicep Template**: Resources are defined more cleanly with a simplified structure and automatic handling of resource references.

**Key Difference**: 

- **Dependencies**: In ARM templates, dependencies are manually specified using `dependsOn`. Bicep automatically resolves dependencies based on resource references, making the code simpler.
- **Resource Names**: ARM uses the `format` function for string interpolation, while Bicep uses a more intuitive `${}` syntax.

### **Security Parameters**

- **ARM Template**: Secure parameters are specified using the `"securestring"` type.
- **Bicep Template**: Secure parameters are marked with the `@secure()` decorator, which is more straightforward.

### **Summary of Differences**

- **Readability**: Bicep templates are more readable and concise compared to ARM templates, which can be verbose and complex.
- **Simplicity**: Bicep abstracts much of the complexity  involved in writing ARM templates, making it easier to define and manage Azure resources.
- **Automatic Dependency Management**: Bicep automatically handles dependencies, reducing the risk of errors and making the code cleaner.

# Basic Bicep Template Example: 

```json
@description('Name of the storage account')
param storageAccountName string

@description('Location for all resources')
param location string = resourceGroup().location

@description('The SKU of the storage account')
param skuName string = 'Standard_LRS'

resource storageAccount 'Microsoft.Storage/storageAccounts@2022-09-01' = {
  name: storageAccountName
  location: location
  sku: {
    name: skuName
  }
  kind: 'StorageV2'
  properties: {}
}

output storageAccountName string = storageAccount.name
```



```json
resource appnetwork 'Microsoft.Network/virtualNetworks@2022-07-01' = [ for i in range(1, 3): {
  name: 'bicep-demo-vnet-${i}'
  location: resourceGroup().location
  properties: {
    addressSpace: {
      addressPrefixes: [
        '10.10.0.0/16'
      ]
    }
  }
}]
```



- **Dynamic Naming**: The use of the loop index `i` allows for dynamic creation of resource names, ensuring uniqueness within the deployment.
- **Scalability**: By simply changing the range, you can  increase or decrease the number of storage accounts deployed without  having to manually repeat code.
- **Consistency**: The location, kind, and SKU properties are consistent across all storage accounts, making the deployment  predictable and uniform.

This approach is efficient for deploying multiple resources with  slight variations, such as different names, while keeping the rest of  the properties consistent.