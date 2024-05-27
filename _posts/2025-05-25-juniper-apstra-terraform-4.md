---
layout: single
title:  "Change Infrastructure using Juniper Apstra Terraform Provider"
categories: Automation
tags: Terraform
toc: true
toc_sticky: true
show_date: true
header:
  teaser: /assets/images/apstra.png
author:
  name     : "Juniper Apstra Terraform"
  avatar   : "/assets/images/apstra.png"

sidebar:
  - title: "Topics"
    nav: my-sidebar
---
# Change Infrastructure using Juniper Apstra Terraform Provider

In the last post, you created your first infrastructure with Terraform: a single IPv4 pool on Juniper Aspstra. In this tutorial, you will modify that resource, and learn how to apply changes to your Terraform projects.

Infrastructure is continuously evolving, and Terraform helps you manage that change. As you change Terraform configurations, Terraform builds an execution plan that only modifies what is necessary to reach your desired state.

## Change Configuration

```
pradeep@juniper apstra-terraform-3 % cat main.tf 
terraform {
  required_providers {
    apstra = {
      source  = "Juniper/apstra"
      version = ">= 0.28.0"
    }
  }
  required_version = ">= 1.1"
}
provider "apstra" {
  url                     = "https://admin:password@10.210.40.194:443"
  tls_validation_disabled = true
  blueprint_mutex_enabled = true
}

# This example creates an IPv4 Resource Pool with three allocations inside:
# - 3 blocks from RFC5737
#
# After creating the pool, it outputs the total number of
# IP addresses in the pool.

resource "apstra_ipv4_pool" "example" {
  name = "RFC 5737 ranges"
  subnets = [
    { network = "192.0.2.0/24" },
    { network = "198.51.100.0/24" },
    { network = "203.0.113.0/24" },
  ]
}

output "example_pool_size" {
  value = format("pool '%s' is sized for %d addresses", apstra_ipv4_pool.example.name, apstra_ipv4_pool.example.total)
}
pradeep@juniper apstra-terraform-3 % 
```

Now change the Name of the IPv4 pool from `RFC 5737 ranges` to `Documentation Subnets`.

```
pradeep@juniper apstra-terraform-3 % cat main.tf 
terraform {
  required_providers {
    apstra = {
      source  = "Juniper/apstra"
      version = ">= 0.28.0"
    }
  }
  required_version = ">= 1.1"
}
provider "apstra" {
  url                     = "https://admin:ApstraT3rraf0rm!@10.210.40.194:443"
  tls_validation_disabled = true
  blueprint_mutex_enabled = true
}

# This example creates an IPv4 Resource Pool with three allocations inside:
# - 3 blocks from RFC5737
#
# After creating the pool, it outputs the total number of
# IP addresses in the pool.

resource "apstra_ipv4_pool" "example" {
  name = "Documentation Subnets"
  subnets = [
    { network = "192.0.2.0/24" },
    { network = "198.51.100.0/24" },
    { network = "203.0.113.0/24" },
  ]
}

output "example_pool_size" {
  value = format("pool '%s' is sized for %d addresses", apstra_ipv4_pool.example.name, apstra_ipv4_pool.example.total)
}
pradeep@juniper apstra-terraform-3 % 
```

## Apply Changes

After changing the configuration, run `terraform apply` again to see how Terraform will apply this change to the existing resources.

