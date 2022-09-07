---
layout: single
title:  "Updating Azure Resources with Terraform"
date:   2022-09-07 06:58:04 +0530
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

# Updating Azure Resources with Terraform

In the previous post, we created our first Azure infrastructure with Terraform: a resource group. Now let us modify the configuration by defining an additional resource that references our resource group and adding tags to our resource group.

In our `main.tf` file, add the resource block below to create a virtual network (VNet).

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
  name     = "myTFResourceGroup"
  location = "westus2"
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

To create a new Azure VNet, we have to specify the name of the resource group to contain the VNet. By referencing the resource group, we are establishing a dependency between the resources. Terraform ensures that resources are created in proper order by constructing a dependency graph for our configuration.



```sh
(base) pradeep:~$terraform plan
azurerm_resource_group.rg: Refreshing state... [id=/subscriptions/xxxxxx-xxxx-xxxx-xxxx-xxxxxxxxx/resourceGroups/myTFResourceGroup]

Terraform used the selected providers to generate the following execution plan. Resource actions are indicated with the following symbols:
  + create

Terraform will perform the following actions:

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

Plan: 1 to add, 0 to change, 0 to destroy.

───────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────

Note: You didn't use the -out option to save this plan, so Terraform can't guarantee to take exactly these actions if you run "terraform apply" now.
(base) pradeep:~$
```

Run `terraform apply` again and respond `yes` to the prompt to confirm the changes.

```sh
(base) pradeep:~$terraform apply
azurerm_resource_group.rg: Refreshing state... [id=/subscriptions/xxxxxx-xxxx-xxxx-xxxx-xxxxxxxxx/resourceGroups/myTFResourceGroup]

Terraform used the selected providers to generate the following execution plan. Resource actions are indicated with the following symbols:
  + create

Terraform will perform the following actions:

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

Plan: 1 to add, 0 to change, 0 to destroy.

Do you want to perform these actions?
  Terraform will perform the actions described above.
  Only 'yes' will be accepted to approve.

  Enter a value: yes

azurerm_virtual_network.vnet: Creating...
azurerm_virtual_network.vnet: Still creating... [10s elapsed]
azurerm_virtual_network.vnet: Creation complete after 12s [id=/subscriptions/xxxxxx-xxxx-xxxx-xxxx-xxxxxxxxx/resourceGroups/myTFResourceGroup/providers/Microsoft.Network/virtualNetworks/myTFVnet]

Apply complete! Resources: 1 added, 0 changed, 0 destroyed.
(base) pradeep:~$
```
Terraform builds an execution plan by comparing your desired state as described in the configuration to the current state, which is saved in either the local terraform.tfstate file or in a remote state backend depending on your configuration.

![Azure-RG-TF-2]({{ site.url }}{{ site.baseurl }}/assets/images/azure-rg-tf-2.png)

In addition to creating new resources, Terraform can modify existing resources.

In our `main.tf` file, let us update the `azurerm_resource_group` resource  by adding the `tags` block as shown below:
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
  name     = "myTFResourceGroup"
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


```sh
(base) pradeep:~$terraform plan
azurerm_resource_group.rg: Refreshing state... [id=/subscriptions/xxxxxx-xxxx-xxxx-xxxx-xxxxxxxxx/resourceGroups/myTFResourceGroup]
azurerm_virtual_network.vnet: Refreshing state... [id=/subscriptions/xxxxxx-xxxx-xxxx-xxxx-xxxxxxxxx/resourceGroups/myTFResourceGroup/providers/Microsoft.Network/virtualNetworks/myTFVnet]

Terraform used the selected providers to generate the following execution plan. Resource actions are indicated with the following symbols:
  ~ update in-place

Terraform will perform the following actions:

  # azurerm_resource_group.rg will be updated in-place
  ~ resource "azurerm_resource_group" "rg" {
        id       = "/subscriptions/xxxxxx-xxxx-xxxx-xxxx-xxxxxxxxx/resourceGroups/myTFResourceGroup"
        name     = "myTFResourceGroup"
      ~ tags     = {
          + "Environment" = "Pradeep's Terraform Getting Started"
          + "Team"        = "PradeepDevOps"
        }
        # (1 unchanged attribute hidden)
    }

Plan: 0 to add, 1 to change, 0 to destroy.

───────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────

Note: You didn't use the -out option to save this plan, so Terraform can't guarantee to take exactly these actions if you run "terraform apply" now.
(base) pradeep:~$
```

