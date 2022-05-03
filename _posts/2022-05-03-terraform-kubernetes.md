---
layout: single
title:  "Creating Kubernetes Resources with Terraform"
date:   2022-05-03 06:56:04 +0530
categories: Automation
tags: Terraform
show_date: true
classes: wide
header:
  teaser: /assets/images/terraform.png
author:
  name     : "Terraform"
  avatar   : "/assets/images/terraform.png"

sidebar:
  - title: "Blog"
   
    text: "Checkout other topics"
    nav: my-sidebar

---

# Creating Kubernetes Resources with Terraform
According to Terraform website,
While you could use `kubectl` or similar CLI-based tools mapped to API calls to manage all Kubernetes resources described in YAML files, orchestration with Terraform presents a few benefits.

Use the same configuration language to provision the Kubernetes infrastructure and to deploy applications into it.

drift detection - `terraform plan` will always present you the difference between reality at a given time and config you intend to apply.

full lifecycle management - Terraform doesn't just initially create resources, but offers a single command for creation, update, and deletion of tracked resources without needing to inspect the API to identify those resources.

synchronous feedback - While asynchronous behaviour is often useful, sometimes it's counter-productive as the job of identifying operation result (failures or details of created resource) is left to the user. e.g. you don't have IP/hostname of load balancer until it has finished provisioning, hence you can't create any DNS record pointing to it.

graph of relationships - Terraform understands relationships between resources which may help in scheduling - e.g. if a Persistent Volume Claim claims space from a particular Persistent Volume Terraform won't even attempt to create the PVC if creation of the PV has failed.

We'll use minikube for the Kubernetes cluster in this example, but any Kubernetes cluster can be used. 

This configuration will create a scalable Nginx Deployment with 2 replicas. It will expose the Nginx frontend using a Service of type NodePort, which will make Nginx accessible via the public IP of the node running the containers.

```sh
pradeep:~$cat k8s.tf 
terraform {
  required_providers {
    kubernetes = {
      source  = "hashicorp/kubernetes"
      version = ">= 2.0.0"
    }
  }
}
provider "kubernetes" {
  config_path = "~/.kube/config"
}
resource "kubernetes_namespace" "test" {
  metadata {
    name = "nginx"
  }
}
resource "kubernetes_deployment" "test" {
  metadata {
    name      = "nginx"
    namespace = kubernetes_namespace.test.metadata.0.name
  }
  spec {
    replicas = 2
    selector {
      match_labels = {
        app = "MyTestApp"
      }
    }
    template {
      metadata {
        labels = {
          app = "MyTestApp"
        }
      }
      spec {
        container {
          image = "nginx"
          name  = "nginx-container"
          port {
            container_port = 80
          }
        }
      }
    }
  }
}
resource "kubernetes_service" "test" {
  metadata {
    name      = "nginx"
    namespace = kubernetes_namespace.test.metadata.0.name
  }
  spec {
    selector = {
      app = kubernetes_deployment.test.spec.0.template.0.metadata.0.labels.app
    }
    type = "NodePort"
    port {
      node_port   = 30201
      port        = 80
      target_port = 80
    }
  }
}
pradeep:~$
```

```sh
pradeep:~$terraform init

Initializing the backend...

Initializing provider plugins...
- Finding hashicorp/kubernetes versions matching ">= 2.0.0"...
- Installing hashicorp/kubernetes v2.11.0...
- Installed hashicorp/kubernetes v2.11.0 (signed by HashiCorp)

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
pradeep:~$
```



