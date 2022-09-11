---
layout: single
title:  "Terraform Docker Example"
date:   2022-09-11 08:58:04 +0530
categories: Automation
tags: Terraform
show_date: true
toc: true
toc_sticky: true
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

In this post, let us deploy a Docker container using Terraform. This example also makes use of some of the Terraform features like Input variables and Output variables.

Let us create a new Terraform configuration for creating Docker containers.
```sh
(base) pradeep:~$pwd
/Users/pradeep/docker-terraform
(base) pradeep:~$ls -la
total 24
drwxr-xr-x    5 pradeep  staff   160 Sep 11 14:27 .
drwxr-xr-x+ 110 pradeep  staff  3520 Sep 11 14:27 ..
-rw-r--r--    1 pradeep  staff   412 Sep 11 14:26 main.tf
-rw-r--r--    1 pradeep  staff   217 Sep 11 14:27 outputs.tf
-rw-r--r--    1 pradeep  staff   154 Sep 11 14:27 variables.tf
(base) pradeep:~$
```

```sh
(base) pradeep:~$cat main.tf 
# main.tf

terraform {
  required_providers {
    docker = {
      source  = "kreuzwerker/docker"
      version = "~> 2.13.0"
    }
  }
}

provider "docker" {}

resource "docker_image" "nginx" {
  name         = "nginx:latest"
  keep_locally = false
}

resource "docker_container" "nginx" {
  image = docker_image.nginx.latest
  name  = var.container_name
  ports {
    internal = 80
    external = 8080
  }
}


(base) pradeep:~$
```

```sh
(base) pradeep:~$cat variables.tf 
variable "container_name" {
  description = "Value of the name for the Docker container"
  type        = string
  default     = "ExampleNginxContainer"
}
(base) pradeep:~$
```

```sh
(base) pradeep:~$cat outputs.tf 
output "container_id" {
  description = "ID of the Docker container"
  value       = docker_container.nginx.id
}

output "image_id" {
  description = "ID of the Docker image"
  value       = docker_image.nginx.id
}


(base) pradeep:~$
(base) pradeep:~$
```
## Terraform Init
```sh
(base) pradeep:~$terraform init

Initializing the backend...

Initializing provider plugins...
- Finding kreuzwerker/docker versions matching "~> 2.13.0"...
- Installing kreuzwerker/docker v2.13.0...
- Installed kreuzwerker/docker v2.13.0 (self-signed, key ID 24E54F214569A8A5)

Partner and community providers are signed by their developers.
If you'd like to know more about provider signing, you can read about it here:
https://www.terraform.io/docs/cli/plugins/signing.html

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
(base) pradeep:~$
```

```sh
(base) pradeep:~$ls -la
total 32
drwxr-xr-x    7 pradeep  staff   224 Sep 11 14:32 .
drwxr-xr-x+ 110 pradeep  staff  3520 Sep 11 14:27 ..
drwxr-xr-x    3 pradeep  staff    96 Sep 11 14:32 .terraform
-rw-r--r--    1 pradeep  staff  1264 Sep 11 14:32 .terraform.lock.hcl
-rw-r--r--    1 pradeep  staff   412 Sep 11 14:26 main.tf
-rw-r--r--    1 pradeep  staff   217 Sep 11 14:27 outputs.tf
-rw-r--r--    1 pradeep  staff   154 Sep 11 14:27 variables.tf
(base) pradeep:~$
```

Lets take a look at the hidden `.terraform` directory 

