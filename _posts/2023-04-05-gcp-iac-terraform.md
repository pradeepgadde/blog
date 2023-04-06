---


layout: single
title:  "Infrastructure as Code with Terraform"
date:   2023-04-05 09:59:04 +0530
categories: Cloud
tags: GCP
classes: wide
show_date: true
header:
  overlay_image: /assets/images/gcp-banner-2.png
  og_image: /assets/images/gcp-banner-2.png
  teaser: /assets/images/gpcne.png
author:
  name     : "Cloud Network Engineer"
  avatar   : "/assets/images/gpcne.png"

sidebar:
  - title: "Topics"
    nav: my-sidebar

---
# Infrastructure as Code with Terraform

In this lab, you will use Terraform to create, update, and destroy  Google Cloud resources. You will start by defining Google Cloud as the  provider.

You will then create a VM instance without mentioning the network to  see how terraform parses the configuration code. You will then edit the  code to add network and create a VM instance on Google Cloud.

You will explore how to update the VM instance. You will edit the  existing configuration to add tags and then edit the machine type. You  will then execute terraform commands to destroy the resources created.

```sh
Welcome to Cloud Shell! Type "help" to get started.
Your Cloud Platform project in this session is set to qwiklabs-gcp-00-5e985de63b63.
Use “gcloud config set project [PROJECT_ID]” to change to a different project.
student_00_66cebfab53e7@cloudshell:~ (qwiklabs-gcp-00-5e985de63b63)$ terraform --version
Terraform v1.4.4
on linux_amd64
student_00_66cebfab53e7@cloudshell:~ (qwiklabs-gcp-00-5e985de63b63)$ mkdir compute
student_00_66cebfab53e7@cloudshell:~ (qwiklabs-gcp-00-5e985de63b63)$ cd compute/
student_00_66cebfab53e7@cloudshell:~/compute (qwiklabs-gcp-00-5e985de63b63)$ touch main.tf
student_00_66cebfab53e7@cloudshell:~/compute (qwiklabs-gcp-00-5e985de63b63)$ vi main.tf
student_00_66cebfab53e7@cloudshell:~/compute (qwiklabs-gcp-00-5e985de63b63)$ cat main.tf
terraform {
  required_providers {
    google = {
      source = "hashicorp/google"
    }
  }
}
provider "google" {
  region  = "us-central1"
  zone    = "us-central1-c"
}
student_00_66cebfab53e7@cloudshell:~/compute (qwiklabs-gcp-00-5e985de63b63)$ terraform init

Initializing the backend...

Initializing provider plugins...
- Finding latest version of hashicorp/google...
- Installing hashicorp/google v4.60.1...
- Installed hashicorp/google v4.60.1 (signed by HashiCorp)

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
student_00_66cebfab53e7@cloudshell:~/compute (qwiklabs-gcp-00-5e985de63b63)$
```



```sh
student_00_66cebfab53e7@cloudshell:~/compute (qwiklabs-gcp-00-5e985de63b63)$ cat main.tf
terraform {
  required_providers {
    google = {
      source = "hashicorp/google"
    }
  }
}
provider "google" {
  region  = "us-central1"
  zone    = "us-central1-c"
}
resource "google_compute_instance" "terraform" {
  name         = "terraform"
  machine_type = "n1-standard-2"
  boot_disk {
    initialize_params {
      image = "debian-cloud/debian-11"
    }
  }
}
student_00_66cebfab53e7@cloudshell:~/compute (qwiklabs-gcp-00-5e985de63b63)$
```



```sh
student_00_66cebfab53e7@cloudshell:~/compute (qwiklabs-gcp-00-5e985de63b63)$ terraform plan
╷
│ Error: Insufficient network_interface blocks
│
│   on main.tf line 12, in resource "google_compute_instance" "terraform":
│   12: resource "google_compute_instance" "terraform" {
│
│ At least 1 "network_interface" blocks are required.
╵
student_00_66cebfab53e7@cloudshell:~/compute (qwiklabs-gcp-00-5e985de63b63)$
```