```sh
pradeep:~$terraform plan

Terraform used the selected providers to generate the following execution plan. Resource actions are indicated with the following symbols:
  + create

Terraform will perform the following actions:

  # kubernetes_deployment.test will be created
  + resource "kubernetes_deployment" "test" {
      + id               = (known after apply)
      + wait_for_rollout = true

      + metadata {
          + generation       = (known after apply)
          + name             = "nginx"
          + namespace        = "nginx"
          + resource_version = (known after apply)
          + uid              = (known after apply)
        }

      + spec {
          + min_ready_seconds         = 0
          + paused                    = false
          + progress_deadline_seconds = 600
          + replicas                  = "2"
          + revision_history_limit    = 10

          + selector {
              + match_labels = {
                  + "app" = "MyTestApp"
                }
            }

          + strategy {
              + type = (known after apply)

              + rolling_update {
                  + max_surge       = (known after apply)
                  + max_unavailable = (known after apply)
                }
            }

          + template {
              + metadata {
                  + generation       = (known after apply)
                  + labels           = {
                      + "app" = "MyTestApp"
                    }
                  + name             = (known after apply)
                  + resource_version = (known after apply)
                  + uid              = (known after apply)
                }

              + spec {
                  + automount_service_account_token  = true
                  + dns_policy                       = "ClusterFirst"
                  + enable_service_links             = true
                  + host_ipc                         = false
                  + host_network                     = false
                  + host_pid                         = false
                  + hostname                         = (known after apply)
                  + node_name                        = (known after apply)
                  + restart_policy                   = "Always"
                  + service_account_name             = (known after apply)
                  + share_process_namespace          = false
                  + termination_grace_period_seconds = 30

                  + container {
                      + image                      = "nginx"
                      + image_pull_policy          = (known after apply)
                      + name                       = "nginx-container"
                      + stdin                      = false
                      + stdin_once                 = false
                      + termination_message_path   = "/dev/termination-log"
                      + termination_message_policy = (known after apply)
                      + tty                        = false

                      + port {
                          + container_port = 80
                          + protocol       = "TCP"
                        }

                      + resources {
                          + limits   = (known after apply)
                          + requests = (known after apply)
                        }
                    }

                  + image_pull_secrets {
                      + name = (known after apply)
                    }

                  + readiness_gate {
                      + condition_type = (known after apply)
                    }

                  + volume {
                      + name = (known after apply)

                      + aws_elastic_block_store {
                          + fs_type   = (known after apply)
                          + partition = (known after apply)
                          + read_only = (known after apply)
                          + volume_id = (known after apply)
                        }

                      + azure_disk {
                          + caching_mode  = (known after apply)
                          + data_disk_uri = (known after apply)
                          + disk_name     = (known after apply)
                          + fs_type       = (known after apply)
                          + kind          = (known after apply)
                          + read_only     = (known after apply)
                        }

                      + azure_file {
                          + read_only        = (known after apply)
                          + secret_name      = (known after apply)
                          + secret_namespace = (known after apply)
                          + share_name       = (known after apply)
                        }

                      + ceph_fs {
                          + monitors    = (known after apply)
                          + path        = (known after apply)
                          + read_only   = (known after apply)
                          + secret_file = (known after apply)
                          + user        = (known after apply)

                          + secret_ref {
                              + name      = (known after apply)
                              + namespace = (known after apply)
                            }
                        }

                      + cinder {
                          + fs_type   = (known after apply)
                          + read_only = (known after apply)
                          + volume_id = (known after apply)
                        }

                      + config_map {
                          + default_mode = (known after apply)
                          + name         = (known after apply)
                          + optional     = (known after apply)

                          + items {
                              + key  = (known after apply)
                              + mode = (known after apply)
                              + path = (known after apply)
                            }
                        }

                      + csi {
                          + driver            = (known after apply)
                          + fs_type           = (known after apply)
                          + read_only         = (known after apply)
                          + volume_attributes = (known after apply)

                          + node_publish_secret_ref {
                              + name = (known after apply)
                            }
                        }

                      + downward_api {
                          + default_mode = (known after apply)

                          + items {
                              + mode = (known after apply)
                              + path = (known after apply)

                              + field_ref {
                                  + api_version = (known after apply)
                                  + field_path  = (known after apply)
                                }

                              + resource_field_ref {
                                  + container_name = (known after apply)
                                  + divisor        = (known after apply)
                                  + resource       = (known after apply)
                                }
                            }
                        }

                      + empty_dir {
                          + medium     = (known after apply)
                          + size_limit = (known after apply)
                        }

                      + fc {
                          + fs_type      = (known after apply)
                          + lun          = (known after apply)
                          + read_only    = (known after apply)
                          + target_ww_ns = (known after apply)
                        }

                      + flex_volume {
                          + driver    = (known after apply)
                          + fs_type   = (known after apply)
                          + options   = (known after apply)
                          + read_only = (known after apply)

                          + secret_ref {
                              + name      = (known after apply)
                              + namespace = (known after apply)
                            }
                        }

                      + flocker {
                          + dataset_name = (known after apply)
                          + dataset_uuid = (known after apply)
                        }

                      + gce_persistent_disk {
                          + fs_type   = (known after apply)
                          + partition = (known after apply)
                          + pd_name   = (known after apply)
                          + read_only = (known after apply)
                        }

                      + git_repo {
                          + directory  = (known after apply)
                          + repository = (known after apply)
                          + revision   = (known after apply)
                        }

                      + glusterfs {
                          + endpoints_name = (known after apply)
                          + path           = (known after apply)
                          + read_only      = (known after apply)
                        }

                      + host_path {
                          + path = (known after apply)
                          + type = (known after apply)
                        }

                      + iscsi {
                          + fs_type         = (known after apply)
                          + iqn             = (known after apply)
                          + iscsi_interface = (known after apply)
                          + lun             = (known after apply)
                          + read_only       = (known after apply)
                          + target_portal   = (known after apply)
                        }

                      + local {
                          + path = (known after apply)
                        }

                      + nfs {
                          + path      = (known after apply)
                          + read_only = (known after apply)
                          + server    = (known after apply)
                        }

                      + persistent_volume_claim {
                          + claim_name = (known after apply)
                          + read_only  = (known after apply)
                        }

                      + photon_persistent_disk {
                          + fs_type = (known after apply)
                          + pd_id   = (known after apply)
                        }

                      + projected {
                          + default_mode = (known after apply)

                          + sources {
                              + config_map {
                                  + name     = (known after apply)
                                  + optional = (known after apply)

                                  + items {
                                      + key  = (known after apply)
                                      + mode = (known after apply)
                                      + path = (known after apply)
                                    }
                                }

                              + downward_api {
                                  + items {
                                      + mode = (known after apply)
                                      + path = (known after apply)

                                      + field_ref {
                                          + api_version = (known after apply)
                                          + field_path  = (known after apply)
                                        }

                                      + resource_field_ref {
                                          + container_name = (known after apply)
                                          + divisor        = (known after apply)
                                          + resource       = (known after apply)
                                        }
                                    }
                                }

                              + secret {
                                  + name     = (known after apply)
                                  + optional = (known after apply)

                                  + items {
                                      + key  = (known after apply)
                                      + mode = (known after apply)
                                      + path = (known after apply)
                                    }
                                }

                              + service_account_token {
                                  + audience           = (known after apply)
                                  + expiration_seconds = (known after apply)
                                  + path               = (known after apply)
                                }
                            }
                        }

                      + quobyte {
                          + group     = (known after apply)
                          + read_only = (known after apply)
                          + registry  = (known after apply)
                          + user      = (known after apply)
                          + volume    = (known after apply)
                        }

                      + rbd {
                          + ceph_monitors = (known after apply)
                          + fs_type       = (known after apply)
                          + keyring       = (known after apply)
                          + rados_user    = (known after apply)
                          + rbd_image     = (known after apply)
                          + rbd_pool      = (known after apply)
                          + read_only     = (known after apply)

                          + secret_ref {
                              + name      = (known after apply)
                              + namespace = (known after apply)
                            }
                        }

                      + secret {
                          + default_mode = (known after apply)
                          + optional     = (known after apply)
                          + secret_name  = (known after apply)

                          + items {
                              + key  = (known after apply)
                              + mode = (known after apply)
                              + path = (known after apply)
                            }
                        }

                      + vsphere_volume {
                          + fs_type     = (known after apply)
                          + volume_path = (known after apply)
                        }
                    }
                }
            }
        }
    }

  # kubernetes_namespace.test will be created
  + resource "kubernetes_namespace" "test" {
      + id = (known after apply)

      + metadata {
          + generation       = (known after apply)
          + name             = "nginx"
          + resource_version = (known after apply)
          + uid              = (known after apply)
        }
    }

  # kubernetes_service.test will be created
  + resource "kubernetes_service" "test" {
      + id                     = (known after apply)
      + status                 = (known after apply)
      + wait_for_load_balancer = true

      + metadata {
          + generation       = (known after apply)
          + name             = "nginx"
          + namespace        = "nginx"
          + resource_version = (known after apply)
          + uid              = (known after apply)
        }

      + spec {
          + cluster_ip                  = (known after apply)
          + external_traffic_policy     = (known after apply)
          + health_check_node_port      = (known after apply)
          + ip_families                 = (known after apply)
          + ip_family_policy            = (known after apply)
          + publish_not_ready_addresses = false
          + selector                    = {
              + "app" = "MyTestApp"
            }
          + session_affinity            = "None"
          + type                        = "NodePort"

          + port {
              + node_port   = 30201
              + port        = 80
              + protocol    = "TCP"
              + target_port = "80"
            }
        }
    }

Plan: 3 to add, 0 to change, 0 to destroy.

─────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────

Note: You didn't use the -out option to save this plan, so Terraform can't guarantee to take exactly these actions if you run "terraform apply" now.
pradeep:~$

```