```sh
(base) pradeep:~$ls -la .terraform
total 0
drwxr-xr-x  3 pradeep  staff   96 Sep 11 14:32 .
drwxr-xr-x  7 pradeep  staff  224 Sep 11 14:32 ..
drwxr-xr-x  3 pradeep  staff   96 Sep 11 14:32 providers
(base) pradeep:~$ls -la .terraform/providers/registry.terraform.io/kreuzwerker/docker/2.13.0/darwin_amd64/README.md 
-rw-r--r--  1 pradeep  staff  1761 Sep 11 14:32 .terraform/providers/registry.terraform.io/kreuzwerker/docker/2.13.0/darwin_amd64/README.md
(base) pradeep:~$cat .terraform/providers/registry.terraform.io/kreuzwerker/docker/2.13.0/darwin_amd64/README.md
# Terraform Provider

- Website: https://www.terraform.io
- Provider: [kreuzwerker/docker](https://registry.terraform.io/providers/kreuzwerker/docker/latest)
- Slack: [@gophers/terraform-provider-docker](https://gophers.slack.com/archives/C01G9TN5V36)


## Requirements
-	[Terraform](https://www.terraform.io/downloads.html) >=0.12.x
-	[Go](https://golang.org/doc/install) 1.15.x (to build the provider plugin)

## Building The Provider

```sh
$ git clone git@github.com:kreuzwerker/terraform-provider-docker
$ make build
```

## Example usage
```hcl
# Set the required provider and versions
terraform {
  required_providers {
    # We recommend pinning to the specific version of the Docker Provider you're using
    # since new versions are released frequently
    docker = {
      source  = "kreuzwerker/docker"
      version = "2.13.0"
    }
  }
}

# Configure the docker provider
provider "docker" {
}

# Create a docker image resource
# -> docker pull nginx:latest
resource "docker_image" "nginx" {
  name         = "nginx:latest"
  keep_locally = true
}

# Create a docker container resource
# -> same as 'docker run --name nginx -p8080:80 -d nginx:latest'
resource "docker_container" "nginx" {
  name    = "nginx"
  image   = docker_image.nginx.latest

  ports {
    external = 8080
    internal = 80
  }
}

# Or create a service resource
# -> same as 'docker service create -d -p 8081:80 --name nginx-service --replicas 2 nginx:latest'
resource "docker_service" "nginx_service" {
  name = "nginx-service"
  task_spec {
    container_spec {
      image = docker_image.nginx.repo_digest
    }
  }

  mode {
    replicated {
      replicas = 2
    }
  }

  endpoint_spec {
    ports {
      published_port = 8081
      target_port    = 80
    }
  }
}
```

## Terraform Plan
```sh
(base) pradeep:~$terraform plan

Terraform used the selected providers to generate the following execution plan. Resource actions are indicated with the following symbols:
  + create

Terraform will perform the following actions:

  # docker_container.nginx will be created
  + resource "docker_container" "nginx" {
      + attach           = false
      + bridge           = (known after apply)
      + command          = (known after apply)
      + container_logs   = (known after apply)
      + entrypoint       = (known after apply)
      + env              = (known after apply)
      + exit_code        = (known after apply)
      + gateway          = (known after apply)
      + hostname         = (known after apply)
      + id               = (known after apply)
      + image            = (known after apply)
      + init             = (known after apply)
      + ip_address       = (known after apply)
      + ip_prefix_length = (known after apply)
      + ipc_mode         = (known after apply)
      + log_driver       = "json-file"
      + logs             = false
      + must_run         = true
      + name             = "ExampleNginxContainer"
      + network_data     = (known after apply)
      + read_only        = false
      + remove_volumes   = true
      + restart          = "no"
      + rm               = false
      + security_opts    = (known after apply)
      + shm_size         = (known after apply)
      + start            = true
      + stdin_open       = false
      + tty              = false

      + healthcheck {
          + interval     = (known after apply)
          + retries      = (known after apply)
          + start_period = (known after apply)
          + test         = (known after apply)
          + timeout      = (known after apply)
        }

      + labels {
          + label = (known after apply)
          + value = (known after apply)
        }

      + ports {
          + external = 8080
          + internal = 80
          + ip       = "0.0.0.0"
          + protocol = "tcp"
        }
    }

  # docker_image.nginx will be created
  + resource "docker_image" "nginx" {
      + id           = (known after apply)
      + keep_locally = false
      + latest       = (known after apply)
      + name         = "nginx:latest"
      + output       = (known after apply)
      + repo_digest  = (known after apply)
    }

Plan: 2 to add, 0 to change, 0 to destroy.

Changes to Outputs:
  + container_id = (known after apply)
  + image_id     = (known after apply)
╷
│ Warning: Deprecated attribute
│ 
│   on main.tf line 20, in resource "docker_container" "nginx":
│   20:   image = docker_image.nginx.latest
│ 
│ The attribute "latest" is deprecated. Refer to the provider documentation for details.
│ 
│ (and one more similar warning elsewhere)
╵

