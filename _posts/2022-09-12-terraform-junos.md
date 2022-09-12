---
layout: single
title:  "Getting started with Terraform Junos Provider"
date:   2022-09-12 01:58:04 +0530
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

# Terraform Junos Provider
In the Terraform registry, we can see Juniper official provider for vSRX devices.

[https://registry.terraform.io/providers/Juniper/junos-vsrx/20.32.106](https://registry.terraform.io/providers/Juniper/junos-vsrx/20.32.106)
![TFC]({{ site.url }}{{ site.baseurl }}/assets/images/tfc-17.png)

Referring to the documentation provider here 
[https://registry.terraform.io/providers/Juniper/junos-vsrx/latest/docs](https://registry.terraform.io/providers/Juniper/junos-vsrx/latest/docs)

![TFC]({{ site.url }}{{ site.baseurl }}/assets/images/tfc-18.png)

Let us configure a vSRX device using Terraform.

The purpose of this Terraform is to configure an IP address of 10.0.0.1/24 on ge-0/0/1.0 interface of the vSRX (whose management IP is 10.210.40.133). On the Junos device, we need to enable Netconf service, other than that, no other requirements.

On the host running Terraform, create the following configuration files.


```sh
pradeep@TF junos-terraform % pwd
/Users/pradga/junos-terraform
pradeep@TF junos-terraform % ls
main.tf vsrx_1
pradeep@TF junos-terraform %
```
Here we are using the official Juniper/junos-vsrx Terraform provider. 
Juniper has released an automation framework to generate custom Terraform providers. The framework is called JTAF—Juniper Terraform Automation Framework.
Refer to [https://github.com/Juniper/junos-terraform](https://github.com/Juniper/junos-terraform).


```sh
pradeep@TF junos-terraform % cat main.tf
terraform {
  required_providers {
    junos-vsrx = {
      source = "Juniper/junos-vsrx"
      version = "20.32.106"
    }
  }
}

provider "junos-vsrx" {
    host = "10.210.40.133"
    port = 830
    username = "root"
    password = "Juniper"
    sshkey = ""
}

module "vsrx_1" {
  source = "./vsrx_1"


  providers = {junos-vsrx = junos-vsrx}

  depends_on = [junos-vsrx_destroycommit.commit-main]
}


resource "junos-vsrx_commit" "commit-main" {
  resource_name = "commit"
  depends_on = [module.vsrx_1]
}

resource "junos-vsrx_destroycommit" "commit-main" {
  resource_name = "destroycommit"
}
pradeep@TF junos-terraform %
```



```sh
pradeep@TF junos-terraform % cat vsrx_1/main.tf

terraform {
  required_providers {
    junos-vsrx = {
      source = "Juniper/junos-vsrx"
      version = "20.32.106"
    }
  }
}

// To test Terraform's ability to remove a single resource,
// comment out one resource below and use `terraform taint` on the commit
// resource so that an apply also applies the commit.
resource "junos-vsrx_InterfacesInterfaceDescription" "vsrx_1" {
    resource_name = "vsrx_1"
    name = "ge-0/0/1"
    description = "Added from Terraform by Pradeep"
}

resource "junos-vsrx_InterfacesInterfaceUnitFamilyInetAddressName" "vsrx_2" {
    resource_name = "vsrx_2"
    name = "ge-0/0/1"
    name__1 = "0"
    name__2 = "10.0.0.1/24"
}
pradeep@TF junos-terraform %
```
Here we have created two resources, one resource for description and another for the IP address configuration.


```sh
pradeep@TF junos-terraform % terraform init
Initializing modules...

Initializing the backend...

Initializing provider plugins...
- Finding juniper/junos-vsrx versions matching "20.32.106"...
- Installing juniper/junos-vsrx v20.32.106...
- Installed juniper/junos-vsrx v20.32.106 (signed by a HashiCorp partner, key ID 67139BCB88D04711)

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
pradeep@TF junos-terraform %
```



```sh
pradeep@TF junos-terraform % terraform plan

Terraform used the selected providers to generate the following execution plan. Resource actions are indicated
with the following symbols:
  + create

Terraform will perform the following actions:

  # junos-vsrx_commit.commit-main will be created
  + resource "junos-vsrx_commit" "commit-main" {
      + id            = (known after apply)
      + resource_name = "commit"
    }

  # junos-vsrx_destroycommit.commit-main will be created
  + resource "junos-vsrx_destroycommit" "commit-main" {
      + id            = (known after apply)
      + resource_name = "destroycommit"
    }

  # module.vsrx_1.junos-vsrx_InterfacesInterfaceDescription.vsrx_1 will be created
  + resource "junos-vsrx_InterfacesInterfaceDescription" "vsrx_1" {
      + description   = "Added from Terraform by Pradeep"
      + id            = (known after apply)
      + name          = "ge-0/0/1"
      + resource_name = "vsrx_1"
    }

  # module.vsrx_1.junos-vsrx_InterfacesInterfaceUnitFamilyInetAddressName.vsrx_2 will be created
  + resource "junos-vsrx_InterfacesInterfaceUnitFamilyInetAddressName" "vsrx_2" {
      + id            = (known after apply)
      + name          = "ge-0/0/1"
      + name__1       = "0"
      + name__2       = "10.0.0.1/24"
      + resource_name = "vsrx_2"
    }

Plan: 4 to add, 0 to change, 0 to destroy.

─────────────────────────────────────────────────────────────────────────────────────────────────────────────────

Note: You didn't use the -out option to save this plan, so Terraform can't guarantee to take exactly these
actions if you run "terraform apply" now.
pradeep@TF junos-terraform %
```

In total, four resources are going to be added, out of these two resources are for our interface configuration and the other two are `commit` and `destroycommit` which are required always.


```sh
pradeep@TF junos-terraform % terraform apply

Terraform used the selected providers to generate the following execution plan. Resource actions are indicated
with the following symbols:
  + create

Terraform will perform the following actions:

  # junos-vsrx_commit.commit-main will be created
  + resource "junos-vsrx_commit" "commit-main" {
      + id            = (known after apply)
      + resource_name = "commit"
    }

  # junos-vsrx_destroycommit.commit-main will be created
  + resource "junos-vsrx_destroycommit" "commit-main" {
      + id            = (known after apply)
      + resource_name = "destroycommit"
    }

  # module.vsrx_1.junos-vsrx_InterfacesInterfaceDescription.vsrx_1 will be created
  + resource "junos-vsrx_InterfacesInterfaceDescription" "vsrx_1" {
      + description   = "Added from Terraform by Pradeep"
      + id            = (known after apply)
      + name          = "ge-0/0/1"
      + resource_name = "vsrx_1"
    }

  # module.vsrx_1.junos-vsrx_InterfacesInterfaceUnitFamilyInetAddressName.vsrx_2 will be created
  + resource "junos-vsrx_InterfacesInterfaceUnitFamilyInetAddressName" "vsrx_2" {
      + id            = (known after apply)
      + name          = "ge-0/0/1"
      + name__1       = "0"
      + name__2       = "10.0.0.1/24"
      + resource_name = "vsrx_2"
    }

Plan: 4 to add, 0 to change, 0 to destroy.

Do you want to perform these actions?
  Terraform will perform the actions described above.
  Only 'yes' will be accepted to approve.

  Enter a value: yes

junos-vsrx_destroycommit.commit-main: Creating...
junos-vsrx_destroycommit.commit-main: Creation complete after 0s [id=10.210.40.133_destroycommit]
module.vsrx_1.junos-vsrx_InterfacesInterfaceDescription.vsrx_1: Creating...
module.vsrx_1.junos-vsrx_InterfacesInterfaceUnitFamilyInetAddressName.vsrx_2: Creating...
module.vsrx_1.junos-vsrx_InterfacesInterfaceDescription.vsrx_1: Creation complete after 7s [id=10.210.40.133_vsrx_1]
module.vsrx_1.junos-vsrx_InterfacesInterfaceUnitFamilyInetAddressName.vsrx_2: Still creating... [10s elapsed]
module.vsrx_1.junos-vsrx_InterfacesInterfaceUnitFamilyInetAddressName.vsrx_2: Creation complete after 13s [id=10.210.40.133_vsrx_2]
junos-vsrx_commit.commit-main: Creating...
junos-vsrx_commit.commit-main: Creation complete after 7s [id=10.210.40.133_commit]

Apply complete! Resources: 4 added, 0 changed, 0 destroyed.
pradeep@TF junos-terraform %
```

Let us verify the Junos device configuration.

```sh
pradeep@TF junos-terraform % ssh root@10.210.40.133
(root@10.210.40.133) Password:
Last login: Mon Sep 12 01:26:44 2022 from 10.210.49.8
--- JUNOS 22.2R1.9 Kernel 64-bit XEN JNPR-12.1-20220607.2c547a1_buil
root@vsrx-1:~ # cli
root@vsrx-1>

root@vsrx-1> show configuration interfaces
ge-0/0/0 {
    unit 0 {
        family inet {
            dhcp;
        }
    }
}
fxp0 {
    unit 0 {
        family inet {
            address 172.25.11.3/24;
        }
    }
}

root@vsrx-1>
```

It looks like the configurtion is not present. It is because, the way Terraform Juniper provider generates the configuration.

For every terraform resource, it creates a configuration group in the Junos. So to verify the configuration inherited from the groups, use the `| display inheritance` statement.


```sh
root@vsrx-1> show configuration interfaces | display inheritance
ge-0/0/0 {
    unit 0 {
        family inet {
            dhcp;
        }
    }
}
fxp0 {
    unit 0 {
        family inet {
            address 172.25.11.3/24;
        }
    }
}
##
## 'ge-0/0/1' was inherited from group 'vsrx_1'
##
ge-0/0/1 {
    ##
    ## 'Added from Terraform by Pradeep' was inherited from group 'vsrx_1'
    ##
    description "Added from Terraform by Pradeep";
    ##
    ## '0' was inherited from group 'vsrx_2'
    ##
    unit 0 {
        ##
        ## 'inet' was inherited from group 'vsrx_2'
        ##
        family inet {
            ##
            ## '10.0.0.1/24' was inherited from group 'vsrx_2'
            ##
            address 10.0.0.1/24;
        }
    }
}

root@vsrx-1>
```
Now we can see that the ge-0/0/1 interface is configured with the description and the IP address.

To omit comments, use `| display inheritance no-comments`.

```sh
root@vsrx-1> show configuration interfaces | display inheritance no-comments
ge-0/0/0 {
    unit 0 {
        family inet {
            dhcp;
        }
    }
}
fxp0 {
    unit 0 {
        family inet {
            address 172.25.11.3/24;
        }
    }
}
ge-0/0/1 {
    description "Added from Terraform by Pradeep";
    unit 0 {
        family inet {
            address 10.0.0.1/24;
        }
    }
}

root@vsrx-1>
```

To verify the actual configuration groups created by Terraform.

```sh
root@vsrx-1> show configuration groups
vsrx_1 {
    interfaces {
        ge-0/0/1 {
            description "Added from Terraform by Pradeep";
        }
    }
}
vsrx_2 {
    interfaces {
        ge-0/0/1 {
            unit 0 {
                family inet {
                    address 10.0.0.1/24;
                }
            }
        }
    }
}

root@vsrx-1>
```
There are two groups—`vsrx_1` and `vsrx_2`. These are nothing but the names of the Terraform resources.

`resource_name = "vsrx_1"` and `resource_name = "vsrx_2"`.

Finally, let us undo the changes by destroying the resources.

```sh
pradeep@TF junos-terraform % terraform destroy
junos-vsrx_destroycommit.commit-main: Refreshing state... [id=10.210.40.133_destroycommit]
module.vsrx_1.junos-vsrx_InterfacesInterfaceDescription.vsrx_1: Refreshing state... [id=10.210.40.133_vsrx_1]
module.vsrx_1.junos-vsrx_InterfacesInterfaceUnitFamilyInetAddressName.vsrx_2: Refreshing state... [id=10.210.40.133_vsrx_2]
junos-vsrx_commit.commit-main: Refreshing state... [id=10.210.40.133_commit]

Terraform used the selected providers to generate the following execution plan. Resource actions are indicated
with the following symbols:
  - destroy

Terraform will perform the following actions:

  # junos-vsrx_commit.commit-main will be destroyed
  - resource "junos-vsrx_commit" "commit-main" {
      - id            = "10.210.40.133_commit" -> null
      - resource_name = "commit" -> null
    }

  # junos-vsrx_destroycommit.commit-main will be destroyed
  - resource "junos-vsrx_destroycommit" "commit-main" {
      - id            = "10.210.40.133_destroycommit" -> null
      - resource_name = "destroycommit" -> null
    }

  # module.vsrx_1.junos-vsrx_InterfacesInterfaceDescription.vsrx_1 will be destroyed
  - resource "junos-vsrx_InterfacesInterfaceDescription" "vsrx_1" {
      - description   = "Added from Terraform by Pradeep" -> null
      - id            = "10.210.40.133_vsrx_1" -> null
      - name          = "ge-0/0/1" -> null
      - resource_name = "vsrx_1" -> null
    }

  # module.vsrx_1.junos-vsrx_InterfacesInterfaceUnitFamilyInetAddressName.vsrx_2 will be destroyed
  - resource "junos-vsrx_InterfacesInterfaceUnitFamilyInetAddressName" "vsrx_2" {
      - id            = "10.210.40.133_vsrx_2" -> null
      - name          = "ge-0/0/1" -> null
      - name__1       = "0" -> null
      - name__2       = "10.0.0.1/24" -> null
      - resource_name = "vsrx_2" -> null
    }

Plan: 0 to add, 0 to change, 4 to destroy.

Do you really want to destroy all resources?
  Terraform will destroy all your managed infrastructure, as shown above.
  There is no undo. Only 'yes' will be accepted to confirm.

  Enter a value: yes

junos-vsrx_commit.commit-main: Destroying... [id=10.210.40.133_commit]
junos-vsrx_commit.commit-main: Destruction complete after 0s
module.vsrx_1.junos-vsrx_InterfacesInterfaceUnitFamilyInetAddressName.vsrx_2: Destroying... [id=10.210.40.133_vsrx_2]
module.vsrx_1.junos-vsrx_InterfacesInterfaceDescription.vsrx_1: Destroying... [id=10.210.40.133_vsrx_1]
module.vsrx_1.junos-vsrx_InterfacesInterfaceDescription.vsrx_1: Destruction complete after 3s
module.vsrx_1.junos-vsrx_InterfacesInterfaceUnitFamilyInetAddressName.vsrx_2: Destruction complete after 6s
junos-vsrx_destroycommit.commit-main: Destroying... [id=10.210.40.133_destroycommit]
junos-vsrx_destroycommit.commit-main: Destruction complete after 5s

Destroy complete! Resources: 4 destroyed.
pradeep@TF junos-terraform %
```


Verify the commit history on the Junos device. The latest two commits are done by Terraform via netconf.

```sh
root@vsrx-1> show system commit
0   2022-09-12 02:03:16 UTC by root via netconf
1   2022-09-12 01:59:18 UTC by root via netconf
2   2022-09-12 01:49:52 UTC by root via netconf
3   2022-09-12 01:48:06 UTC by root via netconf
4   2022-09-12 01:38:30 UTC by root via cli
5   2022-09-12 01:26:24 UTC by root via cli
6   2022-09-12 01:24:10 UTC by root via cli
7   2022-09-12 01:22:52 UTC by root via cli
8   2022-08-29 12:13:36 UTC by root via cli
9   2022-08-29 12:03:10 UTC by root via other
```
As we have destroyed the resources, the corresponding configuration groups are deleted from the device.

```
root@vsrx-1> show configuration groups

root@vsrx-1>
```

This concludes the first post on using Terraform provider for managing  Junos devices.