```sh
pradeep:~$terraform apply --auto-approve

Terraform used the selected providers to generate the following execution plan. Resource actions are indicated with the following symbols:
  + create

Terraform will perform the following actions:

  # kubernetes_deployment.test will be created
  + resource "kubernetes_deployment" "test" {
      + id               = (known after apply)
      + wait_for_rollout = true

      + metadata {
          + generation       = (known after apply)
          + name             = "nginx"
          + namespace        = "nginx"
          + resource_version = (known after apply)
          + uid              = (known after apply)
        }

      + spec {
          + min_ready_seconds         = 0
          + paused                    = false
          + progress_deadline_seconds = 600
          + replicas                  = "2"
          + revision_history_limit    = 10

          + selector {
              + match_labels = {
                  + "app" = "MyTestApp"
                }
            }

          + strategy {
              + type = (known after apply)

              + rolling_update {
                  + max_surge       = (known after apply)
                  + max_unavailable = (known after apply)
                }
            }

          + template {
              + metadata {
                  + generation       = (known after apply)
                  + labels           = {
                      + "app" = "MyTestApp"
                    }
                  + name             = (known after apply)
                  + resource_version = (known after apply)
                  + uid              = (known after apply)
                }

              + spec {
                  + automount_service_account_token  = true
                  + dns_policy                       = "ClusterFirst"
                  + enable_service_links             = true
                  + host_ipc                         = false
                  + host_network                     = false
                  + host_pid                         = false
                  + hostname                         = (known after apply)
                  + node_name                        = (known after apply)
                  + restart_policy                   = "Always"
                  + service_account_name             = (known after apply)
                  + share_process_namespace          = false
                  + termination_grace_period_seconds = 30

                  + container {
                      + image                      = "nginx"
                      + image_pull_policy          = (known after apply)
                      + name                       = "nginx-container"
                      + stdin                      = false
                      + stdin_once                 = false
                      + termination_message_path   = "/dev/termination-log"
                      + termination_message_policy = (known after apply)
                      + tty                        = false

                      + port {
                          + container_port = 80
                          + protocol       = "TCP"
                        }

                      + resources {
                          + limits   = (known after apply)
                          + requests = (known after apply)
                        }
                    }

                  + image_pull_secrets {
                      + name = (known after apply)
                    }

                  + readiness_gate {
                      + condition_type = (known after apply)
                    }

                  + volume {
                      + name = (known after apply)

                      + aws_elastic_block_store {
                          + fs_type   = (known after apply)
                          + partition = (known after apply)
                          + read_only = (known after apply)
                          + volume_id = (known after apply)
                        }

                      + azure_disk {
                          + caching_mode  = (known after apply)
                          + data_disk_uri = (known after apply)
                          + disk_name     = (known after apply)
                          + fs_type       = (known after apply)
                          + kind          = (known after apply)
                          + read_only     = (known after apply)
                        }

                      + azure_file {
                          + read_only        = (known after apply)
                          + secret_name      = (known after apply)
                          + secret_namespace = (known after apply)
                          + share_name       = (known after apply)
                        }

                      + ceph_fs {
                          + monitors    = (known after apply)
                          + path        = (known after apply)
                          + read_only   = (known after apply)
                          + secret_file = (known after apply)
                          + user        = (known after apply)

                          + secret_ref {
                              + name      = (known after apply)
                              + namespace = (known after apply)
                            }
                        }

                      + cinder {
                          + fs_type   = (known after apply)
                          + read_only = (known after apply)
                          + volume_id = (known after apply)
                        }

                      + config_map {
                          + default_mode = (known after apply)
                          + name         = (known after apply)
                          + optional     = (known after apply)

                          + items {
                              + key  = (known after apply)
                              + mode = (known after apply)
                              + path = (known after apply)
                            }
                        }

                      + csi {
                          + driver            = (known after apply)
                          + fs_type           = (known after apply)
                          + read_only         = (known after apply)
                          + volume_attributes = (known after apply)

                          + node_publish_secret_ref {
                              + name = (known after apply)
                            }
                        }

                      + downward_api {
                          + default_mode = (known after apply)

                          + items {
                              + mode = (known after apply)
                              + path = (known after apply)

                              + field_ref {
                                  + api_version = (known after apply)
                                  + field_path  = (known after apply)
                                }

                              + resource_field_ref {
                                  + container_name = (known after apply)
                                  + divisor        = (known after apply)
                                  + resource       = (known after apply)
                                }
                            }
                        }

                      + empty_dir {
                          + medium     = (known after apply)
                          + size_limit = (known after apply)
                        }

                      + fc {
                          + fs_type      = (known after apply)
                          + lun          = (known after apply)
                          + read_only    = (known after apply)
                          + target_ww_ns = (known after apply)
                        }

                      + flex_volume {
                          + driver    = (known after apply)
                          + fs_type   = (known after apply)
                          + options   = (known after apply)
                          + read_only = (known after apply)

                          + secret_ref {
                              + name      = (known after apply)
                              + namespace = (known after apply)
                            }
                        }

                      + flocker {
                          + dataset_name = (known after apply)
                          + dataset_uuid = (known after apply)
                        }

                      + gce_persistent_disk {
                          + fs_type   = (known after apply)
                          + partition = (known after apply)
                          + pd_name   = (known after apply)
                          + read_only = (known after apply)
                        }

                      + git_repo {
                          + directory  = (known after apply)
                          + repository = (known after apply)
                          + revision   = (known after apply)
                        }

                      + glusterfs {
                          + endpoints_name = (known after apply)
                          + path           = (known after apply)
                          + read_only      = (known after apply)
                        }

                      + host_path {
                          + path = (known after apply)
                          + type = (known after apply)
                        }

                      + iscsi {
                          + fs_type         = (known after apply)
                          + iqn             = (known after apply)
                          + iscsi_interface = (known after apply)
                          + lun             = (known after apply)
                          + read_only       = (known after apply)
                          + target_portal   = (known after apply)
                        }

                      + local {
                          + path = (known after apply)
                        }

                      + nfs {
                          + path      = (known after apply)
                          + read_only = (known after apply)
                          + server    = (known after apply)
                        }

                      + persistent_volume_claim {
                          + claim_name = (known after apply)
                          + read_only  = (known after apply)
                        }

                      + photon_persistent_disk {
                          + fs_type = (known after apply)
                          + pd_id   = (known after apply)
                        }

                      + projected {
                          + default_mode = (known after apply)

                          + sources {
                              + config_map {
                                  + name     = (known after apply)
                                  + optional = (known after apply)

                                  + items {
                                      + key  = (known after apply)
                                      + mode = (known after apply)
                                      + path = (known after apply)
                                    }
                                }

                              + downward_api {
                                  + items {
                                      + mode = (known after apply)
                                      + path = (known after apply)

                                      + field_ref {
                                          + api_version = (known after apply)
                                          + field_path  = (known after apply)
                                        }

                                      + resource_field_ref {
                                          + container_name = (known after apply)
                                          + divisor        = (known after apply)
                                          + resource       = (known after apply)
                                        }
                                    }
                                }

                              + secret {
                                  + name     = (known after apply)
                                  + optional = (known after apply)

                                  + items {
                                      + key  = (known after apply)
                                      + mode = (known after apply)
                                      + path = (known after apply)
                                    }
                                }

                              + service_account_token {
                                  + audience           = (known after apply)
                                  + expiration_seconds = (known after apply)
                                  + path               = (known after apply)
                                }
                            }
                        }

                      + quobyte {
                          + group     = (known after apply)
                          + read_only = (known after apply)
                          + registry  = (known after apply)
                          + user      = (known after apply)
                          + volume    = (known after apply)
                        }

                      + rbd {
                          + ceph_monitors = (known after apply)
                          + fs_type       = (known after apply)
                          + keyring       = (known after apply)
                          + rados_user    = (known after apply)
                          + rbd_image     = (known after apply)
                          + rbd_pool      = (known after apply)
                          + read_only     = (known after apply)

                          + secret_ref {
                              + name      = (known after apply)
                              + namespace = (known after apply)
                            }
                        }

                      + secret {
                          + default_mode = (known after apply)
                          + optional     = (known after apply)
                          + secret_name  = (known after apply)

                          + items {
                              + key  = (known after apply)
                              + mode = (known after apply)
                              + path = (known after apply)
                            }
                        }

                      + vsphere_volume {
                          + fs_type     = (known after apply)
                          + volume_path = (known after apply)
                        }
                    }
                }
            }
        }
    }

  # kubernetes_namespace.test will be created
  + resource "kubernetes_namespace" "test" {
      + id = (known after apply)

      + metadata {
          + generation       = (known after apply)
          + name             = "nginx"
          + resource_version = (known after apply)
          + uid              = (known after apply)
        }
    }

  # kubernetes_service.test will be created
  + resource "kubernetes_service" "test" {
      + id                     = (known after apply)
      + status                 = (known after apply)
      + wait_for_load_balancer = true

      + metadata {
          + generation       = (known after apply)
          + name             = "nginx"
          + namespace        = "nginx"
          + resource_version = (known after apply)
          + uid              = (known after apply)
        }

      + spec {
          + cluster_ip                  = (known after apply)
          + external_traffic_policy     = (known after apply)
          + health_check_node_port      = (known after apply)
          + ip_families                 = (known after apply)
          + ip_family_policy            = (known after apply)
          + publish_not_ready_addresses = false
          + selector                    = {
              + "app" = "MyTestApp"
            }
          + session_affinity            = "None"
          + type                        = "NodePort"

          + port {
              + node_port   = 30201
              + port        = 80
              + protocol    = "TCP"
              + target_port = "80"
            }
        }
    }

Plan: 3 to add, 0 to change, 0 to destroy.
kubernetes_namespace.test: Creating...
kubernetes_namespace.test: Creation complete after 0s [id=nginx]
kubernetes_deployment.test: Creating...
kubernetes_deployment.test: Still creating... [10s elapsed]
kubernetes_deployment.test: Creation complete after 15s [id=nginx/nginx]
kubernetes_service.test: Creating...
kubernetes_service.test: Creation complete after 0s [id=nginx/nginx]

Apply complete! Resources: 3 added, 0 changed, 0 destroyed.
pradeep:~$
```





