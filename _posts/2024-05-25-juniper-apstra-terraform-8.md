---
layout: single
title:  "Juniper Apstra Terraform Provider — Outputs"
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

# Juniper Apstra Terraform Provider — Outputs

In the previous tutorial, you used an input variable to parameterize your Terraform configuration. In this tutorial, you will use output values to organize data to be easily queried and displayed to the Terraform user.

Let’s verify our current Terraform configuration

```
pradeep@juniper apstra-terraform-4 % cat main.tf 
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
  url = "https://admin:password@10.210.40.194:443"
  tls_validation_disabled = true
  blueprint_mutex_enabled = true
}

# This example creates an IPv4 Resource Pool with three allocations inside:
# - 3 blocks from RFC5737
#
# After creating the pool, it outputs the total number of
# IP addresses in the pool.

resource "apstra_ipv4_pool" "example" {
  name = var.ipv4_pool_name
  subnets = [
    { network = "192.0.2.0/24"},
    { network = "198.51.100.0/24"},
    { network = "203.0.113.0/24"},
  ]
}

resource "apstra_asn_pool" "rfc5398" {
  name = "RFC5398 ASNs"
  ranges = [
    {
      first = 64496
      last  = 64511
    },
    {
      first = 65536
      last  = 65551
    },
  ]
}
pradeep@juniper apstra-terraform-4 %
```

```
pradeep@juniper apstra-terraform-4 % cat variables.tf 
variable "ipv4_pool_name" {
  description = "Value of the name for the IPv4 Pool"
  type        = string
}

pradeep@juniper apstra-terraform-4 %
```

```
pradeep@juniper apstra-terraform-4 % cat terraform.tfvars 
ipv4_pool_name = "TestPool"
pradeep@juniper apstra-terraform-4 % 
```

## Terraform Init

```
pradeep@juniper apstra-terraform-4 % terraform init

Initializing the backend...

Initializing provider plugins...
- Reusing previous version of juniper/apstra from the dependency lock file
- Using previously-installed juniper/apstra v0.42.0

Terraform has been successfully initialized!

You may now begin working with Terraform. Try running "terraform plan" to see
any changes that are required for your infrastructure. All Terraform commands
should now work.

If you ever set or change modules or backend configuration for Terraform,
rerun this command to reinitialize your working directory. If you forget, other
commands will detect it and remind you to do so if necessary.
pradeep@juniper apstra-terraform-4 %
```

## Terraform Apply

```
pradeep@juniper apstra-terraform-4 % terraform apply

Terraform used the selected providers to generate the following execution plan. Resource actions are indicated with the following symbols:
  + create

Terraform will perform the following actions:

  # apstra_asn_pool.rfc5398 will be created
  + resource "apstra_asn_pool" "rfc5398" {
      + id              = (known after apply)
      + name            = "RFC5398 ASNs"
      + ranges          = [
          + {
              + first           = 64496
              + last            = 64511
              + status          = (known after apply)
              + total           = (known after apply)
              + used            = (known after apply)
              + used_percentage = (known after apply)
            },
          + {
              + first           = 65536
              + last            = 65551
              + status          = (known after apply)
              + total           = (known after apply)
              + used            = (known after apply)
              + used_percentage = (known after apply)
            },
        ]
      + status          = (known after apply)
      + total           = (known after apply)
      + used            = (known after apply)
      + used_percentage = (known after apply)
    }

  # apstra_ipv4_pool.example will be created
  + resource "apstra_ipv4_pool" "example" {
      + id              = (known after apply)
      + name            = "TestPool"
      + status          = (known after apply)
      + subnets         = [
          + {
              + network         = "192.0.2.0/24"
              + status          = (known after apply)
              + total           = (known after apply)
              + used            = (known after apply)
              + used_percentage = (known after apply)
            },
          + {
              + network         = "198.51.100.0/24"
              + status          = (known after apply)
              + total           = (known after apply)
              + used            = (known after apply)
              + used_percentage = (known after apply)
            },
          + {
              + network         = "203.0.113.0/24"
              + status          = (known after apply)
              + total           = (known after apply)
              + used            = (known after apply)
              + used_percentage = (known after apply)
            },
        ]
      + total           = (known after apply)
      + used            = (known after apply)
      + used_percentage = (known after apply)
    }

Plan: 2 to add, 0 to change, 0 to destroy.

Do you want to perform these actions?
  Terraform will perform the actions described above.
  Only 'yes' will be accepted to approve.

  Enter a value: yes

apstra_ipv4_pool.example: Creating...
apstra_asn_pool.rfc5398: Creating...
apstra_asn_pool.rfc5398: Creation complete after 1s [id=b2c9b72b-543d-4afc-bc9a-7b93414afb0e]
apstra_ipv4_pool.example: Creation complete after 1s [id=abb107c9-8cf7-4639-b805-1c164b1e68f1]

Apply complete! Resources: 2 added, 0 changed, 0 destroyed.
pradeep@juniper apstra-terraform-4 %
```

