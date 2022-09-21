---
layout: single
title:  "Terraform Local Provider"
date:   2022-09-21 02:58:04 +0530
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
# Creating Terraform Local File Resources
```sh
(base) pradeep:~$pwd
/Users/pradeep/Downloads/learn-terraform/local
(base) pradeep:~$cat local.tf 
resource "local_file" "demo" {
    filename = "/Users/pradeep/learn-terraform"
    content = "This is a demo of local provider"
}
(base) pradeep:~$
```
## Init
```sh
(base) pradeep:~$terraform init

Initializing the backend...

Initializing provider plugins...
- Finding latest version of hashicorp/local...
- Installing hashicorp/local v2.2.3...
- Installed hashicorp/local v2.2.3 (signed by HashiCorp)

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
## Plan
```sh
(base) pradeep:~$terraform plan

Terraform used the selected providers to generate the following execution plan. Resource actions are indicated
with the following symbols:
  + create

Terraform will perform the following actions:

  # local_file.demo will be created
  + resource "local_file" "demo" {
      + content              = "This is a demo of local provider"
      + directory_permission = "0777"
      + file_permission      = "0777"
      + filename             = "/Users/pradeep/learn-terraform"
      + id                   = (known after apply)
    }

Plan: 1 to add, 0 to change, 0 to destroy.

──────────────────────────────────────────────────────────────────────────────────────────────────────────────────

Note: You didn't use the -out option to save this plan, so Terraform can't guarantee to take exactly these actions
if you run "terraform apply" now.
(base) pradeep:~$
```
## Apply
```sh
(base) pradeep:~$terraform apply

Terraform used the selected providers to generate the following execution plan. Resource actions are indicated
with the following symbols:
  + create

Terraform will perform the following actions:

  # local_file.demo will be created
  + resource "local_file" "demo" {
      + content              = "This is a demo of local provider"
      + directory_permission = "0777"
      + file_permission      = "0777"
      + filename             = "/Users/pradeep/learn-terraform"
      + id                   = (known after apply)
    }

Plan: 1 to add, 0 to change, 0 to destroy.

Do you want to perform these actions?
  Terraform will perform the actions described above.
  Only 'yes' will be accepted to approve.

  Enter a value: yes

local_file.demo: Creating...
local_file.demo: Creation complete after 0s [id=5619c1ddadcfef77e25c6a072807cdde7d815023]

Apply complete! Resources: 1 added, 0 changed, 0 destroyed.
(base) pradeep:~$
```
Verify that the file is created 
```sh
(base) pradeep:~$ls -la /Users/pradeep/learn-terraform     
-rwxr-xr-x  1 pradeep  staff  32 Sep 21 08:05 /Users/pradeep/learn-terraform
(base) pradeep:~$cat /Users/pradeep/learn-terraform
This is a demo of local provider%                                         base) pradeep:~$
```

## Change
Modify the file permissions

```sh
(base) pradeep:~$cat local.tf 
resource "local_file" "demo" {
    filename = "/Users/pradeep/learn-terraform"
    content = "This is a demo of local provider"
    file_permission = "0700"
}
(base) pradeep:~$
```

```sh
(base) pradeep:~$terraform plan
local_file.demo: Refreshing state... [id=5619c1ddadcfef77e25c6a072807cdde7d815023]

Terraform used the selected providers to generate the following execution plan. Resource actions are indicated
with the following symbols:
-/+ destroy and then create replacement

Terraform will perform the following actions:

  # local_file.demo must be replaced
-/+ resource "local_file" "demo" {
      ~ file_permission      = "0777" -> "0700" # forces replacement
      ~ id                   = "5619c1ddadcfef77e25c6a072807cdde7d815023" -> (known after apply)
        # (3 unchanged attributes hidden)
    }

Plan: 1 to add, 0 to change, 1 to destroy.

──────────────────────────────────────────────────────────────────────────────────────────────────────────────────

Note: You didn't use the -out option to save this plan, so Terraform can't guarantee to take exactly these actions
if you run "terraform apply" now.
(base) pradeep:~$
```

```sh
(base) pradeep:~$terraform apply
local_file.demo: Refreshing state... [id=5619c1ddadcfef77e25c6a072807cdde7d815023]

Terraform used the selected providers to generate the following execution plan. Resource actions are indicated
with the following symbols:
-/+ destroy and then create replacement

Terraform will perform the following actions:

  # local_file.demo must be replaced
-/+ resource "local_file" "demo" {
      ~ file_permission      = "0777" -> "0700" # forces replacement
      ~ id                   = "5619c1ddadcfef77e25c6a072807cdde7d815023" -> (known after apply)
        # (3 unchanged attributes hidden)
    }

Plan: 1 to add, 0 to change, 1 to destroy.

Do you want to perform these actions?
  Terraform will perform the actions described above.
  Only 'yes' will be accepted to approve.

  Enter a value: yes

local_file.demo: Destroying... [id=5619c1ddadcfef77e25c6a072807cdde7d815023]
local_file.demo: Destruction complete after 0s
local_file.demo: Creating...
local_file.demo: Creation complete after 0s [id=5619c1ddadcfef77e25c6a072807cdde7d815023]

Apply complete! Resources: 1 added, 0 changed, 1 destroyed.
(base) pradeep:~$
```

```sh
(base) pradeep:~$cat /Users/pradeep/learn-terraform
This is a demo of local provider%                                         (base) pradeep:~$ls -la /Users/pradeep/learn-terraform
-rwx------  1 pradeep  staff  32 Sep 21 08:36 /Users/pradeep/learn-terraform
(base) pradeep:~$
```

## Destroy

```sh
(base) pradeep:~$terraform destroy
local_file.demo: Refreshing state... [id=5619c1ddadcfef77e25c6a072807cdde7d815023]

