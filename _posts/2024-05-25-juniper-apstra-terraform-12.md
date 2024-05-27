---
layout: single
title:  "Juniper Apstra Terraform Provider — Modules"
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

# Juniper Apstra Terraform Provider — Modules

> Reference from HashiCorp Documentation

### What is a Terraform module?

A Terraform module is a set of Terraform configuration files in a single directory. Even a simple configuration consisting of a single directory with one or more .tf files is a module. When you run Terraform commands directly from such a directory, it is considered the root module. 

### Calling modules

Terraform commands will only directly use the configuration files in one directory, which is usually the current working directory. However, your configuration can use module blocks to call modules in other directories. When Terraform encounters a module block, it loads and processes that module's configuration files.

A module that is called by another configuration is sometimes referred to as a "child module" of that configuration.

### Local and remote modules

Modules can either be loaded from the local filesystem, or a remote source. Terraform supports a variety of remote sources, including the Terraform Registry, most version control systems, HTTP URLs, and Terraform Cloud or Terraform Enterprise private module registries.

In this post, let us create two local modules, one for Apstra Tags and another module for Apstra ASN pools.

Here is the main Terraform configuration file, referencing the local modules with the `module` block.

```
pradeep@juniper apstra-terraform-9 % cat main.tf
terraform {
  required_providers {
    apstra = {
      source  = "Juniper/apstra"
      version = ">= 0.28.0"
    }
  }
  required_version = ">= 1.1"
}

module "tags" {
  source = "./tags"
}

module "asnpools" {
  source = "./asnpools"
} 

pradeep@juniper apstra-terraform-9 %
```

Here is the Terraform configuration definition for the Juniper Apstra Tags, defined within the `tags` directory.

```
pradeep@juniper apstra-terraform-9 % cat tags/main.tf
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
  url = "https://admin:ApstraT3rraf0rm!@10.210.40.194:443"
  tls_validation_disabled = true
  blueprint_mutex_enabled = true
}


# this example creates a tags named after enterprise teams
# responsible for various data center asset types.
locals {
  device_owners = toset([
    "research",
    "security",
    "app team 1",
    "app team 2",
  ])
}

resource "apstra_tag" "example" {
  for_each    = local.device_owners
  name        = each.key
  description = format("device maintained by %q team", each.key)
}

pradeep@juniper apstra-terraform-9 %

```
And here is the Terraform configuration definition for the Juniper Apstra ASN pool, defined within the `asnpools` directory.

```
pradeep@juniper apstra-terraform-9 % cat asnpools/main.tf
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
  url = "https://admin:ApstraT3rraf0rm!@10.210.40.194:443"
  tls_validation_disabled = true
  blueprint_mutex_enabled = true
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
pradeep@juniper apstra-terraform-9 %
```



## Init
When you run `terraform init`,  you can see modules are initialized.

```
pradeep@juniper apstra-terraform-9 % terraform init

Initializing the backend...
Initializing modules...

Initializing provider plugins...
- Finding juniper/apstra versions matching ">= 0.28.0"...
- Installing juniper/apstra v0.44.0...
- Installed juniper/apstra v0.44.0 (signed by a HashiCorp partner, key ID CB9C922903A66F3F)

Partner and community providers are signed by their developers.
If you'd like to know more about provider signing, you can read about it here:
https://www.terraform.io/docs/cli/plugins/signing.html

Terraform has created a lock file .terraform.lock.hcl to record the provider
selections it made above. Include this file in your version control repository
so that Terraform can guarantee to make the same selections by default when
you run "terraform init" in the future.

Terraform has been successfully initialized!

You may now begin working with Terraform. Try running "terraform plan" to see
any changes that are required for your infrastructure. All Terraform commands
should now work.

If you ever set or change modules or backend configuration for Terraform,
rerun this command to reinitialize your working directory. If you forget, other
commands will detect it and remind you to do so if necessary.

pradeep@juniper apstra-terraform-9 %
```

## Plan
With modules, you can see that the resource action is applied to the  individual resources defined within the modules.