```sh
student_00_66cebfab53e7@cloudshell:~/compute (qwiklabs-gcp-00-5e985de63b63)$ cat main.tf
terraform {
  required_providers {
    google = {
      source = "hashicorp/google"
    }
  }
}
provider "google" {
  region  = "us-central1"
  zone    = "us-central1-c"
}
resource "google_compute_instance" "terraform" {
  name         = "terraform"
  machine_type = "n1-standard-2"
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
}
student_00_66cebfab53e7@cloudshell:~/compute (qwiklabs-gcp-00-5e985de63b63)$
```



```sh
student_00_66cebfab53e7@cloudshell:~/compute (qwiklabs-gcp-00-5e985de63b63)$ terraform plan

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
      + machine_type         = "n1-standard-2"
      + metadata_fingerprint = (known after apply)
      + min_cpu_platform     = (known after apply)
      + name                 = "terraform"
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
student_00_66cebfab53e7@cloudshell:~/compute (qwiklabs-gcp-00-5e985de63b63)$
```



```sh
student_00_66cebfab53e7@cloudshell:~/compute (qwiklabs-gcp-00-5e985de63b63)$ terraform apply

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
      + machine_type         = "n1-standard-2"
      + metadata_fingerprint = (known after apply)
      + min_cpu_platform     = (known after apply)
      + name                 = "terraform"
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
google_compute_instance.terraform: Creation complete after 17s [id=projects/qwiklabs-gcp-00-5e985de63b63/zones/us-central1-c/instances/terraform]

Apply complete! Resources: 1 added, 0 changed, 0 destroyed.
student_00_66cebfab53e7@cloudshell:~/compute (qwiklabs-gcp-00-5e985de63b63)$
```



![]({{ site.url }}{{ site.baseurl }}/assets/images/gcp-tf-1.png)

![]({{ site.url }}{{ site.baseurl }}/assets/images/gcp-tf-2.png)

In this task, we will be performing 2 types of changes to the infrastructure:

- Adding network tags
- Editing the machine-type

### Adding tags to the compute resource

In addition to creating resources, Terraform can also make changes to those resources.

1. Add a `tags` argument to the instance we just created so that it looks like this:

```sh
student_00_66cebfab53e7@cloudshell:~/compute (qwiklabs-gcp-00-5e985de63b63)$ cat main.tf
terraform {
  required_providers {
    google = {
      source = "hashicorp/google"
    }
  }
}
provider "google" {
  region  = "us-central1"
  zone    = "us-central1-c"
}
resource "google_compute_instance" "terraform" {
  name         = "terraform"
  machine_type = "n1-standard-2"
  tags         = ["web", "dev"]
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
}
student_00_66cebfab53e7@cloudshell:~/compute (qwiklabs-gcp-00-5e985de63b63)$
```



```sh
student_00_66cebfab53e7@cloudshell:~/compute (qwiklabs-gcp-00-5e985de63b63)$ terraform plan
google_compute_instance.terraform: Refreshing state... [id=projects/qwiklabs-gcp-00-5e985de63b63/zones/us-central1-c/instances/terraform]

Terraform used the selected providers to generate the following execution plan. Resource actions are indicated with the following symbols:
  ~ update in-place

Terraform will perform the following actions:

  # google_compute_instance.terraform will be updated in-place
  ~ resource "google_compute_instance" "terraform" {
        id                   = "projects/qwiklabs-gcp-00-5e985de63b63/zones/us-central1-c/instances/terraform"
        name                 = "terraform"
      ~ tags                 = [
          + "dev",
          + "web",
        ]
        # (17 unchanged attributes hidden)

        # (4 unchanged blocks hidden)
    }

Plan: 0 to add, 1 to change, 0 to destroy.

───────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────

Note: You didn't use the -out option to save this plan, so Terraform can't guarantee to take exactly these actions if you run "terraform apply" now.
student_00_66cebfab53e7@cloudshell:~/compute (qwiklabs-gcp-00-5e985de63b63)$
```



