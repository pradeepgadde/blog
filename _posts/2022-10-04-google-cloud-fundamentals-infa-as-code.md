---

layout: single
title:  "Getting Started with Google Cloud—Infrastructure as Code"
date:   2022-10-04 09:59:04 +0530
categories: Cloud
tags: GCP
show_date: true
classes: wide
header:
  teaser: /assets/images/gcp.png
author:
  name     : "Google Cloud Platform"
  avatar   : "/assets/images/gcp.png"

sidebar:
  - title: "Blog"
   
    text: "Checkout other topics"
    nav: my-sidebar

---

## Developing and Deploying the Cloud

- Development in the cloud
- Infrastructure as Code
- Terraform Lab

In this lab, you create a Terraform configuration with a module to automate the deployment of Google Cloud infrastructure. Specifically, you deploy one auto mode network with a firewall rule and two VM instances, as shown here

![]({{ site.url }}{{ site.baseurl }}/assets/images/gcp-45.png)

Configure your Cloud Shell environment to use Terraform.

Install Terraform
Terraform is now integrated into Cloud Shell. Verify which version is installed.

In the Cloud Console, click Activate Cloud Shell (Activate Cloud Shell icon).

If prompted, click Continue.

Confirm that Terraform is installed, and run the following command:

```sh

Welcome to Cloud Shell! Type "help" to get started.
Your Cloud Platform project in this session is set to qwiklabs-gcp-02-fb1d059a0729.
Use “gcloud config set project [PROJECT_ID]” to change to a different project.
student_04_adb32fa0eb88@cloudshell:~ (qwiklabs-gcp-02-fb1d059a0729)$ terraform --version
Terraform v1.3.0
on linux_amd64

Your version of Terraform is out of date! The latest version
is 1.3.1. You can update by downloading from https://www.terraform.io/downloads.html
student_04_adb32fa0eb88@cloudshell:~ (qwiklabs-gcp-02-fb1d059a0729)$ mkdir tfinfra
student_04_adb32fa0eb88@cloudshell:~ (qwiklabs-gcp-02-fb1d059a0729)$ cd tfinfra/
student_04_adb32fa0eb88@cloudshell:~/tfinfra (qwiklabs-gcp-02-fb1d059a0729)$ terraform init

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
student_04_adb32fa0eb88@cloudshell:~/tfinfra (qwiklabs-gcp-02-fb1d059a0729)$
```

Create the auto mode network mynetwork along with its firewall rule and two VM instances (mynet_us_vm and mynet_eu_vm).

Configure the firewall rule
Define a firewall rule to allow HTTP, SSH, RDP, and ICMP traffic on mynetwork.

Define the VM instances by creating a VM instance module. A module is a reusable configuration inside a folder. You will use this module for both VM instances of this lab.

These resources are leveraging the module in the instance folder and provide the name, zone, and network as inputs. Because these instances depend on a VPC network, you are using the google_compute_network.mynetwork.self_link reference to instruct Terraform to resolve these resources in a dependent order. In this case, the network is created before the instance.

It's time to apply the mynetwork configuration.

To rewrite the Terraform configuration files to a canonical format and style, run the following command:

