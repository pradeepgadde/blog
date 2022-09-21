---
layout: single
title:  "Terraform Providers and Configuration Directory"
date:   2022-09-21 03:58:04 +0530
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


In the `.terraform` directory of the current working folder, we can find the plugins.
In this case , we used the `hashicorp/local` provider.

```sh
(base) pradeep:~$ls -la
total 32
drwxr-xr-x  7 pradeep  staff   224 Sep 21 08:55 .
drwxr-xr-x  9 pradeep  staff   288 Sep 21 07:45 ..
drwxr-xr-x  3 pradeep  staff    96 Sep 21 07:45 .terraform
-rw-r--r--  1 pradeep  staff  1153 Sep 21 07:45 .terraform.lock.hcl
-rw-r--r--  1 pradeep  staff   149 Sep 21 08:53 local.tf
-rw-r--r--  1 pradeep  staff   851 Sep 21 08:55 terraform.tfstate
-rw-r--r--  1 pradeep  staff   155 Sep 21 08:55 terraform.tfstate.backup
(base) pradeep:~$ls -la .terraform/providers/registry.terraform.io 
total 0
drwxr-xr-x  3 pradeep  staff  96 Sep 21 07:45 .
drwxr-xr-x  3 pradeep  staff  96 Sep 21 07:45 ..
drwxr-xr-x  3 pradeep  staff  96 Sep 21 07:45 hashicorp
(base) pradeep:~$ls -la .terraform/providers/registry.terraform.io/hashicorp/local/2.2.3/darwin_amd64/terraform-provider-local_v2.2.3_x5
-rwxr-xr-x  1 pradeep  staff  15560448 Sep 21 07:45 .terraform/providers/registry.terraform.io/hashicorp/local/2.2.3/darwin_amd64/terraform-provider-local_v2.2.3_x5
(base) pradeep:~$
```
Plugin Naming Convention:

- Hostname: registry.terraform.io
- Organizational Namespace: hashicorp
- Type: local

If we do not specify any version, `latest` version will be installed.

```sh
Initializing provider plugins...
- Finding latest version of hashicorp/local...
- Installing hashicorp/local v2.2.3...
- Installed hashicorp/local v2.2.3 (signed by HashiCorp)
```

We have two `.tf` files in the same directory.

```sh
(base) pradeep:~$cat pets.tf 
resource "local_file" "pets" {
    filename = "/Users/pradeep/pets.txt"
    content = "Demo of multiple .tf files in the same directory"
}
(base) pradeep:~$
(base) pradeep:~$cat colors.tf 
resource "local_file" "colors" {
    filename = "/Users/pradeep/colors.txt"
    content = "Demo of multiple .tf files in the same directory"
}
(base) pradeep:~$

```

Let's see what happens if we apply this configuration file

```sh
(base) pradeep:~$terraform plan

Terraform used the selected providers to generate the following execution plan. Resource actions are indicated
with the following symbols:
  + create

Terraform will perform the following actions:

  # local_file.colors will be created
  + resource "local_file" "colors" {
      + content              = "Demo of multiple .tf files in the same directory"
      + directory_permission = "0777"
      + file_permission      = "0777"
      + filename             = "/Users/pradeep/colors.txt"
      + id                   = (known after apply)
    }

  # local_file.pets will be created
  + resource "local_file" "pets" {
      + content              = "Demo of multiple .tf files in the same directory"
      + directory_permission = "0777"
      + file_permission      = "0777"
      + filename             = "/Users/pradeep/pets.txt"
      + id                   = (known after apply)
    }

Plan: 2 to add, 0 to change, 0 to destroy.

──────────────────────────────────────────────────────────────────────────────────────────────────────────────────

Note: You didn't use the -out option to save this plan, so Terraform can't guarantee to take exactly these actions
if you run "terraform apply" now.
(base) pradeep:~$

```