```sh
student_00_66cebfab53e7@cloudshell:~/compute (qwiklabs-gcp-00-5e985de63b63)$ terraform apply
google_compute_instance.terraform: Refreshing state... [id=projects/qwiklabs-gcp-00-5e985de63b63/zones/us-central1-c/instances/terraform]

Terraform used the selected providers to generate the following execution plan. Resource actions are indicated with the following symbols:
  ~ update in-place

Terraform will perform the following actions:

  # google_compute_instance.terraform will be updated in-place
  ~ resource "google_compute_instance" "terraform" {
        id                   = "projects/qwiklabs-gcp-00-5e985de63b63/zones/us-central1-c/instances/terraform"
        name                 = "terraform"
      ~ tags                 = [
          + "dev",
          + "web",
        ]
        # (17 unchanged attributes hidden)

        # (4 unchanged blocks hidden)
    }

Plan: 0 to add, 1 to change, 0 to destroy.

Do you want to perform these actions?
  Terraform will perform the actions described above.
  Only 'yes' will be accepted to approve.

  Enter a value: yes

google_compute_instance.terraform: Modifying... [id=projects/qwiklabs-gcp-00-5e985de63b63/zones/us-central1-c/instances/terraform]
google_compute_instance.terraform: Still modifying... [id=projects/qwiklabs-gcp-00-5e985de63b63/zones/us-central1-c/instances/terraform, 10s elapsed]
google_compute_instance.terraform: Modifications complete after 18s [id=projects/qwiklabs-gcp-00-5e985de63b63/zones/us-central1-c/instances/terraform]

Apply complete! Resources: 0 added, 1 changed, 0 destroyed.
student_00_66cebfab53e7@cloudshell:~/compute (qwiklabs-gcp-00-5e985de63b63)$
```



![]({{ site.url }}{{ site.baseurl }}/assets/images/gcp-tf-3.png)



### Editing the machine type without stopping the VM

Machine type of a VM cannot be changed on a running VM. Let us see  how terraform processes the change in machine type for a running VM.

Navigate to **main.tf** and edit the machine_type argument of terraform instance from `n1-standard-2` to `n1-standard-1` so that it looks like this

```sh
student_00_66cebfab53e7@cloudshell:~/compute (qwiklabs-gcp-00-5e985de63b63)$ cat main.tf
terraform {
  required_providers {
    google = {
      source = "hashicorp/google"
    }
  }
}
provider "google" {
  region  = "us-central1"
  zone    = "us-central1-c"
}
resource "google_compute_instance" "terraform" {
  name         = "terraform"
  machine_type = "n1-standard-1"
  tags         = ["web", "dev"]
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
}
student_00_66cebfab53e7@cloudshell:~/compute (qwiklabs-gcp-00-5e985de63b63)$
```

```sh
google_compute_instance.terraform: Refreshing state... [id=projects/qwiklabs-gcp-00-5e985de63b63/zones/us-central1-c/instances/terraform]

Terraform used the selected providers to generate the following execution plan. Resource actions are indicated with the following symbols:
  ~ update in-place

Terraform will perform the following actions:

  # google_compute_instance.terraform will be updated in-place
  ~ resource "google_compute_instance" "terraform" {
        id                   = "projects/qwiklabs-gcp-00-5e985de63b63/zones/us-central1-c/instances/terraform"
      ~ machine_type         = "n1-standard-2" -> "n1-standard-1"
        name                 = "terraform"
        tags                 = [
            "dev",
            "web",
        ]
        # (16 unchanged attributes hidden)

        # (4 unchanged blocks hidden)
    }

Plan: 0 to add, 1 to change, 0 to destroy.

───────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────

Note: You didn't use the -out option to save this plan, so Terraform can't guarantee to take exactly these actions if you run "terraform apply" now.
student_00_66cebfab53e7@cloudshell:~/compute (qwiklabs-gcp-00-5e985de63b63)$
```



