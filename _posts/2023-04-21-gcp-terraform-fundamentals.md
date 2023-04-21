---
layout: single
title:  "GCP Terraform Fundamentals"
date:   2023-04-21 11:59:04 +0530
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

# GCP Terraform Fundamentals

- perform the following tasks:
  - Get started with Terraform in Google Cloud.
  - Install Terraform from installation binaries.
  - Create a VM instance infrastructure using Terraform.



```sh
Welcome to Cloud Shell! Type "help" to get started.
Your Cloud Platform project in this session is set to qwiklabs-gcp-00-a57f12c4a8c9.
Use “gcloud config set project [PROJECT_ID]” to change to a different project.
student_01_b9934b2a0bae@cloudshell:~ (qwiklabs-gcp-00-a57f12c4a8c9)$ gcloud auth list
Credentialed Accounts

ACTIVE: *
ACCOUNT: student-01-b9934b2a0bae@qwiklabs.net

To set the active account, run:
    $ gcloud config set account `ACCOUNT`

student_01_b9934b2a0bae@cloudshell:~ (qwiklabs-gcp-00-a57f12c4a8c9)$ gcloud config list project
[core]
project = qwiklabs-gcp-00-a57f12c4a8c9

Your active configuration is: [cloudshell-1621]
student_01_b9934b2a0bae@cloudshell:~ (qwiklabs-gcp-00-a57f12c4a8c9)$

```

```sh
Welcome to Cloud Shell! Type "help" to get started.
Your Cloud Platform project in this session is set to qwiklabs-gcp-00-a57f12c4a8c9.
Use “gcloud config set project [PROJECT_ID]” to change to a different project.
student_01_b9934b2a0bae@cloudshell:~ (qwiklabs-gcp-00-a57f12c4a8c9)$ terraform
Usage: terraform [global options] <subcommand> [args]

The available commands for execution are listed below.
The primary workflow commands are given first, followed by
less common or more advanced commands.

Main commands:
  init          Prepare your working directory for other commands
  validate      Check whether the configuration is valid
  plan          Show changes required by the current configuration
  apply         Create or update infrastructure
  destroy       Destroy previously-created infrastructure

All other commands:
  console       Try Terraform expressions at an interactive command prompt
  fmt           Reformat your configuration in the standard style
  force-unlock  Release a stuck lock on the current workspace
  get           Install or upgrade remote Terraform modules
  graph         Generate a Graphviz graph of the steps in an operation
  import        Associate existing infrastructure with a Terraform resource
  login         Obtain and save credentials for a remote host
  logout        Remove locally-stored credentials for a remote host
  metadata      Metadata related commands
  output        Show output values from your root module
  providers     Show the providers required for this configuration
  refresh       Update the state to match remote systems
  show          Show the current state or a saved plan
  state         Advanced state management
  taint         Mark a resource instance as not fully functional
  test          Experimental support for module integration testing
  untaint       Remove the 'tainted' state from a resource instance
  version       Show the current Terraform version
  workspace     Workspace management

Global options (use these before the subcommand, if any):
  -chdir=DIR    Switch to a different working directory before executing the
                given subcommand.
  -help         Show this help output, or the help for a specified subcommand.
  -version      An alias for the "version" subcommand.
student_01_b9934b2a0bae@cloudshell:~ (qwiklabs-gcp-00-a57f12c4a8c9)$ touch instance.tf
student_01_b9934b2a0bae@cloudshell:~ (qwiklabs-gcp-00-a57f12c4a8c9)$ ls
instance.tf  README-cloudshell.txt
student_01_b9934b2a0bae@cloudshell:~ (qwiklabs-gcp-00-a57f12c4a8c9)$ cat instance.tf
resource "google_compute_instance" "terraform" {
  project      = "qwiklabs-gcp-00-a57f12c4a8c9"
  name         = "terraform"
  machine_type = "n1-standard-1"
  zone         = "us-west1-c"
  boot_disk {
    initialize_params {
      image = "debian-cloud/debian-11"
    }
  }
  network_interface {
    network = "default"
    access_config {
    }
  }
}student_01_b9934b2a0bae@cloudshell:~ (qwiklabs-gcp-00-a57f12c4a8c9)$ terraform init

Initializing the backend...

Initializing provider plugins...
- Finding latest version of hashicorp/google...
- Installing hashicorp/google v4.62.1...
- Installed hashicorp/google v4.62.1 (signed by HashiCorp)

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
student_01_b9934b2a0bae@cloudshell:~ (qwiklabs-gcp-00-a57f12c4a8c9)$
```



