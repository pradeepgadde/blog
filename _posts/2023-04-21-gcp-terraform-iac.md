---
layout: single
title:  "Infrastructure as Code with Terraform"
date:   2023-04-21 12:59:04 +0530
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

# GCP Infrastructure as Code with Terraform

- perform the following tasks:
  - Build, change, and destroy infrastructure with Terraform
  - Create Resource Dependencies with Terraform
  - Provision infrastructure with Terraform



```sh
Welcome to Cloud Shell! Type "help" to get started.
Your Cloud Platform project in this session is set to qwiklabs-gcp-01-298144853b0c.
Use “gcloud config set project [PROJECT_ID]” to change to a different project.
student_01_b9934b2a0bae@cloudshell:~ (qwiklabs-gcp-01-298144853b0c)$ ls
README-cloudshell.txt
student_01_b9934b2a0bae@cloudshell:~ (qwiklabs-gcp-01-298144853b0c)$ touch main.tf
student_01_b9934b2a0bae@cloudshell:~ (qwiklabs-gcp-01-298144853b0c)$ cat main.tf
terraform {
  required_providers {
    google = {
      source = "hashicorp/google"
    }
  }
}
provider "google" {
  version = "3.5.0"
  project = "qwiklabs-gcp-01-298144853b0c"
  region  = "us-central1"
  zone    = "us-central1-c"
}
resource "google_compute_network" "vpc_network" {
  name = "terraform-network"
}student_01_b9934b2a0bae@cloudshell:~ (qwiklabs-gcp-01-298144853b0c)$ terraform init

Initializing the backend...

Initializing provider plugins...
- Finding hashicorp/google versions matching "3.5.0"...
- Installing hashicorp/google v3.5.0...
- Installed hashicorp/google v3.5.0 (signed by HashiCorp)

Terraform has created a lock file .terraform.lock.hcl to record the provider
selections it made above. Include this file in your version control repository
so that Terraform can guarantee to make the same selections by default when
you run "terraform init" in the future.

╷
│ Warning: Version constraints inside provider configuration blocks are deprecated
│
│   on main.tf line 9, in provider "google":
│    9:   version = "3.5.0"
│
│ Terraform 0.13 and earlier allowed provider version constraints inside the provider configuration block, but that is now deprecated and will be removed in a future version of
│ Terraform. To silence this warning, move the provider version constraint into the required_providers block.
╵

Terraform has been successfully initialized!

You may now begin working with Terraform. Try running "terraform plan" to see
any changes that are required for your infrastructure. All Terraform commands
should now work.

If you ever set or change modules or backend configuration for Terraform,
rerun this command to reinitialize your working directory. If you forget, other
commands will detect it and remind you to do so if necessary.
student_01_b9934b2a0bae@cloudshell:~ (qwiklabs-gcp-01-298144853b0c)$
```



```sh
student_01_b9934b2a0bae@cloudshell:~ (qwiklabs-gcp-01-298144853b0c)$ terraform apply

Terraform used the selected providers to generate the following execution plan. Resource actions are indicated with the following symbols:
  + create

Terraform will perform the following actions:

  # google_compute_network.vpc_network will be created
  + resource "google_compute_network" "vpc_network" {
      + auto_create_subnetworks         = true
      + delete_default_routes_on_create = false
      + gateway_ipv4                    = (known after apply)
      + id                              = (known after apply)
      + ipv4_range                      = (known after apply)
      + name                            = "terraform-network"
      + project                         = (known after apply)
      + routing_mode                    = (known after apply)
      + self_link                       = (known after apply)
    }

Plan: 1 to add, 0 to change, 0 to destroy.
╷
│ Warning: Version constraints inside provider configuration blocks are deprecated
│
│   on main.tf line 9, in provider "google":
│    9:   version = "3.5.0"
│
│ Terraform 0.13 and earlier allowed provider version constraints inside the provider configuration block, but that is now deprecated and will be removed in a future version of
│ Terraform. To silence this warning, move the provider version constraint into the required_providers block.
╵

Do you want to perform these actions?
  Terraform will perform the actions described above.
  Only 'yes' will be accepted to approve.

  Enter a value: yes

google_compute_network.vpc_network: Creating...
google_compute_network.vpc_network: Still creating... [10s elapsed]
google_compute_network.vpc_network: Still creating... [20s elapsed]
google_compute_network.vpc_network: Still creating... [30s elapsed]
google_compute_network.vpc_network: Creation complete after 38s [id=projects/qwiklabs-gcp-01-298144853b0c/global/networks/terraform-network]

Apply complete! Resources: 1 added, 0 changed, 0 destroyed.
student_01_b9934b2a0bae@cloudshell:~ (qwiklabs-gcp-01-298144853b0c)$
```



![]({{ site.url }}{{ site.baseurl }}/assets/images/gcp-tf-iac-1.png)



```sh
student_01_b9934b2a0bae@cloudshell:~ (qwiklabs-gcp-01-298144853b0c)$ terraform show
# google_compute_network.vpc_network:
resource "google_compute_network" "vpc_network" {
    auto_create_subnetworks         = true
    delete_default_routes_on_create = false
    id                              = "projects/qwiklabs-gcp-01-298144853b0c/global/networks/terraform-network"
    name                            = "terraform-network"
    project                         = "qwiklabs-gcp-01-298144853b0c"
    routing_mode                    = "REGIONAL"
    self_link                       = "https://www.googleapis.com/compute/v1/projects/qwiklabs-gcp-01-298144853b0c/global/networks/terraform-network"
}
student_01_b9934b2a0bae@cloudshell:~ (qwiklabs-gcp-01-298144853b0c)$
```

In the previous section, you created basic infrastructure with  Terraform: a VPC network. In this section, you're going to modify your  configuration, and see how Terraform handles change.

Infrastructure is continuously evolving, and Terraform was built to  help manage and enact that change. As you change Terraform  configurations, Terraform builds an execution plan that only modifies  what is necessary to reach your desired state.

By using Terraform to change infrastructure, you can version control  not only your configurations but also your state so you can see how the  infrastructure evolves over time.

```sh
student_01_b9934b2a0bae@cloudshell:~ (qwiklabs-gcp-01-298144853b0c)$ cat main.tf
terraform {
  required_providers {
    google = {
      source = "hashicorp/google"
    }
  }
}
provider "google" {
  version = "3.5.0"
  project = "qwiklabs-gcp-01-298144853b0c"
  region  = "us-central1"
  zone    = "us-central1-c"
}
resource "google_compute_network" "vpc_network" {
  name = "terraform-network"
}
resource "google_compute_instance" "vm_instance" {
  name         = "terraform-instance"
  machine_type = "f1-micro"
  boot_disk {
    initialize_params {
      image = "debian-cloud/debian-11"
    }
  }
  network_interface {
    network = google_compute_network.vpc_network.name
    access_config {
    }
  }
}student_01_b9934b2a0bae@cloudshell:~ (qwiklabs-gcp-01-298144853b0c)$
```



```sh
student_01_b9934b2a0bae@cloudshell:~ (qwiklabs-gcp-01-298144853b0c)$ terraform apply
google_compute_network.vpc_network: Refreshing state... [id=projects/qwiklabs-gcp-01-298144853b0c/global/networks/terraform-network]

Terraform used the selected providers to generate the following execution plan. Resource actions are indicated with the following symbols:
  + create

Terraform will perform the following actions:

  # google_compute_instance.vm_instance will be created
  + resource "google_compute_instance" "vm_instance" {
      + can_ip_forward       = false
      + cpu_platform         = (known after apply)
      + deletion_protection  = false
      + guest_accelerator    = (known after apply)
      + id                   = (known after apply)
      + instance_id          = (known after apply)
      + label_fingerprint    = (known after apply)
      + machine_type         = "f1-micro"
      + metadata_fingerprint = (known after apply)
      + min_cpu_platform     = (known after apply)
      + name                 = "terraform-instance"
      + project              = (known after apply)
      + self_link            = (known after apply)
      + tags_fingerprint     = (known after apply)
      + zone                 = (known after apply)

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
          + name               = (known after apply)
          + network            = "terraform-network"
          + network_ip         = (known after apply)
          + subnetwork         = (known after apply)
          + subnetwork_project = (known after apply)

          + access_config {
              + nat_ip       = (known after apply)
              + network_tier = (known after apply)
            }
        }
    }

Plan: 1 to add, 0 to change, 0 to destroy.
╷
│ Warning: Version constraints inside provider configuration blocks are deprecated
│
│   on main.tf line 9, in provider "google":
│    9:   version = "3.5.0"
│
│ Terraform 0.13 and earlier allowed provider version constraints inside the provider configuration block, but that is now deprecated and will be removed in a future version of
│ Terraform. To silence this warning, move the provider version constraint into the required_providers block.
╵

Do you want to perform these actions?
  Terraform will perform the actions described above.
  Only 'yes' will be accepted to approve.

  Enter a value: yes

google_compute_instance.vm_instance: Creating...
google_compute_instance.vm_instance: Still creating... [10s elapsed]
google_compute_instance.vm_instance: Creation complete after 18s [id=projects/qwiklabs-gcp-01-298144853b0c/zones/us-central1-c/instances/terraform-instance]

Apply complete! Resources: 1 added, 0 changed, 0 destroyed.
student_01_b9934b2a0bae@cloudshell:~ (qwiklabs-gcp-01-298144853b0c)$
```

![]({{ site.url }}{{ site.baseurl }}/assets/images/gcp-tf-iac-2.png)

### Changing resources

In addition to creating resources, Terraform can also make changes to those resources.

