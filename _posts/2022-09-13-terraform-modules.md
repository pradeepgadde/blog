---
layout: single
title:  "Terraform Modules"
date:   2022-09-13 01:58:04 +0530
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
# Terraform Modules 

A Terraform module is a set of Terraform configuration files in a single directory. Even a simple configuration consisting of a single directory with one or more `.tf` files is a module. When you run Terraform commands directly from such a directory, it is considered the **root module**. So in this sense, every Terraform configuration is part of a module. 

Terraform commands will only directly use the configuration files in one directory, which is usually the current working directory. However, your configuration can use module blocks to call modules in other directories. When Terraform encounters a module block, it loads and processes that module's configuration files.

A module that is called by another configuration is sometimes referred to as a "child module" of that configuration.



```sh
root@terraform:~/learn-terraform-modules# ls
LICENSE  README.md  docker-compose.yml  main.tf  outputs.tf  terraform.tfstate  variables.tf
root@terraform:~/learn-terraform-modules#
```
## Root Module
```
root@terraform:~/learn-terraform-modules# cat main.tf 
# Terraform configuration

provider "aws" {
  region = "us-west-2"
  
  ## v Everything between the comments is localstack specific v
  access_key                  = "anaccesskey"
  secret_key                  = "asecretkey"
  s3_force_path_style         = true
  skip_credentials_validation = true
  skip_metadata_api_check     = true
  skip_requesting_account_id  = true

  endpoints {
    ec2 = "http://localhost:4566"
  }
  ## ^ Everything between the comments is localstack specific ^
}

module "vpc" {
  source  = "terraform-aws-modules/vpc/aws"
  version = "2.21.0"

  name = var.vpc_name
  cidr = var.vpc_cidr

  azs             = var.vpc_azs
  private_subnets = var.vpc_private_subnets
  public_subnets  = var.vpc_public_subnets

  enable_nat_gateway = var.vpc_enable_nat_gateway

  tags = var.vpc_tags
}

module "ec2_instances" {
  source  = "terraform-aws-modules/ec2-instance/aws"
  version = "2.12.0"

  name           = "my-ec2-cluster"
  instance_count = 2

  ami                    = "ami-0c5204531f799e0c6"
  instance_type          = "t2.micro"
  vpc_security_group_ids = [module.vpc.default_security_group_id]
  subnet_id              = module.vpc.public_subnets[0]

  tags = {
    Terraform   = "true"
    Environment = "dev"
  }
}
root@terraform:~/learn-terraform-modules# 
```
## Root Variables
```
root@terraform:~/learn-terraform-modules# cat variables.tf 
# Input variable definitions

variable "vpc_name" {
  description = "Name of VPC"
  type        = string
  default     = "example-vpc"
}

variable "vpc_cidr" {
  description = "CIDR block for VPC"
  type        = string
  default     = "10.0.0.0/16"
}

variable "vpc_azs" {
  description = "Availability zones for VPC"
  type        = list
  default     = ["us-west-2a", "us-west-2b", "us-west-2c"]
}

variable "vpc_private_subnets" {
  description = "Private subnets for VPC"
  type        = list(string)
  default     = ["10.0.1.0/24", "10.0.2.0/24"]
}

variable "vpc_public_subnets" {
  description = "Public subnets for VPC"
  type        = list(string)
  default     = ["10.0.101.0/24", "10.0.102.0/24"]
}

variable "vpc_enable_nat_gateway" {
  description = "Enable NAT gateway for VPC"
  type    = bool
  default = true
}

variable "vpc_tags" {
  description = "Tags to apply to resources created by VPC module"
  type        = map(string)
  default     = {
    Terraform   = "true"
    Environment = "dev"
  }
}
```
## Root Outputs
```
root@terraform:~/learn-terraform-modules# cat outputs.tf 
# Output variable definitions

output "vpc_public_subnets" {
  description = "IDs of the VPC's public subnets"
  value       = module.vpc.public_subnets
}

output "ec2_instance_public_ips" {
  description = "Public IP addresses of EC2 instances"
  value       = module.ec2_instances.public_ip
}
root@terraform:~/learn-terraform-modules# ls
LICENSE  README.md  docker-compose.yml  main.tf  outputs.tf  terraform.tfstate  variables.tf
root@terraform:~/learn-terraform-modules#
```