```sh
student_01_b9934b2a0bae@cloudshell:~ (qwiklabs-gcp-00-a57f12c4a8c9)$ terraform plan

Terraform used the selected providers to generate the following execution plan. Resource actions are indicated with the following symbols:
  + create

Terraform will perform the following actions:

  # google_compute_instance.terraform will be created
  + resource "google_compute_instance" "terraform" {
      + can_ip_forward       = false
      + cpu_platform         = (known after apply)
      + current_status       = (known after apply)
      + deletion_protection  = false
      + guest_accelerator    = (known after apply)
      + id                   = (known after apply)
      + instance_id          = (known after apply)
      + label_fingerprint    = (known after apply)
      + machine_type         = "n1-standard-1"
      + metadata_fingerprint = (known after apply)
      + min_cpu_platform     = (known after apply)
      + name                 = "terraform"
      + project              = "qwiklabs-gcp-00-a57f12c4a8c9"
      + self_link            = (known after apply)
      + tags_fingerprint     = (known after apply)
      + zone                 = "us-west1-c"

      + boot_disk {
          + auto_delete                = true
          + device_name                = (known after apply)
          + disk_encryption_key_sha256 = (known after apply)
          + kms_key_self_link          = (known after apply)
          + mode                       = "READ_WRITE"
          + source                     = (known after apply)

          + initialize_params {
              + image  = "debian-cloud/debian-11"
              + labels = (known after apply)
              + size   = (known after apply)
              + type   = (known after apply)
            }
        }

      + network_interface {
          + ipv6_access_type   = (known after apply)
          + name               = (known after apply)
          + network            = "default"
          + network_ip         = (known after apply)
          + stack_type         = (known after apply)
          + subnetwork         = (known after apply)
          + subnetwork_project = (known after apply)

          + access_config {
              + nat_ip       = (known after apply)
              + network_tier = (known after apply)
            }
        }
    }

Plan: 1 to add, 0 to change, 0 to destroy.

───────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────

Note: You didn't use the -out option to save this plan, so Terraform can't guarantee to take exactly these actions if you run "terraform apply" now.
student_01_b9934b2a0bae@cloudshell:~ (qwiklabs-gcp-00-a57f12c4a8c9)$
```



```sh
student_01_b9934b2a0bae@cloudshell:~ (qwiklabs-gcp-00-a57f12c4a8c9)$ terraform apply

Terraform used the selected providers to generate the following execution plan. Resource actions are indicated with the following symbols:
  + create

Terraform will perform the following actions:

  # google_compute_instance.terraform will be created
  + resource "google_compute_instance" "terraform" {
      + can_ip_forward       = false
      + cpu_platform         = (known after apply)
      + current_status       = (known after apply)
      + deletion_protection  = false
      + guest_accelerator    = (known after apply)
      + id                   = (known after apply)
      + instance_id          = (known after apply)
      + label_fingerprint    = (known after apply)
      + machine_type         = "n1-standard-1"
      + metadata_fingerprint = (known after apply)
      + min_cpu_platform     = (known after apply)
      + name                 = "terraform"
      + project              = "qwiklabs-gcp-00-a57f12c4a8c9"
      + self_link            = (known after apply)
      + tags_fingerprint     = (known after apply)
      + zone                 = "us-west1-c"

      + boot_disk {
          + auto_delete                = true
          + device_name                = (known after apply)
          + disk_encryption_key_sha256 = (known after apply)
          + kms_key_self_link          = (known after apply)
          + mode                       = "READ_WRITE"
          + source                     = (known after apply)

          + initialize_params {
              + image  = "debian-cloud/debian-11"
              + labels = (known after apply)
              + size   = (known after apply)
              + type   = (known after apply)
            }
        }

      + network_interface {
          + ipv6_access_type   = (known after apply)
          + name               = (known after apply)
          + network            = "default"
          + network_ip         = (known after apply)
          + stack_type         = (known after apply)
          + subnetwork         = (known after apply)
          + subnetwork_project = (known after apply)

          + access_config {
              + nat_ip       = (known after apply)
              + network_tier = (known after apply)
            }
        }
    }

Plan: 1 to add, 0 to change, 0 to destroy.

Do you want to perform these actions?
  Terraform will perform the actions described above.
  Only 'yes' will be accepted to approve.

  Enter a value: yes

google_compute_instance.terraform: Creating...
google_compute_instance.terraform: Still creating... [10s elapsed]
google_compute_instance.terraform: Creation complete after 17s [id=projects/qwiklabs-gcp-00-a57f12c4a8c9/zones/us-west1-c/instances/terraform]

Apply complete! Resources: 1 added, 0 changed, 0 destroyed.
student_01_b9934b2a0bae@cloudshell:~ (qwiklabs-gcp-00-a57f12c4a8c9)$
```



