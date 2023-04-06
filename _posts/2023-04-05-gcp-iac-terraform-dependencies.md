---


layout: single
title:  "Creating Resource Dependencies with Terraform"
date:   2023-04-05 10:59:04 +0530
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
# Creating Resource Dependencies with Terraform

In this lab, you will create two VMs in the default network. We will  use variables to define the VM's attributes at runtime and use output  values to print a few resource attributes.

We will then add a static IP address to the first VM to examine how  terraform handles implicit dependencies.  We will then create a GCS  bucket by mentioning explicit dependency to the VM to examine how  terraform handles explicit dependency.

- Use variables and output values
- Observe implicit dependency
- Create explicit resource dependency

```sh
Welcome to Cloud Shell! Type "help" to get started.
Your Cloud Platform project in this session is set to qwiklabs-gcp-04-651bb74bbd19.
Use “gcloud config set project [PROJECT_ID]” to change to a different project.
student_00_a88ab091dadd@cloudshell:~ (qwiklabs-gcp-04-651bb74bbd19)$ terraform --version
Terraform v1.4.4
on linux_amd64
student_00_a88ab091dadd@cloudshell:~ (qwiklabs-gcp-04-651bb74bbd19)$ ls
README-cloudshell.txt
student_00_a88ab091dadd@cloudshell:~ (qwiklabs-gcp-04-651bb74bbd19)$ mkdir tfinfra && cd $_
student_00_a88ab091dadd@cloudshell:~/tfinfra (qwiklabs-gcp-04-651bb74bbd19)$ vi provider.tf
student_00_a88ab091dadd@cloudshell:~/tfinfra (qwiklabs-gcp-04-651bb74bbd19)$ cat provider.tf
  provider "google" {
  project = "qwiklabs-gcp-04-651bb74bbd19"
  region  = "us-east1"
  zone    = "us-east1-b"
}
student_00_a88ab091dadd@cloudshell:~/tfinfra (qwiklabs-gcp-04-651bb74bbd19)$ terraform init

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
student_00_a88ab091dadd@cloudshell:~/tfinfra (qwiklabs-gcp-04-651bb74bbd19)$
```

Terraform has now installed the necessary plug-ins to interact with the  Google Cloud API. Authentication is not required for API. The Cloud  Shell credentials give access to the project and APIs.

```sh
student_00_a88ab091dadd@cloudshell:~/tfinfra (qwiklabs-gcp-04-651bb74bbd19)$ vi instance.tf
student_00_a88ab091dadd@cloudshell:~/tfinfra (qwiklabs-gcp-04-651bb74bbd19)$ cat instance.tf
resource google_compute_instance "vm_instance" {
name         = "${var.instance_name}"
zone         = "${var.instance_zone}"
machine_type = "${var.instance_type}"
variable "instance_name" {
boot_disk {
    initialize_params {
      image = "debian-cloud/debian-11"
      }
  }
 network_interface {
    network = "default"
    access_config {
      # Allocate a one-to-one NAT IP to the instance
    }
  }
}
```
```sh
student_00_a88ab091dadd@cloudshell:~/tfinfra (qwiklabs-gcp-04-651bb74bbd19)$ vi variables.tf
student_00_a88ab091dadd@cloudshell:~/tfinfra (qwiklabs-gcp-04-651bb74bbd19)$ cat variables.tf
variable "instance_name" {
output "network_IP" {
  type        = string
machine_type = "${var.instance_type}"
      image = "debian-cloud/debian-11"
  description = "Name for the Google Compute instance"
}
variable "instance_zone" {
  type        = string
  description = "Zone for the Google Compute instance"
}
variable "instance_type" {
  type        = string
  description = "Disk type of the Google Compute instance"
  default     = "n1-standard-1"
  }
```
```sh
student_00_a88ab091dadd@cloudshell:~/tfinfra (qwiklabs-gcp-04-651bb74bbd19)$ vi outputs.tf
student_00_a88ab091dadd@cloudshell:~/tfinfra (qwiklabs-gcp-04-651bb74bbd19)$ vi instance.tf
```
```sh
student_00_a88ab091dadd@cloudshell:~/tfinfra (qwiklabs-gcp-04-651bb74bbd19)$ cat outputs.tf
output "network_IP" {
  value = google_compute_instance.vm_instance.instance_id
  description = "The internal ip address of the instance"
}
output "instance_link" {
  value = google_compute_instance.vm_instance.self_link
  description = "The URI of the created resource."
}
student_00_a88ab091dadd@cloudshell:~/tfinfra (qwiklabs-gcp-04-651bb74bbd19)$
```

