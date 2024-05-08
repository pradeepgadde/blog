---

layout: single
title:  "GCP—Automating the Deployment of Networks Using Terraform"
date:   2022-10-08 04:59:04 +0530
categories: Cloud
tags: GCP
show_date: true
classes: wide
header:
  teaser: /assets/images/gcpne.png
author:
  name     : "Google Cloud Platform"
  avatar   : "/assets/images/gcpne.png"

sidebar:
  - title: "Topics"
    nav: my-sidebar

---

# Automating the Deployment of Networks Using Terraform



Terraform enables you to safely and predictably create, change, and improve infrastructure. It is an open-source tool that codifies APIs into declarative configuration files that can be shared among team members, treated as code, edited, reviewed, and versioned.

In this lab, you create a Terraform configuration with a module to automate the deployment of a custom network with resources. Specifically, you deploy 3 networks with firewall rules and VM instances, as shown in this network diagram:

![]({{ site.url }}{{ site.baseurl }}/assets/images/gcp-125.png)



- Create a configuration for a custom-mode network.
- Create a configuration for a firewall rule.
- Create a module for VM instances.
- Create a configuration for an auto-mode network.
- Create and deploy a configuration.
- Verify the deployment of a configuration.



## Task 1. Set up Terraform and Cloud Shell

```sh
Welcome to Cloud Shell! Type "help" to get started.
Your Cloud Platform project in this session is set to qwiklabs-gcp-01-6ad4cc005c73.
Use “gcloud config set project [PROJECT_ID]” to change to a different project.
student_01_f491365e7a74@cloudshell:~ (qwiklabs-gcp-01-6ad4cc005c73)$ terraform --version
Terraform v1.3.1
on linux_amd64

Your version of Terraform is out of date! The latest version
is 1.3.2. You can update by downloading from https://www.terraform.io/downloads.html
student_01_f491365e7a74@cloudshell:~ (qwiklabs-gcp-01-6ad4cc005c73)$ mkdir tfnet
student_01_f491365e7a74@cloudshell:~ (qwiklabs-gcp-01-6ad4cc005c73)$ cd tfnet/
student_01_f491365e7a74@cloudshell:~/tfnet (qwiklabs-gcp-01-6ad4cc005c73)$ terraform init

Initializing the backend...

Initializing provider plugins...
- Finding latest version of hashicorp/google...
- Installing hashicorp/google v4.39.0...
- Installed hashicorp/google v4.39.0 (signed by HashiCorp)

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
student_01_f491365e7a74@cloudshell:~/tfnet (qwiklabs-gcp-01-6ad4cc005c73)$
```

## Task 2. Create managementnet and its resources

Create the custom-mode network **managementnet** along with its firewall rule and VM instance (**managementnet-us-vm**).