```sh
student_04_adb32fa0eb88@cloudshell:~/tfinfra (qwiklabs-gcp-02-fb1d059a0729)$ terraform fmt
mynetwork.tf
student_04_adb32fa0eb88@cloudshell:~/tfinfra (qwiklabs-gcp-02-fb1d059a0729)$ cat mynetwork.tf
# Create the mynetwork network
resource "google_compute_network" "mynetwork" {
  name = "mynetwork"
  # RESOURCE properties go here
  auto_create_subnetworks = "true"
}
# Add a firewall rule to allow HTTP, SSH, RDP and ICMP traffic on mynetwork
resource "google_compute_firewall" "mynetwork-allow-http-ssh-rdp-icmp" {
  name = "mynetwork-allow-http-ssh-rdp-icmp"
  # RESOURCE properties go here
  network = google_compute_network.mynetwork.self_link
  allow {
    protocol = "tcp"
    ports    = ["22", "80", "3389"]
  }
  allow {
    protocol = "icmp"
  }
  source_ranges = ["0.0.0.0/0"]
}

# Create the mynet-us-vm instance
module "mynet-us-vm" {
  source           = "./instance"
  instance_name    = "mynet-us-vm"
  instance_zone    = "us-west1-c"
  instance_network = google_compute_network.mynetwork.self_link
}
# Create the mynet-eu-vm" instance
module "mynet-eu-vm" {
  source           = "./instance"
  instance_name    = "mynet-eu-vm"
  instance_zone    = "europe-west1-b"
  instance_network = google_compute_network.mynetwork.self_link
}student_04_adb32fa0eb88@cloudshell:~/tfinfra (qwiklabs-gcp-02-fb1d059a0729)$ tree
-bash: tree: command not found
student_04_adb32fa0eb88@cloudshell:~/tfinfra (qwiklabs-gcp-02-fb1d059a0729)$ pwd
/home/student_04_adb32fa0eb88/tfinfra
student_04_adb32fa0eb88@cloudshell:~/tfinfra (qwiklabs-gcp-02-fb1d059a0729)$ ls
instance  mynetwork.tf  provider.tf
student_04_adb32fa0eb88@cloudshell:~/tfinfra (qwiklabs-gcp-02-fb1d059a0729)$ cat provider.tf
provider "google" {}student_04_adb32fa0eb88@cloudshell:~/tfinfra (qwiklabs-gcp-02-fb1d059a0729)$
student_04_adb32fa0eb88@cloudshell:~/tfinfra (qwiklabs-gcp-02-fb1d059a0729)$ cat mynetwork.tf
# Create the mynetwork network
resource "google_compute_network" "mynetwork" {
  name = "mynetwork"
  # RESOURCE properties go here
  auto_create_subnetworks = "true"
}
# Add a firewall rule to allow HTTP, SSH, RDP and ICMP traffic on mynetwork
resource "google_compute_firewall" "mynetwork-allow-http-ssh-rdp-icmp" {
  name = "mynetwork-allow-http-ssh-rdp-icmp"
  # RESOURCE properties go here
  network = google_compute_network.mynetwork.self_link
  allow {
    protocol = "tcp"
    ports    = ["22", "80", "3389"]
  }
  allow {
    protocol = "icmp"
  }
  source_ranges = ["0.0.0.0/0"]
}

# Create the mynet-us-vm instance
module "mynet-us-vm" {
  source           = "./instance"
  instance_name    = "mynet-us-vm"
  instance_zone    = "us-west1-c"
  instance_network = google_compute_network.mynetwork.self_link
}
# Create the mynet-eu-vm" instance
module "mynet-eu-vm" {
  source           = "./instance"
  instance_name    = "mynet-eu-vm"
  instance_zone    = "europe-west1-b"
  instance_network = google_compute_network.mynetwork.self_link
}student_04_adb32fa0eb88@cloudshell:~/tfinfra (qwiklabs-gcp-02-fb1d059a0729)$ cat instance/
main.tf       variables.tf
student_04_adb32fa0eb88@cloudshell:~/tfinfra (qwiklabs-gcp-02-fb1d059a0729)$ cat instance/main.tf
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
    network = "${var.instance_network}"
    access_config {
      # Allocate a one-to-one NAT IP to the instance
    }
  }
}student_04_adb32fa0eb88@cloudshell:~/tfinfra (qwiklabs-gcp-02-fb1d059a0729)$ cat instance/variables.tf
variable "instance_name" {}
variable "instance_zone" {}
variable "instance_type" {
  default = "e2-micro"
  }
variable "instance_network" {}student_04_adb32fa0eb88@cloudshell:~/tfinfra (qwiklabs-gcp-02-fb1d059a0729)$
```



```sh
student_04_adb32fa0eb88@cloudshell:~/tfinfra (qwiklabs-gcp-02-fb1d059a0729)$ terraform init
Initializing modules...
- mynet-eu-vm in instance
- mynet-us-vm in instance

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
student_04_adb32fa0eb88@cloudshell:~/tfinfra (qwiklabs-gcp-02-fb1d059a0729)$
```



```sh
student_04_adb32fa0eb88@cloudshell:~/tfinfra (qwiklabs-gcp-02-fb1d059a0729)$ terraform plan

Terraform used the selected providers to generate the following execution plan. Resource actions are indicated with the following symbols:
  + create

Terraform will perform the following actions:

  # google_compute_firewall.mynetwork-allow-http-ssh-rdp-icmp will be created
  + resource "google_compute_firewall" "mynetwork-allow-http-ssh-rdp-icmp" {
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
      + machine_type         = "e2-micro"
      + metadata_fingerprint = (known after apply)
      + min_cpu_platform     = (known after apply)
      + name                 = "mynet-eu-vm"
      + project              = (known after apply)
      + self_link            = (known after apply)
      + tags_fingerprint     = (known after apply)
      + zone                 = "europe-west1-b"

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
      + machine_type         = "e2-micro"
      + metadata_fingerprint = (known after apply)
      + min_cpu_platform     = (known after apply)
      + name                 = "mynet-us-vm"
      + project              = (known after apply)
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
student_04_adb32fa0eb88@cloudshell:~/tfinfra (qwiklabs-gcp-02-fb1d059a0729)$
```