─────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────

Note: You didn't use the -out option to save this plan, so Terraform can't guarantee to take exactly these actions if you run "terraform apply" now.
(base) pradeep:~$
```

## Terraform Apply
```sh
(base) pradeep:~$terraform apply

Terraform used the selected providers to generate the following execution plan. Resource actions are indicated with the following symbols:
  + create

Terraform will perform the following actions:

  # docker_container.nginx will be created
  + resource "docker_container" "nginx" {
      + attach           = false
      + bridge           = (known after apply)
      + command          = (known after apply)
      + container_logs   = (known after apply)
      + entrypoint       = (known after apply)
      + env              = (known after apply)
      + exit_code        = (known after apply)
      + gateway          = (known after apply)
      + hostname         = (known after apply)
      + id               = (known after apply)
      + image            = (known after apply)
      + init             = (known after apply)
      + ip_address       = (known after apply)
      + ip_prefix_length = (known after apply)
      + ipc_mode         = (known after apply)
      + log_driver       = "json-file"
      + logs             = false
      + must_run         = true
      + name             = "ExampleNginxContainer"
      + network_data     = (known after apply)
      + read_only        = false
      + remove_volumes   = true
      + restart          = "no"
      + rm               = false
      + security_opts    = (known after apply)
      + shm_size         = (known after apply)
      + start            = true
      + stdin_open       = false
      + tty              = false

      + healthcheck {
          + interval     = (known after apply)
          + retries      = (known after apply)
          + start_period = (known after apply)
          + test         = (known after apply)
          + timeout      = (known after apply)
        }

      + labels {
          + label = (known after apply)
          + value = (known after apply)
        }

      + ports {
          + external = 8080
          + internal = 80
          + ip       = "0.0.0.0"
          + protocol = "tcp"
        }
    }

  # docker_image.nginx will be created
  + resource "docker_image" "nginx" {
      + id           = (known after apply)
      + keep_locally = false
      + latest       = (known after apply)
      + name         = "nginx:latest"
      + output       = (known after apply)
      + repo_digest  = (known after apply)
    }

Plan: 2 to add, 0 to change, 0 to destroy.

Changes to Outputs:
  + container_id = (known after apply)
  + image_id     = (known after apply)
╷
│ Warning: Deprecated attribute
│ 
│   on main.tf line 20, in resource "docker_container" "nginx":
│   20:   image = docker_image.nginx.latest
│ 
│ The attribute "latest" is deprecated. Refer to the provider documentation for details.
│ 
│ (and one more similar warning elsewhere)
╵

Do you want to perform these actions?
  Terraform will perform the actions described above.
  Only 'yes' will be accepted to approve.

  Enter a value: yes

docker_image.nginx: Creating...
docker_image.nginx: Still creating... [10s elapsed]
docker_image.nginx: Still creating... [20s elapsed]
docker_image.nginx: Creation complete after 22s [id=sha256:2b7d6430f78d432f89109b29d88d4c36c868cdbf15dc31d2132ceaa02b993763nginx:latest]
docker_container.nginx: Creating...
docker_container.nginx: Creation complete after 1s [id=8ce73469f925aa701608c0614839934e365fa36e4e0dc156c5f81d3176e8ed40]
╷
│ Warning: Deprecated attribute
│ 
│   on main.tf line 20, in resource "docker_container" "nginx":
│   20:   image = docker_image.nginx.latest
│ 
│ The attribute "latest" is deprecated. Refer to the provider documentation for details.
│ 
│ (and one more similar warning elsewhere)
╵

Apply complete! Resources: 2 added, 0 changed, 0 destroyed.

Outputs:

container_id = "8ce73469f925aa701608c0614839934e365fa36e4e0dc156c5f81d3176e8ed40"
image_id = "sha256:2b7d6430f78d432f89109b29d88d4c36c868cdbf15dc31d2132ceaa02b993763nginx:latest"
(base) pradeep:~$
```

## Terraform Output
```sh
(base) pradeep:~$terraform output
container_id = "8ce73469f925aa701608c0614839934e365fa36e4e0dc156c5f81d3176e8ed40"
image_id = "sha256:2b7d6430f78d432f89109b29d88d4c36c868cdbf15dc31d2132ceaa02b993763nginx:latest"
(base) pradeep:~$
```
## Terraform Console
```sh
(base) pradeep:~$terraform console
> var.container_name
"ExampleNginxContainer"
>
> exit
(base) pradeep:~$

```
## Terraform Refresh
```sh
(base) pradeep:~$terraform refresh
docker_image.nginx: Refreshing state... [id=sha256:2b7d6430f78d432f89109b29d88d4c36c868cdbf15dc31d2132ceaa02b993763nginx:latest]
docker_container.nginx: Refreshing state... [id=8ce73469f925aa701608c0614839934e365fa36e4e0dc156c5f81d3176e8ed40]
╷
│ Warning: Deprecated attribute
│ 
│   on main.tf line 20, in resource "docker_container" "nginx":
│   20:   image = docker_image.nginx.latest
│ 
│ The attribute "latest" is deprecated. Refer to the provider documentation for details.
│ 
│ (and one more similar warning elsewhere)
╵

Outputs:

container_id = "8ce73469f925aa701608c0614839934e365fa36e4e0dc156c5f81d3176e8ed40"
image_id = "sha256:2b7d6430f78d432f89109b29d88d4c36c868cdbf15dc31d2132ceaa02b993763nginx:latest"
(base) pradeep:~$
```

```sh
(base) pradeep:~$ls -la
total 64
drwxr-xr-x    9 pradeep  staff   288 Sep 11 14:40 .
drwxr-xr-x+ 110 pradeep  staff  3520 Sep 11 14:27 ..
drwxr-xr-x    3 pradeep  staff    96 Sep 11 14:32 .terraform
-rw-r--r--    1 pradeep  staff  1264 Sep 11 14:32 .terraform.lock.hcl
-rw-r--r--    1 pradeep  staff   412 Sep 11 14:26 main.tf
-rw-r--r--    1 pradeep  staff   217 Sep 11 14:27 outputs.tf
-rw-r--r--    1 pradeep  staff  4516 Sep 11 14:40 terraform.tfstate
-rw-r--r--    1 pradeep  staff  4534 Sep 11 14:40 terraform.tfstate.backup
-rw-r--r--    1 pradeep  staff   154 Sep 11 14:27 variables.tf
(base) pradeep:~$
```
## terrafrom.tfstate
```sh
(base) pradeep:~$cat terraform.tfstate
{
  "version": 4,
  "terraform_version": "1.2.8",
  "serial": 4,
  "lineage": "9b6c90c0-6a1f-7189-f5a5-d242abaf65bd",
  "outputs": {
    "container_id": {
      "value": "8ce73469f925aa701608c0614839934e365fa36e4e0dc156c5f81d3176e8ed40",
      "type": "string"
    },
    "image_id": {
      "value": "sha256:2b7d6430f78d432f89109b29d88d4c36c868cdbf15dc31d2132ceaa02b993763nginx:latest",
      "type": "string"
    }
  },
  "resources": [
    {
      "mode": "managed",
      "type": "docker_container",
      "name": "nginx",
      "provider": "provider[\"registry.terraform.io/kreuzwerker/docker\"]",
      "instances": [
        {
          "schema_version": 2,
          "attributes": {
            "attach": false,
            "bridge": "",
            "capabilities": [],
            "command": [
              "nginx",
              "-g",
              "daemon off;"
            ],
            "container_logs": null,
            "cpu_set": "",
            "cpu_shares": 0,
            "destroy_grace_seconds": null,
            "devices": [],
            "dns": [],
            "dns_opts": [],
            "dns_search": [],
            "domainname": "",
            "entrypoint": [
              "/docker-entrypoint.sh"
            ],
            "env": [],
            "exit_code": null,
            "gateway": "172.17.0.1",
            "group_add": [],
            "healthcheck": [],
            "host": [],
            "hostname": "8ce73469f925",
            "id": "8ce73469f925aa701608c0614839934e365fa36e4e0dc156c5f81d3176e8ed40",
            "image": "sha256:2b7d6430f78d432f89109b29d88d4c36c868cdbf15dc31d2132ceaa02b993763",
            "init": false,
            "ip_address": "172.17.0.2",
            "ip_prefix_length": 16,
            "ipc_mode": "private",
            "labels": [],
            "links": [],
            "log_driver": "json-file",
            "log_opts": {},
            "logs": false,
            "max_retry_count": 0,
            "memory": 0,
            "memory_swap": 0,
            "mounts": [],
            "must_run": true,
            "name": "ExampleNginxContainer",
            "network_alias": null,
            "network_data": [
              {
                "gateway": "172.17.0.1",
                "global_ipv6_address": "",
                "global_ipv6_prefix_length": 0,
                "ip_address": "172.17.0.2",
                "ip_prefix_length": 16,
                "ipv6_gateway": "",
                "network_name": "bridge"
              }
            ],
            "network_mode": "default",
            "networks": null,
            "networks_advanced": [],
            "pid_mode": "",
            "ports": [
              {
                "external": 8080,
                "internal": 80,
                "ip": "0.0.0.0",
                "protocol": "tcp"
              }
            ],
            "privileged": false,
            "publish_all_ports": false,
            "read_only": false,
            "remove_volumes": true,
            "restart": "no",
            "rm": false,
            "security_opts": [],
            "shm_size": 64,
            "start": true,
            "stdin_open": false,
            "sysctls": {},
            "tmpfs": {},
            "tty": false,
            "ulimit": [],
            "upload": [],
            "user": "",
            "userns_mode": "",
            "volumes": [],
            "working_dir": ""
          },
          "sensitive_attributes": [],
          "private": "eyJzY2hlbWFfdmVyc2lvbiI6IjIifQ==",
          "dependencies": [
            "docker_image.nginx"
          ]
        }
      ]
    },
    {
      "mode": "managed",
      "type": "docker_image",
      "name": "nginx",
      "provider": "provider[\"registry.terraform.io/kreuzwerker/docker\"]",
      "instances": [
        {
          "schema_version": 0,
          "attributes": {
            "build": [],
            "force_remove": null,
            "id": "sha256:2b7d6430f78d432f89109b29d88d4c36c868cdbf15dc31d2132ceaa02b993763nginx:latest",
            "keep_locally": false,
            "latest": "sha256:2b7d6430f78d432f89109b29d88d4c36c868cdbf15dc31d2132ceaa02b993763",
            "name": "nginx:latest",
            "output": null,
            "pull_trigger": null,
            "pull_triggers": null,
            "repo_digest": "nginx@sha256:b95a99feebf7797479e0c5eb5ec0bdfa5d9f504bc94da550c2f58e839ea6914f"
          },
          "sensitive_attributes": [],
          "private": "bnVsbA=="
        }
      ]
    }
  ]
}
(base) pradeep:~$
```

## Terraform Destroy

```sh
(base) pradeep:~$terraform destroy
docker_image.nginx: Refreshing state... [id=sha256:2b7d6430f78d432f89109b29d88d4c36c868cdbf15dc31d2132ceaa02b993763nginx:latest]
docker_container.nginx: Refreshing state... [id=8ce73469f925aa701608c0614839934e365fa36e4e0dc156c5f81d3176e8ed40]

Terraform used the selected providers to generate the following execution plan. Resource actions are indicated with the following symbols:
  - destroy