```sh
student_01_f491365e7a74@cloudshell:~ (qwiklabs-gcp-01-6ad4cc005c73)$ cd tfnet/
student_01_f491365e7a74@cloudshell:~/tfnet (qwiklabs-gcp-01-6ad4cc005c73)$ terraform init

Initializing the backend...

Initializing provider plugins...
- Finding latest version of hashicorp/google...
- Installing hashicorp/google v4.39.0...
- Installed hashicorp/google v4.39.0 (signed by HashiCorp)

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
student_01_f491365e7a74@cloudshell:~/tfnet (qwiklabs-gcp-01-6ad4cc005c73)$
student_01_f491365e7a74@cloudshell:~/tfnet (qwiklabs-gcp-01-6ad4cc005c73)$
student_01_f491365e7a74@cloudshell:~/tfnet (qwiklabs-gcp-01-6ad4cc005c73)$ terraform fmt
managementnet.tf
student_01_f491365e7a74@cloudshell:~/tfnet (qwiklabs-gcp-01-6ad4cc005c73)$ ls
instance  managementnet.tf  provider.tf
student_01_f491365e7a74@cloudshell:~/tfnet (qwiklabs-gcp-01-6ad4cc005c73)$ cat managementnet.tf
# Create the managementnet network
resource "google_compute_network" "managementnet" {
  name = "managementnet"
  #RESOURCE properties go here
  auto_create_subnetworks = "false"
}
# Create managementsubnet-us subnetwork
resource "google_compute_subnetwork" "managementsubnet-us" {
  name          = "managementsubnet-us"
  region        = "us-central1"
  network       = google_compute_network.managementnet.self_link
  ip_cidr_range = "10.130.0.0/20"
}
# Add a firewall rule to allow HTTP, SSH, and RDP traffic on managementnet
resource "google_compute_firewall" "managementnet-allow-http-ssh-rdp-icmp" {
  name    = "managementnet-allow-http-ssh-rdp-icmp"
  network = google_compute_network.managementnet.self_link
  allow {
    protocol = "tcp"
    ports    = ["22", "80", "3389"]
  }
  source_ranges = ["0.0.0.0/0"]
  allow {
    protocol = "icmp"
  }
}
# Add the managementnet-us-vm instance
module "managementnet-us-vm" {
  source              = "./instance"
  instance_name       = "managementnet-us-vm"
  instance_zone       = "us-central1-a"
  instance_subnetwork = google_compute_subnetwork.managementsubnet-us.self_link
}student_01_f491365e7a74@cloudshell:~/tfnet (qwiklabs-gcp-01-6ad4cc005c73)$ cat provider.tf
provider "google" {}student_01_f491365e7a74@cloudshell:~/tfnet (qwiklabs-gcp-01-6ad4cc005c73)$ cat instance/main.tf
variable "instance_name" {}
variable "instance_zone" {}
variable "instance_type" {
  default = "n1-standard-1"
  }
variable "instance_subnetwork" {}
resource "google_compute_instance" "vm_instance" {
  name         = "${var.instance_name}"
  zone         = "${var.instance_zone}"
  machine_type = "${var.instance_type}"
  boot_disk {
    initialize_params {
      image = "debian-cloud/debian-11"
      }
  }
  network_interface {
    subnetwork = "${var.instance_subnetwork}"
    access_config {
      # Allocate a one-to-one NAT IP to the instance
    }
  }
}student_01_f491365e7a74@cloudshell:~/tfnet (qwiklabs-gcp-01-6ad4cc005c73)$ terraform init
Initializing modules...
- managementnet-us-vm in instance

Initializing the backend...

Initializing provider plugins...
- Reusing previous version of hashicorp/google from the dependency lock file
- Using previously-installed hashicorp/google v4.39.0

Terraform has been successfully initialized!

You may now begin working with Terraform. Try running "terraform plan" to see
any changes that are required for your infrastructure. All Terraform commands
should now work.

If you ever set or change modules or backend configuration for Terraform,
rerun this command to reinitialize your working directory. If you forget, other
commands will detect it and remind you to do so if necessary.
student_01_f491365e7a74@cloudshell:~/tfnet (qwiklabs-gcp-01-6ad4cc005c73)$ terraform plan

Terraform used the selected providers to generate the following execution plan. Resource actions are indicated with the following symbols:
  + create

Terraform will perform the following actions:

  # google_compute_firewall.managementnet-allow-http-ssh-rdp-icmp will be created
  + resource "google_compute_firewall" "managementnet-allow-http-ssh-rdp-icmp" {
      + creation_timestamp = (known after apply)
      + destination_ranges = (known after apply)
      + direction          = (known after apply)
      + enable_logging     = (known after apply)
      + id                 = (known after apply)
      + name               = "managementnet-allow-http-ssh-rdp-icmp"
      + network            = (known after apply)
      + priority           = 1000
      + project            = (known after apply)
      + self_link          = (known after apply)
      + source_ranges      = [
          + "0.0.0.0/0",
        ]

      + allow {
          + ports    = [
              + "22",
              + "80",
              + "3389",
            ]
          + protocol = "tcp"
        }
      + allow {
          + ports    = []
          + protocol = "icmp"
        }
    }

  # google_compute_network.managementnet will be created
  + resource "google_compute_network" "managementnet" {
      + auto_create_subnetworks         = false
      + delete_default_routes_on_create = false
      + gateway_ipv4                    = (known after apply)
      + id                              = (known after apply)
      + internal_ipv6_range             = (known after apply)
      + mtu                             = (known after apply)
      + name                            = "managementnet"
      + project                         = (known after apply)
      + routing_mode                    = (known after apply)
      + self_link                       = (known after apply)
    }

  # google_compute_subnetwork.managementsubnet-us will be created
  + resource "google_compute_subnetwork" "managementsubnet-us" {
      + creation_timestamp         = (known after apply)
      + external_ipv6_prefix       = (known after apply)
      + fingerprint                = (known after apply)
      + gateway_address            = (known after apply)
      + id                         = (known after apply)
      + ip_cidr_range              = "10.130.0.0/20"
      + ipv6_cidr_range            = (known after apply)
      + name                       = "managementsubnet-us"
      + network                    = (known after apply)
      + private_ipv6_google_access = (known after apply)
      + project                    = (known after apply)
      + purpose                    = (known after apply)
      + region                     = "us-central1"
      + secondary_ip_range         = (known after apply)
      + self_link                  = (known after apply)
      + stack_type                 = (known after apply)
    }

  # module.managementnet-us-vm.google_compute_instance.vm_instance will be created
  + resource "google_compute_instance" "vm_instance" {
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
      + name                 = "managementnet-us-vm"
      + project              = (known after apply)
      + self_link            = (known after apply)
      + tags_fingerprint     = (known after apply)
      + zone                 = "us-central1-a"

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

      + confidential_instance_config {
          + enable_confidential_compute = (known after apply)
        }

      + network_interface {
          + ipv6_access_type   = (known after apply)
          + name               = (known after apply)
          + network            = (known after apply)
          + network_ip         = (known after apply)
          + stack_type         = (known after apply)
          + subnetwork         = (known after apply)
          + subnetwork_project = (known after apply)

          + access_config {
              + nat_ip       = (known after apply)
              + network_tier = (known after apply)
            }
        }

      + reservation_affinity {
          + type = (known after apply)

          + specific_reservation {
              + key    = (known after apply)
              + values = (known after apply)
            }
        }

      + scheduling {
          + automatic_restart           = (known after apply)
          + instance_termination_action = (known after apply)
          + min_node_cpus               = (known after apply)
          + on_host_maintenance         = (known after apply)
          + preemptible                 = (known after apply)
          + provisioning_model          = (known after apply)

          + node_affinities {
              + key      = (known after apply)
              + operator = (known after apply)
              + values   = (known after apply)
            }
        }
    }

Plan: 4 to add, 0 to change, 0 to destroy.

───────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────

Note: You didn't use the -out option to save this plan, so Terraform can't guarantee to take exactly these actions if you run "terraform apply" now.
student_01_f491365e7a74@cloudshell:~/tfnet (qwiklabs-gcp-01-6ad4cc005c73)$ terraform apply

Terraform used the selected providers to generate the following execution plan. Resource actions are indicated with the following symbols:
  + create

Terraform will perform the following actions:

  # google_compute_firewall.managementnet-allow-http-ssh-rdp-icmp will be created
  + resource "google_compute_firewall" "managementnet-allow-http-ssh-rdp-icmp" {
      + creation_timestamp = (known after apply)
      + destination_ranges = (known after apply)
      + direction          = (known after apply)
      + enable_logging     = (known after apply)
      + id                 = (known after apply)
      + name               = "managementnet-allow-http-ssh-rdp-icmp"
      + network            = (known after apply)
      + priority           = 1000
      + project            = (known after apply)
      + self_link          = (known after apply)
      + source_ranges      = [
          + "0.0.0.0/0",
        ]

      + allow {
          + ports    = [
              + "22",
              + "80",
              + "3389",
            ]
          + protocol = "tcp"
        }
      + allow {
          + ports    = []
          + protocol = "icmp"
        }
    }

  # google_compute_network.managementnet will be created
  + resource "google_compute_network" "managementnet" {
      + auto_create_subnetworks         = false
      + delete_default_routes_on_create = false
      + gateway_ipv4                    = (known after apply)
      + id                              = (known after apply)
      + internal_ipv6_range             = (known after apply)
      + mtu                             = (known after apply)
      + name                            = "managementnet"
      + project                         = (known after apply)
      + routing_mode                    = (known after apply)
      + self_link                       = (known after apply)
    }

  # google_compute_subnetwork.managementsubnet-us will be created
  + resource "google_compute_subnetwork" "managementsubnet-us" {
      + creation_timestamp         = (known after apply)
      + external_ipv6_prefix       = (known after apply)
      + fingerprint                = (known after apply)
      + gateway_address            = (known after apply)
      + id                         = (known after apply)
      + ip_cidr_range              = "10.130.0.0/20"
      + ipv6_cidr_range            = (known after apply)
      + name                       = "managementsubnet-us"
      + network                    = (known after apply)
      + private_ipv6_google_access = (known after apply)
      + project                    = (known after apply)
      + purpose                    = (known after apply)
      + region                     = "us-central1"
      + secondary_ip_range         = (known after apply)
      + self_link                  = (known after apply)
      + stack_type                 = (known after apply)
    }

  # module.managementnet-us-vm.google_compute_instance.vm_instance will be created
  + resource "google_compute_instance" "vm_instance" {
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
      + name                 = "managementnet-us-vm"
      + project              = (known after apply)
      + self_link            = (known after apply)
      + tags_fingerprint     = (known after apply)
      + zone                 = "us-central1-a"

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

      + confidential_instance_config {
          + enable_confidential_compute = (known after apply)
        }

      + network_interface {
          + ipv6_access_type   = (known after apply)
          + name               = (known after apply)
          + network            = (known after apply)
          + network_ip         = (known after apply)
          + stack_type         = (known after apply)
          + subnetwork         = (known after apply)
          + subnetwork_project = (known after apply)

          + access_config {
              + nat_ip       = (known after apply)
              + network_tier = (known after apply)
            }
        }

      + reservation_affinity {
          + type = (known after apply)

          + specific_reservation {
              + key    = (known after apply)
              + values = (known after apply)
            }
        }

      + scheduling {
          + automatic_restart           = (known after apply)
          + instance_termination_action = (known after apply)
          + min_node_cpus               = (known after apply)
          + on_host_maintenance         = (known after apply)
          + preemptible                 = (known after apply)
          + provisioning_model          = (known after apply)

          + node_affinities {
              + key      = (known after apply)
              + operator = (known after apply)
              + values   = (known after apply)
            }
        }
    }

Plan: 4 to add, 0 to change, 0 to destroy.

Do you want to perform these actions?
  Terraform will perform the actions described above.
  Only 'yes' will be accepted to approve.

  Enter a value: yes

google_compute_network.managementnet: Creating...
google_compute_network.managementnet: Still creating... [10s elapsed]
google_compute_network.managementnet: Creation complete after 12s [id=projects/qwiklabs-gcp-01-6ad4cc005c73/global/networks/managementnet]
google_compute_subnetwork.managementsubnet-us: Creating...
google_compute_firewall.managementnet-allow-http-ssh-rdp-icmp: Creating...
google_compute_subnetwork.managementsubnet-us: Still creating... [10s elapsed]
google_compute_firewall.managementnet-allow-http-ssh-rdp-icmp: Still creating... [10s elapsed]
google_compute_firewall.managementnet-allow-http-ssh-rdp-icmp: Creation complete after 12s [id=projects/qwiklabs-gcp-01-6ad4cc005c73/global/firewalls/managementnet-allow-http-ssh-rdp-icmp]
google_compute_subnetwork.managementsubnet-us: Creation complete after 17s [id=projects/qwiklabs-gcp-01-6ad4cc005c73/regions/us-central1/subnetworks/managementsubnet-us]
module.managementnet-us-vm.google_compute_instance.vm_instance: Creating...
module.managementnet-us-vm.google_compute_instance.vm_instance: Still creating... [10s elapsed]
module.managementnet-us-vm.google_compute_instance.vm_instance: Creation complete after 19s [id=projects/qwiklabs-gcp-01-6ad4cc005c73/zones/us-central1-a/instances/managementnet-us-vm]

Apply complete! Resources: 4 added, 0 changed, 0 destroyed.
student_01_f491365e7a74@cloudshell:~/tfnet (qwiklabs-gcp-01-6ad4cc005c73)$
```