## Output Apstra Resource configuration

Create a file called `outputs.tf` in your current directory.

Add the configuration below to `outputs.tf` to define outputs for your IPv4 pool ID and the ASN pool ID.

```
pradeep@juniper apstra-terraform-4 % cat outputs.tf 
output "asn_pool_id" {
  description = "ID of the ASN Pool"
  value       = apstra_asn_pool.rfc5398.id
}

output "ipv4_pool_id" {
  description = "ID of the IPv4 Pool"
  value       = apstra_ipv4_pool.example.id
}

pradeep@juniper apstra-terraform-4 % 
```

## Inspect output values

You must apply this configuration before you can use these output values. Apply your configuration now. Respond to the confirmation prompt with `yes`.

```
pradeep@juniper apstra-terraform-4 % terraform apply
apstra_ipv4_pool.example: Refreshing state... [id=abb107c9-8cf7-4639-b805-1c164b1e68f1]
apstra_asn_pool.rfc5398: Refreshing state... [id=b2c9b72b-543d-4afc-bc9a-7b93414afb0e]

Changes to Outputs:
  + asn_pool_id  = "b2c9b72b-543d-4afc-bc9a-7b93414afb0e"
  + ipv4_pool_id = "abb107c9-8cf7-4639-b805-1c164b1e68f1"

You can apply this plan to save these new output values to the Terraform state, without changing any real infrastructure.

Do you want to perform these actions?
  Terraform will perform the actions described above.
  Only 'yes' will be accepted to approve.

  Enter a value: yes


Apply complete! Resources: 0 added, 0 changed, 0 destroyed.

Outputs:

asn_pool_id = "b2c9b72b-543d-4afc-bc9a-7b93414afb0e"
ipv4_pool_id = "abb107c9-8cf7-4639-b805-1c164b1e68f1"
pradeep@juniper apstra-terraform-4 % 
```

Terraform prints output values to the screen when you apply your configuration. 

## Terraform Output

Query the outputs with the `terraform output` command.

```
pradeep@juniper apstra-terraform-4 % terraform output
asn_pool_id = "b2c9b72b-543d-4afc-bc9a-7b93414afb0e"
ipv4_pool_id = "abb107c9-8cf7-4639-b805-1c164b1e68f1"
pradeep@juniper apstra-terraform-4 % 
```

You can verify these IDs from the Terraform state.

```
pradeep@juniper apstra-terraform-4 % terraform show
# apstra_asn_pool.rfc5398:
resource "apstra_asn_pool" "rfc5398" {
    id              = "b2c9b72b-543d-4afc-bc9a-7b93414afb0e"
    name            = "RFC5398 ASNs"
    ranges          = [
        {
            first           = 64496
            last            = 64511
            status          = "pool_element_available"
            total           = 16
            used            = 0
            used_percentage = 0
        },
        {
            first           = 65536
            last            = 65551
            status          = "pool_element_available"
            total           = 16
            used            = 0
            used_percentage = 0
        },
    ]
    status          = "not_in_use"
    total           = 32
    used            = 0
    used_percentage = 0
}

# apstra_ipv4_pool.example:
resource "apstra_ipv4_pool" "example" {
    id              = "abb107c9-8cf7-4639-b805-1c164b1e68f1"
    name            = "TestPool"
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

asn_pool_id = "b2c9b72b-543d-4afc-bc9a-7b93414afb0e"
ipv4_pool_id = "abb107c9-8cf7-4639-b805-1c164b1e68f1"
pradeep@juniper apstra-terraform-4 %
```