```sh
student_00_a88ab091dadd@cloudshell:~/tfinfra (qwiklabs-gcp-04-651bb74bbd19)$ cat instance.tf
resource google_compute_instance "vm_instance" {
name         = "${var.instance_name}"
zone         = "${var.instance_zone}"
machine_type = "${var.instance_type}"
boot_disk {
    initialize_params {
      image = "debian-cloud/debian-11"
      image = "debian-cloud/debian-11"
      }
      }
  }
 network_interface {
    network = "default"
    access_config {
      # Allocate a one-to-one NAT IP to the instance
    }
  }
}
resource "google_compute_address" "vm_static_ip" {
  name = "terraform-static-ip"
}
student_00_a88ab091dadd@cloudshell:~/tfinfra (qwiklabs-gcp-04-651bb74bbd19)$ vi instance.tf
student_00_a88ab091dadd@cloudshell:~/tfinfra (qwiklabs-gcp-04-651bb74bbd19)$ cat instance.tf
resource google_compute_instance "vm_instance" {
name         = "${var.instance_name}"
zone         = "${var.instance_zone}"
machine_type = "${var.instance_type}"
boot_disk {
    initialize_params {
      image = "debian-cloud/debian-11"
      }
  }
 network_interface {
    network = "default"
    access_config {
      # Allocate a one-to-one NAT IP to the instance
    nat_ip = google_compute_address.vm_static_ip.address
  }
  }
}
resource "google_compute_address" "vm_static_ip" {
  name = "terraform-static-ip"
}
student_00_a88ab091dadd@cloudshell:~/tfinfra (qwiklabs-gcp-04-651bb74bbd19)$
```

```sh
tudent_00_a88ab091dadd@cloudshell:~/tfinfra (qwiklabs-gcp-04-651bb74bbd19)$ terraform init

Initializing the backend...

Initializing provider plugins...
- Reusing previous version of hashicorp/google from the dependency lock file
- Using previously-installed hashicorp/google v4.60.1

Terraform has been successfully initialized!

You may now begin working with Terraform. Try running "terraform plan" to see
any changes that are required for your infrastructure. All Terraform commands
should now work.

If you ever set or change modules or backend configuration for Terraform,
rerun this command to reinitialize your working directory. If you forget, other
commands will detect it and remind you to do so if necessary.
student_00_a88ab091dadd@cloudshell:~/tfinfra (qwiklabs-gcp-04-651bb74bbd19)$
```

```sh
student_00_a88ab091dadd@cloudshell:~/tfinfra (qwiklabs-gcp-04-651bb74bbd19)$ terraform plan
var.instance_name
  Name for the Google Compute instance

  Enter a value: myinstance

var.instance_zone
  Zone for the Google Compute instance

  Enter a value: us-east1-b


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

  # google_compute_instance.vm_instance will be created
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
      + name                 = "myinstance"
      + project              = (known after apply)
      + self_link            = (known after apply)
      + tags_fingerprint     = (known after apply)
      + zone                 = "us-east1-b"

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

Plan: 2 to add, 0 to change, 0 to destroy.

Changes to Outputs:
  + instance_link = (known after apply)
  + network_IP    = (known after apply)

───────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────

Note: You didn't use the -out option to save this plan, so Terraform can't guarantee to take exactly these actions if you run "terraform apply" now.
student_00_a88ab091dadd@cloudshell:~/tfinfra (qwiklabs-gcp-04-651bb74bbd19)$
```