1. Add a `tags` argument to your "vm_instance" so that it looks like this:

```sh
student_01_b9934b2a0bae@cloudshell:~ (qwiklabs-gcp-01-298144853b0c)$ cat main.tf
terraform {
  required_providers {
    google = {
      source = "hashicorp/google"
    }
  }
}
provider "google" {
  version = "3.5.0"
  project = "qwiklabs-gcp-01-298144853b0c"
  region  = "us-central1"
  zone    = "us-central1-c"
}
resource "google_compute_network" "vpc_network" {
  name = "terraform-network"
}
resource "google_compute_instance" "vm_instance" {
  name         = "terraform-instance"
  machine_type = "f1-micro"
  tags         = ["web", "dev"]
  boot_disk {
    initialize_params {
      image = "debian-cloud/debian-11"
    }
  }
  network_interface {
    network = google_compute_network.vpc_network.name
    access_config {
    }
  }
}student_01_b9934b2a0bae@cloudshell:~ (qwiklabs-gcp-01-298144853b0c)$
```



```sh
student_01_b9934b2a0bae@cloudshell:~ (qwiklabs-gcp-01-298144853b0c)$ terraform apply
google_compute_network.vpc_network: Refreshing state... [id=projects/qwiklabs-gcp-01-298144853b0c/global/networks/terraform-network]
google_compute_instance.vm_instance: Refreshing state... [id=projects/qwiklabs-gcp-01-298144853b0c/zones/us-central1-c/instances/terraform-instance]

Terraform used the selected providers to generate the following execution plan. Resource actions are indicated with the following symbols:
  ~ update in-place

Terraform will perform the following actions:

  # google_compute_instance.vm_instance will be updated in-place
  ~ resource "google_compute_instance" "vm_instance" {
        id                   = "projects/qwiklabs-gcp-01-298144853b0c/zones/us-central1-c/instances/terraform-instance"
        name                 = "terraform-instance"
      ~ tags                 = [
          + "dev",
          + "web",
        ]
        # (15 unchanged attributes hidden)

        # (4 unchanged blocks hidden)
    }

Plan: 0 to add, 1 to change, 0 to destroy.
╷
│ Warning: Version constraints inside provider configuration blocks are deprecated
│
│   on main.tf line 9, in provider "google":
│    9:   version = "3.5.0"
│
│ Terraform 0.13 and earlier allowed provider version constraints inside the provider configuration block, but that is now deprecated and will be removed in a future version of
│ Terraform. To silence this warning, move the provider version constraint into the required_providers block.
╵

Do you want to perform these actions?
  Terraform will perform the actions described above.
  Only 'yes' will be accepted to approve.

  Enter a value: yes

google_compute_instance.vm_instance: Modifying... [id=projects/qwiklabs-gcp-01-298144853b0c/zones/us-central1-c/instances/terraform-instance]
google_compute_instance.vm_instance: Still modifying... [id=projects/qwiklabs-gcp-01-298144853b0c/z...entral1-c/instances/terraform-instance, 10s elapsed]
google_compute_instance.vm_instance: Modifications complete after 17s [id=projects/qwiklabs-gcp-01-298144853b0c/zones/us-central1-c/instances/terraform-instance]

Apply complete! Resources: 0 added, 1 changed, 0 destroyed.
student_01_b9934b2a0bae@cloudshell:~ (qwiklabs-gcp-01-298144853b0c)$
```

![]({{ site.url }}{{ site.baseurl }}/assets/images/gcp-tf-iac-3.png)

### Destructive changes

A destructive change is a change that requires the provider to  replace the existing resource rather than updating it. This usually  happens because the cloud provider doesn't support updating the resource in the way described by your configuration.

Changing the disk image of your instance is one example of a destructive change.

Edit the `boot_disk` block inside the `vm_instance` resource in your configuration file and change it to the following:



```sh
student_01_b9934b2a0bae@cloudshell:~ (qwiklabs-gcp-01-298144853b0c)$ cat main.tf
terraform {
  required_providers {
    google = {
      source = "hashicorp/google"
    }
  }
}
provider "google" {
  version = "3.5.0"
  project = "qwiklabs-gcp-01-298144853b0c"
  region  = "us-central1"
  zone    = "us-central1-c"
}
resource "google_compute_network" "vpc_network" {
  name = "terraform-network"
}
resource "google_compute_instance" "vm_instance" {
  name         = "terraform-instance"
  machine_type = "f1-micro"
  tags         = ["web", "dev"]
  boot_disk {
    initialize_params {
      image = "cos-cloud/cos-stable"
    }
  }

  network_interface {
    network = google_compute_network.vpc_network.name
    access_config {
    }
  }
}student_01_b9934b2a0bae@cloudshell:~ (qwiklabs-gcp-01-298144853b0c)$
```

```sh
student_01_b9934b2a0bae@cloudshell:~ (qwiklabs-gcp-01-298144853b0c)$ terraform apply
google_compute_network.vpc_network: Refreshing state... [id=projects/qwiklabs-gcp-01-298144853b0c/global/networks/terraform-network]
google_compute_instance.vm_instance: Refreshing state... [id=projects/qwiklabs-gcp-01-298144853b0c/zones/us-central1-c/instances/terraform-instance]

Terraform used the selected providers to generate the following execution plan. Resource actions are indicated with the following symbols:
-/+ destroy and then create replacement

Terraform will perform the following actions:

  # google_compute_instance.vm_instance must be replaced
-/+ resource "google_compute_instance" "vm_instance" {
      ~ cpu_platform         = "Intel Haswell" -> (known after apply)
      - enable_display       = false -> null
      ~ guest_accelerator    = [] -> (known after apply)
      ~ id                   = "projects/qwiklabs-gcp-01-298144853b0c/zones/us-central1-c/instances/terraform-instance" -> (known after apply)
      ~ instance_id          = "7236863538611822261" -> (known after apply)
      ~ label_fingerprint    = "42WmSpB8rSM=" -> (known after apply)
      - labels               = {} -> null
      - metadata             = {} -> null
      ~ metadata_fingerprint = "-IVbTFitpCY=" -> (known after apply)
      + min_cpu_platform     = (known after apply)
        name                 = "terraform-instance"
      ~ project              = "qwiklabs-gcp-01-298144853b0c" -> (known after apply)
      ~ self_link            = "https://www.googleapis.com/compute/v1/projects/qwiklabs-gcp-01-298144853b0c/zones/us-central1-c/instances/terraform-instance" -> (known after apply)
        tags                 = [
            "dev",
            "web",
        ]
      ~ tags_fingerprint     = "XaeQnaHMn9Y=" -> (known after apply)
      ~ zone                 = "us-central1-c" -> (known after apply)
        # (3 unchanged attributes hidden)

      ~ boot_disk {
          ~ device_name                = "persistent-disk-0" -> (known after apply)
          + disk_encryption_key_sha256 = (known after apply)
          + kms_key_self_link          = (known after apply)
          ~ source                     = "https://www.googleapis.com/compute/v1/projects/qwiklabs-gcp-01-298144853b0c/zones/us-central1-c/disks/terraform-instance" -> (known after apply)
            # (2 unchanged attributes hidden)

          ~ initialize_params {
              ~ image  = "https://www.googleapis.com/compute/v1/projects/debian-cloud/global/images/debian-11-bullseye-v20230411" -> "cos-cloud/cos-stable" # forces replacement
              ~ labels = {} -> (known after apply)
              ~ size   = 10 -> (known after apply)
              ~ type   = "pd-standard" -> (known after apply)
            }
        }

      ~ network_interface {
          ~ name               = "nic0" -> (known after apply)
          ~ network            = "https://www.googleapis.com/compute/v1/projects/qwiklabs-gcp-01-298144853b0c/global/networks/terraform-network" -> "terraform-network"
          ~ network_ip         = "10.128.0.2" -> (known after apply)
          ~ subnetwork         = "https://www.googleapis.com/compute/v1/projects/qwiklabs-gcp-01-298144853b0c/regions/us-central1/subnetworks/terraform-network" -> (known after apply)
          ~ subnetwork_project = "qwiklabs-gcp-01-298144853b0c" -> (known after apply)

          ~ access_config {
              ~ nat_ip       = "35.226.140.52" -> (known after apply)
              ~ network_tier = "PREMIUM" -> (known after apply)
            }
        }

      - scheduling {
          - automatic_restart   = true -> null
          - on_host_maintenance = "MIGRATE" -> null
          - preemptible         = false -> null
        }

      - shielded_instance_config {
          - enable_integrity_monitoring = true -> null
          - enable_secure_boot          = false -> null
          - enable_vtpm                 = true -> null
        }
    }

Plan: 1 to add, 0 to change, 1 to destroy.
╷
│ Warning: Version constraints inside provider configuration blocks are deprecated
│
│   on main.tf line 9, in provider "google":
│    9:   version = "3.5.0"
│
│ Terraform 0.13 and earlier allowed provider version constraints inside the provider configuration block, but that is now deprecated and will be removed in a future version of
│ Terraform. To silence this warning, move the provider version constraint into the required_providers block.
╵

Do you want to perform these actions?
  Terraform will perform the actions described above.
  Only 'yes' will be accepted to approve.

  Enter a value: yes

google_compute_instance.vm_instance: Destroying... [id=projects/qwiklabs-gcp-01-298144853b0c/zones/us-central1-c/instances/terraform-instance]
google_compute_instance.vm_instance: Still destroying... [id=projects/qwiklabs-gcp-01-298144853b0c/z...entral1-c/instances/terraform-instance, 10s elapsed]
google_compute_instance.vm_instance: Still destroying... [id=projects/qwiklabs-gcp-01-298144853b0c/z...entral1-c/instances/terraform-instance, 20s elapsed]
google_compute_instance.vm_instance: Destruction complete after 21s
google_compute_instance.vm_instance: Creating...
google_compute_instance.vm_instance: Still creating... [10s elapsed]
google_compute_instance.vm_instance: Creation complete after 18s [id=projects/qwiklabs-gcp-01-298144853b0c/zones/us-central1-c/instances/terraform-instance]

Apply complete! Resources: 1 added, 0 changed, 1 destroyed.
student_01_b9934b2a0bae@cloudshell:~ (qwiklabs-gcp-01-298144853b0c)$
```