## Task 3. Create privatenet and its resources

Create the custom-mode network **privatenet** along with its firewall rule and VM instance (**privatenet-us-vm**)

```sh
student_01_f491365e7a74@cloudshell:~/tfnet (qwiklabs-gcp-01-6ad4cc005c73)$ cat privatenet.tf
# Create privatenet network
resource "google_compute_network" "privatenet" {
name                    = "privatenet"
auto_create_subnetworks = false
}
# Create privatesubnet-us subnetwork
resource "google_compute_subnetwork" "privatesubnet-us" {
name          = "privatesubnet-us"
region        = "us-central1"
network       = google_compute_network.privatenet.self_link
ip_cidr_range = "172.16.0.0/24"
}
# Create privatesubnet-eu subnetwork
resource "google_compute_subnetwork" "privatesubnet-eu" {
name          = "privatesubnet-eu"
region        = "europe-west1"
network       = google_compute_network.privatenet.self_link
ip_cidr_range = "172.20.0.0/24"
}
# Create a firewall rule to allow HTTP, SSH, RDP and ICMP traffic on privatenet
resource "google_compute_firewall" "privatenet-allow-http-ssh-rdp-icmp" {
name    = "privatenet-allow-http-ssh-rdp-icmp"
network = google_compute_network.privatenet.self_link
allow {
    protocol = "tcp"
    ports    = ["22", "80", "3389"]
    }
source_ranges = ["0.0.0.0/0"]
allow {
    protocol = "icmp"
    }
}
# Add the privatenet-us-vm instance
module "privatenet-us-vm" {
  source           = "./instance"
  instance_name    = "privatenet-us-vm"
  instance_zone    = "us-central1-a"
  instance_subnetwork = google_compute_subnetwork.privatesubnet-us.self_link
}
student_01_f491365e7a74@cloudshell:~/tfnet (qwiklabs-gcp-01-6ad4cc005c73)$
```

