---
layout: single
title:  "Customize Terraform Configuration with Variables"
date:   2022-09-11 07:58:04 +0530
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
# Terraform Variables

Input variables make your Terraform configuration more flexible by defining values that your end users can assign to customize the configuration. They provide a consistent interface to change how a given configuration behaves.

Unlike variables found in programming languages, Terraform's input variables don't change values during a Terraform run such as plan, apply, or destroy. Instead, they allow users to more safely customize their infrastructure by assigning different values to the variables before execution begins, rather than editing configuration files manually.

## Parameterize your configuration

Variable declarations can appear anywhere in your configuration files. However, we recommend putting them into a separate file called `variables.tf` to make it easier for users to understand how the configuration is meant to be customized.

To parameterize an argument with an input variable, you will first define the variable in `variables.tf`, then replace the hardcoded value with a reference to that variable in your configuration.

Variable blocks have three optional arguments.

- Description: A short description to document the purpose of the variable.
- Type: The type of data contained in the variable.
- Default: The default value.

If you do not set a default value for a variable, you must assign a value before Terraform can apply the configuration. Terraform does not support unassigned variables.

```sh
(base) pradeep:~$cat variables.tf 
variable "vpc_cidr_block" {
  description = "CIDR block for VPC"
  type        = string
  default     = "10.0.0.0/16"
}


(base) pradeep:~$
```

You can refer to variables in your configuration with `var.<variable_name>`.

## Terraform Console
The Terraform console command opens an interactive console that you can use to evaluate expressions in the context of your configuration. This can be very useful when working with and troubleshooting variable definitions.

Open a console with the `terraform console` command.

```sh
(base) pradeep:~$cat variables.tf 
variable "vpc_cidr_block" {
  description = "CIDR block for VPC"
  type        = string
  default     = "10.0.0.0/16"
}


(base) pradeep:~$
(base) pradeep:~$terraform console
> var.vpc_cidr_block
(known after apply)
> exit
(base) pradeep:~$
```

Terraform supports several variable types in addition to string.

Use a number type to define the number of instances supported by this configuration. Add the following to `variables.tf`.

```sh
(base) pradeep:~$cat variables.tf 
variable "vpc_cidr_block" {
  description = "CIDR block for VPC"
  type        = string
  default     = "10.0.0.0/16"
}

variable "instance_count" {
  description = "Number of instances to provision."
  type        = number
  default     = 2
}




(base) pradeep:~$
```

In addition to strings and numbers, Terraform supports several other variable types. A variable with type bool represents true/false values.

Use a `bool` type variable to control whether your VPC is configured with a VPN gateway. Add the following to `variables.tf`.

```sh
(base) pradeep:~$cat  variables.tf 
variable "vpc_cidr_block" {
  description = "CIDR block for VPC"
  type        = string
  default     = "10.0.0.0/16"
}

variable "instance_count" {
  description = "Number of instances to provision."
  type        = number
  default     = 2
}

variable "enable_vpn_gateway" {
  description = "Enable a VPN gateway in your VPC."
  type        = bool
  default     = false
}




(base) pradeep:~$
```

The variables you have used so far have all been single values. Terraform calls these types of variables `simple`. Terraform also supports `collection` variable types that contain more than one value. Terraform supports several collection variable types.

- List: A sequence of values of the same type.
- Map: A lookup table, matching keys to values, all of the same type.
- Set: An unordered collection of unique values, all of the same type.

```sh
(base) pradeep:~$cat variables.tf 
variable "vpc_cidr_block" {
  description = "CIDR block for VPC"
  type        = string
  default     = "10.0.0.0/16"
}

variable "instance_count" {
  description = "Number of instances to provision."
  type        = number
  default     = 2
}

variable "enable_vpn_gateway" {
  description = "Enable a VPN gateway in your VPC."
  type        = bool
  default     = false
}

variable "public_subnet_count" {
  description = "Number of public subnets."
  type        = number
  default     = 2
}

variable "private_subnet_count" {
  description = "Number of private subnets."
  type        = number
  default     = 2
}

variable "public_subnet_cidr_blocks" {
  description = "Available cidr blocks for public subnets."
  type        = list(string)
  default     = [
    "10.0.1.0/24",
    "10.0.2.0/24",
    "10.0.3.0/24",
    "10.0.4.0/24",
    "10.0.5.0/24",
    "10.0.6.0/24",
    "10.0.7.0/24",
    "10.0.8.0/24",
  ]
}

variable "private_subnet_cidr_blocks" {
  description = "Available cidr blocks for private subnets."
  type        = list(string)
  default     = [
    "10.0.101.0/24",
    "10.0.102.0/24",
    "10.0.103.0/24",
    "10.0.104.0/24",
    "10.0.105.0/24",
    "10.0.106.0/24",
    "10.0.107.0/24",
    "10.0.108.0/24",
  ]
}




(base) pradeep:~$
```


Declare a new `map` variable for resource tags in variables.tf.

```sh
(base) pradeep:~$cat variables.tf 
variable "vpc_cidr_block" {
  description = "CIDR block for VPC"
  type        = string
  default     = "10.0.0.0/16"
}

variable "instance_count" {
  description = "Number of instances to provision."
  type        = number
  default     = 2
}

variable "enable_vpn_gateway" {
  description = "Enable a VPN gateway in your VPC."
  type        = bool
  default     = false
}

variable "public_subnet_count" {
  description = "Number of public subnets."
  type        = number
  default     = 2
}

variable "private_subnet_count" {
  description = "Number of private subnets."
  type        = number
  default     = 2
}

variable "public_subnet_cidr_blocks" {
  description = "Available cidr blocks for public subnets."
  type        = list(string)
  default     = [
    "10.0.1.0/24",
    "10.0.2.0/24",
    "10.0.3.0/24",
    "10.0.4.0/24",
    "10.0.5.0/24",
    "10.0.6.0/24",
    "10.0.7.0/24",
    "10.0.8.0/24",
  ]
}

variable "private_subnet_cidr_blocks" {
  description = "Available cidr blocks for private subnets."
  type        = list(string)
  default     = [
    "10.0.101.0/24",
    "10.0.102.0/24",
    "10.0.103.0/24",
    "10.0.104.0/24",
    "10.0.105.0/24",
    "10.0.106.0/24",
    "10.0.107.0/24",
    "10.0.108.0/24",
  ]
}


variable "resource_tags" {
  description = "Tags to set for all resources"
  type        = map(string)
  default     = {
    project     = "project-alpha",
    environment = "dev"
  }
}



(base) pradeep:~$
```

Retrieve the value of the environment key from the resource_tags map.
```sh
> var.resource_tags["environment"]
"dev"
```

Lists and maps are `collection` types. Terraform also supports two `structural` types. Structural types have a fixed number of values that can be of different types.

- Tuple: A fixed-length sequence of values of specified types.
- Object: A lookup table, matching a fixed set of keys to values of specified types.


Assign values to variables

Terraform requires that every variable be assigned a value. Terraform supports several ways to assign variable values.

- Assign values when prompted
- Assign values with a terraform.tfvars file

Terraform automatically loads all files in the current directory with the exact name terraform.tfvars or matching *.auto.tfvars. You can also use the -var-file flag to specify other files by name.