Terraform will perform the following actions:

  # docker_container.nginx will be destroyed
  - resource "docker_container" "nginx" {
      - attach            = false -> null
      - command           = [
          - "nginx",
          - "-g",
          - "daemon off;",
        ] -> null
      - cpu_shares        = 0 -> null
      - dns               = [] -> null
      - dns_opts          = [] -> null
      - dns_search        = [] -> null
      - entrypoint        = [
          - "/docker-entrypoint.sh",
        ] -> null
      - env               = [] -> null
      - gateway           = "172.17.0.1" -> null
      - group_add         = [] -> null
      - hostname          = "8ce73469f925" -> null
      - id                = "8ce73469f925aa701608c0614839934e365fa36e4e0dc156c5f81d3176e8ed40" -> null
      - image             = "sha256:2b7d6430f78d432f89109b29d88d4c36c868cdbf15dc31d2132ceaa02b993763" -> null
      - init              = false -> null
      - ip_address        = "172.17.0.2" -> null
      - ip_prefix_length  = 16 -> null
      - ipc_mode          = "private" -> null
      - links             = [] -> null
      - log_driver        = "json-file" -> null
      - log_opts          = {} -> null
      - logs              = false -> null
      - max_retry_count   = 0 -> null
      - memory            = 0 -> null
      - memory_swap       = 0 -> null
      - must_run          = true -> null
      - name              = "ExampleNginxContainer" -> null
      - network_data      = [
          - {
              - gateway                   = "172.17.0.1"
              - global_ipv6_address       = ""
              - global_ipv6_prefix_length = 0
              - ip_address                = "172.17.0.2"
              - ip_prefix_length          = 16
              - ipv6_gateway              = ""
              - network_name              = "bridge"
            },
        ] -> null
      - network_mode      = "default" -> null
      - privileged        = false -> null
      - publish_all_ports = false -> null
      - read_only         = false -> null
      - remove_volumes    = true -> null
      - restart           = "no" -> null
      - rm                = false -> null
      - security_opts     = [] -> null
      - shm_size          = 64 -> null
      - start             = true -> null
      - stdin_open        = false -> null
      - sysctls           = {} -> null
      - tmpfs             = {} -> null
      - tty               = false -> null

      - ports {
          - external = 8080 -> null
          - internal = 80 -> null
          - ip       = "0.0.0.0" -> null
          - protocol = "tcp" -> null
        }
    }

  # docker_image.nginx will be destroyed
  - resource "docker_image" "nginx" {
      - id           = "sha256:2b7d6430f78d432f89109b29d88d4c36c868cdbf15dc31d2132ceaa02b993763nginx:latest" -> null
      - keep_locally = false -> null
      - latest       = "sha256:2b7d6430f78d432f89109b29d88d4c36c868cdbf15dc31d2132ceaa02b993763" -> null
      - name         = "nginx:latest" -> null
      - repo_digest  = "nginx@sha256:b95a99feebf7797479e0c5eb5ec0bdfa5d9f504bc94da550c2f58e839ea6914f" -> null
    }

Plan: 0 to add, 0 to change, 2 to destroy.

Changes to Outputs:
  - container_id = "8ce73469f925aa701608c0614839934e365fa36e4e0dc156c5f81d3176e8ed40" -> null
  - image_id     = "sha256:2b7d6430f78d432f89109b29d88d4c36c868cdbf15dc31d2132ceaa02b993763nginx:latest" -> null
╷
│ Warning: Deprecated attribute
│ 
│   on main.tf line 20, in resource "docker_container" "nginx":
│   20:   image = docker_image.nginx.latest
│ 
│ The attribute "latest" is deprecated. Refer to the provider documentation for details.
╵

Do you really want to destroy all resources?
  Terraform will destroy all your managed infrastructure, as shown above.
  There is no undo. Only 'yes' will be accepted to confirm.

  Enter a value: yes

docker_container.nginx: Destroying... [id=8ce73469f925aa701608c0614839934e365fa36e4e0dc156c5f81d3176e8ed40]
docker_container.nginx: Destruction complete after 1s
docker_image.nginx: Destroying... [id=sha256:2b7d6430f78d432f89109b29d88d4c36c868cdbf15dc31d2132ceaa02b993763nginx:latest]
docker_image.nginx: Destruction complete after 0s

Destroy complete! Resources: 2 destroyed.
(base) pradeep:~$
```


After destroying the resources, lets take a look at the output variables

```sh
(base) pradeep:~$terraform output
╷
│ Warning: No outputs found
│ 
│ The state file either has no outputs defined, or all the defined outputs are empty. Please define an output in your configuration with the `output` keyword and run `terraform refresh` for it to become available. If you are using
│ interpolation, please verify the interpolated value is not empty. You can use the `terraform console` command to assist.
╵
(base) pradeep:~$
```


```sh
(base) pradeep:~$terraform console
> var.container_name
"ExampleNginxContainer"
> exit
(base) pradeep:~$
```



