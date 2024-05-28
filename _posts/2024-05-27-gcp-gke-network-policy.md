---

layout: single
title:  "How to Use a Network Policy on Google Kubernetes Engine"
categories: Cloud
tags: GCP
classes: wide
show_date: true
header:
  overlay_image: /assets/images/gcp-banner.png
  og_image: /assets/images/gcp-banner.png
  teaser: /assets/images/pca-gcp.png
author:
  name     : "Professional Cloud Architect"
  avatar   : "/assets/images/pca-gcp.png"

sidebar:
  - title: "Topics"
    nav: my-sidebar

---
# How to Use a Network Policy on Google Kubernetes Engine

Network connections can be restricted at two tiers of your Kubernetes Engine infrastructure. The first, and coarser grained, mechanism is the application of Firewall Rules at the Network, Subnetwork, and Host levels. These rules are applied outside of the Kubernetes Engine at the VPC level.

While Firewall Rules are a powerful security measure, and Kubernetes enables you to define even finer grained rules via Network Policies. Network Policies are used to limit intra-cluster communication. Network policies do not apply to pods attached to the host's network namespace.

For this lab you will provision a private Kubernetes Engine cluster and a bastion host with which to access it. A bastion host provides a single host that has access to the cluster, which, when combined with a private Kubernetes network, ensures that the cluster isn't exposed to malicious behavior from the internet at large. Bastions are particularly useful when you do not have VPN access to the cloud network.

Within the cluster, a simple HTTP server and two client pods will be provisioned. You will learn how to use a Network Policy and labeling to only allow connections from one of the client pods.

Architecture

You will define a private, standard mode Kubernetes cluster that uses Dataplane V2. Dataplane V2 has network policies enabled by default.

Since the cluster is private, neither the API nor the worker nodes will be accessible from the internet. Instead, you will define a bastion host and use a firewall rule to enable access to it. The bastion's IP address is defined as an authorized network for the cluster, which grants it access to the API.

Within the cluster, provision three workloads:

- hello-server: this is a simple HTTP server with an internally-accessible endpoint
- hello-client-allowed: this is a single pod that repeatedly attempts to access hello-server. The pod is labeled such that the Network Policy will allow it to connect to hello-server.
- hello-client-blocked: this runs the same code as hello-client-allowed but the pod is labeled such that the Network Policy will not allow it to connect to hello-server.

### Clone demo

1. Copy the resources needed for this lab exercise from a Cloud Storage bucket:

```sh
Welcome to Cloud Shell! Type "help" to get started.
Your Cloud Platform project in this session is set to qwiklabs-gcp-01-37dcbcf4fa9d.
Use “gcloud config set project [PROJECT_ID]” to change to a different project.
student_01_41d0c1af2373@cloudshell:~ (qwiklabs-gcp-01-37dcbcf4fa9d)$ gsutil cp -r gs://spls/gsp480/gke-network-policy-demo .
Copying gs://spls/gsp480/gke-network-policy-demo/.DS_Store...
Copying gs://spls/gsp480/gke-network-policy-demo/.git/HEAD...                   
Copying gs://spls/gsp480/gke-network-policy-demo/.git/config...                 
Copying gs://spls/gsp480/gke-network-policy-demo/.git/description...            
\ [4 files][  6.4 KiB/  6.4 KiB]                                                
==> NOTE: You are performing a sequence of gsutil operations that may
run significantly faster if you instead use gsutil -m cp ... Please
see the -m section under "gsutil help options" for further information
about when gsutil -m can be advantageous.

Copying gs://spls/gsp480/gke-network-policy-demo/.git/hooks/applypatch-msg.sample...
Copying gs://spls/gsp480/gke-network-policy-demo/.git/hooks/commit-msg.sample...
Copying gs://spls/gsp480/gke-network-policy-demo/.git/hooks/fsmonitor-watchman.sample...
Copying gs://spls/gsp480/gke-network-policy-demo/.git/hooks/post-update.sample...
Copying gs://spls/gsp480/gke-network-policy-demo/.git/hooks/pre-applypatch.sample...
Copying gs://spls/gsp480/gke-network-policy-demo/.git/hooks/pre-commit.sample...
Copying gs://spls/gsp480/gke-network-policy-demo/.git/hooks/pre-merge-commit.sample...
Copying gs://spls/gsp480/gke-network-policy-demo/.git/hooks/pre-push.sample...  
Copying gs://spls/gsp480/gke-network-policy-demo/.git/hooks/pre-rebase.sample...
Copying gs://spls/gsp480/gke-network-policy-demo/.git/hooks/pre-receive.sample...
Copying gs://spls/gsp480/gke-network-policy-demo/.git/hooks/prepare-commit-msg.sample...
Copying gs://spls/gsp480/gke-network-policy-demo/.git/hooks/push-to-checkout.sample...
Copying gs://spls/gsp480/gke-network-policy-demo/.git/hooks/sendemail-validate.sample...
Copying gs://spls/gsp480/gke-network-policy-demo/.git/hooks/update.sample...    
Copying gs://spls/gsp480/gke-network-policy-demo/.git/index...                  
Copying gs://spls/gsp480/gke-network-policy-demo/.git/info/exclude...           
Copying gs://spls/gsp480/gke-network-policy-demo/.git/logs/HEAD...              
Copying gs://spls/gsp480/gke-network-policy-demo/.git/logs/refs/heads/master... 
Copying gs://spls/gsp480/gke-network-policy-demo/.git/logs/refs/remotes/origin/HEAD...
Copying gs://spls/gsp480/gke-network-policy-demo/.git/objects/pack/pack-b7907d21f0645686e5be99b23204f9c1881ddfcc.idx...
Copying gs://spls/gsp480/gke-network-policy-demo/.git/objects/pack/pack-b7907d21f0645686e5be99b23204f9c1881ddfcc.pack...
Copying gs://spls/gsp480/gke-network-policy-demo/.git/objects/pack/pack-b7907d21f0645686e5be99b23204f9c1881ddfcc.rev...
Copying gs://spls/gsp480/gke-network-policy-demo/.git/packed-refs...            
Copying gs://spls/gsp480/gke-network-policy-demo/.git/refs/heads/master...      
Copying gs://spls/gsp480/gke-network-policy-demo/.git/refs/remotes/origin/HEAD...
Copying gs://spls/gsp480/gke-network-policy-demo/.gitignore...                  
Copying gs://spls/gsp480/gke-network-policy-demo/CONTRIBUTING.md...             
Copying gs://spls/gsp480/gke-network-policy-demo/Jenkinsfile...                 
Copying gs://spls/gsp480/gke-network-policy-demo/LICENSE...                     
Copying gs://spls/gsp480/gke-network-policy-demo/Makefile...                    
Copying gs://spls/gsp480/gke-network-policy-demo/OWNERS...                      
Copying gs://spls/gsp480/gke-network-policy-demo/README-QWIKLABS.md...          
Copying gs://spls/gsp480/gke-network-policy-demo/README.md...                   
Copying gs://spls/gsp480/gke-network-policy-demo/common.sh...                   
Copying gs://spls/gsp480/gke-network-policy-demo/create.sh...                   
Copying gs://spls/gsp480/gke-network-policy-demo/enable-apis.sh...              
Copying gs://spls/gsp480/gke-network-policy-demo/generate-tfvars.sh...          
Copying gs://spls/gsp480/gke-network-policy-demo/img/architecture.png...        
Copying gs://spls/gsp480/gke-network-policy-demo/img/terraform_fingerprint_error.png...
Copying gs://spls/gsp480/gke-network-policy-demo/manifests/hello-app/hello-client.yaml...
Copying gs://spls/gsp480/gke-network-policy-demo/manifests/hello-app/hello-server.yaml...
Copying gs://spls/gsp480/gke-network-policy-demo/manifests/network-policy-namespaced.yaml...
Copying gs://spls/gsp480/gke-network-policy-demo/manifests/network-policy.yaml...
Copying gs://spls/gsp480/gke-network-policy-demo/setup_manifests.sh...          
Copying gs://spls/gsp480/gke-network-policy-demo/teardown.sh...                 
Copying gs://spls/gsp480/gke-network-policy-demo/terraform/bastion.tf...        
Copying gs://spls/gsp480/gke-network-policy-demo/terraform/firewall.tf...       
Copying gs://spls/gsp480/gke-network-policy-demo/terraform/main.tf...           
Copying gs://spls/gsp480/gke-network-policy-demo/terraform/network.tf...        
Copying gs://spls/gsp480/gke-network-policy-demo/terraform/provider.tf...       
Copying gs://spls/gsp480/gke-network-policy-demo/terraform/variables.tf...      
Copying gs://spls/gsp480/gke-network-policy-demo/terraform/versions.tf...       
Copying gs://spls/gsp480/gke-network-policy-demo/test/boilerplate/boilerplate.BUILD.txt...
Copying gs://spls/gsp480/gke-network-policy-demo/test/boilerplate/boilerplate.Dockerfile.txt...
Copying gs://spls/gsp480/gke-network-policy-demo/test/boilerplate/boilerplate.Makefile.txt...
Copying gs://spls/gsp480/gke-network-policy-demo/test/boilerplate/boilerplate.WORKSPACE.txt...
Copying gs://spls/gsp480/gke-network-policy-demo/test/boilerplate/boilerplate.bazel.txt...
Copying gs://spls/gsp480/gke-network-policy-demo/test/boilerplate/boilerplate.bzl.txt...
Copying gs://spls/gsp480/gke-network-policy-demo/test/boilerplate/boilerplate.css.txt...
Copying gs://spls/gsp480/gke-network-policy-demo/test/boilerplate/boilerplate.go.preamble...
Copying gs://spls/gsp480/gke-network-policy-demo/test/boilerplate/boilerplate.go.txt...
Copying gs://spls/gsp480/gke-network-policy-demo/test/boilerplate/boilerplate.html.preamble...
Copying gs://spls/gsp480/gke-network-policy-demo/test/boilerplate/boilerplate.html.txt...
Copying gs://spls/gsp480/gke-network-policy-demo/test/boilerplate/boilerplate.java.txt...
Copying gs://spls/gsp480/gke-network-policy-demo/test/boilerplate/boilerplate.js.txt...
Copying gs://spls/gsp480/gke-network-policy-demo/test/boilerplate/boilerplate.py.preamble...
Copying gs://spls/gsp480/gke-network-policy-demo/test/boilerplate/boilerplate.py.txt...
Copying gs://spls/gsp480/gke-network-policy-demo/test/boilerplate/boilerplate.scss.txt...
Copying gs://spls/gsp480/gke-network-policy-demo/test/boilerplate/boilerplate.sh.preamble...
Copying gs://spls/gsp480/gke-network-policy-demo/test/boilerplate/boilerplate.sh.txt...
Copying gs://spls/gsp480/gke-network-policy-demo/test/boilerplate/boilerplate.tf.txt...
Copying gs://spls/gsp480/gke-network-policy-demo/test/boilerplate/boilerplate.ts.txt...
Copying gs://spls/gsp480/gke-network-policy-demo/test/boilerplate/boilerplate.xml.preamble...
Copying gs://spls/gsp480/gke-network-policy-demo/test/boilerplate/boilerplate.xml.txt...
Copying gs://spls/gsp480/gke-network-policy-demo/test/boilerplate/boilerplate.yaml.txt...
Copying gs://spls/gsp480/gke-network-policy-demo/test/make.sh...                
Copying gs://spls/gsp480/gke-network-policy-demo/test/verify_boilerplate.py...  
Copying gs://spls/gsp480/gke-network-policy-demo/validate.sh...                 
- [82 files][491.6 KiB/491.6 KiB]    4.5 KiB/s                                  
==> NOTE: You are performing a sequence of gsutil operations that may
run significantly faster if you instead use gsutil -m cp ... Please
see the -m section under "gsutil help options" for further information
about when gsutil -m can be advantageous.


Operation completed over 82 objects/491.6 KiB.                                   
student_01_41d0c1af2373@cloudshell:~ (qwiklabs-gcp-01-37dcbcf4fa9d)$ cd gke-network-policy-demo
student_01_41d0c1af2373@cloudshell:~/gke-network-policy-demo (qwiklabs-gcp-01-37dcbcf4fa9d)$ chmod -R 755 *
student_01_41d0c1af2373@cloudshell:~/gke-network-policy-demo (qwiklabs-gcp-01-37dcbcf4fa9d)$ 
```

