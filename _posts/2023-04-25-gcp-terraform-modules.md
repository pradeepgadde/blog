---
layout: single
title:  "Interact with Terraform Modules"
date:   2023-04-25 00:59:04 +0530
categories: Cloud
tags: GCP
classes: wide
show_date: true
header:
  overlay_image: /assets/images/gcp-banner-1.png
  og_image: /assets/images/gcp-banner-1.png
  teaser: /assets/images/gpcne.png
author:
  name     : "Cloud Network Engineer"
  avatar   : "/assets/images/gpcne.png"

sidebar:
  - title: "Topics"
    nav: my-sidebar

---

# Interact with Terraform Modules

## Overview

- Use a module from the Registry
- Build a module

A Terraform module is a set of Terraform configuration files in a single directory. Even a simple configuration consisting of a single directory with one or more `.tf` files is a module. When you run Terraform commands directly from such a directory, it is considered the **root module**. A module that is called by another configuration is sometimes referred to as a "child module" of that configuration.

Modules can be loaded from either the local filesystem or a remote  source. Terraform supports a variety of remote sources, including the  Terraform Registry, most version control systems, HTTP URLs, and  Terraform Cloud or Terraform Enterprise private module registries.



## 1. Use modules from the Registry

In this section, you use modules from the [Terraform Registry](https://registry.terraform.io/) to provision an example environment in Google Cloud. The concepts you use here will apply to any modules from any source.



### Create a Terraform configuration

1. To start, run the following commands in Cloud Shell to clone the  example simple project from the Google Terraform modules GitHub  repository and switch to the `v6.0.1` branch:

   ```sh
   Welcome to Cloud Shell! Type "help" to get started.
   Your Cloud Platform project in this session is set to qwiklabs-gcp-00-7e4d42cbcac7.
   Use “gcloud config set project [PROJECT_ID]” to change to a different project.
   student_01_b9934b2a0bae@cloudshell:~ (qwiklabs-gcp-00-7e4d42cbcac7)$ git clone https://github.com/terraform-google-modules/terraform-google-network
   cd terraform-google-network
   git checkout tags/v6.0.1 -b v6.0.1
   Cloning into 'terraform-google-network'...
   remote: Enumerating objects: 4579, done.
   remote: Counting objects: 100% (418/418), done.
   remote: Compressing objects: 100% (169/169), done.
   remote: Total 4579 (delta 273), reused 368 (delta 240), pack-reused 4161
   Receiving objects: 100% (4579/4579), 1007.49 KiB | 1.14 MiB/s, done.
   Resolving deltas: 100% (2982/2982), done.
   Switched to a new branch 'v6.0.1'
   student_01_b9934b2a0bae@cloudshell:~/terraform-google-network (qwiklabs-gcp-00-7e4d42cbcac7)$ ls
   build         codelabs    CONTRIBUTING.md  examples  LICENSE  Makefile       modules     README.md  variables.tf
   CHANGELOG.md  CODEOWNERS  docs             helpers   main.tf  metadata.yaml  outputs.tf  test       versions.tf
   student_01_b9934b2a0bae@cloudshell:~/terraform-google-network (qwiklabs-gcp-00-7e4d42cbcac7)$
   ```

   

```sh
student_01_b9934b2a0bae@cloudshell:~/terraform-google-network (qwiklabs-gcp-00-7e4d42cbcac7)$ cat main.tf
/**
 * Copyright 2019 Google LLC
 *
 * Licensed under the Apache License, Version 2.0 (the "License");
 * you may not use this file except in compliance with the License.
 * You may obtain a copy of the License at
 *
 *      http://www.apache.org/licenses/LICENSE-2.0
 *
 * Unless required by applicable law or agreed to in writing, software
 * distributed under the License is distributed on an "AS IS" BASIS,
 * WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
 * See the License for the specific language governing permissions and
 * limitations under the License.
 */

/******************************************
        VPC configuration
 *****************************************/
module "vpc" {
  source                                 = "./modules/vpc"
  network_name                           = var.network_name
  auto_create_subnetworks                = var.auto_create_subnetworks
  routing_mode                           = var.routing_mode
  project_id                             = var.project_id
  description                            = var.description
  shared_vpc_host                        = var.shared_vpc_host
  delete_default_internet_gateway_routes = var.delete_default_internet_gateway_routes
  mtu                                    = var.mtu
}

/******************************************
        Subnet configuration
 *****************************************/
module "subnets" {
  source           = "./modules/subnets"
  project_id       = var.project_id
  network_name     = module.vpc.network_name
  subnets          = var.subnets
  secondary_ranges = var.secondary_ranges
}

/******************************************
        Routes
 *****************************************/
module "routes" {
  source            = "./modules/routes"
  project_id        = var.project_id
  network_name      = module.vpc.network_name
  routes            = var.routes
  module_depends_on = [module.subnets.subnets]
}

/******************************************
        Firewall rules
 *****************************************/
locals {
  rules = [
    for f in var.firewall_rules : {
      name                    = f.name
      direction               = f.direction
      priority                = lookup(f, "priority", null)
      description             = lookup(f, "description", null)
      ranges                  = lookup(f, "ranges", null)
      source_tags             = lookup(f, "source_tags", null)
      source_service_accounts = lookup(f, "source_service_accounts", null)
      target_tags             = lookup(f, "target_tags", null)
      target_service_accounts = lookup(f, "target_service_accounts", null)
      allow                   = lookup(f, "allow", [])
      deny                    = lookup(f, "deny", [])
      log_config              = lookup(f, "log_config", null)
    }
  ]
}

module "firewall_rules" {
  source       = "./modules/firewall-rules"
  project_id   = var.project_id
  network_name = module.vpc.network_name
  rules        = local.rules
}
student_01_b9934b2a0bae@cloudshell:~/terraform-google-network (qwiklabs-gcp-00-7e4d42cbcac7)$
```



```sh
student_01_b9934b2a0bae@cloudshell:~/terraform-google-network (qwiklabs-gcp-00-7e4d42cbcac7)$ cat variables.tf
/**
 * Copyright 2019 Google LLC
 *
 * Licensed under the Apache License, Version 2.0 (the "License");
 * you may not use this file except in compliance with the License.
 * You may obtain a copy of the License at
 *
 *      http://www.apache.org/licenses/LICENSE-2.0
 *
 * Unless required by applicable law or agreed to in writing, software
 * distributed under the License is distributed on an "AS IS" BASIS,
 * WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
 * See the License for the specific language governing permissions and
 * limitations under the License.
 */

variable "project_id" {
  description = "The ID of the project where this VPC will be created"
  type        = string
}

variable "network_name" {
  description = "The name of the network being created"
  type        = string
}

variable "routing_mode" {
  type        = string
  default     = "GLOBAL"
  description = "The network routing mode (default 'GLOBAL')"
}

variable "shared_vpc_host" {
  type        = bool
  description = "Makes this project a Shared VPC host if 'true' (default 'false')"
  default     = false
}

variable "subnets" {
  type        = list(map(string))
  description = "The list of subnets being created"
}

variable "secondary_ranges" {
  type        = map(list(object({ range_name = string, ip_cidr_range = string })))
  description = "Secondary ranges that will be used in some of the subnets"
  default     = {}
}

variable "routes" {
  type        = list(map(string))
  description = "List of routes being created in this VPC"
  default     = []
}

variable "firewall_rules" {
  type        = any
  description = "List of firewall rules"
  default     = []
}

variable "delete_default_internet_gateway_routes" {
  type        = bool
  description = "If set, ensure that all routes within the network specified whose names begin with 'default-route' and with a next hop of 'default-internet-gateway' are deleted"
  default     = false
}


variable "description" {
  type        = string
  description = "An optional description of this resource. The resource must be recreated to modify this field."
  default     = ""
}

variable "auto_create_subnetworks" {
  type        = bool
  description = "When set to true, the network is created in 'auto subnet mode' and it will create a subnet for each region automatically across the 10.128.0.0/9 address range. When set to false, the network is created in 'custom subnet mode' so the user can explicitly connect subnetwork resources."
  default     = false
}

variable "mtu" {
  type        = number
  description = "The network MTU (If set to 0, meaning MTU is unset - defaults to '1460'). Recommended values: 1460 (default for historic reasons), 1500 (Internet default), or 8896 (for Jumbo packets). Allowed are all values in the range 1300 to 8896, inclusively."
  default     = 0
}
student_01_b9934b2a0bae@cloudshell:~/terraform-google-network (qwiklabs-gcp-00-7e4d42cbcac7)$
```

```sh
student_01_b9934b2a0bae@cloudshell:~/terraform-google-network (qwiklabs-gcp-00-7e4d42cbcac7)$ cat outputs.tf
/**
 * Copyright 2019 Google LLC
 *
 * Licensed under the Apache License, Version 2.0 (the "License");
 * you may not use this file except in compliance with the License.
 * You may obtain a copy of the License at
 *
 *      http://www.apache.org/licenses/LICENSE-2.0
 *
 * Unless required by applicable law or agreed to in writing, software
 * distributed under the License is distributed on an "AS IS" BASIS,
 * WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
 * See the License for the specific language governing permissions and
 * limitations under the License.
 */

output "network" {
  value       = module.vpc
  description = "The created network"
}

output "subnets" {
  value       = module.subnets.subnets
  description = "A map with keys of form subnet_region/subnet_name and values being the outputs of the google_compute_subnetwork resources used to create corresponding subnets."
}

output "network_name" {
  value       = module.vpc.network_name
  description = "The name of the VPC being created"
}

output "network_id" {
  value       = module.vpc.network_id
  description = "The ID of the VPC being created"
}

output "network_self_link" {
  value       = module.vpc.network_self_link
  description = "The URI of the VPC being created"
}

output "project_id" {
  value       = module.vpc.project_id
  description = "VPC project id"
}

output "subnets_names" {
  value       = [for network in module.subnets.subnets : network.name]
  description = "The names of the subnets being created"
}

output "subnets_ids" {
  value       = [for network in module.subnets.subnets : network.id]
  description = "The IDs of the subnets being created"
}

output "subnets_ips" {
  value       = [for network in module.subnets.subnets : network.ip_cidr_range]
  description = "The IPs and CIDRs of the subnets being created"
}

output "subnets_self_links" {
  value       = [for network in module.subnets.subnets : network.self_link]
  description = "The self-links of subnets being created"
}

output "subnets_regions" {
  value       = [for network in module.subnets.subnets : network.region]
  description = "The region where the subnets will be created"
}

output "subnets_private_access" {
  value       = [for network in module.subnets.subnets : network.private_ip_google_access]
  description = "Whether the subnets will have access to Google API's without a public IP"
}

output "subnets_flow_logs" {
  value       = [for network in module.subnets.subnets : length(network.log_config) != 0 ? true : false]
  description = "Whether the subnets will have VPC flow logs enabled"
}

output "subnets_secondary_ranges" {
  value       = [for network in module.subnets.subnets : network.secondary_ip_range]
  description = "The secondary ranges associated with these subnets"
}

output "route_names" {
  value       = [for route in module.routes.routes : route.name]
  description = "The route names associated with this VPC"
}
student_01_b9934b2a0bae@cloudshell:~/terraform-google-network (qwiklabs-gcp-00-7e4d42cbcac7)$
```

```sh
student_01_b9934b2a0bae@cloudshell:~/terraform-google-network (qwiklabs-gcp-00-7e4d42cbcac7)$ gcloud config list --format 'value(core.project)'
qwiklabs-gcp-00-7e4d42cbcac7
student_01_b9934b2a0bae@cloudshell:~/terraform-google-network (qwiklabs-gcp-00-7e4d42cbcac7)$ cd ~/terraform-google-network/examples/simple_project
student_01_b9934b2a0bae@cloudshell:~/terraform-google-network/examples/simple_project (qwiklabs-gcp-00-7e4d42cbcac7)$ ls
main.tf  outputs.tf  README.md  variables.tf  versions.tf
student_01_b9934b2a0bae@cloudshell:~/terraform-google-network/examples/simple_project (qwiklabs-gcp-00-7e4d42cbcac7)$ cat main.tf
/**
 * Copyright 2019 Google LLC
 *
 * Licensed under the Apache License, Version 2.0 (the "License");
 * you may not use this file except in compliance with the License.
 * You may obtain a copy of the License at
 *
 *      http://www.apache.org/licenses/LICENSE-2.0
 *
 * Unless required by applicable law or agreed to in writing, software
 * distributed under the License is distributed on an "AS IS" BASIS,
 * WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
 * See the License for the specific language governing permissions and
 * limitations under the License.
 */

# Whenever a new major version of the network module is released, the
# version constraint below should be updated, e.g. to ~> 4.0.
#
# If that new version includes provider updates, validation of this
# example may fail until that is done.

# [START vpc_custom_create]
module "test-vpc-module" {
  source       = "terraform-google-modules/network/google"
  version      = "~> 6.0"
  project_id   = var.project_id # Replace this with your project ID in quotes
  network_name = "my-custom-mode-network"
  mtu          = 1460

  subnets = [
    {
      subnet_name   = "subnet-01"
      subnet_ip     = "10.10.10.0/24"
      subnet_region = "us-west1"
    },
    {
      subnet_name           = "subnet-02"
      subnet_ip             = "10.10.20.0/24"
      subnet_region         = "us-west1"
      subnet_private_access = "true"
      subnet_flow_logs      = "true"
    },
    {
      subnet_name               = "subnet-03"
      subnet_ip                 = "10.10.30.0/24"
      subnet_region             = "us-west1"
      subnet_flow_logs          = "true"
      subnet_flow_logs_interval = "INTERVAL_10_MIN"
      subnet_flow_logs_sampling = 0.7
      subnet_flow_logs_metadata = "INCLUDE_ALL_METADATA"
      subnet_flow_logs_filter   = "false"
    }
  ]
}
# [END vpc_custom_create]
student_01_b9934b2a0bae@cloudshell:~/terraform-google-network/examples/simple_project (qwiklabs-gcp-00-7e4d42cbcac7)$ ls
main.tf  outputs.tf  README.md  variables.tf  versions.tf
student_01_b9934b2a0bae@cloudshell:~/terraform-google-network/examples/simple_project (qwiklabs-gcp-00-7e4d42cbcac7)$ cat outputs.tf
/**
 * Copyright 2019 Google LLC
 *
 * Licensed under the Apache License, Version 2.0 (the "License");
 * you may not use this file except in compliance with the License.
 * You may obtain a copy of the License at
 *
 *      http://www.apache.org/licenses/LICENSE-2.0
 *
 * Unless required by applicable law or agreed to in writing, software
 * distributed under the License is distributed on an "AS IS" BASIS,
 * WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
 * See the License for the specific language governing permissions and
 * limitations under the License.
 */

output "network_name" {
  value       = module.test-vpc-module.network_name
  description = "The name of the VPC being created"
}

output "network_self_link" {
  value       = module.test-vpc-module.network_self_link
  description = "The URI of the VPC being created"
}

output "project_id" {
  value       = module.test-vpc-module.project_id
  description = "VPC project id"
}

output "subnets_names" {
  value       = module.test-vpc-module.subnets_names
  description = "The names of the subnets being created"
}

output "subnets_ips" {
  value       = module.test-vpc-module.subnets_ips
  description = "The IP and cidrs of the subnets being created"
}

output "subnets_regions" {
  value       = module.test-vpc-module.subnets_regions
  description = "The region where subnets will be created"
}

output "subnets_private_access" {
  value       = module.test-vpc-module.subnets_private_access
  description = "Whether the subnets will have access to Google API's without a public IP"
}

output "subnets_flow_logs" {
  value       = module.test-vpc-module.subnets_flow_logs
  description = "Whether the subnets will have VPC flow logs enabled"
}

output "subnets_secondary_ranges" {
  value       = module.test-vpc-module.subnets_secondary_ranges
  description = "The secondary ranges associated with these subnets"
}

output "route_names" {
  value       = module.test-vpc-module.route_names
  description = "The routes associated with this VPC"
}
student_01_b9934b2a0bae@cloudshell:~/terraform-google-network/examples/simple_project (qwiklabs-gcp-00-7e4d42cbcac7)$ cat variables.tf
/**
 * Copyright 2019 Google LLC
 *
 * Licensed under the Apache License, Version 2.0 (the "License");
 * you may not use this file except in compliance with the License.
 * You may obtain a copy of the License at
 *
 *      http://www.apache.org/licenses/LICENSE-2.0
 *
 * Unless required by applicable law or agreed to in writing, software
 * distributed under the License is distributed on an "AS IS" BASIS,
 * WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
 * See the License for the specific language governing permissions and
 * limitations under the License.
 */

variable "project_id" {
  description = "The project ID to host the network in"
}
student_01_b9934b2a0bae@cloudshell:~/terraform-google-network/examples/simple_project (qwiklabs-gcp-00-7e4d42cbcac7)$
```

```sh
student_01_b9934b2a0bae@cloudshell:~/terraform-google-network/examples/simple_project (qwiklabs-gcp-00-7e4d42cbcac7)$ terraform init

Initializing the backend...
Initializing modules...
Downloading registry.terraform.io/terraform-google-modules/network/google 6.0.1 for test-vpc-module...
- test-vpc-module in .terraform/modules/test-vpc-module
- test-vpc-module.firewall_rules in .terraform/modules/test-vpc-module/modules/firewall-rules
- test-vpc-module.routes in .terraform/modules/test-vpc-module/modules/routes
- test-vpc-module.subnets in .terraform/modules/test-vpc-module/modules/subnets
- test-vpc-module.vpc in .terraform/modules/test-vpc-module/modules/vpc

Initializing provider plugins...
- Finding hashicorp/google versions matching ">= 2.15.0, >= 3.33.0, >= 3.45.0, >= 3.83.0, ~> 4.0, < 5.0.0"...
- Finding hashicorp/null versions matching "~> 3.0"...
- Finding hashicorp/google-beta versions matching ">= 3.45.0, < 5.0.0"...
- Installing hashicorp/google-beta v4.63.0...
- Installed hashicorp/google-beta v4.63.0 (signed by HashiCorp)
- Installing hashicorp/google v4.63.0...
- Installed hashicorp/google v4.63.0 (signed by HashiCorp)
- Installing hashicorp/null v3.2.1...
- Installed hashicorp/null v3.2.1 (signed by HashiCorp)

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
student_01_b9934b2a0bae@cloudshell:~/terraform-google-network/examples/simple_project (qwiklabs-gcp-00-7e4d42cbcac7)$
```



```sh
student_01_b9934b2a0bae@cloudshell:~/terraform-google-network/examples/simple_project (qwiklabs-gcp-00-7e4d42cbcac7)$ terraform apply
var.project_id
  The project ID to host the network in

  Enter a value: qwiklabs-gcp-00-7e4d42cbcac7


Terraform used the selected providers to generate the following execution plan. Resource actions are indicated with the following symbols:
  + create

Terraform will perform the following actions:

  # module.test-vpc-module.module.subnets.google_compute_subnetwork.subnetwork["us-west1/subnet-01"] will be created
  + resource "google_compute_subnetwork" "subnetwork" {
      + creation_timestamp         = (known after apply)
      + external_ipv6_prefix       = (known after apply)
      + fingerprint                = (known after apply)
      + gateway_address            = (known after apply)
      + id                         = (known after apply)
      + ip_cidr_range              = "10.10.10.0/24"
      + ipv6_cidr_range            = (known after apply)
      + name                       = "subnet-01"
      + network                    = "my-custom-mode-network"
      + private_ip_google_access   = false
      + private_ipv6_google_access = (known after apply)
      + project                    = "qwiklabs-gcp-00-7e4d42cbcac7"
      + purpose                    = (known after apply)
      + region                     = "us-west1"
      + secondary_ip_range         = []
      + self_link                  = (known after apply)
      + stack_type                 = (known after apply)
    }

  # module.test-vpc-module.module.subnets.google_compute_subnetwork.subnetwork["us-west1/subnet-02"] will be created
  + resource "google_compute_subnetwork" "subnetwork" {
      + creation_timestamp         = (known after apply)
      + external_ipv6_prefix       = (known after apply)
      + fingerprint                = (known after apply)
      + gateway_address            = (known after apply)
      + id                         = (known after apply)
      + ip_cidr_range              = "10.10.20.0/24"
      + ipv6_cidr_range            = (known after apply)
      + name                       = "subnet-02"
      + network                    = "my-custom-mode-network"
      + private_ip_google_access   = true
      + private_ipv6_google_access = (known after apply)
      + project                    = "qwiklabs-gcp-00-7e4d42cbcac7"
      + purpose                    = (known after apply)
      + region                     = "us-west1"
      + secondary_ip_range         = []
      + self_link                  = (known after apply)
      + stack_type                 = (known after apply)

      + log_config {
          + aggregation_interval = "INTERVAL_5_SEC"
          + filter_expr          = "true"
          + flow_sampling        = 0.5
          + metadata             = "INCLUDE_ALL_METADATA"
        }
    }

  # module.test-vpc-module.module.subnets.google_compute_subnetwork.subnetwork["us-west1/subnet-03"] will be created
  + resource "google_compute_subnetwork" "subnetwork" {
      + creation_timestamp         = (known after apply)
      + external_ipv6_prefix       = (known after apply)
      + fingerprint                = (known after apply)
      + gateway_address            = (known after apply)
      + id                         = (known after apply)
      + ip_cidr_range              = "10.10.30.0/24"
      + ipv6_cidr_range            = (known after apply)
      + name                       = "subnet-03"
      + network                    = "my-custom-mode-network"
      + private_ip_google_access   = false
      + private_ipv6_google_access = (known after apply)
      + project                    = "qwiklabs-gcp-00-7e4d42cbcac7"
      + purpose                    = (known after apply)
      + region                     = "us-west1"
      + secondary_ip_range         = []
      + self_link                  = (known after apply)
      + stack_type                 = (known after apply)

      + log_config {
          + aggregation_interval = "INTERVAL_10_MIN"
          + filter_expr          = "false"
          + flow_sampling        = 0.7
          + metadata             = "INCLUDE_ALL_METADATA"
        }
    }

  # module.test-vpc-module.module.vpc.google_compute_network.network will be created
  + resource "google_compute_network" "network" {
      + auto_create_subnetworks                   = false
      + delete_default_routes_on_create           = false
      + gateway_ipv4                              = (known after apply)
      + id                                        = (known after apply)
      + internal_ipv6_range                       = (known after apply)
      + mtu                                       = 1460
      + name                                      = "my-custom-mode-network"
      + network_firewall_policy_enforcement_order = "AFTER_CLASSIC_FIREWALL"
      + project                                   = "qwiklabs-gcp-00-7e4d42cbcac7"
      + routing_mode                              = "GLOBAL"
      + self_link                                 = (known after apply)
    }

Plan: 4 to add, 0 to change, 0 to destroy.

Changes to Outputs:
  + network_name             = "my-custom-mode-network"
  + network_self_link        = (known after apply)
  + project_id               = "qwiklabs-gcp-00-7e4d42cbcac7"
  + route_names              = []
  + subnets_flow_logs        = [
      + false,
      + true,
      + true,
    ]
  + subnets_ips              = [
      + "10.10.10.0/24",
      + "10.10.20.0/24",
      + "10.10.30.0/24",
    ]
  + subnets_names            = [
      + "subnet-01",
      + "subnet-02",
      + "subnet-03",
    ]
  + subnets_private_access   = [
      + false,
      + true,
      + false,
    ]
  + subnets_regions          = [
      + "us-west1",
      + "us-west1",
      + "us-west1",
    ]
  + subnets_secondary_ranges = [
      + [],
      + [],
      + [],
    ]

Do you want to perform these actions?
  Terraform will perform the actions described above.
  Only 'yes' will be accepted to approve.

  Enter a value: yes

module.test-vpc-module.module.vpc.google_compute_network.network: Creating...
module.test-vpc-module.module.vpc.google_compute_network.network: Still creating... [10s elapsed]
module.test-vpc-module.module.vpc.google_compute_network.network: Still creating... [20s elapsed]
module.test-vpc-module.module.vpc.google_compute_network.network: Creation complete after 23s [id=projects/qwiklabs-gcp-00-7e4d42cbcac7/global/networks/my-custom-mode-network]
module.test-vpc-module.module.subnets.google_compute_subnetwork.subnetwork["us-west1/subnet-02"]: Creating...
module.test-vpc-module.module.subnets.google_compute_subnetwork.subnetwork["us-west1/subnet-03"]: Creating...
module.test-vpc-module.module.subnets.google_compute_subnetwork.subnetwork["us-west1/subnet-01"]: Creating...
module.test-vpc-module.module.subnets.google_compute_subnetwork.subnetwork["us-west1/subnet-02"]: Still creating... [10s elapsed]
module.test-vpc-module.module.subnets.google_compute_subnetwork.subnetwork["us-west1/subnet-03"]: Still creating... [10s elapsed]
module.test-vpc-module.module.subnets.google_compute_subnetwork.subnetwork["us-west1/subnet-01"]: Still creating... [10s elapsed]
module.test-vpc-module.module.subnets.google_compute_subnetwork.subnetwork["us-west1/subnet-03"]: Creation complete after 16s [id=projects/qwiklabs-gcp-00-7e4d42cbcac7/regions/us-west1/subnetworks/subnet-03]
module.test-vpc-module.module.subnets.google_compute_subnetwork.subnetwork["us-west1/subnet-01"]: Creation complete after 16s [id=projects/qwiklabs-gcp-00-7e4d42cbcac7/regions/us-west1/subnetworks/subnet-01]
module.test-vpc-module.module.subnets.google_compute_subnetwork.subnetwork["us-west1/subnet-02"]: Creation complete after 19s [id=projects/qwiklabs-gcp-00-7e4d42cbcac7/regions/us-west1/subnetworks/subnet-02]

Apply complete! Resources: 4 added, 0 changed, 0 destroyed.

Outputs:

network_name = "my-custom-mode-network"
network_self_link = "https://www.googleapis.com/compute/v1/projects/qwiklabs-gcp-00-7e4d42cbcac7/global/networks/my-custom-mode-network"
project_id = "qwiklabs-gcp-00-7e4d42cbcac7"
route_names = []
subnets_flow_logs = [
  false,
  true,
  true,
]
subnets_ips = [
  "10.10.10.0/24",
  "10.10.20.0/24",
  "10.10.30.0/24",
]
subnets_names = [
  "subnet-01",
  "subnet-02",
  "subnet-03",
]
subnets_private_access = [
  false,
  true,
  false,
]
subnets_regions = [
  "us-west1",
  "us-west1",
  "us-west1",
]
subnets_secondary_ranges = [
  tolist([]),
  tolist([]),
  tolist([]),
]
student_01_b9934b2a0bae@cloudshell:~/terraform-google-network/examples/simple_project (qwiklabs-gcp-00-7e4d42cbcac7)$
```

### Understand how modules work

When using a new module for the first time, you must run either `terraform init` or `terraform get` to install the module. When either of these commands is run, Terraform will install any new modules in the `.terraform/modules` directory within your configuration's working directory. For local  modules, Terraform will create a symlink to the module's directory.  Because of this, any changes to local modules will be effective  immediately, without your having to re-run `terraform get`.





### Clean up your infrastructure

Now you have seen how to use modules from the Terraform Registry, how to configure those modules with input variables, and how to get output  values from those modules.

```sh
student_01_b9934b2a0bae@cloudshell:~/terraform-google-network/examples/simple_project (qwiklabs-gcp-00-7e4d42cbcac7)$ terraform destroy
var.project_id
  The project ID to host the network in

  Enter a value: qwiklabs-gcp-00-7e4d42cbcac7

module.test-vpc-module.module.vpc.google_compute_network.network: Refreshing state... [id=projects/qwiklabs-gcp-00-7e4d42cbcac7/global/networks/my-custom-mode-network]
module.test-vpc-module.module.subnets.google_compute_subnetwork.subnetwork["us-west1/subnet-02"]: Refreshing state... [id=projects/qwiklabs-gcp-00-7e4d42cbcac7/regions/us-west1/subnetworks/subnet-02]
module.test-vpc-module.module.subnets.google_compute_subnetwork.subnetwork["us-west1/subnet-03"]: Refreshing state... [id=projects/qwiklabs-gcp-00-7e4d42cbcac7/regions/us-west1/subnetworks/subnet-03]
module.test-vpc-module.module.subnets.google_compute_subnetwork.subnetwork["us-west1/subnet-01"]: Refreshing state... [id=projects/qwiklabs-gcp-00-7e4d42cbcac7/regions/us-west1/subnetworks/subnet-01]

Terraform used the selected providers to generate the following execution plan. Resource actions are indicated with the following symbols:
  - destroy

Terraform will perform the following actions:

  # module.test-vpc-module.module.subnets.google_compute_subnetwork.subnetwork["us-west1/subnet-01"] will be destroyed
  - resource "google_compute_subnetwork" "subnetwork" {
      - creation_timestamp         = "2023-04-24T19:09:32.753-07:00" -> null
      - gateway_address            = "10.10.10.1" -> null
      - id                         = "projects/qwiklabs-gcp-00-7e4d42cbcac7/regions/us-west1/subnetworks/subnet-01" -> null
      - ip_cidr_range              = "10.10.10.0/24" -> null
      - name                       = "subnet-01" -> null
      - network                    = "https://www.googleapis.com/compute/v1/projects/qwiklabs-gcp-00-7e4d42cbcac7/global/networks/my-custom-mode-network" -> null
      - private_ip_google_access   = false -> null
      - private_ipv6_google_access = "DISABLE_GOOGLE_ACCESS" -> null
      - project                    = "qwiklabs-gcp-00-7e4d42cbcac7" -> null
      - purpose                    = "PRIVATE" -> null
      - region                     = "us-west1" -> null
      - secondary_ip_range         = [] -> null
      - self_link                  = "https://www.googleapis.com/compute/v1/projects/qwiklabs-gcp-00-7e4d42cbcac7/regions/us-west1/subnetworks/subnet-01" -> null
      - stack_type                 = "IPV4_ONLY" -> null
    }

  # module.test-vpc-module.module.subnets.google_compute_subnetwork.subnetwork["us-west1/subnet-02"] will be destroyed
  - resource "google_compute_subnetwork" "subnetwork" {
      - creation_timestamp         = "2023-04-24T19:09:33.411-07:00" -> null
      - gateway_address            = "10.10.20.1" -> null
      - id                         = "projects/qwiklabs-gcp-00-7e4d42cbcac7/regions/us-west1/subnetworks/subnet-02" -> null
      - ip_cidr_range              = "10.10.20.0/24" -> null
      - name                       = "subnet-02" -> null
      - network                    = "https://www.googleapis.com/compute/v1/projects/qwiklabs-gcp-00-7e4d42cbcac7/global/networks/my-custom-mode-network" -> null
      - private_ip_google_access   = true -> null
      - private_ipv6_google_access = "DISABLE_GOOGLE_ACCESS" -> null
      - project                    = "qwiklabs-gcp-00-7e4d42cbcac7" -> null
      - purpose                    = "PRIVATE" -> null
      - region                     = "us-west1" -> null
      - secondary_ip_range         = [] -> null
      - self_link                  = "https://www.googleapis.com/compute/v1/projects/qwiklabs-gcp-00-7e4d42cbcac7/regions/us-west1/subnetworks/subnet-02" -> null
      - stack_type                 = "IPV4_ONLY" -> null

      - log_config {
          - aggregation_interval = "INTERVAL_5_SEC" -> null
          - filter_expr          = "true" -> null
          - flow_sampling        = 0.5 -> null
          - metadata             = "INCLUDE_ALL_METADATA" -> null
          - metadata_fields      = [] -> null
        }
    }

  # module.test-vpc-module.module.subnets.google_compute_subnetwork.subnetwork["us-west1/subnet-03"] will be destroyed
  - resource "google_compute_subnetwork" "subnetwork" {
      - creation_timestamp         = "2023-04-24T19:09:31.876-07:00" -> null
      - gateway_address            = "10.10.30.1" -> null
      - id                         = "projects/qwiklabs-gcp-00-7e4d42cbcac7/regions/us-west1/subnetworks/subnet-03" -> null
      - ip_cidr_range              = "10.10.30.0/24" -> null
      - name                       = "subnet-03" -> null
      - network                    = "https://www.googleapis.com/compute/v1/projects/qwiklabs-gcp-00-7e4d42cbcac7/global/networks/my-custom-mode-network" -> null
      - private_ip_google_access   = false -> null
      - private_ipv6_google_access = "DISABLE_GOOGLE_ACCESS" -> null
      - project                    = "qwiklabs-gcp-00-7e4d42cbcac7" -> null
      - purpose                    = "PRIVATE" -> null
      - region                     = "us-west1" -> null
      - secondary_ip_range         = [] -> null
      - self_link                  = "https://www.googleapis.com/compute/v1/projects/qwiklabs-gcp-00-7e4d42cbcac7/regions/us-west1/subnetworks/subnet-03" -> null
      - stack_type                 = "IPV4_ONLY" -> null

      - log_config {
          - aggregation_interval = "INTERVAL_10_MIN" -> null
          - filter_expr          = "false" -> null
          - flow_sampling        = 0.7 -> null
          - metadata             = "INCLUDE_ALL_METADATA" -> null
          - metadata_fields      = [] -> null
        }
    }

  # module.test-vpc-module.module.vpc.google_compute_network.network will be destroyed
  - resource "google_compute_network" "network" {
      - auto_create_subnetworks                   = false -> null
      - delete_default_routes_on_create           = false -> null
      - enable_ula_internal_ipv6                  = false -> null
      - id                                        = "projects/qwiklabs-gcp-00-7e4d42cbcac7/global/networks/my-custom-mode-network" -> null
      - mtu                                       = 1460 -> null
      - name                                      = "my-custom-mode-network" -> null
      - network_firewall_policy_enforcement_order = "AFTER_CLASSIC_FIREWALL" -> null
      - project                                   = "qwiklabs-gcp-00-7e4d42cbcac7" -> null
      - routing_mode                              = "GLOBAL" -> null
      - self_link                                 = "https://www.googleapis.com/compute/v1/projects/qwiklabs-gcp-00-7e4d42cbcac7/global/networks/my-custom-mode-network" -> null
    }

Plan: 0 to add, 0 to change, 4 to destroy.

Changes to Outputs:
  - network_name             = "my-custom-mode-network" -> null
  - network_self_link        = "https://www.googleapis.com/compute/v1/projects/qwiklabs-gcp-00-7e4d42cbcac7/global/networks/my-custom-mode-network" -> null
  - project_id               = "qwiklabs-gcp-00-7e4d42cbcac7" -> null
  - route_names              = [] -> null
  - subnets_flow_logs        = [
      - false,
      - true,
      - true,
    ] -> null
  - subnets_ips              = [
      - "10.10.10.0/24",
      - "10.10.20.0/24",
      - "10.10.30.0/24",
    ] -> null
  - subnets_names            = [
      - "subnet-01",
      - "subnet-02",
      - "subnet-03",
    ] -> null
  - subnets_private_access   = [
      - false,
      - true,
      - false,
    ] -> null
  - subnets_regions          = [
      - "us-west1",
      - "us-west1",
      - "us-west1",
    ] -> null
  - subnets_secondary_ranges = [
      - [],
      - [],
      - [],
    ] -> null

Do you really want to destroy all resources?
  Terraform will destroy all your managed infrastructure, as shown above.
  There is no undo. Only 'yes' will be accepted to confirm.

  Enter a value: yes

module.test-vpc-module.module.subnets.google_compute_subnetwork.subnetwork["us-west1/subnet-02"]: Destroying... [id=projects/qwiklabs-gcp-00-7e4d42cbcac7/regions/us-west1/subnetworks/subnet-02]
module.test-vpc-module.module.subnets.google_compute_subnetwork.subnetwork["us-west1/subnet-01"]: Destroying... [id=projects/qwiklabs-gcp-00-7e4d42cbcac7/regions/us-west1/subnetworks/subnet-01]
module.test-vpc-module.module.subnets.google_compute_subnetwork.subnetwork["us-west1/subnet-03"]: Destroying... [id=projects/qwiklabs-gcp-00-7e4d42cbcac7/regions/us-west1/subnetworks/subnet-03]
module.test-vpc-module.module.subnets.google_compute_subnetwork.subnetwork["us-west1/subnet-01"]: Still destroying... [id=projects/qwiklabs-gcp-00-7e4d42cbcac7/regions/us-west1/subnetworks/subnet-01, 10s elapsed]
module.test-vpc-module.module.subnets.google_compute_subnetwork.subnetwork["us-west1/subnet-03"]: Still destroying... [id=projects/qwiklabs-gcp-00-7e4d42cbcac7/regions/us-west1/subnetworks/subnet-03, 10s elapsed]
module.test-vpc-module.module.subnets.google_compute_subnetwork.subnetwork["us-west1/subnet-02"]: Still destroying... [id=projects/qwiklabs-gcp-00-7e4d42cbcac7/regions/us-west1/subnetworks/subnet-02, 10s elapsed]
module.test-vpc-module.module.subnets.google_compute_subnetwork.subnetwork["us-west1/subnet-03"]: Destruction complete after 12s
module.test-vpc-module.module.subnets.google_compute_subnetwork.subnetwork["us-west1/subnet-01"]: Destruction complete after 13s
module.test-vpc-module.module.subnets.google_compute_subnetwork.subnetwork["us-west1/subnet-02"]: Destruction complete after 14s
module.test-vpc-module.module.vpc.google_compute_network.network: Destroying... [id=projects/qwiklabs-gcp-00-7e4d42cbcac7/global/networks/my-custom-mode-network]
module.test-vpc-module.module.vpc.google_compute_network.network: Still destroying... [id=projects/qwiklabs-gcp-00-7e4d42cbcac7/global/networks/my-custom-mode-network, 10s elapsed]
module.test-vpc-module.module.vpc.google_compute_network.network: Still destroying... [id=projects/qwiklabs-gcp-00-7e4d42cbcac7/global/networks/my-custom-mode-network, 20s elapsed]
module.test-vpc-module.module.vpc.google_compute_network.network: Destruction complete after 22s

Destroy complete! Resources: 4 destroyed.
student_01_b9934b2a0bae@cloudshell:~/terraform-google-network/examples/simple_project (qwiklabs-gcp-00-7e4d42cbcac7)$
```

```sh
student_01_b9934b2a0bae@cloudshell:~/terraform-google-network/examples/simple_project (qwiklabs-gcp-00-7e4d42cbcac7)$ rm -rd terraform-google-network -f
student_01_b9934b2a0bae@cloudshell:~/terraform-google-network/examples/simple_project (qwiklabs-gcp-00-7e4d42cbcac7)$ ls
main.tf  outputs.tf  README.md  terraform.tfstate  terraform.tfstate.backup  variables.tf  versions.tf
student_01_b9934b2a0bae@cloudshell:~/terraform-google-network/examples/simple_project (qwiklabs-gcp-00-7e4d42cbcac7)$
```



## 2. Build a module

In the last task, you used a module from the Terraform Registry to  create a VPC network in Google Cloud. Although using existing Terraform  modules correctly is an important skill, every Terraform practitioner  will also benefit from learning how to create modules. We recommend that you create every Terraform configuration with the assumption that it  may be used as a module, because this will help you design your  configurations to be flexible, reusable, and composable.

As you may already know, Terraform treats every configuration as a module. When you run `terraform` commands, or use Terraform Cloud or Terraform Enterprise to remotely  run Terraform, the target directory containing Terraform configuration  is treated as the root module.

In this task, you create a module to manage Compute Storage buckets used to host static websites.

### Module structure

Terraform treats any local directory referenced in the `source` argument of a `module` block as a module. 

Each of these files serves a purpose:

- `LICENSE` contains the license under which your module will  be distributed. When you share your module, the LICENSE file will let  people using it know the terms under which it has been made available.  Terraform itself does not use this file.
- `README.md` contains documentation in markdown format that  describes how to use your module. Terraform does not use this file, but  services like the Terraform Registry and GitHub will display the  contents of this file to visitors to your module's Terraform Registry or GitHub page.
- `main.tf` contains the main set of configurations for your  module. You can also create other configuration files and organize them  in a way that makes sense for your project.
- `variables.tf` contains the variable definitions for your  module. When your module is used by others, the variables will be  configured as arguments in the module block. Because all Terraform  values must be defined, any variables that don't have a default value  will become required arguments. A variable with a default value can also be provided as a module argument, thus overriding the default value.
- `outputs.tf` contains the output definitions for your module. Module outputs are made available to the configuration using the  module, so they are often used to pass information about the parts of  your infrastructure defined by the module to other parts of your  configuration.

Be aware of these files and ensure that you don't distribute them as part of your module:

- `terraform.tfstate` and `terraform.tfstate.backup` files contain your Terraform state and are how Terraform keeps track of the relationship between your configuration and the infrastructure  provisioned by it.
- The `.terraform` directory contains the modules and  plugins used to provision your infrastructure. These files are specific  to an individual instance of Terraform when provisioning infrastructure, not the configuration of the infrastructure defined in `.tf` files.
- `*.tfvars`files don't need to be distributed with your module unless you are also using it as a standalone Terraform configuration  because module input variables are set via arguments to the module block in your configuration.

### Create a module

Navigate to your home directory and create your root module by constructing a new `main.tf` configuration file. Then create a directory called **modules** that contains another folder called `gcs-static-website-bucket`. You will work with three Terraform configuration files inside the `gcs-static-website-bucket` directory: `website.tf`, `variables.tf`, and `outputs.tf`.

```sh
student_01_b9934b2a0bae@cloudshell:~/terraform-google-network/examples/simple_project (qwiklabs-gcp-00-7e4d42cbcac7)$ cd ~
touch main.tf
mkdir -p modules/gcs-static-website-bucket
student_01_b9934b2a0bae@cloudshell:~ (qwiklabs-gcp-00-7e4d42cbcac7)$ cd modules/gcs-static-website-bucket
touch website.tf variables.tf outputs.tf
student_01_b9934b2a0bae@cloudshell:~/modules/gcs-static-website-bucket (qwiklabs-gcp-00-7e4d42cbcac7)$ ls
outputs.tf  variables.tf  website.tf
student_01_b9934b2a0bae@cloudshell:~/modules/gcs-static-website-bucket (qwiklabs-gcp-00-7e4d42cbcac7)$ cat website.tf
student_01_b9934b2a0bae@cloudshell:~/modules/gcs-static-website-bucket (qwiklabs-gcp-00-7e4d42cbcac7)$ cat outputs.tf
student_01_b9934b2a0bae@cloudshell:~/modules/gcs-static-website-bucket (qwiklabs-gcp-00-7e4d42cbcac7)$ cat variables.tf
student_01_b9934b2a0bae@cloudshell:~/modules/gcs-static-website-bucket (qwiklabs-gcp-00-7e4d42cbcac7)$
```



```sh
student_01_b9934b2a0bae@cloudshell:~/modules/gcs-static-website-bucket (qwiklabs-gcp-00-7e4d42cbcac7)$ tee -a README.md <<EOF
# GCS static website bucket
This module provisions Cloud Storage buckets configured for static website hosting.
EOF
# GCS static website bucket
This module provisions Cloud Storage buckets configured for static website hosting.
student_01_b9934b2a0bae@cloudshell:~/modules/gcs-static-website-bucket (qwiklabs-gcp-00-7e4d42cbcac7)$
```



```sh
student_01_b9934b2a0bae@cloudshell:~/modules/gcs-static-website-bucket (qwiklabs-gcp-00-7e4d42cbcac7)$ tee -a LICENSE <<EOF
Licensed under the Apache License, Version 2.0 (the "License");
you may not use this file except in compliance with the License.
You may obtain a copy of the License at
    http://www.apache.org/licenses/LICENSE-2.0
Unless required by applicable law or agreed to in writing, software
distributed under the License is distributed on an "AS IS" BASIS,
WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
See the License for the specific language governing permissions and
limitations under the License.
EOF
Licensed under the Apache License, Version 2.0 (the "License");
you may not use this file except in compliance with the License.
You may obtain a copy of the License at
    http://www.apache.org/licenses/LICENSE-2.0
Unless required by applicable law or agreed to in writing, software
distributed under the License is distributed on an "AS IS" BASIS,
WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
See the License for the specific language governing permissions and
limitations under the License.
student_01_b9934b2a0bae@cloudshell:~/modules/gcs-static-website-bucket (qwiklabs-gcp-00-7e4d42cbcac7)$
```

```sh
student_01_b9934b2a0bae@cloudshell:~ (qwiklabs-gcp-00-7e4d42cbcac7)$ ls
gopath  main.tf  modules  outputs.tf  README-cloudshell.txt  terraform-google-network  variables.tf
student_01_b9934b2a0bae@cloudshell:~ (qwiklabs-gcp-00-7e4d42cbcac7)$ cat main.tf
module "gcs-static-website-bucket" {
  source = "./modules/gcs-static-website-bucket"
  name       = var.name
  project_id = var.project_id
  location   = "us-east1"
  lifecycle_rules = [{
    action = {
      type = "Delete"
    }
    condition = {
      age        = 365
      with_state = "ANY"
    }
  }]
}
student_01_b9934b2a0bae@cloudshell:~ (qwiklabs-gcp-00-7e4d42cbcac7)$ cat variables.tf
variable "project_id" {
  description = "The ID of the project in which to provision resources."
  type        = string
  default     = "qwiklabs-gcp-00-7e4d42cbcac7"
}
variable "name" {
  description = "Name of the buckets to create."
  type        = string
  default     = "qwiklabs-gcp-00-7e4d42cbcac7"
}
student_01_b9934b2a0bae@cloudshell:~ (qwiklabs-gcp-00-7e4d42cbcac7)$ cat outputs.tf
output "bucket-name" {
  description = "Bucket names."
  value       = "module.gcs-static-website-bucket.bucket"
}
student_01_b9934b2a0bae@cloudshell:~ (qwiklabs-gcp-00-7e4d42cbcac7)$
```

```sh
student_01_b9934b2a0bae@cloudshell:~ (qwiklabs-gcp-00-7e4d42cbcac7)$ ls
gopath  main.tf  modules  outputs.tf  README-cloudshell.txt  terraform-google-network  variables.tf
student_01_b9934b2a0bae@cloudshell:~ (qwiklabs-gcp-00-7e4d42cbcac7)$ cd modules/
student_01_b9934b2a0bae@cloudshell:~/modules (qwiklabs-gcp-00-7e4d42cbcac7)$ ls
gcs-static-website-bucket
student_01_b9934b2a0bae@cloudshell:~/modules (qwiklabs-gcp-00-7e4d42cbcac7)$ cd gcs-static-website-bucket/
student_01_b9934b2a0bae@cloudshell:~/modules/gcs-static-website-bucket (qwiklabs-gcp-00-7e4d42cbcac7)$ ls
LICENSE  outputs.tf  README.md  variables.tf  website.tf
student_01_b9934b2a0bae@cloudshell:~/modules/gcs-static-website-bucket (qwiklabs-gcp-00-7e4d42cbcac7)$ cat website.tf
resource "google_storage_bucket" "bucket" {
  name               = var.name
  project            = var.project_id
  location           = var.location
  storage_class      = var.storage_class
  labels             = var.labels
  force_destroy      = var.force_destroy
  uniform_bucket_level_access = true
  versioning {
    enabled = var.versioning
  }
  dynamic "retention_policy" {
    for_each = var.retention_policy == null ? [] : [var.retention_policy]
    content {
      is_locked        = var.retention_policy.is_locked
      retention_period = var.retention_policy.retention_period
    }
  }
  dynamic "encryption" {
    for_each = var.encryption == null ? [] : [var.encryption]
    content {
      default_kms_key_name = var.encryption.default_kms_key_name
    }
  }
  dynamic "lifecycle_rule" {
    for_each = var.lifecycle_rules
    content {
      action {
        type          = lifecycle_rule.value.action.type
        storage_class = lookup(lifecycle_rule.value.action, "storage_class", null)
      }
      condition {
        age                   = lookup(lifecycle_rule.value.condition, "age", null)
        created_before        = lookup(lifecycle_rule.value.condition, "created_before", null)
        with_state            = lookup(lifecycle_rule.value.condition, "with_state", null)
        matches_storage_class = lookup(lifecycle_rule.value.condition, "matches_storage_class", null)
        num_newer_versions    = lookup(lifecycle_rule.value.condition, "num_newer_versions", null)
      }
    }
  }
}student_01_b9934b2a0bae@cloudshell:~/modules/gcs-static-website-bucket (qwiklabs-gcp-00-7e4d42cbcac7)$ cat variables.tf
variable "name" {
  description = "The name of the bucket."
  type        = string
}
variable "project_id" {
  description = "The ID of the project to create the bucket in."
  type        = string
}
variable "location" {
  description = "The location of the bucket."
  type        = string
}
variable "storage_class" {
  description = "The Storage Class of the new bucket."
  type        = string
  default     = null
}
variable "labels" {
  description = "A set of key/value label pairs to assign to the bucket."
  type        = map(string)
  default     = null
}
variable "bucket_policy_only" {
  description = "Enables Bucket Policy Only access to a bucket."
  type        = bool
  default     = true
}
variable "versioning" {
  description = "While set to true, versioning is fully enabled for this bucket."
  type        = bool
  default     = true
}
variable "force_destroy" {
  description = "When deleting a bucket, this boolean option will delete all contained objects. If false, Terraform will fail to delete buckets which contain objects."
  type        = bool
  default     = true
}
variable "iam_members" {
  description = "The list of IAM members to grant permissions on the bucket."
  type = list(object({
    role   = string
    member = string
  }))
  default = []
}
variable "retention_policy" {
  description = "Configuration of the bucket's data retention policy for how long objects in the bucket should be retained."
  type = object({
    is_locked        = bool
    retention_period = number
  })
  default = null
}
variable "encryption" {
  description = "A Cloud KMS key that will be used to encrypt objects inserted into this bucket"
  type = object({
    default_kms_key_name = string
  })
  default = null
}
variable "lifecycle_rules" {
  description = "The bucket's Lifecycle Rules configuration."
  type = list(object({
    # Object with keys:
    # - type - The type of the action of this Lifecycle Rule. Supported values: Delete and SetStorageClass.
    # - storage_class - (Required if action type is SetStorageClass) The target Storage Class of objects affected by this Lifecycle Rule.
    action = any
    # Object with keys:
    # - age - (Optional) Minimum age of an object in days to satisfy this condition.
    # - created_before - (Optional) Creation date of an object in RFC 3339 (e.g. 2017-06-13) to satisfy this condition.
    # - with_state - (Optional) Match to live and/or archived objects. Supported values include: "LIVE", "ARCHIVED", "ANY".
    # - matches_storage_class - (Optional) Storage Class of objects to satisfy this condition. Supported values include: MULTI_REGIONAL, REGIONAL, NEARLINE, COLDLINE, STANDARD, DURABLE_REDUCED_AVAILABILITY.
    # - num_newer_versions - (Optional) Relevant only for versioned objects. The number of newer versions of an object to satisfy this condition.
    condition = any
  }))
  default = []
}student_01_b9934b2a0bae@cloudshell:~/modules/gcs-static-website-bucket (qwiklabs-gcp-00-7e4d42cbcac7)$ cat outputs.tf
output "bucket" {
  description = "The created storage bucket"
  value       = google_storage_bucket.bucket
}student_01_b9934b2a0bae@cloudshell:~/modules/gcs-static-website-bucket (qwiklabs-gcp-00-7e4d42cbcac7)$
```

### Install the local module

Whenever you add a new module to a configuration, Terraform must install the module before it can be used. Both the `terraform get` and `terraform init` commands will install and update modules. The `terraform init` command will also initialize backends and install plugins.

```sh
student_01_b9934b2a0bae@cloudshell:~/modules/gcs-static-website-bucket (qwiklabs-gcp-00-7e4d42cbcac7)$ terraform init

Initializing the backend...

Initializing provider plugins...
- Finding latest version of hashicorp/google...
- Installing hashicorp/google v4.63.0...
- Installed hashicorp/google v4.63.0 (signed by HashiCorp)

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
student_01_b9934b2a0bae@cloudshell:~/modules/gcs-static-website-bucket (qwiklabs-gcp-00-7e4d42cbcac7)$
```

```sh
student_01_b9934b2a0bae@cloudshell:~/modules/gcs-static-website-bucket (qwiklabs-gcp-00-7e4d42cbcac7)$ terraform apply
var.location
  The location of the bucket.

  Enter a value: us-east1

var.name
  The name of the bucket.

  Enter a value: qwiklabs-gcp-00-7e4d42cbcac7

var.project_id
  The ID of the project to create the bucket in.

  Enter a value: qwiklabs-gcp-00-7e4d42cbcac7


Terraform used the selected providers to generate the following execution plan. Resource actions are indicated with the following symbols:
  + create

Terraform will perform the following actions:

  # google_storage_bucket.bucket will be created
  + resource "google_storage_bucket" "bucket" {
      + force_destroy               = true
      + id                          = (known after apply)
      + location                    = "US-EAST1"
      + name                        = "qwiklabs-gcp-00-7e4d42cbcac7"
      + project                     = "qwiklabs-gcp-00-7e4d42cbcac7"
      + public_access_prevention    = (known after apply)
      + self_link                   = (known after apply)
      + storage_class               = "STANDARD"
      + uniform_bucket_level_access = true
      + url                         = (known after apply)

      + versioning {
          + enabled = true
        }
    }

Plan: 1 to add, 0 to change, 0 to destroy.

Changes to Outputs:
  + bucket = {
      + autoclass                   = []
      + cors                        = []
      + custom_placement_config     = []
      + default_event_based_hold    = null
      + encryption                  = []
      + force_destroy               = true
      + labels                      = null
      + lifecycle_rule              = []
      + location                    = "US-EAST1"
      + logging                     = []
      + name                        = "qwiklabs-gcp-00-7e4d42cbcac7"
      + project                     = "qwiklabs-gcp-00-7e4d42cbcac7"
      + requester_pays              = null
      + retention_policy            = []
      + storage_class               = "STANDARD"
      + timeouts                    = null
      + uniform_bucket_level_access = true
      + versioning                  = [
          + {
              + enabled = true
            },
        ]
    }

Do you want to perform these actions?
  Terraform will perform the actions described above.
  Only 'yes' will be accepted to approve.

  Enter a value: yes

google_storage_bucket.bucket: Creating...
google_storage_bucket.bucket: Creation complete after 4s [id=qwiklabs-gcp-00-7e4d42cbcac7]

Apply complete! Resources: 1 added, 0 changed, 0 destroyed.

Outputs:

bucket = {
  "autoclass" = tolist([])
  "cors" = tolist([])
  "custom_placement_config" = tolist([])
  "default_event_based_hold" = false
  "encryption" = tolist([])
  "force_destroy" = true
  "id" = "qwiklabs-gcp-00-7e4d42cbcac7"
  "labels" = tomap(null) /* of string */
  "lifecycle_rule" = tolist([])
  "location" = "US-EAST1"
  "logging" = tolist([])
  "name" = "qwiklabs-gcp-00-7e4d42cbcac7"
  "project" = "qwiklabs-gcp-00-7e4d42cbcac7"
  "public_access_prevention" = "inherited"
  "requester_pays" = false
  "retention_policy" = tolist([])
  "self_link" = "https://www.googleapis.com/storage/v1/b/qwiklabs-gcp-00-7e4d42cbcac7"
  "storage_class" = "STANDARD"
  "timeouts" = null /* object */
  "uniform_bucket_level_access" = true
  "url" = "gs://qwiklabs-gcp-00-7e4d42cbcac7"
  "versioning" = tolist([
    {
      "enabled" = true
    },
  ])
  "website" = tolist([])
}
student_01_b9934b2a0bae@cloudshell:~/modules/gcs-static-website-bucket (qwiklabs-gcp-00-7e4d42cbcac7)$
```

### Upload files to the bucket

You have now configured and used your own module to create a static  website. You may want to visit this static website. Right now there is  nothing inside your bucket, so there is nothing to see at the website.  In order to see any content, you will need to upload objects to your  bucket. You can upload the contents of the `www` directory  in the GitHub repository.

```sh
student_01_b9934b2a0bae@cloudshell:~/modules/gcs-static-website-bucket (qwiklabs-gcp-00-7e4d42cbcac7)$ cd ~
curl https://raw.githubusercontent.com/hashicorp/learn-terraform-modules/master/modules/aws-s3-static-website-bucket/www/index.html > index.html
curl https://raw.githubusercontent.com/hashicorp/learn-terraform-modules/blob/master/modules/aws-s3-static-website-bucket/www/error.html > error.html
  % Total    % Received % Xferd  Average Speed   Time    Time     Time  Current
                                 Dload  Upload   Total   Spent    Left  Speed
100   257  100   257    0     0    764      0 --:--:-- --:--:-- --:--:--   762
  % Total    % Received % Xferd  Average Speed   Time    Time     Time  Current
                                 Dload  Upload   Total   Spent    Left  Speed
100    14  100    14    0     0     39      0 --:--:-- --:--:-- --:--:--    39
student_01_b9934b2a0bae@cloudshell:~ (qwiklabs-gcp-00-7e4d42cbcac7)$
```

```sh
student_01_b9934b2a0bae@cloudshell:~ (qwiklabs-gcp-00-7e4d42cbcac7)$ ls
error.html  gopath  index.html  main.tf  modules  outputs.tf  README-cloudshell.txt  terraform-google-network  variables.tf
student_01_b9934b2a0bae@cloudshell:~ (qwiklabs-gcp-00-7e4d42cbcac7)$ gsutil cp *.html gs://qwiklabs-gcp-00-7e4d42cbcac7
Copying file://error.html [Content-Type=text/html]...
Copying file://index.html [Content-Type=text/html]...
- [2 files][  271.0 B/  271.0 B]
Operation completed over 2 objects/271.0 B.
student_01_b9934b2a0bae@cloudshell:~ (qwiklabs-gcp-00-7e4d42cbcac7)$
```

```sh
tudent_01_b9934b2a0bae@cloudshell:~ (qwiklabs-gcp-00-7e4d42cbcac7)$ terraform init

Initializing the backend...
Initializing modules...
- gcs-static-website-bucket in modules/gcs-static-website-bucket

Initializing provider plugins...
- Finding latest version of hashicorp/google...
- Installing hashicorp/google v4.63.0...
- Installed hashicorp/google v4.63.0 (signed by HashiCorp)

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
student_01_b9934b2a0bae@cloudshell:~ (qwiklabs-gcp-00-7e4d42cbcac7)$ terraform destroy

No changes. No objects need to be destroyed.

Either you have not created any objects yet or the existing objects were already deleted outside of Terraform.

Destroy complete! Resources: 0 destroyed.
student_01_b9934b2a0bae@cloudshell:~ (qwiklabs-gcp-00-7e4d42cbcac7)$
```

In this lab, you learned the foundations of Terraform modules and how to use a pre-existing module from the Registry. You then built your own  module to create a static website hosted on a Cloud Storage bucket. In  doing so, you defined inputs, outputs, and variables for your  configuration files and learned the best-practices for building modules.

![]({{ site.url }}{{ site.baseurl }}/assets/images/gcp-tf-m1.png)

![]({{ site.url }}{{ site.baseurl }}/assets/images/gcp-tf-m2.png)

![]({{ site.url }}{{ site.baseurl }}/assets/images/gcp-tf-m3.png)

![]({{ site.url }}{{ site.baseurl }}/assets/images/gcp-tf-m4.png)

![]({{ site.url }}{{ site.baseurl }}/assets/images/gcp-tf-m5.png)

![]({{ site.url }}{{ site.baseurl }}/assets/images/gcp-tf-m6.png)

![]({{ site.url }}{{ site.baseurl }}/assets/images/gcp-tf-m7.png)

![]({{ site.url }}{{ site.baseurl }}/assets/images/gcp-tf-m8.png)