```sh
student_00_66cebfab53e7@cloudshell:~/compute (qwiklabs-gcp-00-5e985de63b63)$ terraform apply
google_compute_instance.terraform: Refreshing state... [id=projects/qwiklabs-gcp-00-5e985de63b63/zones/us-central1-c/instances/terraform]

Terraform used the selected providers to generate the following execution plan. Resource actions are indicated with the following symbols:
  ~ update in-place

Terraform will perform the following actions:

  # google_compute_instance.terraform will be updated in-place
  ~ resource "google_compute_instance" "terraform" {
        id                   = "projects/qwiklabs-gcp-00-5e985de63b63/zones/us-central1-c/instances/terraform"
      ~ machine_type         = "n1-standard-2" -> "n1-standard-1"
        name                 = "terraform"
        tags                 = [
            "dev",
            "web",
        ]
        # (16 unchanged attributes hidden)

        # (4 unchanged blocks hidden)
    }

Plan: 0 to add, 1 to change, 0 to destroy.

Do you want to perform these actions?
  Terraform will perform the actions described above.
  Only 'yes' will be accepted to approve.

  Enter a value: yes

google_compute_instance.terraform: Modifying... [id=projects/qwiklabs-gcp-00-5e985de63b63/zones/us-central1-c/instances/terraform]
╷
│ Error: Changing the machine_type, min_cpu_platform, service_account, enable_display, shielded_instance_config, scheduling.node_affinities or network_interface.[#d].(network/subnetwork/subnetwork_project) or advanced_machine_features on a started instance requires stopping it. To acknowledge this, please set allow_stopping_for_update = true in your config. You can also stop it by setting desired_status = "TERMINATED", but the instance will not be restarted after the update.
│
│   with google_compute_instance.terraform,
│   on main.tf line 12, in resource "google_compute_instance" "terraform":
│   12: resource "google_compute_instance" "terraform" {
│
╵
student_00_66cebfab53e7@cloudshell:~/compute (qwiklabs-gcp-00-5e985de63b63)$
```

The machine-type cannot be changed on a running VM. To ensure the VM stops before updating the `machine_type`, set `allow_stopping_for_update argument` to `true` so that the code looks like this:

```sh
student_00_66cebfab53e7@cloudshell:~/compute (qwiklabs-gcp-00-5e985de63b63)$ cat main.tf
terraform {
  required_providers {
    google = {
      source = "hashicorp/google"
    }
  }
}
provider "google" {
  region  = "us-central1"
  zone    = "us-central1-c"
}
resource "google_compute_instance" "terraform" {
  name         = "terraform"
  machine_type = "n1-standard-1"
  tags         = ["web", "dev"]
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
allow_stopping_for_update = true
}
student_00_66cebfab53e7@cloudshell:~/compute (qwiklabs-gcp-00-5e985de63b63)$
```

```sh
tudent_00_66cebfab53e7@cloudshell:~/compute (qwiklabs-gcp-00-5e985de63b63)$ terraform plan
google_compute_instance.terraform: Refreshing state... [id=projects/qwiklabs-gcp-00-5e985de63b63/zones/us-central1-c/instances/terraform]

Terraform used the selected providers to generate the following execution plan. Resource actions are indicated with the following symbols:
  ~ update in-place

Terraform will perform the following actions:

  # google_compute_instance.terraform will be updated in-place
  ~ resource "google_compute_instance" "terraform" {
      + allow_stopping_for_update = true
        id                        = "projects/qwiklabs-gcp-00-5e985de63b63/zones/us-central1-c/instances/terraform"
      ~ machine_type              = "n1-standard-2" -> "n1-standard-1"
        name                      = "terraform"
        tags                      = [
            "dev",
            "web",
        ]
        # (16 unchanged attributes hidden)

        # (4 unchanged blocks hidden)
    }

Plan: 0 to add, 1 to change, 0 to destroy.

───────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────

Note: You didn't use the -out option to save this plan, so Terraform can't guarantee to take exactly these actions if you run "terraform apply" now.
student_00_66cebfab53e7@cloudshell:~/compute (qwiklabs-gcp-00-5e985de63b63)$
```