## Lab setup

First, set the Google Cloud region and zone.

This lab will use the following Google Cloud Service APIs, and have been enabled for you:

- `compute.googleapis.com`
- `container.googleapis.com`
- `cloudbuild.googleapis.com`

In addition, the Terraform configuration takes three parameters to  determine where the Kubernetes Engine cluster should be created:

- `project ID`
- `region`
- `zone`

For simplicity, these parameters are specified in a file named `terraform.tfvars`, in the `terraform` directory.

1. To ensure the appropriate APIs are enabled and to generate the `terraform/terraform.tfvars` file based on your gcloud defaults, run:

```sh
student_01_41d0c1af2373@cloudshell:~/gke-network-policy-demo (qwiklabs-gcp-01-37dcbcf4fa9d)$ cat terraform/terraform.tfvars 
project="qwiklabs-gcp-01-37dcbcf4fa9d"
region="us-central1"
zone="us-central1-c"
ssh_user_bastion="student_01_41d0c1af2373"
student_01_41d0c1af2373@cloudshell:~/gke-network-policy-demo (qwiklabs-gcp-01-37dcbcf4fa9d)$ 
```



```sh
student_01_41d0c1af2373@cloudshell:~/gke-network-policy-demo (qwiklabs-gcp-01-37dcbcf4fa9d)$ gcloud config set compute/region "us-central1"
WARNING: Property validation for compute/region was skipped.
Updated property [compute/region].
student_01_41d0c1af2373@cloudshell:~/gke-network-policy-demo (qwiklabs-gcp-01-37dcbcf4fa9d)$ gcloud config set compute/zone "us-central1-c"
WARNING: Property validation for compute/zone was skipped.
Updated property [compute/zone].
student_01_41d0c1af2373@cloudshell:~/gke-network-policy-demo (qwiklabs-gcp-01-37dcbcf4fa9d)$ make setup-project
# Enables the Google Cloud APIs needed
./enable-apis.sh

The following APIs will be enabled in your Google Cloud account:
- compute.googleapis.com
- container.googleapis.com
- cloudbuild.googleapis.com

Would you like to proceed? [y/n]: y
Enabling the Compute API, the Container API, the Cloud Build API
Operation "operations/acat.p2-540480615830-73c5a5ea-02a2-4c60-af86-53e796a80d21" finished successfully.
# Runs the generate-tfvars.sh
./generate-tfvars.sh
Your active configuration is: [cloudshell-19786]
Your active configuration is: [cloudshell-19786]
Your active configuration is: [cloudshell-19786]
student_01_41d0c1af2373@cloudshell:~/gke-network-policy-demo (qwiklabs-gcp-01-37dcbcf4fa9d)$ 
```

### Provisioning the Kubernetes Engine cluster

1. Next, apply the Terraform configuration within the project root:

```sh
student_01_41d0c1af2373@cloudshell:~/gke-network-policy-demo (qwiklabs-gcp-01-37dcbcf4fa9d)$ make tf-apply
# Downloads the terraform providers and applies the configuration
cd terraform && terraform init && terraform apply

Initializing the backend...

Initializing provider plugins...
- Finding latest version of hashicorp/google...
- Finding latest version of hashicorp/template...
- Installing hashicorp/template v2.2.0...
- Installed hashicorp/template v2.2.0 (signed by HashiCorp)
- Installing hashicorp/google v5.30.0...
- Installed hashicorp/google v5.30.0 (signed by HashiCorp)

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
data.template_file.startup_script: Reading...
data.template_file.startup_script: Read complete after 0s [id=59aeb82d40328ddaaa8a24ee8093dbac2660c1ba865a300fba6d222625b5474e]
data.google_container_engine_versions.gke_version: Reading...
data.google_container_engine_versions.gke_version: Read complete after 2s [id=2024-05-28 16:58:26.59238108 +0000 UTC]

Terraform used the selected providers to generate the following execution plan. Resource actions are indicated with the following symbols:
  + create

Terraform will perform the following actions:

  # google_compute_firewall.bastion-ssh will be created
  + resource "google_compute_firewall" "bastion-ssh" {
      + creation_timestamp = (known after apply)
      + destination_ranges = (known after apply)
      + direction          = "INGRESS"
      + enable_logging     = (known after apply)
      + id                 = (known after apply)
      + name               = "bastion-ssh"
      + network            = (known after apply)
      + priority           = 1000
      + project            = "qwiklabs-gcp-01-37dcbcf4fa9d"
      + self_link          = (known after apply)
      + source_ranges      = [
          + "0.0.0.0/0",
        ]
      + target_tags        = [
          + "bastion",
        ]

      + allow {
          + ports    = [
              + "22",
            ]
          + protocol = "tcp"
        }
    }

  # google_compute_instance.gke-bastion will be created
  + resource "google_compute_instance" "gke-bastion" {
      + allow_stopping_for_update = true
      + can_ip_forward            = false
      + cpu_platform              = (known after apply)
      + current_status            = (known after apply)
      + deletion_protection       = false
      + effective_labels          = (known after apply)
      + guest_accelerator         = (known after apply)
      + id                        = (known after apply)
      + instance_id               = (known after apply)
      + label_fingerprint         = (known after apply)
      + machine_type              = "g1-small"
      + metadata_fingerprint      = (known after apply)
      + metadata_startup_script   = <<-EOT
            sudo apt-get update -y
            sudo apt-get install -y kubectl
            echo "gcloud container clusters get-credentials gke-demo-cluster --zone us-central1-c --project qwiklabs-gcp-01-37dcbcf4fa9d" >> /etc/profile
        EOT
      + min_cpu_platform          = (known after apply)
      + name                      = "gke-demo-bastion"
      + project                   = "qwiklabs-gcp-01-37dcbcf4fa9d"
      + self_link                 = (known after apply)
      + tags                      = [
          + "bastion",
        ]
      + tags_fingerprint          = (known after apply)
      + terraform_labels          = (known after apply)
      + zone                      = "us-central1-c"

      + boot_disk {
          + auto_delete                = true
          + device_name                = (known after apply)
          + disk_encryption_key_sha256 = (known after apply)
          + kms_key_self_link          = (known after apply)
          + mode                       = "READ_WRITE"
          + source                     = (known after apply)

          + initialize_params {
              + image                  = "debian-cloud/debian-11"
              + labels                 = (known after apply)
              + provisioned_iops       = (known after apply)
              + provisioned_throughput = (known after apply)
              + size                   = (known after apply)
              + type                   = (known after apply)
            }
        }

      + network_interface {
          + internal_ipv6_prefix_length = (known after apply)
          + ipv6_access_type            = (known after apply)
          + ipv6_address                = (known after apply)
          + name                        = (known after apply)
          + network                     = (known after apply)
          + network_ip                  = (known after apply)
          + stack_type                  = (known after apply)
          + subnetwork                  = (known after apply)
          + subnetwork_project          = (known after apply)

          + access_config {
              + nat_ip       = (known after apply)
              + network_tier = (known after apply)
            }
        }

      + service_account {
          + email  = (known after apply)
          + scopes = [
              + "https://www.googleapis.com/auth/cloud-platform",
              + "https://www.googleapis.com/auth/compute.readonly",
              + "https://www.googleapis.com/auth/devstorage.read_only",
              + "https://www.googleapis.com/auth/userinfo.email",
            ]
        }
    }

  # google_compute_network.gke-network will be created
  + resource "google_compute_network" "gke-network" {
      + auto_create_subnetworks                   = false
      + delete_default_routes_on_create           = false
      + gateway_ipv4                              = (known after apply)
      + id                                        = (known after apply)
      + internal_ipv6_range                       = (known after apply)
      + mtu                                       = (known after apply)
      + name                                      = "kube-net"
      + network_firewall_policy_enforcement_order = "AFTER_CLASSIC_FIREWALL"
      + numeric_id                                = (known after apply)
      + project                                   = "qwiklabs-gcp-01-37dcbcf4fa9d"
      + routing_mode                              = (known after apply)
      + self_link                                 = (known after apply)
    }

  # google_compute_subnetwork.cluster-subnet will be created
  + resource "google_compute_subnetwork" "cluster-subnet" {
      + creation_timestamp         = (known after apply)
      + external_ipv6_prefix       = (known after apply)
      + fingerprint                = (known after apply)
      + gateway_address            = (known after apply)
      + id                         = (known after apply)
      + internal_ipv6_prefix       = (known after apply)
      + ip_cidr_range              = "10.0.96.0/22"
      + ipv6_cidr_range            = (known after apply)
      + name                       = "kube-net-subnet"
      + network                    = (known after apply)
      + private_ip_google_access   = (known after apply)
      + private_ipv6_google_access = (known after apply)
      + project                    = "qwiklabs-gcp-01-37dcbcf4fa9d"
      + purpose                    = (known after apply)
      + region                     = "us-central1"
      + secondary_ip_range         = [
          + {
              + ip_cidr_range = "10.0.92.0/22"
              + range_name    = "secondary-range"
            },
        ]
      + self_link                  = (known after apply)
      + stack_type                 = (known after apply)
    }

  # google_container_cluster.primary will be created
  + resource "google_container_cluster" "primary" {
      + cluster_ipv4_cidr                        = (known after apply)
      + datapath_provider                        = "ADVANCED_DATAPATH"
      + default_max_pods_per_node                = (known after apply)
      + deletion_protection                      = false
      + enable_cilium_clusterwide_network_policy = false
      + enable_intranode_visibility              = (known after apply)
      + enable_kubernetes_alpha                  = false
      + enable_l4_ilb_subsetting                 = false
      + enable_legacy_abac                       = false
      + enable_shielded_nodes                    = true
      + endpoint                                 = (known after apply)
      + id                                       = (known after apply)
      + initial_node_count                       = 3
      + label_fingerprint                        = (known after apply)
      + location                                 = "us-central1-c"
      + logging_service                          = (known after apply)
      + master_version                           = (known after apply)
      + min_master_version                       = "1.29.5-gke.1010000"
      + monitoring_service                       = (known after apply)
      + name                                     = "gke-demo-cluster"
      + network                                  = (known after apply)
      + networking_mode                          = (known after apply)
      + node_locations                           = (known after apply)
      + node_version                             = (known after apply)
      + operation                                = (known after apply)
      + private_ipv6_google_access               = (known after apply)
      + project                                  = "qwiklabs-gcp-01-37dcbcf4fa9d"
      + self_link                                = (known after apply)
      + services_ipv4_cidr                       = (known after apply)
      + subnetwork                               = (known after apply)
      + tpu_ipv4_cidr_block                      = (known after apply)

      + ip_allocation_policy {
          + cluster_ipv4_cidr_block       = (known after apply)
          + cluster_secondary_range_name  = "secondary-range"
          + services_ipv4_cidr_block      = (known after apply)
          + services_secondary_range_name = (known after apply)
          + stack_type                    = "IPV4"
        }

      + master_authorized_networks_config {
          + gcp_public_cidrs_access_enabled = (known after apply)

          + cidr_blocks {
              + cidr_block   = (known after apply)
              + display_name = "bastion"
            }
        }

      + node_config {
          + disk_size_gb      = (known after apply)
          + disk_type         = (known after apply)
          + effective_taints  = (known after apply)
          + guest_accelerator = (known after apply)
          + image_type        = "COS_CONTAINERD"
          + labels            = {
              + "status" = "poc"
            }
          + local_ssd_count   = (known after apply)
          + logging_variant   = (known after apply)
          + machine_type      = "n1-standard-1"
          + metadata          = (known after apply)
          + min_cpu_platform  = (known after apply)
          + oauth_scopes      = [
              + "https://www.googleapis.com/auth/compute",
              + "https://www.googleapis.com/auth/devstorage.read_only",
              + "https://www.googleapis.com/auth/logging.write",
              + "https://www.googleapis.com/auth/monitoring",
            ]
          + preemptible       = false
          + service_account   = (known after apply)
          + spot              = false
          + tags              = [
              + "poc",
            ]
        }

      + private_cluster_config {
          + enable_private_nodes   = true
          + master_ipv4_cidr_block = "10.0.90.0/28"
          + peering_name           = (known after apply)
          + private_endpoint       = (known after apply)
          + public_endpoint        = (known after apply)
        }
    }

Plan: 5 to add, 0 to change, 0 to destroy.

Do you want to perform these actions?
  Terraform will perform the actions described above.
  Only 'yes' will be accepted to approve.

  Enter a value: yes

google_compute_network.gke-network: Creating...
google_compute_network.gke-network: Still creating... [10s elapsed]
google_compute_network.gke-network: Still creating... [20s elapsed]
google_compute_network.gke-network: Creation complete after 22s [id=projects/qwiklabs-gcp-01-37dcbcf4fa9d/global/networks/kube-net]
google_compute_subnetwork.cluster-subnet: Creating...
google_compute_firewall.bastion-ssh: Creating...
google_compute_subnetwork.cluster-subnet: Still creating... [10s elapsed]
google_compute_firewall.bastion-ssh: Still creating... [10s elapsed]
google_compute_firewall.bastion-ssh: Creation complete after 12s [id=projects/qwiklabs-gcp-01-37dcbcf4fa9d/global/firewalls/bastion-ssh]
google_compute_subnetwork.cluster-subnet: Still creating... [20s elapsed]
google_compute_subnetwork.cluster-subnet: Still creating... [30s elapsed]
google_compute_subnetwork.cluster-subnet: Creation complete after 37s [id=projects/qwiklabs-gcp-01-37dcbcf4fa9d/regions/us-central1/subnetworks/kube-net-subnet]
google_compute_instance.gke-bastion: Creating...
google_compute_instance.gke-bastion: Still creating... [10s elapsed]
google_compute_instance.gke-bastion: Provisioning with 'local-exec'...
google_compute_instance.gke-bastion (local-exec): Executing: ["/bin/bash" "-c" "        READY=\"\"\n        for i in $(seq 1 18); do\n          if gcloud compute ssh student_01_41d0c1af2373@gke-demo-bastion --command uptime; then\n            READY=\"yes\"\n            break;\n          fi\n          echo \"Waiting for gke-demo-bastion to initialize...\"\n          sleep 10;\n        done\n\n        if [[ -z $READY ]]; then\n          echo \"gke-demo-bastion failed to start in time.\"\n          echo \"Please verify that the instance starts and then re-run `terraform apply`\"\n          exit 1\n        fi\n\n        gcloud compute  --project qwiklabs-gcp-01-37dcbcf4fa9d scp --zone us-central1-c --recurse ../manifests student_01_41d0c1af2373@gke-demo-bastion:\n"]
google_compute_instance.gke-bastion (local-exec): WARNING: The private SSH key file for gcloud does not exist.
google_compute_instance.gke-bastion (local-exec): WARNING: The public SSH key file for gcloud does not exist.
google_compute_instance.gke-bastion (local-exec): WARNING: You do not have an SSH key for gcloud.
google_compute_instance.gke-bastion (local-exec): WARNING: SSH keygen will be executed to generate a key.
google_compute_instance.gke-bastion (local-exec): This tool needs to create the directory [/home/student_01_41d0c1af2373/.ssh] before being able to generate SSH keys.

google_compute_instance.gke-bastion (local-exec): Do you want to continue (Y/n)?
google_compute_instance.gke-bastion: Still creating... [20s elapsed]
google_compute_instance.gke-bastion (local-exec): Generating public/private rsa key pair.
google_compute_instance.gke-bastion (local-exec): Your identification has been saved in /home/student_01_41d0c1af2373/.ssh/google_compute_engine
google_compute_instance.gke-bastion (local-exec): Your public key has been saved in /home/student_01_41d0c1af2373/.ssh/google_compute_engine.pub
google_compute_instance.gke-bastion (local-exec): The key fingerprint is:
google_compute_instance.gke-bastion (local-exec): SHA256:endxWKmnp2ePmPV8+zJPsZZMyOd0ivECOpMMPmfs9l0 student_01_41d0c1af2373@cs-295826878337-default
google_compute_instance.gke-bastion (local-exec): The key's randomart image is:
google_compute_instance.gke-bastion (local-exec): +---[RSA 3072]----+
google_compute_instance.gke-bastion (local-exec): |                 |
google_compute_instance.gke-bastion (local-exec): |               . |
google_compute_instance.gke-bastion (local-exec): |              o  |
google_compute_instance.gke-bastion (local-exec): |            .+.  |
google_compute_instance.gke-bastion (local-exec): |      . S . =oo=.|
google_compute_instance.gke-bastion (local-exec): |     . = o . O*.=|
google_compute_instance.gke-bastion (local-exec): |      + @ . = E*.|
google_compute_instance.gke-bastion (local-exec): |       *.+ o O*=.|
google_compute_instance.gke-bastion (local-exec): |       .... =o.*X|
google_compute_instance.gke-bastion (local-exec): +----[SHA256]-----+
google_compute_instance.gke-bastion (local-exec): Using OS Login user [student-01-41d0c1af2373] instead of requested user [student_01_41d0c1af2373]
google_compute_instance.gke-bastion (local-exec): Warning: Permanently added 'compute.5151878871691015066' (ED25519) to the list of known hosts.
google_compute_instance.gke-bastion (local-exec): student-01-41d0c1af2373@34.173.159.114: Permission denied (publickey).

google_compute_instance.gke-bastion (local-exec): Recommendation: To check for possible causes of SSH connectivity issues and get
google_compute_instance.gke-bastion (local-exec): recommendations, rerun the ssh command with the --troubleshoot option.

google_compute_instance.gke-bastion (local-exec): gcloud compute ssh gke-demo-bastion --project=qwiklabs-gcp-01-37dcbcf4fa9d --zone=us-central1-c --troubleshoot

google_compute_instance.gke-bastion (local-exec): Or, to investigate an IAP tunneling issue:

google_compute_instance.gke-bastion (local-exec): gcloud compute ssh gke-demo-bastion --project=qwiklabs-gcp-01-37dcbcf4fa9d --zone=us-central1-c --troubleshoot --tunnel-through-iap

google_compute_instance.gke-bastion (local-exec): ERROR: (gcloud.compute.ssh) [/usr/bin/ssh] exited with return code [255].
google_compute_instance.gke-bastion (local-exec): Waiting for gke-demo-bastion to initialize...
google_compute_instance.gke-bastion: Still creating... [30s elapsed]
google_compute_instance.gke-bastion (local-exec): Using OS Login user [student-01-41d0c1af2373] instead of requested user [student_01_41d0c1af2373]
google_compute_instance.gke-bastion: Still creating... [40s elapsed]
google_compute_instance.gke-bastion (local-exec):  17:00:13 up 0 min,  0 users,  load average: 0.98, 0.23, 0.08
google_compute_instance.gke-bastion (local-exec): Using OS Login user [student-01-41d0c1af2373] instead of requested user [student_01_41d0c1af2373]
google_compute_instance.gke-bastion: Still creating... [50s elapsed]
google_compute_instance.gke-bastion: Creation complete after 52s [id=projects/qwiklabs-gcp-01-37dcbcf4fa9d/zones/us-central1-c/instances/gke-demo-bastion]
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
google_container_cluster.primary: Creation complete after 6m17s [id=projects/qwiklabs-gcp-01-37dcbcf4fa9d/locations/us-central1-c/clusters/gke-demo-cluster]

Apply complete! Resources: 5 added, 0 changed, 0 destroyed.
student_01_41d0c1af2373@cloudshell:~/gke-network-policy-demo (qwiklabs-gcp-01-37dcbcf4fa9d)$ 
```