```sh
pradeep:~$kubectl get all -n nginx
NAME                        READY   STATUS    RESTARTS   AGE
pod/nginx-d6c6668d8-bt6jm   1/1     Running   0          62s
pod/nginx-d6c6668d8-rfstp   1/1     Running   0          62s

NAME            TYPE       CLUSTER-IP       EXTERNAL-IP   PORT(S)        AGE
service/nginx   NodePort   10.107.195.237   <none>        80:30201/TCP   46s

NAME                    READY   UP-TO-DATE   AVAILABLE   AGE
deployment.apps/nginx   2/2     2            2           62s

NAME                              DESIRED   CURRENT   READY   AGE
replicaset.apps/nginx-d6c6668d8   2         2         2       62s
pradeep:~$

```

```sh
pradeep:~$kubectl get pods -o wide
NAME                     READY   STATUS    RESTARTS        AGE   IP              NODE       NOMINATED NODE   READINESS GATES
nginx-85b98978db-t44nc   1/1     Running   1 (3m19s ago)   9d    10.244.120.74   minikube   <none>           <none>
pradeep:~$kubectl get nodes -o wide
NAME       STATUS   ROLES                  AGE   VERSION   INTERNAL-IP    EXTERNAL-IP   OS-IMAGE             KERNEL-VERSION      CONTAINER-RUNTIME
minikube   Ready    control-plane,master   9d    v1.23.3   192.168.49.2   <none>        Ubuntu 20.04.2 LTS   5.10.104-linuxkit   docker://20.10.12
```