```
pradeep@juniper apstra-terraform-3 % terraform apply
apstra_ipv4_pool.example: Refreshing state... [id=72f6ff51-9062-47ae-9b1d-05eb36835888]

Terraform used the selected providers to generate the following execution plan. Resource actions are indicated with the following symbols:
  ~ update in-place

Terraform will perform the following actions:

  # apstra_ipv4_pool.example will be updated in-place
  ~ resource "apstra_ipv4_pool" "example" {
        id              = "72f6ff51-9062-47ae-9b1d-05eb36835888"
      ~ name            = "RFC 5737 ranges" -> "Documentation Subnets"
      ~ status          = "not_in_use" -> (known after apply)
      ~ total           = 768 -> (known after apply)
      ~ used            = 0 -> (known after apply)
      ~ used_percentage = 0 -> (known after apply)
        # (1 unchanged attribute hidden)
    }

Plan: 0 to add, 1 to change, 0 to destroy.

Changes to Outputs:
  ~ example_pool_size = "pool 'RFC 5737 ranges' is sized for 768 addresses" -> (known after apply)

Do you want to perform these actions?
  Terraform will perform the actions described above.
  Only 'yes' will be accepted to approve.

  Enter a value: yes

apstra_ipv4_pool.example: Modifying... [id=72f6ff51-9062-47ae-9b1d-05eb36835888]
apstra_ipv4_pool.example: Modifications complete after 3s [id=72f6ff51-9062-47ae-9b1d-05eb36835888]

Apply complete! Resources: 0 added, 1 changed, 0 destroyed.

Outputs:

example_pool_size = "pool 'Documentation Subnets' is sized for 768 addresses"
pradeep@juniper apstra-terraform-3 % 
```

Terraform can update some attributes in-place (indicated with the `~` prefix)

```
pradeep@juniper apstra-terraform-3 % terraform show
# apstra_ipv4_pool.example:
resource "apstra_ipv4_pool" "example" {
    id              = "72f6ff51-9062-47ae-9b1d-05eb36835888"
    name            = "Documentation Subnets"
    status          = "not_in_use"
    subnets         = [
        {
            network         = "192.0.2.0/24"
            status          = "pool_element_available"
            total           = 256
            used            = 0
            used_percentage = 0
        },
        {
            network         = "198.51.100.0/24"
            status          = "pool_element_available"
            total           = 256
            used            = 0
            used_percentage = 0
        },
        {
            network         = "203.0.113.0/24"
            status          = "pool_element_available"
            total           = 256
            used            = 0
            used_percentage = 0
        },
    ]
    total           = 768
    used            = 0
    used_percentage = 0
}


Outputs:

example_pool_size = "pool 'Documentation Subnets' is sized for 768 addresses"
pradeep@juniper apstra-terraform-3 % 
```

![tf_apstra_11]({{ site.url }}{{ site.baseurl }}/assets/images/tf_apstra_11.png) 

Modify the resource configuration once again.

```
pradeep@juniper apstra-terraform-3 % cat main.tf 
terraform {
  required_providers {
    apstra = {
      source  = "Juniper/apstra"
      version = ">= 0.28.0"
    }
  }
  required_version = ">= 1.1"
}
provider "apstra" {
  url                     = "https://admin:ApstraT3rraf0rm!@10.210.40.194:443"
  tls_validation_disabled = true
  blueprint_mutex_enabled = true
}

# This example creates an IPv4 Resource Pool with three allocations inside:
# - 3 blocks from RFC1918
#
# After creating the pool, it outputs the total number of
# IP addresses in the pool.

resource "apstra_ipv4_pool" "example" {
  name = "RFC1918 Subnets"
  subnets = [
    { network = "10.0.0.0/8" },
    { network = "172.16.0.0/12" },
    { network = "192.168.0.0/16" },
  ]
}

output "example_pool_size" {
  value = format("pool '%s' is sized for %d addresses", apstra_ipv4_pool.example.name, apstra_ipv4_pool.example.total)
}
pradeep@juniper apstra-terraform-3 % 
```
Apply the change and notice how the resource is modified as indicated by `~`, `-`, `+`. Note that there is one more possibility, but not covered here. The prefix `-/+` means that Terraform will destroy and recreate the resource, rather than updating it in-place.

