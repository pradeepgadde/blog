---
layout: single
title:  "Terraform Output"
date:   2022-09-11 09:58:04 +0530
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

# Terraform Outputs

In this example, let us take a look at the Terraform sensitive outputs.
We will re-run the previous example, but this time, with sensitive flag turned on!

```sh
(base) pradeep:~$cat outputs.tf 
output "container_id" {
  description = "ID of the Docker container"
  value       = docker_container.nginx.id
  sensitive   = true
}

output "image_id" {
  description = "ID of the Docker image"
  value       = docker_image.nginx.id
}


(base) pradeep:~$
```


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
  + container_id = (sensitive value)
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
docker_image.nginx: Creation complete after 18s [id=sha256:2b7d6430f78d432f89109b29d88d4c36c868cdbf15dc31d2132ceaa02b993763nginx:latest]
docker_container.nginx: Creating...
docker_container.nginx: Creation complete after 1s [id=38fce0ff21b832dae05fd2cb1892c75546810ebd92dcd0ea840bd3d90bab053d]
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

container_id = <sensitive>
image_id = "sha256:2b7d6430f78d432f89109b29d88d4c36c868cdbf15dc31d2132ceaa02b993763nginx:latest"
(base) pradeep:~$
```

We can see out of the two output variables, we have marked one of them as sensitive, so its value is not shown when we applied.

```sh
base) pradeep:~$terraform output
container_id = <sensitive>
image_id = "sha256:2b7d6430f78d432f89109b29d88d4c36c868cdbf15dc31d2132ceaa02b993763nginx:latest"
(base) pradeep:~$
```

but, if we output the specific variable, we can see its value

```sh
(base) pradeep:~$terraform output container_id
"38fce0ff21b832dae05fd2cb1892c75546810ebd92dcd0ea840bd3d90bab053d"
(base) pradeep:~$
```

The Terraform CLI output is designed to be parsed by humans. To get machine-readable format for automation, use the `-json` flag for JSON-formatted output.

```sh
(base) pradeep:~$terraform output -json             
{
  "container_id": {
    "sensitive": true,
    "type": "string",
    "value": "38fce0ff21b832dae05fd2cb1892c75546810ebd92dcd0ea840bd3d90bab053d"
  },
  "image_id": {
    "sensitive": false,
    "type": "string",
    "value": "sha256:2b7d6430f78d432f89109b29d88d4c36c868cdbf15dc31d2132ceaa02b993763nginx:latest"
  }
}
(base) pradeep:~$
```



Output values are stored as plain text in your state file — including those flagged as sensitive. Use the `grep`command to see the values of the sensitive outputs in your state file.

```sh
(base) pradeep:~$grep --after-context=10 outputs terraform.tfstate
  "outputs": {
    "container_id": {
      "value": "38fce0ff21b832dae05fd2cb1892c75546810ebd92dcd0ea840bd3d90bab053d",
      "type": "string",
      "sensitive": true
    },
    "image_id": {
      "value": "sha256:2b7d6430f78d432f89109b29d88d4c36c868cdbf15dc31d2132ceaa02b993763nginx:latest",
      "type": "string"
    }
  },
(base) pradeep:~$grep --after-context=5 outputs terraform.tfstate
  "outputs": {
    "container_id": {
      "value": "38fce0ff21b832dae05fd2cb1892c75546810ebd92dcd0ea840bd3d90bab053d",
      "type": "string",
      "sensitive": true
    },
(base) pradeep:~$
```



The `sensitive` argument for outputs can help avoid inadvertent exposure of those values. However, you must still keep your Terraform state secure to avoid exposing these values.

Terraform does not redact sensitive output values with the `-json` option, because it assumes that an automation tool will use the output.