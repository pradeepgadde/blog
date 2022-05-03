---
layout: single
title:  "Getting Started with Terraform"
date:   2022-05-03 04:56:04 +0530
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

# Getting started with Terraform

First, let us install Terraform package on our machine.
I have installed this earlier, so let us check the version first.
```sh
pradeep:~$terraform -version
Terraform v1.0.8
on darwin_amd64

Your version of Terraform is out of date! The latest version
is 1.1.9. You can update by downloading from https://www.terraform.io/downloads.html
```

As our current version is out of date, let us download the latest version.

```sh
pradeep:~$ls -la terraform
-rwxr-xr-x@ 1 pradeep  staff  72569296 Apr 20 19:02 terraform
pradeep:~$which terraform 
/usr/local/bin/terraform
pradeep:~$mv terraform /usr/local/bin 
pradeep:~$which terraform            
/usr/local/bin/terraform
pradeep:~$terraform -v
Terraform v1.1.9
on darwin_amd64
pradeep:~$
```

We have successfully updated the version.

Let us check what all we can do with `terraform` command.

```sh
pradeep:~$terraform -h
Usage: terraform [global options] <subcommand> [args]

The available commands for execution are listed below.
The primary workflow commands are given first, followed by
less common or more advanced commands.

Main commands:
  init          Prepare your working directory for other commands
  validate      Check whether the configuration is valid
  plan          Show changes required by the current configuration
  apply         Create or update infrastructure
  destroy       Destroy previously-created infrastructure

All other commands:
  console       Try Terraform expressions at an interactive command prompt
  fmt           Reformat your configuration in the standard style
  force-unlock  Release a stuck lock on the current workspace
  get           Install or upgrade remote Terraform modules
  graph         Generate a Graphviz graph of the steps in an operation
  import        Associate existing infrastructure with a Terraform resource
  login         Obtain and save credentials for a remote host
  logout        Remove locally-stored credentials for a remote host
  output        Show output values from your root module
  providers     Show the providers required for this configuration
  refresh       Update the state to match remote systems
  show          Show the current state or a saved plan
  state         Advanced state management
  taint         Mark a resource instance as not fully functional
  test          Experimental support for module integration testing
  untaint       Remove the 'tainted' state from a resource instance
  version       Show the current Terraform version
  workspace     Workspace management

Global options (use these before the subcommand, if any):
  -chdir=DIR    Switch to a different working directory before executing the
                given subcommand.
  -help         Show this help output, or the help for a specified subcommand.
  -version      An alias for the "version" subcommand.
pradeep:~$

```

Let us confirm the Docker version as well.

```sh
pradeep:~$docker -v
Docker version 20.10.14, build a224086
pradeep:~$
```
Let us create a new directory and define a file for our infrastructure.

```sh
pradeep:~$mkdir learn-terraform                 
pradeep:~$cd learn-terraform
pradeep:~$vi main.tf
pradeep:~$

```

```sh
pradeep:~$cat main.tf 
terraform {
  required_providers {
    docker = {
      source = "kreuzwerker/docker"
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
  name  = "tutorial"
  ports {
    internal = 80
    external = 8000
  }
}


pradeep:~$
```

The set of files used to describe infrastructure in Terraform is known as a Terraform *configuration*.

Each Terraform configuration must be in its own working directory. You created a working directory previously in `learn-terraform`. Review the `main.tf` file.

## Terraform {}