```sh
student_04_adb32fa0eb88@cloudshell:~/tfinfra (qwiklabs-gcp-02-fb1d059a0729)$ terraform apply

Terraform used the selected providers to generate the following execution plan. Resource actions are indicated with the following symbols:
  + create

Terraform will perform the following actions:

  # google_compute_firewall.mynetwork-allow-http-ssh-rdp-icmp will be created
  + resource "google_compute_firewall" "mynetwork-allow-http-ssh-rdp-icmp" {
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
      + machine_type         = "e2-micro"
      + metadata_fingerprint = (known after apply)
      + min_cpu_platform     = (known after apply)
      + name                 = "mynet-eu-vm"
      + project              = (known after apply)
      + self_link            = (known after apply)
      + tags_fingerprint     = (known after apply)
      + zone                 = "europe-west1-b"

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
      + machine_type         = "e2-micro"
      + metadata_fingerprint = (known after apply)
      + min_cpu_platform     = (known after apply)
      + name                 = "mynet-us-vm"
      + project              = (known after apply)
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
google_compute_network.mynetwork: Still creating... [40s elapsed]
google_compute_network.mynetwork: Creation complete after 44s [id=projects/qwiklabs-gcp-02-fb1d059a0729/global/networks/mynetwork]
module.mynet-eu-vm.google_compute_instance.vm_instance: Creating...
module.mynet-us-vm.google_compute_instance.vm_instance: Creating...
google_compute_firewall.mynetwork-allow-http-ssh-rdp-icmp: Creating...
module.mynet-eu-vm.google_compute_instance.vm_instance: Still creating... [10s elapsed]
module.mynet-us-vm.google_compute_instance.vm_instance: Still creating... [10s elapsed]
google_compute_firewall.mynetwork-allow-http-ssh-rdp-icmp: Still creating... [10s elapsed]
google_compute_firewall.mynetwork-allow-http-ssh-rdp-icmp: Creation complete after 12s [id=projects/qwiklabs-gcp-02-fb1d059a0729/global/firewalls/mynetwork-allow-http-ssh-rdp-icmp]
module.mynet-eu-vm.google_compute_instance.vm_instance: Still creating... [20s elapsed]
module.mynet-us-vm.google_compute_instance.vm_instance: Still creating... [20s elapsed]
module.mynet-us-vm.google_compute_instance.vm_instance: Creation complete after 20s [id=projects/qwiklabs-gcp-02-fb1d059a0729/zones/us-west1-c/instances/mynet-us-vm]
module.mynet-eu-vm.google_compute_instance.vm_instance: Creation complete after 23s [id=projects/qwiklabs-gcp-02-fb1d059a0729/zones/europe-west1-b/instances/mynet-eu-vm]

Apply complete! Resources: 4 added, 0 changed, 0 destroyed.
student_04_adb32fa0eb88@cloudshell:~/tfinfra (qwiklabs-gcp-02-fb1d059a0729)$
```

![]({{ site.url }}{{ site.baseurl }}/assets/images/gcp-46.png)

![]({{ site.url }}{{ site.baseurl }}/assets/images/gcp-47.png)



```sh
Linux mynet-us-vm 5.10.0-18-cloud-amd64 #1 SMP Debian 5.10.140-1 (2022-09-02) x86_64

The programs included with the Debian GNU/Linux system are free software;
the exact distribution terms for each program are described in the
individual files in /usr/share/doc/*/copyright.

Debian GNU/Linux comes with ABSOLUTELY NO WARRANTY, to the extent
permitted by applicable law.
Creating directory '/home/student-04-adb32fa0eb88'.
student-04-adb32fa0eb88@mynet-us-vm:~$ ping -c 3 35.190.206.33
PING 35.190.206.33 (35.190.206.33) 56(84) bytes of data.
64 bytes from 35.190.206.33: icmp_seq=1 ttl=54 time=137 ms
64 bytes from 35.190.206.33: icmp_seq=2 ttl=54 time=135 ms
64 bytes from 35.190.206.33: icmp_seq=3 ttl=54 time=135 ms

--- 35.190.206.33 ping statistics ---
3 packets transmitted, 3 received, 0% packet loss, time 2003ms
rtt min/avg/max/mdev = 135.090/135.590/136.587/0.704 ms
student-04-adb32fa0eb88@mynet-us-vm:~$ ping -c 3 10.132.0.2
PING 10.132.0.2 (10.132.0.2) 56(84) bytes of data.
64 bytes from 10.132.0.2: icmp_seq=1 ttl=64 time=137 ms
64 bytes from 10.132.0.2: icmp_seq=2 ttl=64 time=135 ms
64 bytes from 10.132.0.2: icmp_seq=3 ttl=64 time=135 ms

--- 10.132.0.2 ping statistics ---
3 packets transmitted, 3 received, 0% packet loss, time 2000ms
rtt min/avg/max/mdev = 134.863/135.626/137.080/1.028 ms
student-04-adb32fa0eb88@mynet-us-vm:~$ 
```





In this lab, you created a Terraform configuration with a module to automate the deployment of Google Cloud infrastructure. As your configuration changes, Terraform can create incremental execution plans, which allows you to build your overall configuration step by step.

The instance module allowed you to re-use the same resource configuration for multiple resources while providing properties as input variables. You can leverage the configuration and module that you created as a starting point for future deployments.