---
layout: single
title:  "Juniper Apstra Terraform Provider — Data Sources"
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

# Juniper Apstra Terraform Provider — Data Sources

Data sources allow Terraform to use information defined outside of Terraform, defined by another separate Terraform configuration, or modified by functions.

Terraform data sources let you dynamically fetch data from APIs or other Terraform state backends. Examples of data sources include ASN Pool IDs from Juniper Apstra or Terraform outputs from other configurations. Data sources make your configuration more flexible and dynamic and let you reference values from other configurations, helping you scope your configuration while still referencing any dependent resource attributes.

Each provider may offer data sources alongside its set of resource types.

Juniper Apstra also offers many data sources.

The following example shows outputting all ASN pool IDs from Juniper Apstra.

```
pradeep@juniper % cat main.tf 
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


data "apstra_asn_pools" "all" {}

output asn_pool_ids {
   value = data.apstra_asn_pools.all.ids
}

pradeep@juniper % 
```

A data source is accessed via a special kind of resource known as a *data resource*, declared using a `data` block:

A `data` block requests that Terraform read from a given data source ("apstra_asn_pool") and export the result under the given local name ("all"). The name is used to refer to this resource from elsewhere in the same Terraform module, but has no significance outside of the scope of a module.

The data source and name together serve as an identifier for a given resource and so must be unique within a module.

## Terraform Init

```
pradeep@juniper % terraform init

Initializing the backend...

Initializing provider plugins...
- Finding juniper/apstra versions matching ">= 0.28.0"...
- Installing juniper/apstra v0.42.0...
- Installed juniper/apstra v0.42.0 (signed by a HashiCorp partner, key ID CB9C922903A66F3F)

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
pradeep@juniper % 
```

## Terraform Apply

```
pradeep@juniper % terraform apply
data.apstra_asn_pools.all: Reading...
data.apstra_asn_pools.all: Read complete after 1s

Changes to Outputs:
  + asn_pool_ids = [
      + "Private-4200000000-4294967294",
      + "Private-64512-65534",
    ]

You can apply this plan to save these new output values to the Terraform state, without changing any real infrastructure.

Do you want to perform these actions?
  Terraform will perform the actions described above.
  Only 'yes' will be accepted to approve.

  Enter a value: yes


Apply complete! Resources: 0 added, 0 changed, 0 destroyed.

Outputs:

asn_pool_ids = toset([
  "Private-4200000000-4294967294",
  "Private-64512-65534",
])
pradeep@juniper % 
```

![tf_apstra_33]({{ site.url }}{{ site.baseurl }}/assets/images/tf_apstra_33.png) 

Let’s take a look at another data source configuration

The following example shows outputting a report of free space across all ASN resource pools:

```
pradeep@juniper % cat main.tf 
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

# The following example shows outputting a report of free space across all
# ASN resource pools:

data "apstra_asn_pools" "all" {}

data "apstra_asn_pool" "all" {
  for_each = toset(data.apstra_asn_pools.all.ids)
  id = each.value
}

output "asn_report" {
  value = {for k, v in data.apstra_asn_pool.all : k => {
    name = v.name
    free = v.total - v.used
  }}
}
pradeep@juniper %
```

Apply

```
pradeep@juniper % terraform apply
data.apstra_asn_pools.all: Reading...
data.apstra_asn_pools.all: Read complete after 1s
data.apstra_asn_pool.all["Private-64512-65534"]: Reading...
data.apstra_asn_pool.all["Private-4200000000-4294967294"]: Reading...
data.apstra_asn_pool.all["Private-64512-65534"]: Read complete after 0s [id=Private-64512-65534]
data.apstra_asn_pool.all["Private-4200000000-4294967294"]: Read complete after 1s [id=Private-4200000000-4294967294]

Changes to Outputs:
  - asn_pool_ids = [
      - "Private-4200000000-4294967294",
      - "Private-64512-65534",
    ] -> null
  + asn_report   = {
      + Private-4200000000-4294967294 = {
          + free = 94967295
          + name = "Private-4200000000-4294967294"
        }
      + Private-64512-65534           = {
          + free = 1023
          + name = "Private-64512-65534"
        }
    }

You can apply this plan to save these new output values to the Terraform state, without changing any real infrastructure.

Do you want to perform these actions?
  Terraform will perform the actions described above.
  Only 'yes' will be accepted to approve.

  Enter a value: yes


Apply complete! Resources: 0 added, 0 changed, 0 destroyed.

Outputs:

asn_report = {
  "Private-4200000000-4294967294" = {
    "free" = 94967295
    "name" = "Private-4200000000-4294967294"
  }
  "Private-64512-65534" = {
    "free" = 1023
    "name" = "Private-64512-65534"
  }
}
pradeep@juniper %
```

