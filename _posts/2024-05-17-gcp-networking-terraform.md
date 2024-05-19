---

layout: single
title:  "GCP Deploying Networks with Terraform"
categories: Cloud
tags: GCP
toc: true
toc_sticky: true
show_date: true
header:
  overlay_image: /assets/images/gcp-banner.png
  og_image: /assets/images/gcp-banner.png
  teaser: /assets/images/gpcne.png
author:
  name     : "Cloud Network Engineer"
  avatar   : "/assets/images/gpcne.png"

sidebar:
  - title: "Topics"
    nav: my-sidebar
---
# GCP Deploying Networks with Terraform

In this lab, you created Terraform configurations and modules to  automate the deployment of a custom network. As the configuration  changes, Terraform can determine what changed and create incremental  execution plans, which allows you to build your overall configuration  step-by-step.

The instance module allowed you to re-use the same resource  configuration for multiple resources while providing properties as input variables. You can leverage the configurations and modules that you  created as a starting point for future deployments.

## VM Instance Module

```sh
student_01_aa8f3a74b714@cloudshell:~/tfnet (qwiklabs-gcp-03-8f557a4564b9)$ ls
instance  managementnet.tf  mynetwork.tf  privatenet.tf  provider.tf  terraform.tfstate  terraform.tfstate.backup
student_01_aa8f3a74b714@cloudshell:~/tfnet (qwiklabs-gcp-03-8f557a4564b9)$ cat instance/main.tf 
variable "instance_name" {}
variable "instance_zone" {}

variable "instance_type" {
  default = "e2-standard-2"
}

variable "instance_subnetwork" {}

resource "google_compute_instance" "vm_instance" {
  name         = var.instance_name
  zone         = var.instance_zone
  machine_type = var.instance_type

  boot_disk {
    initialize_params {
      image = "debian-cloud/debian-11"
    }
  }

  network_interface {
    subnetwork = var.instance_subnetwork

    access_config {
      # Allocate a one-to-one NAT IP to the instance
    }
  }
}
student_01_aa8f3a74b714@cloudshell:~/tfnet (qwiklabs-gcp-03-8f557a4564b9)$
```
## Custom VPC with one VM
```sh
student_01_aa8f3a74b714@cloudshell:~/tfnet (qwiklabs-gcp-03-8f557a4564b9)$ cat managementnet.tf 
# Create managementnet network
resource "google_compute_network" "managementnet" {
  name                    = "managementnet"
  auto_create_subnetworks = "false"
}
# Create managementsubnet-us subnetwork
resource "google_compute_subnetwork" "managementsubnet-us" {
  name          = "managementsubnet-us"
  region        = "us-west1"
  network       = google_compute_network.managementnet.self_link
  ip_cidr_range = "10.130.0.0/20"
}
# Create a firewall rule to allow HTTP, SSH, RDP and ICMP traffic on managementnet
resource "google_compute_firewall" "managementnet_allow_http_ssh_rdp_icmp" {
  name = "managementnet-allow-http-ssh-rdp-icmp"
  source_ranges = [
    "0.0.0.0/0"
  ]
  network = google_compute_network.managementnet.self_link

  allow {
    protocol = "tcp"
    ports    = ["22", "80", "3389"]
  }

  allow {
    protocol = "icmp"
  }
}
# Add the managementnet-us-vm instance
module "managementnet-us-vm" {
  source              = "./instance"
  instance_name       = "managementnet-us-vm"
  instance_zone       = "us-west1-c"
  instance_subnetwork = google_compute_subnetwork.managementsubnet-us.self_link
}
student_01_aa8f3a74b714@cloudshell:~/tfnet (qwiklabs-gcp-03-8f557a4564b9)$
```
## Another Custom VPC with one VM
```sh
student_01_aa8f3a74b714@cloudshell:~/tfnet (qwiklabs-gcp-03-8f557a4564b9)$ cat privatenet.tf 

# Create privatenet network
resource "google_compute_network" "privatenet" {
  name                    = "privatenet"
  auto_create_subnetworks = false
}
resource "google_compute_subnetwork" "privatesubnet-us" {
  name          = "privatesubnet-us"
  region        = "us-west1"
  network       = google_compute_network.privatenet.self_link
  ip_cidr_range = "172.16.0.0/24"
}
# Create privatesubnet-second-subnet subnetwork
resource "google_compute_subnetwork" "privatesubnet-second-subnet" {
  name          = "privatesubnet-second-subnet"
  region        = "us-central1"
  network       = google_compute_network.privatenet.self_link
  ip_cidr_range = "172.20.0.0/24"
}
# Create a firewall rule to allow HTTP, SSH, RDP and ICMP traffic on privatenet
resource "google_compute_firewall" "privatenet-allow-http-ssh-rdp-icmp" {
  name = "privatenet-allow-http-ssh-rdp-icmp"
  source_ranges = [
    "0.0.0.0/0"
  ]
  network = google_compute_network.privatenet.self_link

  allow {
    protocol = "tcp"
    ports    = ["22", "80", "3389"]
  }

  allow {
    protocol = "icmp"
  }
}
# Add the privatenet-us-vm instance
module "privatenet-us-vm" {
  source              = "./instance"
  instance_name       = "privatenet-us-vm"
  instance_zone       = "us-west1-c"
  instance_subnetwork = google_compute_subnetwork.privatesubnet-us.self_link
}
student_01_aa8f3a74b714@cloudshell:~/tfnet (qwiklabs-gcp-03-8f557a4564b9)$
```
## AutoMode VPC with two VMs
```sh
student_01_aa8f3a74b714@cloudshell:~/tfnet (qwiklabs-gcp-03-8f557a4564b9)$ cat mynetwork.tf 

# Create the mynetwork network
resource "google_compute_network" "mynetwork" {
  name                    = "mynetwork"
  auto_create_subnetworks = "true"
}

# Create a firewall rule to allow HTTP, SSH, RDP and ICMP traffic on mynetwork
resource "google_compute_firewall" "mynetwork-allow-http-ssh-rdp-icmp" {
  name = "mynetwork-allow-http-ssh-rdp-icmp"
  source_ranges = [
    "0.0.0.0/0"
  ]
  network = google_compute_network.mynetwork.self_link

  allow {
    protocol = "tcp"
    ports    = ["22", "80", "3389"]
  }

  allow {
    protocol = "icmp"
  }
}
# Create the mynet-us-vm instance
module "mynet-us-vm" {
  source              = "./instance"
  instance_name       = "mynet-us-vm"
  instance_zone       = "us-west1-c"
  instance_subnetwork = google_compute_network.mynetwork.self_link
}

# Create the mynet-second-vm" instance
module "mynet-second-vm" {
  source              = "./instance"
  instance_name       = "mynet-second-vm"
  instance_zone       = "us-central1-b"
  instance_subnetwork = google_compute_network.mynetwork.self_link
}
student_01_aa8f3a74b714@cloudshell:~/tfnet (qwiklabs-gcp-03-8f557a4564b9)$
```