```
pradeep@juniper apstra-terraform-9 % terraform plan
Terraform used the selected providers to generate the following execution plan. Resource actions are indicated with the following symbols:
  + create

Terraform will perform the following actions:

  # module.asnpools.apstra_asn_pool.rfc5398 will be created
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

  # module.tags.apstra_tag.example["app team 1"] will be created
  + resource "apstra_tag" "example" {
      + description = "device maintained by \"app team 1\" team"
      + id          = (known after apply)
      + name        = "app team 1"
    }

  # module.tags.apstra_tag.example["app team 2"] will be created
  + resource "apstra_tag" "example" {
      + description = "device maintained by \"app team 2\" team"
      + id          = (known after apply)
      + name        = "app team 2"
    }

  # module.tags.apstra_tag.example["research"] will be created
  + resource "apstra_tag" "example" {
      + description = "device maintained by \"research\" team"
      + id          = (known after apply)
      + name        = "research"
    }

  # module.tags.apstra_tag.example["security"] will be created
  + resource "apstra_tag" "example" {
      + description = "device maintained by \"security\" team"
      + id          = (known after apply)
      + name        = "security"
    }

Plan: 5 to add, 0 to change, 0 to destroy.

──────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────

Note: You didn't use the -out option to save this plan, so Terraform can't guarantee to take exactly these actions if you run "terraform apply" now.

pradeep@juniper apstra-terraform-9 %
```

## Apply

```
pradeep@juniper apstra-terraform-9 % terraform apply

Terraform used the selected providers to generate the following execution plan. Resource actions are indicated with the following symbols:
  + create

Terraform will perform the following actions:

  # module.asnpools.apstra_asn_pool.rfc5398 will be created
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

  # module.tags.apstra_tag.example["app team 1"] will be created
  + resource "apstra_tag" "example" {
      + description = "device maintained by \"app team 1\" team"
      + id          = (known after apply)
      + name        = "app team 1"
    }

  # module.tags.apstra_tag.example["app team 2"] will be created
  + resource "apstra_tag" "example" {
      + description = "device maintained by \"app team 2\" team"
      + id          = (known after apply)
      + name        = "app team 2"
    }

  # module.tags.apstra_tag.example["research"] will be created
  + resource "apstra_tag" "example" {
      + description = "device maintained by \"research\" team"
      + id          = (known after apply)
      + name        = "research"
    }

  # module.tags.apstra_tag.example["security"] will be created
  + resource "apstra_tag" "example" {
      + description = "device maintained by \"security\" team"
      + id          = (known after apply)
      + name        = "security"
    }

Plan: 5 to add, 0 to change, 0 to destroy.

Do you want to perform these actions?
  Terraform will perform the actions described above.
  Only 'yes' will be accepted to approve.

  Enter a value: yes

module.tags.apstra_tag.example["app team 1"]: Creating...
module.tags.apstra_tag.example["research"]: Creating...
module.tags.apstra_tag.example["security"]: Creating...
module.tags.apstra_tag.example["app team 2"]: Creating...
module.asnpools.apstra_asn_pool.rfc5398: Creating...
module.tags.apstra_tag.example["security"]: Creation complete after 2s [id=301f5238-2895-41af-876d-abc54705e3f4]
module.tags.apstra_tag.example["app team 2"]: Creation complete after 2s [id=3f04be69-5edd-47c7-8664-b6b97d5e8827]
module.tags.apstra_tag.example["app team 1"]: Creation complete after 2s [id=640fdaee-0675-4723-a0e0-fdfc538ec382]
module.tags.apstra_tag.example["research"]: Creation complete after 2s [id=80140541-380c-4b31-9a77-27524121eaef]
module.asnpools.apstra_asn_pool.rfc5398: Creation complete after 2s [id=084f3659-24fa-4c05-9b5c-2989199bda00]

Apply complete! Resources: 5 added, 0 changed, 0 destroyed.

pradeep@juniper apstra-terraform-9 %
```