```sh
student_01_b9934b2a0bae@cloudshell:~ (qwiklabs-gcp-01-298144853b0c)$ terraform apply
google_compute_network.vpc_network: Refreshing state... [id=projects/qwiklabs-gcp-01-298144853b0c/global/networks/terraform-network]
google_compute_instance.vm_instance: Refreshing state... [id=projects/qwiklabs-gcp-01-298144853b0c/zones/us-central1-c/instances/terraform-instance]

Terraform used the selected providers to generate the following execution plan. Resource actions are indicated with the following symbols:
-/+ destroy and then create replacement

Terraform will perform the following actions:

  # google_compute_instance.vm_instance must be replaced
-/+ resource "google_compute_instance" "vm_instance" {
      ~ cpu_platform         = "Intel Haswell" -> (known after apply)
      - enable_display       = false -> null
      ~ guest_accelerator    = [] -> (known after apply)
      ~ id                   = "projects/qwiklabs-gcp-01-298144853b0c/zones/us-central1-c/instances/terraform-instance" -> (known after apply)
      ~ instance_id          = "7236863538611822261" -> (known after apply)
      ~ label_fingerprint    = "42WmSpB8rSM=" -> (known after apply)
      - labels               = {} -> null
      - metadata             = {} -> null
      ~ metadata_fingerprint = "-IVbTFitpCY=" -> (known after apply)
      + min_cpu_platform     = (known after apply)
        name                 = "terraform-instance"
      ~ project              = "qwiklabs-gcp-01-298144853b0c" -> (known after apply)
      ~ self_link            = "https://www.googleapis.com/compute/v1/projects/qwiklabs-gcp-01-298144853b0c/zones/us-central1-c/instances/terraform-instance" -> (known after apply)
        tags                 = [
            "dev",
            "web",
        ]
      ~ tags_fingerprint     = "XaeQnaHMn9Y=" -> (known after apply)
      ~ zone                 = "us-central1-c" -> (known after apply)
        # (3 unchanged attributes hidden)

      ~ boot_disk {
          ~ device_name                = "persistent-disk-0" -> (known after apply)
          + disk_encryption_key_sha256 = (known after apply)
          + kms_key_self_link          = (known after apply)
          ~ source                     = "https://www.googleapis.com/compute/v1/projects/qwiklabs-gcp-01-298144853b0c/zones/us-central1-c/disks/terraform-instance" -> (known after apply)
            # (2 unchanged attributes hidden)

          ~ initialize_params {
              ~ image  = "https://www.googleapis.com/compute/v1/projects/debian-cloud/global/images/debian-11-bullseye-v20230411" -> "cos-cloud/cos-stable" # forces replacement
              ~ labels = {} -> (known after apply)
              ~ size   = 10 -> (known after apply)
              ~ type   = "pd-standard" -> (known after apply)
            }
        }

      ~ network_interface {
          ~ name               = "nic0" -> (known after apply)
          ~ network            = "https://www.googleapis.com/compute/v1/projects/qwiklabs-gcp-01-298144853b0c/global/networks/terraform-network" -> "terraform-network"
          ~ network_ip         = "10.128.0.2" -> (known after apply)
          ~ subnetwork         = "https://www.googleapis.com/compute/v1/projects/qwiklabs-gcp-01-298144853b0c/regions/us-central1/subnetworks/terraform-network" -> (known after apply)
          ~ subnetwork_project = "qwiklabs-gcp-01-298144853b0c" -> (known after apply)

          ~ access_config {
              ~ nat_ip       = "35.226.140.52" -> (known after apply)
              ~ network_tier = "PREMIUM" -> (known after apply)
            }
        }

      - scheduling {
          - automatic_restart   = true -> null
          - on_host_maintenance = "MIGRATE" -> null
          - preemptible         = false -> null
        }

      - shielded_instance_config {
          - enable_integrity_monitoring = true -> null
          - enable_secure_boot          = false -> null
          - enable_vtpm                 = true -> null
        }
    }

Plan: 1 to add, 0 to change, 1 to destroy.
╷
│ Warning: Version constraints inside provider configuration blocks are deprecated
│
│   on main.tf line 9, in provider "google":
│    9:   version = "3.5.0"
│
│ Terraform 0.13 and earlier allowed provider version constraints inside the provider configuration block, but that is now deprecated and will be removed in a future version of
│ Terraform. To silence this warning, move the provider version constraint into the required_providers block.
╵

Do you want to perform these actions?
  Terraform will perform the actions described above.
  Only 'yes' will be accepted to approve.

  Enter a value: yes

google_compute_instance.vm_instance: Destroying... [id=projects/qwiklabs-gcp-01-298144853b0c/zones/us-central1-c/instances/terraform-instance]
google_compute_instance.vm_instance: Still destroying... [id=projects/qwiklabs-gcp-01-298144853b0c/z...entral1-c/instances/terraform-instance, 10s elapsed]
google_compute_instance.vm_instance: Still destroying... [id=projects/qwiklabs-gcp-01-298144853b0c/z...entral1-c/instances/terraform-instance, 20s elapsed]
google_compute_instance.vm_instance: Destruction complete after 21s
google_compute_instance.vm_instance: Creating...
google_compute_instance.vm_instance: Still creating... [10s elapsed]
google_compute_instance.vm_instance: Creation complete after 18s [id=projects/qwiklabs-gcp-01-298144853b0c/zones/us-central1-c/instances/terraform-instance]

Apply complete! Resources: 1 added, 0 changed, 1 destroyed.
student_01_b9934b2a0bae@cloudshell:~ (qwiklabs-gcp-01-298144853b0c)$
```

### Destroy infrastructure

You have now seen how to build and change infrastructure. Before  moving on to creating multiple resources and showing resource  dependencies, you will see how to completely destroy your  Terraform-managed infrastructure.

Destroying your infrastructure is a rare event in production  environments. But if you're using Terraform to spin up multiple  environments such as development, testing, and staging, then destroying  is often a useful action.

Resources can be destroyed using the `terraform destroy` command, which is similar to `terraform apply` but it behaves as if all of the resources have been removed from the configuration.

- Try the `terraform destroy` command. Answer `yes` to execute this plan and destroy the infrastructure:

```sh
student_01_b9934b2a0bae@cloudshell:~ (qwiklabs-gcp-01-298144853b0c)$ terraform destroy
google_compute_network.vpc_network: Refreshing state... [id=projects/qwiklabs-gcp-01-298144853b0c/global/networks/terraform-network]
google_compute_instance.vm_instance: Refreshing state... [id=projects/qwiklabs-gcp-01-298144853b0c/zones/us-central1-c/instances/terraform-instance]

Terraform used the selected providers to generate the following execution plan. Resource actions are indicated with the following symbols:
  - destroy

Terraform will perform the following actions:

  # google_compute_instance.vm_instance will be destroyed
  - resource "google_compute_instance" "vm_instance" {
      - can_ip_forward       = false -> null
      - cpu_platform         = "Intel Haswell" -> null
      - deletion_protection  = false -> null
      - enable_display       = false -> null
      - guest_accelerator    = [] -> null
      - id                   = "projects/qwiklabs-gcp-01-298144853b0c/zones/us-central1-c/instances/terraform-instance" -> null
      - instance_id          = "3222647942534864683" -> null
      - label_fingerprint    = "42WmSpB8rSM=" -> null
      - labels               = {} -> null
      - machine_type         = "f1-micro" -> null
      - metadata             = {} -> null
      - metadata_fingerprint = "-IVbTFitpCY=" -> null
      - name                 = "terraform-instance" -> null
      - project              = "qwiklabs-gcp-01-298144853b0c" -> null
      - self_link            = "https://www.googleapis.com/compute/v1/projects/qwiklabs-gcp-01-298144853b0c/zones/us-central1-c/instances/terraform-instance" -> null
      - tags                 = [
          - "dev",
          - "web",
        ] -> null
      - tags_fingerprint     = "XaeQnaHMn9Y=" -> null
      - zone                 = "us-central1-c" -> null

      - boot_disk {
          - auto_delete = true -> null
          - device_name = "persistent-disk-0" -> null
          - mode        = "READ_WRITE" -> null
          - source      = "https://www.googleapis.com/compute/v1/projects/qwiklabs-gcp-01-298144853b0c/zones/us-central1-c/disks/terraform-instance" -> null

          - initialize_params {
              - image  = "https://www.googleapis.com/compute/v1/projects/cos-cloud/global/images/cos-stable-105-17412-1-66" -> null
              - labels = {} -> null
              - size   = 10 -> null
              - type   = "pd-standard" -> null
            }
        }

      - network_interface {
          - name               = "nic0" -> null
          - network            = "https://www.googleapis.com/compute/v1/projects/qwiklabs-gcp-01-298144853b0c/global/networks/terraform-network" -> null
          - network_ip         = "10.128.0.3" -> null
          - subnetwork         = "https://www.googleapis.com/compute/v1/projects/qwiklabs-gcp-01-298144853b0c/regions/us-central1/subnetworks/terraform-network" -> null
          - subnetwork_project = "qwiklabs-gcp-01-298144853b0c" -> null

          - access_config {
              - nat_ip       = "35.226.140.52" -> null
              - network_tier = "PREMIUM" -> null
            }
        }

      - scheduling {
          - automatic_restart   = true -> null
          - on_host_maintenance = "MIGRATE" -> null
          - preemptible         = false -> null
        }

      - shielded_instance_config {
          - enable_integrity_monitoring = true -> null
          - enable_secure_boot          = false -> null
          - enable_vtpm                 = true -> null
        }
    }

  # google_compute_network.vpc_network will be destroyed
  - resource "google_compute_network" "vpc_network" {
      - auto_create_subnetworks         = true -> null
      - delete_default_routes_on_create = false -> null
      - id                              = "projects/qwiklabs-gcp-01-298144853b0c/global/networks/terraform-network" -> null
      - name                            = "terraform-network" -> null
      - project                         = "qwiklabs-gcp-01-298144853b0c" -> null
      - routing_mode                    = "REGIONAL" -> null
      - self_link                       = "https://www.googleapis.com/compute/v1/projects/qwiklabs-gcp-01-298144853b0c/global/networks/terraform-network" -> null
    }

Plan: 0 to add, 0 to change, 2 to destroy.
╷
│ Warning: Version constraints inside provider configuration blocks are deprecated
│
│   on main.tf line 9, in provider "google":
│    9:   version = "3.5.0"
│
│ Terraform 0.13 and earlier allowed provider version constraints inside the provider configuration block, but that is now deprecated and will be removed in a future version of
│ Terraform. To silence this warning, move the provider version constraint into the required_providers block.
╵

Do you really want to destroy all resources?
  Terraform will destroy all your managed infrastructure, as shown above.
  There is no undo. Only 'yes' will be accepted to confirm.

  Enter a value: yes

google_compute_instance.vm_instance: Destroying... [id=projects/qwiklabs-gcp-01-298144853b0c/zones/us-central1-c/instances/terraform-instance]
google_compute_instance.vm_instance: Still destroying... [id=projects/qwiklabs-gcp-01-298144853b0c/z...entral1-c/instances/terraform-instance, 10s elapsed]
google_compute_instance.vm_instance: Destruction complete after 20s
google_compute_network.vpc_network: Destroying... [id=projects/qwiklabs-gcp-01-298144853b0c/global/networks/terraform-network]
google_compute_network.vpc_network: Still destroying... [id=projects/qwiklabs-gcp-01-298144853b0c/global/networks/terraform-network, 10s elapsed]
google_compute_network.vpc_network: Still destroying... [id=projects/qwiklabs-gcp-01-298144853b0c/global/networks/terraform-network, 20s elapsed]
google_compute_network.vpc_network: Still destroying... [id=projects/qwiklabs-gcp-01-298144853b0c/global/networks/terraform-network, 30s elapsed]
google_compute_network.vpc_network: Still destroying... [id=projects/qwiklabs-gcp-01-298144853b0c/global/networks/terraform-network, 40s elapsed]
google_compute_network.vpc_network: Destruction complete after 47s

Destroy complete! Resources: 2 destroyed.
student_01_b9934b2a0bae@cloudshell:~ (qwiklabs-gcp-01-298144853b0c)$
```

Just like with `terraform apply`, Terraform determines the  order in which things must be destroyed. Google Cloud won't allow a VPC  network to be deleted if there are resources still in it, so Terraform  waits until the instance is destroyed before destroying the network.  When performing operations, Terraform creates a dependency graph to  determine the correct order of operations. In more complicated cases  with multiple resources, Terraform will perform operations in parallel  when it's safe to do so.

## Create resource dependencies

In this section, you will learn more about resource dependencies and  how to use resource parameters to share information about one resource  with other resources.

Real-world infrastructure has a diverse set of resources and resource types. Terraform configurations can contain multiple resources,  multiple resource types, and these types can even span multiple  providers.

In this section, you will be shown a basic example of how to  configure multiple resources and how to use resource attributes to  configure other resources.

- Recreate your network and instance. After you respond to the prompt with `yes`, the resources will be created:

```sh
student_01_b9934b2a0bae@cloudshell:~ (qwiklabs-gcp-01-298144853b0c)$ terraform apply

Terraform used the selected providers to generate the following execution plan. Resource actions are indicated with the following symbols:
  + create

Terraform will perform the following actions:

  # google_compute_instance.vm_instance will be created
  + resource "google_compute_instance" "vm_instance" {
      + can_ip_forward       = false
      + cpu_platform         = (known after apply)
      + deletion_protection  = false
      + guest_accelerator    = (known after apply)
      + id                   = (known after apply)
      + instance_id          = (known after apply)
      + label_fingerprint    = (known after apply)
      + machine_type         = "f1-micro"
      + metadata_fingerprint = (known after apply)
      + min_cpu_platform     = (known after apply)
      + name                 = "terraform-instance"
      + project              = (known after apply)
      + self_link            = (known after apply)
      + tags                 = [
          + "dev",
          + "web",
        ]
      + tags_fingerprint     = (known after apply)
      + zone                 = (known after apply)

      + boot_disk {
          + auto_delete                = true
          + device_name                = (known after apply)
          + disk_encryption_key_sha256 = (known after apply)
          + kms_key_self_link          = (known after apply)
          + mode                       = "READ_WRITE"
          + source                     = (known after apply)

          + initialize_params {
              + image  = "cos-cloud/cos-stable"
              + labels = (known after apply)
              + size   = (known after apply)
              + type   = (known after apply)
            }
        }

      + network_interface {
          + name               = (known after apply)
          + network            = "terraform-network"
          + network_ip         = (known after apply)
          + subnetwork         = (known after apply)
          + subnetwork_project = (known after apply)

          + access_config {
              + nat_ip       = (known after apply)
              + network_tier = (known after apply)
            }
        }
    }

  # google_compute_network.vpc_network will be created
  + resource "google_compute_network" "vpc_network" {
      + auto_create_subnetworks         = true
      + delete_default_routes_on_create = false
      + gateway_ipv4                    = (known after apply)
      + id                              = (known after apply)
      + ipv4_range                      = (known after apply)
      + name                            = "terraform-network"
      + project                         = (known after apply)
      + routing_mode                    = (known after apply)
      + self_link                       = (known after apply)
    }

Plan: 2 to add, 0 to change, 0 to destroy.
╷
│ Warning: Version constraints inside provider configuration blocks are deprecated
│
│   on main.tf line 9, in provider "google":
│    9:   version = "3.5.0"
│
│ Terraform 0.13 and earlier allowed provider version constraints inside the provider configuration block, but that is now deprecated and will be removed in a future version of
│ Terraform. To silence this warning, move the provider version constraint into the required_providers block.
╵

Do you want to perform these actions?
  Terraform will perform the actions described above.
  Only 'yes' will be accepted to approve.

  Enter a value: yes

google_compute_network.vpc_network: Creating...
google_compute_network.vpc_network: Still creating... [10s elapsed]
google_compute_network.vpc_network: Still creating... [20s elapsed]
google_compute_network.vpc_network: Still creating... [30s elapsed]
google_compute_network.vpc_network: Creation complete after 37s [id=projects/qwiklabs-gcp-01-298144853b0c/global/networks/terraform-network]
google_compute_instance.vm_instance: Creating...
google_compute_instance.vm_instance: Still creating... [10s elapsed]
google_compute_instance.vm_instance: Creation complete after 16s [id=projects/qwiklabs-gcp-01-298144853b0c/zones/us-central1-c/instances/terraform-instance]

Apply complete! Resources: 2 added, 0 changed, 0 destroyed.
student_01_b9934b2a0bae@cloudshell:~ (qwiklabs-gcp-01-298144853b0c)$
```

### Assigning a static IP address

1. Now add to your configuration by assigning a static IP to the VM instance in `main.tf`:

```sh
student_01_b9934b2a0bae@cloudshell:~ (qwiklabs-gcp-01-298144853b0c)$ cat main.tf
terraform {
  required_providers {
    google = {
      source = "hashicorp/google"
    }
  }
}
provider "google" {
  version = "3.5.0"
  project = "qwiklabs-gcp-01-298144853b0c"
  region  = "us-central1"
  zone    = "us-central1-c"
}
resource "google_compute_network" "vpc_network" {
  name = "terraform-network"
}
resource "google_compute_instance" "vm_instance" {
  name         = "terraform-instance"
  machine_type = "f1-micro"
  tags         = ["web", "dev"]
  boot_disk {
    initialize_params {
      image = "cos-cloud/cos-stable"
    }
  }

  network_interface {
    network = google_compute_network.vpc_network.name
    access_config {
    }
  }
}

resource "google_compute_address" "vm_static_ip" {
  name = "terraform-static-ip"
}student_01_b9934b2a0bae@cloudshell:~ (qwiklabs-gcp-01-298144853b0c)$
```

