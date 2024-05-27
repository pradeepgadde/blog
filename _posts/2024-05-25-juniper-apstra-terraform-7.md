---
layout: single
title:  "Juniper Apstra Terraform Provider — Input Variables"
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

# Juniper Apstra Terraform Provider — Input Variables

You now have enough Terraform knowledge to create useful configurations, but the examples so far have used hard-coded values. Terraform configurations can include variables to make your configuration more dynamic and flexible.

Let’s verify our current Terraform configuration

```
pradeep@juniper apstra-terraform-4 % cat main.tf 
terraform {
  required_providers {
    apstra = {
      source  = "Juniper/apstra"
      version = ">= 0.28.0"
    }
  }
  required_version = ">= 1.1"
}
provider "apstra" {
  url = "https://admin:password@10.210.40.194:443"
  tls_validation_disabled = true
  blueprint_mutex_enabled = true
}

# This example creates an IPv4 Resource Pool with three allocations inside:
# - 3 blocks from RFC5737
#
# After creating the pool, it outputs the total number of
# IP addresses in the pool.

resource "apstra_ipv4_pool" "example" {
  name = "RFC 5737 ranges"
  subnets = [
    { network = "192.0.2.0/24"},
    { network = "198.51.100.0/24"},
    { network = "203.0.113.0/24"},
  ]
}


pradeep@juniper apstra-terraform-4 % 
```

## Set the IPv4 Pool name with a variable

The current configuration includes a number of hard-coded values. Terraform variables allow you to write configuration that is flexible and easier to re-use.

Add a variable to define the IPv4 pool name.

Variable declarations can appear anywhere in your configuration files. However, Terraform recommendation is to put them into a separate file called `variables.tf` to make it easier for users to understand how they can customize the configuration.

To parameterize an argument with an input variable, you must first define the variable, then replace the hardcoded value with a reference to that variable in your configuration.

Create a new file called `variables.tf` with a block defining a new `ipv4_pool_name` variable.

```
pradeep@juniper apstra-terraform-4 % cat variables.tf 
variable "ipv4_pool_name" {
  description = "Value of the name for the IPv4 Pool"
  type        = string
  default     = "ExampleIPv4Pool"
}

pradeep@juniper apstra-terraform-4 % 
```

Variable blocks have three optional arguments.

- **Description**: A short description to document the purpose of the variable.
- **Type**: The type of data contained in the variable.
- **Default**: The default value.

Terraform loads all files in the current directory ending in `.tf`, so you can name your configuration files however you choose.

In `main.tf`, update the `apsta_ipv4_pool` resource block to use the new variable (with the `var.<variable_name>` notation). The `ipv4_pool_name` variable block will default to its default value (`ExampleIPv4Pool`) unless you declare a different value.

```
pradeep@juniper apstra-terraform-4 % cat main.tf 
terraform {
  required_providers {
    apstra = {
      source  = "Juniper/apstra"
      version = ">= 0.28.0"
    }
  }
  required_version = ">= 1.1"
}
provider "apstra" {
  url = "https://admin:password@10.210.40.194:443"
  tls_validation_disabled = true
  blueprint_mutex_enabled = true
}

# This example creates an IPv4 Resource Pool with three allocations inside:
# - 3 blocks from RFC5737
#
# After creating the pool, it outputs the total number of
# IP addresses in the pool.

resource "apstra_ipv4_pool" "example" {
  name = var.ipv4_pool_name
  subnets = [
    { network = "192.0.2.0/24"},
    { network = "198.51.100.0/24"},
    { network = "203.0.113.0/24"},
  ]
}


pradeep@juniper apstra-terraform-4 % 
```

## Apply your configuration

Apply the configuration. Respond to the confirmation prompt with a `yes`.