![tf_apstra_36]({{ site.url }}{{ site.baseurl }}/assets/images/tf_apstra_36.png) 

![tf_apstra_37]({{ site.url }}{{ site.baseurl }}/assets/images/tf_apstra_37.png) 

## Destroy

```
pradeep@juniper apstra-terraform-9 % terraform destroy

module.tags.apstra_tag.example["app team 2"]: Refreshing state... [id=3f04be69-5edd-47c7-8664-b6b97d5e8827]
module.tags.apstra_tag.example["research"]: Refreshing state... [id=80140541-380c-4b31-9a77-27524121eaef]
module.tags.apstra_tag.example["app team 1"]: Refreshing state... [id=640fdaee-0675-4723-a0e0-fdfc538ec382]
module.tags.apstra_tag.example["security"]: Refreshing state... [id=301f5238-2895-41af-876d-abc54705e3f4]
module.asnpools.apstra_asn_pool.rfc5398: Refreshing state... [id=084f3659-24fa-4c05-9b5c-2989199bda00]

Terraform used the selected providers to generate the following execution plan. Resource actions are indicated with the following symbols:
  - destroy

Terraform will perform the following actions:

  # module.asnpools.apstra_asn_pool.rfc5398 will be destroyed
  - resource "apstra_asn_pool" "rfc5398" {
      - id              = "084f3659-24fa-4c05-9b5c-2989199bda00" -> null
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

  # module.tags.apstra_tag.example["app team 1"] will be destroyed
  - resource "apstra_tag" "example" {
      - description = "device maintained by \"app team 1\" team" -> null
      - id          = "640fdaee-0675-4723-a0e0-fdfc538ec382" -> null
      - name        = "app team 1" -> null
    }

  # module.tags.apstra_tag.example["app team 2"] will be destroyed
  - resource "apstra_tag" "example" {
      - description = "device maintained by \"app team 2\" team" -> null
      - id          = "3f04be69-5edd-47c7-8664-b6b97d5e8827" -> null
      - name        = "app team 2" -> null
    }

  # module.tags.apstra_tag.example["research"] will be destroyed
  - resource "apstra_tag" "example" {
      - description = "device maintained by \"research\" team" -> null
      - id          = "80140541-380c-4b31-9a77-27524121eaef" -> null
      - name        = "research" -> null
    }

  # module.tags.apstra_tag.example["security"] will be destroyed
  - resource "apstra_tag" "example" {
      - description = "device maintained by \"security\" team" -> null
      - id          = "301f5238-2895-41af-876d-abc54705e3f4" -> null
      - name        = "security" -> null
    }

Plan: 0 to add, 0 to change, 5 to destroy.

Do you really want to destroy all resources?
  Terraform will destroy all your managed infrastructure, as shown above.
  There is no undo. Only 'yes' will be accepted to confirm.

  Enter a value: yes

module.tags.apstra_tag.example["research"]: Destroying... [id=80140541-380c-4b31-9a77-27524121eaef]
module.tags.apstra_tag.example["app team 1"]: Destroying... [id=640fdaee-0675-4723-a0e0-fdfc538ec382]
module.tags.apstra_tag.example["security"]: Destroying... [id=301f5238-2895-41af-876d-abc54705e3f4]
module.tags.apstra_tag.example["app team 2"]: Destroying... [id=3f04be69-5edd-47c7-8664-b6b97d5e8827]
module.asnpools.apstra_asn_pool.rfc5398: Destroying... [id=084f3659-24fa-4c05-9b5c-2989199bda00]
module.tags.apstra_tag.example["app team 2"]: Destruction complete after 1s
module.asnpools.apstra_asn_pool.rfc5398: Destruction complete after 1s
module.tags.apstra_tag.example["app team 1"]: Destruction complete after 1s
module.tags.apstra_tag.example["security"]: Destruction complete after 1s
module.tags.apstra_tag.example["research"]: Destruction complete after 1s

Destroy complete! Resources: 5 destroyed.

pradeep@juniper apstra-terraform-9 %
```

