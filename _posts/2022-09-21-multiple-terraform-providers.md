---
layout: single
title:  "Multiple Terraform Providers"
date:   2022-09-21 03:59:04 +0530
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

# Using Multiple Terraform Providers

```sh
(base) pradeep:~$ls
main.tf
(base) pradeep:~$
(base) pradeep:~$cat main.tf 
resource "local_file" "pets" {
    filename = "/Users/pradeep/pets.txt"
    content = "Demo of multiple .tf files in the same directory"
}

resource "random_password" "password" {
  length           = 8
  special          = true
}%                                                                       (base) pradeep:~$
```

```sh
(base) pradeep:~$terraform init

Initializing the backend...

Initializing provider plugins...
- Finding latest version of hashicorp/random...
- Reusing previous version of hashicorp/local from the dependency lock file
- Installing hashicorp/random v3.4.3...
- Installed hashicorp/random v3.4.3 (signed by HashiCorp)
- Using previously-installed hashicorp/local v2.2.3

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
(base) pradeep:~$
```
We can see, in this case, we are using previously-installed `hashicorp/local v2.2.3` along with another plugin `hashicorp/random v3.4.3`.

```sh
(base) pradeep:~$ls -la .terraform/providers/registry.terraform.io/hashicorp/
total 0
drwxr-xr-x  4 pradeep  staff  128 Sep 21 10:35 .
drwxr-xr-x  3 pradeep  staff   96 Sep 21 07:45 ..
drwxr-xr-x  3 pradeep  staff   96 Sep 21 07:45 local
drwxr-xr-x  3 pradeep  staff   96 Sep 21 10:35 random
```

```sh
(base) pradeep:~$terraform apply

Terraform used the selected providers to generate the following execution plan. Resource actions are indicated
with the following symbols:
  + create

Terraform will perform the following actions:

  # local_file.pets will be created
  + resource "local_file" "pets" {
      + content              = "Demo of multiple .tf files in the same directory"
      + directory_permission = "0777"
      + file_permission      = "0777"
      + filename             = "/Users/pradeep/pets.txt"
      + id                   = (known after apply)
    }

  # random_password.password will be created
  + resource "random_password" "password" {
      + bcrypt_hash = (sensitive value)
      + id          = (known after apply)
      + length      = 8
      + lower       = true
      + min_lower   = 0
      + min_numeric = 0
      + min_special = 0
      + min_upper   = 0
      + number      = true
      + numeric     = true
      + result      = (sensitive value)
      + special     = true
      + upper       = true
    }

Plan: 2 to add, 0 to change, 0 to destroy.

Do you want to perform these actions?
  Terraform will perform the actions described above.
  Only 'yes' will be accepted to approve.

  Enter a value: yes

local_file.pets: Creating...
local_file.pets: Creation complete after 0s [id=2f20cf483ae1926cb52d153141858330208a61ce]
random_password.password: Creating...
random_password.password: Creation complete after 0s [id=none]

Apply complete! Resources: 2 added, 0 changed, 0 destroyed.
(base) pradeep:~$
```

```sh
(base) pradeep:~$cat terraform.tfstate 
{
  "version": 4,
  "terraform_version": "1.2.8",
  "serial": 3,
  "lineage": "7eff2629-18ae-80ec-462f-6baff3839485",
  "outputs": {},
  "resources": [
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
    },
    {
      "mode": "managed",
      "type": "random_password",
      "name": "password",
      "provider": "provider[\"registry.terraform.io/hashicorp/random\"]",
      "instances": [
        {
          "schema_version": 3,
          "attributes": {
            "bcrypt_hash": "$2a$10$DUpH9/Glrt8S/Ot9Xi1eeeyM5SmZqSrDNLS2NTxoxq4gWIS2aaYtO",
            "id": "none",
            "keepers": null,
            "length": 8,
            "lower": true,
            "min_lower": 0,
            "min_numeric": 0,
            "min_special": 0,
            "min_upper": 0,
            "number": true,
            "numeric": true,
            "override_special": null,
            "result": "B3zo2*K]",
            "special": true,
            "upper": true
          },
          "sensitive_attributes": []
        }
      ]
    }
  ]
}
(base) pradeep:~$
```

A file `pets.txt` and a random password `"result": "B3zo2*K]"` are created.

```sh
(base) pradeep:~$cat main.tf 
resource "local_file" "pets" {
    filename = "/Users/pradeep/pets.txt"
    content = "Demo of multiple .tf files in the same directory"
}

resource "random_password" "password" {
  length           = 8
  special          = true
}

resource "random_pet" "my-pet" {
	      prefix = "Mrs"
	      separator = "."
	      length = "1"
}
(base) pradeep:~$
```


```sh
(base) pradeep:~$terraform plan
local_file.pets: Refreshing state... [id=2f20cf483ae1926cb52d153141858330208a61ce]
random_password.password: Refreshing state... [id=none]

Terraform used the selected providers to generate the following execution plan. Resource actions are indicated
with the following symbols:
  + create

Terraform will perform the following actions:

  # random_pet.my-pet will be created
  + resource "random_pet" "my-pet" {
      + id        = (known after apply)
      + length    = 1
      + prefix    = "Mrs"
      + separator = "."
    }

Plan: 1 to add, 0 to change, 0 to destroy.

──────────────────────────────────────────────────────────────────────────────────────────────────────────────────

Note: You didn't use the -out option to save this plan, so Terraform can't guarantee to take exactly these actions
if you run "terraform apply" now.
(base) pradeep:~$
```

```sh
(base) pradeep:~$terraform apply
local_file.pets: Refreshing state... [id=2f20cf483ae1926cb52d153141858330208a61ce]
random_password.password: Refreshing state... [id=none]

Terraform used the selected providers to generate the following execution plan. Resource actions are indicated
with the following symbols:
  + create

Terraform will perform the following actions:

  # random_pet.my-pet will be created
  + resource "random_pet" "my-pet" {
      + id        = (known after apply)
      + length    = 1
      + prefix    = "Mrs"
      + separator = "."
    }

Plan: 1 to add, 0 to change, 0 to destroy.

Do you want to perform these actions?
  Terraform will perform the actions described above.
  Only 'yes' will be accepted to approve.

  Enter a value: yes

random_pet.my-pet: Creating...
random_pet.my-pet: Creation complete after 0s [id=Mrs.firefly]

Apply complete! Resources: 1 added, 0 changed, 0 destroyed.
(base) pradeep:~$
```

```sh
(base) pradeep:~$terraform show
# local_file.pets:
resource "local_file" "pets" {
    content              = "Demo of multiple .tf files in the same directory"
    directory_permission = "0777"
    file_permission      = "0777"
    filename             = "/Users/pradeep/pets.txt"
    id                   = "2f20cf483ae1926cb52d153141858330208a61ce"
}

# random_password.password:
resource "random_password" "password" {
    bcrypt_hash = (sensitive value)
    id          = "none"
    length      = 8
    lower       = true
    min_lower   = 0
    min_numeric = 0
    min_special = 0
    min_upper   = 0
    number      = true
    numeric     = true
    result      = (sensitive value)
    special     = true
    upper       = true
}

# random_pet.my-pet:
resource "random_pet" "my-pet" {
    id        = "Mrs.firefly"
    length    = 1
    prefix    = "Mrs"
    separator = "."
}
(base) pradeep:~$
```
Other than password and pets, there are few more random resources can be created using the `random` provider.

![TF-Random]({{ site.url }}{{ site.baseurl }}/assets/images/tf-random.png)