```sh
student_01_b9934b2a0bae@cloudshell:~ (qwiklabs-gcp-00-a57f12c4a8c9)$ terraform show
# google_compute_instance.terraform:
resource "google_compute_instance" "terraform" {
    can_ip_forward       = false
    cpu_platform         = "Intel Broadwell"
    current_status       = "RUNNING"
    deletion_protection  = false
    enable_display       = false
    guest_accelerator    = []
    id                   = "projects/qwiklabs-gcp-00-a57f12c4a8c9/zones/us-west1-c/instances/terraform"
    instance_id          = "675711896257504640"
    label_fingerprint    = "42WmSpB8rSM="
    machine_type         = "n1-standard-1"
    metadata_fingerprint = "Kms8MV9-UMo="
    name                 = "terraform"
    project              = "qwiklabs-gcp-00-a57f12c4a8c9"
    self_link            = "https://www.googleapis.com/compute/v1/projects/qwiklabs-gcp-00-a57f12c4a8c9/zones/us-west1-c/instances/terraform"
    tags_fingerprint     = "42WmSpB8rSM="
    zone                 = "us-west1-c"

    boot_disk {
        auto_delete = true
        device_name = "persistent-disk-0"
        mode        = "READ_WRITE"
        source      = "https://www.googleapis.com/compute/v1/projects/qwiklabs-gcp-00-a57f12c4a8c9/zones/us-west1-c/disks/terraform"

        initialize_params {
            image  = "https://www.googleapis.com/compute/v1/projects/debian-cloud/global/images/debian-11-bullseye-v20230411"
            labels = {}
            size   = 10
            type   = "pd-standard"
        }
    }

    network_interface {
        name               = "nic0"
        network            = "https://www.googleapis.com/compute/v1/projects/qwiklabs-gcp-00-a57f12c4a8c9/global/networks/default"
        network_ip         = "10.138.0.2"
        queue_count        = 0
        stack_type         = "IPV4_ONLY"
        subnetwork         = "https://www.googleapis.com/compute/v1/projects/qwiklabs-gcp-00-a57f12c4a8c9/regions/us-west1/subnetworks/default"
        subnetwork_project = "qwiklabs-gcp-00-a57f12c4a8c9"

        access_config {
            nat_ip       = "34.145.65.124"
            network_tier = "PREMIUM"
        }
    }

    scheduling {
        automatic_restart   = true
        min_node_cpus       = 0
        on_host_maintenance = "MIGRATE"
        preemptible         = false
        provisioning_model  = "STANDARD"
    }

    shielded_instance_config {
        enable_integrity_monitoring = true
        enable_secure_boot          = false
        enable_vtpm                 = true
    }
}
student_01_b9934b2a0bae@cloudshell:~ (qwiklabs-gcp-00-a57f12c4a8c9)$
```



