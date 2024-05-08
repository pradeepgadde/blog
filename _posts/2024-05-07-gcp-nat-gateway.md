---

layout: single
title:  "Using a NAT Gateway with Kubernetes Engine"
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

# Using a NAT Gateway with Kubernetes Engine

This lab uses the Modular NAT Gateway on Compute Engine for Terraform to automate creation of a NAT gateway managed instance group. You  direct traffic from the instances by using tag-based routing, although  only instances with matching tags use the NAT gateway route.

Under normal circumstances, Kubernetes Engine nodes route all egress  traffic through the internet gateway associated with their node cluster. The internet gateway connection, in turn, is defined by the Compute  Engine network associated with the node cluster. Each node in the  cluster has an ephemeral external IP address. When nodes are created and destroyed during autoscaling, new node IP addresses are allocated  automatically.

The default gateway behavior works well under normal circumstances.  However, you might want to modify how ephemeral external IP addresses  are allocated in order to:

- Provide a third-party service with a consistent external IP address.
- Monitor and filter egress traffic out of the Kubernetes Engine cluster.

In this lab, you will learn how to:

- Create a NAT gateway instance and configure its routing details for an existing Kubernetes Engine cluster.
- Create a custom routing rule for the NAT gateway instance.

## Creating the NAT gateway with Terraform