```sh
student_01_f491365e7a74@cloudshell:~/tfnet (qwiklabs-gcp-01-6ad4cc005c73)$ terraform fmt
privatenet.tf
student_01_f491365e7a74@cloudshell:~/tfnet (qwiklabs-gcp-01-6ad4cc005c73)$ terraform init
Initializing modules...
- privatenet-us-vm in instance

Initializing the backend...

Initializing provider plugins...
- Reusing previous version of hashicorp/google from the dependency lock file
- Using previously-installed hashicorp/google v4.39.0

Terraform has been successfully initialized!

You may now begin working with Terraform. Try running "terraform plan" to see
any changes that are required for your infrastructure. All Terraform commands
should now work.

If you ever set or change modules or backend configuration for Terraform,
rerun this command to reinitialize your working directory. If you forget, other
commands will detect it and remind you to do so if necessary.
student_01_f491365e7a74@cloudshell:~/tfnet (qwiklabs-gcp-01-6ad4cc005c73)$ terraform plan
google_compute_network.managementnet: Refreshing state... [id=projects/qwiklabs-gcp-01-6ad4cc005c73/global/networks/managementnet]
google_compute_subnetwork.managementsubnet-us: Refreshing state... [id=projects/qwiklabs-gcp-01-6ad4cc005c73/regions/us-central1/subnetworks/managementsubnet-us]
google_compute_firewall.managementnet-allow-http-ssh-rdp-icmp: Refreshing state... [id=projects/qwiklabs-gcp-01-6ad4cc005c73/global/firewalls/managementnet-allow-http-ssh-rdp-icmp]
module.managementnet-us-vm.google_compute_instance.vm_instance: Refreshing state... [id=projects/qwiklabs-gcp-01-6ad4cc005c73/zones/us-central1-a/instances/managementnet-us-vm]

Terraform used the selected providers to generate the following execution plan. Resource actions are indicated with the following symbols:
  + create

Terraform will perform the following actions:

  # google_compute_firewall.privatenet-allow-http-ssh-rdp-icmp will be created
  + resource "google_compute_firewall" "privatenet-allow-http-ssh-rdp-icmp" {
      + creation_timestamp = (known after apply)
      + destination_ranges = (known after apply)
      + direction          = (known after apply)
      + enable_logging     = (known after apply)
      + id                 = (known after apply)
      + name               = "privatenet-allow-http-ssh-rdp-icmp"
      + network            = (known after apply)
      + priority           = 1000
      + project            = (known after apply)
      + self_link          = (known after apply)
      + source_ranges      = [
          + "0.0.0.0/0",
        ]

      + allow {
          + ports    = [
              + "22",
              + "80",
              + "3389",
            ]
          + protocol = "tcp"
        }
      + allow {
          + ports    = []
          + protocol = "icmp"
        }
    }

  # google_compute_network.privatenet will be created
  + resource "google_compute_network" "privatenet" {
      + auto_create_subnetworks         = false
      + delete_default_routes_on_create = false
      + gateway_ipv4                    = (known after apply)
      + id                              = (known after apply)
      + internal_ipv6_range             = (known after apply)
      + mtu                             = (known after apply)
      + name                            = "privatenet"
      + project                         = (known after apply)
      + routing_mode                    = (known after apply)
      + self_link                       = (known after apply)
    }

  # google_compute_subnetwork.privatesubnet-eu will be created
  + resource "google_compute_subnetwork" "privatesubnet-eu" {
      + creation_timestamp         = (known after apply)
      + external_ipv6_prefix       = (known after apply)
      + fingerprint                = (known after apply)
      + gateway_address            = (known after apply)
      + id                         = (known after apply)
      + ip_cidr_range              = "172.20.0.0/24"
      + ipv6_cidr_range            = (known after apply)
      + name                       = "privatesubnet-eu"
      + network                    = (known after apply)
      + private_ipv6_google_access = (known after apply)
      + project                    = (known after apply)
      + purpose                    = (known after apply)
      + region                     = "europe-west1"
      + secondary_ip_range         = (known after apply)
      + self_link                  = (known after apply)
      + stack_type                 = (known after apply)
    }

  # google_compute_subnetwork.privatesubnet-us will be created
  + resource "google_compute_subnetwork" "privatesubnet-us" {
      + creation_timestamp         = (known after apply)
      + external_ipv6_prefix       = (known after apply)
      + fingerprint                = (known after apply)
      + gateway_address            = (known after apply)
      + id                         = (known after apply)
      + ip_cidr_range              = "172.16.0.0/24"
      + ipv6_cidr_range            = (known after apply)
      + name                       = "privatesubnet-us"
      + network                    = (known after apply)
      + private_ipv6_google_access = (known after apply)
      + project                    = (known after apply)
      + purpose                    = (known after apply)
      + region                     = "us-central1"
      + secondary_ip_range         = (known after apply)
      + self_link                  = (known after apply)
      + stack_type                 = (known after apply)
    }

  # module.privatenet-us-vm.google_compute_instance.vm_instance will be created
  + resource "google_compute_instance" "vm_instance" {
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
      + name                 = "privatenet-us-vm"
      + project              = (known after apply)
      + self_link            = (known after apply)
      + tags_fingerprint     = (known after apply)
      + zone                 = "us-central1-a"

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

      + confidential_instance_config {
          + enable_confidential_compute = (known after apply)
        }

      + network_interface {
          + ipv6_access_type   = (known after apply)
          + name               = (known after apply)
          + network            = (known after apply)
          + network_ip         = (known after apply)
          + stack_type         = (known after apply)
          + subnetwork         = (known after apply)
          + subnetwork_project = (known after apply)

          + access_config {
              + nat_ip       = (known after apply)
              + network_tier = (known after apply)
            }
        }

      + reservation_affinity {
          + type = (known after apply)

          + specific_reservation {
              + key    = (known after apply)
              + values = (known after apply)
            }
        }

      + scheduling {
          + automatic_restart           = (known after apply)
          + instance_termination_action = (known after apply)
          + min_node_cpus               = (known after apply)
          + on_host_maintenance         = (known after apply)
          + preemptible                 = (known after apply)
          + provisioning_model          = (known after apply)

          + node_affinities {
              + key      = (known after apply)
              + operator = (known after apply)
              + values   = (known after apply)
            }
        }
    }

Plan: 5 to add, 0 to change, 0 to destroy.

───────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────

Note: You didn't use the -out option to save this plan, so Terraform can't guarantee to take exactly these actions if you run "terraform apply" now.
student_01_f491365e7a74@cloudshell:~/tfnet (qwiklabs-gcp-01-6ad4cc005c73)$ terraform apply
google_compute_network.managementnet: Refreshing state... [id=projects/qwiklabs-gcp-01-6ad4cc005c73/global/networks/managementnet]
google_compute_firewall.managementnet-allow-http-ssh-rdp-icmp: Refreshing state... [id=projects/qwiklabs-gcp-01-6ad4cc005c73/global/firewalls/managementnet-allow-http-ssh-rdp-icmp]
google_compute_subnetwork.managementsubnet-us: Refreshing state... [id=projects/qwiklabs-gcp-01-6ad4cc005c73/regions/us-central1/subnetworks/managementsubnet-us]
module.managementnet-us-vm.google_compute_instance.vm_instance: Refreshing state... [id=projects/qwiklabs-gcp-01-6ad4cc005c73/zones/us-central1-a/instances/managementnet-us-vm]

Terraform used the selected providers to generate the following execution plan. Resource actions are indicated with the following symbols:
  + create

Terraform will perform the following actions:

  # google_compute_firewall.privatenet-allow-http-ssh-rdp-icmp will be created
  + resource "google_compute_firewall" "privatenet-allow-http-ssh-rdp-icmp" {
      + creation_timestamp = (known after apply)
      + destination_ranges = (known after apply)
      + direction          = (known after apply)
      + enable_logging     = (known after apply)
      + id                 = (known after apply)
      + name               = "privatenet-allow-http-ssh-rdp-icmp"
      + network            = (known after apply)
      + priority           = 1000
      + project            = (known after apply)
      + self_link          = (known after apply)
      + source_ranges      = [
          + "0.0.0.0/0",
        ]

      + allow {
          + ports    = [
              + "22",
              + "80",
              + "3389",
            ]
          + protocol = "tcp"
        }
      + allow {
          + ports    = []
          + protocol = "icmp"
        }
    }

  # google_compute_network.privatenet will be created
  + resource "google_compute_network" "privatenet" {
      + auto_create_subnetworks         = false
      + delete_default_routes_on_create = false
      + gateway_ipv4                    = (known after apply)
      + id                              = (known after apply)
      + internal_ipv6_range             = (known after apply)
      + mtu                             = (known after apply)
      + name                            = "privatenet"
      + project                         = (known after apply)
      + routing_mode                    = (known after apply)
      + self_link                       = (known after apply)
    }

  # google_compute_subnetwork.privatesubnet-eu will be created
  + resource "google_compute_subnetwork" "privatesubnet-eu" {
      + creation_timestamp         = (known after apply)
      + external_ipv6_prefix       = (known after apply)
      + fingerprint                = (known after apply)
      + gateway_address            = (known after apply)
      + id                         = (known after apply)
      + ip_cidr_range              = "172.20.0.0/24"
      + ipv6_cidr_range            = (known after apply)
      + name                       = "privatesubnet-eu"
      + network                    = (known after apply)
      + private_ipv6_google_access = (known after apply)
      + project                    = (known after apply)
      + purpose                    = (known after apply)
      + region                     = "europe-west1"
      + secondary_ip_range         = (known after apply)
      + self_link                  = (known after apply)
      + stack_type                 = (known after apply)
    }

  # google_compute_subnetwork.privatesubnet-us will be created
  + resource "google_compute_subnetwork" "privatesubnet-us" {
      + creation_timestamp         = (known after apply)
      + external_ipv6_prefix       = (known after apply)
      + fingerprint                = (known after apply)
      + gateway_address            = (known after apply)
      + id                         = (known after apply)
      + ip_cidr_range              = "172.16.0.0/24"
      + ipv6_cidr_range            = (known after apply)
      + name                       = "privatesubnet-us"
      + network                    = (known after apply)
      + private_ipv6_google_access = (known after apply)
      + project                    = (known after apply)
      + purpose                    = (known after apply)
      + region                     = "us-central1"
      + secondary_ip_range         = (known after apply)
      + self_link                  = (known after apply)
      + stack_type                 = (known after apply)
    }

  # module.privatenet-us-vm.google_compute_instance.vm_instance will be created
  + resource "google_compute_instance" "vm_instance" {
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
      + name                 = "privatenet-us-vm"
      + project              = (known after apply)
      + self_link            = (known after apply)
      + tags_fingerprint     = (known after apply)
      + zone                 = "us-central1-a"

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

      + confidential_instance_config {
          + enable_confidential_compute = (known after apply)
        }

      + network_interface {
          + ipv6_access_type   = (known after apply)
          + name               = (known after apply)
          + network            = (known after apply)
          + network_ip         = (known after apply)
          + stack_type         = (known after apply)
          + subnetwork         = (known after apply)
          + subnetwork_project = (known after apply)

          + access_config {
              + nat_ip       = (known after apply)
              + network_tier = (known after apply)
            }
        }

      + reservation_affinity {
          + type = (known after apply)

          + specific_reservation {
              + key    = (known after apply)
              + values = (known after apply)
            }
        }

      + scheduling {
          + automatic_restart           = (known after apply)
          + instance_termination_action = (known after apply)
          + min_node_cpus               = (known after apply)
          + on_host_maintenance         = (known after apply)
          + preemptible                 = (known after apply)
          + provisioning_model          = (known after apply)

          + node_affinities {
              + key      = (known after apply)
              + operator = (known after apply)
              + values   = (known after apply)
            }
        }
    }

Plan: 5 to add, 0 to change, 0 to destroy.

Do you want to perform these actions?
  Terraform will perform the actions described above.
  Only 'yes' will be accepted to approve.

  Enter a value: yes

google_compute_network.privatenet: Creating...
google_compute_network.privatenet: Still creating... [10s elapsed]
google_compute_network.privatenet: Creation complete after 12s [id=projects/qwiklabs-gcp-01-6ad4cc005c73/global/networks/privatenet]
google_compute_subnetwork.privatesubnet-us: Creating...
google_compute_subnetwork.privatesubnet-eu: Creating...
google_compute_firewall.privatenet-allow-http-ssh-rdp-icmp: Creating...
google_compute_subnetwork.privatesubnet-us: Still creating... [10s elapsed]
google_compute_subnetwork.privatesubnet-eu: Still creating... [10s elapsed]
google_compute_firewall.privatenet-allow-http-ssh-rdp-icmp: Still creating... [10s elapsed]
google_compute_firewall.privatenet-allow-http-ssh-rdp-icmp: Creation complete after 12s [id=projects/qwiklabs-gcp-01-6ad4cc005c73/global/firewalls/privatenet-allow-http-ssh-rdp-icmp]
google_compute_subnetwork.privatesubnet-us: Creation complete after 14s [id=projects/qwiklabs-gcp-01-6ad4cc005c73/regions/us-central1/subnetworks/privatesubnet-us]
module.privatenet-us-vm.google_compute_instance.vm_instance: Creating...
google_compute_subnetwork.privatesubnet-eu: Still creating... [20s elapsed]
module.privatenet-us-vm.google_compute_instance.vm_instance: Still creating... [10s elapsed]
google_compute_subnetwork.privatesubnet-eu: Still creating... [30s elapsed]
google_compute_subnetwork.privatesubnet-eu: Creation complete after 31s [id=projects/qwiklabs-gcp-01-6ad4cc005c73/regions/europe-west1/subnetworks/privatesubnet-eu]
module.privatenet-us-vm.google_compute_instance.vm_instance: Creation complete after 18s [id=projects/qwiklabs-gcp-01-6ad4cc005c73/zones/us-central1-a/instances/privatenet-us-vm]

Apply complete! Resources: 5 added, 0 changed, 0 destroyed.
student_01_f491365e7a74@cloudshell:~/tfnet (qwiklabs-gcp-01-6ad4cc005c73)$
```