The `terraform {}` block contains Terraform settings, including the required providers Terraform will use to provision your infrastructure. For each provider, the `source` attribute defines an optional hostname, a namespace, and the provider type. Terraform installs providers from the [Terraform Registry](https://registry.terraform.io/) by default. In this example configuration, the `docker` provider's source is defined as `kreuzwerker/docker`, which is shorthand for `registry.terraform.io/kreuzwerker/docker`.

You can also set a version constraint for each provider defined in the `required_providers` block. The `version`attribute is optional, but we recommend using it to constrain the provider version so that Terraform does not install a version of the provider that does not work with your configuration. If you do not specify a provider version, Terraform will automatically download the most recent version during initialization.

## Provider {}

The `provider` block configures the specified provider, in this case `docker`. A provider is a plugin that Terraform uses to create and manage your resources.

You can use multiple provider blocks in your Terraform configuration to manage resources from different providers. You can even use different providers together. For example, you could pass the Docker image ID to a Kubernetes service.

## Resources {}

Use `resource` blocks to define components of your infrastructure. A resource might be a physical or virtual component such as a Docker container, or it can be a logical resource such as a Heroku application.

Resource blocks have two strings before the block: the resource type and the resource name. In this example, the first resource type is `docker_image` and the name is `nginx`. The prefix of the type maps to the name of the provider. In the example configuration, Terraform manages the `docker_image` resource with the `docker` provider. Together, the resource type and resource name form a unique ID for the resource. For example, the ID for your Docker image is `docker_image.nginx`.



## Initialize the directory

When you create a new configuration — or check out an existing configuration from version control — you need to initialize the directory with `terraform init`.

Initializing a configuration directory downloads and installs the providers defined in the configuration, which in this case is the `docker` provider.

```sh
pradeep:~$terraform init

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
pradeep:~$
```

Terraform downloads the `docker` provider and installs it in a hidden subdirectory of your current working directory, named `.terraform`. The `terraform init` command prints out which version of the provider was installed. Terraform also creates a lock file named `.terraform.lock.hcl` which specifies the exact provider versions used, so that you can control when you want to update the providers used for your project.



```sh
pradeep:~$ls -la
total 16
drwxr-xr-x    5 pradeep  staff    160 May  3 09:47 .
drwx------@ 467 pradeep  staff  14944 May  3 09:40 ..
drwxr-xr-x    3 pradeep  staff     96 May  3 09:47 .terraform
-rw-r--r--    1 pradeep  staff   1264 May  3 09:47 .terraform.lock.hcl
-rw-r--r--    1 pradeep  staff    392 May  3 09:41 main.tf

```

```sh
pradeep:~$ls -la .terraform/providers/registry.terraform.io/kreuzwerker/docker/2.13.0/darwin_amd64 
total 44664
drwxr-xr-x  6 pradeep  staff       192 May  3 09:47 .
drwxr-xr-x  3 pradeep  staff        96 May  3 09:47 ..
-rw-r--r--  1 pradeep  staff     24391 May  3 09:47 CHANGELOG.md
-rw-r--r--  1 pradeep  staff     16725 May  3 09:47 LICENSE
-rw-r--r--  1 pradeep  staff      1761 May  3 09:47 README.md
-rwxr-xr-x  1 pradeep  staff  22816128 May  3 09:47 terraform-provider-docker_v2.13.0
pradeep:~$
```
```sh
pradeep:~$cat .terraform.lock.hcl 
# This file is maintained automatically by "terraform init".
# Manual edits may be lost in future updates.

provider "registry.terraform.io/kreuzwerker/docker" {
  version     = "2.13.0"
  constraints = "~> 2.13.0"
  hashes = [
    "h1:Gp4HOAx3tkOaeWyM7qh184nCRmVjQSsu3CgKHac51ng=",
    "zh:0df685adc7b5740ae0def7235a44e1bce2f71beaf155319c2464ad2fba5cb321",
    "zh:2cf4b4f840fa84f1b906f4cca58c9782375e9988ad354afcd85b0180cd784205",
    "zh:347b189655afdc0df1919a26fb64cb745bb02d8fa2006a087cb6679a1b62319d",
    "zh:441521c85fecad348ca012db7b9d14544cbe0a237012f8a03d5660c73e9a32a6",
    "zh:462a1f67d26182fbb5ee78bb8d4764a2983804fa5f9971ca006da439e9e97055",
    "zh:53822eb743cd487cabbed3360221cc0404b80f933b746d80426a4e10fa2f958a",
    "zh:55c6eda01dd3d3f877aad16de6bf91e84bfa9c93f852869581429640be19d472",
    "zh:690bb327398f800f7945bab35b1ad2c6ec1c0fa7f8a1e5696b0bc4597540e3af",
    "zh:6c55a9a761596ca974a9cbaeee3179fb8f50916fad18d2422a2d818c3f4dc241",
    "zh:6efd9e6ffa4c4c73fd39c856456022aad6a3a0b176c550409345e894475bbf4f",
    "zh:811a37e3a66d5e99a81e0e66c817363205b030962fcec68bb96ab53b029ffeac",
    "zh:aacb4ab8dd11e834952877390bc19beabf9fb0591c101e96da559201f4b284ca",
    "zh:cecdf49f9488a10ac9416be354e7de3ed45114a25235cebc4ec6771696d980e9",
  ]
}
pradeep:~$

```

## Format and validate the configuration

We recommend using consistent formatting in all of your configuration files. The `terraform fmt` command automatically updates configurations in the current directory for readability and consistency.

Format your configuration. Terraform will print out the names of the files it modified, if any. In this case, your configuration file was already formatted correctly, so Terraform won't return any file names.

```sh
pradeep:~$terraform fmt
main.tf
pradeep:~$
```

You can also make sure your configuration is syntactically valid and internally consistent by using the `terraform validate` command.

Validate your configuration. The example configuration provided above is valid, so Terraform will return a success message.

```sh
pradeep:~$terraform validate
Success! The configuration is valid.

pradeep:~$

```

## Create infrastructure

Apply the configuration now with the `terraform apply` command. Terraform will print output similar to what is shown below. We have truncated some of the output to save space.

```sh
pradeep:~$terraform apply
╷
│ Error: Error pinging Docker server: Cannot connect to the Docker daemon at unix:///var/run/docker.sock. Is the docker daemon running?
│ 
│   with provider["registry.terraform.io/kreuzwerker/docker"],
│   on main.tf line 10, in provider "docker":
│   10: provider "docker" {}
│ 
╵
pradeep:~$
```

Hmm, I have not started Docker Desktop in my machine yet, hence the above error.

Let us start the Docker Desktop.

and Apply again

```sh
pradeep:~$terraform apply

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
      + name             = "tutorial"
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
          + external = 8000
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

Do you want to perform these actions?
  Terraform will perform the actions described above.
  Only 'yes' will be accepted to approve.

  Enter a value: 

```

Before it applies any changes, Terraform prints out the *execution plan* which describes the actions Terraform will take in order to change your infrastructure to match the configuration.

The output format is similar to the diff format generated by tools such as Git. The output has a `+` next to `docker_container.nginx`, meaning that Terraform will create this resource. Beneath that, it shows the attributes that will be set. When the value displayed is `(known after apply)`, it means that the value will not be known until the resource is created. For example, Docker assigns a random ID to images upon creation, so Terraform cannot know the value of the `id` attribute until you apply the change and the Docker provider returns that value.

Terraform will now pause and wait for your approval before proceeding. If anything in the plan seems incorrect or dangerous, it is safe to abort here with no changes made to your infrastructure.

In this case the plan is acceptable, so type `yes` at the confirmation prompt to proceed.

```sh
Enter a value: yes

docker_image.nginx: Creating...
docker_image.nginx: Still creating... [10s elapsed]
docker_image.nginx: Creation complete after 13s [id=sha256:fa5269854a5e615e51a72b17ad3fd1e01268f278a6684c8ed3c5f0cdce3f230bnginx:latest]
docker_container.nginx: Creating...
docker_container.nginx: Creation complete after 1s [id=1bb420d09a2c1bf37dbb8f8659fcb98228147b81ed00f58d4297d63cb0831047]

Apply complete! Resources: 2 added, 0 changed, 0 destroyed.
pradeep:~$

```

```sh
pradeep:~$docker ps
CONTAINER ID   IMAGE          COMMAND                  CREATED         STATUS         PORTS                  NAMES
1bb420d09a2c   fa5269854a5e   "/docker-entrypoint.…"   8 minutes ago   Up 8 minutes   0.0.0.0:8000->80/tcp   tutorial
pradeep:~$
```



![Terraform Docker]({{ site.url }}{{ site.baseurl }}/assets/images/terraform-docker.png)

## Inspect state

When you applied your configuration, Terraform wrote data into a file called `terraform.tfstate`. Terraform stores the IDs and properties of the resources it manages in this file, so that it can update or destroy those resources going forward.

```sh
pradeep:~$ls
main.tf			terraform.tfstate
pradeep:~$cat terraform.tfstate 
{
  "version": 4,
  "terraform_version": "1.1.9",
  "serial": 3,
  "lineage": "3dec399d-fdb8-7d81-70cb-e805a004761b",
  "outputs": {},
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
            "dns": null,
            "dns_opts": null,
            "dns_search": null,
            "domainname": "",
            "entrypoint": [
              "/docker-entrypoint.sh"
            ],
            "env": [],
            "exit_code": null,
            "gateway": "172.17.0.1",
            "group_add": null,
            "healthcheck": null,
            "host": [],
            "hostname": "1bb420d09a2c",
            "id": "1bb420d09a2c1bf37dbb8f8659fcb98228147b81ed00f58d4297d63cb0831047",
            "image": "sha256:fa5269854a5e615e51a72b17ad3fd1e01268f278a6684c8ed3c5f0cdce3f230b",
            "init": false,
            "ip_address": "172.17.0.2",
            "ip_prefix_length": 16,
            "ipc_mode": "private",
            "labels": [],
            "links": null,
            "log_driver": "json-file",
            "log_opts": null,
            "logs": false,
            "max_retry_count": 0,
            "memory": 0,
            "memory_swap": 0,
            "mounts": [],
            "must_run": true,
            "name": "tutorial",
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
                "external": 8000,
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
            "sysctls": null,
            "tmpfs": null,
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
            "id": "sha256:fa5269854a5e615e51a72b17ad3fd1e01268f278a6684c8ed3c5f0cdce3f230bnginx:latest",
            "keep_locally": false,
            "latest": "sha256:fa5269854a5e615e51a72b17ad3fd1e01268f278a6684c8ed3c5f0cdce3f230b",
            "name": "nginx:latest",
            "output": null,
            "pull_trigger": null,
            "pull_triggers": null,
            "repo_digest": "nginx@sha256:859ab6768a6f26a79bc42b231664111317d095a4f04e4b6fe79ce37b3d199097"
          },
          "sensitive_attributes": [],
          "private": "bnVsbA=="
        }
      ]
    }
  ]
}
pradeep:~$

```



The Terraform state file is the only way Terraform can track which resources it manages, and often contains sensitive information, so you must store your state file securely and restrict access to only trusted team members who need to manage your infrastructure. In production, we recommend [storing your state remotely](https://learn.hashicorp.com/tutorials/terraform/cloud-migrate?in=terraform/cloud) with Terraform Cloud or Terraform Enterprise. Terraform also supports several other [remote backends](https://www.terraform.io/docs/language/settings/backends/index.html) you can use to store and manage your state.

Inspect the current state using `terraform show`.

```sh
pradeep:~$terraform show
# docker_container.nginx:
resource "docker_container" "nginx" {
    attach            = false
    command           = [
        "nginx",
        "-g",
        "daemon off;",
    ]
    cpu_shares        = 0
    entrypoint        = [
        "/docker-entrypoint.sh",
    ]
    env               = []
    gateway           = "172.17.0.1"
    hostname          = "1bb420d09a2c"
    id                = "1bb420d09a2c1bf37dbb8f8659fcb98228147b81ed00f58d4297d63cb0831047"
    image             = "sha256:fa5269854a5e615e51a72b17ad3fd1e01268f278a6684c8ed3c5f0cdce3f230b"
    init              = false
    ip_address        = "172.17.0.2"
    ip_prefix_length  = 16
    ipc_mode          = "private"
    log_driver        = "json-file"
    logs              = false
    max_retry_count   = 0
    memory            = 0
    memory_swap       = 0
    must_run          = true
    name              = "tutorial"
    network_data      = [
        {
            gateway                   = "172.17.0.1"
            global_ipv6_address       = ""
            global_ipv6_prefix_length = 0
            ip_address                = "172.17.0.2"
            ip_prefix_length          = 16
            ipv6_gateway              = ""
            network_name              = "bridge"
        },
    ]
    network_mode      = "default"
    privileged        = false
    publish_all_ports = false
    read_only         = false
    remove_volumes    = true
    restart           = "no"
    rm                = false
    security_opts     = []
    shm_size          = 64
    start             = true
    stdin_open        = false
    tty               = false

    ports {
        external = 8000
        internal = 80
        ip       = "0.0.0.0"
        protocol = "tcp"
    }
}

# docker_image.nginx:
resource "docker_image" "nginx" {
    id           = "sha256:fa5269854a5e615e51a72b17ad3fd1e01268f278a6684c8ed3c5f0cdce3f230bnginx:latest"
    keep_locally = false
    latest       = "sha256:fa5269854a5e615e51a72b17ad3fd1e01268f278a6684c8ed3c5f0cdce3f230b"
    name         = "nginx:latest"
    repo_digest  = "nginx@sha256:859ab6768a6f26a79bc42b231664111317d095a4f04e4b6fe79ce37b3d199097"
}
pradeep:~$

```

## Manually Managing State

Terraform has a built-in command called `terraform state` for advanced state management. Use the `list`subcommand to list of the resources in your project's state.

```sh
pradeep:~$terraform state list
docker_container.nginx
docker_image.nginx
pradeep:~$

```

You have now created and updated a Docker container on your machine with Terraform. 

Once you no longer need infrastructure, you may want to destroy it to reduce your security exposure, costs, or resource overhead. For example, you may remove a production environment from service, or manage short-lived environments like build or testing systems. In addition to building and modifying infrastructure, Terraform can destroy or recreate the infrastructure it manages.

## Destroy

The terraform destroy command terminates resources managed by your Terraform project. This command is the inverse of terraform apply in that it terminates all the resources specified in your Terraform state. It does not destroy resources running elsewhere that are not managed by the current Terraform project.

Destroy the resources you created.

```sh
pradeep:~$terraform destroy
docker_image.nginx: Refreshing state... [id=sha256:fa5269854a5e615e51a72b17ad3fd1e01268f278a6684c8ed3c5f0cdce3f230bnginx:latest]
docker_container.nginx: Refreshing state... [id=1bb420d09a2c1bf37dbb8f8659fcb98228147b81ed00f58d4297d63cb0831047]

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
      - hostname          = "1bb420d09a2c" -> null
      - id                = "1bb420d09a2c1bf37dbb8f8659fcb98228147b81ed00f58d4297d63cb0831047" -> null
      - image             = "sha256:fa5269854a5e615e51a72b17ad3fd1e01268f278a6684c8ed3c5f0cdce3f230b" -> null
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
      - name              = "tutorial" -> null
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
          - external = 8000 -> null
          - internal = 80 -> null
          - ip       = "0.0.0.0" -> null
          - protocol = "tcp" -> null
        }
    }

  # docker_image.nginx will be destroyed
  - resource "docker_image" "nginx" {
      - id           = "sha256:fa5269854a5e615e51a72b17ad3fd1e01268f278a6684c8ed3c5f0cdce3f230bnginx:latest" -> null
      - keep_locally = false -> null
      - latest       = "sha256:fa5269854a5e615e51a72b17ad3fd1e01268f278a6684c8ed3c5f0cdce3f230b" -> null
      - name         = "nginx:latest" -> null
      - repo_digest  = "nginx@sha256:859ab6768a6f26a79bc42b231664111317d095a4f04e4b6fe79ce37b3d199097" -> null
    }

Plan: 0 to add, 0 to change, 2 to destroy.

Do you really want to destroy all resources?
  Terraform will destroy all your managed infrastructure, as shown above.
  There is no undo. Only 'yes' will be accepted to confirm.

  Enter a value: 

```

The `-` prefix indicates that the container will be destroyed. As with apply, Terraform shows its execution plan and waits for approval before making any changes.

Answer `yes` to execute this plan and destroy the infrastructure.

```sh
  Enter a value: yes

docker_container.nginx: Destroying... [id=1bb420d09a2c1bf37dbb8f8659fcb98228147b81ed00f58d4297d63cb0831047]
docker_container.nginx: Destruction complete after 1s
docker_image.nginx: Destroying... [id=sha256:fa5269854a5e615e51a72b17ad3fd1e01268f278a6684c8ed3c5f0cdce3f230bnginx:latest]
docker_image.nginx: Destruction complete after 0s

Destroy complete! Resources: 2 destroyed.
pradeep:~$

```

```sh
pradeep:~$docker ps    
CONTAINER ID   IMAGE     COMMAND   CREATED   STATUS    PORTS     NAMES
pradeep:~$

```

Just like with `apply`, Terraform determines the order to destroy your resources. In this case, Terraform identified a single container with no other dependencies, so it destroyed the container. In more complicated cases with multiple resources, Terraform will destroy them in a suitable order to respect dependencies.