```sh
(base) pradeep:~$terraform apply

Terraform used the selected providers to generate the following execution plan. Resource actions are indicated
with the following symbols:
  + create

Terraform will perform the following actions:

  # local_file.colors will be created
  + resource "local_file" "colors" {
      + content              = "Demo of multiple .tf files in the same directory"
      + directory_permission = "0777"
      + file_permission      = "0777"
      + filename             = "/Users/pradeep/colors.txt"
      + id                   = (known after apply)
    }

  # local_file.pets will be created
  + resource "local_file" "pets" {
      + content              = "Demo of multiple .tf files in the same directory"
      + directory_permission = "0777"
      + file_permission      = "0777"
      + filename             = "/Users/pradeep/pets.txt"
      + id                   = (known after apply)
    }

Plan: 2 to add, 0 to change, 0 to destroy.

Do you want to perform these actions?
  Terraform will perform the actions described above.
  Only 'yes' will be accepted to approve.

  Enter a value: yes

local_file.pets: Creating...
local_file.colors: Creating...
local_file.pets: Creation complete after 0s [id=2f20cf483ae1926cb52d153141858330208a61ce]
local_file.colors: Creation complete after 0s [id=2f20cf483ae1926cb52d153141858330208a61ce]

Apply complete! Resources: 2 added, 0 changed, 0 destroyed.
(base) pradeep:~$

```



```sh
(base) pradeep:~$cat /Users/pradeep/colors.txt 
Demo of multiple .tf files in the same directory%                         (base) pradeep:~$
```



```sh
(base) pradeep:~$cat /Users/pradeep/pets.txt 
Demo of multiple .tf files in the same directory%                         (base) pradeep:~$
````



```sh
(base) pradeep:~$terraform show
# local_file.colors:
resource "local_file" "colors" {
    content              = "Demo of multiple .tf files in the same directory"
    directory_permission = "0777"
    file_permission      = "0777"
    filename             = "/Users/pradeep/colors.txt"
    id                   = "2f20cf483ae1926cb52d153141858330208a61ce"
}

# local_file.pets:
resource "local_file" "pets" {
    content              = "Demo of multiple .tf files in the same directory"
    directory_permission = "0777"
    file_permission      = "0777"
    filename             = "/Users/pradeep/pets.txt"
    id                   = "2f20cf483ae1926cb52d153141858330208a61ce"
}
(base) pradeep:~$

```



```sh
(base) pradeep:~$terraform destroy
local_file.pets: Refreshing state... [id=2f20cf483ae1926cb52d153141858330208a61ce]
local_file.colors: Refreshing state... [id=2f20cf483ae1926cb52d153141858330208a61ce]

Terraform used the selected providers to generate the following execution plan. Resource actions are indicated
with the following symbols:
  - destroy

Terraform will perform the following actions:

  # local_file.colors will be destroyed
  - resource "local_file" "colors" {
      - content              = "Demo of multiple .tf files in the same directory" -> null
      - directory_permission = "0777" -> null
      - file_permission      = "0777" -> null
      - filename             = "/Users/pradeep/colors.txt" -> null
      - id                   = "2f20cf483ae1926cb52d153141858330208a61ce" -> null
    }

  # local_file.pets will be destroyed
  - resource "local_file" "pets" {
      - content              = "Demo of multiple .tf files in the same directory" -> null
      - directory_permission = "0777" -> null
      - file_permission      = "0777" -> null
      - filename             = "/Users/pradeep/pets.txt" -> null
      - id                   = "2f20cf483ae1926cb52d153141858330208a61ce" -> null
    }

Plan: 0 to add, 0 to change, 2 to destroy.

Do you really want to destroy all resources?
  Terraform will destroy all your managed infrastructure, as shown above.
  There is no undo. Only 'yes' will be accepted to confirm.

  Enter a value: yes

local_file.colors: Destroying... [id=2f20cf483ae1926cb52d153141858330208a61ce]
local_file.pets: Destroying... [id=2f20cf483ae1926cb52d153141858330208a61ce]
local_file.colors: Destruction complete after 0s
local_file.pets: Destruction complete after 0s

Destroy complete! Resources: 2 destroyed.
(base) pradeep:~$