```sh
student_01_b9934b2a0bae@cloudshell:~ (qwiklabs-gcp-01-298144853b0c)$ terraform plan
google_compute_network.vpc_network: Refreshing state... [id=projects/qwiklabs-gcp-01-298144853b0c/global/networks/terraform-network]
google_compute_instance.vm_instance: Refreshing state... [id=projects/qwiklabs-gcp-01-298144853b0c/zones/us-central1-c/instances/terraform-instance]

Terraform used the selected providers to generate the following execution plan. Resource actions are indicated with the following symbols:
  + create

Terraform will perform the following actions:

  # google_compute_address.vm_static_ip will be created
  + resource "google_compute_address" "vm_static_ip" {
      + address            = (known after apply)
      + address_type       = "EXTERNAL"
      + creation_timestamp = (known after apply)
      + id                 = (known after apply)
      + name               = "terraform-static-ip"
      + network_tier       = (known after apply)
      + project            = (known after apply)
      + purpose            = (known after apply)
      + region             = (known after apply)
      + self_link          = (known after apply)
      + subnetwork         = (known after apply)
      + users              = (known after apply)
    }

Plan: 1 to add, 0 to change, 0 to destroy.
╷
│ Warning: Version constraints inside provider configuration blocks are deprecated
│
│   on main.tf line 9, in provider "google":
│    9:   version = "3.5.0"
│
│ Terraform 0.13 and earlier allowed provider version constraints inside the provider configuration block, but that is now deprecated and will be removed in a future version of
│ Terraform. To silence this warning, move the provider version constraint into the required_providers block.
╵

───────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────

Note: You didn't use the -out option to save this plan, so Terraform can't guarantee to take exactly these actions if you run "terraform apply" now.
student_01_b9934b2a0bae@cloudshell:~ (qwiklabs-gcp-01-298144853b0c)$
```

Unlike `terraform apply`, the `plan` command  will only show what would be changed, and never actually apply the  changes directly. Notice that the only change you have made so far is to add a static IP. Next, you need to attach the IP address to your  instance.

1. Update the `network_interface` configuration for your instance like so:

```sh
student_01_b9934b2a0bae@cloudshell:~ (qwiklabs-gcp-01-298144853b0c)$ cat main.tf
terraform {
  required_providers {
    google = {
      source = "hashicorp/google"
    }
  }
}
provider "google" {
  version = "3.5.0"
  project = "qwiklabs-gcp-01-298144853b0c"
  region  = "us-central1"
  zone    = "us-central1-c"
}
resource "google_compute_network" "vpc_network" {
  name = "terraform-network"
}
resource "google_compute_instance" "vm_instance" {
  name         = "terraform-instance"
  machine_type = "f1-micro"
  tags         = ["web", "dev"]
  boot_disk {
    initialize_params {
      image = "cos-cloud/cos-stable"
    }
  }

  network_interface {
    network = google_compute_network.vpc_network.self_link
    access_config {
      nat_ip = google_compute_address.vm_static_ip.address
    }
  }
}

resource "google_compute_address" "vm_static_ip" {
  name = "terraform-static-ip"
}student_01_b9934b2a0bae@cloudshell:~ (qwiklabs-gcp-01-298144853b0c)$
```

```sh
student_01_b9934b2a0bae@cloudshell:~ (qwiklabs-gcp-01-298144853b0c)$ terraform plan -out static_ip
google_compute_network.vpc_network: Refreshing state... [id=projects/qwiklabs-gcp-01-298144853b0c/global/networks/terraform-network]
google_compute_instance.vm_instance: Refreshing state... [id=projects/qwiklabs-gcp-01-298144853b0c/zones/us-central1-c/instances/terraform-instance]

Terraform used the selected providers to generate the following execution plan. Resource actions are indicated with the following symbols:
  + create
  ~ update in-place

Terraform will perform the following actions:

  # google_compute_address.vm_static_ip will be created
  + resource "google_compute_address" "vm_static_ip" {
      + address            = (known after apply)
      + address_type       = "EXTERNAL"
      + creation_timestamp = (known after apply)
      + id                 = (known after apply)
      + name               = "terraform-static-ip"
      + network_tier       = (known after apply)
      + project            = (known after apply)
      + purpose            = (known after apply)
      + region             = (known after apply)
      + self_link          = (known after apply)
      + subnetwork         = (known after apply)
      + users              = (known after apply)
    }

  # google_compute_instance.vm_instance will be updated in-place
  ~ resource "google_compute_instance" "vm_instance" {
        id                   = "projects/qwiklabs-gcp-01-298144853b0c/zones/us-central1-c/instances/terraform-instance"
        name                 = "terraform-instance"
        tags                 = [
            "dev",
            "web",
        ]
        # (15 unchanged attributes hidden)

      ~ network_interface {
            name               = "nic0"
            # (4 unchanged attributes hidden)

          ~ access_config {
              ~ nat_ip       = "34.173.2.82" -> (known after apply)
                # (1 unchanged attribute hidden)
            }
        }

        # (3 unchanged blocks hidden)
    }

Plan: 1 to add, 1 to change, 0 to destroy.
╷
│ Warning: Version constraints inside provider configuration blocks are deprecated
│
│   on main.tf line 9, in provider "google":
│    9:   version = "3.5.0"
│
│ Terraform 0.13 and earlier allowed provider version constraints inside the provider configuration block, but that is now deprecated and will be removed in a future version of
│ Terraform. To silence this warning, move the provider version constraint into the required_providers block.
╵

───────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────

Saved the plan to: static_ip

To perform exactly these actions, run the following command to apply:
    terraform apply "static_ip"
student_01_b9934b2a0bae@cloudshell:~ (qwiklabs-gcp-01-298144853b0c)$
```

Saving the plan this way ensures that you can apply exactly the same  plan in the future. If you try to apply the file created by the plan,  Terraform will first check to make sure the exact same set of changes  will be made before applying the plan.

In this case, you can see that Terraform will create a new `google_compute_address` and update the existing VM to use it.

1. Run `terraform apply "static_ip"` to see how Terraform plans to apply this change:

```sh
student_01_b9934b2a0bae@cloudshell:~ (qwiklabs-gcp-01-298144853b0c)$ terraform apply "static_ip"
google_compute_address.vm_static_ip: Creating...
google_compute_address.vm_static_ip: Creation complete after 7s [id=projects/qwiklabs-gcp-01-298144853b0c/regions/us-central1/addresses/terraform-static-ip]
google_compute_instance.vm_instance: Modifying... [id=projects/qwiklabs-gcp-01-298144853b0c/zones/us-central1-c/instances/terraform-instance]
google_compute_instance.vm_instance: Still modifying... [id=projects/qwiklabs-gcp-01-298144853b0c/z...entral1-c/instances/terraform-instance, 10s elapsed]
google_compute_instance.vm_instance: Still modifying... [id=projects/qwiklabs-gcp-01-298144853b0c/z...entral1-c/instances/terraform-instance, 20s elapsed]
google_compute_instance.vm_instance: Modifications complete after 27s [id=projects/qwiklabs-gcp-01-298144853b0c/zones/us-central1-c/instances/terraform-instance]
╷
│ Warning: Version constraints inside provider configuration blocks are deprecated
│
│   on main.tf line 9, in provider "google":
│    9:   version = "3.5.0"
│
│ Terraform 0.13 and earlier allowed provider version constraints inside the provider configuration block, but that is now deprecated and will be removed in a future version of
│ Terraform. To silence this warning, move the provider version constraint into the required_providers block.
╵

Apply complete! Resources: 1 added, 1 changed, 0 destroyed.
student_01_b9934b2a0bae@cloudshell:~ (qwiklabs-gcp-01-298144853b0c)$
```

As shown above, Terraform created the static IP before modifying the VM  instance. Due to the interpolation expression that passes the IP address to the instance's network interface configuration, Terraform is able to infer a dependency, and knows it must create the static IP before  updating the instance.



### Implicit and explicit dependencies

By studying the resource attributes used in interpolation  expressions, Terraform can automatically infer when one resource depends on another. In the example above, the reference to `google_compute_address.vm_static_ip.address` creates an *implicit dependency* on the `google_compute_address` named `vm_static_ip`.

Terraform uses this dependency information to determine the correct  order in which to create and update different resources. In the example  above, Terraform knows that the `vm_static_ip` must be created before the `vm_instance` is updated to use it.

Implicit dependencies via interpolation expressions are the primary  way to inform Terraform about these relationships, and should be used  whenever possible.

Sometimes there are dependencies between resources that are *not* visible to Terraform. The `depends_on` argument can be added to any resource and accepts a list of resources to create explicit dependencies for.

For example, perhaps an application you will run on your instance  expects to use a specific Cloud Storage bucket, but that dependency is  configured inside the application code and thus not visible to  Terraform. In that case, you can use `depends_on` to explicitly declare the dependency.



Add a Cloud Storage bucket and an instance with an explicit dependency on the bucket by adding the following to `main.tf`:

```sh
student_01_b9934b2a0bae@cloudshell:~ (qwiklabs-gcp-01-298144853b0c)$ cat main.tf
terraform {
  required_providers {
    google = {
      source = "hashicorp/google"
    }
  }
}
provider "google" {
  version = "3.5.0"
  project = "qwiklabs-gcp-01-298144853b0c"
  region  = "us-central1"
  zone    = "us-central1-c"
}
resource "google_compute_network" "vpc_network" {
  name = "terraform-network"
}
resource "google_compute_instance" "vm_instance" {
  name         = "terraform-instance"
  machine_type = "f1-micro"
  tags         = ["web", "dev"]
  boot_disk {
    initialize_params {
      image = "cos-cloud/cos-stable"
    }
  }

  network_interface {
    network = google_compute_network.vpc_network.self_link
    access_config {
      nat_ip = google_compute_address.vm_static_ip.address
    }
  }
}

resource "google_compute_address" "vm_static_ip" {
  name = "terraform-static-ip"
}
# New resource for the storage bucket our application will use.
resource "google_storage_bucket" "example_bucket" {
  name     = "pradeepgadde"
  location = "US"
  website {
    main_page_suffix = "index.html"
    not_found_page   = "404.html"
  }
}
# Create a new instance that uses the bucket
resource "google_compute_instance" "another_instance" {
  # Tells Terraform that this VM instance must be created only after the
  # storage bucket has been created.
  depends_on = [google_storage_bucket.example_bucket]
  name         = "terraform-instance-2"
  machine_type = "f1-micro"
  boot_disk {
    initialize_params {
      image = "cos-cloud/cos-stable"
    }
  }
  network_interface {
    network = google_compute_network.vpc_network.self_link
    access_config {
    }
  }
}student_01_b9934b2a0bae@cloudshell:~ (qwiklabs-gcp-01-298144853b0c)$
```