```sh
student_00_a88ab091dadd@cloudshell:~/tfinfra (qwiklabs-gcp-04-651bb74bbd19)$ terraform apply
var.instance_name
  Name for the Google Compute instance

  Enter a value: myinstance

var.instance_zone
  Zone for the Google Compute instance

  Enter a value: us-east1-b


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

  # google_compute_instance.vm_instance will be created
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
      + name                 = "myinstance"
      + project              = (known after apply)
      + self_link            = (known after apply)
      + tags_fingerprint     = (known after apply)
      + zone                 = "us-east1-b"

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

Plan: 2 to add, 0 to change, 0 to destroy.

Changes to Outputs:
  + instance_link = (known after apply)
  + network_IP    = (known after apply)

Do you want to perform these actions?
  Terraform will perform the actions described above.
  Only 'yes' will be accepted to approve.

  Enter a value: yes

google_compute_address.vm_static_ip: Creating...
google_compute_address.vm_static_ip: Creation complete after 6s [id=projects/qwiklabs-gcp-04-651bb74bbd19/regions/us-east1/addresses/terraform-static-ip]
google_compute_instance.vm_instance: Creating...
google_compute_instance.vm_instance: Still creating... [10s elapsed]
google_compute_instance.vm_instance: Creation complete after 20s [id=projects/qwiklabs-gcp-04-651bb74bbd19/zones/us-east1-b/instances/myinstance]

Apply complete! Resources: 2 added, 0 changed, 0 destroyed.

Outputs:

instance_link = "https://www.googleapis.com/compute/v1/projects/qwiklabs-gcp-04-651bb74bbd19/zones/us-east1-b/instances/myinstance"
network_IP = "136231100128416515"
student_00_a88ab091dadd@cloudshell:~/tfinfra (qwiklabs-gcp-04-651bb74bbd19)$
```

![]({{ site.url }}{{ site.baseurl }}/assets/images/gcp-tf-6.png)
![]({{ site.url }}{{ site.baseurl }}/assets/images/gcp-tf-7.png)


Explicit dependencies are used to inform dependencies between resources that are not visible to Terraform. In this example, consider that you will run on your instance that expects to use a specific Cloud Storage bucket, but that dependency is configured inside the application code and thus not visible to Terraform. In that case, you can use depends_on to explicitly declare the dependency.

Add a Cloud Storage bucket and an instance with an explicit dependency on the bucket by adding the following base code into `exp.tf`:

```sh
student_00_a88ab091dadd@cloudshell:~/tfinfra (qwiklabs-gcp-04-651bb74bbd19)$ cat exp.tf
resource "google_compute_instance" "another_instance" {
  name         = "terraform-instance-2"
  machine_type = "f1-micro"
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
  # Tells Terraform that this VM instance must be created only after the
  # storage bucket has been created.
  depends_on = [google_storage_bucket.example_bucket]
}
resource "google_storage_bucket" "example_bucket" {
  name     = "pradeepgadde-gcp-cs"
  location = "US"
  website {
    main_page_suffix = "index.html"
    not_found_page   = "404.html"
  }
}
student_00_a88ab091dadd@cloudshell:~/tfinfra (qwiklabs-gcp-04-651bb74bbd19)$
```

```sh
student_00_a88ab091dadd@cloudshell:~/tfinfra (qwiklabs-gcp-04-651bb74bbd19)$ terraform plan
var.instance_name
  Name for the Google Compute instance

  Enter a value: myinstance

var.instance_zone
  Zone for the Google Compute instance

  Enter a value: us-east1-b

google_compute_address.vm_static_ip: Refreshing state... [id=projects/qwiklabs-gcp-04-651bb74bbd19/regions/us-east1/addresses/terraform-static-ip]
google_compute_instance.vm_instance: Refreshing state... [id=projects/qwiklabs-gcp-04-651bb74bbd19/zones/us-east1-b/instances/myinstance]

Terraform used the selected providers to generate the following execution plan. Resource actions are indicated with the following symbols:
  + create

Terraform will perform the following actions:

  # google_compute_instance.another_instance will be created
  + resource "google_compute_instance" "another_instance" {
      + can_ip_forward       = false
      + cpu_platform         = (known after apply)
      + current_status       = (known after apply)
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

  # google_storage_bucket.example_bucket will be created
  + resource "google_storage_bucket" "example_bucket" {
      + force_destroy               = false
      + id                          = (known after apply)
      + location                    = "US"
      + name                        = "pradeepgadde-gcp-cs"
      + project                     = (known after apply)
      + public_access_prevention    = (known after apply)
      + self_link                   = (known after apply)
      + storage_class               = "STANDARD"
      + uniform_bucket_level_access = (known after apply)
      + url                         = (known after apply)

      + website {
          + main_page_suffix = "index.html"
          + not_found_page   = "404.html"
        }
    }

Plan: 2 to add, 0 to change, 0 to destroy.

───────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────

Note: You didn't use the -out option to save this plan, so Terraform can't guarantee to take exactly these actions if you run "terraform apply" now.
student_00_a88ab091dadd@cloudshell:~/tfinfra (qwiklabs-gcp-04-651bb74bbd19)$
```