```sh
student_01_aa8f3a74b714@cloudshell:~/tfnet (qwiklabs-gcp-03-8f557a4564b9)$ ls
instance  managementnet.tf  mynetwork.tf  privatenet.tf  provider.tf  terraform.tfstate  terraform.tfstate.backup
student_01_aa8f3a74b714@cloudshell:~/tfnet (qwiklabs-gcp-03-8f557a4564b9)$ cat provider.tf 
provider "google" {}student_01_aa8f3a74b714@cloudshell:~/tfnet (qwiklabs-gcp-03-8f557a4564b9)$ 
```

## Verification

### List of Networks
```sh
student_01_aa8f3a74b714@cloudshell:~/tfnet (qwiklabs-gcp-03-8f557a4564b9)$ gcloud compute networks list
NAME: default
SUBNET_MODE: AUTO
BGP_ROUTING_MODE: REGIONAL
IPV4_RANGE: 
GATEWAY_IPV4: 

NAME: managementnet
SUBNET_MODE: CUSTOM
BGP_ROUTING_MODE: REGIONAL
IPV4_RANGE: 
GATEWAY_IPV4: 

NAME: mynetwork
SUBNET_MODE: AUTO
BGP_ROUTING_MODE: REGIONAL
IPV4_RANGE: 
GATEWAY_IPV4: 

NAME: privatenet
SUBNET_MODE: CUSTOM
BGP_ROUTING_MODE: REGIONAL
IPV4_RANGE: 
GATEWAY_IPV4: 
student_01_aa8f3a74b714@cloudshell:~/tfnet (qwiklabs-gcp-03-8f557a4564b9)$ 
```

### List of VMs