```sh
student_01_b9934b2a0bae@cloudshell:~ (qwiklabs-gcp-00-a57f12c4a8c9)$ cat terraform.tfstate
{
  "version": 4,
  "terraform_version": "1.4.5",
  "serial": 1,
  "lineage": "33b551a0-6e7c-3df0-4156-70a5ddd881b0",
  "outputs": {},
  "resources": [
    {
      "mode": "managed",
      "type": "google_compute_instance",
      "name": "terraform",
      "provider": "provider[\"registry.terraform.io/hashicorp/google\"]",
      "instances": [
        {
          "schema_version": 6,
          "attributes": {
            "advanced_machine_features": [],
            "allow_stopping_for_update": null,
            "attached_disk": [],
            "boot_disk": [
              {
                "auto_delete": true,
                "device_name": "persistent-disk-0",
                "disk_encryption_key_raw": "",
                "disk_encryption_key_sha256": "",
                "initialize_params": [
                  {
                    "image": "https://www.googleapis.com/compute/v1/projects/debian-cloud/global/images/debian-11-bullseye-v20230411",
                    "labels": {},
                    "size": 10,
                    "type": "pd-standard"
                  }
                ],
                "kms_key_self_link": "",
                "mode": "READ_WRITE",
                "source": "https://www.googleapis.com/compute/v1/projects/qwiklabs-gcp-00-a57f12c4a8c9/zones/us-west1-c/disks/terraform"
              }
            ],
            "can_ip_forward": false,
            "confidential_instance_config": [],
            "cpu_platform": "Intel Broadwell",
            "current_status": "RUNNING",
            "deletion_protection": false,
            "description": "",
            "desired_status": null,
            "enable_display": false,
            "guest_accelerator": [],
            "hostname": "",
            "id": "projects/qwiklabs-gcp-00-a57f12c4a8c9/zones/us-west1-c/instances/terraform",
            "instance_id": "675711896257504640",
            "label_fingerprint": "42WmSpB8rSM=",
            "labels": null,
            "machine_type": "n1-standard-1",
            "metadata": null,
            "metadata_fingerprint": "Kms8MV9-UMo=",
            "metadata_startup_script": null,
            "min_cpu_platform": "",
            "name": "terraform",
            "network_interface": [
              {
                "access_config": [
                  {
                    "nat_ip": "34.145.65.124",
                    "network_tier": "PREMIUM",
                    "public_ptr_domain_name": ""
                  }
                ],
                "alias_ip_range": [],
                "ipv6_access_config": [],
                "ipv6_access_type": "",
                "name": "nic0",
                "network": "https://www.googleapis.com/compute/v1/projects/qwiklabs-gcp-00-a57f12c4a8c9/global/networks/default",
                "network_ip": "10.138.0.2",
                "nic_type": "",
                "queue_count": 0,
                "stack_type": "IPV4_ONLY",
                "subnetwork": "https://www.googleapis.com/compute/v1/projects/qwiklabs-gcp-00-a57f12c4a8c9/regions/us-west1/subnetworks/default",
                "subnetwork_project": "qwiklabs-gcp-00-a57f12c4a8c9"
              }
            ],
            "project": "qwiklabs-gcp-00-a57f12c4a8c9",
            "reservation_affinity": [],
            "resource_policies": null,
            "scheduling": [
              {
                "automatic_restart": true,
                "instance_termination_action": "",
                "min_node_cpus": 0,
                "node_affinities": [],
                "on_host_maintenance": "MIGRATE",
                "preemptible": false,
                "provisioning_model": "STANDARD"
              }
            ],
            "scratch_disk": [],
            "self_link": "https://www.googleapis.com/compute/v1/projects/qwiklabs-gcp-00-a57f12c4a8c9/zones/us-west1-c/instances/terraform",
            "service_account": [],
            "shielded_instance_config": [
              {
                "enable_integrity_monitoring": true,
                "enable_secure_boot": false,
                "enable_vtpm": true
              }
            ],
            "tags": null,
            "tags_fingerprint": "42WmSpB8rSM=",
            "timeouts": null,
            "zone": "us-west1-c"
          },
          "sensitive_attributes": [],
          "private": "eyJlMmJmYjczMC1lY2FhLTExZTYtOGY4OC0zNDM2M2JjN2M0YzAiOnsiY3JlYXRlIjoxMjAwMDAwMDAwMDAwLCJkZWxldGUiOjEyMDAwMDAwMDAwMDAsInVwZGF0ZSI6MTIwMDAwMDAwMDAwMH0sInNjaGVtYV92ZXJzaW9uIjoiNiJ9"
        }
      ]
    }
  ],
  "check_results": null
}
student_01_b9934b2a0bae@cloudshell:~ (qwiklabs-gcp-00-a57f12c4a8c9)$
```