Existing versions of kubectl and custom Kubernetes clients contain  provider-specific code to manage authentication between the client and  Google Kubernetes Engine. Starting with v1.26, this code will no longer  be included as part of the OSS kubectl. GKE users will need to download  and use a separate authentication plugin to generate GKE-specific  tokens. This new binary, `gke-gcloud-auth-plugin`, uses the [Kubernetes Client-go Credential Plugin](https://kubernetes.io/docs/reference/access-authn-authz/authentication/#client-go-credential-plugins) mechanism to extend kubectl’s authentication to support GKE. For more information, you can check out the following [documentation](https://cloud.google.com/blog/products/containers-kubernetes/kubectl-auth-changes-in-gke).

To have kubectl use the new binary plugin for authentication instead  of using the default provider-specific code, use the following steps.



```sh
student_01_41d0c1af2373@cloudshell:~/gke-network-policy-demo (qwiklabs-gcp-01-37dcbcf4fa9d)$ gcloud compute ssh gke-demo-bastion
Linux gke-demo-bastion 5.10.0-29-cloud-amd64 #1 SMP Debian 5.10.216-1 (2024-05-03) x86_64

The programs included with the Debian GNU/Linux system are free software;
the exact distribution terms for each program are described in the
individual files in /usr/share/doc/*/copyright.

Debian GNU/Linux comes with ABSOLUTELY NO WARRANTY, to the extent
permitted by applicable law.
Fetching cluster endpoint and auth data.
CRITICAL: ACTION REQUIRED: gke-gcloud-auth-plugin, which is needed for continued use of kubectl, was not found or is not executable. Install gke-gcloud-auth-plugin for use with kubectl by following https://cloud.google.com/kubernetes-engine/docs/how-to/cluster-access-for-kubectl#install_plugin
kubeconfig entry generated for gke-demo-cluster.
student-01-41d0c1af2373@gke-demo-bastion:~$ 
```

```sh
student-01-41d0c1af2373@gke-demo-bastion:~$ sudo apt-get install google-cloud-sdk-gke-gcloud-auth-plugin
Reading package lists... Done
Building dependency tree... Done
Reading state information... Done
The following additional packages will be installed:
  google-cloud-cli-gke-gcloud-auth-plugin
The following NEW packages will be installed:
  google-cloud-cli-gke-gcloud-auth-plugin google-cloud-sdk-gke-gcloud-auth-plugin
0 upgraded, 2 newly installed, 0 to remove and 2 not upgraded.
Need to get 3227 kB of archives.
After this operation, 11.3 MB of additional disk space will be used.
Do you want to continue? [Y/n] y
Get:1 https://packages.cloud.google.com/apt cloud-sdk-bullseye/main amd64 google-cloud-cli-gke-gcloud-auth-plugin amd64 477.0.0-0 [3222 kB]
Get:2 https://packages.cloud.google.com/apt cloud-sdk-bullseye/main all google-cloud-sdk-gke-gcloud-auth-plugin all 467.0.0-0 [5018 B]
Fetched 3227 kB in 0s (14.2 MB/s)                            
Selecting previously unselected package google-cloud-cli-gke-gcloud-auth-plugin.
(Reading database ... 65176 files and directories currently installed.)
Preparing to unpack .../google-cloud-cli-gke-gcloud-auth-plugin_477.0.0-0_amd64.deb ...
Unpacking google-cloud-cli-gke-gcloud-auth-plugin (477.0.0-0) ...
Selecting previously unselected package google-cloud-sdk-gke-gcloud-auth-plugin.
Preparing to unpack .../google-cloud-sdk-gke-gcloud-auth-plugin_467.0.0-0_all.deb ...
Unpacking google-cloud-sdk-gke-gcloud-auth-plugin (467.0.0-0) ...
Setting up google-cloud-cli-gke-gcloud-auth-plugin (477.0.0-0) ...
Setting up google-cloud-sdk-gke-gcloud-auth-plugin (467.0.0-0) ...
student-01-41d0c1af2373@gke-demo-bastion:~$ 
```

```sh
student-01-41d0c1af2373@gke-demo-bastion:~$ echo "export USE_GKE_GCLOUD_AUTH_PLUGIN=True" >> ~/.bashrc
student-01-41d0c1af2373@gke-demo-bastion:~$ source ~/.bashrc
student-01-41d0c1af2373@gke-demo-bastion:~$ gcloud container clusters get-credentials gke-demo-cluster --zone us-central1-c
Fetching cluster endpoint and auth data.
kubeconfig entry generated for gke-demo-cluster.
student-01-41d0c1af2373@gke-demo-bastion:~$ 
```

The newly-created cluster will now be available for the standard `kubectl` commands on the bastion.

## Installing the hello server

The test application consists of one simple HTTP server, deployed as `hello-server`, and two clients, one of which will be labeled `app=hello` and the other `app=not-hello`.

All three services can be deployed by applying the hello-app manifests.

```sh
student-01-41d0c1af2373@gke-demo-bastion:~$ kubectl apply -f ./manifests/hello-app/
deployment.apps/hello-client-allowed created
deployment.apps/hello-client-blocked created
service/hello-server created
deployment.apps/hello-server created
student-01-41d0c1af2373@gke-demo-bastion:~$ 
```

```sh
student-01-41d0c1af2373@gke-demo-bastion:~$ kubectl get pods
NAME                                    READY   STATUS    RESTARTS   AGE
hello-client-allowed-6ddc786588-nlqmr   1/1     Running   0          19s
hello-client-blocked-67f55c4664-krjkq   1/1     Running   0          19s
hello-server-7f846f969d-9cwv2           1/1     Running   0          19s
student-01-41d0c1af2373@gke-demo-bastion:~$ 
```

## Confirming default access to the hello server

1. First, tail the "allowed" client:

```sh
student-01-41d0c1af2373@gke-demo-bastion:~$ kubectl logs --tail 10 -f $(kubectl get pods -oname -l app=hello)
Hostname: hello-server-7f846f969d-9cwv2
Hello, world!
Version: 1.0.0
Hostname: hello-server-7f846f969d-9cwv2
Hello, world!
Version: 1.0.0
Hostname: hello-server-7f846f969d-9cwv2
Hello, world!
Version: 1.0.0
Hostname: hello-server-7f846f969d-9cwv2
Hello, world!
Version: 1.0.0
Hostname: hello-server-7f846f969d-9cwv2
Hello, world!
Version: 1.0.0
Hostname: hello-server-7f846f969d-9cwv2
Hello, world!
Version: 1.0.0
Hostname: hello-server-7f846f969d-9cwv2
Hello, world!
Version: 1.0.0
Hostname: hello-server-7f846f969d-9cwv2
Hello, world!
Version: 1.0.0
Hostname: hello-server-7f846f969d-9cwv2
^C
student-01-41d0c1af2373@gke-demo-bastion:~$ 
```

Second, tail the logs of the "blocked" client:

```sh
student-01-41d0c1af2373@gke-demo-bastion:~$ kubectl logs --tail 10 -f $(kubectl get pods -oname -l app=not-hello)
Hostname: hello-server-7f846f969d-9cwv2
Hello, world!
Version: 1.0.0
Hostname: hello-server-7f846f969d-9cwv2
Hello, world!
Version: 1.0.0
Hostname: hello-server-7f846f969d-9cwv2
Hello, world!
Version: 1.0.0
Hostname: hello-server-7f846f969d-9cwv2
Hello, world!
Version: 1.0.0
Hostname: hello-server-7f846f969d-9cwv2
Hello, world!
Version: 1.0.0
Hostname: hello-server-7f846f969d-9cwv2
^C
student-01-41d0c1af2373@gke-demo-bastion:~$ 
```

You will notice that both pods are successfully able to connect to the `hello-server` service. This is because you have not yet defined a Network Policy to  restrict access. In each of these windows you should see successful  responses from the server.

## Restricting access with a Network Policy

Now you will block access to the `hello-server` pod from all pods that are not labeled with `app=hello`.

The policy definition you'll use is contained in `manifests/network-policy.yaml`

```yaml
student-01-41d0c1af2373@gke-demo-bastion:~$ cat ./manifests/network-policy.yaml
# Copyright 2018 Google LLC
#
# Licensed under the Apache License, Version 2.0 (the "License");
# you may not use this file except in compliance with the License.
# You may obtain a copy of the License at
#
#     https://www.apache.org/licenses/LICENSE-2.0
#
# Unless required by applicable law or agreed to in writing, software
# distributed under the License is distributed on an "AS IS" BASIS,
# WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
# See the License for the specific language governing permissions and
# limitations under the License.


# This network policy places restrictions on which pods can communicate to the
# hello-server service. Network policies can be thought of as in-cluster firewall
# rules, where the source and destination are specified as selectors. In this case
# we use labels as the selection criteria.
# See https://kubernetes.io/docs/concepts/services-networking/network-policies/
kind: NetworkPolicy
apiVersion: networking.k8s.io/v1
metadata:
  # Name the network policy
  name: hello-server-allow-from-hello-client
spec:

  # Define this as an ingress rule which allows us to restrict access to a set of pods.
  policyTypes:
  - Ingress

  # Defines the set of pods to which this policy applies
  # In this case, we apply the policy to pods labeled as app=hello-server
  podSelector:
    matchLabels:
      app: hello-server

  # Define the sources allowed by this policy
  # In this case, we allow ingress from all pods labeled as app=hello
  ingress:
  - from:
    - podSelector:
        matchLabels:
          app: hello
student-01-41d0c1af2373@gke-demo-bastion:~$ 
```

```sh
student-01-41d0c1af2373@gke-demo-bastion:~$ kubectl apply -f ./manifests/network-policy.yaml
networkpolicy.networking.k8s.io/hello-server-allow-from-hello-client created
student-01-41d0c1af2373@gke-demo-bastion:~$ 
```

Tail the logs of the "blocked" client again: You'll now see that the output looks like this in the window tailing the "blocked" client:

```sh
student-01-41d0c1af2373@gke-demo-bastion:~$ kubectl logs --tail 10 -f $(kubectl get pods -oname -l app=not-hello)
Hostname: hello-server-7f846f969d-9cwv2
wget: download timed out
wget: download timed out
wget: download timed out
wget: download timed out
wget: download timed out
wget: download timed out
wget: download timed out
wget: download timed out
wget: download timed out
wget: download timed out
^C
student-01-41d0c1af2373@gke-demo-bastion:~$ 
```

The network policy has now prevented communication to the `hello-server` from the unlabeled pod.

## Restricting namespaces with Network Policies

In the previous example, you defined a network policy that restricts  connections based on pod labels. It is often useful to instead label  entire namespaces, particularly when teams or applications are granted  their own namespaces.

You'll now modify the network policy to only allow traffic from a designated namespace, then you'll move the `hello-allowed` pod into that new namespace.

First, delete the existing network policy:

```sh
student-01-41d0c1af2373@gke-demo-bastion:~$ kubectl delete -f ./manifests/network-policy.yaml
networkpolicy.networking.k8s.io "hello-server-allow-from-hello-client" deleted
student-01-41d0c1af2373@gke-demo-bastion:~$ 
```

Create the namespaced version:

```yaml
student-01-41d0c1af2373@gke-demo-bastion:~$ cat ./manifests/network-policy-namespaced.yaml
# Copyright 2018 Google LLC
#
# Licensed under the Apache License, Version 2.0 (the "License");
# you may not use this file except in compliance with the License.
# You may obtain a copy of the License at
#
#     https://www.apache.org/licenses/LICENSE-2.0
#
# Unless required by applicable law or agreed to in writing, software
# distributed under the License is distributed on an "AS IS" BASIS,
# WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
# See the License for the specific language governing permissions and
# limitations under the License.

# Defines a new namespace used to demonstrate whitelisting an entire namespace
# See https://kubernetes.io/docs/concepts/overview/working-with-objects/namespaces/
kind: Namespace
apiVersion: v1
metadata:
  name: hello-apps
  labels:
    team: "hello"

---

# This network policy places restrictions on which pods can communicate to the
# hello-server service. Network policies can be thought of as in-cluster firewall
# rules, where the source and destination are specified as selectors. In this case
# we use labels as the selection criteria for namespaces.
# See https://kubernetes.io/docs/concepts/services-networking/network-policies/
kind: NetworkPolicy
apiVersion: networking.k8s.io/v1
metadata:
  # Name the network policy
  name: hello-server-allow-from-hello-client
spec:

  # Define this as an ingress rule which allows us to restrict access to a set of pods.
  policyTypes:
  - Ingress

  # Defines the set of pods to which this policy applies
  # In this case, we apply the policy to pods labeled as app=hello-server
  podSelector:
    matchLabels:
      app: hello-server

  # Define the sources allowed by this policy
  # In this case, we allow ingress from all pods the namespace labeled as team=hello
  # Note: as of Kubernetes 1.10. It is not possible to restrict connections
  # by both namespace and pod labels simultaneously. However support is
  # expected to be added in the future.
  ingress:
  - from:
    - namespaceSelector:
        matchLabels:
          team: hello
student-01-41d0c1af2373@gke-demo-bastion:~$ 
```

```sh
student-01-41d0c1af2373@gke-demo-bastion:~$ kubectl create -f ./manifests/network-policy-namespaced.yaml
namespace/hello-apps created
networkpolicy.networking.k8s.io/hello-server-allow-from-hello-client created
student-01-41d0c1af2373@gke-demo-bastion:~$ 
```

Now observe the logs of the `hello-allowed-client` pod in the default namespace:

```sh
student-01-41d0c1af2373@gke-demo-bastion:~$ kubectl logs --tail 10 -f $(kubectl get pods -oname -l app=hello)
Hostname: hello-server-7f846f969d-9cwv2
Hello, world!
Version: 1.0.0
Hostname: hello-server-7f846f969d-9cwv2
wget: download timed out
wget: download timed out
wget: download timed out
wget: download timed out
wget: download timed out
wget: download timed out
wget: download timed out
^C
student-01-41d0c1af2373@gke-demo-bastion:~$ 
```

You will notice it is no longer able to connect to the `hello-server`.

1. Press CTRL+C to exit.
2. Finally, deploy a second copy of the hello-clients app into the new namespace.

```yaml
student-01-41d0c1af2373@gke-demo-bastion:~$ cat ./manifests/hello-app/hello-client.yaml
# Copyright 2018 Google LLC
#
# Licensed under the Apache License, Version 2.0 (the "License");
# you may not use this file except in compliance with the License.
# You may obtain a copy of the License at
#
#     https://www.apache.org/licenses/LICENSE-2.0
#
# Unless required by applicable law or agreed to in writing, software
# distributed under the License is distributed on an "AS IS" BASIS,
# WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
# See the License for the specific language governing permissions and
# limitations under the License.

# The deployments below both run a simple loop that attempts to access the hello-server
# endpoint:
#
# hello-client-allowed:
#   Runs a single pod labeled with app=hello, which matches the ingress rule for
#   the hello-server service defined in the network policy and will therefore still be
#   able to access the service when the network policy is enabled.
#
# hello-client-blocked:
#   Runs a single pod lacking the app=hello label, which does not match the ingress rule
#   for the hello-server service defined in the network policy. Once the network policy
#   is enabled, the pod will lose access to the service.

# hello-client-allowed deployment
apiVersion: apps/v1
kind: Deployment
metadata:
  name: hello-client-allowed
spec:
  # Only run a single pod
  replicas: 1

  # Control any pod labeled with app=hello
  selector:
    matchLabels:
      app: hello

  # Define pod properties
  template:

    # Ensure created pods are labeled with app=hello to match the deployment selector
    metadata:
      labels:
        app: hello
    spec:
      # This pod does not require access to the Kubernetes API server, so we prevent
      # even the default token from being mounted
      automountServiceAccountToken: false

      # Pod-level security context to define the default UID and GIDs under which to
      # run all container processes. We use 9999 for all IDs since it is unprivileged
      # and known to be unallocated on the node instances.
      securityContext:
        runAsUser: 9999
        runAsGroup: 9999
        fsGroup: 9999

      # Define container properties
      containers:
      - image: alpine:3.7
        name: hello-client-allowed

        # A simple while loop that attempts to access the hello-server service every
        # two seconds.
        command: ["sh", "-c"]
        args: ["while true; do wget -qO- --timeout=2 http://hello-server.default.svc:8080; sleep 2; done"]

        # Container-level security settings
        # Note, containers are unprivileged by default
        securityContext:
          # Prevents the container from writing to its filesystem
          readOnlyRootFilesystem: true

---
# hello-client-blocked deployment
apiVersion: apps/v1
kind: Deployment
metadata:
  name: hello-client-blocked
spec:

  # Only run a single pod
  replicas: 1

  # Control any pod labeled with app=hello
  selector:
    matchLabels:
      app: not-hello

  # Define pod properties
  template:

    # Ensure created pods are labeled with app=hello to match the deployment selector
    metadata:
      labels:
        app: not-hello
    spec:
      # This pod does not require access to the Kubernetes API server, so we prevent
      # even the default token from being mounted
      automountServiceAccountToken: false

      # Pod-level security context to define the default UID and GIDs under which to
      # run all container processes. We use 9999 for all IDs since it is unprivileged
      # and known to be unallocated on the node instances.
      securityContext:
        runAsUser: 9999
        runAsGroup: 9999
        fsGroup: 9999

      # Define container properties
      containers:
      - image: alpine:3.7
        name: hello-client-blocked

        # A simple while loop that attempts to access the hello-server service every
        # two seconds.
        command: ["sh", "-c"]
        args: ["while true; do wget -qO- --timeout=2 http://hello-server.default.svc:8080; sleep 2; done"]

        # Container-level security settings
        # Note, containers are unprivileged by default
        securityContext:
          # Prevents the container from writing to its filesystem
          readOnlyRootFilesystem: true

student-01-41d0c1af2373@gke-demo-bastion:~$ 
```



```sh
student-01-41d0c1af2373@gke-demo-bastion:~$ kubectl -n hello-apps apply -f ./manifests/hello-app/hello-client.yaml
deployment.apps/hello-client-allowed created
deployment.apps/hello-client-blocked created
student-01-41d0c1af2373@gke-demo-bastion:~$ 
```

## Validation

Next, check the logs for the two new `hello-app` clients.

1. View the logs for the "hello"-labeled app in the app in the `hello-apps` namespace by running:

```sh
student-01-41d0c1af2373@gke-demo-bastion:~$ kubectl logs --tail 10 -f -n hello-apps $(kubectl get pods -oname -l app=hello -n hello-apps)
Hostname: hello-server-7f846f969d-9cwv2
Hello, world!
Version: 1.0.0
Hostname: hello-server-7f846f969d-9cwv2
Hello, world!
Version: 1.0.0
Hostname: hello-server-7f846f969d-9cwv2
Hello, world!
Version: 1.0.0
Hostname: hello-server-7f846f969d-9cwv2
Hello, world!
Version: 1.0.0
Hostname: hello-server-7f846f969d-9cwv2
^C
student-01-41d0c1af2373@gke-demo-bastion:~$ 
```

Both clients are able to connect successfully because *as of Kubernetes 1.10.x NetworkPolicies do not support restricting access to pods within a given namespace*. You can allowlist by pod label, namespace label, or allowlist the union (i.e. OR) of both. But you cannot yet allowlist the intersection (i.e.  AND) of pod labels and namespace labels.

```sh
student-01-41d0c1af2373@gke-demo-bastion:~$ kubectl get pods -n hello-apps
NAME                                    READY   STATUS    RESTARTS   AGE
hello-client-allowed-6ddc786588-7xhpk   1/1     Running   0          2m38s
hello-client-blocked-67f55c4664-96flx   1/1     Running   0          2m38s
student-01-41d0c1af2373@gke-demo-bastion:~$ kubectl get pods -n hello-apps --show-labels
NAME                                    READY   STATUS    RESTARTS   AGE     LABELS
hello-client-allowed-6ddc786588-7xhpk   1/1     Running   0          2m48s   app=hello,pod-template-hash=6ddc786588
hello-client-blocked-67f55c4664-96flx   1/1     Running   0          2m48s   app=not-hello,pod-template-hash=67f55c4664
student-01-41d0c1af2373@gke-demo-bastion:~$ 
```

## Teardown

```sh
student_01_41d0c1af2373@cloudshell:~/gke-network-policy-demo (qwiklabs-gcp-01-37dcbcf4fa9d)$ make teardown
/home/student_01_41d0c1af2373/gke-network-policy-demo/teardown.sh
Your active configuration is: [cloudshell-19786]
Your active configuration is: [cloudshell-19786]
Your active configuration is: [cloudshell-19786]
Using OS Login user [student-01-41d0c1af2373] instead of requested user [student_01_41d0c1af2373]
deployment.apps "hello-client-allowed" deleted
deployment.apps "hello-client-blocked" deleted
service "hello-server" deleted
deployment.apps "hello-server" deleted
Updated [https://www.googleapis.com/compute/v1/projects/qwiklabs-gcp-01-37dcbcf4fa9d/zones/us-central1-c/instances/gke-demo-bastion].
data.template_file.startup_script: Reading...
data.template_file.startup_script: Read complete after 0s [id=59aeb82d40328ddaaa8a24ee8093dbac2660c1ba865a300fba6d222625b5474e]
data.google_container_engine_versions.gke_version: Reading...
google_compute_network.gke-network: Refreshing state... [id=projects/qwiklabs-gcp-01-37dcbcf4fa9d/global/networks/kube-net]
google_compute_firewall.bastion-ssh: Refreshing state... [id=projects/qwiklabs-gcp-01-37dcbcf4fa9d/global/firewalls/bastion-ssh]
google_compute_subnetwork.cluster-subnet: Refreshing state... [id=projects/qwiklabs-gcp-01-37dcbcf4fa9d/regions/us-central1/subnetworks/kube-net-subnet]
data.google_container_engine_versions.gke_version: Read complete after 1s [id=2024-05-28 17:21:28.075800846 +0000 UTC]
google_compute_instance.gke-bastion: Refreshing state... [id=projects/qwiklabs-gcp-01-37dcbcf4fa9d/zones/us-central1-c/instances/gke-demo-bastion]
google_container_cluster.primary: Refreshing state... [id=projects/qwiklabs-gcp-01-37dcbcf4fa9d/locations/us-central1-c/clusters/gke-demo-cluster]

Terraform used the selected providers to generate the following execution plan. Resource actions are indicated with the following symbols:
  - destroy

Terraform will perform the following actions:

  # google_compute_firewall.bastion-ssh will be destroyed
  - resource "google_compute_firewall" "bastion-ssh" {
      - creation_timestamp      = "2024-05-28T09:58:54.248-07:00" -> null
      - destination_ranges      = [] -> null
      - direction               = "INGRESS" -> null
      - disabled                = false -> null
      - id                      = "projects/qwiklabs-gcp-01-37dcbcf4fa9d/global/firewalls/bastion-ssh" -> null
      - name                    = "bastion-ssh" -> null
      - network                 = "https://www.googleapis.com/compute/v1/projects/qwiklabs-gcp-01-37dcbcf4fa9d/global/networks/kube-net" -> null
      - priority                = 1000 -> null
      - project                 = "qwiklabs-gcp-01-37dcbcf4fa9d" -> null
      - self_link               = "https://www.googleapis.com/compute/v1/projects/qwiklabs-gcp-01-37dcbcf4fa9d/global/firewalls/bastion-ssh" -> null
      - source_ranges           = [
          - "0.0.0.0/0",
        ] -> null
      - source_service_accounts = [] -> null
      - source_tags             = [] -> null
      - target_service_accounts = [] -> null
      - target_tags             = [
          - "bastion",
        ] -> null

      - allow {
          - ports    = [
              - "22",
            ] -> null
          - protocol = "tcp" -> null
        }
    }

  # google_compute_instance.gke-bastion will be destroyed
  - resource "google_compute_instance" "gke-bastion" {
      - allow_stopping_for_update = true -> null
      - can_ip_forward            = false -> null
      - cpu_platform              = "Intel Haswell" -> null
      - current_status            = "RUNNING" -> null
      - deletion_protection       = false -> null
      - effective_labels          = {} -> null
      - enable_display            = false -> null
      - guest_accelerator         = [] -> null
      - id                        = "projects/qwiklabs-gcp-01-37dcbcf4fa9d/zones/us-central1-c/instances/gke-demo-bastion" -> null
      - instance_id               = "5151878871691015066" -> null
      - label_fingerprint         = "42WmSpB8rSM=" -> null
      - labels                    = {} -> null
      - machine_type              = "g1-small" -> null
      - metadata                  = {} -> null
      - metadata_fingerprint      = "ASFUGMftdHk=" -> null
      - name                      = "gke-demo-bastion" -> null
      - project                   = "qwiklabs-gcp-01-37dcbcf4fa9d" -> null
      - resource_policies         = [] -> null
      - self_link                 = "https://www.googleapis.com/compute/v1/projects/qwiklabs-gcp-01-37dcbcf4fa9d/zones/us-central1-c/instances/gke-demo-bastion" -> null
      - tags                      = [
          - "bastion",
        ] -> null
      - tags_fingerprint          = "NfTTNVh6sLU=" -> null
      - terraform_labels          = {} -> null
      - zone                      = "us-central1-c" -> null

      - boot_disk {
          - auto_delete = true -> null
          - device_name = "persistent-disk-0" -> null
          - mode        = "READ_WRITE" -> null
          - source      = "https://www.googleapis.com/compute/v1/projects/qwiklabs-gcp-01-37dcbcf4fa9d/zones/us-central1-c/disks/gke-demo-bastion" -> null

          - initialize_params {
              - enable_confidential_compute = false -> null
              - image                       = "https://www.googleapis.com/compute/v1/projects/debian-cloud/global/images/debian-11-bullseye-v20240515" -> null
              - labels                      = {} -> null
              - provisioned_iops            = 0 -> null
              - provisioned_throughput      = 0 -> null
              - resource_manager_tags       = {} -> null
              - size                        = 10 -> null
              - type                        = "pd-standard" -> null
            }
        }

      - network_interface {
          - internal_ipv6_prefix_length = 0 -> null
          - name                        = "nic0" -> null
          - network                     = "https://www.googleapis.com/compute/v1/projects/qwiklabs-gcp-01-37dcbcf4fa9d/global/networks/kube-net" -> null
          - network_ip                  = "10.0.96.2" -> null
          - queue_count                 = 0 -> null
          - stack_type                  = "IPV4_ONLY" -> null
          - subnetwork                  = "https://www.googleapis.com/compute/v1/projects/qwiklabs-gcp-01-37dcbcf4fa9d/regions/us-central1/subnetworks/kube-net-subnet" -> null
          - subnetwork_project          = "qwiklabs-gcp-01-37dcbcf4fa9d" -> null

          - access_config {
              - nat_ip       = "34.173.159.114" -> null
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

      - service_account {
          - email  = "540480615830-compute@developer.gserviceaccount.com" -> null
          - scopes = [
              - "https://www.googleapis.com/auth/cloud-platform",
              - "https://www.googleapis.com/auth/compute.readonly",
              - "https://www.googleapis.com/auth/devstorage.read_only",
              - "https://www.googleapis.com/auth/userinfo.email",
            ] -> null
        }

      - shielded_instance_config {
          - enable_integrity_monitoring = true -> null
          - enable_secure_boot          = false -> null
          - enable_vtpm                 = true -> null
        }
    }

  # google_compute_network.gke-network will be destroyed
  - resource "google_compute_network" "gke-network" {
      - auto_create_subnetworks                   = false -> null
      - delete_default_routes_on_create           = false -> null
      - enable_ula_internal_ipv6                  = false -> null
      - id                                        = "projects/qwiklabs-gcp-01-37dcbcf4fa9d/global/networks/kube-net" -> null
      - mtu                                       = 0 -> null
      - name                                      = "kube-net" -> null
      - network_firewall_policy_enforcement_order = "AFTER_CLASSIC_FIREWALL" -> null
      - numeric_id                                = "4970880471572132824" -> null
      - project                                   = "qwiklabs-gcp-01-37dcbcf4fa9d" -> null
      - routing_mode                              = "REGIONAL" -> null
      - self_link                                 = "https://www.googleapis.com/compute/v1/projects/qwiklabs-gcp-01-37dcbcf4fa9d/global/networks/kube-net" -> null
    }

  # google_compute_subnetwork.cluster-subnet will be destroyed
  - resource "google_compute_subnetwork" "cluster-subnet" {
      - creation_timestamp         = "2024-05-28T09:58:55.057-07:00" -> null
      - gateway_address            = "10.0.96.1" -> null
      - id                         = "projects/qwiklabs-gcp-01-37dcbcf4fa9d/regions/us-central1/subnetworks/kube-net-subnet" -> null
      - ip_cidr_range              = "10.0.96.0/22" -> null
      - name                       = "kube-net-subnet" -> null
      - network                    = "https://www.googleapis.com/compute/v1/projects/qwiklabs-gcp-01-37dcbcf4fa9d/global/networks/kube-net" -> null
      - private_ip_google_access   = true -> null
      - private_ipv6_google_access = "DISABLE_GOOGLE_ACCESS" -> null
      - project                    = "qwiklabs-gcp-01-37dcbcf4fa9d" -> null
      - purpose                    = "PRIVATE" -> null
      - region                     = "us-central1" -> null
      - secondary_ip_range         = [
          - {
              - ip_cidr_range = "10.0.92.0/22"
              - range_name    = "secondary-range"
            },
        ] -> null
      - self_link                  = "https://www.googleapis.com/compute/v1/projects/qwiklabs-gcp-01-37dcbcf4fa9d/regions/us-central1/subnetworks/kube-net-subnet" -> null
      - stack_type                 = "IPV4_ONLY" -> null
    }

  # google_container_cluster.primary will be destroyed
  - resource "google_container_cluster" "primary" {
      - cluster_ipv4_cidr                        = "10.0.92.0/22" -> null
      - datapath_provider                        = "ADVANCED_DATAPATH" -> null
      - default_max_pods_per_node                = 110 -> null
      - deletion_protection                      = false -> null
      - enable_autopilot                         = false -> null
      - enable_cilium_clusterwide_network_policy = false -> null
      - enable_intranode_visibility              = false -> null
      - enable_kubernetes_alpha                  = false -> null
      - enable_l4_ilb_subsetting                 = false -> null
      - enable_legacy_abac                       = false -> null
      - enable_shielded_nodes                    = true -> null
      - enable_tpu                               = false -> null
      - endpoint                                 = "35.223.134.53" -> null
      - id                                       = "projects/qwiklabs-gcp-01-37dcbcf4fa9d/locations/us-central1-c/clusters/gke-demo-cluster" -> null
      - initial_node_count                       = 3 -> null
      - label_fingerprint                        = "a9dc16a7" -> null
      - location                                 = "us-central1-c" -> null
      - logging_service                          = "logging.googleapis.com/kubernetes" -> null
      - master_version                           = "1.29.5-gke.1010000" -> null
      - min_master_version                       = "1.29.5-gke.1010000" -> null
      - monitoring_service                       = "monitoring.googleapis.com/kubernetes" -> null
      - name                                     = "gke-demo-cluster" -> null
      - network                                  = "projects/qwiklabs-gcp-01-37dcbcf4fa9d/global/networks/kube-net" -> null
      - networking_mode                          = "VPC_NATIVE" -> null
      - node_locations                           = [] -> null
      - node_version                             = "1.29.5-gke.1010000" -> null
      - project                                  = "qwiklabs-gcp-01-37dcbcf4fa9d" -> null
      - resource_labels                          = {} -> null
      - self_link                                = "https://container.googleapis.com/v1/projects/qwiklabs-gcp-01-37dcbcf4fa9d/zones/us-central1-c/clusters/gke-demo-cluster" -> null
      - services_ipv4_cidr                       = "34.118.224.0/20" -> null
      - subnetwork                               = "projects/qwiklabs-gcp-01-37dcbcf4fa9d/regions/us-central1/subnetworks/kube-net-subnet" -> null

      - addons_config {
          - gce_persistent_disk_csi_driver_config {
              - enabled = true -> null
            }
          - network_policy_config {
              - disabled = true -> null
            }
        }

      - binary_authorization {
          - enabled = false -> null
        }

      - cluster_autoscaling {
          - autoscaling_profile = "BALANCED" -> null
          - enabled             = false -> null
        }

      - database_encryption {
          - state = "DECRYPTED" -> null
        }

      - default_snat_status {
          - disabled = false -> null
        }

      - ip_allocation_policy {
          - cluster_ipv4_cidr_block      = "10.0.92.0/22" -> null
          - cluster_secondary_range_name = "secondary-range" -> null
          - services_ipv4_cidr_block     = "34.118.224.0/20" -> null
          - stack_type                   = "IPV4" -> null

          - pod_cidr_overprovision_config {
              - disabled = false -> null
            }
        }

      - logging_config {
          - enable_components = [
              - "SYSTEM_COMPONENTS",
              - "WORKLOADS",
            ] -> null
        }

      - master_auth {
          - cluster_ca_certificate = "LS0tLS1CRUdJTiBDRVJUSUZJQ0FURS0tLS0tCk1JSUVMVENDQXBXZ0F3SUJBZ0lSQU95R3hkWXBycytsT0t5d09ER2ZsZXd3RFFZSktvWklodmNOQVFFTEJRQXcKTHpFdE1Dc0dBMVVFQXhNa09EbGlaak5oWldNdE1EVXdZaTAwWkdJd0xXRTVOekF0T0dOallUSTRNVFZoWW1GbApNQ0FYRFRJME1EVXlPREUyTURBeU5Wb1lEekl3TlRRd05USXhNVGN3TURJMVdqQXZNUzB3S3dZRFZRUURFeVE0Ck9XSm1NMkZsWXkwd05UQmlMVFJrWWpBdFlUazNNQzA0WTJOaE1qZ3hOV0ZpWVdVd2dnR2lNQTBHQ1NxR1NJYjMKRFFFQkFRVUFBNElCandBd2dnR0tBb0lCZ1FEVFhBL1l6cWNvUGZZZnQ4bDhFaGtpZTNya3ExeWZmVGhVRnZmSQpuamYwd21TVUZEVnI4dVFKY1NYNlJoWTlkT2FIazVoRy9PeEZXMUw4RGVYMlBLQmFWWnBEdFg5RVFIWXF0M1BqClNFMG9oSHdKcDlya1hOSDFyQWFPVi93eVZkd01pRlAzNm55UDBKMDZqZ1JHUG1JYjBkYVBlMUNGR0tWZElSYmQKMUdyTFpKODdzcndUcXJWUEYvV0JzRHhpVTVMdExiRGc3VEllV2VhYWttSnlpK2YvLzVIaTlOc2YvUDRjQlRlYQpNN3J1TDRiamlKdVpOOFkyYURmdHoyZjIycC83MktSMndseE9Fdlo3V2tYamU4cjJNbFVKaEZzVWU5YU5vdCs0CmxDNGxsVGw1TDJyQ2pTRDRTRnpkOGFDM1U2cFZJbVBmMFpxZ0QyS05tV3Z2UktBdFljdlBXTVFtZVhEZjM1NUUKR05veENIelVROHJ1V3VrWlhTWDJGcjJBTGI3MTc5aUxnM21xYWlxWjY0MXh3RlJsT3hBY05Pc0tsTUpLbGRTWQpCMExEUUdXSW1CL1RJOEp5Kys0cVMxcmNuVnhzMG92WXFsNC8zVDVpOWthblNXeThQQ3ZQTmdwU1VBRk9ZVUpWCjY4WUxwaXp5OTE5eFU4NmpzbTkyNU1yQXhROENBd0VBQWFOQ01FQXdEZ1lEVlIwUEFRSC9CQVFEQWdJRU1BOEcKQTFVZEV3RUIvd1FGTUFNQkFmOHdIUVlEVlIwT0JCWUVGSzhYZzJOaE9Fc1hCY21VbWRKZ3B2QzQrUW5jTUEwRwpDU3FHU0liM0RRRUJDd1VBQTRJQmdRQzZUUXhxaXB6aStxZFhKQTUzVmdVTWZFUmRYZXVUWnBqYUhBTWQ5Uk9pCm8ydlZnUnpGODd2MHhxUFI2NlVjYVJPSTRSajVyY1UrVEszTnovSzZNVXJsSWdoWmlkZ0pYSE43WkVIMHVZeGoKMmltZFNVZFZNcFg2YUt1Mk05TVAwUzg0dTVsdmRKOFd0WXpFQ2hTUHRKRkduT3g5YUI2WXNSbHFXYk96ZHR6RgpCVmx5RlcxYTZCbWtBN1FFM2N4V1N5bExXcGJsVjg3RDg3RHFBTm5ONjBpSlhHanBSdHJBREdzV1VMVTV1eE1rCjR3L1R4SDZrdnhlZVFkYXJXcnpnVnNMOXlHRVNXVWlaT2p5cXU5QVF5NGU5MTBYOEMrenJWeEJzY3Vic1Z4VWkKOXRVQTFRcUFGTzFhQ2E1MEZRRDVlSEw5a1hvakloTktPNjlWZWpYMjBsSjFVVStHUG5wQm5RUStwbDVQK2tvdwpuNE1TQ2pYQU5nM0YvTjU3RjRsMUFjb0E0NWd0a1V2K1F5N3dWaFVLMDdSUEUxNEV4VGphSzRQdGFMb1VjZm5NClFSdGpTR2xWQ244Z2kvcFpVNWhnY1hseUNUa0Z6c1NnVEJkcTZzdnc4dHc4bGs1eTVrbTQvSkNLbVVrUVFvMjEKUkxVY1JNbGNINkhCTFFxemUvQVJPMXc9Ci0tLS0tRU5EIENFUlRJRklDQVRFLS0tLS0K" -> null

          - client_certificate_config {
              - issue_client_certificate = false -> null
            }
        }

      - master_authorized_networks_config {
          - gcp_public_cidrs_access_enabled = false -> null

          - cidr_blocks {
              - cidr_block   = "34.173.159.114/32" -> null
              - display_name = "bastion" -> null
            }
        }

      - monitoring_config {
          - enable_components = [
              - "SYSTEM_COMPONENTS",
            ] -> null

          - advanced_datapath_observability_config {
              - enable_metrics = false -> null
              - enable_relay   = false -> null
              - relay_mode     = "DISABLED" -> null
            }

          - managed_prometheus {
              - enabled = true -> null
            }
        }

      - network_policy {
          - enabled  = false -> null
          - provider = "PROVIDER_UNSPECIFIED" -> null
        }

      - node_config {
          - disk_size_gb                = 100 -> null
          - disk_type                   = "pd-balanced" -> null
          - effective_taints            = [] -> null
          - enable_confidential_storage = false -> null
          - guest_accelerator           = [] -> null
          - image_type                  = "COS_CONTAINERD" -> null
          - labels                      = {
              - "status" = "poc"
            } -> null
          - local_ssd_count             = 0 -> null
          - logging_variant             = "DEFAULT" -> null
          - machine_type                = "n1-standard-1" -> null
          - metadata                    = {
              - "disable-legacy-endpoints" = "true"
            } -> null
          - oauth_scopes                = [
              - "https://www.googleapis.com/auth/compute",
              - "https://www.googleapis.com/auth/devstorage.read_only",
              - "https://www.googleapis.com/auth/logging.write",
              - "https://www.googleapis.com/auth/monitoring",
            ] -> null
          - preemptible                 = false -> null
          - resource_labels             = {} -> null
          - resource_manager_tags       = {} -> null
          - service_account             = "default" -> null
          - spot                        = false -> null
          - tags                        = [
              - "poc",
            ] -> null

          - shielded_instance_config {
              - enable_integrity_monitoring = true -> null
              - enable_secure_boot          = false -> null
            }
        }

      - node_pool {
          - initial_node_count          = 3 -> null
          - instance_group_urls         = [
              - "https://www.googleapis.com/compute/v1/projects/qwiklabs-gcp-01-37dcbcf4fa9d/zones/us-central1-c/instanceGroupManagers/gke-gke-demo-cluster-default-pool-5687b519-grp",
            ] -> null
          - managed_instance_group_urls = [
              - "https://www.googleapis.com/compute/v1/projects/qwiklabs-gcp-01-37dcbcf4fa9d/zones/us-central1-c/instanceGroups/gke-gke-demo-cluster-default-pool-5687b519-grp",
            ] -> null
          - max_pods_per_node           = 110 -> null
          - name                        = "default-pool" -> null
          - node_count                  = 3 -> null
          - node_locations              = [
              - "us-central1-c",
            ] -> null
          - version                     = "1.29.5-gke.1010000" -> null

          - management {
              - auto_repair  = true -> null
              - auto_upgrade = true -> null
            }

          - network_config {
              - create_pod_range     = false -> null
              - enable_private_nodes = true -> null
              - pod_ipv4_cidr_block  = "10.0.92.0/22" -> null
              - pod_range            = "secondary-range" -> null
            }

          - node_config {
              - disk_size_gb                = 100 -> null
              - disk_type                   = "pd-balanced" -> null
              - effective_taints            = [] -> null
              - enable_confidential_storage = false -> null
              - guest_accelerator           = [] -> null
              - image_type                  = "COS_CONTAINERD" -> null
              - labels                      = {
                  - "status" = "poc"
                } -> null
              - local_ssd_count             = 0 -> null
              - logging_variant             = "DEFAULT" -> null
              - machine_type                = "n1-standard-1" -> null
              - metadata                    = {
                  - "disable-legacy-endpoints" = "true"
                } -> null
              - oauth_scopes                = [
                  - "https://www.googleapis.com/auth/compute",
                  - "https://www.googleapis.com/auth/devstorage.read_only",
                  - "https://www.googleapis.com/auth/logging.write",
                  - "https://www.googleapis.com/auth/monitoring",
                ] -> null
              - preemptible                 = false -> null
              - resource_labels             = {} -> null
              - resource_manager_tags       = {} -> null
              - service_account             = "default" -> null
              - spot                        = false -> null
              - tags                        = [
                  - "poc",
                ] -> null

              - shielded_instance_config {
                  - enable_integrity_monitoring = true -> null
                  - enable_secure_boot          = false -> null
                }
            }

          - upgrade_settings {
              - max_surge       = 1 -> null
              - max_unavailable = 0 -> null
              - strategy        = "SURGE" -> null
            }
        }

      - node_pool_defaults {
          - node_config_defaults {
              - logging_variant = "DEFAULT" -> null
            }
        }

      - notification_config {
          - pubsub {
              - enabled = false -> null
            }
        }

      - private_cluster_config {
          - enable_private_endpoint = false -> null
          - enable_private_nodes    = true -> null
          - master_ipv4_cidr_block  = "10.0.90.0/28" -> null
          - private_endpoint        = "10.0.90.2" -> null
          - public_endpoint         = "35.223.134.53" -> null

          - master_global_access_config {
              - enabled = false -> null
            }
        }

      - release_channel {
          - channel = "REGULAR" -> null
        }

      - security_posture_config {
          - mode               = "BASIC" -> null
          - vulnerability_mode = "VULNERABILITY_MODE_UNSPECIFIED" -> null
        }

      - service_external_ips_config {
          - enabled = false -> null
        }
    }

Plan: 0 to add, 0 to change, 5 to destroy.
google_compute_firewall.bastion-ssh: Destroying... [id=projects/qwiklabs-gcp-01-37dcbcf4fa9d/global/firewalls/bastion-ssh]
google_container_cluster.primary: Destroying... [id=projects/qwiklabs-gcp-01-37dcbcf4fa9d/locations/us-central1-c/clusters/gke-demo-cluster]
google_compute_firewall.bastion-ssh: Still destroying... [id=projects/qwiklabs-gcp-01-37dcbcf4fa9d/global/firewalls/bastion-ssh, 10s elapsed]
google_container_cluster.primary: Still destroying... [id=projects/qwiklabs-gcp-01-37dcbcf4fa9d/l...s-central1-c/clusters/gke-demo-cluster, 10s elapsed]
google_compute_firewall.bastion-ssh: Destruction complete after 12s
google_container_cluster.primary: Still destroying... [id=projects/qwiklabs-gcp-01-37dcbcf4fa9d/l...s-central1-c/clusters/gke-demo-cluster, 20s elapsed]
google_container_cluster.primary: Still destroying... [id=projects/qwiklabs-gcp-01-37dcbcf4fa9d/l...s-central1-c/clusters/gke-demo-cluster, 30s elapsed]
google_container_cluster.primary: Still destroying... [id=projects/qwiklabs-gcp-01-37dcbcf4fa9d/l...s-central1-c/clusters/gke-demo-cluster, 40s elapsed]
google_container_cluster.primary: Still destroying... [id=projects/qwiklabs-gcp-01-37dcbcf4fa9d/l...s-central1-c/clusters/gke-demo-cluster, 50s elapsed]
google_container_cluster.primary: Still destroying... [id=projects/qwiklabs-gcp-01-37dcbcf4fa9d/l...s-central1-c/clusters/gke-demo-cluster, 1m0s elapsed]
google_container_cluster.primary: Still destroying... [id=projects/qwiklabs-gcp-01-37dcbcf4fa9d/l...s-central1-c/clusters/gke-demo-cluster, 1m10s elapsed]
google_container_cluster.primary: Still destroying... [id=projects/qwiklabs-gcp-01-37dcbcf4fa9d/l...s-central1-c/clusters/gke-demo-cluster, 1m20s elapsed]
google_container_cluster.primary: Still destroying... [id=projects/qwiklabs-gcp-01-37dcbcf4fa9d/l...s-central1-c/clusters/gke-demo-cluster, 1m30s elapsed]
google_container_cluster.primary: Still destroying... [id=projects/qwiklabs-gcp-01-37dcbcf4fa9d/l...s-central1-c/clusters/gke-demo-cluster, 1m40s elapsed]
google_container_cluster.primary: Still destroying... [id=projects/qwiklabs-gcp-01-37dcbcf4fa9d/l...s-central1-c/clusters/gke-demo-cluster, 1m50s elapsed]
google_container_cluster.primary: Still destroying... [id=projects/qwiklabs-gcp-01-37dcbcf4fa9d/l...s-central1-c/clusters/gke-demo-cluster, 2m0s elapsed]
google_container_cluster.primary: Still destroying... [id=projects/qwiklabs-gcp-01-37dcbcf4fa9d/l...s-central1-c/clusters/gke-demo-cluster, 2m10s elapsed]
google_container_cluster.primary: Still destroying... [id=projects/qwiklabs-gcp-01-37dcbcf4fa9d/l...s-central1-c/clusters/gke-demo-cluster, 2m20s elapsed]
google_container_cluster.primary: Still destroying... [id=projects/qwiklabs-gcp-01-37dcbcf4fa9d/l...s-central1-c/clusters/gke-demo-cluster, 2m30s elapsed]
google_container_cluster.primary: Still destroying... [id=projects/qwiklabs-gcp-01-37dcbcf4fa9d/l...s-central1-c/clusters/gke-demo-cluster, 2m40s elapsed]
google_container_cluster.primary: Still destroying... [id=projects/qwiklabs-gcp-01-37dcbcf4fa9d/l...s-central1-c/clusters/gke-demo-cluster, 2m50s elapsed]
google_container_cluster.primary: Still destroying... [id=projects/qwiklabs-gcp-01-37dcbcf4fa9d/l...s-central1-c/clusters/gke-demo-cluster, 3m0s elapsed]
google_container_cluster.primary: Still destroying... [id=projects/qwiklabs-gcp-01-37dcbcf4fa9d/l...s-central1-c/clusters/gke-demo-cluster, 3m10s elapsed]
google_container_cluster.primary: Still destroying... [id=projects/qwiklabs-gcp-01-37dcbcf4fa9d/l...s-central1-c/clusters/gke-demo-cluster, 3m20s elapsed]
google_container_cluster.primary: Still destroying... [id=projects/qwiklabs-gcp-01-37dcbcf4fa9d/l...s-central1-c/clusters/gke-demo-cluster, 3m30s elapsed]
google_container_cluster.primary: Destruction complete after 3m40s
google_compute_instance.gke-bastion: Destroying... [id=projects/qwiklabs-gcp-01-37dcbcf4fa9d/zones/us-central1-c/instances/gke-demo-bastion]
google_compute_instance.gke-bastion: Still destroying... [id=projects/qwiklabs-gcp-01-37dcbcf4fa9d/z...-central1-c/instances/gke-demo-bastion, 10s elapsed]
google_compute_instance.gke-bastion: Still destroying... [id=projects/qwiklabs-gcp-01-37dcbcf4fa9d/z...-central1-c/instances/gke-demo-bastion, 20s elapsed]
google_compute_instance.gke-bastion: Still destroying... [id=projects/qwiklabs-gcp-01-37dcbcf4fa9d/z...-central1-c/instances/gke-demo-bastion, 30s elapsed]
google_compute_instance.gke-bastion: Still destroying... [id=projects/qwiklabs-gcp-01-37dcbcf4fa9d/z...-central1-c/instances/gke-demo-bastion, 40s elapsed]
google_compute_instance.gke-bastion: Destruction complete after 48s
google_compute_subnetwork.cluster-subnet: Destroying... [id=projects/qwiklabs-gcp-01-37dcbcf4fa9d/regions/us-central1/subnetworks/kube-net-subnet]
google_compute_subnetwork.cluster-subnet: Still destroying... [id=projects/qwiklabs-gcp-01-37dcbcf4fa9d/r...s-central1/subnetworks/kube-net-subnet, 10s elapsed]
google_compute_subnetwork.cluster-subnet: Still destroying... [id=projects/qwiklabs-gcp-01-37dcbcf4fa9d/r...s-central1/subnetworks/kube-net-subnet, 20s elapsed]
google_compute_subnetwork.cluster-subnet: Destruction complete after 24s
google_compute_network.gke-network: Destroying... [id=projects/qwiklabs-gcp-01-37dcbcf4fa9d/global/networks/kube-net]
google_compute_network.gke-network: Still destroying... [id=projects/qwiklabs-gcp-01-37dcbcf4fa9d/global/networks/kube-net, 10s elapsed]
google_compute_network.gke-network: Still destroying... [id=projects/qwiklabs-gcp-01-37dcbcf4fa9d/global/networks/kube-net, 20s elapsed]
google_compute_network.gke-network: Destruction complete after 22s

Destroy complete! Resources: 5 destroyed.
student_01_41d0c1af2373@cloudshell:~/gke-network-policy-demo (qwiklabs-gcp-01-37dcbcf4fa9d)$ 
```

## Makefile

````makefile
student_01_41d0c1af2373@cloudshell:~/gke-network-policy-demo (qwiklabs-gcp-01-37dcbcf4fa9d)$ cat Makefile 
# Copyright 2018 Google LLC
#
# Licensed under the Apache License, Version 2.0 (the "License");
# you may not use this file except in compliance with the License.
# You may obtain a copy of the License at
#
#     https://www.apache.org/licenses/LICENSE-2.0
#
# Unless required by applicable law or agreed to in writing, software
# distributed under the License is distributed on an "AS IS" BASIS,
# WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
# See the License for the specific language governing permissions and
# limitations under the License.
# Make will use bash instead of sh
SHELL := /usr/bin/env bash
ROOT := ${CURDIR}

# create/delete/validate is for CICD
.PHONY: create
create:
        $(ROOT)/create.sh
.PHONY: delete
teardown:
        $(ROOT)/teardown.sh

.PHONY: validate
validate:
        ${ROOT}/validate.sh

# All is the first target in the file so it will get picked up when you just run 'make' on its own
lint: check_shell check_shebangs check_python check_golang check_terraform check_docker check_base_files check_headers check_trailing_whitespace

# The .PHONY directive tells make that this isn't a real target and so
# the presence of a file named 'check_shell' won't cause this target to stop
# working
.PHONY: check_shell
check_shell:
        @source test/make.sh && check_shell

.PHONY: ci
ci: verify-header

.PHONY: verify-header
verify-header:
        python test/verify_boilerplate.py
        @echo "\n Test passed - Verified all file Apache 2 headers"

.PHONY: setup-project
setup-project:
        # Enables the Google Cloud APIs needed
        ./enable-apis.sh
        # Runs the generate-tfvars.sh
        ./generate-tfvars.sh

.PHONY: tf-apply
tf-apply:
        # Downloads the terraform providers and applies the configuration
        cd terraform && terraform init && terraform apply

.PHONY: tf-destroy
tf-destroy:
        # Downloads the terraform providers and applies the configuration
        cd terraform && terraform destroy


.PHONY: clean-up
clean-up:
        ./remove_manifests.sh

.PHONY: check_python
check_python:
        @source test/make.sh && check_python

.PHONY: check_golang
check_golang:
        @source test/make.sh && golang

.PHONY: check_terraform
check_terraform:
        @source test/make.sh && check_terraform

.PHONY: check_docker
check_docker:
        @source test/make.sh && docker

.PHONY: check_base_files
check_base_files:
        @source test/make.sh && basefiles

.PHONY: check_shebangs
check_shebangs:
        @source test/make.sh && check_bash

.PHONY: check_trailing_whitespace
check_trailing_whitespace:
        @source test/make.sh && check_trailing_whitespace

.PHONY: check_headers
check_headers:
        @echo "Checking file headers"
        @python3.7 test/verify_boilerplate.py
student_01_41d0c1af2373@cloudshell:~/gke-network-policy-demo (qwiklabs-gcp-01-37dcbcf4fa9d)$ 
````

```sh
student_01_41d0c1af2373@cloudshell:~/gke-network-policy-demo (qwiklabs-gcp-01-37dcbcf4fa9d)$ cat create.sh 
#!/bin/bash -e

# Copyright 2018 Google LLC
#
# Licensed under the Apache License, Version 2.0 (the "License");
# you may not use this file except in compliance with the License.
# You may obtain a copy of the License at
#
#     https://www.apache.org/licenses/LICENSE-2.0
#
# Unless required by applicable law or agreed to in writing, software
# distributed under the License is distributed on an "AS IS" BASIS,
# WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
# See the License for the specific language governing permissions and
# limitations under the License.

# "---------------------------------------------------------"
# "-                                                       -"
# "-  Create starts a GKE Cluster and installs             -"
# "-  a Cassandra StatefulSet                              -"
# "-                                                       -"
# "---------------------------------------------------------"

set -o errexit
set -o nounset
set -o pipefail

ROOT="$( cd "$( dirname "${BASH_SOURCE[0]}" )" && pwd )"
# shellcheck disable=SC1090
source "$ROOT"/common.sh

# Enable Compute Engine, Kubernetes Engine, and Container Builder
echo "Enabling the Compute API, the Container API, the Cloud Build API"
gcloud services enable \
compute.googleapis.com \
container.googleapis.com \
cloudbuild.googleapis.com

# Runs the generate-tfvars.sh
"$ROOT"/generate-tfvars.sh

cd "$ROOT"/terraform && \
terraform init -input=false && \
terraform apply -input=false -auto-approve

# Roll out hello-app
"$ROOT"/setup_manifests.sh
student_01_41d0c1af2373@cloudshell:~/gke-network-policy-demo (qwiklabs-gcp-01-37dcbcf4fa9d)$ 
```

```sh
student_01_41d0c1af2373@cloudshell:~/gke-network-policy-demo (qwiklabs-gcp-01-37dcbcf4fa9d)$ cat common.sh 
#!/bin/bash -e

# Copyright 2018 Google LLC
#
# Licensed under the Apache License, Version 2.0 (the "License");
# you may not use this file except in compliance with the License.
# You may obtain a copy of the License at
#
#     https://www.apache.org/licenses/LICENSE-2.0
#
# Unless required by applicable law or agreed to in writing, software
# distributed under the License is distributed on an "AS IS" BASIS,
# WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
# See the License for the specific language governing permissions and
# limitations under the License.

# "---------------------------------------------------------"
# "-                                                       -"
# "-  Common commands for all scripts                      -"
# "-                                                       -"
# "---------------------------------------------------------"

# git is required for this tutorial
command -v git >/dev/null 2>&1 || { \
 echo >&2 "I require git but it's not installed.  Aborting."; exit 1; }

# glcoud is required for this tutorial
command -v gcloud >/dev/null 2>&1 || { \
 echo >&2 "I require gcloud but it's not installed.  Aborting."; exit 1; }

# bastion set up
BASTION_INSTANCE_NAME=gke-demo-bastion
# set to jenkins if there is no $USER
# USER=$(whoami)
# [[ "${USER}" == "root" ]] && export USER=jenkins
# echo "user is: $USER"


# validate deployment status via bastion server
function validate_deployment_bastion() {
    deployment=$1
    ROLLOUT=$(gcloud compute ssh "${USER}"@"${BASTION_INSTANCE_NAME}" --command "kubectl rollout status deploy/${deployment}")
    MESSAGE="deployment \"${deployment}\" successfully rolled out"
    # Test the ROLLOUT variable to see if the grep has returned the expected value.
    # Depending on the test print success or failure messages.
    if [[ $ROLLOUT = *"$MESSAGE"* ]]; then
    echo "Validation Passed: the deployment ${deployment} has been deployed"
    else
    echo "Validation Failed: the deployment ${deployment} has not been deployed"
    exit 1
    fi
}


# gcloud config holds values related to your environment. If you already
# defined a default region we will retrieve it and use it
REGION="$(gcloud config get-value compute/region)"
if [[ -z "${REGION}" ]]; then
    echo "https://cloud.google.com/compute/docs/regions-zones/changing-default-zone-region" 1>&2
    echo "gcloud cli must be configured with a default region." 1>&2
    echo "run 'gcloud config set compute/region REGION'." 1>&2
    echo "replace 'REGION' with the region name like us-west1." 1>&2
    exit 1;
fi

# gcloud config holds values related to your environment. If you already
# defined a default zone we will retrieve it and use it
ZONE="$(gcloud config get-value compute/zone)"
if [[ -z "${ZONE}" ]]; then
    echo "https://cloud.google.com/compute/docs/regions-zones/changing-default-zone-region" 1>&2
    echo "gcloud cli must be configured with a default zone." 1>&2
    echo "run 'gcloud config set compute/zone ZONE'." 1>&2
    echo "replace 'ZONE' with the zone name like us-west1-a." 1>&2
    exit 1;
fi

# gcloud config holds values related to your environment. If you already
# defined a default project we will retrieve it and use it
PROJECT="$(gcloud config get-value core/project)"
if [[ -z "${PROJECT}" ]]; then
    echo "gcloud cli must be configured with a default project." 1>&2
    echo "run 'gcloud config set core/project PROJECT'." 1>&2
    echo "replace 'PROJECT' with the project name." 1>&2
    exit 1;
fi

student_01_41d0c1af2373@cloudshell:~/gke-network-policy-demo (qwiklabs-gcp-01-37dcbcf4fa9d)$ 
```

```sh
student_01_41d0c1af2373@cloudshell:~/gke-network-policy-demo (qwiklabs-gcp-01-37dcbcf4fa9d)$ cat teardown.sh 
#!/bin/bash -e

# Copyright 2018 Google LLC
#
# Licensed under the Apache License, Version 2.0 (the "License");
# you may not use this file except in compliance with the License.
# You may obtain a copy of the License at
#
#     https://www.apache.org/licenses/LICENSE-2.0
#
# Unless required by applicable law or agreed to in writing, software
# distributed under the License is distributed on an "AS IS" BASIS,
# WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
# See the License for the specific language governing permissions and
# limitations under the License.

# "---------------------------------------------------------"
# "-                                                       -"
# "-  Delete uninstalls Cassandra and deletes              -"
# "-  the GKE cluster                                      -"
# "-                                                       -"
# "---------------------------------------------------------"

# Do not set errexit as it makes partial deletes impossible
set -o nounset
set -o pipefail

ROOT="$( cd "$( dirname "${BASH_SOURCE[0]}" )" && pwd )"
# shellcheck disable=SC1090
source "$ROOT"/common.sh

# Tear down hello-app
gcloud compute ssh "${USER}"@"${BASTION_INSTANCE_NAME}" --command "kubectl delete -f manifests/hello-app/"

# remove metadata for bastion (SSH keys)
gcloud compute instances remove-metadata --all "${BASTION_INSTANCE_NAME}" --project "${PROJECT}" --zone "${ZONE}"

# Terraform destroy
cd terraform && terraform destroy -auto-approve

student_01_41d0c1af2373@cloudshell:~/gke-network-policy-demo (qwiklabs-gcp-01-37dcbcf4fa9d)$ 
```

```sh
student_01_41d0c1af2373@cloudshell:~/gke-network-policy-demo (qwiklabs-gcp-01-37dcbcf4fa9d)$ cat validate.sh 
#!/usr/bin/env bash
# Copyright 2018 Google LLC
#
# Licensed under the Apache License, Version 2.0 (the "License");
# you may not use this file except in compliance with the License.
# You may obtain a copy of the License at
#
#     https://www.apache.org/licenses/LICENSE-2.0
#
# Unless required by applicable law or agreed to in writing, software
# distributed under the License is distributed on an "AS IS" BASIS,
# WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
# See the License for the specific language governing permissions and
# limitations under the License.

# bash "strict-mode", fail immediately if there is a problem
set -euo pipefail

ROOT="$( cd "$( dirname "${BASH_SOURCE[0]}" )" && pwd )"
# shellcheck disable=SC1090
source "$ROOT"/common.sh

HELLO_WORLD='Hello, world!'
TIMED_OUT='wget: download timed out'

# Kubectl logs of the hello app pod will return $TIMED_OUT for the first 5 second.
# So let's wait for a while before we start pulling logs
sleep 20

# A helper method to make calls to the gke cluster through the bastion.
call_bastion() {
  local command=$1; shift;
  # shellcheck disable=SC2005
  echo "$(gcloud compute ssh "$USER"@gke-demo-bastion --command "${command}")"
}

# We expect to see "Hello, world!" in the logs with the app=hello label.
call_bastion "kubectl logs --tail 10 \$(kubectl get pods -oname -l app=hello)" \
| grep "$HELLO_WORLD" &> /dev/null || exit 1
echo "step 1 of the validation passed."

# We expect to see the same behavior in the logs with the app=not-hello label
# until the network-policy is applied.
call_bastion "kubectl logs --tail 10 \
 \$(kubectl get pods -oname -l app=not-hello)" | grep "$HELLO_WORLD" \
 &> /dev/null || exit 1
echo "step 2 of the validation passed."

# Apply the network policy.
call_bastion "kubectl apply -f ./manifests/network-policy.yaml" &> /dev/null

# Sleep for 10s while more logs come in.
sleep 10

# Now we expect to see a 'timed out' message because the network policy
# prevents the communication.
call_bastion "kubectl logs --tail 10 \
 \$(kubectl get pods -oname -l app=not-hello)" | grep "$TIMED_OUT" \
 &> /dev/null || exit 1
echo "step 3 of the validation passed."

# If the network policy is working correctly, we still see the original behavior
# in the logs with the app=hello label.
call_bastion "kubectl logs --tail 10 \$(kubectl get pods -oname -l app=hello)" \
| grep "$HELLO_WORLD" &> /dev/null || exit 1
echo "step 4 of the validation passed."

call_bastion "kubectl delete -f ./manifests/network-policy.yaml" &> /dev/null
student_01_41d0c1af2373@cloudshell:~/gke-network-policy-demo (qwiklabs-gcp-01-37dcbcf4fa9d)$ 
```