```sh
student_01_aa8f3a74b714@cloudshell:~/tfnet (qwiklabs-gcp-03-8f557a4564b9)$ gcloud compute instances list
NAME: mynet-second-vm
ZONE: us-central1-b
MACHINE_TYPE: e2-standard-2
PREEMPTIBLE: 
INTERNAL_IP: 10.128.0.2
EXTERNAL_IP: 35.192.130.110
STATUS: RUNNING

NAME: managementnet-us-vm
ZONE: us-west1-c
MACHINE_TYPE: e2-standard-2
PREEMPTIBLE: 
INTERNAL_IP: 10.130.0.2
EXTERNAL_IP: 34.83.12.47
STATUS: RUNNING

NAME: mynet-us-vm
ZONE: us-west1-c
MACHINE_TYPE: e2-standard-2
PREEMPTIBLE: 
INTERNAL_IP: 10.138.0.2
EXTERNAL_IP: 34.82.50.79
STATUS: RUNNING

NAME: privatenet-us-vm
ZONE: us-west1-c
MACHINE_TYPE: e2-standard-2
PREEMPTIBLE: 
INTERNAL_IP: 172.16.0.2
EXTERNAL_IP: 34.168.162.35
STATUS: RUNNING
student_01_aa8f3a74b714@cloudshell:~/tfnet (qwiklabs-gcp-03-8f557a4564b9)$ 
```
### List of Subnets in each VPC
```sh
student_01_aa8f3a74b714@cloudshell:~/tfnet (qwiklabs-gcp-03-8f557a4564b9)$ gcloud compute networks subnets list --network managementnet
NAME: managementsubnet-us
REGION: us-west1
NETWORK: managementnet
RANGE: 10.130.0.0/20
STACK_TYPE: IPV4_ONLY
IPV6_ACCESS_TYPE: 
INTERNAL_IPV6_PREFIX: 
EXTERNAL_IPV6_PREFIX: 
student_01_aa8f3a74b714@cloudshell:~/tfnet (qwiklabs-gcp-03-8f557a4564b9)$
```
```sh
student_01_aa8f3a74b714@cloudshell:~/tfnet (qwiklabs-gcp-03-8f557a4564b9)$ gcloud compute networks subnets list --network privatenet
NAME: privatesubnet-second-subnet
REGION: us-central1
NETWORK: privatenet
RANGE: 172.20.0.0/24
STACK_TYPE: IPV4_ONLY
IPV6_ACCESS_TYPE: 
INTERNAL_IPV6_PREFIX: 
EXTERNAL_IPV6_PREFIX: 

NAME: privatesubnet-us
REGION: us-west1
NETWORK: privatenet
RANGE: 172.16.0.0/24
STACK_TYPE: IPV4_ONLY
IPV6_ACCESS_TYPE: 
INTERNAL_IPV6_PREFIX: 
EXTERNAL_IPV6_PREFIX: 
student_01_aa8f3a74b714@cloudshell:~/tfnet (qwiklabs-gcp-03-8f557a4564b9)$
```
```sh
student_01_aa8f3a74b714@cloudshell:~/tfnet (qwiklabs-gcp-03-8f557a4564b9)$ gcloud compute networks subnets list --network mynetwork
NAME: mynetwork
REGION: us-central1
NETWORK: mynetwork
RANGE: 10.128.0.0/20
STACK_TYPE: IPV4_ONLY
IPV6_ACCESS_TYPE: 
INTERNAL_IPV6_PREFIX: 
EXTERNAL_IPV6_PREFIX: 

NAME: mynetwork
REGION: europe-west1
NETWORK: mynetwork
RANGE: 10.132.0.0/20
STACK_TYPE: IPV4_ONLY
IPV6_ACCESS_TYPE: 
INTERNAL_IPV6_PREFIX: 
EXTERNAL_IPV6_PREFIX: 

NAME: mynetwork
REGION: us-west1
NETWORK: mynetwork
RANGE: 10.138.0.0/20
STACK_TYPE: IPV4_ONLY
IPV6_ACCESS_TYPE: 
INTERNAL_IPV6_PREFIX: 
EXTERNAL_IPV6_PREFIX: 

NAME: mynetwork
REGION: asia-east1
NETWORK: mynetwork
RANGE: 10.140.0.0/20
STACK_TYPE: IPV4_ONLY
IPV6_ACCESS_TYPE: 
INTERNAL_IPV6_PREFIX: 
EXTERNAL_IPV6_PREFIX: 

NAME: mynetwork
REGION: us-east1
NETWORK: mynetwork
RANGE: 10.142.0.0/20
STACK_TYPE: IPV4_ONLY
IPV6_ACCESS_TYPE: 
INTERNAL_IPV6_PREFIX: 
EXTERNAL_IPV6_PREFIX: 

NAME: mynetwork
REGION: asia-northeast1
NETWORK: mynetwork
RANGE: 10.146.0.0/20
STACK_TYPE: IPV4_ONLY
IPV6_ACCESS_TYPE: 
INTERNAL_IPV6_PREFIX: 
EXTERNAL_IPV6_PREFIX: 

NAME: mynetwork
REGION: asia-southeast1
NETWORK: mynetwork
RANGE: 10.148.0.0/20
STACK_TYPE: IPV4_ONLY
IPV6_ACCESS_TYPE: 
INTERNAL_IPV6_PREFIX: 
EXTERNAL_IPV6_PREFIX: 

NAME: mynetwork
REGION: us-east4
NETWORK: mynetwork
RANGE: 10.150.0.0/20
STACK_TYPE: IPV4_ONLY
IPV6_ACCESS_TYPE: 
INTERNAL_IPV6_PREFIX: 
EXTERNAL_IPV6_PREFIX: 

NAME: mynetwork
REGION: australia-southeast1
NETWORK: mynetwork
RANGE: 10.152.0.0/20
STACK_TYPE: IPV4_ONLY
IPV6_ACCESS_TYPE: 
INTERNAL_IPV6_PREFIX: 
EXTERNAL_IPV6_PREFIX: 

NAME: mynetwork
REGION: europe-west2
NETWORK: mynetwork
RANGE: 10.154.0.0/20
STACK_TYPE: IPV4_ONLY
IPV6_ACCESS_TYPE: 
INTERNAL_IPV6_PREFIX: 
EXTERNAL_IPV6_PREFIX: 

NAME: mynetwork
REGION: europe-west3
NETWORK: mynetwork
RANGE: 10.156.0.0/20
STACK_TYPE: IPV4_ONLY
IPV6_ACCESS_TYPE: 
INTERNAL_IPV6_PREFIX: 
EXTERNAL_IPV6_PREFIX: 

NAME: mynetwork
REGION: asia-south1
NETWORK: mynetwork
RANGE: 10.160.0.0/20
STACK_TYPE: IPV4_ONLY
IPV6_ACCESS_TYPE: 
INTERNAL_IPV6_PREFIX: 
EXTERNAL_IPV6_PREFIX: 

NAME: mynetwork
REGION: europe-west4
NETWORK: mynetwork
RANGE: 10.164.0.0/20
STACK_TYPE: IPV4_ONLY
IPV6_ACCESS_TYPE: 
INTERNAL_IPV6_PREFIX: 
EXTERNAL_IPV6_PREFIX: 

NAME: mynetwork
REGION: europe-north1
NETWORK: mynetwork
RANGE: 10.166.0.0/20
STACK_TYPE: IPV4_ONLY
IPV6_ACCESS_TYPE: 
INTERNAL_IPV6_PREFIX: 
EXTERNAL_IPV6_PREFIX: 

NAME: mynetwork
REGION: us-west2
NETWORK: mynetwork
RANGE: 10.168.0.0/20
STACK_TYPE: IPV4_ONLY
IPV6_ACCESS_TYPE: 
INTERNAL_IPV6_PREFIX: 
EXTERNAL_IPV6_PREFIX: 

NAME: mynetwork
REGION: us-west3
NETWORK: mynetwork
RANGE: 10.180.0.0/20
STACK_TYPE: IPV4_ONLY
IPV6_ACCESS_TYPE: 
INTERNAL_IPV6_PREFIX: 
EXTERNAL_IPV6_PREFIX: 

NAME: mynetwork
REGION: us-west4
NETWORK: mynetwork
RANGE: 10.182.0.0/20
STACK_TYPE: IPV4_ONLY
IPV6_ACCESS_TYPE: 
INTERNAL_IPV6_PREFIX: 
EXTERNAL_IPV6_PREFIX: 

NAME: mynetwork
REGION: europe-central2
NETWORK: mynetwork
RANGE: 10.186.0.0/20
STACK_TYPE: IPV4_ONLY
IPV6_ACCESS_TYPE: 
INTERNAL_IPV6_PREFIX: 
EXTERNAL_IPV6_PREFIX: 

NAME: mynetwork
REGION: southamerica-west1
NETWORK: mynetwork
RANGE: 10.194.0.0/20
STACK_TYPE: IPV4_ONLY
IPV6_ACCESS_TYPE: 
INTERNAL_IPV6_PREFIX: 
EXTERNAL_IPV6_PREFIX: 

NAME: mynetwork
REGION: us-east5
NETWORK: mynetwork
RANGE: 10.202.0.0/20
STACK_TYPE: IPV4_ONLY
IPV6_ACCESS_TYPE: 
INTERNAL_IPV6_PREFIX: 
EXTERNAL_IPV6_PREFIX: 

NAME: mynetwork
REGION: us-south1
NETWORK: mynetwork
RANGE: 10.206.0.0/20
STACK_TYPE: IPV4_ONLY
IPV6_ACCESS_TYPE: 
INTERNAL_IPV6_PREFIX: 
EXTERNAL_IPV6_PREFIX: 

NAME: mynetwork
REGION: me-west1
NETWORK: mynetwork
RANGE: 10.208.0.0/20
STACK_TYPE: IPV4_ONLY
IPV6_ACCESS_TYPE: 
INTERNAL_IPV6_PREFIX: 
EXTERNAL_IPV6_PREFIX: 

NAME: mynetwork
REGION: europe-west12
NETWORK: mynetwork
RANGE: 10.210.0.0/20
STACK_TYPE: IPV4_ONLY
IPV6_ACCESS_TYPE: 
INTERNAL_IPV6_PREFIX: 
EXTERNAL_IPV6_PREFIX: 

NAME: mynetwork
REGION: me-central1
NETWORK: mynetwork
RANGE: 10.212.0.0/20
STACK_TYPE: IPV4_ONLY
IPV6_ACCESS_TYPE: 
INTERNAL_IPV6_PREFIX: 
EXTERNAL_IPV6_PREFIX: 

NAME: mynetwork
REGION: europe-west10
NETWORK: mynetwork
RANGE: 10.214.0.0/20
STACK_TYPE: IPV4_ONLY
IPV6_ACCESS_TYPE: 
INTERNAL_IPV6_PREFIX: 
EXTERNAL_IPV6_PREFIX: 

NAME: mynetwork
REGION: me-central2
NETWORK: mynetwork
RANGE: 10.216.0.0/20
STACK_TYPE: IPV4_ONLY
IPV6_ACCESS_TYPE: 
INTERNAL_IPV6_PREFIX: 
EXTERNAL_IPV6_PREFIX: 

NAME: mynetwork
REGION: africa-south1
NETWORK: mynetwork
RANGE: 10.218.0.0/20
STACK_TYPE: IPV4_ONLY
IPV6_ACCESS_TYPE: 
INTERNAL_IPV6_PREFIX: 
EXTERNAL_IPV6_PREFIX: 
student_01_aa8f3a74b714@cloudshell:~/tfnet (qwiklabs-gcp-03-8f557a4564b9)$ 
```

