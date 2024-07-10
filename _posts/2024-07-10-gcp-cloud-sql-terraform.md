---

layout: single
title:  "Cloud SQL with Terraform"
categories: Cloud
tags: GCP
classes: wide
show_date: true
header:
  overlay_image: /assets/images/gcp-banner-2.png
  og_image: /assets/images/gcp-banner-2.png
  teaser: /assets/images/pca-gcp.png
author:
  name     : "Professional Cloud Architect"
  avatar   : "/assets/images/pca-gcp.png"

sidebar:
  - title: "Topics"
    nav: my-sidebar

---
# Cloud SQL with Terraform

Create Cloud SQL instances with Terraform, then set up the Cloud SQL Proxy, testing the connection with a MySQL client.

- Create a Cloud SQL instance
- Install the Cloud SQL Proxy
- Test connectivity with MySQL client using Cloud Shell

### Cloud SQL

Cloud SQL is a fully managed database service that makes it easy to  set up, maintain, manage, and administer your relational databases on  Google Cloud. You can use Cloud SQL with either [MySQL](https://cloud.google.com/sql/docs/mysql/) or [PostgreSQL](https://cloud.google.com/sql/docs/postgres/).

## Download necessary files

1. Create a directory and fetch the required Terraform scripts from the Cloud Storage bucket with:

```sh
Welcome to Cloud Shell! Type "help" to get started.
Your Cloud Platform project in this session is set to qwiklabs-gcp-03-32e5941578c8.
Use “gcloud config set project [PROJECT_ID]” to change to a different project.
student_01_e488ef31cb8a@cloudshell:~ (qwiklabs-gcp-03-32e5941578c8)$ mkdir sql-with-terraform
cd sql-with-terraform
gsutil cp -r gs://spls/gsp234/gsp234.zip .
Copying gs://spls/gsp234/gsp234.zip...
/ [1 files][  3.8 KiB/  3.8 KiB]                                                
Operation completed over 1 objects/3.8 KiB.                                      
student_01_e488ef31cb8a@cloudshell:~/sql-with-terraform (qwiklabs-gcp-03-32e5941578c8)$ unzip gsp234.zip
Archive:  gsp234.zip
  inflating: outputs.tf              
  inflating: main.tf                 
  inflating: variables.tf            
student_01_e488ef31cb8a@cloudshell:~/sql-with-terraform (qwiklabs-gcp-03-32e5941578c8)$ ls
gsp234.zip  main.tf  outputs.tf  variables.tf
student_01_e488ef31cb8a@cloudshell:~/sql-with-terraform (qwiklabs-gcp-03-32e5941578c8)$
```



## Understand the code

```sh
student_01_e488ef31cb8a@cloudshell:~/sql-with-terraform (qwiklabs-gcp-03-32e5941578c8)$ cat main.tf 
/*
 * Copyright 2017 Google Inc.
 *
 * Licensed under the Apache License, Version 2.0 (the "License");
 * you may not use this file except in compliance with the License.
 * You may obtain a copy of the License at
 *
 *   http://www.apache.org/licenses/LICENSE-2.0
 *
 * Unless required by applicable law or agreed to in writing, software
 * distributed under the License is distributed on an "AS IS" BASIS,
 * WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
 * See the License for the specific language governing permissions and
 * limitations under the License.
 */

 provider "google" {
  version = "~> 2.13"
}

provider "google-beta" {
  version = "~> 2.13"
}

provider "random" {
  version = "~> 2.2"
}

resource "random_id" "name" {
  byte_length = 2
}

resource "google_sql_database_instance" "master" {
  name                 = "example-mysql-${random_id.name.hex}"
  project              = var.project
  region               = var.region
  database_version     = var.database_version
  master_instance_name = var.master_instance_name

  settings {
    tier                        = var.tier
    activation_policy           = var.activation_policy
    authorized_gae_applications = var.authorized_gae_applications
    disk_autoresize             = var.disk_autoresize
    dynamic "backup_configuration" {
      for_each = [var.backup_configuration]
      content {

        binary_log_enabled = lookup(backup_configuration.value, "binary_log_enabled", null)
        enabled            = lookup(backup_configuration.value, "enabled", null)
        start_time         = lookup(backup_configuration.value, "start_time", null)
      }
    }
    dynamic "ip_configuration" {
      for_each = [var.ip_configuration]
      content {

        ipv4_enabled    = lookup(ip_configuration.value, "ipv4_enabled", true)
        private_network = lookup(ip_configuration.value, "private_network", null)
        require_ssl     = lookup(ip_configuration.value, "require_ssl", null)

        dynamic "authorized_networks" {
          for_each = lookup(ip_configuration.value, "authorized_networks", [])
          content {
            expiration_time = lookup(authorized_networks.value, "expiration_time", null)
            name            = lookup(authorized_networks.value, "name", null)
            value           = lookup(authorized_networks.value, "value", null)
          }
        }
      }
    }
    dynamic "location_preference" {
      for_each = [var.location_preference]
      content {

        follow_gae_application = lookup(location_preference.value, "follow_gae_application", null)
        zone                   = lookup(location_preference.value, "zone", null)
      }
    }
    dynamic "maintenance_window" {
      for_each = [var.maintenance_window]
      content {

        day          = lookup(maintenance_window.value, "day", null)
        hour         = lookup(maintenance_window.value, "hour", null)
        update_track = lookup(maintenance_window.value, "update_track", null)
      }
    }
    disk_size        = var.disk_size
    disk_type        = var.disk_type
    pricing_plan     = var.pricing_plan
    replication_type = var.replication_type
    availability_type = var.availability_type
  }

  dynamic "replica_configuration" {
    for_each = [var.replica_configuration]
    content {

      ca_certificate            = lookup(replica_configuration.value, "ca_certificate", null)
      client_certificate        = lookup(replica_configuration.value, "client_certificate", null)
      client_key                = lookup(replica_configuration.value, "client_key", null)
      connect_retry_interval    = lookup(replica_configuration.value, "connect_retry_interval", null)
      dump_file_path            = lookup(replica_configuration.value, "dump_file_path", null)
      failover_target           = lookup(replica_configuration.value, "failover_target", null)
      master_heartbeat_period   = lookup(replica_configuration.value, "master_heartbeat_period", null)
      password                  = lookup(replica_configuration.value, "password", null)
      ssl_cipher                = lookup(replica_configuration.value, "ssl_cipher", null)
      username                  = lookup(replica_configuration.value, "username", null)
      verify_server_certificate = lookup(replica_configuration.value, "verify_server_certificate", null)
    }
  }

  timeouts {
    create = "60m"
    delete = "2h"
  }
}

resource "google_sql_database" "default" {
  count     = var.master_instance_name == "" ? 1 : 0
  name      = var.db_name
  project   = var.project
  instance  = google_sql_database_instance.master.name
  charset   = var.db_charset
  collation = var.db_collation
}

resource "random_id" "user-password" {
  byte_length = 8
}

resource "google_sql_user" "default" {
  count    = var.master_instance_name == "" ? 1 : 0
  name     = var.user_name
  project  = var.project
  instance = google_sql_database_instance.master.name
  host     = var.user_host
  password = var.user_password == "" ? random_id.user-password.hex : var.user_password
}


student_01_e488ef31cb8a@cloudshell:~/sql-with-terraform (qwiklabs-gcp-03-32e5941578c8)$ 
```

```sh
student_01_e488ef31cb8a@cloudshell:~/sql-with-terraform (qwiklabs-gcp-03-32e5941578c8)$ cat variables.tf 
/*
 * Copyright 2017 Google Inc.
 *
 * Licensed under the Apache License, Version 2.0 (the "License");
 * you may not use this file except in compliance with the License.
 * You may obtain a copy of the License at
 *
 *   http://www.apache.org/licenses/LICENSE-2.0
 *
 * Unless required by applicable law or agreed to in writing, software
 * distributed under the License is distributed on an "AS IS" BASIS,
 * WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
 * See the License for the specific language governing permissions and
 * limitations under the License.
 */

variable "project" {
  description = "The project to deploy to, if not set the default provider project is used."
  default     = "qwiklabs-gcp-03-32e5941578c8"
}

variable "region" {
  description = "Region for cloud resources"
  default     = "us-east4"
}

variable "database_version" {
  description = "The version of of the database. For example, `MYSQL_5_6` or `POSTGRES_9_6`."
  default     = "MYSQL_5_6"
}

variable "master_instance_name" {
  description = "The name of the master instance to replicate"
  default     = ""
}

variable "tier" {
  description = "The machine tier (First Generation) or type (Second Generation). See this page for supported tiers and pricing: https://cloud.google.com/sql/pricing"
  default     = "db-f1-micro"
}

variable "db_name" {
  description = "Name of the default database to create"
  default     = "default"
}

variable "db_charset" {
  description = "The charset for the default database"
  default     = ""
}

variable "db_collation" {
  description = "The collation for the default database. Example for MySQL databases: 'utf8_general_ci', and Postgres: 'en_US.UTF8'"
  default     = ""
}

variable "user_name" {
  description = "The name of the default user"
  default     = "default"
}

variable "user_host" {
  description = "The host for the default user"
  default     = "%"
}

variable "user_password" {
  description = "The password for the default user. If not set, a random one will be generated and available in the generated_user_password output variable."
  default     = ""
}

variable "activation_policy" {
  description = "This specifies when the instance should be active. Can be either `ALWAYS`, `NEVER` or `ON_DEMAND`."
  default     = "ALWAYS"
}

variable "authorized_gae_applications" {
  description = "A list of Google App Engine (GAE) project names that are allowed to access this instance."
  default     = []
}

variable "disk_autoresize" {
  description = "Second Generation only. Configuration to increase storage size automatically."
  default     = true
}

variable "disk_size" {
  description = "Second generation only. The size of data disk, in GB. Size of a running instance cannot be reduced but can be increased."
  default     = 10
}

variable "disk_type" {
  description = "Second generation only. The type of data disk: `PD_SSD` or `PD_HDD`."
  default     = "PD_SSD"
}

variable "pricing_plan" {
  description = "First generation only. Pricing plan for this instance, can be one of `PER_USE` or `PACKAGE`."
  default     = "PER_USE"
}

variable "replication_type" {
  description = "Replication type for this instance, can be one of `ASYNCHRONOUS` or `SYNCHRONOUS`."
  default     = "SYNCHRONOUS"
}

variable "database_flags" {
  description = "List of Cloud SQL flags that are applied to the database server"
  default     = []
}

variable "backup_configuration" {
  description = "The backup_configuration settings subblock for the database setings"
  default     = {}
}

variable "ip_configuration" {
  description = "The ip_configuration settings subblock"
  default     = {}
}

variable "location_preference" {
  description = "The location_preference settings subblock"
  default     = {}
}

variable "maintenance_window" {
  description = "The maintenance_window settings subblock"
  default     = {}
}

variable "replica_configuration" {
  description = "The optional replica_configuration block for the database instance"
  default     = {}
}

variable "availability_type" {
  description = "This specifies whether a PostgreSQL instance should be set up for high availability (REGIONAL) or single zone (ZONAL)."
  default     = "ZONAL"
}


student_01_e488ef31cb8a@cloudshell:~/sql-with-terraform (qwiklabs-gcp-03-32e5941578c8)$ 
```

```sh
student_01_e488ef31cb8a@cloudshell:~/sql-with-terraform (qwiklabs-gcp-03-32e5941578c8)$ cat outputs.tf 
/*
 * Copyright 2017 Google Inc.
 *
 * Licensed under the Apache License, Version 2.0 (the "License");
 * you may not use this file except in compliance with the License.
 * You may obtain a copy of the License at
 *
 *   http://www.apache.org/licenses/LICENSE-2.0
 *
 * Unless required by applicable law or agreed to in writing, software
 * distributed under the License is distributed on an "AS IS" BASIS,
 * WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
 * See the License for the specific language governing permissions and
 * limitations under the License.
 */

output "instance_name" {
  description = "The name of the database instance"
  value       = google_sql_database_instance.master.name
}

output "instance_address" {
  description = "The IPv4 address of the master database instnace"
  value       = google_sql_database_instance.master.ip_address.0.ip_address
}

output "instance_address_time_to_retire" {
  description = "The time the master instance IP address will be retired. RFC 3339 format."
  value       = google_sql_database_instance.master.ip_address.0.time_to_retire
}

output "self_link" {
  description = "Self link to the master instance"
  value       = google_sql_database_instance.master.self_link
}

output "generated_user_password" {
  description = "The auto generated default user password if no input password was provided"
  value       = random_id.user-password.hex
  sensitive   = true
}


student_01_e488ef31cb8a@cloudshell:~/sql-with-terraform (qwiklabs-gcp-03-32e5941578c8)$ 
```



## Run Terraform

The `terraform init` command is used to initialize a working directory containing Terraform configuration files.

This command performs several different initialization steps in order to prepare a working directory for use. This command is always safe to  run multiple times, to bring the working directory up to date with  changes in the configuration.

### Terraform Init

```sh
student_01_e488ef31cb8a@cloudshell:~/sql-with-terraform (qwiklabs-gcp-03-32e5941578c8)$ terraform init

Initializing the backend...

Initializing provider plugins...
- Finding hashicorp/google versions matching "~> 2.13"...
- Finding hashicorp/google-beta versions matching "~> 2.13"...
- Finding hashicorp/random versions matching "~> 2.2"...
- Installing hashicorp/google v2.20.3...
- Installed hashicorp/google v2.20.3 (signed by HashiCorp)
- Installing hashicorp/google-beta v2.20.3...
- Installed hashicorp/google-beta v2.20.3 (signed by HashiCorp)
- Installing hashicorp/random v2.3.1...
- Installed hashicorp/random v2.3.1 (signed by HashiCorp)

Terraform has created a lock file .terraform.lock.hcl to record the provider
selections it made above. Include this file in your version control repository
so that Terraform can guarantee to make the same selections by default when
you run "terraform init" in the future.

╷
│ Warning: Version constraints inside provider configuration blocks are deprecated
│ 
│   on main.tf line 18, in provider "google":
│   18:   version = "~> 2.13"
│ 
│ Terraform 0.13 and earlier allowed provider version constraints inside the provider configuration block, but that is now deprecated and will be removed in a future version of
│ Terraform. To silence this warning, move the provider version constraint into the required_providers block.
│ 
│ (and 5 more similar warnings elsewhere)
╵

Terraform has been successfully initialized!

You may now begin working with Terraform. Try running "terraform plan" to see
any changes that are required for your infrastructure. All Terraform commands
should now work.

If you ever set or change modules or backend configuration for Terraform,
rerun this command to reinitialize your working directory. If you forget, other
commands will detect it and remind you to do so if necessary.
student_01_e488ef31cb8a@cloudshell:~/sql-with-terraform (qwiklabs-gcp-03-32e5941578c8)$ 
```

### Terraform Plan

```sh
student_01_e488ef31cb8a@cloudshell:~/sql-with-terraform (qwiklabs-gcp-03-32e5941578c8)$ terraform plan -out=tfplan

Terraform used the selected providers to generate the following execution plan. Resource actions are indicated with the following symbols:
  + create

Terraform will perform the following actions:

  # google_sql_database.default[0] will be created
  + resource "google_sql_database" "default" {
      + charset   = (known after apply)
      + collation = (known after apply)
      + id        = (known after apply)
      + instance  = (known after apply)
      + name      = "default"
      + project   = "qwiklabs-gcp-03-32e5941578c8"
      + self_link = (known after apply)
    }

  # google_sql_database_instance.master will be created
  + resource "google_sql_database_instance" "master" {
      + connection_name               = (known after apply)
      + database_version              = "MYSQL_5_6"
      + first_ip_address              = (known after apply)
      + id                            = (known after apply)
      + ip_address                    = (known after apply)
      + master_instance_name          = (known after apply)
      + name                          = (known after apply)
      + private_ip_address            = (known after apply)
      + project                       = "qwiklabs-gcp-03-32e5941578c8"
      + public_ip_address             = (known after apply)
      + region                        = "us-east4"
      + self_link                     = (known after apply)
      + server_ca_cert                = (known after apply)
      + service_account_email_address = (known after apply)

      + replica_configuration {}

      + settings {
          + activation_policy           = "ALWAYS"
          + authorized_gae_applications = []
          + availability_type           = "ZONAL"
          + crash_safe_replication      = (known after apply)
          + disk_autoresize             = true
          + disk_size                   = 10
          + disk_type                   = "PD_SSD"
          + pricing_plan                = "PER_USE"
          + replication_type            = "SYNCHRONOUS"
          + tier                        = "db-f1-micro"
          + version                     = (known after apply)

          + backup_configuration {
              + start_time = (known after apply)
            }

          + ip_configuration {
              + ipv4_enabled = true
            }

          + location_preference {}

          + maintenance_window {}
        }

      + timeouts {
          + create = "60m"
          + delete = "2h"
        }
    }

  # google_sql_user.default[0] will be created
  + resource "google_sql_user" "default" {
      + host     = "%"
      + id       = (known after apply)
      + instance = (known after apply)
      + name     = "default"
      + password = (sensitive value)
      + project  = "qwiklabs-gcp-03-32e5941578c8"
    }

  # random_id.name will be created
  + resource "random_id" "name" {
      + b64         = (known after apply)
      + b64_std     = (known after apply)
      + b64_url     = (known after apply)
      + byte_length = 2
      + dec         = (known after apply)
      + hex         = (known after apply)
      + id          = (known after apply)
    }

  # random_id.user-password will be created
  + resource "random_id" "user-password" {
      + b64         = (known after apply)
      + b64_std     = (known after apply)
      + b64_url     = (known after apply)
      + byte_length = 8
      + dec         = (known after apply)
      + hex         = (known after apply)
      + id          = (known after apply)
    }

Plan: 5 to add, 0 to change, 0 to destroy.

Changes to Outputs:
  + generated_user_password         = (sensitive value)
  + instance_address                = (known after apply)
  + instance_address_time_to_retire = (known after apply)
  + instance_name                   = (known after apply)
  + self_link                       = (known after apply)
╷
│ Warning: Version constraints inside provider configuration blocks are deprecated
│ 
│   on main.tf line 18, in provider "google":
│   18:   version = "~> 2.13"
│ 
│ Terraform 0.13 and earlier allowed provider version constraints inside the provider configuration block, but that is now deprecated and will be removed in a future version of
│ Terraform. To silence this warning, move the provider version constraint into the required_providers block.
│ 
│ (and 2 more similar warnings elsewhere)
╵

───────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────

Saved the plan to: tfplan

To perform exactly these actions, run the following command to apply:
    terraform apply "tfplan"
student_01_e488ef31cb8a@cloudshell:~/sql-with-terraform (qwiklabs-gcp-03-32e5941578c8)$ 
```

### Terraform Apply

```sh
student_01_e488ef31cb8a@cloudshell:~/sql-with-terraform (qwiklabs-gcp-03-32e5941578c8)$ terraform apply tfplan
random_id.name: Creating...
random_id.user-password: Creating...
random_id.user-password: Creation complete after 0s [id=zsd-o9nxheo]
random_id.name: Creation complete after 0s [id=0vQ]
google_sql_database_instance.master: Creating...
google_sql_database_instance.master: Still creating... [10s elapsed]
google_sql_database_instance.master: Still creating... [20s elapsed]
google_sql_database_instance.master: Still creating... [30s elapsed]
google_sql_database_instance.master: Still creating... [40s elapsed]
google_sql_database_instance.master: Still creating... [50s elapsed]
google_sql_database_instance.master: Still creating... [1m0s elapsed]
google_sql_database_instance.master: Still creating... [1m10s elapsed]
google_sql_database_instance.master: Still creating... [1m20s elapsed]
google_sql_database_instance.master: Still creating... [1m30s elapsed]
google_sql_database_instance.master: Still creating... [1m40s elapsed]
google_sql_database_instance.master: Still creating... [1m50s elapsed]
google_sql_database_instance.master: Still creating... [2m0s elapsed]
google_sql_database_instance.master: Still creating... [2m10s elapsed]
google_sql_database_instance.master: Still creating... [2m20s elapsed]
google_sql_database_instance.master: Still creating... [2m30s elapsed]
google_sql_database_instance.master: Still creating... [2m40s elapsed]
google_sql_database_instance.master: Still creating... [2m50s elapsed]
google_sql_database_instance.master: Still creating... [3m0s elapsed]
google_sql_database_instance.master: Still creating... [3m10s elapsed]
google_sql_database_instance.master: Still creating... [3m20s elapsed]
google_sql_database_instance.master: Still creating... [3m30s elapsed]
google_sql_database_instance.master: Still creating... [3m40s elapsed]
google_sql_database_instance.master: Still creating... [3m50s elapsed]
google_sql_database_instance.master: Still creating... [4m0s elapsed]
google_sql_database_instance.master: Still creating... [4m10s elapsed]
google_sql_database_instance.master: Still creating... [4m20s elapsed]
google_sql_database_instance.master: Still creating... [4m30s elapsed]
google_sql_database_instance.master: Still creating... [4m40s elapsed]
google_sql_database_instance.master: Still creating... [4m50s elapsed]
google_sql_database_instance.master: Still creating... [5m0s elapsed]
google_sql_database_instance.master: Still creating... [5m10s elapsed]
google_sql_database_instance.master: Still creating... [5m20s elapsed]
google_sql_database_instance.master: Still creating... [5m30s elapsed]
google_sql_database_instance.master: Still creating... [5m40s elapsed]
google_sql_database_instance.master: Still creating... [5m50s elapsed]
google_sql_database_instance.master: Still creating... [6m0s elapsed]
google_sql_database_instance.master: Still creating... [6m10s elapsed]
google_sql_database_instance.master: Still creating... [6m20s elapsed]
google_sql_database_instance.master: Still creating... [6m30s elapsed]
google_sql_database_instance.master: Still creating... [6m40s elapsed]
google_sql_database_instance.master: Still creating... [6m50s elapsed]
google_sql_database_instance.master: Still creating... [7m0s elapsed]
google_sql_database_instance.master: Still creating... [7m10s elapsed]
google_sql_database_instance.master: Still creating... [7m20s elapsed]
google_sql_database_instance.master: Still creating... [7m30s elapsed]
google_sql_database_instance.master: Still creating... [7m40s elapsed]
google_sql_database_instance.master: Still creating... [7m50s elapsed]
google_sql_database_instance.master: Still creating... [8m0s elapsed]
google_sql_database_instance.master: Still creating... [8m10s elapsed]
google_sql_database_instance.master: Still creating... [8m20s elapsed]
google_sql_database_instance.master: Still creating... [8m30s elapsed]
google_sql_database_instance.master: Still creating... [8m40s elapsed]
google_sql_database_instance.master: Still creating... [8m50s elapsed]
google_sql_database_instance.master: Still creating... [9m0s elapsed]
google_sql_database_instance.master: Still creating... [9m10s elapsed]
google_sql_database_instance.master: Still creating... [9m20s elapsed]
google_sql_database_instance.master: Creation complete after 9m28s [id=example-mysql-d2f4]
google_sql_user.default[0]: Creating...
google_sql_database.default[0]: Creating...
google_sql_user.default[0]: Creation complete after 1s [id=default/%/example-mysql-d2f4]
google_sql_database.default[0]: Creation complete after 3s [id=example-mysql-d2f4:default]
╷
│ Warning: Version constraints inside provider configuration blocks are deprecated
│ 
│   on main.tf line 18, in provider "google":
│   18:   version = "~> 2.13"
│ 
│ Terraform 0.13 and earlier allowed provider version constraints inside the provider configuration block, but that is now deprecated and will be removed in a future version of
│ Terraform. To silence this warning, move the provider version constraint into the required_providers block.
│ 
│ (and 2 more similar warnings elsewhere)
╵

Apply complete! Resources: 5 added, 0 changed, 0 destroyed.

Outputs:

generated_user_password = <sensitive>
instance_address = "35.245.200.195"
instance_address_time_to_retire = ""
instance_name = "example-mysql-d2f4"
self_link = "https://www.googleapis.com/sql/v1beta4/projects/qwiklabs-gcp-03-32e5941578c8/instances/example-mysql-d2f4"
student_01_e488ef31cb8a@cloudshell:~/sql-with-terraform (qwiklabs-gcp-03-32e5941578c8)$ 
```



## Cloud SQL Proxy

### What the proxy provides

The Cloud SQL Proxy provides secure access to your Cloud SQL Second Generation instances without having to [allowlist IP addresses](https://cloud.google.com/sql/docs/mysql/configure-ip) or [configure SSL](https://cloud.google.com/sql/docs/mysql/configure-ssl-instance).

Accessing your Cloud SQL instance using the Cloud SQL Proxy offers these advantages:

- **Secure connections:** The proxy automatically encrypts  traffic to and from the database using TLS 1.2 with a 128-bit AES  cipher; SSL certificates are used to verify client and server  identities.
- **Easier connection management:** The proxy handles authentication with Cloud SQL, removing the need to provide static IP addresses.

You do not need to use the proxy or configure SSL to connect to Cloud SQL from App Engine standard or the flexible environment. These connections use a "built-in" proxy implementation automatically.

### How the Cloud SQL Proxy works

The Cloud SQL Proxy works by having a local client, called the proxy, running in the local environment. Your application communicates with  the proxy with the standard protocol used by your database. The proxy  uses a secure tunnel to communicate with its companion process running  on the server.



## Installing the Cloud SQL Proxy

1. Download the proxy:

```sh
student_01_e488ef31cb8a@cloudshell:~/sql-with-terraform (qwiklabs-gcp-03-32e5941578c8)$ wget https://dl.google.com/cloudsql/cloud_sql_proxy.linux.amd64 -O cloud_sql_proxy
--2024-07-10 16:19:57--  https://dl.google.com/cloudsql/cloud_sql_proxy.linux.amd64
Resolving dl.google.com (dl.google.com)... 172.217.194.190, 172.217.194.91, 172.217.194.136, ...
Connecting to dl.google.com (dl.google.com)|172.217.194.190|:443... connected.
HTTP request sent, awaiting response... 200 OK
Length: 19921714 (19M) [application/octet-stream]
Saving to: ‘cloud_sql_proxy’

cloud_sql_proxy                              100%[==============================================================================================>]  19.00M  --.-KB/s    in 0.1s    

2024-07-10 16:19:57 (169 MB/s) - ‘cloud_sql_proxy’ saved [19921714/19921714]

student_01_e488ef31cb8a@cloudshell:~/sql-with-terraform (qwiklabs-gcp-03-32e5941578c8)$ chmod +x cloud_sql_proxy
student_01_e488ef31cb8a@cloudshell:~/sql-with-terraform (qwiklabs-gcp-03-32e5941578c8)$ ls
cloud_sql_proxy  gsp234.zip  main.tf  outputs.tf  terraform.tfstate  tfplan  variables.tf
student_01_e488ef31cb8a@cloudshell:~/sql-with-terraform (qwiklabs-gcp-03-32e5941578c8)$ 
```

### Proxy startup options

When you start the proxy, you provide it with the following sets of information:

- What Cloud SQL instances it should establish connections to
- Where it will listen for data coming from your application to be sent to Cloud SQL
- Where it will find the credentials it will use to authenticate your application to Cloud SQL

The proxy startup options you provide determine whether it will  listen on a TCP port or on a Unix socket. If it is listening on a Unix  socket, it creates the socket at the location you choose; usually, the `/cloudsql/` directory. For TCP, the proxy listens on `localhost` by default.

## Test connection to the database

1. Start by running the Cloud SQL proxy for the Cloud SQL instance:

```sh
student_01_e488ef31cb8a@cloudshell:~/sql-with-terraform (qwiklabs-gcp-03-32e5941578c8)$ export GOOGLE_PROJECT=$(gcloud config get-value project)
Your active configuration is: [cloudshell-11903]
student_01_e488ef31cb8a@cloudshell:~/sql-with-terraform (qwiklabs-gcp-03-32e5941578c8)$ MYSQL_DB_NAME=$(terraform output -json | jq -r '.instance_name.value')
MYSQL_CONN_NAME="${GOOGLE_PROJECT}:us-east4:${MYSQL_DB_NAME}"
student_01_e488ef31cb8a@cloudshell:~/sql-with-terraform (qwiklabs-gcp-03-32e5941578c8)$ ./cloud_sql_proxy -instances=${MYSQL_CONN_NAME}=tcp:3306
2024/07/10 16:21:42 current FDs rlimit set to 1048576, wanted limit is 8500. Nothing to do here.
2024/07/10 16:21:46 Listening on 127.0.0.1:3306 for qwiklabs-gcp-03-32e5941578c8:us-east4:example-mysql-d2f4
2024/07/10 16:21:46 Ready for new connections
2024/07/10 16:21:46 Generated RSA key in 181.071908ms

```

Now you'll start another Cloud Shell tab by clicking on plus (**+**) icon. You'll use this shell to connect to the Cloud SQL proxy.

1. Navigate to `sql-with-terraform` directory:
2. Get the generated password for MYSQL
3. Test the MySQL connection
4. When prompted, enter the value of `MYSQL_PASSWORD`, found in the output above, and press **Enter**.
5. You should successfully log into the MYSQL command line. Exit from MYSQL by typing **Ctrl** + **D**.



```sh
student_01_e488ef31cb8a@cloudshell:~ (qwiklabs-gcp-03-32e5941578c8)$ cd ~/sql-with-terraform
student_01_e488ef31cb8a@cloudshell:~/sql-with-terraform (qwiklabs-gcp-03-32e5941578c8)$ echo MYSQL_PASSWORD=$(terraform output -json | jq -r '.generated_user_password.value')
MYSQL_PASSWORD=cec77ea3d9f185ea
student_01_e488ef31cb8a@cloudshell:~/sql-with-terraform (qwiklabs-gcp-03-32e5941578c8)$ mysql -udefault -p --host 127.0.0.1 default
Enter password: 
Welcome to the MySQL monitor.  Commands end with ; or \g.
Your MySQL connection id is 40
Server version: 5.6.51-google (Google)

Copyright (c) 2000, 2024, Oracle and/or its affiliates.

Oracle is a registered trademark of Oracle Corporation and/or its
affiliates. Other names may be trademarks of their respective
owners.

Type 'help;' or '\h' for help. Type '\c' to clear the current input statement.

mysql> ^DBye
student_01_e488ef31cb8a@cloudshell:~/sql-with-terraform (qwiklabs-gcp-03-32e5941578c8)$ 
```
If you go back to the first Cloud Shell tab you'll see logs for the connections made to the Cloud SQL Proxy.

```sh
student_01_e488ef31cb8a@cloudshell:~/sql-with-terraform (qwiklabs-gcp-03-32e5941578c8)$ ./cloud_sql_proxy -instances=${MYSQL_CONN_NAME}=tcp:3306
2024/07/10 16:21:42 current FDs rlimit set to 1048576, wanted limit is 8500. Nothing to do here.
2024/07/10 16:21:46 Listening on 127.0.0.1:3306 for qwiklabs-gcp-03-32e5941578c8:us-east4:example-mysql-d2f4
2024/07/10 16:21:46 Ready for new connections
2024/07/10 16:21:46 Generated RSA key in 181.071908ms
2024/07/10 16:22:57 New connection for "qwiklabs-gcp-03-32e5941578c8:us-east4:example-mysql-d2f4"
2024/07/10 16:22:57 refreshing ephemeral certificate for instance qwiklabs-gcp-03-32e5941578c8:us-east4:example-mysql-d2f4
2024/07/10 16:23:01 Scheduling refresh of ephemeral certificate in 54m58s
2024/07/10 16:23:15 Client closed local connection on 127.0.0.1:3306
```



In this lab, you used Terraform to create a Cloud SQL instance and set  up the Cloud SQL Proxy. You then tested the connection between the two  with a MySQL client.

## History

```sh
student_01_e488ef31cb8a@cloudshell:~/sql-with-terraform (qwiklabs-gcp-03-32e5941578c8)$ history 
    1  mkdir sql-with-terraform
    2  cd sql-with-terraform
    3  gsutil cp -r gs://spls/gsp234/gsp234.zip .
    4  unzip gsp234.zip
    5  ls
    6  cat main.tf 
    7  cat variables.tf 
    8  cat outputs.tf 
    9  vi variables.tf 
   10  cat variables.tf 
   11  terraform init
   12  terraform plan -out=tfplan
   13  terraform apply tfplan
   14  wget https://dl.google.com/cloudsql/cloud_sql_proxy.linux.amd64 -O cloud_sql_proxy
   15  chmod +x cloud_sql_proxy
   16  ls
   17  export GOOGLE_PROJECT=$(gcloud config get-value project)
   18  MYSQL_DB_NAME=$(terraform output -json | jq -r '.instance_name.value')
   19  MYSQL_CONN_NAME="${GOOGLE_PROJECT}:us-east4:${MYSQL_DB_NAME}"
   20  cd ~/sql-with-terraform
   21  echo MYSQL_PASSWORD=$(terraform output -json | jq -r '.generated_user_password.value')
   22  mysql -udefault -p --host 127.0.0.1 default
   23  history 
student_01_e488ef31cb8a@cloudshell:~/sql-with-terraform (qwiklabs-gcp-03-32e5941578c8)$ 
```

```sh
student_01_e488ef31cb8a@cloudshell:~/sql-with-terraform (qwiklabs-gcp-03-32e5941578c8)$ history 
    1  mkdir sql-with-terraform
    2  cd sql-with-terraform
    3  gsutil cp -r gs://spls/gsp234/gsp234.zip .
    4  unzip gsp234.zip
    5  ls
    6  cat main.tf 
    7  cat variables.tf 
    8  cat outputs.tf 
    9  vi variables.tf 
   10  cat variables.tf 
   11  terraform init
   12  terraform plan -out=tfplan
   13  terraform apply tfplan
   14  wget https://dl.google.com/cloudsql/cloud_sql_proxy.linux.amd64 -O cloud_sql_proxy
   15  chmod +x cloud_sql_proxy
   16  ls
   17  export GOOGLE_PROJECT=$(gcloud config get-value project)
   18  MYSQL_DB_NAME=$(terraform output -json | jq -r '.instance_name.value')
   19  MYSQL_CONN_NAME="${GOOGLE_PROJECT}:us-east4:${MYSQL_DB_NAME}"
   20  ./cloud_sql_proxy -instances=${MYSQL_CONN_NAME}=tcp:3306
   21  history 
student_01_e488ef31cb8a@cloudshell:~/sql-with-terraform (qwiklabs-gcp-03-32e5941578c8)$ 
```