[Compute Engine routes](https://cloud.google.com/compute/docs/reference/beta/routes) have a default priority of 1000, with lower numbers indicating higher  priority. The Terraform module creates a Compute Engine route with  priority 800, redirecting all outbound traffic from the Kubernetes  Engine nodes to the NAT gateway instance instead of using the default  internet gateway. The example code in the module also creates a static  route with priority 700, redirecting traffic from the Kubernetes Engine  nodes to the Kubernetes Engine master, which preserves normal cluster  operation by splitting the egress traffic.

After the NAT gateway instance is up and running, the startup script  configures IP forwarding and adds the firewall rules needed to perform  address translation.

## Setting up The GKE Example

```sh
Welcome to Cloud Shell! Type "help" to get started.
Your Cloud Platform project in this session is set to qwiklabs-gcp-01-f3cb8e405444.
Use “gcloud config set project [PROJECT_ID]” to change to a different project.
student_01_7c431b95ba82@cloudshell:~ (qwiklabs-gcp-01-f3cb8e405444)$ touch variables.tf
student_01_7c431b95ba82@cloudshell:~ (qwiklabs-gcp-01-f3cb8e405444)$ vi variables.tf 
student_01_7c431b95ba82@cloudshell:~ (qwiklabs-gcp-01-f3cb8e405444)$ cat variables.tf 
variable "project_id" {
  description = "The project ID to deploy to"
  default="qwiklabs-gcp-01-f3cb8e405444"
  version      = "~> 3.3.0"
}

variable "network" {
  description = "The VPC network self link"
  default="default"
}

variable "subnet" {
  description = "The subnet self link"
  default="default"
}
student_01_7c431b95ba82@cloudshell:~ (qwiklabs-gcp-01-f3cb8e405444)$ touch main.tf
student_01_7c431b95ba82@cloudshell:~ (qwiklabs-gcp-01-f3cb8e405444)$ vi main.tf 
student_01_7c431b95ba82@cloudshell:~ (qwiklabs-gcp-01-f3cb8e405444)$ cat main.tf 
module "test-vpc-module" {
  source       = "terraform-google-modules/network/google"
  version      = "~> 3.3.0"
  project_id   = var.project_id # Replace this with your project ID in quotes
  network_name = "custom-network1"
  mtu          = 1460

  subnets = [
    {
      subnet_name   = "subnet-us-east-192"
      subnet_ip     = "192.168.1.0/24"
      subnet_region = "us-east4"
    }
  ]
}
student_01_7c431b95ba82@cloudshell:~ (qwiklabs-gcp-01-f3cb8e405444)$ 
```

```sh
Welcome to Cloud Shell! Type "help" to get started.
Your Cloud Platform project in this session is set to qwiklabs-gcp-01-f3cb8e405444.
Use “gcloud config set project [PROJECT_ID]” to change to a different project.
student_01_7c431b95ba82@cloudshell:~ (qwiklabs-gcp-01-f3cb8e405444)$ ls
main.tf  README-cloudshell.txt  variables.tf
student_01_7c431b95ba82@cloudshell:~ (qwiklabs-gcp-01-f3cb8e405444)$ terraform init

Initializing the backend...
Initializing modules...
Downloading registry.terraform.io/terraform-google-modules/network/google 3.3.0 for test-vpc-module...
- test-vpc-module in .terraform/modules/test-vpc-module
- test-vpc-module.firewall_rules in .terraform/modules/test-vpc-module/modules/firewall-rules
- test-vpc-module.routes in .terraform/modules/test-vpc-module/modules/routes
- test-vpc-module.subnets in .terraform/modules/test-vpc-module/modules/subnets
- test-vpc-module.vpc in .terraform/modules/test-vpc-module/modules/vpc

Initializing provider plugins...
- Finding hashicorp/google versions matching ">= 2.12.0, ~> 3.45, < 4.0.0"...
- Finding hashicorp/google-beta versions matching "~> 3.45"...
- Installing hashicorp/google-beta v3.90.1...
- Installed hashicorp/google-beta v3.90.1 (signed by HashiCorp)
- Installing hashicorp/google v3.90.1...
- Installed hashicorp/google v3.90.1 (signed by HashiCorp)

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
student_01_7c431b95ba82@cloudshell:~ (qwiklabs-gcp-01-f3cb8e405444)$ terraform apply

Terraform used the selected providers to generate the following execution plan. Resource actions are indicated with the following symbols:
  + create

Terraform will perform the following actions:

  # module.test-vpc-module.module.subnets.google_compute_subnetwork.subnetwork["us-east4/subnet-us-east-192"] will be created
  + resource "google_compute_subnetwork" "subnetwork" {
      + creation_timestamp         = (known after apply)
      + external_ipv6_prefix       = (known after apply)
      + fingerprint                = (known after apply)
      + gateway_address            = (known after apply)
      + id                         = (known after apply)
      + ip_cidr_range              = "192.168.1.0/24"
      + ipv6_cidr_range            = (known after apply)
      + name                       = "subnet-us-east-192"
      + network                    = "custom-network1"
      + private_ip_google_access   = false
      + private_ipv6_google_access = (known after apply)
      + project                    = "qwiklabs-gcp-01-f3cb8e405444"
      + purpose                    = (known after apply)
      + region                     = "us-east4"
      + secondary_ip_range         = []
      + self_link                  = (known after apply)
      + stack_type                 = (known after apply)
    }

  # module.test-vpc-module.module.vpc.google_compute_network.network will be created
  + resource "google_compute_network" "network" {
      + auto_create_subnetworks         = false
      + delete_default_routes_on_create = false
      + gateway_ipv4                    = (known after apply)
      + id                              = (known after apply)
      + mtu                             = 1460
      + name                            = "custom-network1"
      + project                         = "qwiklabs-gcp-01-f3cb8e405444"
      + routing_mode                    = "GLOBAL"
      + self_link                       = (known after apply)
    }

Plan: 2 to add, 0 to change, 0 to destroy.

Do you want to perform these actions?
  Terraform will perform the actions described above.
  Only 'yes' will be accepted to approve.

  Enter a value: yes

module.test-vpc-module.module.vpc.google_compute_network.network: Creating...
module.test-vpc-module.module.vpc.google_compute_network.network: Still creating... [10s elapsed]
module.test-vpc-module.module.vpc.google_compute_network.network: Still creating... [20s elapsed]
module.test-vpc-module.module.vpc.google_compute_network.network: Creation complete after 23s [id=projects/qwiklabs-gcp-01-f3cb8e405444/global/networks/custom-network1]
module.test-vpc-module.module.subnets.google_compute_subnetwork.subnetwork["us-east4/subnet-us-east-192"]: Creating...
module.test-vpc-module.module.subnets.google_compute_subnetwork.subnetwork["us-east4/subnet-us-east-192"]: Still creating... [10s elapsed]
module.test-vpc-module.module.subnets.google_compute_subnetwork.subnetwork["us-east4/subnet-us-east-192"]: Creation complete after 17s [id=projects/qwiklabs-gcp-01-f3cb8e405444/regions/us-east4/subnetworks/subnet-us-east-192]

Apply complete! Resources: 2 added, 0 changed, 0 destroyed.
```

## Create a Private Cluster

```sh
student_01_7c431b95ba82@cloudshell:~ (qwiklabs-gcp-01-f3cb8e405444)$ cat main.tf 
module "test-vpc-module" {
  source       = "terraform-google-modules/network/google"
  version      = "~> 3.3.0"
  project_id   = var.project_id # Replace this with your project ID in quotes
  network_name = "custom-network1"
  mtu          = 1460

  subnets = [
    {
      subnet_name   = "subnet-us-east-192"
      subnet_ip     = "192.168.1.0/24"
      subnet_region = "us-east4"
    }
  ]
}
resource "google_container_cluster" "primary" {
  project            = var.project_id
  name               = "nat-test-cluster"
  location           = "us-east4-c"
  initial_node_count = 3
  network            = var.network # Replace with a reference or self link to your network, in quotes
  subnetwork         = var.subnet  # Replace with a reference or self link to your subnet, in quotes
  private_cluster_config {
    master_ipv4_cidr_block  = "172.16.0.0/28"
    enable_private_endpoint = true
    enable_private_nodes    = true
  }
  ip_allocation_policy {
  }
  master_authorized_networks_config {
  }
}
student_01_7c431b95ba82@cloudshell:~ (qwiklabs-gcp-01-f3cb8e405444)$ 
```

```sh
student_01_7c431b95ba82@cloudshell:~ (qwiklabs-gcp-01-f3cb8e405444)$ terraform apply
module.test-vpc-module.module.vpc.google_compute_network.network: Refreshing state... [id=projects/qwiklabs-gcp-01-f3cb8e405444/global/networks/custom-network1]
module.test-vpc-module.module.subnets.google_compute_subnetwork.subnetwork["us-east4/subnet-us-east-192"]: Refreshing state... [id=projects/qwiklabs-gcp-01-f3cb8e405444/regions/us-east4/subnetworks/subnet-us-east-192]

Terraform used the selected providers to generate the following execution plan. Resource actions are indicated with the following symbols:
  + create

Terraform will perform the following actions:

  # google_container_cluster.primary will be created
  + resource "google_container_cluster" "primary" {
      + cluster_ipv4_cidr           = (known after apply)
      + datapath_provider           = (known after apply)
      + default_max_pods_per_node   = (known after apply)
      + enable_binary_authorization = false
      + enable_intranode_visibility = (known after apply)
      + enable_kubernetes_alpha     = false
      + enable_legacy_abac          = false
      + enable_shielded_nodes       = (known after apply)
      + endpoint                    = (known after apply)
      + id                          = (known after apply)
      + initial_node_count          = 3
      + instance_group_urls         = (known after apply)
      + label_fingerprint           = (known after apply)
      + location                    = "us-east4-c"
      + logging_service             = (known after apply)
      + master_version              = (known after apply)
      + monitoring_service          = (known after apply)
      + name                        = "nat-test-cluster"
      + network                     = "default"
      + networking_mode             = (known after apply)
      + node_locations              = (known after apply)
      + node_version                = (known after apply)
      + operation                   = (known after apply)
      + private_ipv6_google_access  = (known after apply)
      + project                     = "qwiklabs-gcp-01-f3cb8e405444"
      + self_link                   = (known after apply)
      + services_ipv4_cidr          = (known after apply)
      + subnetwork                  = "default"
      + tpu_ipv4_cidr_block         = (known after apply)

      + ip_allocation_policy {
          + cluster_ipv4_cidr_block       = (known after apply)
          + cluster_secondary_range_name  = (known after apply)
          + services_ipv4_cidr_block      = (known after apply)
          + services_secondary_range_name = (known after apply)
        }

      + master_authorized_networks_config {}

      + private_cluster_config {
          + enable_private_endpoint = true
          + enable_private_nodes    = true
          + master_ipv4_cidr_block  = "172.16.0.0/28"
          + peering_name            = (known after apply)
          + private_endpoint        = (known after apply)
          + public_endpoint         = (known after apply)
        }
    }

Plan: 1 to add, 0 to change, 0 to destroy.

Do you want to perform these actions?
  Terraform will perform the actions described above.
  Only 'yes' will be accepted to approve.

  Enter a value: yes

google_container_cluster.primary: Creating...
google_container_cluster.primary: Still creating... [10s elapsed]
google_container_cluster.primary: Still creating... [20s elapsed]
google_container_cluster.primary: Still creating... [30s elapsed]
google_container_cluster.primary: Still creating... [40s elapsed]
google_container_cluster.primary: Still creating... [50s elapsed]
google_container_cluster.primary: Still creating... [1m0s elapsed]
google_container_cluster.primary: Still creating... [1m10s elapsed]
google_container_cluster.primary: Still creating... [1m20s elapsed]
google_container_cluster.primary: Still creating... [1m30s elapsed]
google_container_cluster.primary: Still creating... [1m40s elapsed]
google_container_cluster.primary: Still creating... [1m50s elapsed]
google_container_cluster.primary: Still creating... [2m0s elapsed]
google_container_cluster.primary: Still creating... [2m10s elapsed]
google_container_cluster.primary: Still creating... [2m20s elapsed]
google_container_cluster.primary: Still creating... [2m30s elapsed]
google_container_cluster.primary: Still creating... [2m40s elapsed]
google_container_cluster.primary: Still creating... [2m50s elapsed]
google_container_cluster.primary: Still creating... [3m0s elapsed]
google_container_cluster.primary: Still creating... [3m10s elapsed]
google_container_cluster.primary: Still creating... [3m20s elapsed]
google_container_cluster.primary: Still creating... [3m30s elapsed]
google_container_cluster.primary: Still creating... [3m40s elapsed]
google_container_cluster.primary: Still creating... [3m50s elapsed]
google_container_cluster.primary: Still creating... [4m0s elapsed]
google_container_cluster.primary: Still creating... [4m10s elapsed]
google_container_cluster.primary: Still creating... [4m20s elapsed]
google_container_cluster.primary: Still creating... [4m30s elapsed]
google_container_cluster.primary: Still creating... [4m40s elapsed]
google_container_cluster.primary: Still creating... [4m50s elapsed]
google_container_cluster.primary: Still creating... [5m0s elapsed]
google_container_cluster.primary: Still creating... [5m10s elapsed]
google_container_cluster.primary: Still creating... [5m20s elapsed]
google_container_cluster.primary: Still creating... [5m30s elapsed]
google_container_cluster.primary: Still creating... [5m40s elapsed]
google_container_cluster.primary: Still creating... [5m50s elapsed]
google_container_cluster.primary: Still creating... [6m0s elapsed]
google_container_cluster.primary: Still creating... [6m10s elapsed]
google_container_cluster.primary: Still creating... [6m20s elapsed]
google_container_cluster.primary: Still creating... [6m30s elapsed]
google_container_cluster.primary: Still creating... [6m40s elapsed]
google_container_cluster.primary: Still creating... [6m50s elapsed]
google_container_cluster.primary: Still creating... [7m0s elapsed]
google_container_cluster.primary: Still creating... [7m10s elapsed]
google_container_cluster.primary: Creation complete after 7m19s [id=projects/qwiklabs-gcp-01-f3cb8e405444/locations/us-east4-c/clusters/nat-test-cluster]

Apply complete! Resources: 1 added, 0 changed, 0 destroyed.
student_01_7c431b95ba82@cloudshell:~ (qwiklabs-gcp-01-f3cb8e405444)$ 
```

## Create a Firewall Rule that Allows SSH Connections

```sh
student_01_7c431b95ba82@cloudshell:~ (qwiklabs-gcp-01-f3cb8e405444)$ cat main.tf 
module "test-vpc-module" {
  source       = "terraform-google-modules/network/google"
  version      = "~> 3.3.0"
  project_id   = var.project_id # Replace this with your project ID in quotes
  network_name = "custom-network1"
  mtu          = 1460

  subnets = [
    {
      subnet_name   = "subnet-us-east-192"
      subnet_ip     = "192.168.1.0/24"
      subnet_region = "us-east4"
    }
  ]
}
resource "google_container_cluster" "primary" {
  project            = var.project_id
  name               = "nat-test-cluster"
  location           = "us-east4-c"
  initial_node_count = 3
  network            = var.network # Replace with a reference or self link to your network, in quotes
  subnetwork         = var.subnet  # Replace with a reference or self link to your subnet, in quotes
  private_cluster_config {
    master_ipv4_cidr_block  = "172.16.0.0/28"
    enable_private_endpoint = true
    enable_private_nodes    = true
  }
  ip_allocation_policy {
  }
  master_authorized_networks_config {
  }
}
resource "google_compute_firewall" "rules" {
  project = var.project_id
  name    = "allow-ssh"
  network = var.network
  allow {
    protocol = "tcp"
    ports    = ["22"]
  }
  source_ranges = ["35.235.240.0/20"]
}
student_01_7c431b95ba82@cloudshell:~ (qwiklabs-gcp-01-f3cb8e405444)$ 
```

```sh
student_01_7c431b95ba82@cloudshell:~ (qwiklabs-gcp-01-f3cb8e405444)$ terraform apply
module.test-vpc-module.module.vpc.google_compute_network.network: Refreshing state... [id=projects/qwiklabs-gcp-01-f3cb8e405444/global/networks/custom-network1]
google_container_cluster.primary: Refreshing state... [id=projects/qwiklabs-gcp-01-f3cb8e405444/locations/us-east4-c/clusters/nat-test-cluster]
module.test-vpc-module.module.subnets.google_compute_subnetwork.subnetwork["us-east4/subnet-us-east-192"]: Refreshing state... [id=projects/qwiklabs-gcp-01-f3cb8e405444/regions/us-east4/subnetworks/subnet-us-east-192]

Terraform used the selected providers to generate the following execution plan. Resource actions are indicated with the following symbols:
  + create

Terraform will perform the following actions:

  # google_compute_firewall.rules will be created
  + resource "google_compute_firewall" "rules" {
      + creation_timestamp = (known after apply)
      + destination_ranges = (known after apply)
      + direction          = (known after apply)
      + enable_logging     = (known after apply)
      + id                 = (known after apply)
      + name               = "allow-ssh"
      + network            = "default"
      + priority           = 1000
      + project            = "qwiklabs-gcp-01-f3cb8e405444"
      + self_link          = (known after apply)
      + source_ranges      = [
          + "35.235.240.0/20",
        ]

      + allow {
          + ports    = [
              + "22",
            ]
          + protocol = "tcp"
        }
    }

Plan: 1 to add, 0 to change, 0 to destroy.

Do you want to perform these actions?
  Terraform will perform the actions described above.
  Only 'yes' will be accepted to approve.

  Enter a value: yes

google_compute_firewall.rules: Creating...
google_compute_firewall.rules: Still creating... [10s elapsed]
google_compute_firewall.rules: Still creating... [20s elapsed]
google_compute_firewall.rules: Creation complete after 23s [id=projects/qwiklabs-gcp-01-f3cb8e405444/global/firewalls/allow-ssh]

Apply complete! Resources: 1 added, 0 changed, 0 destroyed.
student_01_7c431b95ba82@cloudshell:~ (qwiklabs-gcp-01-f3cb8e405444)$ 
```

## Create IAP SSH permissions for one of your nodes

> Identity-Aware Proxy
>
>  The Identity-Aware Proxy(Cloud IAP) controls access to your cloud applications and VMs running on Google Cloud Platform(GCP). [ ](https://cloud.google.com/iap/docs)Identity-Aware Proxy (IAP) lets you manage who has access to services hosted on App Engine,  Compute Engine, or an HTTPS Load Balancer.

1. In the Cloud Console, navigate to **IAM & Admin** > **Identity-Aware Proxy** from the navigation bar.

2. Click **Enable API**.

3. Click **Go to Identity-Aware Proxy**.

4. Select the **SSH and TCP Resources** tab.

5. Select the checkbox next to the first node in the list under **All Tunnel Resources** > **us-east4-c**. Its name will be similar to `gke-nat-test-cluster-default-pool-b50db58d-075t`.

6. In the right pane, click **Add principal**.

7. In the **New principals** field, enter your student email address that you logged into the lab with

8. Grant yourself access to the resources through Cloud IAP's TCP forwarding feature, in the Role drop-down list, select **Cloud IAP** > **IAP-secured Tunnel User**.

   Click **Save**.

## Log in to the node and confirm that it cannot reach the internet

```sh
student_01_7c431b95ba82@cloudshell:~ (qwiklabs-gcp-01-f3cb8e405444)$ export NODE_NAME=gke-nat-test-cluster-default-pool-27c73a05-7m27 
student_01_7c431b95ba82@cloudshell:~ (qwiklabs-gcp-01-f3cb8e405444)$ gcloud config set project qwiklabs-gcp-01-f3cb8e405444
Updated property [core/project].
student_01_7c431b95ba82@cloudshell:~ (qwiklabs-gcp-01-f3cb8e405444)$ gcloud compute ssh $NODE_NAME \
    --zone us-east4-c \
    --tunnel-through-iap
WARNING: The private SSH key file for gcloud does not exist.
WARNING: The public SSH key file for gcloud does not exist.
WARNING: You do not have an SSH key for gcloud.
WARNING: SSH keygen will be executed to generate a key.
This tool needs to create the directory [/home/student_01_7c431b95ba82/.ssh] before being able to generate SSH keys.

Do you want to continue (Y/n)?  y

Generating public/private rsa key pair.
Enter passphrase (empty for no passphrase): 
Enter same passphrase again: 
Your identification has been saved in /home/student_01_7c431b95ba82/.ssh/google_compute_engine
Your public key has been saved in /home/student_01_7c431b95ba82/.ssh/google_compute_engine.pub
The key fingerprint is:
SHA256:4eGVC2nVv0TS0XOMYb7GQ/V5LmcyeXgnb3ChNMlzQbE student_01_7c431b95ba82@cs-572110841795-default
The key's randomart image is:
+---[RSA 3072]----+
|          .. .=Xo|
|         o .+o=oO|
|        * o  XoE=|
|       + = ..oB=o|
|        S .  o&.O|
|             ..#.|
|                o|
|               . |
|                 |
+----[SHA256]-----+
WARNING: 

To increase the performance of the tunnel, consider installing NumPy. For instructions,
please see https://cloud.google.com/iap/docs/using-tcp-forwarding#increasing_the_tcp_upload_bandwidth

Warning: Permanently added 'compute.1539977079881687124' (ED25519) to the list of known hosts.

Welcome to Kubernetes v1.28.7-gke.1000!

You can find documentation for Kubernetes at:
  http://docs.kubernetes.io/

The source for this release can be found at:
  /home/kubernetes/kubernetes-src.tar.gz
Or you can download it at:
  https://storage.googleapis.com/kubernetes-release-gke/release/v1.28.7-gke.1000/kubernetes-src.tar.gz

It is based on the Kubernetes source at:
  https://github.com/kubernetes/kubernetes/tree/v1.28.7-gke.1000

For Kubernetes copyright and licensing information, see:
  /home/kubernetes/LICENSES

Creating directory '/home/student-01-7c431b95ba82'.
student-01-7c431b95ba82@gke-nat-test-cluster-default-pool-27c73a05-7m27 ~ $ 
```

From the node prompt, attempt to connect to the internet, you should get no result.To end the command, you might have to enter `Ctrl+C`.

```sh
student-01-7c431b95ba82@gke-nat-test-cluster-default-pool-27c73a05-7m27 ~ $ curl example.com
^C
student-01-7c431b95ba82@gke-nat-test-cluster-default-pool-27c73a05-7m27 ~ $ 
```

## Create a NAT configuration using Cloud Router

You must create the Cloud Router in the same region as the instances  that use Cloud NAT. Cloud NAT is only used to place NAT information onto the VMs. It is not used as part of the actual NAT gateway.

 Google Cloud Router dynamically exchanges routes between your Virtual  Private Cloud (VPC) and on-premises networks by using Border Gateway  Protocol (BGP)

This configuration allows all instances in the region to use Cloud  NAT for all primary and alias IP ranges. It also automatically allocates the external IP addresses for the NAT gateway. For more options, see  the gcloud command-line interface documentation.

```sh
student_01_7c431b95ba82@cloudshell:~ (qwiklabs-gcp-01-f3cb8e405444)$ cat main.tf 
module "test-vpc-module" {
  source       = "terraform-google-modules/network/google"
  version      = "~> 3.3.0"
  project_id   = var.project_id # Replace this with your project ID in quotes
  network_name = "custom-network1"
  mtu          = 1460

  subnets = [
    {
      subnet_name   = "subnet-us-east-192"
      subnet_ip     = "192.168.1.0/24"
      subnet_region = "us-east4"
    }
  ]
}
resource "google_container_cluster" "primary" {
  project            = var.project_id
  name               = "nat-test-cluster"
  location           = "us-east4-c"
  initial_node_count = 3
  network            = var.network # Replace with a reference or self link to your network, in quotes
  subnetwork         = var.subnet  # Replace with a reference or self link to your subnet, in quotes
  private_cluster_config {
    master_ipv4_cidr_block  = "172.16.0.0/28"
    enable_private_endpoint = true
    enable_private_nodes    = true
  }
  ip_allocation_policy {
  }
  master_authorized_networks_config {
  }
}
resource "google_compute_firewall" "rules" {
  project = var.project_id
  name    = "allow-ssh"
  network = var.network
  allow {
    protocol = "tcp"
    ports    = ["22"]
  }
  source_ranges = ["35.235.240.0/20"]
}
resource "google_compute_router" "router" {
  project = var.project_id
  name    = "nat-router"
  network = var.network
  region  = "us-east4"
}
module "cloud-nat" {
  source                             = "terraform-google-modules/cloud-nat/google"
  version                            = "~> 2.0.0"
  project_id                         = var.project_id
  region                             = "us-east4"
  router                             = google_compute_router.router.name
  name                               = "nat-config"
  source_subnetwork_ip_ranges_to_nat = "ALL_SUBNETWORKS_ALL_IP_RANGES"
}
student_01_7c431b95ba82@cloudshell:~ (qwiklabs-gcp-01-f3cb8e405444)$ 
```

```sh
student_01_7c431b95ba82@cloudshell:~ (qwiklabs-gcp-01-f3cb8e405444)$ terraform init

Initializing the backend...
Initializing modules...
Downloading registry.terraform.io/terraform-google-modules/cloud-nat/google 2.0.0 for cloud-nat...
- cloud-nat in .terraform/modules/cloud-nat

Initializing provider plugins...
- Reusing previous version of hashicorp/google from the dependency lock file
- Finding latest version of hashicorp/random...
- Reusing previous version of hashicorp/google-beta from the dependency lock file
- Using previously-installed hashicorp/google v3.90.1
- Installing hashicorp/random v3.6.1...
- Installed hashicorp/random v3.6.1 (signed by HashiCorp)
- Using previously-installed hashicorp/google-beta v3.90.1

Terraform has made some changes to the provider dependency selections recorded
in the .terraform.lock.hcl file. Review those changes and commit them to your
version control system if they represent changes you intended to make.

Terraform has been successfully initialized!

You may now begin working with Terraform. Try running "terraform plan" to see
any changes that are required for your infrastructure. All Terraform commands
should now work.

If you ever set or change modules or backend configuration for Terraform,
rerun this command to reinitialize your working directory. If you forget, other
commands will detect it and remind you to do so if necessary.
student_01_7c431b95ba82@cloudshell:~ (qwiklabs-gcp-01-f3cb8e405444)$ 
```

```sh
student_01_7c431b95ba82@cloudshell:~ (qwiklabs-gcp-01-f3cb8e405444)$ terraform apply
google_compute_firewall.rules: Refreshing state... [id=projects/qwiklabs-gcp-01-f3cb8e405444/global/firewalls/allow-ssh]
module.test-vpc-module.module.vpc.google_compute_network.network: Refreshing state... [id=projects/qwiklabs-gcp-01-f3cb8e405444/global/networks/custom-network1]
google_container_cluster.primary: Refreshing state... [id=projects/qwiklabs-gcp-01-f3cb8e405444/locations/us-east4-c/clusters/nat-test-cluster]
module.test-vpc-module.module.subnets.google_compute_subnetwork.subnetwork["us-east4/subnet-us-east-192"]: Refreshing state... [id=projects/qwiklabs-gcp-01-f3cb8e405444/regions/us-east4/subnetworks/subnet-us-east-192]

Terraform used the selected providers to generate the following execution plan. Resource actions are indicated with the following symbols:
  + create

Terraform will perform the following actions:

  # google_compute_router.router will be created
  + resource "google_compute_router" "router" {
      + creation_timestamp = (known after apply)
      + id                 = (known after apply)
      + name               = "nat-router"
      + network            = "default"
      + project            = "qwiklabs-gcp-01-f3cb8e405444"
      + region             = "us-east4"
      + self_link          = (known after apply)
    }

  # module.cloud-nat.google_compute_router_nat.main will be created
  + resource "google_compute_router_nat" "main" {
      + enable_endpoint_independent_mapping = true
      + icmp_idle_timeout_sec               = 30
      + id                                  = (known after apply)
      + min_ports_per_vm                    = 64
      + name                                = "nat-config"
      + nat_ip_allocate_option              = "AUTO_ONLY"
      + project                             = "qwiklabs-gcp-01-f3cb8e405444"
      + region                              = "us-east4"
      + router                              = "nat-router"
      + source_subnetwork_ip_ranges_to_nat  = "ALL_SUBNETWORKS_ALL_IP_RANGES"
      + tcp_established_idle_timeout_sec    = 1200
      + tcp_transitory_idle_timeout_sec     = 30
      + udp_idle_timeout_sec                = 30
    }

  # module.cloud-nat.random_string.name_suffix will be created
  + resource "random_string" "name_suffix" {
      + id          = (known after apply)
      + length      = 6
      + lower       = true
      + min_lower   = 0
      + min_numeric = 0
      + min_special = 0
      + min_upper   = 0
      + number      = true
      + numeric     = true
      + result      = (known after apply)
      + special     = false
      + upper       = false
    }

Plan: 3 to add, 0 to change, 0 to destroy.

Do you want to perform these actions?
  Terraform will perform the actions described above.
  Only 'yes' will be accepted to approve.

  Enter a value: yes

module.cloud-nat.random_string.name_suffix: Creating...
module.cloud-nat.random_string.name_suffix: Creation complete after 0s [id=mhawl3]
google_compute_router.router: Creating...
google_compute_router.router: Creation complete after 5s [id=projects/qwiklabs-gcp-01-f3cb8e405444/regions/us-east4/routers/nat-router]
module.cloud-nat.google_compute_router_nat.main: Creating...
module.cloud-nat.google_compute_router_nat.main: Still creating... [10s elapsed]
module.cloud-nat.google_compute_router_nat.main: Creation complete after 16s [id=qwiklabs-gcp-01-f3cb8e405444/us-east4/nat-router/nat-config]

Apply complete! Resources: 3 added, 0 changed, 0 destroyed.
student_01_7c431b95ba82@cloudshell:~ (qwiklabs-gcp-01-f3cb8e405444)$ 
```

## Attempt to connect to the internet again

It might take up to three minutes for the NAT configuration to  propagate, so wait at least a minute before trying to access the  internet again.

```sh
student_01_7c431b95ba82@cloudshell:~ (qwiklabs-gcp-01-f3cb8e405444)$ gcloud compute ssh $NODE_NAME \
    --zone us-east4-c \
    --tunnel-through-iap
WARNING: 

To increase the performance of the tunnel, consider installing NumPy. For instructions,
please see https://cloud.google.com/iap/docs/using-tcp-forwarding#increasing_the_tcp_upload_bandwidth


Welcome to Kubernetes v1.28.7-gke.1000!

You can find documentation for Kubernetes at:
  http://docs.kubernetes.io/

The source for this release can be found at:
  /home/kubernetes/kubernetes-src.tar.gz
Or you can download it at:
  https://storage.googleapis.com/kubernetes-release-gke/release/v1.28.7-gke.1000/kubernetes-src.tar.gz

It is based on the Kubernetes source at:
  https://github.com/kubernetes/kubernetes/tree/v1.28.7-gke.1000

For Kubernetes copyright and licensing information, see:
  /home/kubernetes/LICENSES

student-01-7c431b95ba82@gke-nat-test-cluster-default-pool-27c73a05-7m27 ~ $ curl example.com
<!doctype html>
<html>
<head>
    <title>Example Domain</title>

    <meta charset="utf-8" />
    <meta http-equiv="Content-type" content="text/html; charset=utf-8" />
    <meta name="viewport" content="width=device-width, initial-scale=1" />
    <style type="text/css">
    body {
        background-color: #f0f0f2;
        margin: 0;
        padding: 0;
        font-family: -apple-system, system-ui, BlinkMacSystemFont, "Segoe UI", "Open Sans", "Helvetica Neue", Helvetica, Arial, sans-serif;
        
    }
    div {
        width: 600px;
        margin: 5em auto;
        padding: 2em;
        background-color: #fdfdff;
        border-radius: 0.5em;
        box-shadow: 2px 3px 7px 2px rgba(0,0,0,0.02);
    }
    a:link, a:visited {
        color: #38488f;
        text-decoration: none;
    }
    @media (max-width: 700px) {
        div {
            margin: 0 auto;
            width: auto;
        }
    }
    </style>    
</head>

<body>
<div>
    <h1>Example Domain</h1>
    <p>This domain is for use in illustrative examples in documents. You may use this
    domain in literature without prior coordination or asking for permission.</p>
    <p><a href="https://www.iana.org/domains/example">More information...</a></p>
</div>
</body>
</html>
student-01-7c431b95ba82@gke-nat-test-cluster-default-pool-27c73a05-7m27 ~ $ 
```

It should now successfully display the output of the example domain page!