## Task 4. Create mynetwork and its resources

Create the auto-mode network **mynetwork** along with its firewall rule and two VM instances (**mynet-us-vm** and **mynet-eu-vm**).

```sh
student_01_f491365e7a74@cloudshell:~/tfnet (qwiklabs-gcp-01-6ad4cc005c73)$ cat mynetwork.tf
# Create the mynetwork network
resource "google_compute_network" "mynetwork" {
name                    = "mynetwork"
#RESOURCE properties go here
auto_create_subnetworks = "true"
}

# Create a firewall rule to allow HTTP, SSH, RDP and ICMP traffic on mynetwork
resource "google_compute_firewall" "mynetwork_allow_http_ssh_rdp_icmp" {
name    = "mynetwork-allow-http-ssh-rdp-icmp"
network = google_compute_network.mynetwork.self_link
allow {
    protocol = "tcp"
    ports    = ["22", "80", "3389"]
    }
source_ranges = ["0.0.0.0/0"]
allow {
    protocol = "icmp"
    }
}
# Create the mynet-us-vm instance
module "mynet-us-vm" {
  source           = "./instance"
  instance_name    = "mynet-us-vm"
  instance_zone    = "us-central1-a"
  instance_subnetwork = google_compute_network.mynetwork.self_link
}
# Create the mynet-eu-vm" instance
module "mynet-eu-vm" {
  source           = "./instance"
  instance_name    = "mynet-eu-vm"
  instance_zone    = "europe-west1-d"
  instance_subnetwork = google_compute_network.mynetwork.self_link
}
student_01_f491365e7a74@cloudshell:~/tfnet (qwiklabs-gcp-01-6ad4cc005c73)$
```