Terraform used the selected providers to generate the following execution plan. Resource actions are indicated
with the following symbols:
  - destroy

Terraform will perform the following actions:

  # local_file.demo will be destroyed
  - resource "local_file" "demo" {
      - content              = "This is a demo of local provider" -> null
      - directory_permission = "0777" -> null
      - file_permission      = "0700" -> null
      - filename             = "/Users/pradeep/learn-terraform" -> null
      - id                   = "5619c1ddadcfef77e25c6a072807cdde7d815023" -> null
    }

Plan: 0 to add, 0 to change, 1 to destroy.

Do you really want to destroy all resources?
  Terraform will destroy all your managed infrastructure, as shown above.
  There is no undo. Only 'yes' will be accepted to confirm.

  Enter a value: yes

local_file.demo: Destroying... [id=5619c1ddadcfef77e25c6a072807cdde7d815023]
local_file.demo: Destruction complete after 0s

Destroy complete! Resources: 1 destroyed.
(base) pradeep:~$
```
Verify if the local file is deleted or not.
```sh
(base) pradeep:~$ls -la /Users/pradeep/learn-terraform
ls: /Users/pradeep/learn-terraform: No such file or directory
(base) pradeep:~$cat /Users/pradeep/learn-terraform
cat: /Users/pradeep/learn-terraform: No such file or directory
(base) pradeep:~$
```
Add some sensitive content

```sh
(base) pradeep:~$cat local.tf 
resource "local_file" "demo" {
    filename = "/Users/pradeep/learn-terraform"
    content = "This is a demo of local provider"
    file_permission = "0700"
    sensitive_content = "Another demo"
}
(base) pradeep:~$
```

```sh
(base) pradeep:~$terraform plan
╷
│ Error: Invalid combination of arguments
│ 
│   with local_file.demo,
│   on local.tf line 1, in resource "local_file" "demo":
│    1: resource "local_file" "demo" {
│ 
│ "content_base64": only one of `content,content_base64,sensitive_content,source` can be specified, but
│ `content,sensitive_content` were specified.
╵
╷
│ Error: Invalid combination of arguments
│ 
│   with local_file.demo,
│   on local.tf line 1, in resource "local_file" "demo":
│    1: resource "local_file" "demo" {
│ 
│ "source": only one of `content,content_base64,sensitive_content,source` can be specified, but
│ `content,sensitive_content` were specified.
╵
╷
│ Error: Invalid combination of arguments
│ 
│   with local_file.demo,
│   on local.tf line 3, in resource "local_file" "demo":
│    3:     content = "This is a demo of local provider"
│ 
│ "content": only one of `content,content_base64,sensitive_content,source` can be specified, but
│ `content,sensitive_content` were specified.
╵
╷
│ Error: Invalid combination of arguments
│ 
│   with local_file.demo,
│   on local.tf line 5, in resource "local_file" "demo":
│    5:     sensitive_content = "Another demo"
│ 
│ "sensitive_content": only one of `content,content_base64,sensitive_content,source` can be specified, but
│ `content,sensitive_content` were specified.
╵
(base) pradeep:~$
```

```sh
(base) pradeep:~$cat local.tf 
resource "local_file" "demo" {
    filename = "/Users/pradeep/learn-terraform"
    file_permission = "0700"
    sensitive_content = "Another demo"
}
(base) pradeep:~$
```

```sh
(base) pradeep:~$terraform plan

Terraform used the selected providers to generate the following execution plan. Resource actions are indicated
with the following symbols:
  + create

Terraform will perform the following actions:

  # local_file.demo will be created
  + resource "local_file" "demo" {
      + directory_permission = "0777"
      + file_permission      = "0700"
      + filename             = "/Users/pradeep/learn-terraform"
      + id                   = (known after apply)
      + sensitive_content    = (sensitive value)
    }

Plan: 1 to add, 0 to change, 0 to destroy.
╷
│ Warning: Argument is deprecated
│ 
│   with local_file.demo,
│   on local.tf line 4, in resource "local_file" "demo":
│    4:     sensitive_content = "Another demo"
│ 
│ Use the `local_sensitive_file` resource instead.
│ 
│ (and one more similar warning elsewhere)
╵

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

  # local_file.demo will be created
  + resource "local_file" "demo" {
      + directory_permission = "0777"
      + file_permission      = "0700"
      + filename             = "/Users/pradeep/learn-terraform"
      + id                   = (known after apply)
      + sensitive_content    = (sensitive value)
    }

Plan: 1 to add, 0 to change, 0 to destroy.
╷
│ Warning: Argument is deprecated
│ 
│   with local_file.demo,
│   on local.tf line 4, in resource "local_file" "demo":
│    4:     sensitive_content = "Another demo"
│ 
│ Use the `local_sensitive_file` resource instead.
│ 
│ (and one more similar warning elsewhere)
╵

Do you want to perform these actions?
  Terraform will perform the actions described above.
  Only 'yes' will be accepted to approve.

  Enter a value: yes

local_file.demo: Creating...
local_file.demo: Creation complete after 0s [id=2beb569b8980210262aae09cb1eefb25de810d0a]
╷
│ Warning: Argument is deprecated
│ 
│   with local_file.demo,
│   on local.tf line 4, in resource "local_file" "demo":
│    4:     sensitive_content = "Another demo"
│ 
│ Use the `local_sensitive_file` resource instead.
╵

Apply complete! Resources: 1 added, 0 changed, 0 destroyed.
(base) pradeep:~$
```

```sh
(base) pradeep:~$cat /Users/pradeep/learn-terraform
Another demo%                                                             (base) pradeep:~$
```