The order that resources are defined in a terraform configuration file  has no effect on how Terraform applies your changes. Organize your  configuration files in a way that makes the most sense for you and your  team.

```sh
student_01_b9934b2a0bae@cloudshell:~ (qwiklabs-gcp-01-298144853b0c)$ terraform plann
terraform apply
google_compute_network.vpc_network: Refreshing state... [id=projects/qwiklabs-gcp-01-298144853b0c/global/networks/terraform-network]
google_compute_address.vm_static_ip: Refreshing state... [id=projects/qwiklabs-gcp-01-298144853b0c/regions/us-central1/addresses/terraform-static-ip]
google_compute_instance.vm_instance: Refreshing state... [id=projects/qwiklabs-gcp-01-298144853b0c/zones/us-central1-c/instances/terraform-instance]

Terraform used the selected providers to generate the following execution plan. Resource actions are indicated with the following symbols:
  + create

Terraform will perform the following actions:

  # google_compute_instance.another_instance will be created
  + resource "google_compute_instance" "another_instance" {
      + can_ip_forward       = false
      + cpu_platform         = (known after apply)
      + deletion_protection  = false
      + guest_accelerator    = (known after apply)
      + id                   = (known after apply)
      + instance_id          = (known after apply)
      + label_fingerprint    = (known after apply)
      + machine_type         = "f1-micro"
      + metadata_fingerprint = (known after apply)
      + min_cpu_platform     = (known after apply)
      + name                 = "terraform-instance-2"
      + project              = (known after apply)
      + self_link            = (known after apply)
      + tags_fingerprint     = (known after apply)
      + zone                 = (known after apply)

      + boot_disk {
          + auto_delete                = true
          + device_name                = (known after apply)
          + disk_encryption_key_sha256 = (known after apply)
          + kms_key_self_link          = (known after apply)
          + mode                       = "READ_WRITE"
          + source                     = (known after apply)

          + initialize_params {
              + image  = "cos-cloud/cos-stable"
              + labels = (known after apply)
              + size   = (known after apply)
              + type   = (known after apply)
            }
        }

      + network_interface {
          + name               = (known after apply)
          + network            = "https://www.googleapis.com/compute/v1/projects/qwiklabs-gcp-01-298144853b0c/global/networks/terraform-network"
          + network_ip         = (known after apply)
          + subnetwork         = (known after apply)
          + subnetwork_project = (known after apply)

          + access_config {
              + nat_ip       = (known after apply)
              + network_tier = (known after apply)
            }
        }
    }

  # google_storage_bucket.example_bucket will be created
  + resource "google_storage_bucket" "example_bucket" {
      + bucket_policy_only = (known after apply)
      + force_destroy      = false
      + id                 = (known after apply)
      + location           = "US"
      + name               = "pradeepgadde"
      + project            = (known after apply)
      + self_link          = (known after apply)
      + storage_class      = "STANDARD"
      + url                = (known after apply)

      + website {
          + main_page_suffix = "index.html"
          + not_found_page   = "404.html"
        }
    }

Plan: 2 to add, 0 to change, 0 to destroy.
╷
│ Warning: Version constraints inside provider configuration blocks are deprecated
│
│   on main.tf line 9, in provider "google":
│    9:   version = "3.5.0"
│
│ Terraform 0.13 and earlier allowed provider version constraints inside the provider configuration block, but that is now deprecated and will be removed in a future version of
│ Terraform. To silence this warning, move the provider version constraint into the required_providers block.
╵

───────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────

Note: You didn't use the -out option to save this plan, so Terraform can't guarantee to take exactly these actions if you run "terraform apply" now.
google_compute_address.vm_static_ip: Refreshing state... [id=projects/qwiklabs-gcp-01-298144853b0c/regions/us-central1/addresses/terraform-static-ip]
google_compute_network.vpc_network: Refreshing state... [id=projects/qwiklabs-gcp-01-298144853b0c/global/networks/terraform-network]
google_compute_instance.vm_instance: Refreshing state... [id=projects/qwiklabs-gcp-01-298144853b0c/zones/us-central1-c/instances/terraform-instance]

Terraform used the selected providers to generate the following execution plan. Resource actions are indicated with the following symbols:
  + create

Terraform will perform the following actions:

  # google_compute_instance.another_instance will be created
  + resource "google_compute_instance" "another_instance" {
      + can_ip_forward       = false
      + cpu_platform         = (known after apply)
      + deletion_protection  = false
      + guest_accelerator    = (known after apply)
      + id                   = (known after apply)
      + instance_id          = (known after apply)
      + label_fingerprint    = (known after apply)
      + machine_type         = "f1-micro"
      + metadata_fingerprint = (known after apply)
      + min_cpu_platform     = (known after apply)
      + name                 = "terraform-instance-2"
      + project              = (known after apply)
      + self_link            = (known after apply)
      + tags_fingerprint     = (known after apply)
      + zone                 = (known after apply)

      + boot_disk {
          + auto_delete                = true
          + device_name                = (known after apply)
          + disk_encryption_key_sha256 = (known after apply)
          + kms_key_self_link          = (known after apply)
          + mode                       = "READ_WRITE"
          + source                     = (known after apply)

          + initialize_params {
              + image  = "cos-cloud/cos-stable"
              + labels = (known after apply)
              + size   = (known after apply)
              + type   = (known after apply)
            }
        }

      + network_interface {
          + name               = (known after apply)
          + network            = "https://www.googleapis.com/compute/v1/projects/qwiklabs-gcp-01-298144853b0c/global/networks/terraform-network"
          + network_ip         = (known after apply)
          + subnetwork         = (known after apply)
          + subnetwork_project = (known after apply)

          + access_config {
              + nat_ip       = (known after apply)
              + network_tier = (known after apply)
            }
        }
    }

  # google_storage_bucket.example_bucket will be created
  + resource "google_storage_bucket" "example_bucket" {
      + bucket_policy_only = (known after apply)
      + force_destroy      = false
      + id                 = (known after apply)
      + location           = "US"
      + name               = "pradeepgadde"
      + project            = (known after apply)
      + self_link          = (known after apply)
      + storage_class      = "STANDARD"
      + url                = (known after apply)

      + website {
          + main_page_suffix = "index.html"
          + not_found_page   = "404.html"
        }
    }

Plan: 2 to add, 0 to change, 0 to destroy.
╷
│ Warning: Version constraints inside provider configuration blocks are deprecated
│
│   on main.tf line 9, in provider "google":
│    9:   version = "3.5.0"
│
│ Terraform 0.13 and earlier allowed provider version constraints inside the provider configuration block, but that is now deprecated and will be removed in a future version of
│ Terraform. To silence this warning, move the provider version constraint into the required_providers block.
╵

Do you want to perform these actions?
  Terraform will perform the actions described above.
  Only 'yes' will be accepted to approve.

  Enter a value: yes

google_storage_bucket.example_bucket: Creating...
google_storage_bucket.example_bucket: Creation complete after 2s [id=pradeepgadde]
google_compute_instance.another_instance: Creating...
google_compute_instance.another_instance: Still creating... [10s elapsed]
google_compute_instance.another_instance: Creation complete after 16s [id=projects/qwiklabs-gcp-01-298144853b0c/zones/us-central1-c/instances/terraform-instance-2]

Apply complete! Resources: 2 added, 0 changed, 0 destroyed.
student_01_b9934b2a0bae@cloudshell:~ (qwiklabs-gcp-01-298144853b0c)$
```

![]({{ site.url }}{{ site.baseurl }}/assets/images/gcp-tf-iac-4.png)



Before moving on, remove these new resources from your configuration and run `terraform apply` once again to destroy them. You won't use the bucket or the second instance any further in the getting started guide.

```sh
student_01_b9934b2a0bae@cloudshell:~ (qwiklabs-gcp-01-298144853b0c)$ cat main.tf
terraform {
  required_providers {
    google = {
      source = "hashicorp/google"
    }
  }
}
provider "google" {
  version = "3.5.0"
  project = "qwiklabs-gcp-01-298144853b0c"
  region  = "us-central1"
  zone    = "us-central1-c"
}
resource "google_compute_network" "vpc_network" {
  name = "terraform-network"
}
resource "google_compute_instance" "vm_instance" {
  name         = "terraform-instance"
  machine_type = "f1-micro"
  tags         = ["web", "dev"]
  boot_disk {
    initialize_params {
      image = "cos-cloud/cos-stable"
    }
  }

  network_interface {
    network = google_compute_network.vpc_network.self_link
    access_config {
      nat_ip = google_compute_address.vm_static_ip.address
    }
  }
}

resource "google_compute_address" "vm_static_ip" {
  name = "terraform-static-ip"
}
# New resource for the storage bucket our application will use.
student_01_b9934b2a0bae@cloudshell:~ (qwiklabs-gcp-01-298144853b0c)$
```

