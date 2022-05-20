---

layout: single
title:  "Azure Virtual Network Settings"
date:   2022-05-20 04:59:04 +0530
categories: Cloud
tags: Azure
show_date: true
classes: wide
header:
  teaser: /assets/images/microsoft-certified-fundamentals-badge.svg
author:
  name     : "Microsoft"
  avatar   : "/assets/images/azure.png"

sidebar:
  - title: "Blog"
   
    text: "Checkout other topics"
    nav: my-sidebar

---
In this post, let us take a look at configuring Azure Virtual Networks.

You can create and configure Azure virtual networks from the Azure portal, Azure PowerShell, Azure CLI, Azure Cloud Shell or an ARM template.

Here is how we create Azure Virtual Networks.

When you create an Azure virtual network, you configure a number of basic settings. You'll have the option to configure advanced settings, such as multiple subnets, distributed denial of service (DDoS) protection, Bastion host, Azure firewall, service endpoints, and use of a NAT gateway.



When you set up a virtual network, you define the internal address space in Classless Interdomain Routing (CIDR) format. This address space needs to be unique within your subscription and any other networks that you connect to.

Within each virtual network address range, you can create one or more subnets that partition the virtual network's address space. Routing between subnets will then depend on the default traffic routes. You also can define custom routes. Alternatively, you can define one subnet that encompasses all the virtual networks' address ranges.

![Azure VN]({{ site.url }}{{ site.baseurl }}/assets/images/azure-vn-1.png)
![Azure VN]({{ site.url }}{{ site.baseurl }}/assets/images/azure-vn-2.png)
![Azure VN]({{ site.url }}{{ site.baseurl }}/assets/images/azure-vn-3.png)
![Azure VN]({{ site.url }}{{ site.baseurl }}/assets/images/azure-vn-4.png)
![Azure VN]({{ site.url }}{{ site.baseurl }}/assets/images/azure-vn-5.png)

![Azure VN]({{ site.url }}{{ site.baseurl }}/assets/images/azure-vn-6.png)
![Azure VN]({{ site.url }}{{ site.baseurl }}/assets/images/azure-vn-7.png)

Azure Bastion service provides a secure and seamless RDP/SSH connectivity to your virtual machines directly in the Azure portal over SSL. 

 You can configure a subnet to use a static outbound IP address when accessing the internet.

Service Endpoint Options include Azure Cosmos DB, Azure Service Bus, Azure Key Vault, and so on.

Azure Firewall service is managed cloud-based network security service that protects your Azure Virtual Network resources. 

Network security groups have security rules that enable you to filter the type of network traffic that can flow in and out of virtual network subnets and network interfaces. You create the network security group separately. Then you associate it with each subnet in the virtual network.

Azure automatically creates a route table for each subnet within an Azure virtual network and adds system default routes to the table. You can add custom route tables to modify traffic between subnets and virtual networks.



After you've created a virtual network, you can change any further settings on the **Virtual network** pane in the Azure portal. Alternatively, you can use PowerShell commands or commands in Cloud Shell to make changes.

Here is a template for automating Virtual Network Creation.

> parametes.json

```json
{
    "$schema": "https://schema.management.azure.com/schemas/2015-01-01/deploymentParameters.json#",
    "contentVersion": "1.0.0.0",
    "parameters": {
        "location": {
            "value": "southeastasia"
        },
        "extendedLocation": {
            "value": {}
        },
        "virtualNetworkName": {
            "value": "VN-1"
        },
        "resourceGroup": {
            "value": "Pradeep-VN"
        },
        "addressSpaces": {
            "value": [
                "10.0.0.0/16",
                "192.168.1.0/24"
            ]
        },
        "ipv6Enabled": {
            "value": false
        },
        "subnetCount": {
            "value": 1
        },
        "subnet0_name": {
            "value": "default"
        },
        "subnet0_addressRange": {
            "value": "10.0.0.0/24"
        },
        "ddosProtectionPlanEnabled": {
            "value": false
        },
        "firewallEnabled": {
            "value": false
        },
        "bastionEnabled": {
            "value": false
        }
    }
}
```

> tempate.json
```json
{
    "$schema": "http://schema.management.azure.com/schemas/2015-01-01/deploymentTemplate.json#",
    "contentVersion": "1.0.0.0",
    "parameters": {
        "location": {
            "type": "string"
        },
        "extendedLocation": {
            "type": "object"
        },
        "virtualNetworkName": {
            "type": "string"
        },
        "resourceGroup": {
            "type": "string"
        },
        "addressSpaces": {
            "type": "array"
        },
        "ipv6Enabled": {
            "type": "bool"
        },
        "subnetCount": {
            "type": "int"
        },
        "subnet0_name": {
            "type": "string"
        },
        "subnet0_addressRange": {
            "type": "string"
        },
        "ddosProtectionPlanEnabled": {
            "type": "bool"
        },
        "firewallEnabled": {
            "type": "bool"
        },
        "bastionEnabled": {
            "type": "bool"
        }
    },
    "variables": {},
    "resources": [
        {
            "name": "[parameters('virtualNetworkName')]",
            "type": "Microsoft.Network/VirtualNetworks",
            "apiVersion": "2021-01-01",
            "location": "[parameters('location')]",
            "extendedLocation": "[if(empty(parameters('extendedLocation')), json('null'), parameters('extendedLocation'))]",
            "dependsOn": [],
            "tags": {},
            "properties": {
                "addressSpace": {
                    "addressPrefixes": [
                        "10.0.0.0/16",
                        "192.168.1.0/24"
                    ]
                },
                "subnets": [
                    {
                        "name": "default",
                        "properties": {
                            "addressPrefix": "10.0.0.0/24"
                        }
                    }
                ],
                "enableDdosProtection": "[parameters('ddosProtectionPlanEnabled')]"
            }
        }
    ]
}
```