```sh
student_00_66cebfab53e7@cloudshell:~/compute (qwiklabs-gcp-00-5e985de63b63)$ terraform apply
google_compute_instance.terraform: Refreshing state... [id=projects/qwiklabs-gcp-00-5e985de63b63/zones/us-central1-c/instances/terraform]

Terraform used the selected providers to generate the following execution plan. Resource actions are indicated with the following symbols:
  ~ update in-place

Terraform will perform the following actions:

  # google_compute_instance.terraform will be updated in-place
  ~ resource "google_compute_instance" "terraform" {
      + allow_stopping_for_update = true
        id                        = "projects/qwiklabs-gcp-00-5e985de63b63/zones/us-central1-c/instances/terraform"
      ~ machine_type              = "n1-standard-2" -> "n1-standard-1"
        name                      = "terraform"
        tags                      = [
            "dev",
            "web",
        ]
        # (16 unchanged attributes hidden)

        # (4 unchanged blocks hidden)
    }

Plan: 0 to add, 1 to change, 0 to destroy.

Do you want to perform these actions?
  Terraform will perform the actions described above.
  Only 'yes' will be accepted to approve.

  Enter a value: yes

google_compute_instance.terraform: Modifying... [id=projects/qwiklabs-gcp-00-5e985de63b63/zones/us-central1-c/instances/terraform]
google_compute_instance.terraform: Still modifying... [id=projects/qwiklabs-gcp-00-5e985de63b63/zones/us-central1-c/instances/terraform, 10s elapsed]
google_compute_instance.terraform: Still modifying... [id=projects/qwiklabs-gcp-00-5e985de63b63/zones/us-central1-c/instances/terraform, 20s elapsed]
google_compute_instance.terraform: Still modifying... [id=projects/qwiklabs-gcp-00-5e985de63b63/zones/us-central1-c/instances/terraform, 30s elapsed]
google_compute_instance.terraform: Still modifying... [id=projects/qwiklabs-gcp-00-5e985de63b63/zones/us-central1-c/instances/terraform, 40s elapsed]
google_compute_instance.terraform: Modifications complete after 48s [id=projects/qwiklabs-gcp-00-5e985de63b63/zones/us-central1-c/instances/terraform]

Apply complete! Resources: 0 added, 1 changed, 0 destroyed.
student_00_66cebfab53e7@cloudshell:~/compute (qwiklabs-gcp-00-5e985de63b63)$
```



![]({{ site.url }}{{ site.baseurl }}/assets/images/gcp-tf-4.png)



## Destroy the infrastructure

You have now seen how to build and change infrastructure. Before  moving on to creating multiple resources and showing resource  dependencies, you will see how to completely destroy your  Terraform-managed infrastructure.