```sh
student_01_b9934b2a0bae@cloudshell:~ (qwiklabs-gcp-00-a57f12c4a8c9)$ cat .terraform.lock.hcl
# This file is maintained automatically by "terraform init".
# Manual edits may be lost in future updates.

provider "registry.terraform.io/hashicorp/google" {
  version = "4.62.1"
  hashes = [
    "h1:1zH3V4b71z8pbYvKhOJhILst3qRhPlV8Wc9E44JU2Q0=",
    "zh:15cb2755054d99ec0d7919f52f1a8a08c018d3f076a46251c5b0382f94337cdf",
    "zh:2286d2d182dd3df835665e8bb591ab72ed75af83a822ed91e44ed02d02e399d1",
    "zh:2507695cd914fe08cccb2c6fd7ff0a1f566647fd733b89ba83396d8bbac1d4b7",
    "zh:256a120ba34df742d328af2d8d2152a3f709eb8de571acec412b814d12b83c80",
    "zh:45f19aced5d9f597d8f7d63b7ca980aa3d0f58c342bc56aec8cbe0321955cc06",
    "zh:6eef7ba36ad5cf011f1d8b6f16dc84ea93f3f0b19df209f3bd1c694414529e04",
    "zh:739a0ab7647153c9b555d3e68b56f746bef17d822969124108775473ca375bfe",
    "zh:c5eb04297d298d75592d7671d4c707fdb3a7367aa1c2720c042b6f018268c0e1",
    "zh:d227ca244766b76913f70639c612d1e6b7a996484ad4177f41a5316fddf26594",
    "zh:dca333c358afe417d3f0610c2b89fc7e3741260a82f9626bcd1006e9016e3eff",
    "zh:ebfe6d76660341ea11c120dc2d1780fffed415cfb9b514421cb12963d6f8ac69",
    "zh:f569b65999264a9416862bca5cd2a6177d94ccb0424f3a4ef424428912b9cb3c",
  ]
}
student_01_b9934b2a0bae@cloudshell:~ (qwiklabs-gcp-00-a57f12c4a8c9)$
```



```sh
student_01_b9934b2a0bae@cloudshell:~ (qwiklabs-gcp-00-a57f12c4a8c9)$ cat .terraform.lock.hcl
# This file is maintained automatically by "terraform init".
# Manual edits may be lost in future updates.

provider "registry.terraform.io/hashicorp/google" {
  version = "4.62.1"
  hashes = [
    "h1:1zH3V4b71z8pbYvKhOJhILst3qRhPlV8Wc9E44JU2Q0=",
    "zh:15cb2755054d99ec0d7919f52f1a8a08c018d3f076a46251c5b0382f94337cdf",
    "zh:2286d2d182dd3df835665e8bb591ab72ed75af83a822ed91e44ed02d02e399d1",
    "zh:2507695cd914fe08cccb2c6fd7ff0a1f566647fd733b89ba83396d8bbac1d4b7",
    "zh:256a120ba34df742d328af2d8d2152a3f709eb8de571acec412b814d12b83c80",
    "zh:45f19aced5d9f597d8f7d63b7ca980aa3d0f58c342bc56aec8cbe0321955cc06",
    "zh:6eef7ba36ad5cf011f1d8b6f16dc84ea93f3f0b19df209f3bd1c694414529e04",
    "zh:739a0ab7647153c9b555d3e68b56f746bef17d822969124108775473ca375bfe",
    "zh:c5eb04297d298d75592d7671d4c707fdb3a7367aa1c2720c042b6f018268c0e1",
    "zh:d227ca244766b76913f70639c612d1e6b7a996484ad4177f41a5316fddf26594",
    "zh:dca333c358afe417d3f0610c2b89fc7e3741260a82f9626bcd1006e9016e3eff",
    "zh:ebfe6d76660341ea11c120dc2d1780fffed415cfb9b514421cb12963d6f8ac69",
    "zh:f569b65999264a9416862bca5cd2a6177d94ccb0424f3a4ef424428912b9cb3c",
  ]
}
student_01_b9934b2a0bae@cloudshell:~ (qwiklabs-gcp-00-a57f12c4a8c9)$
```