```sh
pradeep:~$minikube ssh
docker@minikube:~$ curl 10.244.120.74
<!DOCTYPE html>
<html>
<head>
<title>Welcome to nginx!</title>
<style>
html { color-scheme: light dark; }
body { width: 35em; margin: 0 auto;
font-family: Tahoma, Verdana, Arial, sans-serif; }
</style>
</head>
<body>
<h1>Welcome to nginx!</h1>
<p>If you see this page, the nginx web server is successfully installed and
working. Further configuration is required.</p>

<p>For online documentation and support please refer to
<a href="http://nginx.org/">nginx.org</a>.<br/>
Commercial support is available at
<a href="http://nginx.com/">nginx.com</a>.</p>

<p><em>Thank you for using nginx.</em></p>
</body>
</html>
```

```sh

docker@minikube:~$ curl localhost:30201
<!DOCTYPE html>
<html>
<head>
<title>Welcome to nginx!</title>
<style>
html { color-scheme: light dark; }
body { width: 35em; margin: 0 auto;
font-family: Tahoma, Verdana, Arial, sans-serif; }
</style>
</head>
<body>
<h1>Welcome to nginx!</h1>
<p>If you see this page, the nginx web server is successfully installed and
working. Further configuration is required.</p>

<p>For online documentation and support please refer to
<a href="http://nginx.org/">nginx.org</a>.<br/>
Commercial support is available at
<a href="http://nginx.com/">nginx.com</a>.</p>

<p><em>Thank you for using nginx.</em></p>
</body>
</html>
docker@minikube:~$ 

```