```sh
(base) pradeep:~$terraform apply
azurerm_resource_group.rg: Refreshing state... [id=/subscriptions/xxxxxx-xxxx-xxxx-xxxx-xxxxxxxxx/resourceGroups/myTFResourceGroup]
azurerm_virtual_network.vnet: Refreshing state... [id=/subscriptions/xxxxxx-xxxx-xxxx-xxxx-xxxxxxxxx/resourceGroups/myTFResourceGroup/providers/Microsoft.Network/virtualNetworks/myTFVnet]

Terraform used the selected providers to generate the following execution plan. Resource actions are indicated with the following symbols:
  ~ update in-place

Terraform will perform the following actions:

  # azurerm_resource_group.rg will be updated in-place
  ~ resource "azurerm_resource_group" "rg" {
        id       = "/subscriptions/xxxxxx-xxxx-xxxx-xxxx-xxxxxxxxx/resourceGroups/myTFResourceGroup"
        name     = "myTFResourceGroup"
      ~ tags     = {
          + "Environment" = "Pradeep's Terraform Getting Started"
          + "Team"        = "PradeepDevOps"
        }
        # (1 unchanged attribute hidden)
    }

Plan: 0 to add, 1 to change, 0 to destroy.

Do you want to perform these actions?
  Terraform will perform the actions described above.
  Only 'yes' will be accepted to approve.

  Enter a value: yes

azurerm_resource_group.rg: Modifying... [id=/subscriptions/xxxxxx-xxxx-xxxx-xxxx-xxxxxxxxx/resourceGroups/myTFResourceGroup]
azurerm_resource_group.rg: Modifications complete after 4s [id=/subscriptions/xxxxxx-xxxx-xxxx-xxxx-xxxxxxxxx/resourceGroups/myTFResourceGroup]

Apply complete! Resources: 0 added, 1 changed, 0 destroyed.
(base) pradeep:~$
```

The prefix `~` means that Terraform will update the resource in-place.

Review updates to state

Use `terraform show` again to see the new values associated with this resource group.

```sh
(base) pradeep:~$terraform show
# azurerm_resource_group.rg:
resource "azurerm_resource_group" "rg" {
    id       = "/subscriptions/xxxxxx-xxxx-xxxx-xxxx-xxxxxxxxx/resourceGroups/myTFResourceGroup"
    location = "westus2"
    name     = "myTFResourceGroup"
    tags     = {
        "Environment" = "Pradeep's Terraform Getting Started"
        "Team"        = "PradeepDevOps"
    }
}

# azurerm_virtual_network.vnet:
resource "azurerm_virtual_network" "vnet" {
    address_space           = [
        "10.0.0.0/16",
    ]
    dns_servers             = []
    flow_timeout_in_minutes = 0
    guid                    = "xxxxxx-xxxx-xxxx-xxxx-xxxxxxxxx"
    id                      = "/subscriptions/xxxxxx-xxxx-xxxx-xxxx-xxxxxxxxx/resourceGroups/myTFResourceGroup/providers/Microsoft.Network/virtualNetworks/myTFVnet"
    location                = "westus2"
    name                    = "myTFVnet"
    resource_group_name     = "myTFResourceGroup"
    subnet                  = []
    tags                    = {}
}
(base) pradeep:~$
```

```sh
(base) pradeep:~$terraform state list
azurerm_resource_group.rg
azurerm_virtual_network.vnet
(base) pradeep:~$
```
