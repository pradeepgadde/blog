--- 
layout: single
title:  "Juniper Apstra Terraform Provider Versions"
categories: Automation
tags: Terraform
toc: true
toc_sticky: true
show_date: true
header:
  teaser: /assets/images/apstra.png
author:
  name     : "Juniper Apstra Terraform"
  avatar   : "/assets/images/apstra.png"

sidebar:
  - title: "Topics"
    nav: my-sidebar
---

# Juniper Apstra Terraform Provider Versions

Terraform providers manage resources by communicating between Terraform and target APIs. Whenever the target APIs change (for example when Juniper releases a new version of Apstra) or add functionality, provider maintainers may update and version the provider. 

Juniper Apstra Version and API details:

![tf_apstra_9]({{ site.url }}{{ site.baseurl }}/assets/images/tf_apstra_9.png) 

As on November 9, 2023 the latest version of Juniper Apstra Terraform Provider is `0.42.0`.

![tf_apstra_5]({{ site.url }}{{ site.baseurl }}/assets/images/tf_apstra_5.png)

 

When multiple users or automation tools run the same Terraform configuration, they should all use the same versions of their required providers. There are two ways for you to manage provider versions in your configuration.

1. Specify provider version constraints in your configuration's `terraform` block.
2. Use the [dependency lock file](https://developer.hashicorp.com/terraform/language/files/dependency-lock)

If you do not scope provider version appropriately, Terraform will download the latest provider version that fulfills the version constraint. This may lead to unexpected infrastructure changes. By specifying carefully scoped provider versions and using the dependency lock file, you can ensure Terraform is using the correct provider version so your configuration is applied consistently.

## Review Configuration

This directory is a pre-initialized Terraform project with three files: `main.tf`, `terraform.tf`, and `.terraform.lock.hcl`.  Juniper  has released a newer version of the Apstra provider since this workspace was first initialized.

### Explore `main.tf`

Open the `main.tf` file. This file uses the Apstra provider to deploy an ASN pool named `RFC5398 ASNs`.

```
pradeep@juniper apstra-terraform-2 % cat main.tf     
provider "apstra" {
  url = "https://admin:password@10.210.40.194:443"
  tls_validation_disabled = true
  blueprint_mutex_enabled = true
}

resource "apstra_asn_pool" "rfc5398" {
  name = "RFC5398 ASNs"
  ranges = [
    {
      first = 64496
      last  = 64511
    },
    {
      first = 65536
      last  = 65551
    },
  ]
}

pradeep@juniper apstra-terraform-2 % 
```

### Explore `terraform.tf`

Open the `terraform.tf` file. Here you will find the `terraform` block which specifies the required provider version `0.28.0` and required Terraform version `>= 1.1` for this configuration.

```
pradeep@juniper apstra-terraform-2 % cat terraform.tf

terraform {
  required_providers {
    apstra = {
      source  = "Juniper/apstra"
      version = "0.28.0"
    }
  }
  required_version = ">= 1.1"
}
pradeep@juniper apstra-terraform-2 % 
```

The `terraform` block contains the `required_providers` block, which specifies the provider local name, the [source address](https://developer.hashicorp.com/terraform/language/providers/requirements#source-addresses), and the version.

When you initialize this configuration, Terraform will download:

1. Version `0.28.0` of the Juniper/apstra provider.

The Terraform block also specifies that only Terraform binaries newer than v1.1.x can run this configuration by using the `>=` operator as well.

```
pradeep@juniper apstra-terraform-2 % terraform init

Initializing the backend...

Initializing provider plugins...
- Finding juniper/apstra versions matching "0.28.0"...
- Installing juniper/apstra v0.28.0...
- Installed juniper/apstra v0.28.0 (signed by a HashiCorp partner, key ID CB9C922903A66F3F)

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
pradeep@juniper apstra-terraform-2 %
```

### Explore `terraform.lock.hcl`

When you initialize a Terraform configuration for the first time with Terraform 1.1 or later, Terraform will generate a new `.terraform.lock.hcl` file in the current working directory. You should include the lock file in your version control repository to ensure that Terraform uses the same provider versions across your team and in ephemeral remote execution environments.

```
pradeep@juniper apstra-terraform-2 % ls -la
total 24
drwxr-xr-x    6 pradga  staff   192 Nov  9 22:11 .
drwx------+ 105 pradga  staff  3360 Nov  9 22:11 ..
drwxr-xr-x    3 pradga  staff    96 Nov  9 22:11 .terraform
-rw-r--r--    1 pradga  staff   957 Nov  9 22:11 .terraform.lock.hcl
-rw-r--r--@   1 pradga  staff   337 Nov  9 22:08 main.tf
-rw-r--r--    1 pradga  staff   151 Nov  9 22:10 terraform.tf
pradeep@juniper apstra-terraform-2 % 
```
Open the `.terraform.lock.hcl` file.

```
pradeep@juniper apstra-terraform-2 % cat .terraform.lock.hcl 
# This file is maintained automatically by "terraform init".
# Manual edits may be lost in future updates.

provider "registry.terraform.io/juniper/apstra" {
  version     = "0.28.0"
  constraints = "0.28.0"
  hashes = [
    "h1:qtZZRvvYEFNewp7IGJaDPRo+JKwtwJRnFc3h+JvbVio=",
    "zh:0d03fe87077f77ff3bd3bf2fd69f907a67a8dbe02e9022ef42edf398c7f887b2",
    "zh:4274534941a6efd250040990a536306aca2de2776b36f2eed52c74704fd1066b",
    "zh:4b08a7f1f2fbe981a73ce3dd0793ae8f875b2cd0486ef98a6cbcfc1df5b749e1",
    "zh:5dcfef9dc052e9e66681c8c28e886626625b3567dd70264a737febe80de9492c",
    "zh:7a1782e5da90dbff22adfdb1be572a647ef2205432eee6125b4c702d64416b4d",
    "zh:a5098a8bdd303b05f92d5cbe7262ba8ed516a705d00fb01aaa839eaf3dd67e7e",
    "zh:c6e1445f6d08efcfd5224fb9006661fd2a4981a89a28560c1aba520b8e760c87",
    "zh:c786ad130bb2a89c0aef8932caec3e087af16f2ac8bca21bb68295598b32520f",
    "zh:f809ab383cca0a5f83072981c64208cbd7fa67e986a86ee02dd2c82333221e32",
  ]
}
pradeep@juniper apstra-terraform-2 % 
```

## Modify Version Constraints

Modify the Provider version constraints, to a version greater than or equal to 0.28.0 with `>= 0.28.0` version constraint.

```
pradeep@juniper apstra-terraform-2 % cat terraform.tf

terraform {
  required_providers {
    apstra = {
      source  = "Juniper/apstra"
      version = ">= 0.28.0"
    }
  }
  required_version = ">= 1.1"
}
pradeep@juniper apstra-terraform-2 % 
```

Notice that instead of installing the latest version of the Apstra provider that conforms with the configured version constraints, Terraform installed the version specified in the lock file. While initializing your workspace, Terraform read the dependency lock file and downloaded the specified version of the Apstra  provider.

If Terraform did not find a lock file, it would download the latest versions of the providers that fulfill the version constraints you defined in the `required_providers` block.

```
pradeep@juniper apstra-terraform-2 % terraform init

Initializing the backend...

Initializing provider plugins...
- Reusing previous version of juniper/apstra from the dependency lock file
- Using previously-installed juniper/apstra v0.28.0

Terraform has been successfully initialized!

You may now begin working with Terraform. Try running "terraform plan" to see
any changes that are required for your infrastructure. All Terraform commands
should now work.

If you ever set or change modules or backend configuration for Terraform,
rerun this command to reinitialize your working directory. If you forget, other
commands will detect it and remind you to do so if necessary.
pradeep@juniper apstra-terraform-2 % 
```

The lock file instructs Terraform to always install the same provider version, ensuring that consistent runs across your team or remote sessions.

Apply your configuration. Respond to the confirmation prompt with a `yes` to create the example infrastructure.

```
pradeep@juniper apstra-terraform-2 % terraform apply

Terraform used the selected providers to generate the following execution plan. Resource actions are indicated with the following symbols:
  + create

Terraform will perform the following actions:

  # apstra_asn_pool.rfc5398 will be created
  + resource "apstra_asn_pool" "rfc5398" {
      + id              = (known after apply)
      + name            = "RFC5398 ASNs"
      + ranges          = [
          + {
              + first           = 64496
              + last            = 64511
              + status          = (known after apply)
              + total           = (known after apply)
              + used            = (known after apply)
              + used_percentage = (known after apply)
            },
          + {
              + first           = 65536
              + last            = 65551
              + status          = (known after apply)
              + total           = (known after apply)
              + used            = (known after apply)
              + used_percentage = (known after apply)
            },
        ]
      + status          = (known after apply)
      + total           = (known after apply)
      + used            = (known after apply)
      + used_percentage = (known after apply)
    }

Plan: 1 to add, 0 to change, 0 to destroy.

Do you want to perform these actions?
  Terraform will perform the actions described above.
  Only 'yes' will be accepted to approve.

  Enter a value: yes

apstra_asn_pool.rfc5398: Creating...
apstra_asn_pool.rfc5398: Creation complete after 1s [id=54f5abae-7fdd-425b-b0a6-f67863dfe91b]

Apply complete! Resources: 1 added, 0 changed, 0 destroyed.
pradeep@juniper apstra-terraform-2 % 
```

![tf_apstra_5]({{ site.url }}{{ site.baseurl }}/assets/images/tf_apstra_6.png) 

## Upgrade the Juniper Apstra Provider Version


The `-upgrade` flag will upgrade all providers to the latest version consistent within the version constraints specified in your configuration.

Upgrade the Apstra provider.

```
pradeep@juniper apstra-terraform-2 % terraform init -upgrade

Initializing the backend...

Initializing provider plugins...
- Finding juniper/apstra versions matching ">= 0.28.0"...
- Installing juniper/apstra v0.42.0...
- Installed juniper/apstra v0.42.0 (signed by a HashiCorp partner, key ID CB9C922903A66F3F)

Partner and community providers are signed by their developers.
If you'd like to know more about provider signing, you can read about it here:
https://www.terraform.io/docs/cli/plugins/signing.html

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
pradeep@juniper apstra-terraform-2 % 
```

Notice that Terraform installs the latest version of the Apstra provider.

Open the `.terraform.lock.hcl` file and notice that the Apstr provider's version is now the latest version.

```
pradeep@juniper apstra-terraform-2 % cat .terraform.lock.hcl 
# This file is maintained automatically by "terraform init".
# Manual edits may be lost in future updates.

provider "registry.terraform.io/juniper/apstra" {
  version     = "0.42.0"
  constraints = ">= 0.28.0"
  hashes = [
    "h1:X64/jd5jAxfqnst3cXuS5Jl1SpaazltHWIqVlVP6BjM=",
    "zh:3d2f288330933841ae9aac0a4e76a1c0a68f4a8a67a4acfc8d15b8d6983eca61",
    "zh:4162df5131075a2151722d68697a48c8a56c9ed5e9c6ac025f7fd76174785b8a",
    "zh:487d5c347fd510f4e5c6cd1f077142a65f3fb4356db02979ac8fcbf25590efab",
    "zh:4c82acd309e85e675348189b669e88a54af6b54509d0ba5ab5645dd4d9f588ac",
    "zh:7cf4a62b2d4e294fac58ac7078cf0f7747c1640940c0afc1f9f36cda7003c488",
    "zh:a12c509ba54c735ab975933b8215fbc72fa95b4d8e823dc52c348a3892595072",
    "zh:db90a961eeebb3ddbcc673d5f2236f5e7f7f945513bf6286c9f832c2e048583d",
    "zh:df56f7293e9494867b82a11a5c47151d07b9acd708c6aeb30f74dc9a875fa446",
    "zh:f809ab383cca0a5f83072981c64208cbd7fa67e986a86ee02dd2c82333221e32",
  ]
}
pradeep@juniper apstra-terraform-2 % 
```

Apply your configuration with the new provider version installed 

```
pradeep@juniper apstra-terraform-2 % terraform apply
apstra_asn_pool.rfc5398: Refreshing state... [id=54f5abae-7fdd-425b-b0a6-f67863dfe91b]

No changes. Your infrastructure matches the configuration.

Terraform has compared your real infrastructure against your configuration and found no differences, so no changes are needed.

Apply complete! Resources: 0 added, 0 changed, 0 destroyed.
pradeep@juniper apstra-terraform-2 % 
```

If the apply step completes successfully, it is safe to commit the configuration with the updated lock file to version control. If the plan or apply steps fail, do **not** commit the lock file to version control.

## Destroy

After verifying that the resources were deployed successfully, destroy them. Remember to respond to the confirmation prompt with `yes`.

```
pradeep@juniper apstra-terraform-2 % terraform destroy
apstra_asn_pool.rfc5398: Refreshing state... [id=54f5abae-7fdd-425b-b0a6-f67863dfe91b]

Terraform used the selected providers to generate the following execution plan. Resource actions are indicated with the following symbols:
  - destroy

Terraform will perform the following actions:

  # apstra_asn_pool.rfc5398 will be destroyed
  - resource "apstra_asn_pool" "rfc5398" {
      - id              = "54f5abae-7fdd-425b-b0a6-f67863dfe91b" -> null
      - name            = "RFC5398 ASNs" -> null
      - ranges          = [
          - {
              - first           = 64496 -> null
              - last            = 64511 -> null
              - status          = "pool_element_available" -> null
              - total           = 16 -> null
              - used            = 0 -> null
              - used_percentage = 0 -> null
            },
          - {
              - first           = 65536 -> null
              - last            = 65551 -> null
              - status          = "pool_element_available" -> null
              - total           = 16 -> null
              - used            = 0 -> null
              - used_percentage = 0 -> null
            },
        ] -> null
      - status          = "not_in_use" -> null
      - total           = 32 -> null
      - used            = 0 -> null
      - used_percentage = 0 -> null
    }

Plan: 0 to add, 0 to change, 1 to destroy.

Do you really want to destroy all resources?
  Terraform will destroy all your managed infrastructure, as shown above.
  There is no undo. Only 'yes' will be accepted to confirm.

  Enter a value: yes

apstra_asn_pool.rfc5398: Destroying... [id=54f5abae-7fdd-425b-b0a6-f67863dfe91b]
apstra_asn_pool.rfc5398: Destruction complete after 1s

Destroy complete! Resources: 1 destroyed.
pradeep@juniper apstra-terraform-2 % 
```

In this post, you used the dependency lock file to manage provider versions, and upgraded the lock file.