```sh
pradeep:~$terraform destroy --auto-approve
kubernetes_namespace.test: Refreshing state... [id=nginx]
kubernetes_deployment.test: Refreshing state... [id=nginx/nginx]
kubernetes_service.test: Refreshing state... [id=nginx/nginx]

Terraform used the selected providers to generate the following execution plan. Resource actions are indicated with the following symbols:
  - destroy

Terraform will perform the following actions:

  # kubernetes_deployment.test will be destroyed
  - resource "kubernetes_deployment" "test" {
      - id               = "nginx/nginx" -> null
      - wait_for_rollout = true -> null

      - metadata {
          - annotations      = {} -> null
          - generation       = 1 -> null
          - labels           = {} -> null
          - name             = "nginx" -> null
          - namespace        = "nginx" -> null
          - resource_version = "2410" -> null
          - uid              = "2e115e67-c8e7-436d-8c62-48350a2a7fe4" -> null
        }

      - spec {
          - min_ready_seconds         = 0 -> null
          - paused                    = false -> null
          - progress_deadline_seconds = 600 -> null
          - replicas                  = "2" -> null
          - revision_history_limit    = 10 -> null

          - selector {
              - match_labels = {
                  - "app" = "MyTestApp"
                } -> null
            }

          - strategy {
              - type = "RollingUpdate" -> null

              - rolling_update {
                  - max_surge       = "25%" -> null
                  - max_unavailable = "25%" -> null
                }
            }

          - template {
              - metadata {
                  - annotations = {} -> null
                  - generation  = 0 -> null
                  - labels      = {
                      - "app" = "MyTestApp"
                    } -> null
                }

              - spec {
                  - active_deadline_seconds          = 0 -> null
                  - automount_service_account_token  = true -> null
                  - dns_policy                       = "ClusterFirst" -> null
                  - enable_service_links             = true -> null
                  - host_ipc                         = false -> null
                  - host_network                     = false -> null
                  - host_pid                         = false -> null
                  - node_selector                    = {} -> null
                  - restart_policy                   = "Always" -> null
                  - share_process_namespace          = false -> null
                  - termination_grace_period_seconds = 30 -> null

                  - container {
                      - args                       = [] -> null
                      - command                    = [] -> null
                      - image                      = "nginx" -> null
                      - image_pull_policy          = "Always" -> null
                      - name                       = "nginx-container" -> null
                      - stdin                      = false -> null
                      - stdin_once                 = false -> null
                      - termination_message_path   = "/dev/termination-log" -> null
                      - termination_message_policy = "File" -> null
                      - tty                        = false -> null

                      - port {
                          - container_port = 80 -> null
                          - host_port      = 0 -> null
                          - protocol       = "TCP" -> null
                        }

                      - resources {}
                    }
                }
            }
        }
    }

  # kubernetes_namespace.test will be destroyed
  - resource "kubernetes_namespace" "test" {
      - id = "nginx" -> null

      - metadata {
          - annotations      = {} -> null
          - generation       = 0 -> null
          - labels           = {} -> null
          - name             = "nginx" -> null
          - resource_version = "2364" -> null
          - uid              = "649e4d4b-0cf7-4392-8b96-dfd9b87d2515" -> null
        }
    }

  # kubernetes_service.test will be destroyed
  - resource "kubernetes_service" "test" {
      - id                     = "nginx/nginx" -> null
      - status                 = [
          - {
              - load_balancer = [
                  - {
                      - ingress = []
                    },
                ]
            },
        ] -> null
      - wait_for_load_balancer = true -> null

      - metadata {
          - annotations      = {} -> null
          - generation       = 0 -> null
          - labels           = {} -> null
          - name             = "nginx" -> null
          - namespace        = "nginx" -> null
          - resource_version = "2426" -> null
          - uid              = "50938d65-185f-4a89-bd95-d237f92cd6af" -> null
        }

      - spec {
          - cluster_ip                  = "10.107.195.237" -> null
          - external_ips                = [] -> null
          - external_traffic_policy     = "Cluster" -> null
          - health_check_node_port      = 0 -> null
          - ip_families                 = [
              - "IPv4",
            ] -> null
          - ip_family_policy            = "SingleStack" -> null
          - load_balancer_source_ranges = [] -> null
          - publish_not_ready_addresses = false -> null
          - selector                    = {
              - "app" = "MyTestApp"
            } -> null
          - session_affinity            = "None" -> null
          - type                        = "NodePort" -> null

          - port {
              - node_port   = 30201 -> null
              - port        = 80 -> null
              - protocol    = "TCP" -> null
              - target_port = "80" -> null
            }
        }
    }

Plan: 0 to add, 0 to change, 3 to destroy.
kubernetes_service.test: Destroying... [id=nginx/nginx]
kubernetes_service.test: Destruction complete after 0s
kubernetes_deployment.test: Destroying... [id=nginx/nginx]
kubernetes_deployment.test: Destruction complete after 0s
kubernetes_namespace.test: Destroying... [id=nginx]
kubernetes_namespace.test: Destruction complete after 6s

Destroy complete! Resources: 3 destroyed.
pradeep:~$

```



```sh
pradeep:~$kubectl get all -n nginx
No resources found in nginx namespace.
pradeep:~$

```

This concludes our post on using Terraform to deploy Kubernetes resources.