```sh
student_01_b9934b2a0bae@cloudshell:~ (qwiklabs-gcp-01-298144853b0c)$ terraform apply
google_compute_instance.another_instance: Refreshing state... [id=projects/qwiklabs-gcp-01-298144853b0c/zones/us-central1-c/instances/terraform-instance-2]
google_compute_network.vpc_network: Refreshing state... [id=projects/qwiklabs-gcp-01-298144853b0c/global/networks/terraform-network]
google_compute_address.vm_static_ip: Refreshing state... [id=projects/qwiklabs-gcp-01-298144853b0c/regions/us-central1/addresses/terraform-static-ip]
google_storage_bucket.example_bucket: Refreshing state... [id=pradeepgadde]
google_compute_instance.vm_instance: Refreshing state... [id=projects/qwiklabs-gcp-01-298144853b0c/zones/us-central1-c/instances/terraform-instance]

Terraform used the selected providers to generate the following execution plan. Resource actions are indicated with the following symbols:
  - destroy

Terraform will perform the following actions:

  # google_compute_instance.another_instance will be destroyed
  # (because google_compute_instance.another_instance is not in configuration)
  - resource "google_compute_instance" "another_instance" {
      - can_ip_forward       = false -> null
      - cpu_platform         = "Intel Haswell" -> null
      - deletion_protection  = false -> null
      - enable_display       = false -> null
      - guest_accelerator    = [] -> null
      - id                   = "projects/qwiklabs-gcp-01-298144853b0c/zones/us-central1-c/instances/terraform-instance-2" -> null
      - instance_id          = "1799040118417202001" -> null
      - label_fingerprint    = "42WmSpB8rSM=" -> null
      - labels               = {} -> null
      - machine_type         = "f1-micro" -> null
      - metadata             = {} -> null
      - metadata_fingerprint = "-IVbTFitpCY=" -> null
      - name                 = "terraform-instance-2" -> null
      - project              = "qwiklabs-gcp-01-298144853b0c" -> null
      - self_link            = "https://www.googleapis.com/compute/v1/projects/qwiklabs-gcp-01-298144853b0c/zones/us-central1-c/instances/terraform-instance-2" -> null
      - tags                 = [] -> null
      - tags_fingerprint     = "42WmSpB8rSM=" -> null
      - zone                 = "us-central1-c" -> null

      - boot_disk {
          - auto_delete = true -> null
          - device_name = "persistent-disk-0" -> null
          - mode        = "READ_WRITE" -> null
          - source      = "https://www.googleapis.com/compute/v1/projects/qwiklabs-gcp-01-298144853b0c/zones/us-central1-c/disks/terraform-instance-2" -> null

          - initialize_params {
              - image  = "https://www.googleapis.com/compute/v1/projects/cos-cloud/global/images/cos-stable-105-17412-1-66" -> null
              - labels = {} -> null
              - size   = 10 -> null
              - type   = "pd-standard" -> null
            }
        }

      - network_interface {
          - name               = "nic0" -> null
          - network            = "https://www.googleapis.com/compute/v1/projects/qwiklabs-gcp-01-298144853b0c/global/networks/terraform-network" -> null
          - network_ip         = "10.128.0.3" -> null
          - subnetwork         = "https://www.googleapis.com/compute/v1/projects/qwiklabs-gcp-01-298144853b0c/regions/us-central1/subnetworks/terraform-network" -> null
          - subnetwork_project = "qwiklabs-gcp-01-298144853b0c" -> null

          - access_config {
              - nat_ip       = "34.173.2.82" -> null
              - network_tier = "PREMIUM" -> null
            }
        }

      - scheduling {
          - automatic_restart   = true -> null
          - on_host_maintenance = "MIGRATE" -> null
          - preemptible         = false -> null
        }

      - shielded_instance_config {
          - enable_integrity_monitoring = true -> null
          - enable_secure_boot          = false -> null
          - enable_vtpm                 = true -> null
        }
    }

  # google_storage_bucket.example_bucket will be destroyed
  # (because google_storage_bucket.example_bucket is not in configuration)
  - resource "google_storage_bucket" "example_bucket" {
      - bucket_policy_only = false -> null
      - force_destroy      = false -> null
      - id                 = "pradeepgadde" -> null
      - labels             = {} -> null
      - location           = "US" -> null
      - name               = "pradeepgadde" -> null
      - project            = "qwiklabs-gcp-01-298144853b0c" -> null
      - requester_pays     = false -> null
      - self_link          = "https://www.googleapis.com/storage/v1/b/pradeepgadde" -> null
      - storage_class      = "STANDARD" -> null
      - url                = "gs://pradeepgadde" -> null

      - website {
          - main_page_suffix = "index.html" -> null
          - not_found_page   = "404.html" -> null
        }
    }

Plan: 0 to add, 0 to change, 2 to destroy.
╷
│ Warning: Version constraints inside provider configuration blocks are deprecated
│
│   on main.tf line 9, in provider "google":
│    9:   version = "3.5.0"
│
│ Terraform 0.13 and earlier allowed provider version constraints inside the provider configuration block, but that is now deprecated and will be removed in a future version of
│ Terraform. To silence this warning, move the provider version constraint into the required_providers block.
╵

Do you want to perform these actions?
  Terraform will perform the actions described above.
  Only 'yes' will be accepted to approve.

  Enter a value: yes

google_compute_instance.another_instance: Destroying... [id=projects/qwiklabs-gcp-01-298144853b0c/zones/us-central1-c/instances/terraform-instance-2]
google_compute_instance.another_instance: Still destroying... [id=projects/qwiklabs-gcp-01-298144853b0c/z...tral1-c/instances/terraform-instance-2, 10s elapsed]
google_compute_instance.another_instance: Still destroying... [id=projects/qwiklabs-gcp-01-298144853b0c/z...tral1-c/instances/terraform-instance-2, 20s elapsed]
google_compute_instance.another_instance: Destruction complete after 22s
google_storage_bucket.example_bucket: Destroying... [id=pradeepgadde]
google_storage_bucket.example_bucket: Destruction complete after 1s

Apply complete! Resources: 0 added, 0 changed, 2 destroyed.
student_01_b9934b2a0bae@cloudshell:~ (qwiklabs-gcp-01-298144853b0c)$
```

## Provision infrastructure

The compute instance you launched at this point is based on the  Google image given, but has no additional software installed or  configuration applied.

