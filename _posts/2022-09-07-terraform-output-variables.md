---
layout: single
title:  "Terraform Query Data with Output Variables"
date:   2022-09-07 09:58:04 +0530
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

In the previous posts, we used input variables to parameterize our Terraform configuration. In this post, we will use output values to organize data to be easily queried and displayed to the Terraform user.

When building complex infrastructure, Terraform stores hundreds or thousands of attribute values for all your resources. As a user of Terraform, you may only be interested in a few values of importance. Outputs designate which data to display. This data is outputted when apply is called, and can be queried using the `terraform output` command.

Create a file called `outputs.tf`

```sh
(base) pradeep:~$cat outputs.tf 
output "resource_group_id" {
  value = azurerm_resource_group.rg.id
}
(base) pradeep:~$
```
This defines an output variable named `resource_group_id`. The name of the variable must conform to Terraform variable naming conventions if it is to be used as an input to other modules. The value field specifies the value, the `id` attribute of your resource group.

You can define multiple output blocks to specify multiple output variables.

Observe your resource outputs

When you add an output to a previously applied configuration, you must re-run `terraform apply` to observe the new output.

```sh
(base) pradeep:~$terraform output resource_group_id
╷
│ Warning: No outputs found
│ 
│ The state file either has no outputs defined, or all the defined outputs are empty. Please define an output in your configuration with the `output` keyword and run `terraform refresh` for it to become
│ available. If you are using interpolation, please verify the interpolated value is not empty. You can use the `terraform console` command to assist.
╵
(base) pradeep:~$
```

```sh
(base) pradeep:~$terraform apply
azurerm_resource_group.rg: Refreshing state... [id=/subscriptions/xxxxxx-xxxx-xxxx-xxxx-xxxxxxxxx/resourceGroups/PradeepNewResourceGroup]
azurerm_virtual_network.vnet: Refreshing state... [id=/subscriptions/xxxxxx-xxxx-xxxx-xxxx-xxxxxxxxx/resourceGroups/PradeepNewResourceGroup/providers/Microsoft.Network/virtualNetworks/myTFVnet]

Terraform used the selected providers to generate the following execution plan. Resource actions are indicated with the following symbols:
-/+ destroy and then create replacement

Terraform will perform the following actions:

  # azurerm_resource_group.rg must be replaced
-/+ resource "azurerm_resource_group" "rg" {
      ~ id       = "/subscriptions/xxxxxx-xxxx-xxxx-xxxx-xxxxxxxxx/resourceGroups/PradeepNewResourceGroup" -> (known after apply)
      ~ name     = "PradeepNewResourceGroup" -> "myTFResourceGroup" # forces replacement
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
      ~ id                      = "/subscriptions/xxxxxx-xxxx-xxxx-xxxx-xxxxxxxxx/resourceGroups/PradeepNewResourceGroup/providers/Microsoft.Network/virtualNetworks/myTFVnet" -> (known after apply)
        name                    = "myTFVnet"
      ~ resource_group_name     = "PradeepNewResourceGroup" -> "myTFResourceGroup" # forces replacement
      ~ subnet                  = [] -> (known after apply)
      - tags                    = {} -> null
        # (2 unchanged attributes hidden)
    }

Plan: 2 to add, 0 to change, 2 to destroy.

Changes to Outputs:
  + resource_group_id = (known after apply)

Do you want to perform these actions?
  Terraform will perform the actions described above.
  Only 'yes' will be accepted to approve.

  Enter a value: yes

azurerm_virtual_network.vnet: Destroying... [id=/subscriptions/xxxxxx-xxxx-xxxx-xxxx-xxxxxxxxx/resourceGroups/PradeepNewResourceGroup/providers/Microsoft.Network/virtualNetworks/myTFVnet]
azurerm_virtual_network.vnet: Still destroying... [id=/subscriptions/xxxxxx-xxxx-xxxx-xxxx-xxxxxxxxx/osoft.Network/virtualNetworks/myTFVnet, 10s elapsed]
azurerm_virtual_network.vnet: Destruction complete after 11s
azurerm_resource_group.rg: Destroying... [id=/subscriptions/xxxxxx-xxxx-xxxx-xxxx-xxxxxxxxx/resourceGroups/PradeepNewResourceGroup]
azurerm_resource_group.rg: Still destroying... [id=/subscriptions/...resourceGroups/PradeepNewResourceGroup, 10s elapsed]
azurerm_resource_group.rg: Still destroying... [id=/subscriptions/xxxxxx-xxxx-xxxx-xxxx-xxxxxxxxx/resourceGroups/PradeepNewResourceGroup, 20s elapsed]
azurerm_resource_group.rg: Still destroying... [id=/subscriptions/xxxxxx-xxxx-xxxx-xxxx-xxxxxxxxx/resourceGroups/PradeepNewResourceGroup, 30s elapsed]
azurerm_resource_group.rg: Still destroying... [id=/subscriptions/xxxxxx-xxxx-xxxx-xxxx-xxxxxxxxx/resourceGroups/PradeepNewResourceGroup, 40s elapsed]
azurerm_resource_group.rg: Still destroying... [id=/subscriptions/xxxxxx-xxxx-xxxx-xxxx-xxxxxxxxx/resourceGroups/PradeepNewResourceGroup, 50s elapsed]
azurerm_resource_group.rg: Still destroying... [id=/subscriptions/xxxxxx-xxxx-xxxx-xxxx-xxxxxxxxx/resourceGroups/PradeepNewResourceGroup, 1m0s elapsed]
azurerm_resource_group.rg: Still destroying... [id=/subscriptions/xxxxxx-xxxx-xxxx-xxxx-xxxxxxxxx/resourceGroups/PradeepNewResourceGroup, 1m10s elapsed]
azurerm_resource_group.rg: Still destroying... [id=/subscriptions/xxxxxx-xxxx-xxxx-xxxx-xxxxxxxxx/resourceGroups/PradeepNewResourceGroup, 1m20s elapsed]
azurerm_resource_group.rg: Destruction complete after 1m23s
azurerm_resource_group.rg: Creating...
azurerm_resource_group.rg: Creation complete after 1s [id=/subscriptions/xxxxxx-xxxx-xxxx-xxxx-xxxxxxxxx/resourceGroups/myTFResourceGroup]
azurerm_virtual_network.vnet: Creating...
azurerm_virtual_network.vnet: Creation complete after 10s [id=/subscriptions/xxxxxx-xxxx-xxxx-xxxx-xxxxxxxxx/resourceGroups/myTFResourceGroup/providers/Microsoft.Network/virtualNetworks/myTFVnet]

Apply complete! Resources: 2 added, 0 changed, 2 destroyed.

Outputs:

resource_group_id = "/subscriptions/xxxxxx-xxxx-xxxx-xxxx-xxxxxxxxx/resourceGroups/myTFResourceGroup"
(base) pradeep:~$
```
Since we used a varible override earlier, which does not get saved, the resources are destroyed and then recreated (because of RG name change).