```sh
student_00_a88ab091dadd@cloudshell:~/tfinfra (qwiklabs-gcp-04-651bb74bbd19)$ terraform apply
var.instance_name
  Name for the Google Compute instance

  Enter a value: myinstance

var.instance_zone
  Zone for the Google Compute instance

  Enter a value: us-east1-b

google_compute_address.vm_static_ip: Refreshing state... [id=projects/qwiklabs-gcp-04-651bb74bbd19/regions/us-east1/addresses/terraform-static-ip]
google_compute_instance.vm_instance: Refreshing state... [id=projects/qwiklabs-gcp-04-651bb74bbd19/zones/us-east1-b/instances/myinstance]

Terraform used the selected providers to generate the following execution plan. Resource actions are indicated with the following symbols:
  + create

Terraform will perform the following actions:

  # google_compute_instance.another_instance will be created
  + resource "google_compute_instance" "another_instance" {
      + can_ip_forward       = false
      + cpu_platform         = (known after apply)
      + current_status       = (known after apply)
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

  # google_storage_bucket.example_bucket will be created
  + resource "google_storage_bucket" "example_bucket" {
      + force_destroy               = false
      + id                          = (known after apply)
      + location                    = "US"
      + name                        = "pradeepgadde-gcp-cs"
      + project                     = (known after apply)
      + public_access_prevention    = (known after apply)
      + self_link                   = (known after apply)
      + storage_class               = "STANDARD"
      + uniform_bucket_level_access = (known after apply)
      + url                         = (known after apply)

      + website {
          + main_page_suffix = "index.html"
          + not_found_page   = "404.html"
        }
    }

Plan: 2 to add, 0 to change, 0 to destroy.

Do you want to perform these actions?
  Terraform will perform the actions described above.
  Only 'yes' will be accepted to approve.

  Enter a value: yes

google_storage_bucket.example_bucket: Creating...
google_storage_bucket.example_bucket: Creation complete after 2s [id=pradeepgadde-gcp-cs]
google_compute_instance.another_instance: Creating...
google_compute_instance.another_instance: Still creating... [10s elapsed]
google_compute_instance.another_instance: Creation complete after 19s [id=projects/qwiklabs-gcp-04-651bb74bbd19/zones/us-east1-b/instances/terraform-instance-2]

Apply complete! Resources: 2 added, 0 changed, 0 destroyed.

Outputs:

instance_link = "https://www.googleapis.com/compute/v1/projects/qwiklabs-gcp-04-651bb74bbd19/zones/us-east1-b/instances/myinstance"
network_IP = "136231100128416515"
student_00_a88ab091dadd@cloudshell:~/tfinfra (qwiklabs-gcp-04-651bb74bbd19)$
```

![]({{ site.url }}{{ site.baseurl }}/assets/images/gcp-tf-8.png)
![]({{ site.url }}{{ site.baseurl }}/assets/images/gcp-tf-9.png)

To view resource dependency graph of the resource created, execute the following command

```sh
student_00_a88ab091dadd@cloudshell:~/tfinfra (qwiklabs-gcp-04-651bb74bbd19)$ terraform graph | dot -Tsvg > graph.svg
student_00_a88ab091dadd@cloudshell:~/tfinfra (qwiklabs-gcp-04-651bb74bbd19)$ ls
exp.tf  graph.svg  instance.tf  outputs.tf  provider.tf  terraform.tfstate  terraform.tfstate.backup  variables.tf
student_00_a88ab091dadd@cloudshell:~/tfinfra (qwiklabs-gcp-04-651bb74bbd19)$
```
![]({{ site.url }}{{ site.baseurl }}/assets/images/graph.svg)

In this lab, we have a VM instance with a static IP address to view how implicit resource dependencies are handled with Terraform. We then created an explicit dependency by adding the `depend_o`n argument so that we can create a GCS bucket before creating a VM instance. We  also viewed the dependency graph that terraform uses to trace the order of resource creation.