```
pradeep@juniper apstra-terraform-4 % terraform apply
apstra_ipv4_pool.example: Refreshing state... [id=b9261e73-401a-4497-b201-93eeb3b2c184]

Terraform used the selected providers to generate the following execution plan. Resource actions are indicated with the following symbols:
  ~ update in-place

Terraform will perform the following actions:

  # apstra_ipv4_pool.example will be updated in-place
  ~ resource "apstra_ipv4_pool" "example" {
        id              = "b9261e73-401a-4497-b201-93eeb3b2c184"
      ~ name            = "RFC 5737 ranges" -> "ExampleIPv4Pool"
      ~ status          = "not_in_use" -> (known after apply)
      ~ total           = 768 -> (known after apply)
      ~ used            = 0 -> (known after apply)
      ~ used_percentage = 0 -> (known after apply)
        # (1 unchanged attribute hidden)
    }

Plan: 0 to add, 1 to change, 0 to destroy.

Do you want to perform these actions?
  Terraform will perform the actions described above.
  Only 'yes' will be accepted to approve.

  Enter a value: yes

apstra_ipv4_pool.example: Modifying... [id=b9261e73-401a-4497-b201-93eeb3b2c184]
apstra_ipv4_pool.example: Modifications complete after 1s [id=b9261e73-401a-4497-b201-93eeb3b2c184]

Apply complete! Resources: 0 added, 1 changed, 0 destroyed.
pradeep@juniper apstra-terraform-4 % 
```

In the Apstra Web UI, verify that the pool name is changed to `ExampleIPv4Pool`.

![tf_apstra_25]({{ site.url }}{{ site.baseurl }}/assets/images/tf_apstra_25.png) 

## Override Defaults

Now apply the configuration again, this time overriding the default IPv4 pool name by passing in a variable using the `-var` flag. Terraform will update the pool's `name` attribute with the new name. Respond to the confirmation prompt with `yes`.

```
pradeep@juniper apstra-terraform-4 % terraform apply -var "ipv4_pool_name=AnotherExamplePool"
apstra_ipv4_pool.example: Refreshing state... [id=b9261e73-401a-4497-b201-93eeb3b2c184]

Terraform used the selected providers to generate the following execution plan. Resource actions are indicated with the following symbols:
  ~ update in-place

Terraform will perform the following actions:

  # apstra_ipv4_pool.example will be updated in-place
  ~ resource "apstra_ipv4_pool" "example" {
        id              = "b9261e73-401a-4497-b201-93eeb3b2c184"
      ~ name            = "ExampleIPv4Pool" -> "AnotherExamplePool"
      ~ status          = "not_in_use" -> (known after apply)
      ~ total           = 768 -> (known after apply)
      ~ used            = 0 -> (known after apply)
      ~ used_percentage = 0 -> (known after apply)
        # (1 unchanged attribute hidden)
    }

Plan: 0 to add, 1 to change, 0 to destroy.

Do you want to perform these actions?
  Terraform will perform the actions described above.
  Only 'yes' will be accepted to approve.

  Enter a value: yes

apstra_ipv4_pool.example: Modifying... [id=b9261e73-401a-4497-b201-93eeb3b2c184]
apstra_ipv4_pool.example: Modifications complete after 1s [id=b9261e73-401a-4497-b201-93eeb3b2c184]

Apply complete! Resources: 0 added, 1 changed, 0 destroyed.
pradeep@juniper apstra-terraform-4 % 
```

Verify that the pool name is changed to `AnotherExamplePool`.

![tf_apstra_26]({{ site.url }}{{ site.baseurl }}/assets/images/tf_apstra_26.png) 

Setting variables via the command-line will not save their values. Terraform supports many ways to use and set variables so you can avoid having to enter them repeatedly as you execute commands.

If you try to plan/apply without passing command-line variables, Terraform uses the default value.