Note, the Outputs section at the end which shows the `resource_group_id`.
Query the output using the output command with the output ID.

```sh
(base) pradeep:~$terraform output resource_group_id
"/subscriptions/xxxxxx-xxxx-xxxx-xxxx-xxxxxxxxx/resourceGroups/myTFResourceGroup"
(base) pradeep:~$
```

Let us destroy these resources and check again

```sh
(base) pradeep:~$terraform destroy
azurerm_resource_group.rg: Refreshing state... [id=/subscriptions/xxxxxx-xxxx-xxxx-xxxx-xxxxxxxxx/resourceGroups/myTFResourceGroup]
azurerm_virtual_network.vnet: Refreshing state... [id=/subscriptions/xxxxxx-xxxx-xxxx-xxxx-xxxxxxxxx/resourceGroups/myTFResourceGroup/providers/Microsoft.Network/virtualNetworks/myTFVnet]

Terraform used the selected providers to generate the following execution plan. Resource actions are indicated with the following symbols:
  - destroy

Terraform will perform the following actions:

  # azurerm_resource_group.rg will be destroyed
  - resource "azurerm_resource_group" "rg" {
      - id       = "/subscriptions/xxxxxx-xxxx-xxxx-xxxx-xxxxxxxxx/resourceGroups/myTFResourceGroup" -> null
      - location = "westus2" -> null
      - name     = "myTFResourceGroup" -> null
      - tags     = {
          - "Environment" = "Pradeep's Terraform Getting Started"
          - "Team"        = "PradeepDevOps"
        } -> null
    }

  # azurerm_virtual_network.vnet will be destroyed
  - resource "azurerm_virtual_network" "vnet" {
      - address_space           = [
          - "10.0.0.0/16",
        ] -> null
      - dns_servers             = [] -> null
      - flow_timeout_in_minutes = 0 -> null
      - guid                    = "xxxxxx-xxxx-xxxx-xxxx-xxxxxxxxx" -> null
      - id                      = "/subscriptions/xxxxxx-xxxx-xxxx-xxxx-xxxxxxxxx/resourceGroups/myTFResourceGroup/providers/Microsoft.Network/virtualNetworks/myTFVnet" -> null
      - location                = "westus2" -> null
      - name                    = "myTFVnet" -> null
      - resource_group_name     = "myTFResourceGroup" -> null
      - subnet                  = [] -> null
      - tags                    = {} -> null
    }

Plan: 0 to add, 0 to change, 2 to destroy.

Changes to Outputs:
  - resource_group_id = "/subscriptions/xxxxxx-xxxx-xxxx-xxxx-xxxxxxxxx/resourceGroups/myTFResourceGroup" -> null

Do you really want to destroy all resources?
  Terraform will destroy all your managed infrastructure, as shown above.
  There is no undo. Only 'yes' will be accepted to confirm.

  Enter a value: yes

azurerm_virtual_network.vnet: Destroying... [id=/subscriptions/xxxxxx-xxxx-xxxx-xxxx-xxxxxxxxx/resourceGroups/myTFResourceGroup/providers/Microsoft.Network/virtualNetworks/myTFVnet]
azurerm_virtual_network.vnet: Still destroying... [id=/subscriptions/f3f98f91-9191-4dd5-a561-...osoft.Network/virtualNetworks/myTFVnet, 10s elapsed]
azurerm_virtual_network.vnet: Destruction complete after 12s
azurerm_resource_group.rg: Destroying... [id=/subscriptions/xxxxxx-xxxx-xxxx-xxxx-xxxxxxxxx/resourceGroups/myTFResourceGroup]
azurerm_resource_group.rg: Still destroying... [id=/subscriptions/xxxxxx-xxxx-xxxx-xxxx-xxxxxxxxx/resourceGroups/myTFResourceGroup, 10s elapsed]
azurerm_resource_group.rg: Still destroying... [id=/subscriptions/xxxxxx-xxxx-xxxx-xxxx-xxxxxxxxx/resourceGroups/myTFResourceGroup, 20s elapsed]
azurerm_resource_group.rg: Still destroying... [id=/subscriptions/xxxxxx-xxxx-xxxx-xxxx-xxxxxxxxx/resourceGroups/myTFResourceGroup, 30s elapsed]
azurerm_resource_group.rg: Still destroying... [id=/subscriptions/xxxxxx-xxxx-xxxx-xxxx-xxxxxxxxx/resourceGroups/myTFResourceGroup, 40s elapsed]
azurerm_resource_group.rg: Still destroying... [id=/subscriptions/xxxxxx-xxxx-xxxx-xxxx-xxxxxxxxx/resourceGroups/myTFResourceGroup, 50s elapsed]
azurerm_resource_group.rg: Still destroying... [id=/subscriptions/xxxxxx-xxxx-xxxx-xxxx-xxxxxxxxx/resourceGroups/myTFResourceGroup, 1m0s elapsed]
azurerm_resource_group.rg: Still destroying... [id=/subscriptions/xxxxxx-xxxx-xxxx-xxxx-xxxxxxxxx/resourceGroups/myTFResourceGroup, 1m10s elapsed]
azurerm_resource_group.rg: Still destroying... [id=/subscriptions/xxxxxx-xxxx-xxxx-xxxx-xxxxxxxxx/resourceGroups/myTFResourceGroup, 1m20s elapsed]
azurerm_resource_group.rg: Destruction complete after 1m23s

Destroy complete! Resources: 2 destroyed.
(base) pradeep:~$
```

```sh
(base) pradeep:~$terraform state  list
(base) pradeep:~$
```

```sh
(base) pradeep:~$terraform output                  
╷
│ Warning: No outputs found
│ 
│ The state file either has no outputs defined, or all the defined outputs are empty. Please define an output in your configuration with the `output` keyword and run `terraform refresh` for it to become
│ available. If you are using interpolation, please verify the interpolated value is not empty. You can use the `terraform console` command to assist.
╵
(base) pradeep:~$
```
