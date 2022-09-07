---
layout: single
title:  "Terraform Input Vairables"
date:   2022-09-07 08:58:04 +0530
categories: Automation
tags: Terraform
show_date: true
classes: wide
header:
  teaser: /assets/images/terraform.png
author:
  name     : "Terraform"
  avatar   : "/assets/images/terraform.png"

sidebar:
  - title: "Blog"
   
    text: "Checkout other topics"
    nav: my-sidebar

---

So far we have used hard-coded values in the  Terraform configuration. We can include variables to make our configuration more dynamic and flexible.

create a new file called `variables.tf`

```sh
(base) pradeep:~$cat variables.tf 
variable "resource_group_name" {
  default = "myTFResourceGroup"
}


(base) pradeep:~$
```

This declaration includes a default value for the variable, so the `resource_group_name` variable will not be a required input.

Update your `azurerm_resource_group` configuration to use the input variable for the resource group name.  Use `var.resource_group_name` for the name of the resource group instead of hard-coding.

```sh
(base) pradeep:~$cat main.tf 
# Configure the Azure provider
terraform {
  required_providers {
    azurerm = {
      source  = "hashicorp/azurerm"
      version = "~> 3.0.2"
    }
  }

  required_version = ">= 1.1.0"
}

provider "azurerm" {
  features {}
}

resource "azurerm_resource_group" "rg" {
  name     = var.resource_group_name
  location = "westus2"
  tags = {
    Environment = "Pradeep's Terraform Getting Started"
    Team = "PradeepDevOps"
  }
}

# Create a virtual network
resource "azurerm_virtual_network" "vnet" {
  name                = "myTFVnet"
  address_space       = ["10.0.0.0/16"]
  location            = "westus2"
  resource_group_name = azurerm_resource_group.rg.name
}
(base) pradeep:~$
```

Re-run `terraform apply` to recreate your infrastructure using input variables. 

```sh
(base) pradeep:~$terraform apply

Terraform used the selected providers to generate the following execution plan. Resource actions are indicated with the following symbols:
  + create

Terraform will perform the following actions:

  # azurerm_resource_group.rg will be created
  + resource "azurerm_resource_group" "rg" {
      + id       = (known after apply)
      + location = "westus2"
      + name     = "myTFResourceGroup"
      + tags     = {
          + "Environment" = "Pradeep's Terraform Getting Started"
          + "Team"        = "PradeepDevOps"
        }
    }

  # azurerm_virtual_network.vnet will be created
  + resource "azurerm_virtual_network" "vnet" {
      + address_space       = [
          + "10.0.0.0/16",
        ]
      + dns_servers         = (known after apply)
      + guid                = (known after apply)
      + id                  = (known after apply)
      + location            = "westus2"
      + name                = "myTFVnet"
      + resource_group_name = "myTFResourceGroup"
      + subnet              = (known after apply)
    }

Plan: 2 to add, 0 to change, 0 to destroy.

Do you want to perform these actions?
  Terraform will perform the actions described above.
  Only 'yes' will be accepted to approve.

  Enter a value: yes

azurerm_resource_group.rg: Creating...
azurerm_resource_group.rg: Creation complete after 3s [id=/subscriptions/xxxxxx-xxxx-xxxx-xxxx-xxxxxxxxx/resourceGroups/myTFResourceGroup]
azurerm_virtual_network.vnet: Creating...
azurerm_virtual_network.vnet: Still creating... [10s elapsed]
azurerm_virtual_network.vnet: Creation complete after 10s [id=/subscriptions/xxxxxx-xxxx-xxxx-xxxx-xxxxxxxxx/resourceGroups/myTFResourceGroup/providers/Microsoft.Network/virtualNetworks/myTFVnet]

Apply complete! Resources: 2 added, 0 changed, 0 destroyed.
(base) pradeep:~$
```


Now apply the configuration again, this time overriding the default resource group name by passing in a variable using the `-var` flag. 

Updating the resource group name is a destructive update that forces Terraform to recreate the resource, and in turn the virtual network that depends on it. Respond to the confirmation prompt with yes to rename the resource group and create the new resources.