## Firewall Rules

```sh
student_01_aa8f3a74b714@cloudshell:~/tfnet (qwiklabs-gcp-03-8f557a4564b9)$ gcloud compute firewall-rules list
NAME: default-allow-icmp
NETWORK: default
DIRECTION: INGRESS
PRIORITY: 65534
ALLOW: icmp
DENY: 
DISABLED: False

NAME: default-allow-internal
NETWORK: default
DIRECTION: INGRESS
PRIORITY: 65534
ALLOW: tcp:0-65535,udp:0-65535,icmp
DENY: 
DISABLED: False

NAME: default-allow-rdp
NETWORK: default
DIRECTION: INGRESS
PRIORITY: 65534
ALLOW: tcp:3389
DENY: 
DISABLED: False

NAME: default-allow-ssh
NETWORK: default
DIRECTION: INGRESS
PRIORITY: 65534
ALLOW: tcp:22
DENY: 
DISABLED: False

NAME: managementnet-allow-http-ssh-rdp-icmp
NETWORK: managementnet
DIRECTION: INGRESS
PRIORITY: 1000
ALLOW: icmp,tcp:22,tcp:80,tcp:3389
DENY: 
DISABLED: False

NAME: mynetwork-allow-http-ssh-rdp-icmp
NETWORK: mynetwork
DIRECTION: INGRESS
PRIORITY: 1000
ALLOW: icmp,tcp:22,tcp:80,tcp:3389
DENY: 
DISABLED: False

NAME: privatenet-allow-http-ssh-rdp-icmp
NETWORK: privatenet
DIRECTION: INGRESS
PRIORITY: 1000
ALLOW: icmp,tcp:22,tcp:80,tcp:3389
DENY: 
DISABLED: False

To show all fields of the firewall, please show in JSON format: --format=json
To show all fields in table format, please see the examples in --help.

student_01_aa8f3a74b714@cloudshell:~/tfnet (qwiklabs-gcp-03-8f557a4564b9)$ 
```