```

```sh
(base) pradeep:~$ls -la
total 40
drwxr-xr-x  8 pradeep  staff   256 Sep 21 10:11 .
drwxr-xr-x  9 pradeep  staff   288 Sep 21 07:45 ..
drwxr-xr-x  3 pradeep  staff    96 Sep 21 07:45 .terraform
-rw-r--r--  1 pradeep  staff  1153 Sep 21 07:45 .terraform.lock.hcl
-rw-r--r--  1 pradeep  staff   143 Sep 21 10:05 colors.tf
-rw-r--r--  1 pradeep  staff   139 Sep 21 10:03 pets.tf
-rw-r--r--  1 pradeep  staff   155 Sep 21 10:11 terraform.tfstate
-rw-r--r--  1 pradeep  staff  1607 Sep 21 10:11 terraform.tfstate.backup
(base) pradeep:~$

```



```sh
(base) pradeep:~$cat .terraform.lock.hcl 
# This file is maintained automatically by "terraform init".
# Manual edits may be lost in future updates.

provider "registry.terraform.io/hashicorp/local" {
  version = "2.2.3"
  hashes = [
    "h1:KmHz81iYgw9Xn2L3Carc2uAzvFZ1XsE7Js3qlVeC77k=",
    "zh:04f0978bb3e052707b8e82e46780c371ac1c66b689b4a23bbc2f58865ab7d5c0",
    "zh:6484f1b3e9e3771eb7cc8e8bab8b35f939a55d550b3f4fb2ab141a24269ee6aa",
    "zh:78a56d59a013cb0f7eb1c92815d6eb5cf07f8b5f0ae20b96d049e73db915b238",
    "zh:78d5eefdd9e494defcb3c68d282b8f96630502cac21d1ea161f53cfe9bb483b3",
    "zh:8aa9950f4c4db37239bcb62e19910c49e47043f6c8587e5b0396619923657797",
    "zh:996beea85f9084a725ff0e6473a4594deb5266727c5f56e9c1c7c62ded6addbb",
    "zh:9a7ef7a21f48fabfd145b2e2a4240ca57517ad155017e86a30860d7c0c109de3",
    "zh:a63e70ac052aa25120113bcddd50c1f3cfe61f681a93a50cea5595a4b2cc3e1c",
    "zh:a6e8d46f94108e049ad85dbed60354236dc0b9b5ec8eabe01c4580280a43d3b8",
    "zh:bb112ce7efbfcfa0e65ed97fa245ef348e0fd5bfa5a7e4ab2091a9bd469f0a9e",
    "zh:d7bec0da5c094c6955efed100f3fe22fca8866859f87c025be1760feb174d6d9",
    "zh:fb9f271b72094d07cef8154cd3d50e9aa818a0ea39130bc193132ad7b23076fd",
  ]
}
(base) pradeep:~$

```



```sh
(base) pradeep:~$cat terraform.tfstate
{
  "version": 4,
  "terraform_version": "1.2.8",
  "serial": 4,
  "lineage": "0f92aa37-49a7-b740-b518-243055789f7d",
  "outputs": {},
  "resources": []
}
(base) pradeep:~$
```



```sh
(base) pradeep:~$cat terraform.tfstate.backup 
{
  "version": 4,
  "terraform_version": "1.2.8",
  "serial": 1,
  "lineage": "0f92aa37-49a7-b740-b518-243055789f7d",
  "outputs": {},
  "resources": [
    {
      "mode": "managed",
      "type": "local_file",
      "name": "colors",
      "provider": "provider[\"registry.terraform.io/hashicorp/local\"]",
      "instances": [
        {
          "schema_version": 0,
          "attributes": {
            "content": "Demo of multiple .tf files in the same directory",
            "content_base64": null,
            "directory_permission": "0777",
            "file_permission": "0777",
            "filename": "/Users/pradeep/colors.txt",
            "id": "2f20cf483ae1926cb52d153141858330208a61ce",
            "sensitive_content": null,
            "source": null
          },
          "sensitive_attributes": [],
          "private": "bnVsbA=="
        }
      ]
    },
    {
      "mode": "managed",
      "type": "local_file",
      "name": "pets",
      "provider": "provider[\"registry.terraform.io/hashicorp/local\"]",
      "instances": [
        {
          "schema_version": 0,
          "attributes": {
            "content": "Demo of multiple .tf files in the same directory",
            "content_base64": null,
            "directory_permission": "0777",
            "file_permission": "0777",
            "filename": "/Users/pradeep/pets.txt",
            "id": "2f20cf483ae1926cb52d153141858330208a61ce",
            "sensitive_content": null,
            "source": null
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