![tf_apstra_29]({{ site.url }}{{ site.baseurl }}/assets/images/tf_apstra_29.png) 

![tf_apstra_30]({{ site.url }}{{ site.baseurl }}/assets/images/tf_apstra_30.png) 

You can use Terraform outputs to connect your Terraform projects with other parts of your infrastructure, or with other Terraform projects.

## Destroy Resources

```
pradeep@juniper apstra-terraform-4 % terraform destroy
apstra_asn_pool.rfc5398: Refreshing state... [id=b2c9b72b-543d-4afc-bc9a-7b93414afb0e]
apstra_ipv4_pool.example: Refreshing state... [id=abb107c9-8cf7-4639-b805-1c164b1e68f1]

Terraform used the selected providers to generate the following execution plan. Resource actions are indicated with the following symbols:
  - destroy

Terraform will perform the following actions:

  # apstra_asn_pool.rfc5398 will be destroyed
  - resource "apstra_asn_pool" "rfc5398" {
      - id              = "b2c9b72b-543d-4afc-bc9a-7b93414afb0e" -> null
      - name            = "RFC5398 ASNs" -> null
      - ranges          = [
          - {
              - first           = 64496 -> null
              - last            = 64511 -> null
              - status          = "pool_element_available" -> null
              - total           = 16 -> null
              - used            = 0 -> null
              - used_percentage = 0 -> null
            },
          - {
              - first           = 65536 -> null
              - last            = 65551 -> null
              - status          = "pool_element_available" -> null
              - total           = 16 -> null
              - used            = 0 -> null
              - used_percentage = 0 -> null
            },
        ] -> null
      - status          = "not_in_use" -> null
      - total           = 32 -> null
      - used            = 0 -> null
      - used_percentage = 0 -> null
    }

  # apstra_ipv4_pool.example will be destroyed
  - resource "apstra_ipv4_pool" "example" {
      - id              = "abb107c9-8cf7-4639-b805-1c164b1e68f1" -> null
      - name            = "TestPool" -> null
      - status          = "not_in_use" -> null
      - subnets         = [
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
        ] -> null
      - total           = 768 -> null
      - used            = 0 -> null
      - used_percentage = 0 -> null
    }

Plan: 0 to add, 0 to change, 2 to destroy.

Changes to Outputs:
  - asn_pool_id  = "b2c9b72b-543d-4afc-bc9a-7b93414afb0e" -> null
  - ipv4_pool_id = "abb107c9-8cf7-4639-b805-1c164b1e68f1" -> null

Do you really want to destroy all resources?
  Terraform will destroy all your managed infrastructure, as shown above.
  There is no undo. Only 'yes' will be accepted to confirm.

  Enter a value: yes

apstra_asn_pool.rfc5398: Destroying... [id=b2c9b72b-543d-4afc-bc9a-7b93414afb0e]
apstra_ipv4_pool.example: Destroying... [id=abb107c9-8cf7-4639-b805-1c164b1e68f1]
apstra_ipv4_pool.example: Destruction complete after 1s
apstra_asn_pool.rfc5398: Destruction complete after 1s

Destroy complete! Resources: 2 destroyed.
pradeep@juniper apstra-terraform-4 % 
```

Verify Terraform Outputs after destroying the resources

```
pradeep@juniper apstra-terraform-4 % terraform output     
╷
│ Warning: No outputs found
│ 
│ The state file either has no outputs defined, or all the defined outputs are empty. Please define an output in your configuration with the `output` keyword and run `terraform refresh` for it to become
│ available. If you are using interpolation, please verify the interpolated value is not empty. You can use the `terraform console` command to assist.
╵
pradeep@juniper apstra-terraform-4 % 
```

