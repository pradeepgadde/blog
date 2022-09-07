---
layout: single
title:  "Destroying Azure Resources with Terraform"
date:   2022-09-07 07:58:04 +0530
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

# Destroying Azure Resources with Terraform

So far we  used Terraform to create and update Azure resources. In this post, let us try Terraform to destroy our infrastructure.


Destroy

The `terraform destroy` command terminates resources managed by your Terraform project. This command is the inverse of `terraform apply` in that it terminates all the resources specified in your Terraform state. It does not destroy resources running elsewhere that are not managed by the current Terraform project.

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

Do you really want to destroy all resources?
  Terraform will destroy all your managed infrastructure, as shown above.
  There is no undo. Only 'yes' will be accepted to confirm.

  Enter a value: yes

azurerm_virtual_network.vnet: Destroying... [id=/subscriptions/xxxxxx-xxxx-xxxx-xxxx-xxxxxxxxx/resourceGroups/myTFResourceGroup/providers/Microsoft.Network/virtualNetworks/myTFVnet]
azurerm_virtual_network.vnet: Still destroying... [id=/subscriptions/xxxxxx-xxxx-xxxx-xxxx-xxxxxxxxx-...osoft.Network/virtualNetworks/myTFVnet, 10s elapsed]
azurerm_virtual_network.vnet: Destruction complete after 13s
azurerm_resource_group.rg: Destroying... [id=/subscriptions/xxxxxx-xxxx-xxxx-xxxx-xxxxxxxxx/resourceGroups/myTFResourceGroup]
azurerm_resource_group.rg: Still destroying... [id=/subscriptions/xxxxxx-xxxx-xxxx-xxxx-xxxxxxxxx/resourceGroups/myTFResourceGroup, 10s elapsed]
azurerm_resource_group.rg: Still destroying... [id=/subscriptions/xxxxxx-xxxx-xxxx-xxxx-xxxxxxxxx/resourceGroups/myTFResourceGroup, 20s elapsed]
azurerm_resource_group.rg: Still destroying... [id=/subscriptions/xxxxxx-xxxx-xxxx-xxxx-xxxxxxxxx/resourceGroups/myTFResourceGroup, 30s elapsed]
azurerm_resource_group.rg: Still destroying... [id=/subscriptions/xxxxxx-xxxx-xxxx-xxxx-xxxxxxxxx/resourceGroups/myTFResourceGroup, 40s elapsed]
azurerm_resource_group.rg: Still destroying... [id=/subscriptions/xxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxresourceGroups/myTFResourceGroup, 50s elapsed]
azurerm_resource_group.rg: Still destroying... [id=/subscriptions/xxxxxx-xxxx-xxxx-xxxx-xxxxxxxxx/resourceGroups/myTFResourceGroup, 1m0s elapsed]
azurerm_resource_group.rg: Still destroying... [id=/subscriptions/xxxxxx-xxxx-xxxx-xxxx-xxxxxxxxx/resourceGroups/myTFResourceGroup, 1m10s elapsed]
azurerm_resource_group.rg: Still destroying... [id=/subscriptions/xxxxxx-xxxx-xxxx-xxxx-xxxxxxxxx/resourceGroups/myTFResourceGroup, 1m20s elapsed]
azurerm_resource_group.rg: Destruction complete after 1m24s

Destroy complete! Resources: 2 destroyed.
(base) pradeep:~$
```

The `-` prefix in the output indicates that the instance will be destroyed. As with apply, Terraform shows its execution plan and waits for approval before making any changes.

Just like with apply, Terraform determines the order in which things must be destroyed. In more complicated cases with multiple resources, Terraform will destroy them in a suitable order to respect dependencies.


```sh
(base) pradeep:~$terraform show

(base) pradeep:~$
```

```sh
(base) pradeep:~$ls
main.tf				terraform.tfstate		terraform.tfstate.backup
(base) pradeep:~$
```
We can see a terraform state file and backup terraform state.

```sh
(base) pradeep:~$cat terraform.tfstate
{
  "version": 4,
  "terraform_version": "1.2.8",
  "serial": 8,
  "lineage": "44e2918b-xxxx-08ca-6966-xxxxxxxx",
  "outputs": {},
  "resources": []
}
```
As expected, the resources list is empty as we destroyed all our infrastructure some time ago.

Let us see what is there in the backup state!
```sh
(base) pradeep:~$cat terraform.tfstate.backup 
{
  "version": 4,
  "terraform_version": "1.2.8",
  "serial": 5,
  "lineage": "44e2918b-xxxx-08ca-6966-xxxxxxxx",
  "outputs": {},
  "resources": [
    {
      "mode": "managed",
      "type": "azurerm_resource_group",
      "name": "rg",
      "provider": "provider[\"registry.terraform.io/hashicorp/azurerm\"]",
      "instances": [
        {
          "schema_version": 0,
          "attributes": {
            "id": "/subscriptions/xxxxxx-xxxx-xxxx-xxxx-xxxxxxxxx/resourceGroups/myTFResourceGroup",
            "location": "westus2",
            "name": "myTFResourceGroup",
            "tags": {
              "Environment": "Pradeep's Terraform Getting Started",
              "Team": "PradeepDevOps"
            },
            "timeouts": null
          },
          "sensitive_attributes": [],
          "private": "snipped"
        }
      ]
    },
    {
      "mode": "managed",
      "type": "azurerm_virtual_network",
      "name": "vnet",
      "provider": "provider[\"registry.terraform.io/hashicorp/azurerm\"]",
      "instances": [
        {
          "schema_version": 0,
          "attributes": {
            "address_space": [
              "10.0.0.0/16"
            ],
            "bgp_community": "",
            "ddos_protection_plan": [],
            "dns_servers": [],
            "edge_zone": "",
            "flow_timeout_in_minutes": 0,
            "guid": "xxxxxx-xxxx-xxxx-xxxx-xxxxxxxxx",
            "id": "/subscriptions/xxxxxx-xxxx-xxxx-xxxx-xxxxxxxxx/resourceGroups/myTFResourceGroup/providers/Microsoft.Network/virtualNetworks/myTFVnet",
            "location": "westus2",
            "name": "myTFVnet",
            "resource_group_name": "myTFResourceGroup",
            "subnet": [],
            "tags": {},
            "timeouts": null
          },
          "sensitive_attributes": [],
          "private": "snipped",
          "dependencies": [
            "azurerm_resource_group.rg"
          ]
        }
      ]
    }
  ]
}
(base) pradeep:~$
```