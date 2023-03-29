---

layout: single
title:  "Deploy Kubernetes Load Balancer Service with Terraform"
date:   2023-03-29 12:59:04 +0530
categories: Cloud
tags: GCP
classes: wide
show_date: true
header:
  overlay_image: /assets/images/gpcne.png
  og_image: /assets/images/gpcne.png
  teaser: /assets/images/gpcne.png
author:
  name     : "Cloud Network Engineer"
  avatar   : "/assets/images/gpcne.png"

sidebar:
  - title: "Topics"
    nav: my-sidebar

---

# Deploy Kubernetes Load Balancer Service with Terraform

Deploy a Kubernetes cluster along with a service using Terraform

While you could use `kubectl` or similar CLI-based tools  mapped to API calls to manage all Kubernetes resources described in YAML files, orchestration with Terraform presents a few benefits:

- **One language** - You can use the same [configuration language](https://www.terraform.io/docs/configuration/syntax.html) to provision the Kubernetes infrastructure and to deploy applications into it.
- **Drift detection** - `terraform plan` will always present you the difference between reality at a given time and the config you intend to apply.
- **Full lifecycle management** - Terraform doesn't just  initially create resources, but offers a single command to create,  update, and delete tracked resources without needing to inspect the API  to identify those resources.
- **Synchronous feedback** - While asynchronous behavior is  often useful, sometimes it's counter-productive as the job of  identifying operation results (failures or details of created resource)  is left to the user. e.g. you don't have the IP/hostname of the load  balancer until it has finished provisioning, hence you can't create any  DNS record pointing to it.
- **[Graph of relationships](https://www.terraform.io/docs/internals/graph.html)** - Terraform understands relationships between resources which may help  in scheduling - e.g. Terraform won't try to create a service in a  Kubernetes cluster until the cluster exists.



```sh

Welcome to Cloud Shell! Type "help" to get started.
Your Cloud Platform project in this session is set to qwiklabs-gcp-04-7f0fc69ae096.
Use “gcloud config set project [PROJECT_ID]” to change to a different project.
student_01_8961e1e5d696@cloudshell:~ (qwiklabs-gcp-04-7f0fc69ae096)$ gsutil -m cp -r gs://spls/gsp233/* .
Copying gs://spls/gsp233/tf-gke-k8s-service-lb/versions.tf...
Copying gs://spls/gsp233/tf-gke-k8s-service-lb/k8s.tf...
Copying gs://spls/gsp233/tf-gke-k8s-service-lb/README.md...
Copying gs://spls/gsp233/tf-gke-k8s-service-lb/main.tf...
Copying gs://spls/gsp233/tf-gke-k8s-service-lb/test.sh...
student_01_8961e1e5d696@cloudshell:~ (qwiklabs-gcp-04-7f0fc69ae096)$ cd tf-gke-k8s-service-lb
student_01_8961e1e5d696@cloudshell:~/tf-gke-k8s-service-lb (qwiklabs-gcp-04-7f0fc69ae096)$ ls
k8s.tf  main.tf  README.md  test.sh  versions.tf
student_01_8961e1e5d696@cloudshell:~/tf-gke-k8s-service-lb (qwiklabs-gcp-04-7f0fc69ae096)$ cat k8s.tf
provider "kubernetes" {
  version = "~> 1.10.0"
  host    = google_container_cluster.default.endpoint
  token   = data.google_client_config.current.access_token
  client_certificate = base64decode(
    google_container_cluster.default.master_auth[0].client_certificate,
  )
  client_key = base64decode(google_container_cluster.default.master_auth[0].client_key)
  cluster_ca_certificate = base64decode(
    google_container_cluster.default.master_auth[0].cluster_ca_certificate,
  )
}

resource "kubernetes_namespace" "staging" {
  metadata {
    name = "staging"
  }
}

resource "google_compute_address" "default" {
  name   = var.network_name
  region = var.region
}

resource "kubernetes_service" "nginx" {
  metadata {
    namespace = kubernetes_namespace.staging.metadata[0].name
    name      = "nginx"
  }

  spec {
    selector = {
      run = "nginx"
    }

    session_affinity = "ClientIP"

    port {
      protocol    = "TCP"
      port        = 80
      target_port = 80
    }

    type             = "LoadBalancer"
    load_balancer_ip = google_compute_address.default.address
  }
}

resource "kubernetes_replication_controller" "nginx" {
  metadata {
    name      = "nginx"
    namespace = kubernetes_namespace.staging.metadata[0].name

    labels = {
      run = "nginx"
    }
  }

  spec {
    selector = {
      run = "nginx"
    }

    template {
      metadata {
          name = "nginx"
          labels = {
              run = "nginx"
          }
      }

      spec {
        container {
            image = "nginx:latest"
            name  = "nginx"

            resources {
                limits {
                    cpu    = "0.5"
                    memory = "512Mi"
                }

                requests {
                    cpu    = "250m"
                    memory = "50Mi"
                }
            }
        }
      }
    }
  }
}

output "load-balancer-ip" {
  value = google_compute_address.default.address
}

student_01_8961e1e5d696@cloudshell:~/tf-gke-k8s-service-lb (qwiklabs-gcp-04-7f0fc69ae096)$ ls
k8s.tf  main.tf  README.md  test.sh  versions.tf
student_01_8961e1e5d696@cloudshell:~/tf-gke-k8s-service-lb (qwiklabs-gcp-04-7f0fc69ae096)$ cat main.tf
variable "region" {
  type        = string
  description = "Region for the resource."
}

variable "location" {
  type        = string
  description = "Location represents region/zone for the resource."
}

variable "network_name" {
  default = "tf-gke-k8s"
}

provider "google" {
  region = var.region
}

resource "google_compute_network" "default" {
  name                    = var.network_name
  auto_create_subnetworks = false
}

resource "google_compute_subnetwork" "default" {
  name                     = var.network_name
  ip_cidr_range            = "10.127.0.0/20"
  network                  = google_compute_network.default.self_link
  region                   = var.region
  private_ip_google_access = true
}

data "google_client_config" "current" {
}

data "google_container_engine_versions" "default" {
  location = var.location
}

resource "google_container_cluster" "default" {
  name               = var.network_name
  location           = var.location
  initial_node_count = 3
  min_master_version = data.google_container_engine_versions.default.latest_master_version
  network            = google_compute_subnetwork.default.name
  subnetwork         = google_compute_subnetwork.default.name

  // Use legacy ABAC until these issues are resolved:
  //   https://github.com/mcuadros/terraform-provider-helm/issues/56
  //   https://github.com/terraform-providers/terraform-provider-kubernetes/pull/73
  enable_legacy_abac = true

  // Wait for the GCE LB controller to cleanup the resources.
  // Wait for the GCE LB controller to cleanup the resources.
  provisioner "local-exec" {
    when    = destroy
    command = "sleep 90"
  }
}

output "network" {
  value = google_compute_subnetwork.default.network
}

output "subnetwork_name" {
  value = google_compute_subnetwork.default.name
}

output "cluster_name" {
  value = google_container_cluster.default.name
}

output "cluster_region" {
  value = var.region
}

output "cluster_location" {
  value = google_container_cluster.default.location
}

student_01_8961e1e5d696@cloudshell:~/tf-gke-k8s-service-lb (qwiklabs-gcp-04-7f0fc69ae096)$ ls
k8s.tf  main.tf  README.md  test.sh  versions.tf
student_01_8961e1e5d696@cloudshell:~/tf-gke-k8s-service-lb (qwiklabs-gcp-04-7f0fc69ae096)$ cat versions.tf

terraform {
  required_version = ">= 0.12"
}
student_01_8961e1e5d696@cloudshell:~/tf-gke-k8s-service-lb (qwiklabs-gcp-04-7f0fc69ae096)$ cat test.sh
#!/usr/bin/env bash

set -x
set -e

URL="http://$(terraform output load-balancer-ip)"
status=0
count=0
while [[ $count -lt 120 && $status -ne 200 ]]; do
  echo "INFO: Waiting for load balancer..."
  status=$(curl -sf -m 5 -o /dev/null -w "%{http_code}" "${URL}" || true)
  ((count=count+1))
  sleep 5
done
if [[ $count -lt 120 ]]; then
  echo "INFO: PASS"
else
  echo "ERROR: Failed"
fistudent_01_8961e1e5d696@cloudshell:~/tf-gke-k8s-service-lb (qwiklabs-gcp-04-7f0fc69ae096)$
```



```sh
student_01_8961e1e5d696@cloudshell:~/tf-gke-k8s-service-lb (qwiklabs-gcp-04-7f0fc69ae096)$ terraform init

Initializing the backend...

Initializing provider plugins...
- Finding hashicorp/kubernetes versions matching "~> 1.10.0"...
- Finding latest version of hashicorp/google...
- Installing hashicorp/kubernetes v1.10.0...
- Installed hashicorp/kubernetes v1.10.0 (signed by HashiCorp)
- Installing hashicorp/google v4.59.0...
- Installed hashicorp/google v4.59.0 (signed by HashiCorp)

Terraform has created a lock file .terraform.lock.hcl to record the provider
selections it made above. Include this file in your version control repository
so that Terraform can guarantee to make the same selections by default when
you run "terraform init" in the future.

╷
│ Warning: Version constraints inside provider configuration blocks are deprecated
│
│   on k8s.tf line 2, in provider "kubernetes":
│    2:   version = "~> 1.10.0"
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
student_01_8961e1e5d696@cloudshell:~/tf-gke-k8s-service-lb (qwiklabs-gcp-04-7f0fc69ae096)$
```





```sh
student_01_8961e1e5d696@cloudshell:~/tf-gke-k8s-service-lb (qwiklabs-gcp-04-7f0fc69ae096)$ terraform apply -var="region=us-central1" -var="location=us-central1-f"
data.google_container_engine_versions.default: Reading...
data.google_client_config.current: Reading...
data.google_client_config.current: Read complete after 0s [id=projects/qwiklabs-gcp-04-7f0fc69ae096/regions/us-central1/zones/]
data.google_container_engine_versions.default: Read complete after 1s [id=2023-03-29 18:21:37.909430461 +0000 UTC]

Terraform used the selected providers to generate the following execution plan. Resource actions are indicated with the following symbols:
  + create

Terraform will perform the following actions:

  # google_compute_address.default will be created
  + resource "google_compute_address" "default" {
      + address            = (known after apply)
      + address_type       = "EXTERNAL"
      + creation_timestamp = (known after apply)
      + id                 = (known after apply)
      + name               = "tf-gke-k8s"
      + network_tier       = (known after apply)
      + project            = (known after apply)
      + purpose            = (known after apply)
      + region             = "us-central1"
      + self_link          = (known after apply)
      + subnetwork         = (known after apply)
      + users              = (known after apply)
    }

  # google_compute_network.default will be created
  + resource "google_compute_network" "default" {
      + auto_create_subnetworks         = false
      + delete_default_routes_on_create = false
      + gateway_ipv4                    = (known after apply)
      + id                              = (known after apply)
      + internal_ipv6_range             = (known after apply)
      + mtu                             = (known after apply)
      + name                            = "tf-gke-k8s"
      + project                         = (known after apply)
      + routing_mode                    = (known after apply)
      + self_link                       = (known after apply)
    }

  # google_compute_subnetwork.default will be created
  + resource "google_compute_subnetwork" "default" {
      + creation_timestamp         = (known after apply)
      + external_ipv6_prefix       = (known after apply)
      + fingerprint                = (known after apply)
      + gateway_address            = (known after apply)
      + id                         = (known after apply)
      + ip_cidr_range              = "10.127.0.0/20"
      + ipv6_cidr_range            = (known after apply)
      + name                       = "tf-gke-k8s"
      + network                    = (known after apply)
      + private_ip_google_access   = true
      + private_ipv6_google_access = (known after apply)
      + project                    = (known after apply)
      + purpose                    = (known after apply)
      + region                     = "us-central1"
      + secondary_ip_range         = (known after apply)
      + self_link                  = (known after apply)
      + stack_type                 = (known after apply)
    }

  # google_container_cluster.default will be created
  + resource "google_container_cluster" "default" {
      + cluster_ipv4_cidr           = (known after apply)
      + datapath_provider           = (known after apply)
      + default_max_pods_per_node   = (known after apply)
      + enable_binary_authorization = false
      + enable_intranode_visibility = (known after apply)
      + enable_kubernetes_alpha     = false
      + enable_l4_ilb_subsetting    = false
      + enable_legacy_abac          = true
      + enable_shielded_nodes       = true
      + endpoint                    = (known after apply)
      + id                          = (known after apply)
      + initial_node_count          = 3
      + label_fingerprint           = (known after apply)
      + location                    = "us-central1-f"
      + logging_service             = (known after apply)
      + master_version              = (known after apply)
      + min_master_version          = "1.25.7-gke.1000"
      + monitoring_service          = (known after apply)
      + name                        = "tf-gke-k8s"
      + network                     = "tf-gke-k8s"
      + networking_mode             = (known after apply)
      + node_locations              = (known after apply)
      + node_version                = (known after apply)
      + operation                   = (known after apply)
      + private_ipv6_google_access  = (known after apply)
      + project                     = (known after apply)
      + self_link                   = (known after apply)
      + services_ipv4_cidr          = (known after apply)
      + subnetwork                  = "tf-gke-k8s"
      + tpu_ipv4_cidr_block         = (known after apply)
    }

  # kubernetes_namespace.staging will be created
  + resource "kubernetes_namespace" "staging" {
      + id = (known after apply)

      + metadata {
          + generation       = (known after apply)
          + name             = "staging"
          + resource_version = (known after apply)
          + self_link        = (known after apply)
          + uid              = (known after apply)
        }
    }

  # kubernetes_replication_controller.nginx will be created
  + resource "kubernetes_replication_controller" "nginx" {
      + id = (known after apply)

      + metadata {
          + generation       = (known after apply)
          + labels           = {
              + "run" = "nginx"
            }
          + name             = "nginx"
          + namespace        = "staging"
          + resource_version = (known after apply)
          + self_link        = (known after apply)
          + uid              = (known after apply)
        }

      + spec {
          + min_ready_seconds = 0
          + replicas          = 1
          + selector          = {
              + "run" = "nginx"
            }

          + template {
              + active_deadline_seconds          = (known after apply)
              + dns_policy                       = (known after apply)
              + host_ipc                         = (known after apply)
              + host_network                     = (known after apply)
              + host_pid                         = (known after apply)
              + hostname                         = (known after apply)
              + node_name                        = (known after apply)
              + node_selector                    = (known after apply)
              + priority_class_name              = (known after apply)
              + restart_policy                   = (known after apply)
              + service_account_name             = (known after apply)
              + share_process_namespace          = false
              + subdomain                        = (known after apply)
              + termination_grace_period_seconds = (known after apply)

              + metadata {
                  + generation       = (known after apply)
                  + labels           = {
                      + "run" = "nginx"
                    }
                  + name             = "nginx"
                  + resource_version = (known after apply)
                  + self_link        = (known after apply)
                  + uid              = (known after apply)
                }

              + spec {
                  + active_deadline_seconds          = (known after apply)
                  + dns_policy                       = (known after apply)
                  + host_ipc                         = (known after apply)
                  + host_network                     = (known after apply)
                  + host_pid                         = (known after apply)
                  + hostname                         = (known after apply)
                  + node_name                        = (known after apply)
                  + node_selector                    = (known after apply)
                  + priority_class_name              = (known after apply)
                  + restart_policy                   = (known after apply)
                  + service_account_name             = (known after apply)
                  + share_process_namespace          = false
                  + subdomain                        = (known after apply)
                  + termination_grace_period_seconds = (known after apply)

                  + container {
                      + image                    = "nginx:latest"
                      + image_pull_policy        = (known after apply)
                      + name                     = "nginx"
                      + stdin                    = false
                      + stdin_once               = false
                      + termination_message_path = "/dev/termination-log"
                      + tty                      = false

                      + resources {
                          + limits {
                              + cpu    = "0.5"
                              + memory = "512Mi"
                            }
                          + requests {
                              + cpu    = "250m"
                              + memory = "50Mi"
                            }
                        }
                    }
                }
            }
        }
    }

  # kubernetes_service.nginx will be created
  + resource "kubernetes_service" "nginx" {
      + id                    = (known after apply)
      + load_balancer_ingress = (known after apply)

      + metadata {
          + generation       = (known after apply)
          + name             = "nginx"
          + namespace        = "staging"
          + resource_version = (known after apply)
          + self_link        = (known after apply)
          + uid              = (known after apply)
        }

      + spec {
          + cluster_ip                  = (known after apply)
          + external_traffic_policy     = (known after apply)
          + load_balancer_ip            = (known after apply)
          + publish_not_ready_addresses = false
          + selector                    = {
              + "run" = "nginx"
            }
          + session_affinity            = "ClientIP"
          + type                        = "LoadBalancer"

          + port {
              + node_port   = (known after apply)
              + port        = 80
              + protocol    = "TCP"
              + target_port = "80"
            }
        }
    }

Plan: 7 to add, 0 to change, 0 to destroy.

Changes to Outputs:
  + cluster_location = "us-central1-f"
  + cluster_name     = "tf-gke-k8s"
  + cluster_region   = "us-central1"
  + load-balancer-ip = (known after apply)
  + network          = (known after apply)
  + subnetwork_name  = "tf-gke-k8s"
╷
│ Warning: Version constraints inside provider configuration blocks are deprecated
│
│   on k8s.tf line 2, in provider "kubernetes":
│    2:   version = "~> 1.10.0"
│
│ Terraform 0.13 and earlier allowed provider version constraints inside the provider configuration block, but that is now deprecated and will be removed in a future version of
│ Terraform. To silence this warning, move the provider version constraint into the required_providers block.
╵

Do you want to perform these actions?
  Terraform will perform the actions described above.
  Only 'yes' will be accepted to approve.

  Enter a value: yes

google_compute_network.default: Creating...
google_compute_address.default: Creating...
google_compute_network.default: Still creating... [10s elapsed]
google_compute_address.default: Still creating... [10s elapsed]
google_compute_address.default: Creation complete after 16s [id=projects/qwiklabs-gcp-04-7f0fc69ae096/regions/us-central1/addresses/tf-gke-k8s]
google_compute_network.default: Still creating... [20s elapsed]
google_compute_network.default: Creation complete after 22s [id=projects/qwiklabs-gcp-04-7f0fc69ae096/global/networks/tf-gke-k8s]
google_compute_subnetwork.default: Creating...
google_compute_subnetwork.default: Still creating... [10s elapsed]
google_compute_subnetwork.default: Still creating... [20s elapsed]
google_compute_subnetwork.default: Creation complete after 24s [id=projects/qwiklabs-gcp-04-7f0fc69ae096/regions/us-central1/subnetworks/tf-gke-k8s]
google_container_cluster.default: Creating...
google_container_cluster.default: Still creating... [10s elapsed]
google_container_cluster.default: Still creating... [20s elapsed]
google_container_cluster.default: Still creating... [30s elapsed]
google_container_cluster.default: Still creating... [40s elapsed]
google_container_cluster.default: Still creating... [50s elapsed]
google_container_cluster.default: Still creating... [1m0s elapsed]
google_container_cluster.default: Still creating... [1m10s elapsed]
google_container_cluster.default: Still creating... [1m20s elapsed]
google_container_cluster.default: Still creating... [1m30s elapsed]
google_container_cluster.default: Still creating... [1m40s elapsed]
google_container_cluster.default: Still creating... [1m50s elapsed]
google_container_cluster.default: Still creating... [2m0s elapsed]
google_container_cluster.default: Still creating... [2m10s elapsed]
google_container_cluster.default: Still creating... [2m20s elapsed]
google_container_cluster.default: Still creating... [2m30s elapsed]
google_container_cluster.default: Still creating... [2m40s elapsed]
google_container_cluster.default: Still creating... [2m50s elapsed]
google_container_cluster.default: Still creating... [3m0s elapsed]
google_container_cluster.default: Still creating... [3m10s elapsed]
google_container_cluster.default: Still creating... [3m20s elapsed]
google_container_cluster.default: Still creating... [3m30s elapsed]
google_container_cluster.default: Still creating... [3m40s elapsed]
google_container_cluster.default: Still creating... [3m50s elapsed]
google_container_cluster.default: Still creating... [4m0s elapsed]
google_container_cluster.default: Still creating... [4m10s elapsed]
google_container_cluster.default: Still creating... [4m20s elapsed]
google_container_cluster.default: Still creating... [4m30s elapsed]
google_container_cluster.default: Still creating... [4m40s elapsed]
google_container_cluster.default: Still creating... [4m50s elapsed]
google_container_cluster.default: Still creating... [5m0s elapsed]
google_container_cluster.default: Still creating... [5m10s elapsed]
google_container_cluster.default: Still creating... [5m20s elapsed]
google_container_cluster.default: Creation complete after 5m24s [id=projects/qwiklabs-gcp-04-7f0fc69ae096/locations/us-central1-f/clusters/tf-gke-k8s]
kubernetes_namespace.staging: Creating...
kubernetes_namespace.staging: Creation complete after 1s [id=staging]
kubernetes_service.nginx: Creating...
kubernetes_replication_controller.nginx: Creating...
kubernetes_replication_controller.nginx: Creation complete after 1s [id=staging/nginx]
kubernetes_service.nginx: Still creating... [10s elapsed]
kubernetes_service.nginx: Still creating... [20s elapsed]
kubernetes_service.nginx: Still creating... [30s elapsed]
kubernetes_service.nginx: Creation complete after 38s [id=staging/nginx]

Apply complete! Resources: 7 added, 0 changed, 0 destroyed.

Outputs:

cluster_location = "us-central1-f"
cluster_name = "tf-gke-k8s"
cluster_region = "us-central1"
load-balancer-ip = "34.134.52.198"
network = "https://www.googleapis.com/compute/v1/projects/qwiklabs-gcp-04-7f0fc69ae096/global/networks/tf-gke-k8s"
subnetwork_name = "tf-gke-k8s"
student_01_8961e1e5d696@cloudshell:~/tf-gke-k8s-service-lb (qwiklabs-gcp-04-7f0fc69ae096)$
```



![]({{ site.url }}{{ site.baseurl }}/assets/images/gcp-213.png)



![]({{ site.url }}{{ site.baseurl }}/assets/images/gcp-214.png)



![]({{ site.url }}{{ site.baseurl }}/assets/images/gcp-215.png)



![]({{ site.url }}{{ site.baseurl }}/assets/images/gcp-216.png)



```sh
student_01_8961e1e5d696@cloudshell:~/tf-gke-k8s-service-lb (qwiklabs-gcp-04-7f0fc69ae096)$ terraform destroy -var="region=us-central1" -var="location=us-central1-f"
google_compute_network.default: Refreshing state... [id=projects/qwiklabs-gcp-04-7f0fc69ae096/global/networks/tf-gke-k8s]
data.google_client_config.current: Reading...
google_compute_address.default: Refreshing state... [id=projects/qwiklabs-gcp-04-7f0fc69ae096/regions/us-central1/addresses/tf-gke-k8s]
data.google_container_engine_versions.default: Reading...
data.google_client_config.current: Read complete after 1s [id=projects/qwiklabs-gcp-04-7f0fc69ae096/regions/us-central1/zones/]
google_compute_subnetwork.default: Refreshing state... [id=projects/qwiklabs-gcp-04-7f0fc69ae096/regions/us-central1/subnetworks/tf-gke-k8s]
data.google_container_engine_versions.default: Read complete after 1s [id=2023-03-29 18:32:12.941543985 +0000 UTC]
google_container_cluster.default: Refreshing state... [id=projects/qwiklabs-gcp-04-7f0fc69ae096/locations/us-central1-f/clusters/tf-gke-k8s]
kubernetes_namespace.staging: Refreshing state... [id=staging]
kubernetes_service.nginx: Refreshing state... [id=staging/nginx]
kubernetes_replication_controller.nginx: Refreshing state... [id=staging/nginx]

Terraform used the selected providers to generate the following execution plan. Resource actions are indicated with the following symbols:
  - destroy

Terraform will perform the following actions:

  # google_compute_address.default will be destroyed
  - resource "google_compute_address" "default" {
      - address            = "34.134.52.198" -> null
      - address_type       = "EXTERNAL" -> null
      - creation_timestamp = "2023-03-29T11:21:52.949-07:00" -> null
      - id                 = "projects/qwiklabs-gcp-04-7f0fc69ae096/regions/us-central1/addresses/tf-gke-k8s" -> null
      - name               = "tf-gke-k8s" -> null
      - network_tier       = "PREMIUM" -> null
      - prefix_length      = 0 -> null
      - project            = "qwiklabs-gcp-04-7f0fc69ae096" -> null
      - region             = "us-central1" -> null
      - self_link          = "https://www.googleapis.com/compute/v1/projects/qwiklabs-gcp-04-7f0fc69ae096/regions/us-central1/addresses/tf-gke-k8s" -> null
      - users              = [
          - "https://www.googleapis.com/compute/v1/projects/qwiklabs-gcp-04-7f0fc69ae096/regions/us-central1/forwardingRules/afbab9b37c68d443bab0a82540df21f4",
        ] -> null
    }

  # google_compute_network.default will be destroyed
  - resource "google_compute_network" "default" {
      - auto_create_subnetworks         = false -> null
      - delete_default_routes_on_create = false -> null
      - enable_ula_internal_ipv6        = false -> null
      - id                              = "projects/qwiklabs-gcp-04-7f0fc69ae096/global/networks/tf-gke-k8s" -> null
      - mtu                             = 0 -> null
      - name                            = "tf-gke-k8s" -> null
      - project                         = "qwiklabs-gcp-04-7f0fc69ae096" -> null
      - routing_mode                    = "REGIONAL" -> null
      - self_link                       = "https://www.googleapis.com/compute/v1/projects/qwiklabs-gcp-04-7f0fc69ae096/global/networks/tf-gke-k8s" -> null
    }

  # google_compute_subnetwork.default will be destroyed
  - resource "google_compute_subnetwork" "default" {
      - creation_timestamp         = "2023-03-29T11:22:13.087-07:00" -> null
      - gateway_address            = "10.127.0.1" -> null
      - id                         = "projects/qwiklabs-gcp-04-7f0fc69ae096/regions/us-central1/subnetworks/tf-gke-k8s" -> null
      - ip_cidr_range              = "10.127.0.0/20" -> null
      - name                       = "tf-gke-k8s" -> null
      - network                    = "https://www.googleapis.com/compute/v1/projects/qwiklabs-gcp-04-7f0fc69ae096/global/networks/tf-gke-k8s" -> null
      - private_ip_google_access   = true -> null
      - private_ipv6_google_access = "DISABLE_GOOGLE_ACCESS" -> null
      - project                    = "qwiklabs-gcp-04-7f0fc69ae096" -> null
      - purpose                    = "PRIVATE" -> null
      - region                     = "us-central1" -> null
      - secondary_ip_range         = [] -> null
      - self_link                  = "https://www.googleapis.com/compute/v1/projects/qwiklabs-gcp-04-7f0fc69ae096/regions/us-central1/subnetworks/tf-gke-k8s" -> null
      - stack_type                 = "IPV4_ONLY" -> null
    }

  # google_container_cluster.default will be destroyed
  - resource "google_container_cluster" "default" {
      - cluster_ipv4_cidr           = "10.88.0.0/14" -> null
      - enable_autopilot            = false -> null
      - enable_binary_authorization = false -> null
      - enable_intranode_visibility = false -> null
      - enable_kubernetes_alpha     = false -> null
      - enable_l4_ilb_subsetting    = false -> null
      - enable_legacy_abac          = true -> null
      - enable_shielded_nodes       = true -> null
      - enable_tpu                  = false -> null
      - endpoint                    = "35.238.117.234" -> null
      - id                          = "projects/qwiklabs-gcp-04-7f0fc69ae096/locations/us-central1-f/clusters/tf-gke-k8s" -> null
      - initial_node_count          = 3 -> null
      - label_fingerprint           = "a9dc16a7" -> null
      - location                    = "us-central1-f" -> null
      - logging_service             = "logging.googleapis.com/kubernetes" -> null
      - master_version              = "1.25.7-gke.1000" -> null
      - min_master_version          = "1.25.7-gke.1000" -> null
      - monitoring_service          = "monitoring.googleapis.com/kubernetes" -> null
      - name                        = "tf-gke-k8s" -> null
      - network                     = "projects/qwiklabs-gcp-04-7f0fc69ae096/global/networks/tf-gke-k8s" -> null
      - networking_mode             = "ROUTES" -> null
      - node_locations              = [] -> null
      - node_version                = "1.25.7-gke.1000" -> null
      - project                     = "qwiklabs-gcp-04-7f0fc69ae096" -> null
      - resource_labels             = {} -> null
      - self_link                   = "https://container.googleapis.com/v1/projects/qwiklabs-gcp-04-7f0fc69ae096/zones/us-central1-f/clusters/tf-gke-k8s" -> null
      - services_ipv4_cidr          = "10.91.240.0/20" -> null
      - subnetwork                  = "projects/qwiklabs-gcp-04-7f0fc69ae096/regions/us-central1/subnetworks/tf-gke-k8s" -> null

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
          - enabled = false -> null
        }

      - database_encryption {
          - state = "DECRYPTED" -> null
        }

      - default_snat_status {
          - disabled = false -> null
        }

      - logging_config {
          - enable_components = [
              - "SYSTEM_COMPONENTS",
              - "WORKLOADS",
            ] -> null
        }

      - master_auth {
          - cluster_ca_certificate = "LS0tLS1CRUdJTiBDRVJUSUZJQ0FURS0tLS0tCk1JSUVMRENDQXBTZ0F3SUJBZ0lRZHo1enBVcHBSWmlrVGF3OHJjZ0M1VEFOQmdrcWhraUc5dzBCQVFzRkFEQXYKTVMwd0t3WURWUVFERXlSbU1tUmpZekppWVMwMk1tVmlMVFExTTJFdFlUZzBaUzFsTWpnd1lURXlOemhsWm1FdwpJQmNOTWpNd016STVNVGN5TWpNMldoZ1BNakExTXpBek1qRXhPREl5TXpaYU1DOHhMVEFyQmdOVkJBTVRKR1l5ClpHTmpNbUpoTFRZeVpXSXRORFV6WVMxaE9EUmxMV1V5T0RCaE1USTNPR1ZtWVRDQ0FhSXdEUVlKS29aSWh2Y04KQVFFQkJRQURnZ0dQQURDQ0FZb0NnZ0dCQUtnbFo3UHJmTFExaUd2QTNMWWtveFg4eGJNbTQ1cFB0cUp4UW1FbwozN1lJbkRzNUJMOGU4NUVsSGxDUGdYa00rTzFxSWc1M29OdFg4azFNdDhPU2h6RzdUVHVSTmZVd1E2bXkrc1QvClR3YmpETmROVEFscEt4UTJCanlqQVpkMVRSR1JBaWxmZ0xEclBQeHhma2ZtQ2lFcjNhWjNvMW93Z1J0bWVoZTQKRU9nMXNiQTJHSlNrUy9Bbys5UGw5dnlZUEF6OFJvdWlxODNKTmVRczlNamZvcGNNbG9zRmZLcDNVWlZnb1dSZQpUMEhudTc5dnhJZVMxeWNEdXU3dkhjejBxVnFBU1JYNldIUnAyQWpMQVlFOXA3ZEQ1T2ZXNWthR2NQYVFwSnlXCitCTnVCM1BNa01yV1ExNVhYbTIzQnd3T0dJMURvdGVuTlM5bE9nQ0N2VnFrcWYvWUJmUWYzQ09MNSsxYXNZVjgKaVoxZFBneWltOUJXOUF1VDVpdExRRGFsYisrZjcvTHFoem1mTDNkbEZTM3NGUnN6SE03L0M2QWNHUzZDcUtZQQp6UDJIMXFvNURBalpOTUxNZVF5WVFMWDg3YmhxclQ0czU5bGxhSGhGUnZvMm44dDJLV2ZBOURSZTIvdjFqblRtCnYvNUo4cmpuWGcyM0EzNS9sMzVRQ3RqWEN3SURBUUFCbzBJd1FEQU9CZ05WSFE4QkFmOEVCQU1DQWdRd0R3WUQKVlIwVEFRSC9CQVV3QXdFQi96QWRCZ05WSFE0RUZnUVU2d0tVK1dtVlpzVkd3eThCM3dhV0xVa1AzdmN3RFFZSgpLb1pJaHZjTkFRRUxCUUFEZ2dHQkFKUjY1QnE4aE1Vb1VhU3FLaGtWdGtQVWVUOEdvNlVtUnBYYkF0eFo2KzlvCnZBUG1WcmNBbU1RZVloOG9sSzZ0SU1Gdk0wM2VBcDNNZGw3d0Fyc0c0b0xla0JVRWVtMXRPUDVzcTAwS0NYelQKQzZRNjdDRFlOYXo2N2hhbksxNGdhNld4cGU3NERJRmYrQnlYekRJVmF2dno3TXp5TDJNd3RvbkFuM1k1QkNDcgpLc00vaXgrR2FLNnB4UlFrNzB6Z3h0TGgvQi8yNmpPejVNWHMwOC9sYVhkcllJVmJNQ1FDb0pOQ0dMR3FBbU15ClRwK0FJMWhONVBQVHBPVHJ1SFpCMk5OVWNpc0ttTnNDWVA3QW9Rc05rbmVYSTZhYTFjRUhGQ1o5L1ZwQkxJVTUKUUNBMlJHb09uT1V1S25PMUZGT1RhaWViYXFMc054b05qVm9QVHhZc1FTZnE3dnpQQjM3ZnV3VGpWQXZ5TXhxSgo3SCtHbys1dXptMzFTMTd1ek8zNFRFNlVBTCtGenBLa3FlOXAxV21kZ3hhREs1dGwwczZZcG02bzdWZWo4TTNmCktWOXUrMlVINlpxbG94cjFYRWhhVjF6YVkrdG5mMjU4elBGNFd1cFBGSDlpQ3VtQURRMjk2TEJEVG40ZXZ5ZVkKaWRZYmVTU25WelZhV0ZLWU54NmFzUT09Ci0tLS0tRU5EIENFUlRJRklDQVRFLS0tLS0K" -> null

          - client_certificate_config {
              - issue_client_certificate = false -> null
            }
        }

      - monitoring_config {
          - enable_components = [
              - "SYSTEM_COMPONENTS",
            ] -> null
        }

      - network_policy {
          - enabled  = false -> null
          - provider = "PROVIDER_UNSPECIFIED" -> null
        }

      - node_config {
          - disk_size_gb      = 100 -> null
          - disk_type         = "pd-balanced" -> null
          - guest_accelerator = [] -> null
          - image_type        = "COS_CONTAINERD" -> null
          - labels            = {} -> null
          - local_ssd_count   = 0 -> null
          - logging_variant   = "DEFAULT" -> null
          - machine_type      = "e2-medium" -> null
          - metadata          = {
              - "disable-legacy-endpoints" = "true"
            } -> null
          - oauth_scopes      = [
              - "https://www.googleapis.com/auth/devstorage.read_only",
              - "https://www.googleapis.com/auth/logging.write",
              - "https://www.googleapis.com/auth/monitoring",
              - "https://www.googleapis.com/auth/service.management.readonly",
              - "https://www.googleapis.com/auth/servicecontrol",
              - "https://www.googleapis.com/auth/trace.append",
            ] -> null
          - preemptible       = false -> null
          - resource_labels   = {} -> null
          - service_account   = "default" -> null
          - spot              = false -> null
          - tags              = [] -> null
          - taint             = [] -> null

          - shielded_instance_config {
              - enable_integrity_monitoring = true -> null
              - enable_secure_boot          = false -> null
            }
        }

      - node_pool {
          - initial_node_count          = 3 -> null
          - instance_group_urls         = [
              - "https://www.googleapis.com/compute/v1/projects/qwiklabs-gcp-04-7f0fc69ae096/zones/us-central1-f/instanceGroupManagers/gke-tf-gke-k8s-default-pool-a086ea08-grp",
            ] -> null
          - managed_instance_group_urls = [
              - "https://www.googleapis.com/compute/v1/projects/qwiklabs-gcp-04-7f0fc69ae096/zones/us-central1-f/instanceGroups/gke-tf-gke-k8s-default-pool-a086ea08-grp",
            ] -> null
          - max_pods_per_node           = 0 -> null
          - name                        = "default-pool" -> null
          - node_count                  = 3 -> null
          - node_locations              = [
              - "us-central1-f",
            ] -> null
          - version                     = "1.25.7-gke.1000" -> null

          - management {
              - auto_repair  = true -> null
              - auto_upgrade = true -> null
            }

          - network_config {
              - create_pod_range     = false -> null
              - enable_private_nodes = false -> null
            }

          - node_config {
              - disk_size_gb      = 100 -> null
              - disk_type         = "pd-balanced" -> null
              - guest_accelerator = [] -> null
              - image_type        = "COS_CONTAINERD" -> null
              - labels            = {} -> null
              - local_ssd_count   = 0 -> null
              - logging_variant   = "DEFAULT" -> null
              - machine_type      = "e2-medium" -> null
              - metadata          = {
                  - "disable-legacy-endpoints" = "true"
                } -> null
              - oauth_scopes      = [
                  - "https://www.googleapis.com/auth/devstorage.read_only",
                  - "https://www.googleapis.com/auth/logging.write",
                  - "https://www.googleapis.com/auth/monitoring",
                  - "https://www.googleapis.com/auth/service.management.readonly",
                  - "https://www.googleapis.com/auth/servicecontrol",
                  - "https://www.googleapis.com/auth/trace.append",
                ] -> null
              - preemptible       = false -> null
              - resource_labels   = {} -> null
              - service_account   = "default" -> null
              - spot              = false -> null
              - tags              = [] -> null
              - taint             = [] -> null

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
          - enable_private_nodes    = false -> null
          - private_endpoint        = "10.127.0.2" -> null
          - public_endpoint         = "35.238.117.234" -> null

          - master_global_access_config {
              - enabled = false -> null
            }
        }

      - release_channel {
          - channel = "UNSPECIFIED" -> null
        }

      - service_external_ips_config {
          - enabled = false -> null
        }
    }

  # kubernetes_namespace.staging will be destroyed
  - resource "kubernetes_namespace" "staging" {
      - id = "staging" -> null

      - metadata {
          - annotations      = {} -> null
          - generation       = 0 -> null
          - labels           = {} -> null
          - name             = "staging" -> null
          - resource_version = "1440" -> null
          - uid              = "6951c10a-d20c-4441-8877-5a9b689e49e1" -> null
        }
    }

  # kubernetes_replication_controller.nginx will be destroyed
  - resource "kubernetes_replication_controller" "nginx" {
      - id = "staging/nginx" -> null

      - metadata {
          - annotations      = {} -> null
          - generation       = 1 -> null
          - labels           = {
              - "run" = "nginx"
            } -> null
          - name             = "nginx" -> null
          - namespace        = "staging" -> null
          - resource_version = "1519" -> null
          - uid              = "f43c194b-a446-4bec-a4c8-5832d5281f13" -> null
        }

      - spec {
          - min_ready_seconds = 0 -> null
          - replicas          = 1 -> null
          - selector          = {
              - "run" = "nginx"
            } -> null

          - template {
              - active_deadline_seconds          = 0 -> null
              - automount_service_account_token  = false -> null
              - host_ipc                         = false -> null
              - host_network                     = false -> null
              - host_pid                         = false -> null
              - node_selector                    = {} -> null
              - share_process_namespace          = false -> null
              - termination_grace_period_seconds = 0 -> null

              - metadata {
                  - annotations = {} -> null
                  - generation  = 0 -> null
                  - labels      = {
                      - "run" = "nginx"
                    } -> null
                  - name        = "nginx" -> null
                }

              - spec {
                  - active_deadline_seconds          = 0 -> null
                  - automount_service_account_token  = false -> null
                  - dns_policy                       = "ClusterFirst" -> null
                  - host_ipc                         = false -> null
                  - host_network                     = false -> null
                  - host_pid                         = false -> null
                  - node_selector                    = {} -> null
                  - restart_policy                   = "Always" -> null
                  - share_process_namespace          = false -> null
                  - termination_grace_period_seconds = 0 -> null

                  - container {
                      - args                     = [] -> null
                      - command                  = [] -> null
                      - image                    = "nginx:latest" -> null
                      - image_pull_policy        = "Always" -> null
                      - name                     = "nginx" -> null
                      - stdin                    = false -> null
                      - stdin_once               = false -> null
                      - termination_message_path = "/dev/termination-log" -> null
                      - tty                      = false -> null

                      - resources {
                          - limits {
                              - cpu    = "500m" -> null
                              - memory = "512Mi" -> null
                            }
                          - requests {
                              - cpu    = "250m" -> null
                              - memory = "50Mi" -> null
                            }
                        }
                    }
                }
            }
        }
    }

  # kubernetes_service.nginx will be destroyed
  - resource "kubernetes_service" "nginx" {
      - id                    = "staging/nginx" -> null
      - load_balancer_ingress = [
          - {
              - hostname = ""
              - ip       = "34.134.52.198"
            },
        ] -> null

      - metadata {
          - annotations      = {} -> null
          - generation       = 0 -> null
          - labels           = {} -> null
          - name             = "nginx" -> null
          - namespace        = "staging" -> null
          - resource_version = "1878" -> null
          - uid              = "fbab9b37-c68d-443b-ab0a-82540df21f49" -> null
        }

      - spec {
          - cluster_ip                  = "10.91.251.51" -> null
          - external_ips                = [] -> null
          - external_traffic_policy     = "Cluster" -> null
          - load_balancer_ip            = "34.134.52.198" -> null
          - load_balancer_source_ranges = [] -> null
          - publish_not_ready_addresses = false -> null
          - selector                    = {
              - "run" = "nginx"
            } -> null
          - session_affinity            = "ClientIP" -> null
          - type                        = "LoadBalancer" -> null

          - port {
              - node_port   = 31935 -> null
              - port        = 80 -> null
              - protocol    = "TCP" -> null
              - target_port = "80" -> null
            }
        }
    }

Plan: 0 to add, 0 to change, 7 to destroy.

Changes to Outputs:
  - cluster_location = "us-central1-f" -> null
  - cluster_name     = "tf-gke-k8s" -> null
  - cluster_region   = "us-central1" -> null
  - load-balancer-ip = "34.134.52.198" -> null
  - network          = "https://www.googleapis.com/compute/v1/projects/qwiklabs-gcp-04-7f0fc69ae096/global/networks/tf-gke-k8s" -> null
  - subnetwork_name  = "tf-gke-k8s" -> null
╷
│ Warning: Version constraints inside provider configuration blocks are deprecated
│
│   on k8s.tf line 2, in provider "kubernetes":
│    2:   version = "~> 1.10.0"
│
│ Terraform 0.13 and earlier allowed provider version constraints inside the provider configuration block, but that is now deprecated and
│ will be removed in a future version of Terraform. To silence this warning, move the provider version constraint into the
│ required_providers block.
╵

Do you really want to destroy all resources?
  Terraform will destroy all your managed infrastructure, as shown above.
  There is no undo. Only 'yes' will be accepted to confirm.

  Enter a value: yes

kubernetes_service.nginx: Destroying... [id=staging/nginx]
kubernetes_replication_controller.nginx: Destroying... [id=staging/nginx]
kubernetes_service.nginx: Destruction complete after 1s
google_compute_address.default: Destroying... [id=projects/qwiklabs-gcp-04-7f0fc69ae096/regions/us-central1/addresses/tf-gke-k8s]
kubernetes_replication_controller.nginx: Destruction complete after 2s
kubernetes_namespace.staging: Destroying... [id=staging]
google_compute_address.default: Destruction complete after 3s
kubernetes_namespace.staging: Still destroying... [id=staging, 10s elapsed]
kubernetes_namespace.staging: Still destroying... [id=staging, 20s elapsed]
kubernetes_namespace.staging: Still destroying... [id=staging, 30s elapsed]
kubernetes_namespace.staging: Destruction complete after 35s
google_container_cluster.default: Destroying... [id=projects/qwiklabs-gcp-04-7f0fc69ae096/locations/us-central1-f/clusters/tf-gke-k8s]
google_container_cluster.default: Provisioning with 'local-exec'...
google_container_cluster.default (local-exec): Executing: ["/bin/sh" "-c" "sleep 90"]
google_container_cluster.default: Still destroying... [id=projects/qwiklabs-gcp-04-7f0fc69ae096/l...ions/us-central1-f/clusters/tf-gke-k8s, 10s elapsed]
google_container_cluster.default: Still destroying... [id=projects/qwiklabs-gcp-04-7f0fc69ae096/l...ions/us-central1-f/clusters/tf-gke-k8s, 20s elapsed]
google_container_cluster.default: Still destroying... [id=projects/qwiklabs-gcp-04-7f0fc69ae096/l...ions/us-central1-f/clusters/tf-gke-k8s, 30s elapsed]
google_container_cluster.default: Still destroying... [id=projects/qwiklabs-gcp-04-7f0fc69ae096/l...ions/us-central1-f/clusters/tf-gke-k8s, 40s elapsed]
google_container_cluster.default: Still destroying... [id=projects/qwiklabs-gcp-04-7f0fc69ae096/l...ions/us-central1-f/clusters/tf-gke-k8s, 50s elapsed]
google_container_cluster.default: Still destroying... [id=projects/qwiklabs-gcp-04-7f0fc69ae096/l...ions/us-central1-f/clusters/tf-gke-k8s, 1m0s elapsed]
google_container_cluster.default: Still destroying... [id=projects/qwiklabs-gcp-04-7f0fc69ae096/l...ions/us-central1-f/clusters/tf-gke-k8s, 1m10s elapsed]
google_container_cluster.default: Still destroying... [id=projects/qwiklabs-gcp-04-7f0fc69ae096/l...ions/us-central1-f/clusters/tf-gke-k8s, 1m20s elapsed]
google_container_cluster.default: Still destroying... [id=projects/qwiklabs-gcp-04-7f0fc69ae096/l...ions/us-central1-f/clusters/tf-gke-k8s, 1m30s elapsed]
google_container_cluster.default: Still destroying... [id=projects/qwiklabs-gcp-04-7f0fc69ae096/l...ions/us-central1-f/clusters/tf-gke-k8s, 1m40s elapsed]
google_container_cluster.default: Still destroying... [id=projects/qwiklabs-gcp-04-7f0fc69ae096/l...ions/us-central1-f/clusters/tf-gke-k8s, 1m50s elapsed]
google_container_cluster.default: Still destroying... [id=projects/qwiklabs-gcp-04-7f0fc69ae096/l...ions/us-central1-f/clusters/tf-gke-k8s, 2m0s elapsed]
google_container_cluster.default: Still destroying... [id=projects/qwiklabs-gcp-04-7f0fc69ae096/l...ions/us-central1-f/clusters/tf-gke-k8s, 2m10s elapsed]
google_container_cluster.default: Still destroying... [id=projects/qwiklabs-gcp-04-7f0fc69ae096/l...ions/us-central1-f/clusters/tf-gke-k8s, 2m20s elapsed]
google_container_cluster.default: Still destroying... [id=projects/qwiklabs-gcp-04-7f0fc69ae096/l...ions/us-central1-f/clusters/tf-gke-k8s, 2m30s elapsed]
google_container_cluster.default: Still destroying... [id=projects/qwiklabs-gcp-04-7f0fc69ae096/l...ions/us-central1-f/clusters/tf-gke-k8s, 2m40s elapsed]
google_container_cluster.default: Still destroying... [id=projects/qwiklabs-gcp-04-7f0fc69ae096/l...ions/us-central1-f/clusters/tf-gke-k8s, 2m50s elapsed]
google_container_cluster.default: Still destroying... [id=projects/qwiklabs-gcp-04-7f0fc69ae096/l...ions/us-central1-f/clusters/tf-gke-k8s, 3m0s elapsed]
google_container_cluster.default: Still destroying... [id=projects/qwiklabs-gcp-04-7f0fc69ae096/l...ions/us-central1-f/clusters/tf-gke-k8s, 3m10s elapsed]
google_container_cluster.default: Still destroying... [id=projects/qwiklabs-gcp-04-7f0fc69ae096/l...ions/us-central1-f/clusters/tf-gke-k8s, 3m20s elapsed]
google_container_cluster.default: Still destroying... [id=projects/qwiklabs-gcp-04-7f0fc69ae096/l...ions/us-central1-f/clusters/tf-gke-k8s, 3m30s elapsed]
google_container_cluster.default: Still destroying... [id=projects/qwiklabs-gcp-04-7f0fc69ae096/l...ions/us-central1-f/clusters/tf-gke-k8s, 3m40s elapsed]
google_container_cluster.default: Still destroying... [id=projects/qwiklabs-gcp-04-7f0fc69ae096/l...ions/us-central1-f/clusters/tf-gke-k8s, 3m50s elapsed]
google_container_cluster.default: Still destroying... [id=projects/qwiklabs-gcp-04-7f0fc69ae096/l...ions/us-central1-f/clusters/tf-gke-k8s, 4m0s elapsed]
google_container_cluster.default: Still destroying... [id=projects/qwiklabs-gcp-04-7f0fc69ae096/l...ions/us-central1-f/clusters/tf-gke-k8s, 4m10s elapsed]
google_container_cluster.default: Still destroying... [id=projects/qwiklabs-gcp-04-7f0fc69ae096/l...ions/us-central1-f/clusters/tf-gke-k8s, 4m20s elapsed]
google_container_cluster.default: Still destroying... [id=projects/qwiklabs-gcp-04-7f0fc69ae096/l...ions/us-central1-f/clusters/tf-gke-k8s, 4m30s elapsed]
google_container_cluster.default: Still destroying... [id=projects/qwiklabs-gcp-04-7f0fc69ae096/l...ions/us-central1-f/clusters/tf-gke-k8s, 4m40s elapsed]
google_container_cluster.default: Still destroying... [id=projects/qwiklabs-gcp-04-7f0fc69ae096/l...ions/us-central1-f/clusters/tf-gke-k8s, 4m50s elapsed]

google_container_cluster.default: Still destroying... [id=projects/qwiklabs-gcp-04-7f0fc69ae096/l...ions/us-central1-f/clusters/tf-gke-k8s, 5m0s elapsed]
google_container_cluster.default: Still destroying... [id=projects/qwiklabs-gcp-04-7f0fc69ae096/l...ions/us-central1-f/clusters/tf-gke-k8s, 5m10s elapsed]
google_container_cluster.default: Destruction complete after 5m16s
google_compute_subnetwork.default: Destroying... [id=projects/qwiklabs-gcp-04-7f0fc69ae096/regions/us-central1/subnetworks/tf-gke-k8s]
google_compute_subnetwork.default: Still destroying... [id=projects/qwiklabs-gcp-04-7f0fc69ae096/regions/us-central1/subnetworks/tf-gke-k8s, 10s elapsed]
google_compute_subnetwork.default: Destruction complete after 14s
google_compute_network.default: Destroying... [id=projects/qwiklabs-gcp-04-7f0fc69ae096/global/networks/tf-gke-k8s]
google_compute_network.default: Still destroying... [id=projects/qwiklabs-gcp-04-7f0fc69ae096/global/networks/tf-gke-k8s, 10s elapsed]
google_compute_network.default: Still destroying... [id=projects/qwiklabs-gcp-04-7f0fc69ae096/global/networks/tf-gke-k8s, 20s elapsed]
google_compute_network.default: Destruction complete after 22s

Destroy complete! Resources: 7 destroyed.
student_01_8961e1e5d696@cloudshell:~/tf-gke-k8s-service-lb (qwiklabs-gcp-04-7f0fc69ae096)$
student_01_8961e1e5d696@cloudshell:~/tf-gke-k8s-service-lb (qwiklabs-gcp-04-7f0fc69ae096)$
```

![]({{ site.url }}{{ site.baseurl }}/assets/images/gcp-217.png)