Google Cloud allows customers to manage their own [custom operating system images](https://cloud.google.com/compute/docs/images/create-delete-deprecate-private-images). This can be a great way to ensure the instances you provision with Terraform are pre-configured based on your needs. [Packer](https://www.packer.io/) is the perfect tool for this and includes a [builder for Google Cloud](https://www.packer.io/docs/builders/googlecompute.html).

Terraform uses provisioners to upload files, run shell scripts, or  install and trigger other software like configuration management tools.



### Defining a provisioner

1. To define a provisioner, modify the resource block defining the first `vm_instance` in your configuration to look like the following:

```sh
student_01_b9934b2a0bae@cloudshell:~ (qwiklabs-gcp-01-298144853b0c)$ cat main.tf
terraform {
  required_providers {
    google = {
      source = "hashicorp/google"
    }
  }
}
provider "google" {
  version = "3.5.0"
  project = "qwiklabs-gcp-01-298144853b0c"
  region  = "us-central1"
  zone    = "us-central1-c"
}
resource "google_compute_network" "vpc_network" {
  name = "terraform-network"
}
resource "google_compute_instance" "vm_instance" {
  name         = "terraform-instance"
  machine_type = "f1-micro"
  tags         = ["web", "dev"]
  provisioner "local-exec" {
    command = "echo ${google_compute_instance.vm_instance.name}:  ${google_compute_instance.vm_instance.network_interface[0].access_config[0].nat_ip} >> ip_address.txt"
  }
  boot_disk {
    initialize_params {
      image = "cos-cloud/cos-stable"
    }
  }

  network_interface {
    network = google_compute_network.vpc_network.self_link
    access_config {
      nat_ip = google_compute_address.vm_static_ip.address
    }
  }
}

resource "google_compute_address" "vm_static_ip" {
  name = "terraform-static-ip"
}
# New resource for the storage bucket our application will use.
student_01_b9934b2a0bae@cloudshell:~ (qwiklabs-gcp-01-298144853b0c)$
```

This adds a `provisioner` block within the `resource` block. Multiple `provisioner` blocks can be added to define multiple provisioning steps. Terraform supports [many provisioners](https://www.terraform.io/docs/provisioners/index.html), but for this example you are using the `local-exec` provisioner.

The `local-exec` provisioner executes a command locally on the machine running Terraform, not the VM instance itself. You're using this provisioner versus the others so we don't have to worry about  specifying any [connection info](https://www.terraform.io/docs/provisioners/connection.html) right now.

This also shows a more complex example of string interpolation than  you've seen before. Each VM instance can have multiple network  interfaces, so refer to the first one with `network_interface[0]`, count starting from 0, as most programming languages do. Each network  interface can have multiple access_config blocks as well, so once again  you specify the first one.



```sh
student_01_b9934b2a0bae@cloudshell:~ (qwiklabs-gcp-01-298144853b0c)$ terraform apply
google_compute_address.vm_static_ip: Refreshing state... [id=projects/qwiklabs-gcp-01-298144853b0c/regions/us-central1/addresses/terraform-static-ip]
google_compute_network.vpc_network: Refreshing state... [id=projects/qwiklabs-gcp-01-298144853b0c/global/networks/terraform-network]
google_compute_instance.vm_instance: Refreshing state... [id=projects/qwiklabs-gcp-01-298144853b0c/zones/us-central1-c/instances/terraform-instance]

No changes. Your infrastructure matches the configuration.

Terraform has compared your real infrastructure against your configuration and found no differences, so no changes are needed.
╷
│ Warning: Version constraints inside provider configuration blocks are deprecated
│
│   on main.tf line 9, in provider "google":
│    9:   version = "3.5.0"
│
│ Terraform 0.13 and earlier allowed provider version constraints inside the provider configuration block, but that is now deprecated and will be removed in a future version of
│ Terraform. To silence this warning, move the provider version constraint into the required_providers block.
╵

Apply complete! Resources: 0 added, 0 changed, 0 destroyed.
student_01_b9934b2a0bae@cloudshell:~ (qwiklabs-gcp-01-298144853b0c)$
```

At this point, the output may be confusing at first.

Terraform found nothing to do - and if you check, you'll find that there's no `ip_address.txt` file on your local machine.

Terraform treats provisioners differently from other arguments.  Provisioners only run when a resource is created, but adding a  provisioner does not force that resource to be destroyed and recreated.



```sh
student_01_b9934b2a0bae@cloudshell:~ (qwiklabs-gcp-01-298144853b0c)$ ls
main.tf  README-cloudshell.txt  static_ip  terraform.tfstate  terraform.tfstate.backup
student_01_b9934b2a0bae@cloudshell:~ (qwiklabs-gcp-01-298144853b0c)$
```



1. Use `terraform taint` to tell Terraform to recreate the instance:

```sh
student_01_b9934b2a0bae@cloudshell:~ (qwiklabs-gcp-01-298144853b0c)$ terraform taint google_compute_instance.vm_instance
Resource instance google_compute_instance.vm_instance has been marked as tainted.
student_01_b9934b2a0bae@cloudshell:~ (qwiklabs-gcp-01-298144853b0c)$
```

A tainted resource will be destroyed and recreated during the next `apply`.

1. Run `terraform apply` now:

```sh
student_01_b9934b2a0bae@cloudshell:~ (qwiklabs-gcp-01-298144853b0c)$ terraform apply
google_compute_network.vpc_network: Refreshing state... [id=projects/qwiklabs-gcp-01-298144853b0c/global/networks/terraform-network]
google_compute_address.vm_static_ip: Refreshing state... [id=projects/qwiklabs-gcp-01-298144853b0c/regions/us-central1/addresses/terraform-static-ip]
google_compute_instance.vm_instance: Refreshing state... [id=projects/qwiklabs-gcp-01-298144853b0c/zones/us-central1-c/instances/terraform-instance]

Terraform used the selected providers to generate the following execution plan. Resource actions are indicated with the following symbols:
-/+ destroy and then create replacement

Terraform will perform the following actions:

  # google_compute_instance.vm_instance is tainted, so must be replaced
-/+ resource "google_compute_instance" "vm_instance" {
      ~ cpu_platform         = "Intel Haswell" -> (known after apply)
      - enable_display       = false -> null
      ~ guest_accelerator    = [] -> (known after apply)
      ~ id                   = "projects/qwiklabs-gcp-01-298144853b0c/zones/us-central1-c/instances/terraform-instance" -> (known after apply)
      ~ instance_id          = "3453360189672988146" -> (known after apply)
      ~ label_fingerprint    = "42WmSpB8rSM=" -> (known after apply)
      - labels               = {} -> null
      - metadata             = {} -> null
      ~ metadata_fingerprint = "-IVbTFitpCY=" -> (known after apply)
      + min_cpu_platform     = (known after apply)
        name                 = "terraform-instance"
      ~ project              = "qwiklabs-gcp-01-298144853b0c" -> (known after apply)
      ~ self_link            = "https://www.googleapis.com/compute/v1/projects/qwiklabs-gcp-01-298144853b0c/zones/us-central1-c/instances/terraform-instance" -> (known after apply)
        tags                 = [
            "dev",
            "web",
        ]
      ~ tags_fingerprint     = "XaeQnaHMn9Y=" -> (known after apply)
      ~ zone                 = "us-central1-c" -> (known after apply)
        # (3 unchanged attributes hidden)

      ~ boot_disk {
          ~ device_name                = "persistent-disk-0" -> (known after apply)
          + disk_encryption_key_sha256 = (known after apply)
          + kms_key_self_link          = (known after apply)
          ~ source                     = "https://www.googleapis.com/compute/v1/projects/qwiklabs-gcp-01-298144853b0c/zones/us-central1-c/disks/terraform-instance" -> (known after apply)
            # (2 unchanged attributes hidden)

          ~ initialize_params {
              ~ image  = "https://www.googleapis.com/compute/v1/projects/cos-cloud/global/images/cos-stable-105-17412-1-66" -> "cos-cloud/cos-stable"
              ~ labels = {} -> (known after apply)
              ~ size   = 10 -> (known after apply)
              ~ type   = "pd-standard" -> (known after apply)
            }
        }

      ~ network_interface {
          ~ name               = "nic0" -> (known after apply)
          ~ network_ip         = "10.128.0.2" -> (known after apply)
          ~ subnetwork         = "https://www.googleapis.com/compute/v1/projects/qwiklabs-gcp-01-298144853b0c/regions/us-central1/subnetworks/terraform-network" -> (known after apply)
          ~ subnetwork_project = "qwiklabs-gcp-01-298144853b0c" -> (known after apply)
            # (1 unchanged attribute hidden)

          ~ access_config {
              ~ network_tier = "PREMIUM" -> (known after apply)
                # (1 unchanged attribute hidden)
            }
        }

      - scheduling {
          - automatic_restart   = true -> null
          - on_host_maintenance = "MIGRATE" -> null
          - preemptible         = false -> null
        }

      - shielded_instance_config {
          - enable_integrity_monitoring = true -> null
          - enable_secure_boot          = false -> null
          - enable_vtpm                 = true -> null
        }
    }

Plan: 1 to add, 0 to change, 1 to destroy.
╷
│ Warning: Version constraints inside provider configuration blocks are deprecated
│
│   on main.tf line 9, in provider "google":
│    9:   version = "3.5.0"
│
│ Terraform 0.13 and earlier allowed provider version constraints inside the provider configuration block, but that is now deprecated and will be removed in a future version of
│ Terraform. To silence this warning, move the provider version constraint into the required_providers block.
╵

Do you want to perform these actions?
  Terraform will perform the actions described above.
  Only 'yes' will be accepted to approve.

  Enter a value: yes

google_compute_instance.vm_instance: Destroying... [id=projects/qwiklabs-gcp-01-298144853b0c/zones/us-central1-c/instances/terraform-instance]
google_compute_instance.vm_instance: Still destroying... [id=projects/qwiklabs-gcp-01-298144853b0c/z...entral1-c/instances/terraform-instance, 10s elapsed]
google_compute_instance.vm_instance: Still destroying... [id=projects/qwiklabs-gcp-01-298144853b0c/z...entral1-c/instances/terraform-instance, 20s elapsed]
google_compute_instance.vm_instance: Still destroying... [id=projects/qwiklabs-gcp-01-298144853b0c/z...entral1-c/instances/terraform-instance, 30s elapsed]
google_compute_instance.vm_instance: Destruction complete after 31s
google_compute_instance.vm_instance: Creating...
google_compute_instance.vm_instance: Still creating... [10s elapsed]
google_compute_instance.vm_instance: Provisioning with 'local-exec'...
google_compute_instance.vm_instance (local-exec): Executing: ["/bin/sh" "-c" "echo terraform-instance:  35.226.140.52 >> ip_address.txt"]
google_compute_instance.vm_instance: Creation complete after 13s [id=projects/qwiklabs-gcp-01-298144853b0c/zones/us-central1-c/instances/terraform-instance]

Apply complete! Resources: 1 added, 0 changed, 1 destroyed.
student_01_b9934b2a0bae@cloudshell:~ (qwiklabs-gcp-01-298144853b0c)$
```

1. Verify everything worked by looking at the contents of the `ip_address.txt` file.

It contains the IP address, just as you asked

```sh
student_01_b9934b2a0bae@cloudshell:~ (qwiklabs-gcp-01-298144853b0c)$ ls
ip_address.txt  main.tf  README-cloudshell.txt  static_ip  terraform.tfstate  terraform.tfstate.backup
student_01_b9934b2a0bae@cloudshell:~ (qwiklabs-gcp-01-298144853b0c)$ cat ip_address.txt
terraform-instance: 35.226.140.52
student_01_b9934b2a0bae@cloudshell:~ (qwiklabs-gcp-01-298144853b0c)$
```

![]({{ site.url }}{{ site.baseurl }}/assets/images/gcp-tf-iac-5.png)

![]({{ site.url }}{{ site.baseurl }}/assets/images/gcp-tf-iac-6.png)

### Failed provisioners and tainted resources

If a resource is successfully created but fails a provisioning step, Terraform will error and mark the resource as *tainted*. A resource that is tainted still exists, but shouldn't be considered safe to use, since provisioning failed.

When you generate your next execution plan, Terraform will remove any tainted resources and create new resources, attempting to provision  them again after creation.

### Destroy provisioners

Provisioners can also be defined that run only during a destroy  operation. These are useful for performing system cleanup, extracting  data, etc.

For many resources, using built-in cleanup mechanisms is recommended  if possible (such as init scripts), but provisioners can be used if  necessary.

This lab won't show any destroyed provisioner examples. If you need to use destroy provisioners, please see the [Provisioners documentation](https://www.terraform.io/docs/provisioners/).



In this lab, we learned how to build, change, and destroy  infrastructure with Terraform. We then created resource dependencies,  and provisioned basic infrastructure with Terraform configuration files.