```sh
student_00_66cebfab53e7@cloudshell:~/compute (qwiklabs-gcp-00-5e985de63b63)$ terraform destroy
google_compute_instance.terraform: Refreshing state... [id=projects/qwiklabs-gcp-00-5e985de63b63/zones/us-central1-c/instances/terraform]

Terraform used the selected providers to generate the following execution plan. Resource actions are indicated with the following symbols:
  - destroy

Terraform will perform the following actions:

  # google_compute_instance.terraform will be destroyed
  - resource "google_compute_instance" "terraform" {
      - allow_stopping_for_update = true -> null
      - can_ip_forward            = false -> null
      - cpu_platform              = "Intel Haswell" -> null
      - current_status            = "RUNNING" -> null
      - deletion_protection       = false -> null
      - enable_display            = false -> null
      - guest_accelerator         = [] -> null
      - id                        = "projects/qwiklabs-gcp-00-5e985de63b63/zones/us-central1-c/instances/terraform" -> null
      - instance_id               = "3774993110594216940" -> null
      - label_fingerprint         = "42WmSpB8rSM=" -> null
      - labels                    = {} -> null
      - machine_type              = "n1-standard-1" -> null
      - metadata                  = {} -> null
      - metadata_fingerprint      = "7q-qmkCexKA=" -> null
      - name                      = "terraform" -> null
      - project                   = "qwiklabs-gcp-00-5e985de63b63" -> null
      - resource_policies         = [] -> null
      - self_link                 = "https://www.googleapis.com/compute/v1/projects/qwiklabs-gcp-00-5e985de63b63/zones/us-central1-c/instances/terraform" -> null
      - tags                      = [
          - "dev",
          - "web",
        ] -> null
      - tags_fingerprint          = "XaeQnaHMn9Y=" -> null
      - zone                      = "us-central1-c" -> null

      - boot_disk {
          - auto_delete = true -> null
          - device_name = "persistent-disk-0" -> null
          - mode        = "READ_WRITE" -> null
          - source      = "https://www.googleapis.com/compute/v1/projects/qwiklabs-gcp-00-5e985de63b63/zones/us-central1-c/disks/terraform" -> null

          - initialize_params {
              - image  = "https://www.googleapis.com/compute/v1/projects/debian-cloud/global/images/debian-11-bullseye-v20230306" -> null
              - labels = {} -> null
              - size   = 10 -> null
              - type   = "pd-standard" -> null
            }
        }

      - network_interface {
          - name               = "nic0" -> null
          - network            = "https://www.googleapis.com/compute/v1/projects/qwiklabs-gcp-00-5e985de63b63/global/networks/default" -> null
          - network_ip         = "10.128.0.2" -> null
          - queue_count        = 0 -> null
          - stack_type         = "IPV4_ONLY" -> null
          - subnetwork         = "https://www.googleapis.com/compute/v1/projects/qwiklabs-gcp-00-5e985de63b63/regions/us-central1/subnetworks/default" -> null
          - subnetwork_project = "qwiklabs-gcp-00-5e985de63b63" -> null

          - access_config {
              - nat_ip       = "34.68.20.115" -> null
              - network_tier = "PREMIUM" -> null
            }
        }

      - scheduling {
          - automatic_restart   = true -> null
          - min_node_cpus       = 0 -> null
          - on_host_maintenance = "MIGRATE" -> null
          - preemptible         = false -> null
          - provisioning_model  = "STANDARD" -> null
        }

      - shielded_instance_config {
          - enable_integrity_monitoring = true -> null
          - enable_secure_boot          = false -> null
          - enable_vtpm                 = true -> null
        }
    }

Plan: 0 to add, 0 to change, 1 to destroy.

Do you really want to destroy all resources?
  Terraform will destroy all your managed infrastructure, as shown above.
  There is no undo. Only 'yes' will be accepted to confirm.

  Enter a value: yes

google_compute_instance.terraform: Destroying... [id=projects/qwiklabs-gcp-00-5e985de63b63/zones/us-central1-c/instances/terraform]
google_compute_instance.terraform: Still destroying... [id=projects/qwiklabs-gcp-00-5e985de63b63/zones/us-central1-c/instances/terraform, 10s elapsed]
google_compute_instance.terraform: Still destroying... [id=projects/qwiklabs-gcp-00-5e985de63b63/zones/us-central1-c/instances/terraform, 20s elapsed]
google_compute_instance.terraform: Destruction complete after 25s

Destroy complete! Resources: 1 destroyed.
student_00_66cebfab53e7@cloudshell:~/compute (qwiklabs-gcp-00-5e985de63b63)$
```



![]({{ site.url }}{{ site.baseurl }}/assets/images/gcp-tf-5.png)