```sh
(base) pradeep:~$terraform apply -var "resource_group_name=PradeepNewResourceGroup"
azurerm_resource_group.rg: Refreshing state... [id=/subscriptions/xxxxxx-xxxx-xxxx-xxxx-xxxxxxxxx/resourceGroups/myTFResourceGroup]
azurerm_virtual_network.vnet: Refreshing state... [id=/subscriptions/xxxxxx-xxxx-xxxx-xxxx-xxxxxxxxx/resourceGroups/myTFResourceGroup/providers/Microsoft.Network/virtualNetworks/myTFVnet]

Terraform used the selected providers to generate the following execution plan. Resource actions are indicated with the following symbols:
-/+ destroy and then create replacement

Terraform will perform the following actions:

  # azurerm_resource_group.rg must be replaced
-/+ resource "azurerm_resource_group" "rg" {
      ~ id       = "/subscriptions/xxxxxx-xxxx-xxxx-xxxx-xxxxxxxxx/resourceGroups/myTFResourceGroup" -> (known after apply)
      ~ name     = "myTFResourceGroup" -> "PradeepNewResourceGroup" # forces replacement
        tags     = {
            "Environment" = "Pradeep's Terraform Getting Started"
            "Team"        = "PradeepDevOps"
        }
        # (1 unchanged attribute hidden)
    }

  # azurerm_virtual_network.vnet must be replaced
-/+ resource "azurerm_virtual_network" "vnet" {
      ~ dns_servers             = [] -> (known after apply)
      - flow_timeout_in_minutes = 0 -> null
      ~ guid                    = "xxxxxx-xxxx-xxxx-xxxx-xxxxxxxxx" -> (known after apply)
      ~ id                      = "/subscriptions/xxxxxx-xxxx-xxxx-xxxx-xxxxxxxxx/resourceGroups/myTFResourceGroup/providers/Microsoft.Network/virtualNetworks/myTFVnet" -> (known after apply)
        name                    = "myTFVnet"
      ~ resource_group_name     = "myTFResourceGroup" -> "PradeepNewResourceGroup" # forces replacement
      ~ subnet                  = [] -> (known after apply)
      - tags                    = {} -> null
        # (2 unchanged attributes hidden)
    }

Plan: 2 to add, 0 to change, 2 to destroy.

Do you want to perform these actions?
  Terraform will perform the actions described above.
  Only 'yes' will be accepted to approve.

  Enter a value: yes

azurerm_virtual_network.vnet: Destroying... [id=/subscriptions/xxxxxx-xxxx-xxxx-xxxx-xxxxxxxxx/resourceGroups/myTFResourceGroup/providers/Microsoft.Network/virtualNetworks/myTFVnet]
azurerm_virtual_network.vnet: Still destroying... [id=/subscriptions/f3f98f91-9191-4dd5-a561-...osoft.Network/virtualNetworks/myTFVnet, 10s elapsed]
azurerm_virtual_network.vnet: Destruction complete after 11s
azurerm_resource_group.rg: Destroying... [id=/subscriptions/xxxxxx-xxxx-xxxx-xxxx-xxxxxxxxx/resourceGroups/myTFResourceGroup]
azurerm_resource_group.rg: Still destroying... [id=/subscriptions/xxxxxx-xxxx-xxxx-xxxx-xxxxxxxxx/resourceGroups/myTFResourceGroup, 10s elapsed]
azurerm_resource_group.rg: Still destroying... [id=/subscriptions/xxxxxx-xxxx-xxxx-xxxx-xxxxxxxxx/resourceGroups/myTFResourceGroup, 20s elapsed]
azurerm_resource_group.rg: Still destroying... [id=/subscriptions/xxxxxx-xxxx-xxxx-xxxx-xxxxxxxxx/resourceGroups/myTFResourceGroup, 30s elapsed]
azurerm_resource_group.rg: Still destroying... [id=/subscriptions/xxxxxx-xxxx-xxxx-xxxx-xxxxxxxxx/resourceGroups/myTFResourceGroup, 40s elapsed]
azurerm_resource_group.rg: Still destroying... [id=/subscriptions/xxxxxx-xxxx-xxxx-xxxx-xxxxxxxxx/resourceGroups/myTFResourceGroup, 50s elapsed]
azurerm_resource_group.rg: Still destroying... [id=/subscriptions/xxxxxx-xxxx-xxxx-xxxx-xxxxxxxxx/resourceGroups/myTFResourceGroup, 1m0s elapsed]
azurerm_resource_group.rg: Still destroying... [id=/subscriptions/xxxxxx-xxxx-xxxx-xxxx-xxxxxxxxx/resourceGroups/myTFResourceGroup, 1m10s elapsed]
azurerm_resource_group.rg: Still destroying... [id=/subscriptions/xxxxxx-xxxx-xxxx-xxxx-xxxxxxxxx/resourceGroups/myTFResourceGroup, 1m20s elapsed]
azurerm_resource_group.rg: Destruction complete after 1m22s
azurerm_resource_group.rg: Creating...
azurerm_resource_group.rg: Creation complete after 2s [id=/subscriptions/xxxxxx-xxxx-xxxx-xxxx-xxxxxxxxx/resourceGroups/PradeepNewResourceGroup]
azurerm_virtual_network.vnet: Creating...
azurerm_virtual_network.vnet: Still creating... [10s elapsed]
azurerm_virtual_network.vnet: Creation complete after 14s [id=/subscriptions/xxxxxx-xxxx-xxxx-xxxx-xxxxxxxxx/resourceGroups/PradeepNewResourceGroup/providers/Microsoft.Network/virtualNetworks/myTFVnet]

Apply complete! Resources: 2 added, 0 changed, 2 destroyed.
(base) pradeep:~$
```

Setting variables via the command-line will not save their values. Terraform supports many ways to use and set variables so you can avoid having to enter them repeatedly as you execute commands. 