```
pradeep@juniper apstra-terraform-3 % terraform apply
apstra_ipv4_pool.example: Refreshing state... [id=72f6ff51-9062-47ae-9b1d-05eb36835888]

Terraform used the selected providers to generate the following execution plan. Resource actions are indicated with the following symbols:
  ~ update in-place

Terraform will perform the following actions:

  # apstra_ipv4_pool.example will be updated in-place
  ~ resource "apstra_ipv4_pool" "example" {
        id              = "72f6ff51-9062-47ae-9b1d-05eb36835888"
      ~ name            = "Documentation Subnets" -> "RFC1918 Subnets"
      ~ status          = "not_in_use" -> (known after apply)
      ~ subnets         = [
          - {
              - network         = "192.0.2.0/24" -> null
              - status          = "pool_element_available" -> null
              - total           = 256 -> null
              - used            = 0 -> null
              - used_percentage = 0 -> null
            },
          - {
              - network         = "198.51.100.0/24" -> null
              - status          = "pool_element_available" -> null
              - total           = 256 -> null
              - used            = 0 -> null
              - used_percentage = 0 -> null
            },
          - {
              - network         = "203.0.113.0/24" -> null
              - status          = "pool_element_available" -> null
              - total           = 256 -> null
              - used            = 0 -> null
              - used_percentage = 0 -> null
            },
          + {
              + network         = "10.0.0.0/8"
              + status          = (known after apply)
              + total           = (known after apply)
              + used            = (known after apply)
              + used_percentage = (known after apply)
            },
          + {
              + network         = "172.16.0.0/12"
              + status          = (known after apply)
              + total           = (known after apply)
              + used            = (known after apply)
              + used_percentage = (known after apply)
            },
          + {
              + network         = "192.168.0.0/16"
              + status          = (known after apply)
              + total           = (known after apply)
              + used            = (known after apply)
              + used_percentage = (known after apply)
            },
        ]
      ~ total           = 768 -> (known after apply)
      ~ used            = 0 -> (known after apply)
      ~ used_percentage = 0 -> (known after apply)
    }

Plan: 0 to add, 1 to change, 0 to destroy.

Changes to Outputs:
  ~ example_pool_size = "pool 'Documentation Subnets' is sized for 768 addresses" -> (known after apply)

Do you want to perform these actions?
  Terraform will perform the actions described above.
  Only 'yes' will be accepted to approve.

  Enter a value: yes

apstra_ipv4_pool.example: Modifying... [id=72f6ff51-9062-47ae-9b1d-05eb36835888]
apstra_ipv4_pool.example: Modifications complete after 1s [id=72f6ff51-9062-47ae-9b1d-05eb36835888]

Apply complete! Resources: 0 added, 1 changed, 0 destroyed.

Outputs:

example_pool_size = "pool 'RFC1918 Subnets' is sized for 17891328 addresses"
pradeep@juniper apstra-terraform-3 % 
```

Go to Apstra Web UI and verify the change in the resource 

![tf_apstra_12]({{ site.url }}{{ site.baseurl }}/assets/images/tf_apstra_12.png) 

You can use `terraform show` again to have Terraform print out the new values associated with this resource.

```
pradeep@juniper apstra-terraform-3 % terraform show
# apstra_ipv4_pool.example:
resource "apstra_ipv4_pool" "example" {
    id              = "72f6ff51-9062-47ae-9b1d-05eb36835888"
    name            = "RFC1918 Subnets"
    status          = "not_in_use"
    subnets         = [
        {
            network         = "10.0.0.0/8"
            status          = "pool_element_available"
            total           = 16777216
            used            = 0
            used_percentage = 0
        },
        {
            network         = "172.16.0.0/12"
            status          = "pool_element_available"
            total           = 1048576
            used            = 0
            used_percentage = 0
        },
        {
            network         = "192.168.0.0/16"
            status          = "pool_element_available"
            total           = 65536
            used            = 0
            used_percentage = 0
        },
    ]
    total           = 17891328
    used            = 0
    used_percentage = 0
}


Outputs:

example_pool_size = "pool 'RFC1918 Subnets' is sized for 17891328 addresses"
pradeep@juniper apstra-terraform-3 % 
```