```
pradeep@juniper apstra-terraform-4 % terraform plan
apstra_ipv4_pool.example: Refreshing state... [id=b9261e73-401a-4497-b201-93eeb3b2c184]

Terraform used the selected providers to generate the following execution plan. Resource actions are indicated with the following symbols:
  ~ update in-place

Terraform will perform the following actions:

  # apstra_ipv4_pool.example will be updated in-place
  ~ resource "apstra_ipv4_pool" "example" {
        id              = "b9261e73-401a-4497-b201-93eeb3b2c184"
      ~ name            = "AnotherExamplePool" -> "ExampleIPv4Pool"
      ~ status          = "not_in_use" -> (known after apply)
      ~ total           = 768 -> (known after apply)
      ~ used            = 0 -> (known after apply)
      ~ used_percentage = 0 -> (known after apply)
        # (1 unchanged attribute hidden)
    }

Plan: 0 to add, 1 to change, 0 to destroy.

──────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────

Note: You didn't use the -out option to save this plan, so Terraform can't guarantee to take exactly these actions if you run "terraform apply" now.
pradeep@juniper apstra-terraform-4 % 
```

## What happens if we don't define a default value for a variable?

Let's see. 

Modify the `variables.tf` definition to remove the `default` value.

```
pradeep@juniper apstra-terraform-4 % cat variables.tf 
variable "ipv4_pool_name" {
  description = "Value of the name for the IPv4 Pool"
  type        = string
}

pradeep@juniper apstra-terraform-4 %
```

Verify the plan. Since there is no default value and no variable value passed from command line, system prompts you yo enter a value for the variable. Once you enter a value, it proceeds further.

```
pradeep@juniper apstra-terraform-4 % terraform plan 
var.ipv4_pool_name
  Value of the name for the IPv4 Pool

  Enter a value: DemoIPv4Pool

apstra_ipv4_pool.example: Refreshing state... [id=b9261e73-401a-4497-b201-93eeb3b2c184]

Terraform used the selected providers to generate the following execution plan. Resource actions are indicated with the following symbols:
  ~ update in-place

Terraform will perform the following actions:

  # apstra_ipv4_pool.example will be updated in-place
  ~ resource "apstra_ipv4_pool" "example" {
        id              = "b9261e73-401a-4497-b201-93eeb3b2c184"
      ~ name            = "AnotherExamplePool" -> "DemoIPv4Pool"
      ~ status          = "not_in_use" -> (known after apply)
      ~ total           = 768 -> (known after apply)
      ~ used            = 0 -> (known after apply)
      ~ used_percentage = 0 -> (known after apply)
        # (1 unchanged attribute hidden)
    }

Plan: 0 to add, 1 to change, 0 to destroy.

──────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────

Note: You didn't use the -out option to save this plan, so Terraform can't guarantee to take exactly these actions if you run "terraform apply" now.
pradeep@juniper apstra-terraform-4 % 
```

## Assign Values with a File

Entering variable values manually is time consuming and error prone. Instead, you can capture variable values in a file.

Create a file named `terraform.tfvars` with the following contents.

```
pradeep@juniper apstra-terraform-4 % cat terraform.tfvars 
ipv4_pool_name = "TestPool"
pradeep@juniper apstra-terraform-4 % 
```

Terraform automatically loads all files in the current directory with the exact name `terraform.tfvars` or matching `*.auto.tfvars`. You can also use the `-var-file` flag to specify other files by name.

These files use syntax similar to Terraform configuration files (HCL), but they cannot contain configuration such as resource definitions. Like Terraform configuration files, these files can also contain JSON.

Apply the configuration with these new values. Respond to the confirmation prompt with `yes` to apply these changes.