```sh
student_01_f491365e7a74@cloudshell:~/tfnet (qwiklabs-gcp-01-6ad4cc005c73)$ terraform init
Initializing modules...

Initializing the backend...

Initializing provider plugins...
- Reusing previous version of hashicorp/google from the dependency lock file
- Using previously-installed hashicorp/google v4.39.0

Terraform has been successfully initialized!

You may now begin working with Terraform. Try running "terraform plan" to see
any changes that are required for your infrastructure. All Terraform commands
should now work.

If you ever set or change modules or backend configuration for Terraform,
rerun this command to reinitialize your working directory. If you forget, other
commands will detect it and remind you to do so if necessary.
student_01_f491365e7a74@cloudshell:~/tfnet (qwiklabs-gcp-01-6ad4cc005c73)$ terraform plan
google_compute_network.managementnet: Refreshing state... [id=projects/qwiklabs-gcp-01-6ad4cc005c73/global/networks/managementnet]
google_compute_network.privatenet: Refreshing state... [id=projects/qwiklabs-gcp-01-6ad4cc005c73/global/networks/privatenet]
google_compute_subnetwork.privatesubnet-us: Refreshing state... [id=projects/qwiklabs-gcp-01-6ad4cc005c73/regions/us-central1/subnetworks/privatesubnet-us]
google_compute_subnetwork.privatesubnet-eu: Refreshing state... [id=projects/qwiklabs-gcp-01-6ad4cc005c73/regions/europe-west1/subnetworks/privatesubnet-eu]
google_compute_firewall.privatenet-allow-http-ssh-rdp-icmp: Refreshing state... [id=projects/qwiklabs-gcp-01-6ad4cc005c73/global/firewalls/privatenet-allow-http-ssh-rdp-icmp]
google_compute_subnetwork.managementsubnet-us: Refreshing state... [id=projects/qwiklabs-gcp-01-6ad4cc005c73/regions/us-central1/subnetworks/managementsubnet-us]
google_compute_firewall.managementnet-allow-http-ssh-rdp-icmp: Refreshing state... [id=projects/qwiklabs-gcp-01-6ad4cc005c73/global/firewalls/managementnet-allow-http-ssh-rdp-icmp]
module.privatenet-us-vm.google_compute_instance.vm_instance: Refreshing state... [id=projects/qwiklabs-gcp-01-6ad4cc005c73/zones/us-central1-a/instances/privatenet-us-vm]
module.managementnet-us-vm.google_compute_instance.vm_instance: Refreshing state... [id=projects/qwiklabs-gcp-01-6ad4cc005c73/zones/us-central1-a/instances/managementnet-us-vm]

Terraform used the selected providers to generate the following execution plan. Resource actions are indicated with the following symbols:
  + create

Terraform will perform the following actions:

  # google_compute_firewall.mynetwork_allow_http_ssh_rdp_icmp will be created
  + resource "google_compute_firewall" "mynetwork_allow_http_ssh_rdp_icmp" {
      + creation_timestamp = (known after apply)
      + destination_ranges = (known after apply)
      + direction          = (known after apply)
      + enable_logging     = (known after apply)
      + id                 = (known after apply)
      + name               = "mynetwork-allow-http-ssh-rdp-icmp"
      + network            = (known after apply)
      + priority           = 1000
      + project            = (known after apply)
      + self_link          = (known after apply)
      + source_ranges      = [
          + "0.0.0.0/0",
        ]

      + allow {
          + ports    = [
              + "22",
              + "80",
              + "3389",
            ]
          + protocol = "tcp"
        }
      + allow {
          + ports    = []
          + protocol = "icmp"
        }
    }

  # google_compute_network.mynetwork will be created
  + resource "google_compute_network" "mynetwork" {
      + auto_create_subnetworks         = true
      + delete_default_routes_on_create = false
      + gateway_ipv4                    = (known after apply)
      + id                              = (known after apply)
      + internal_ipv6_range             = (known after apply)
      + mtu                             = (known after apply)
      + name                            = "mynetwork"
      + project                         = (known after apply)
      + routing_mode                    = (known after apply)
      + self_link                       = (known after apply)
    }

  # module.mynet-eu-vm.google_compute_instance.vm_instance will be created
  + resource "google_compute_instance" "vm_instance" {
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
      + name                 = "mynet-eu-vm"
      + project              = (known after apply)
      + self_link            = (known after apply)
      + tags_fingerprint     = (known after apply)
      + zone                 = "europe-west1-d"

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

      + confidential_instance_config {
          + enable_confidential_compute = (known after apply)
        }

      + network_interface {
          + ipv6_access_type   = (known after apply)
          + name               = (known after apply)
          + network            = (known after apply)
          + network_ip         = (known after apply)
          + stack_type         = (known after apply)
          + subnetwork         = (known after apply)
          + subnetwork_project = (known after apply)

          + access_config {
              + nat_ip       = (known after apply)
              + network_tier = (known after apply)
            }
        }

      + reservation_affinity {
          + type = (known after apply)

          + specific_reservation {
              + key    = (known after apply)
              + values = (known after apply)
            }
        }

      + scheduling {
          + automatic_restart           = (known after apply)
          + instance_termination_action = (known after apply)
          + min_node_cpus               = (known after apply)
          + on_host_maintenance         = (known after apply)
          + preemptible                 = (known after apply)
          + provisioning_model          = (known after apply)

          + node_affinities {
              + key      = (known after apply)
              + operator = (known after apply)
              + values   = (known after apply)
            }
        }
    }

  # module.mynet-us-vm.google_compute_instance.vm_instance will be created
  + resource "google_compute_instance" "vm_instance" {
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
      + name                 = "mynet-us-vm"
      + project              = (known after apply)
      + self_link            = (known after apply)
      + tags_fingerprint     = (known after apply)
      + zone                 = "us-central1-a"

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

      + confidential_instance_config {
          + enable_confidential_compute = (known after apply)
        }

      + network_interface {
          + ipv6_access_type   = (known after apply)
          + name               = (known after apply)
          + network            = (known after apply)
          + network_ip         = (known after apply)
          + stack_type         = (known after apply)
          + subnetwork         = (known after apply)
          + subnetwork_project = (known after apply)

          + access_config {
              + nat_ip       = (known after apply)
              + network_tier = (known after apply)
            }
        }

      + reservation_affinity {
          + type = (known after apply)

          + specific_reservation {
              + key    = (known after apply)
              + values = (known after apply)
            }
        }

      + scheduling {
          + automatic_restart           = (known after apply)
          + instance_termination_action = (known after apply)
          + min_node_cpus               = (known after apply)
          + on_host_maintenance         = (known after apply)
          + preemptible                 = (known after apply)
          + provisioning_model          = (known after apply)

          + node_affinities {
              + key      = (known after apply)
              + operator = (known after apply)
              + values   = (known after apply)
            }
        }
    }

Plan: 4 to add, 0 to change, 0 to destroy.

───────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────

Note: You didn't use the -out option to save this plan, so Terraform can't guarantee to take exactly these actions if you run "terraform apply" now.
student_01_f491365e7a74@cloudshell:~/tfnet (qwiklabs-gcp-01-6ad4cc005c73)$ terraform apply
google_compute_network.privatenet: Refreshing state... [id=projects/qwiklabs-gcp-01-6ad4cc005c73/global/networks/privatenet]
google_compute_network.managementnet: Refreshing state... [id=projects/qwiklabs-gcp-01-6ad4cc005c73/global/networks/managementnet]
google_compute_subnetwork.managementsubnet-us: Refreshing state... [id=projects/qwiklabs-gcp-01-6ad4cc005c73/regions/us-central1/subnetworks/managementsubnet-us]
google_compute_firewall.managementnet-allow-http-ssh-rdp-icmp: Refreshing state... [id=projects/qwiklabs-gcp-01-6ad4cc005c73/global/firewalls/managementnet-allow-http-ssh-rdp-icmp]
google_compute_subnetwork.privatesubnet-eu: Refreshing state... [id=projects/qwiklabs-gcp-01-6ad4cc005c73/regions/europe-west1/subnetworks/privatesubnet-eu]
google_compute_subnetwork.privatesubnet-us: Refreshing state... [id=projects/qwiklabs-gcp-01-6ad4cc005c73/regions/us-central1/subnetworks/privatesubnet-us]
google_compute_firewall.privatenet-allow-http-ssh-rdp-icmp: Refreshing state... [id=projects/qwiklabs-gcp-01-6ad4cc005c73/global/firewalls/privatenet-allow-http-ssh-rdp-icmp]
module.managementnet-us-vm.google_compute_instance.vm_instance: Refreshing state... [id=projects/qwiklabs-gcp-01-6ad4cc005c73/zones/us-central1-a/instances/managementnet-us-vm]
module.privatenet-us-vm.google_compute_instance.vm_instance: Refreshing state... [id=projects/qwiklabs-gcp-01-6ad4cc005c73/zones/us-central1-a/instances/privatenet-us-vm]

Terraform used the selected providers to generate the following execution plan. Resource actions are indicated with the following symbols:
  + create

Terraform will perform the following actions:

  # google_compute_firewall.mynetwork_allow_http_ssh_rdp_icmp will be created
  + resource "google_compute_firewall" "mynetwork_allow_http_ssh_rdp_icmp" {
      + creation_timestamp = (known after apply)
      + destination_ranges = (known after apply)
      + direction          = (known after apply)
      + enable_logging     = (known after apply)
      + id                 = (known after apply)
      + name               = "mynetwork-allow-http-ssh-rdp-icmp"
      + network            = (known after apply)
      + priority           = 1000
      + project            = (known after apply)
      + self_link          = (known after apply)
      + source_ranges      = [
          + "0.0.0.0/0",
        ]

      + allow {
          + ports    = [
              + "22",
              + "80",
              + "3389",
            ]
          + protocol = "tcp"
        }
      + allow {
          + ports    = []
          + protocol = "icmp"
        }
    }

  # google_compute_network.mynetwork will be created
  + resource "google_compute_network" "mynetwork" {
      + auto_create_subnetworks         = true
      + delete_default_routes_on_create = false
      + gateway_ipv4                    = (known after apply)
      + id                              = (known after apply)
      + internal_ipv6_range             = (known after apply)
      + mtu                             = (known after apply)
      + name                            = "mynetwork"
      + project                         = (known after apply)
      + routing_mode                    = (known after apply)
      + self_link                       = (known after apply)
    }

  # module.mynet-eu-vm.google_compute_instance.vm_instance will be created
  + resource "google_compute_instance" "vm_instance" {
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
      + name                 = "mynet-eu-vm"
      + project              = (known after apply)
      + self_link            = (known after apply)
      + tags_fingerprint     = (known after apply)
      + zone                 = "europe-west1-d"

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

      + confidential_instance_config {
          + enable_confidential_compute = (known after apply)
        }

      + network_interface {
          + ipv6_access_type   = (known after apply)
          + name               = (known after apply)
          + network            = (known after apply)
          + network_ip         = (known after apply)
          + stack_type         = (known after apply)
          + subnetwork         = (known after apply)
          + subnetwork_project = (known after apply)

          + access_config {
              + nat_ip       = (known after apply)
              + network_tier = (known after apply)
            }
        }

      + reservation_affinity {
          + type = (known after apply)

          + specific_reservation {
              + key    = (known after apply)
              + values = (known after apply)
            }
        }

      + scheduling {
          + automatic_restart           = (known after apply)
          + instance_termination_action = (known after apply)
          + min_node_cpus               = (known after apply)
          + on_host_maintenance         = (known after apply)
          + preemptible                 = (known after apply)
          + provisioning_model          = (known after apply)

          + node_affinities {
              + key      = (known after apply)
              + operator = (known after apply)
              + values   = (known after apply)
            }
        }
    }

  # module.mynet-us-vm.google_compute_instance.vm_instance will be created
  + resource "google_compute_instance" "vm_instance" {
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
      + name                 = "mynet-us-vm"
      + project              = (known after apply)
      + self_link            = (known after apply)
      + tags_fingerprint     = (known after apply)
      + zone                 = "us-central1-a"

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

      + confidential_instance_config {
          + enable_confidential_compute = (known after apply)
        }

      + network_interface {
          + ipv6_access_type   = (known after apply)
          + name               = (known after apply)
          + network            = (known after apply)
          + network_ip         = (known after apply)
          + stack_type         = (known after apply)
          + subnetwork         = (known after apply)
          + subnetwork_project = (known after apply)

          + access_config {
              + nat_ip       = (known after apply)
              + network_tier = (known after apply)
            }
        }

      + reservation_affinity {
          + type = (known after apply)

          + specific_reservation {
              + key    = (known after apply)
              + values = (known after apply)
            }
        }

      + scheduling {
          + automatic_restart           = (known after apply)
          + instance_termination_action = (known after apply)
          + min_node_cpus               = (known after apply)
          + on_host_maintenance         = (known after apply)
          + preemptible                 = (known after apply)
          + provisioning_model          = (known after apply)

          + node_affinities {
              + key      = (known after apply)
              + operator = (known after apply)
              + values   = (known after apply)
            }
        }
    }

Plan: 4 to add, 0 to change, 0 to destroy.

Do you want to perform these actions?
  Terraform will perform the actions described above.
  Only 'yes' will be accepted to approve.

  Enter a value: yes

google_compute_network.mynetwork: Creating...
google_compute_network.mynetwork: Still creating... [10s elapsed]
google_compute_network.mynetwork: Still creating... [20s elapsed]
google_compute_network.mynetwork: Still creating... [30s elapsed]
google_compute_network.mynetwork: Creation complete after 33s [id=projects/qwiklabs-gcp-01-6ad4cc005c73/global/networks/mynetwork]
google_compute_firewall.mynetwork_allow_http_ssh_rdp_icmp: Creating...
module.mynet-us-vm.google_compute_instance.vm_instance: Creating...
module.mynet-eu-vm.google_compute_instance.vm_instance: Creating...
google_compute_firewall.mynetwork_allow_http_ssh_rdp_icmp: Still creating... [10s elapsed]
module.mynet-us-vm.google_compute_instance.vm_instance: Still creating... [10s elapsed]
module.mynet-eu-vm.google_compute_instance.vm_instance: Still creating... [10s elapsed]
google_compute_firewall.mynetwork_allow_http_ssh_rdp_icmp: Creation complete after 12s [id=projects/qwiklabs-gcp-01-6ad4cc005c73/global/firewalls/mynetwork-allow-http-ssh-rdp-icmp]
module.mynet-us-vm.google_compute_instance.vm_instance: Creation complete after 19s [id=projects/qwiklabs-gcp-01-6ad4cc005c73/zones/us-central1-a/instances/mynet-us-vm]
module.mynet-eu-vm.google_compute_instance.vm_instance: Still creating... [20s elapsed]
module.mynet-eu-vm.google_compute_instance.vm_instance: Creation complete after 22s [id=projects/qwiklabs-gcp-01-6ad4cc005c73/zones/europe-west1-d/instances/mynet-eu-vm]

Apply complete! Resources: 4 added, 0 changed, 0 destroyed.
student_01_f491365e7a74@cloudshell:~/tfnet (qwiklabs-gcp-01-6ad4cc005c73)$
```


## Task 5. Review
In this lab, you created Terraform configurations and modules to automate the deployment of a custom network. As the configuration changes, Terraform can determine what changed and create incremental execution plans, which allows you to build your overall configuration step-by-step.

The instance module allowed you to re-use the same resource configuration for multiple resources while providing properties as input variables. You can leverage the configurations and modules that you created as a starting point for future deployments.



![]({{ site.url }}{{ site.baseurl }}/assets/images/gcp-126.png)

![]({{ site.url }}{{ site.baseurl }}/assets/images/gcp-127.png)

![]({{ site.url }}{{ site.baseurl }}/assets/images/gcp-128.png)