## Terraform State

```sh
student_01_aa8f3a74b714@cloudshell:~/tfnet (qwiklabs-gcp-03-8f557a4564b9)$ terraform show
# google_compute_firewall.managementnet_allow_http_ssh_rdp_icmp:
resource "google_compute_firewall" "managementnet_allow_http_ssh_rdp_icmp" {
    creation_timestamp      = "2024-05-18T23:21:51.431-07:00"
    destination_ranges      = []
    direction               = "INGRESS"
    disabled                = false
    id                      = "projects/qwiklabs-gcp-03-8f557a4564b9/global/firewalls/managementnet-allow-http-ssh-rdp-icmp"
    name                    = "managementnet-allow-http-ssh-rdp-icmp"
    network                 = "https://www.googleapis.com/compute/v1/projects/qwiklabs-gcp-03-8f557a4564b9/global/networks/managementnet"
    priority                = 1000
    project                 = "qwiklabs-gcp-03-8f557a4564b9"
    self_link               = "https://www.googleapis.com/compute/v1/projects/qwiklabs-gcp-03-8f557a4564b9/global/firewalls/managementnet-allow-http-ssh-rdp-icmp"
    source_ranges           = [
        "0.0.0.0/0",
    ]
    source_service_accounts = []
    source_tags             = []
    target_service_accounts = []
    target_tags             = []

    allow {
        ports    = [
            "22",
            "80",
            "3389",
        ]
        protocol = "tcp"
    }
    allow {
        ports    = []
        protocol = "icmp"
    }
}

# google_compute_firewall.mynetwork-allow-http-ssh-rdp-icmp:
resource "google_compute_firewall" "mynetwork-allow-http-ssh-rdp-icmp" {
    creation_timestamp = "2024-05-18T23:40:57.450-07:00"
    destination_ranges = []
    direction          = "INGRESS"
    disabled           = false
    id                 = "projects/qwiklabs-gcp-03-8f557a4564b9/global/firewalls/mynetwork-allow-http-ssh-rdp-icmp"
    name               = "mynetwork-allow-http-ssh-rdp-icmp"
    network            = "https://www.googleapis.com/compute/v1/projects/qwiklabs-gcp-03-8f557a4564b9/global/networks/mynetwork"
    priority           = 1000
    project            = "qwiklabs-gcp-03-8f557a4564b9"
    self_link          = "https://www.googleapis.com/compute/v1/projects/qwiklabs-gcp-03-8f557a4564b9/global/firewalls/mynetwork-allow-http-ssh-rdp-icmp"
    source_ranges      = [
        "0.0.0.0/0",
    ]

    allow {
        ports    = [
            "22",
            "80",
            "3389",
        ]
        protocol = "tcp"
    }
    allow {
        ports    = []
        protocol = "icmp"
    }
}

# google_compute_firewall.privatenet-allow-http-ssh-rdp-icmp:
resource "google_compute_firewall" "privatenet-allow-http-ssh-rdp-icmp" {
    creation_timestamp      = "2024-05-18T23:34:39.638-07:00"
    destination_ranges      = []
    direction               = "INGRESS"
    disabled                = false
    id                      = "projects/qwiklabs-gcp-03-8f557a4564b9/global/firewalls/privatenet-allow-http-ssh-rdp-icmp"
    name                    = "privatenet-allow-http-ssh-rdp-icmp"
    network                 = "https://www.googleapis.com/compute/v1/projects/qwiklabs-gcp-03-8f557a4564b9/global/networks/privatenet"
    priority                = 1000
    project                 = "qwiklabs-gcp-03-8f557a4564b9"
    self_link               = "https://www.googleapis.com/compute/v1/projects/qwiklabs-gcp-03-8f557a4564b9/global/firewalls/privatenet-allow-http-ssh-rdp-icmp"
    source_ranges           = [
        "0.0.0.0/0",
    ]
    source_service_accounts = []
    source_tags             = []
    target_service_accounts = []
    target_tags             = []

    allow {
        ports    = [
            "22",
            "80",
            "3389",
        ]
        protocol = "tcp"
    }
    allow {
        ports    = []
        protocol = "icmp"
    }
}

# google_compute_network.managementnet:
resource "google_compute_network" "managementnet" {
    auto_create_subnetworks                   = false
    delete_default_routes_on_create           = false
    enable_ula_internal_ipv6                  = false
    id                                        = "projects/qwiklabs-gcp-03-8f557a4564b9/global/networks/managementnet"
    mtu                                       = 0
    name                                      = "managementnet"
    network_firewall_policy_enforcement_order = "AFTER_CLASSIC_FIREWALL"
    numeric_id                                = "4849012589208732"
    project                                   = "qwiklabs-gcp-03-8f557a4564b9"
    routing_mode                              = "REGIONAL"
    self_link                                 = "https://www.googleapis.com/compute/v1/projects/qwiklabs-gcp-03-8f557a4564b9/global/networks/managementnet"
}

# google_compute_network.mynetwork:
resource "google_compute_network" "mynetwork" {
    auto_create_subnetworks                   = true
    delete_default_routes_on_create           = false
    enable_ula_internal_ipv6                  = false
    id                                        = "projects/qwiklabs-gcp-03-8f557a4564b9/global/networks/mynetwork"
    mtu                                       = 0
    name                                      = "mynetwork"
    network_firewall_policy_enforcement_order = "AFTER_CLASSIC_FIREWALL"
    numeric_id                                = "4853593895493443617"
    project                                   = "qwiklabs-gcp-03-8f557a4564b9"
    routing_mode                              = "REGIONAL"
    self_link                                 = "https://www.googleapis.com/compute/v1/projects/qwiklabs-gcp-03-8f557a4564b9/global/networks/mynetwork"
}

# google_compute_network.privatenet:
resource "google_compute_network" "privatenet" {
    auto_create_subnetworks                   = false
    delete_default_routes_on_create           = false
    enable_ula_internal_ipv6                  = false
    id                                        = "projects/qwiklabs-gcp-03-8f557a4564b9/global/networks/privatenet"
    mtu                                       = 0
    name                                      = "privatenet"
    network_firewall_policy_enforcement_order = "AFTER_CLASSIC_FIREWALL"
    numeric_id                                = "4225781711040160668"
    project                                   = "qwiklabs-gcp-03-8f557a4564b9"
    routing_mode                              = "REGIONAL"
    self_link                                 = "https://www.googleapis.com/compute/v1/projects/qwiklabs-gcp-03-8f557a4564b9/global/networks/privatenet"
}

# google_compute_subnetwork.managementsubnet-us:
resource "google_compute_subnetwork" "managementsubnet-us" {
    creation_timestamp         = "2024-05-18T23:21:53.724-07:00"
    gateway_address            = "10.130.0.1"
    id                         = "projects/qwiklabs-gcp-03-8f557a4564b9/regions/us-west1/subnetworks/managementsubnet-us"
    ip_cidr_range              = "10.130.0.0/20"
    name                       = "managementsubnet-us"
    network                    = "https://www.googleapis.com/compute/v1/projects/qwiklabs-gcp-03-8f557a4564b9/global/networks/managementnet"
    private_ip_google_access   = false
    private_ipv6_google_access = "DISABLE_GOOGLE_ACCESS"
    project                    = "qwiklabs-gcp-03-8f557a4564b9"
    purpose                    = "PRIVATE"
    region                     = "us-west1"
    secondary_ip_range         = []
    self_link                  = "https://www.googleapis.com/compute/v1/projects/qwiklabs-gcp-03-8f557a4564b9/regions/us-west1/subnetworks/managementsubnet-us"
    stack_type                 = "IPV4_ONLY"
}

# google_compute_subnetwork.privatesubnet-second-subnet:
resource "google_compute_subnetwork" "privatesubnet-second-subnet" {
    creation_timestamp         = "2024-05-18T23:34:42.259-07:00"
    gateway_address            = "172.20.0.1"
    id                         = "projects/qwiklabs-gcp-03-8f557a4564b9/regions/us-central1/subnetworks/privatesubnet-second-subnet"
    ip_cidr_range              = "172.20.0.0/24"
    name                       = "privatesubnet-second-subnet"
    network                    = "https://www.googleapis.com/compute/v1/projects/qwiklabs-gcp-03-8f557a4564b9/global/networks/privatenet"
    private_ip_google_access   = false
    private_ipv6_google_access = "DISABLE_GOOGLE_ACCESS"
    project                    = "qwiklabs-gcp-03-8f557a4564b9"
    purpose                    = "PRIVATE"
    region                     = "us-central1"
    secondary_ip_range         = []
    self_link                  = "https://www.googleapis.com/compute/v1/projects/qwiklabs-gcp-03-8f557a4564b9/regions/us-central1/subnetworks/privatesubnet-second-subnet"
    stack_type                 = "IPV4_ONLY"
}

# google_compute_subnetwork.privatesubnet-us:
resource "google_compute_subnetwork" "privatesubnet-us" {
    creation_timestamp         = "2024-05-18T23:34:40.907-07:00"
    gateway_address            = "172.16.0.1"
    id                         = "projects/qwiklabs-gcp-03-8f557a4564b9/regions/us-west1/subnetworks/privatesubnet-us"
    ip_cidr_range              = "172.16.0.0/24"
    name                       = "privatesubnet-us"
    network                    = "https://www.googleapis.com/compute/v1/projects/qwiklabs-gcp-03-8f557a4564b9/global/networks/privatenet"
    private_ip_google_access   = false
    private_ipv6_google_access = "DISABLE_GOOGLE_ACCESS"
    project                    = "qwiklabs-gcp-03-8f557a4564b9"
    purpose                    = "PRIVATE"
    region                     = "us-west1"
    secondary_ip_range         = []
    self_link                  = "https://www.googleapis.com/compute/v1/projects/qwiklabs-gcp-03-8f557a4564b9/regions/us-west1/subnetworks/privatesubnet-us"
    stack_type                 = "IPV4_ONLY"
}


# module.managementnet-us-vm.google_compute_instance.vm_instance:
resource "google_compute_instance" "vm_instance" {
    can_ip_forward       = false
    cpu_platform         = "Intel Broadwell"
    current_status       = "RUNNING"
    deletion_protection  = false
    effective_labels     = {}
    enable_display       = false
    guest_accelerator    = []
    id                   = "projects/qwiklabs-gcp-03-8f557a4564b9/zones/us-west1-c/instances/managementnet-us-vm"
    instance_id          = "8100293643159741566"
    label_fingerprint    = "42WmSpB8rSM="
    labels               = {}
    machine_type         = "e2-standard-2"
    metadata             = {}
    metadata_fingerprint = "7QFr01P_g1k="
    name                 = "managementnet-us-vm"
    project              = "qwiklabs-gcp-03-8f557a4564b9"
    resource_policies    = []
    self_link            = "https://www.googleapis.com/compute/v1/projects/qwiklabs-gcp-03-8f557a4564b9/zones/us-west1-c/instances/managementnet-us-vm"
    tags                 = []
    tags_fingerprint     = "42WmSpB8rSM="
    terraform_labels     = {}
    zone                 = "us-west1-c"

    boot_disk {
        auto_delete = true
        device_name = "persistent-disk-0"
        mode        = "READ_WRITE"
        source      = "https://www.googleapis.com/compute/v1/projects/qwiklabs-gcp-03-8f557a4564b9/zones/us-west1-c/disks/managementnet-us-vm"

        initialize_params {
            enable_confidential_compute = false
            image                       = "https://www.googleapis.com/compute/v1/projects/debian-cloud/global/images/debian-11-bullseye-v20240515"
            labels                      = {}
            provisioned_iops            = 0
            provisioned_throughput      = 0
            resource_manager_tags       = {}
            size                        = 10
            type                        = "pd-standard"
        }
    }

    network_interface {
        internal_ipv6_prefix_length = 0
        name                        = "nic0"
        network                     = "https://www.googleapis.com/compute/v1/projects/qwiklabs-gcp-03-8f557a4564b9/global/networks/managementnet"
        network_ip                  = "10.130.0.2"
        queue_count                 = 0
        stack_type                  = "IPV4_ONLY"
        subnetwork                  = "https://www.googleapis.com/compute/v1/projects/qwiklabs-gcp-03-8f557a4564b9/regions/us-west1/subnetworks/managementsubnet-us"
        subnetwork_project          = "qwiklabs-gcp-03-8f557a4564b9"

        access_config {
            nat_ip       = "34.83.12.47"
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


# module.mynet-second-vm.google_compute_instance.vm_instance:
resource "google_compute_instance" "vm_instance" {
    can_ip_forward       = false
    cpu_platform         = "Intel Broadwell"
    current_status       = "RUNNING"
    deletion_protection  = false
    effective_labels     = {}
    enable_display       = false
    guest_accelerator    = []
    id                   = "projects/qwiklabs-gcp-03-8f557a4564b9/zones/us-central1-b/instances/mynet-second-vm"
    instance_id          = "5946476632090091540"
    label_fingerprint    = "42WmSpB8rSM="
    machine_type         = "e2-standard-2"
    metadata_fingerprint = "7QFr01P_g1k="
    name                 = "mynet-second-vm"
    project              = "qwiklabs-gcp-03-8f557a4564b9"
    self_link            = "https://www.googleapis.com/compute/v1/projects/qwiklabs-gcp-03-8f557a4564b9/zones/us-central1-b/instances/mynet-second-vm"
    tags_fingerprint     = "42WmSpB8rSM="
    terraform_labels     = {}
    zone                 = "us-central1-b"

    boot_disk {
        auto_delete = true
        device_name = "persistent-disk-0"
        mode        = "READ_WRITE"
        source      = "https://www.googleapis.com/compute/v1/projects/qwiklabs-gcp-03-8f557a4564b9/zones/us-central1-b/disks/mynet-second-vm"

        initialize_params {
            enable_confidential_compute = false
            image                       = "https://www.googleapis.com/compute/v1/projects/debian-cloud/global/images/debian-11-bullseye-v20240515"
            labels                      = {}
            provisioned_iops            = 0
            provisioned_throughput      = 0
            size                        = 10
            type                        = "pd-standard"
        }
    }

    network_interface {
        internal_ipv6_prefix_length = 0
        name                        = "nic0"
        network                     = "https://www.googleapis.com/compute/v1/projects/qwiklabs-gcp-03-8f557a4564b9/global/networks/mynetwork"
        network_ip                  = "10.128.0.2"
        queue_count                 = 0
        stack_type                  = "IPV4_ONLY"
        subnetwork                  = "https://www.googleapis.com/compute/v1/projects/qwiklabs-gcp-03-8f557a4564b9/regions/us-central1/subnetworks/mynetwork"
        subnetwork_project          = "qwiklabs-gcp-03-8f557a4564b9"

        access_config {
            nat_ip       = "35.192.130.110"
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


# module.mynet-us-vm.google_compute_instance.vm_instance:
resource "google_compute_instance" "vm_instance" {
    can_ip_forward       = false
    cpu_platform         = "Intel Broadwell"
    current_status       = "RUNNING"
    deletion_protection  = false
    effective_labels     = {}
    enable_display       = false
    guest_accelerator    = []
    id                   = "projects/qwiklabs-gcp-03-8f557a4564b9/zones/us-west1-c/instances/mynet-us-vm"
    instance_id          = "2026587906631802900"
    label_fingerprint    = "42WmSpB8rSM="
    machine_type         = "e2-standard-2"
    metadata_fingerprint = "7QFr01P_g1k="
    name                 = "mynet-us-vm"
    project              = "qwiklabs-gcp-03-8f557a4564b9"
    self_link            = "https://www.googleapis.com/compute/v1/projects/qwiklabs-gcp-03-8f557a4564b9/zones/us-west1-c/instances/mynet-us-vm"
    tags_fingerprint     = "42WmSpB8rSM="
    terraform_labels     = {}
    zone                 = "us-west1-c"

    boot_disk {
        auto_delete = true
        device_name = "persistent-disk-0"
        mode        = "READ_WRITE"
        source      = "https://www.googleapis.com/compute/v1/projects/qwiklabs-gcp-03-8f557a4564b9/zones/us-west1-c/disks/mynet-us-vm"

        initialize_params {
            enable_confidential_compute = false
            image                       = "https://www.googleapis.com/compute/v1/projects/debian-cloud/global/images/debian-11-bullseye-v20240515"
            labels                      = {}
            provisioned_iops            = 0
            provisioned_throughput      = 0
            size                        = 10
            type                        = "pd-standard"
        }
    }

    network_interface {
        internal_ipv6_prefix_length = 0
        name                        = "nic0"
        network                     = "https://www.googleapis.com/compute/v1/projects/qwiklabs-gcp-03-8f557a4564b9/global/networks/mynetwork"
        network_ip                  = "10.138.0.2"
        queue_count                 = 0
        stack_type                  = "IPV4_ONLY"
        subnetwork                  = "https://www.googleapis.com/compute/v1/projects/qwiklabs-gcp-03-8f557a4564b9/regions/us-west1/subnetworks/mynetwork"
        subnetwork_project          = "qwiklabs-gcp-03-8f557a4564b9"

        access_config {
            nat_ip       = "34.82.50.79"
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


# module.privatenet-us-vm.google_compute_instance.vm_instance:
resource "google_compute_instance" "vm_instance" {
    can_ip_forward       = false
    cpu_platform         = "Intel Broadwell"
    current_status       = "RUNNING"
    deletion_protection  = false
    effective_labels     = {}
    enable_display       = false
    guest_accelerator    = []
    id                   = "projects/qwiklabs-gcp-03-8f557a4564b9/zones/us-west1-c/instances/privatenet-us-vm"
    instance_id          = "98529172802249588"
    label_fingerprint    = "42WmSpB8rSM="
    labels               = {}
    machine_type         = "e2-standard-2"
    metadata             = {}
    metadata_fingerprint = "7QFr01P_g1k="
    name                 = "privatenet-us-vm"
    project              = "qwiklabs-gcp-03-8f557a4564b9"
    resource_policies    = []
    self_link            = "https://www.googleapis.com/compute/v1/projects/qwiklabs-gcp-03-8f557a4564b9/zones/us-west1-c/instances/privatenet-us-vm"
    tags                 = []
    tags_fingerprint     = "42WmSpB8rSM="
    terraform_labels     = {}
    zone                 = "us-west1-c"

    boot_disk {
        auto_delete = true
        device_name = "persistent-disk-0"
        mode        = "READ_WRITE"
        source      = "https://www.googleapis.com/compute/v1/projects/qwiklabs-gcp-03-8f557a4564b9/zones/us-west1-c/disks/privatenet-us-vm"

        initialize_params {
            enable_confidential_compute = false
            image                       = "https://www.googleapis.com/compute/v1/projects/debian-cloud/global/images/debian-11-bullseye-v20240515"
            labels                      = {}
            provisioned_iops            = 0
            provisioned_throughput      = 0
            resource_manager_tags       = {}
            size                        = 10
            type                        = "pd-standard"
        }
    }

    network_interface {
        internal_ipv6_prefix_length = 0
        name                        = "nic0"
        network                     = "https://www.googleapis.com/compute/v1/projects/qwiklabs-gcp-03-8f557a4564b9/global/networks/privatenet"
        network_ip                  = "172.16.0.2"
        queue_count                 = 0
        stack_type                  = "IPV4_ONLY"
        subnetwork                  = "https://www.googleapis.com/compute/v1/projects/qwiklabs-gcp-03-8f557a4564b9/regions/us-west1/subnetworks/privatesubnet-us"
        subnetwork_project          = "qwiklabs-gcp-03-8f557a4564b9"

        access_config {
            nat_ip       = "34.168.162.35"
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
student_01_aa8f3a74b714@cloudshell:~/tfnet (qwiklabs-gcp-03-8f557a4564b9)$ 
```