```sh
root@terraform:~/learn-terraform-modules# terraform init
Initializing modules...
Downloading terraform-aws-modules/ec2-instance/aws 2.12.0 for ec2_instances...
- ec2_instances in .terraform/modules/ec2_instances
Downloading terraform-aws-modules/vpc/aws 2.21.0 for vpc...
- vpc in .terraform/modules/vpc

Initializing the backend...

Initializing provider plugins...
- Finding latest version of hashicorp/aws...
- Installing hashicorp/aws v4.30.0...
- Installed hashicorp/aws v4.30.0 (self-signed, key ID 34365D9472D7468F)

Partner and community providers are signed by their developers.
If you'd like to know more about provider signing, you can read about it here:
https://www.terraform.io/docs/plugins/signing.html

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
root@terraform:~/learn-terraform-modules# terraform apply

An execution plan has been generated and is shown below.
Resource actions are indicated with the following symbols:
  + create

Terraform will perform the following actions:

  # module.ec2_instances.aws_instance.this[0] will be created
  + resource "aws_instance" "this" {
      + ami                                  = "ami-0c5204531f799e0c6"
      + arn                                  = (known after apply)
      + associate_public_ip_address          = (known after apply)
      + availability_zone                    = (known after apply)
      + cpu_core_count                       = (known after apply)
      + cpu_threads_per_core                 = (known after apply)
      + disable_api_stop                     = (known after apply)
      + disable_api_termination              = false
      + ebs_optimized                        = false
      + get_password_data                    = false
      + host_id                              = (known after apply)
      + host_resource_group_arn              = (known after apply)
      + id                                   = (known after apply)
      + instance_initiated_shutdown_behavior = (known after apply)
      + instance_state                       = (known after apply)
      + instance_type                        = "t2.micro"
      + ipv6_address_count                   = (known after apply)
      + ipv6_addresses                       = (known after apply)
      + key_name                             = (known after apply)
      + monitoring                           = false
      + outpost_arn                          = (known after apply)
      + password_data                        = (known after apply)
      + placement_group                      = (known after apply)
      + placement_partition_number           = (known after apply)
      + primary_network_interface_id         = (known after apply)
      + private_dns                          = (known after apply)
      + private_ip                           = (known after apply)
      + public_dns                           = (known after apply)
      + public_ip                            = (known after apply)
      + secondary_private_ips                = (known after apply)
      + security_groups                      = (known after apply)
      + source_dest_check                    = true
      + subnet_id                            = (known after apply)
      + tags                                 = {
          + "Environment" = "dev"
          + "Name"        = "my-ec2-cluster-1"
          + "Terraform"   = "true"
        }
      + tags_all                             = {
          + "Environment" = "dev"
          + "Name"        = "my-ec2-cluster-1"
          + "Terraform"   = "true"
        }
      + tenancy                              = "default"
      + user_data                            = (known after apply)
      + user_data_base64                     = (known after apply)
      + user_data_replace_on_change          = false
      + volume_tags                          = {
          + "Name" = "my-ec2-cluster-1"
        }
      + vpc_security_group_ids               = (known after apply)

      + capacity_reservation_specification {
          + capacity_reservation_preference = (known after apply)

          + capacity_reservation_target {
              + capacity_reservation_id                 = (known after apply)
              + capacity_reservation_resource_group_arn = (known after apply)
            }
        }

      + credit_specification {
          + cpu_credits = "standard"
        }

      + ebs_block_device {
          + delete_on_termination = (known after apply)
          + device_name           = (known after apply)
          + encrypted             = (known after apply)
          + iops                  = (known after apply)
          + kms_key_id            = (known after apply)
          + snapshot_id           = (known after apply)
          + tags                  = (known after apply)
          + throughput            = (known after apply)
          + volume_id             = (known after apply)
          + volume_size           = (known after apply)
          + volume_type           = (known after apply)
        }

      + enclave_options {
          + enabled = (known after apply)
        }

      + ephemeral_block_device {
          + device_name  = (known after apply)
          + no_device    = (known after apply)
          + virtual_name = (known after apply)
        }

      + maintenance_options {
          + auto_recovery = (known after apply)
        }

      + metadata_options {
          + http_endpoint               = (known after apply)
          + http_put_response_hop_limit = (known after apply)
          + http_tokens                 = (known after apply)
          + instance_metadata_tags      = (known after apply)
        }

      + network_interface {
          + delete_on_termination = (known after apply)
          + device_index          = (known after apply)
          + network_card_index    = (known after apply)
          + network_interface_id  = (known after apply)
        }

      + private_dns_name_options {
          + enable_resource_name_dns_a_record    = (known after apply)
          + enable_resource_name_dns_aaaa_record = (known after apply)
          + hostname_type                        = (known after apply)
        }

      + root_block_device {
          + delete_on_termination = (known after apply)
          + device_name           = (known after apply)
          + encrypted             = (known after apply)
          + iops                  = (known after apply)
          + kms_key_id            = (known after apply)
          + tags                  = (known after apply)
          + throughput            = (known after apply)
          + volume_id             = (known after apply)
          + volume_size           = (known after apply)
          + volume_type           = (known after apply)
        }
    }

  # module.ec2_instances.aws_instance.this[1] will be created
  + resource "aws_instance" "this" {
      + ami                                  = "ami-0c5204531f799e0c6"
      + arn                                  = (known after apply)
      + associate_public_ip_address          = (known after apply)
      + availability_zone                    = (known after apply)
      + cpu_core_count                       = (known after apply)
      + cpu_threads_per_core                 = (known after apply)
      + disable_api_stop                     = (known after apply)
      + disable_api_termination              = false
      + ebs_optimized                        = false
      + get_password_data                    = false
      + host_id                              = (known after apply)
      + host_resource_group_arn              = (known after apply)
      + id                                   = (known after apply)
      + instance_initiated_shutdown_behavior = (known after apply)
      + instance_state                       = (known after apply)
      + instance_type                        = "t2.micro"
      + ipv6_address_count                   = (known after apply)
      + ipv6_addresses                       = (known after apply)
      + key_name                             = (known after apply)
      + monitoring                           = false
      + outpost_arn                          = (known after apply)
      + password_data                        = (known after apply)
      + placement_group                      = (known after apply)
      + placement_partition_number           = (known after apply)
      + primary_network_interface_id         = (known after apply)
      + private_dns                          = (known after apply)
      + private_ip                           = (known after apply)
      + public_dns                           = (known after apply)
      + public_ip                            = (known after apply)
      + secondary_private_ips                = (known after apply)
      + security_groups                      = (known after apply)
      + source_dest_check                    = true
      + subnet_id                            = (known after apply)
      + tags                                 = {
          + "Environment" = "dev"
          + "Name"        = "my-ec2-cluster-2"
          + "Terraform"   = "true"
        }
      + tags_all                             = {
          + "Environment" = "dev"
          + "Name"        = "my-ec2-cluster-2"
          + "Terraform"   = "true"
        }
      + tenancy                              = "default"
      + user_data                            = (known after apply)
      + user_data_base64                     = (known after apply)
      + user_data_replace_on_change          = false
      + volume_tags                          = {
          + "Name" = "my-ec2-cluster-2"
        }
      + vpc_security_group_ids               = (known after apply)

      + capacity_reservation_specification {
          + capacity_reservation_preference = (known after apply)

          + capacity_reservation_target {
              + capacity_reservation_id                 = (known after apply)
              + capacity_reservation_resource_group_arn = (known after apply)
            }
        }

      + credit_specification {
          + cpu_credits = "standard"
        }

      + ebs_block_device {
          + delete_on_termination = (known after apply)
          + device_name           = (known after apply)
          + encrypted             = (known after apply)
          + iops                  = (known after apply)
          + kms_key_id            = (known after apply)
          + snapshot_id           = (known after apply)
          + tags                  = (known after apply)
          + throughput            = (known after apply)
          + volume_id             = (known after apply)
          + volume_size           = (known after apply)
          + volume_type           = (known after apply)
        }

      + enclave_options {
          + enabled = (known after apply)
        }

      + ephemeral_block_device {
          + device_name  = (known after apply)
          + no_device    = (known after apply)
          + virtual_name = (known after apply)
        }

      + maintenance_options {
          + auto_recovery = (known after apply)
        }

      + metadata_options {
          + http_endpoint               = (known after apply)
          + http_put_response_hop_limit = (known after apply)
          + http_tokens                 = (known after apply)
          + instance_metadata_tags      = (known after apply)
        }

      + network_interface {
          + delete_on_termination = (known after apply)
          + device_index          = (known after apply)
          + network_card_index    = (known after apply)
          + network_interface_id  = (known after apply)
        }

      + private_dns_name_options {
          + enable_resource_name_dns_a_record    = (known after apply)
          + enable_resource_name_dns_aaaa_record = (known after apply)
          + hostname_type                        = (known after apply)
        }

      + root_block_device {
          + delete_on_termination = (known after apply)
          + device_name           = (known after apply)
          + encrypted             = (known after apply)
          + iops                  = (known after apply)
          + kms_key_id            = (known after apply)
          + tags                  = (known after apply)
          + throughput            = (known after apply)
          + volume_id             = (known after apply)
          + volume_size           = (known after apply)
          + volume_type           = (known after apply)
        }
    }

  # module.vpc.aws_eip.nat[0] will be created
  + resource "aws_eip" "nat" {
      + allocation_id        = (known after apply)
      + association_id       = (known after apply)
      + carrier_ip           = (known after apply)
      + customer_owned_ip    = (known after apply)
      + domain               = (known after apply)
      + id                   = (known after apply)
      + instance             = (known after apply)
      + network_border_group = (known after apply)
      + network_interface    = (known after apply)
      + private_dns          = (known after apply)
      + private_ip           = (known after apply)
      + public_dns           = (known after apply)
      + public_ip            = (known after apply)
      + public_ipv4_pool     = (known after apply)
      + tags                 = {
          + "Environment" = "dev"
          + "Name"        = "example-vpc-us-west-2a"
          + "Terraform"   = "true"
        }
      + tags_all             = {
          + "Environment" = "dev"
          + "Name"        = "example-vpc-us-west-2a"
          + "Terraform"   = "true"
        }
      + vpc                  = true
    }

  # module.vpc.aws_eip.nat[1] will be created
  + resource "aws_eip" "nat" {
      + allocation_id        = (known after apply)
      + association_id       = (known after apply)
      + carrier_ip           = (known after apply)
      + customer_owned_ip    = (known after apply)
      + domain               = (known after apply)
      + id                   = (known after apply)
      + instance             = (known after apply)
      + network_border_group = (known after apply)
      + network_interface    = (known after apply)
      + private_dns          = (known after apply)
      + private_ip           = (known after apply)
      + public_dns           = (known after apply)
      + public_ip            = (known after apply)
      + public_ipv4_pool     = (known after apply)
      + tags                 = {
          + "Environment" = "dev"
          + "Name"        = "example-vpc-us-west-2b"
          + "Terraform"   = "true"
        }
      + tags_all             = {
          + "Environment" = "dev"
          + "Name"        = "example-vpc-us-west-2b"
          + "Terraform"   = "true"
        }
      + vpc                  = true
    }

  # module.vpc.aws_internet_gateway.this[0] will be created
  + resource "aws_internet_gateway" "this" {
      + arn      = (known after apply)
      + id       = (known after apply)
      + owner_id = (known after apply)
      + tags     = {
          + "Environment" = "dev"
          + "Name"        = "example-vpc"
          + "Terraform"   = "true"
        }
      + tags_all = {
          + "Environment" = "dev"
          + "Name"        = "example-vpc"
          + "Terraform"   = "true"
        }
      + vpc_id   = (known after apply)
    }

  # module.vpc.aws_nat_gateway.this[0] will be created
  + resource "aws_nat_gateway" "this" {
      + allocation_id        = (known after apply)
      + connectivity_type    = "public"
      + id                   = (known after apply)
      + network_interface_id = (known after apply)
      + private_ip           = (known after apply)
      + public_ip            = (known after apply)
      + subnet_id            = (known after apply)
      + tags                 = {
          + "Environment" = "dev"
          + "Name"        = "example-vpc-us-west-2a"
          + "Terraform"   = "true"
        }
      + tags_all             = {
          + "Environment" = "dev"
          + "Name"        = "example-vpc-us-west-2a"
          + "Terraform"   = "true"
        }
    }

  # module.vpc.aws_nat_gateway.this[1] will be created
  + resource "aws_nat_gateway" "this" {
      + allocation_id        = (known after apply)
      + connectivity_type    = "public"
      + id                   = (known after apply)
      + network_interface_id = (known after apply)
      + private_ip           = (known after apply)
      + public_ip            = (known after apply)
      + subnet_id            = (known after apply)
      + tags                 = {
          + "Environment" = "dev"
          + "Name"        = "example-vpc-us-west-2b"
          + "Terraform"   = "true"
        }
      + tags_all             = {
          + "Environment" = "dev"
          + "Name"        = "example-vpc-us-west-2b"
          + "Terraform"   = "true"
        }
    }

  # module.vpc.aws_route.private_nat_gateway[0] will be created
  + resource "aws_route" "private_nat_gateway" {
      + destination_cidr_block = "0.0.0.0/0"
      + id                     = (known after apply)
      + instance_id            = (known after apply)
      + instance_owner_id      = (known after apply)
      + nat_gateway_id         = (known after apply)
      + network_interface_id   = (known after apply)
      + origin                 = (known after apply)
      + route_table_id         = (known after apply)
      + state                  = (known after apply)

      + timeouts {
          + create = "5m"
        }
    }

  # module.vpc.aws_route.private_nat_gateway[1] will be created
  + resource "aws_route" "private_nat_gateway" {
      + destination_cidr_block = "0.0.0.0/0"
      + id                     = (known after apply)
      + instance_id            = (known after apply)
      + instance_owner_id      = (known after apply)
      + nat_gateway_id         = (known after apply)
      + network_interface_id   = (known after apply)
      + origin                 = (known after apply)
      + route_table_id         = (known after apply)
      + state                  = (known after apply)

      + timeouts {
          + create = "5m"
        }
    }

  # module.vpc.aws_route.public_internet_gateway[0] will be created
  + resource "aws_route" "public_internet_gateway" {
      + destination_cidr_block = "0.0.0.0/0"
      + gateway_id             = (known after apply)
      + id                     = (known after apply)
      + instance_id            = (known after apply)
      + instance_owner_id      = (known after apply)
      + network_interface_id   = (known after apply)
      + origin                 = (known after apply)
      + route_table_id         = (known after apply)
      + state                  = (known after apply)

      + timeouts {
          + create = "5m"
        }
    }

  # module.vpc.aws_route_table.private[0] will be created
  + resource "aws_route_table" "private" {
      + arn              = (known after apply)
      + id               = (known after apply)
      + owner_id         = (known after apply)
      + propagating_vgws = (known after apply)
      + route            = (known after apply)
      + tags             = {
          + "Environment" = "dev"
          + "Name"        = "example-vpc-private-us-west-2a"
          + "Terraform"   = "true"
        }
      + tags_all         = {
          + "Environment" = "dev"
          + "Name"        = "example-vpc-private-us-west-2a"
          + "Terraform"   = "true"
        }
      + vpc_id           = (known after apply)
    }

  # module.vpc.aws_route_table.private[1] will be created
  + resource "aws_route_table" "private" {
      + arn              = (known after apply)
      + id               = (known after apply)
      + owner_id         = (known after apply)
      + propagating_vgws = (known after apply)
      + route            = (known after apply)
      + tags             = {
          + "Environment" = "dev"
          + "Name"        = "example-vpc-private-us-west-2b"
          + "Terraform"   = "true"
        }
      + tags_all         = {
          + "Environment" = "dev"
          + "Name"        = "example-vpc-private-us-west-2b"
          + "Terraform"   = "true"
        }
      + vpc_id           = (known after apply)
    }

  # module.vpc.aws_route_table.public[0] will be created
  + resource "aws_route_table" "public" {
      + arn              = (known after apply)
      + id               = (known after apply)
      + owner_id         = (known after apply)
      + propagating_vgws = (known after apply)
      + route            = (known after apply)
      + tags             = {
          + "Environment" = "dev"
          + "Name"        = "example-vpc-public"
          + "Terraform"   = "true"
        }
      + tags_all         = {
          + "Environment" = "dev"
          + "Name"        = "example-vpc-public"
          + "Terraform"   = "true"
        }
      + vpc_id           = (known after apply)
    }

  # module.vpc.aws_route_table_association.private[0] will be created
  + resource "aws_route_table_association" "private" {
      + id             = (known after apply)
      + route_table_id = (known after apply)
      + subnet_id      = (known after apply)
    }

  # module.vpc.aws_route_table_association.private[1] will be created
  + resource "aws_route_table_association" "private" {
      + id             = (known after apply)
      + route_table_id = (known after apply)
      + subnet_id      = (known after apply)
    }

  # module.vpc.aws_route_table_association.public[0] will be created
  + resource "aws_route_table_association" "public" {
      + id             = (known after apply)
      + route_table_id = (known after apply)
      + subnet_id      = (known after apply)
    }

  # module.vpc.aws_route_table_association.public[1] will be created
  + resource "aws_route_table_association" "public" {
      + id             = (known after apply)
      + route_table_id = (known after apply)
      + subnet_id      = (known after apply)
    }

  # module.vpc.aws_subnet.private[0] will be created
  + resource "aws_subnet" "private" {
      + arn                                            = (known after apply)
      + assign_ipv6_address_on_creation                = false
      + availability_zone                              = "us-west-2a"
      + availability_zone_id                           = (known after apply)
      + cidr_block                                     = "10.0.1.0/24"
      + enable_dns64                                   = false
      + enable_resource_name_dns_a_record_on_launch    = false
      + enable_resource_name_dns_aaaa_record_on_launch = false
      + id                                             = (known after apply)
      + ipv6_cidr_block_association_id                 = (known after apply)
      + ipv6_native                                    = false
      + map_public_ip_on_launch                        = false
      + owner_id                                       = (known after apply)
      + private_dns_hostname_type_on_launch            = (known after apply)
      + tags                                           = {
          + "Environment" = "dev"
          + "Name"        = "example-vpc-private-us-west-2a"
          + "Terraform"   = "true"
        }
      + tags_all                                       = {
          + "Environment" = "dev"
          + "Name"        = "example-vpc-private-us-west-2a"
          + "Terraform"   = "true"
        }
      + vpc_id                                         = (known after apply)
    }

  # module.vpc.aws_subnet.private[1] will be created
  + resource "aws_subnet" "private" {
      + arn                                            = (known after apply)
      + assign_ipv6_address_on_creation                = false
      + availability_zone                              = "us-west-2b"
      + availability_zone_id                           = (known after apply)
      + cidr_block                                     = "10.0.2.0/24"
      + enable_dns64                                   = false
      + enable_resource_name_dns_a_record_on_launch    = false
      + enable_resource_name_dns_aaaa_record_on_launch = false
      + id                                             = (known after apply)
      + ipv6_cidr_block_association_id                 = (known after apply)
      + ipv6_native                                    = false
      + map_public_ip_on_launch                        = false
      + owner_id                                       = (known after apply)
      + private_dns_hostname_type_on_launch            = (known after apply)
      + tags                                           = {
          + "Environment" = "dev"
          + "Name"        = "example-vpc-private-us-west-2b"
          + "Terraform"   = "true"
        }
      + tags_all                                       = {
          + "Environment" = "dev"
          + "Name"        = "example-vpc-private-us-west-2b"
          + "Terraform"   = "true"
        }
      + vpc_id                                         = (known after apply)
    }

  # module.vpc.aws_subnet.public[0] will be created
  + resource "aws_subnet" "public" {
      + arn                                            = (known after apply)
      + assign_ipv6_address_on_creation                = false
      + availability_zone                              = "us-west-2a"
      + availability_zone_id                           = (known after apply)
      + cidr_block                                     = "10.0.101.0/24"
      + enable_dns64                                   = false
      + enable_resource_name_dns_a_record_on_launch    = false
      + enable_resource_name_dns_aaaa_record_on_launch = false
      + id                                             = (known after apply)
      + ipv6_cidr_block_association_id                 = (known after apply)
      + ipv6_native                                    = false
      + map_public_ip_on_launch                        = true
      + owner_id                                       = (known after apply)
      + private_dns_hostname_type_on_launch            = (known after apply)
      + tags                                           = {
          + "Environment" = "dev"
          + "Name"        = "example-vpc-public-us-west-2a"
          + "Terraform"   = "true"
        }
      + tags_all                                       = {
          + "Environment" = "dev"
          + "Name"        = "example-vpc-public-us-west-2a"
          + "Terraform"   = "true"
        }
      + vpc_id                                         = (known after apply)
    }

  # module.vpc.aws_subnet.public[1] will be created
  + resource "aws_subnet" "public" {
      + arn                                            = (known after apply)
      + assign_ipv6_address_on_creation                = false
      + availability_zone                              = "us-west-2b"
      + availability_zone_id                           = (known after apply)
      + cidr_block                                     = "10.0.102.0/24"
      + enable_dns64                                   = false
      + enable_resource_name_dns_a_record_on_launch    = false
      + enable_resource_name_dns_aaaa_record_on_launch = false
      + id                                             = (known after apply)
      + ipv6_cidr_block_association_id                 = (known after apply)
      + ipv6_native                                    = false
      + map_public_ip_on_launch                        = true
      + owner_id                                       = (known after apply)
      + private_dns_hostname_type_on_launch            = (known after apply)
      + tags                                           = {
          + "Environment" = "dev"
          + "Name"        = "example-vpc-public-us-west-2b"
          + "Terraform"   = "true"
        }
      + tags_all                                       = {
          + "Environment" = "dev"
          + "Name"        = "example-vpc-public-us-west-2b"
          + "Terraform"   = "true"
        }
      + vpc_id                                         = (known after apply)
    }

  # module.vpc.aws_vpc.this[0] will be created
  + resource "aws_vpc" "this" {
      + arn                                  = (known after apply)
      + assign_generated_ipv6_cidr_block     = false
      + cidr_block                           = "10.0.0.0/16"
      + default_network_acl_id               = (known after apply)
      + default_route_table_id               = (known after apply)
      + default_security_group_id            = (known after apply)
      + dhcp_options_id                      = (known after apply)
      + enable_classiclink                   = (known after apply)
      + enable_classiclink_dns_support       = (known after apply)
      + enable_dns_hostnames                 = false
      + enable_dns_support                   = true
      + id                                   = (known after apply)
      + instance_tenancy                     = "default"
      + ipv6_association_id                  = (known after apply)
      + ipv6_cidr_block                      = (known after apply)
      + ipv6_cidr_block_network_border_group = (known after apply)
      + main_route_table_id                  = (known after apply)
      + owner_id                             = (known after apply)
      + tags                                 = {
          + "Environment" = "dev"
          + "Name"        = "example-vpc"
          + "Terraform"   = "true"
        }
      + tags_all                             = {
          + "Environment" = "dev"
          + "Name"        = "example-vpc"
          + "Terraform"   = "true"
        }
    }

Plan: 22 to add, 0 to change, 0 to destroy.

Changes to Outputs:
  + ec2_instance_public_ips = [
      + (known after apply),
      + (known after apply),
    ]
  + vpc_public_subnets      = [
      + (known after apply),
      + (known after apply),
    ]


Warning: Argument is deprecated

Use s3_use_path_style instead.


Warning: Attribute Deprecated

Use s3_use_path_style instead.


Warning: Argument is deprecated

  on .terraform/modules/vpc/main.tf line 36, in resource "aws_vpc" "this":
  36:   enable_classiclink               = var.enable_classiclink

With the retirement of EC2-Classic the enable_classiclink attribute has been
deprecated and will be removed in a future version.

(and 2 more similar warnings elsewhere)

Do you want to perform these actions?
  Terraform will perform the actions described above.
  Only 'yes' will be accepted to approve.

  Enter a value: yes

module.vpc.aws_eip.nat[0]: Creating...
module.vpc.aws_vpc.this[0]: Creating...
module.vpc.aws_eip.nat[1]: Creating...
module.vpc.aws_eip.nat[1]: Creation complete after 1s [id=eipalloc-0a97f65e]
module.vpc.aws_eip.nat[0]: Creation complete after 1s [id=eipalloc-020f489a]
module.vpc.aws_vpc.this[0]: Creation complete after 1s [id=vpc-50b14e1a]
module.vpc.aws_internet_gateway.this[0]: Creating...
module.vpc.aws_subnet.public[0]: Creating...
module.vpc.aws_route_table.public[0]: Creating...
module.vpc.aws_subnet.public[1]: Creating...
module.vpc.aws_route_table.private[1]: Creating...
module.vpc.aws_subnet.private[0]: Creating...
module.vpc.aws_subnet.private[1]: Creating...
module.vpc.aws_route_table.private[0]: Creating...
module.vpc.aws_subnet.private[1]: Creation complete after 0s [id=subnet-beca5b02]
module.vpc.aws_internet_gateway.this[0]: Creation complete after 0s [id=igw-aadd7944]
module.vpc.aws_subnet.private[0]: Creation complete after 0s [id=subnet-9e2bdd71]
module.vpc.aws_route_table.public[0]: Creation complete after 0s [id=rtb-3d197c34]
module.vpc.aws_route.public_internet_gateway[0]: Creating...
module.vpc.aws_route_table.private[1]: Creation complete after 1s [id=rtb-b1e3d174]
module.vpc.aws_route_table.private[0]: Creation complete after 1s [id=rtb-e5e27eeb]
module.vpc.aws_route_table_association.private[0]: Creating...
module.vpc.aws_route_table_association.private[1]: Creating...
module.vpc.aws_route_table_association.private[0]: Creation complete after 0s [id=rtbassoc-57281da8]
module.vpc.aws_route_table_association.private[1]: Creation complete after 0s [id=rtbassoc-1de95be5]
module.vpc.aws_route.public_internet_gateway[0]: Creation complete after 1s [id=r-rtb-3d197c341080289494]
module.vpc.aws_subnet.public[0]: Still creating... [10s elapsed]
module.vpc.aws_subnet.public[1]: Still creating... [10s elapsed]
module.vpc.aws_subnet.public[0]: Creation complete after 10s [id=subnet-c06bbdf9]
module.vpc.aws_subnet.public[1]: Creation complete after 10s [id=subnet-95b9a548]
module.ec2_instances.aws_instance.this[0]: Creating...
module.vpc.aws_route_table_association.public[1]: Creating...
module.ec2_instances.aws_instance.this[1]: Creating...
module.vpc.aws_route_table_association.public[0]: Creating...
module.vpc.aws_nat_gateway.this[1]: Creating...
module.vpc.aws_nat_gateway.this[0]: Creating...
module.vpc.aws_route_table_association.public[0]: Creation complete after 0s [id=rtbassoc-8c0a77e3]
module.vpc.aws_route_table_association.public[1]: Creation complete after 0s [id=rtbassoc-84ac6d3f]
module.vpc.aws_nat_gateway.this[0]: Creation complete after 0s [id=nat-4b5b3d1433012053c]
module.vpc.aws_nat_gateway.this[1]: Creation complete after 0s [id=nat-d846e3356ef57db3c]
module.vpc.aws_route.private_nat_gateway[0]: Creating...
module.vpc.aws_route.private_nat_gateway[1]: Creating...
module.vpc.aws_route.private_nat_gateway[0]: Creation complete after 0s [id=r-rtb-e5e27eeb1080289494]
module.vpc.aws_route.private_nat_gateway[1]: Creation complete after 0s [id=r-rtb-b1e3d1741080289494]
module.ec2_instances.aws_instance.this[0]: Still creating... [10s elapsed]
module.ec2_instances.aws_instance.this[1]: Still creating... [10s elapsed]
module.ec2_instances.aws_instance.this[1]: Creation complete after 10s [id=i-d8b22f71be1c7b8ec]
module.ec2_instances.aws_instance.this[0]: Creation complete after 11s [id=i-104d295bd2daf0815]

Apply complete! Resources: 22 added, 0 changed, 0 destroyed.

Outputs:

ec2_instance_public_ips = [
  "54.214.14.22",
  "54.214.82.36",
]
vpc_public_subnets = [
  "subnet-c06bbdf9",
  "subnet-95b9a548",
]
root@terraform:~/learn-terraform-modules# 
```
When using a new module for the first time, you must run either `terraform init` or `terraform get` to install the module. When you run these commands, Terraform will install any new modules in the `.terraform/modules` directory within your configuration's working directory. For local modules, Terraform will create a symlink to the module's directory. Because of this, any changes to local modules will be effective immediately, without having to reinitialize or re-run terraform get.
```sh
root@terraform:~/learn-terraform-modules# tree .terraform/modules/ -L 1
.terraform/modules/
├── ec2_instances
├── modules.json
└── vpc

2 directories, 1 file

root@terraform:~/learn-terraform-modules# tree .terraform/modules/
.terraform/modules/
├── ec2_instances
│   ├── CHANGELOG.md
│   ├── LICENSE
│   ├── Makefile
│   ├── README.md
│   ├── examples
│   │   ├── basic
│   │   │   ├── README.md
│   │   │   ├── main.tf
│   │   │   └── outputs.tf
│   │   └── volume-attachment
│   │       ├── README.md
│   │       ├── main.tf
│   │       └── outputs.tf
│   ├── main.tf
│   ├── outputs.tf
│   └── variables.tf
├── modules.json
└── vpc
    ├── CHANGELOG.md
    ├── Gemfile
    ├── LICENSE
    ├── Makefile
    ├── README.md
    ├── examples
    │   ├── complete-vpc
    │   │   ├── README.md
    │   │   ├── main.tf
    │   │   └── outputs.tf
    │   ├── ipv6
    │   │   ├── README.md
    │   │   ├── main.tf
    │   │   └── outputs.tf
    │   ├── issue-108-route-already-exists
    │   │   ├── README.md
    │   │   ├── main.tf
    │   │   └── outputs.tf
    │   ├── issue-224-vpcendpoint-apigw
    │   │   └── main.tf
    │   ├── issue-44-asymmetric-private-subnets
    │   │   ├── README.md
    │   │   ├── main.tf
    │   │   └── outputs.tf
    │   ├── issue-46-no-private-subnets
    │   │   ├── README.md
    │   │   ├── main.tf
    │   │   └── outputs.tf
    │   ├── manage-default-vpc
    │   │   ├── README.md
    │   │   ├── main.tf
    │   │   └── outputs.tf
    │   ├── network-acls
    │   │   ├── README.md
    │   │   ├── main.tf
    │   │   └── outputs.tf
    │   ├── secondary-cidr-blocks
    │   │   ├── README.md
    │   │   ├── main.tf
    │   │   └── outputs.tf
    │   ├── simple-vpc
    │   │   ├── README.md
    │   │   ├── main.tf
    │   │   └── outputs.tf
    │   ├── test_fixture
    │   │   ├── README.md
    │   │   ├── main.tf
    │   │   ├── outputs.tf
    │   │   └── variables.tf
    │   └── vpc-separate-private-route-tables
    │       ├── README.md
    │       ├── main.tf
    │       └── outputs.tf
    ├── main.tf
    ├── outputs.tf
    ├── test
    │   └── integration
    │       └── default
    │           └── test_vpc.rb
    ├── variables.tf
    └── vpc-endpoints.tf

21 directories, 59 files
root@terraform:~/learn-terraform-modules# tree .terraform/modules/ -L 1
.terraform/modules/
├── ec2_instances
├── modules.json
└── vpc

2 directories, 1 file
root@terraform:~/learn-terraform-modules#

```



```yaml
root@terraform:~/learn-terraform-modules# cat docker-compose.yml 
version: "3.2"
services:
  localstack:
    image: localstack/localstack:latest
    container_name: localstack_demo
    ports:
      - "4563-4584:4563-4584"
      - "8055:8080"
    environment:
      - SERVICES=s3,ec2,apigateway
      - DEBUG=1
      - DATA_DIR=/tmp/localstack/data
    volumes:
      - "./.localstack:/tmp/localstack"
      - "/var/run/docker.sock:/var/run/docker.sock"
root@terraform:~/learn-terraform-modules# 
```

```sh
root@terraform:~/learn-terraform-modules# docker ps
CONTAINER ID        IMAGE                   COMMAND                  CREATED             STATUS                    PORTS                                                                               NAMES
836e2311a11b        localstack/localstack   "docker-entrypoint.sh"   12 minutes ago      Up 11 minutes (healthy)   0.0.0.0:4566->4566/tcp, 4510-4559/tcp, 5678/tcp, 0.0.0.0:4571-4572->4571-4572/tcp   intelligent_dhawan
root@terraform:~/learn-terraform-modules# 
```