```
pradeep@juniper apstra-terraform-4 % terraform apply
apstra_ipv4_pool.example: Refreshing state... [id=b9261e73-401a-4497-b201-93eeb3b2c184]

Terraform used the selected providers to generate the following execution plan. Resource actions are indicated with the following symbols:
  ~ update in-place

Terraform will perform the following actions:

  # apstra_ipv4_pool.example will be updated in-place
  ~ resource "apstra_ipv4_pool" "example" {
        id              = "b9261e73-401a-4497-b201-93eeb3b2c184"
      ~ name            = "AnotherExamplePool" -> "TestPool"
      ~ status          = "not_in_use" -> (known after apply)
      ~ total           = 768 -> (known after apply)
      ~ used            = 0 -> (known after apply)
      ~ used_percentage = 0 -> (known after apply)
        # (1 unchanged attribute hidden)
    }

Plan: 0 to add, 1 to change, 0 to destroy.

Do you want to perform these actions?
  Terraform will perform the actions described above.
  Only 'yes' will be accepted to approve.

  Enter a value: yes

apstra_ipv4_pool.example: Modifying... [id=b9261e73-401a-4497-b201-93eeb3b2c184]
apstra_ipv4_pool.example: Modifications complete after 1s [id=b9261e73-401a-4497-b201-93eeb3b2c184]

Apply complete! Resources: 0 added, 1 changed, 0 destroyed.
pradeep@juniper apstra-terraform-4 %
```

Notice that, though there is no default value for the pool name, it didn't prompt, because there is `terraform.tfvars` file in the current directory and it has a value for the variable.

In the Apstra Web UI, check the pool name. It should change to `TestPool`.

![tf_apstra_27]({{ site.url }}{{ site.baseurl }}/assets/images/tf_apstra_27.png) 

In addition to command line flags and variable files, you can use environment variables to set input variables. 

## Variable Definition Precedence

The above mechanisms for setting variables can be used together in any combination. If the same variable is assigned multiple values, Terraform uses the *last* value it finds, overriding any previous values. Note that the same variable cannot be assigned multiple values within a single source.

Terraform loads variables in the following order, with later sources taking precedence over earlier ones:

- Environment variables
- The `terraform.tfvars` file, if present.
- The `terraform.tfvars.json` file, if present.
- Any `*.auto.tfvars` or `*.auto.tfvars.json` files, processed in lexical order of their filenames.
- Any `-var` and `-var-file` options on the command line, in the order they are provided. (This includes variables set by a Terraform Cloud workspace.)

Let’s test this, now. We have the `terraform.tfvars` file and we are passing `-var` option on the command line.

```
pradeep@juniper apstra-terraform-4 % terraform apply -var ipv4_pool_name=SampleIPv4Pool
apstra_ipv4_pool.example: Refreshing state... [id=b9261e73-401a-4497-b201-93eeb3b2c184]

Terraform used the selected providers to generate the following execution plan. Resource actions are indicated with the following symbols:
  ~ update in-place

Terraform will perform the following actions:

  # apstra_ipv4_pool.example will be updated in-place
  ~ resource "apstra_ipv4_pool" "example" {
        id              = "b9261e73-401a-4497-b201-93eeb3b2c184"
      ~ name            = "TestPool" -> "SampleIPv4Pool"
      ~ status          = "not_in_use" -> (known after apply)
      ~ total           = 768 -> (known after apply)
      ~ used            = 0 -> (known after apply)
      ~ used_percentage = 0 -> (known after apply)
        # (1 unchanged attribute hidden)
    }

Plan: 0 to add, 1 to change, 0 to destroy.

Do you want to perform these actions?
  Terraform will perform the actions described above.
  Only 'yes' will be accepted to approve.

  Enter a value: yes

apstra_ipv4_pool.example: Modifying... [id=b9261e73-401a-4497-b201-93eeb3b2c184]
apstra_ipv4_pool.example: Modifications complete after 1s [id=b9261e73-401a-4497-b201-93eeb3b2c184]

Apply complete! Resources: 0 added, 1 changed, 0 destroyed.
pradeep@juniper apstra-terraform-4 % 
```

We can see that the later option (the command line) took precedence over the earlier one ( .tfvars file).

![tf_apstra_28]({{ site.url }}{{ site.baseurl }}/assets/images/tf_apstra_28.png) 