```sh
student_01_b9934b2a0bae@cloudshell:~ (qwiklabs-gcp-00-a57f12c4a8c9)$ ls .terraform
providers
student_01_b9934b2a0bae@cloudshell:~ (qwiklabs-gcp-00-a57f12c4a8c9)$ ls .terraform/providers/
registry.terraform.io
student_01_b9934b2a0bae@cloudshell:~ (qwiklabs-gcp-00-a57f12c4a8c9)$ ls .terraform/providers/registry.terraform.io/
hashicorp
student_01_b9934b2a0bae@cloudshell:~ (qwiklabs-gcp-00-a57f12c4a8c9)$ ls .terraform/providers/registry.terraform.io/hashicorp/
google
student_01_b9934b2a0bae@cloudshell:~ (qwiklabs-gcp-00-a57f12c4a8c9)$ ls .terraform/providers/registry.terraform.io/hashicorp/google/
4.62.1
student_01_b9934b2a0bae@cloudshell:~ (qwiklabs-gcp-00-a57f12c4a8c9)$ ls .terraform/providers/registry.terraform.io/hashicorp/google/4.62.1/
linux_amd64
student_01_b9934b2a0bae@cloudshell:~ (qwiklabs-gcp-00-a57f12c4a8c9)$ ls .terraform/providers/registry.terraform.io/hashicorp/google/4.62.1/linux_amd64/
terraform-provider-google_v4.62.1_x5
student_01_b9934b2a0bae@cloudshell:~ (qwiklabs-gcp-00-a57f12c4a8c9)$
```



```sh
student_01_b9934b2a0bae@cloudshell:~ (qwiklabs-gcp-00-a57f12c4a8c9)$ ls .terraform.d/
checkpoint_cache  checkpoint_signature
student_01_b9934b2a0bae@cloudshell:~ (qwiklabs-gcp-00-a57f12c4a8c9)$ ls .terraform.d/checkpoint_cache
.terraform.d/checkpoint_cache
student_01_b9934b2a0bae@cloudshell:~ (qwiklabs-gcp-00-a57f12c4a8c9)$ ls .terraform.d/checkpoint_signature
.terraform.d/checkpoint_signature
student_01_b9934b2a0bae@cloudshell:~ (qwiklabs-gcp-00-a57f12c4a8c9)$ cat .terraform.d/checkpoint_cache
5wi1.4.5{"product":"terraform","current_version":"1.4.5","current_release":1681326934,"current_download_url":"https://releases.hashicorp.com/terraform/1.4.5","current_changelog_url":"https://github.com/hashicorp/terraform/blob/v1.4.5/CHANGELOG.md","project_website":"https://www.terraform.io","alerts":[]}student_01_b9934b2a0bae@cloudshell:~ (qwiklabs-gcp-00-a57f12c4a8c9)$ cat .terraform.d/checkpoint_signature
e8371d02-9957-906d-92e2-7b4bb3f93284


This signature is a randomly generated UUID used to de-duplicate
alerts and version information. This signature is random, it is
not based on any personally identifiable information. To create
a new signature, you can simply delete this file at any time.
See the documentation for the software using Checkpoint for more
information on how to disable it.

student_01_b9934b2a0bae@cloudshell:~ (qwiklabs-gcp-00-a57f12c4a8c9)$
```



![]({{ site.url }}{{ site.baseurl }}/assets/images/gcp-tf-fund-1.png)







