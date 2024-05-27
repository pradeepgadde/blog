---
layout: single
title:  "Juniper Apstra Terraform Provider — Functions"
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

# Juniper Apstra Terraform Provider — Functions

The Terraform language includes a number of built-in functions that you can call from within expressions to transform and combine values. The general syntax for function calls is a function name followed by comma-separated arguments in parentheses.

The Terraform language does not support user-defined functions, and so only the functions built in to the language are available for use. We can experiment with the behavior of Terraform's built-in functions from the Terraform expression console, by running the `terraform console` command.

## Terraform Console

```
> help
The Terraform console allows you to experiment with Terraform interpolations.
You may access resources in the state (if you have one) just as you would
from a configuration. For example: "aws_instance.foo.id" would evaluate
to the ID of "aws_instance.foo" if it exists in your state.

Type in the interpolation to test and hit <enter> to see the result.

To exit the console, type "exit" and hit <enter>, or use Control-C or
Control-D.
>  
```
## String Functions

`upper` converts all cased letters in the given string to uppercase.

```
> upper("Juniper Apstra")
"JUNIPER APSTRA"
> 
```
`lower` converts all cased letters in the given string to lowercase.
```
> lower("Terraform")
"terraform"
>  
```
`sort` takes a list of strings and returns a new list with those strings sorted lexicographically.

```
> sort(["j","u","n","i","p","e","r"])
tolist([
  "e",
  "i",
  "j",
  "n",
  "p",
  "r",
  "u",
])
>  
```

`slice` extracts some consecutive elements from within a list. Here is the syntax `slice(list, startindex, endindex)`. `startindex` is inclusive, while `endindex` is exclusive. 

```
> slice(["j","u","n","i","p","e","r"],1,3)
[
  "u",
  "n",
]
> 
```

`distinct` takes a list and returns a new list with any duplicate elements removed.

The first occurrence of each value is retained and the relative ordering of these elements is preserved.

```
> distinct(["r","a","c","e","c","a","r"])
tolist([
  "r",
  "a",
  "c",
  "e",
])
>  
```

`split` produces a list by dividing a given string at all occurrences of a given separator.

```
> split(".", "192.168.1.1")
tolist([
  "192",
  "168",
  "1",
  "1",
])
>  
```
`join` produces a string by concatenating all of the elements of the specified list of strings with the specified separator.

```
> join("-", ["Juniper", "Apstra", "Terraform"])
"Juniper-Apstra-Terraform"
> 
```



## Numeric Functions

`min` takes one or more numbers and returns the smallest number from the set.
`max` takes one or more numbers and returns the greatest number from the set.

```
> min(1,2,3)
1
> 
```

```
> max(5,6,7)
7
> 
```

`floor` returns the closest whole number that is less than or equal to the given value, which may be a fraction.

```
> floor(3.13)
3
> floor(5)
5
> floor(1.89)
1
> 
```

## IP Network Functions

`cidrnetmask` converts an IPv4 address prefix given in CIDR notation into a subnet mask address.
```
> cidrnetmask("172.16.0.0/12")
"255.240.0.0"
> cidrnetmask("192.168.1.0/24")
"255.255.255.0"
> cidrnetmask("192.168.1.0/27")
"255.255.255.224"
>  
```
`cidrhost` calculates a full host IP address for a given host number within a given IP network address prefix.

```
> cidrhost("192.168.0.0/16", 300)
"192.168.1.44"
> cidrhost("192.168.0.0/16", 200)
"192.168.0.200"
> cidrhost("192.168.0.0/16", 512)
"192.168.2.0"
> cidrhost("192.168.0.0/16", 1024)
"192.168.4.0"
>  
```



## Type Conversion Functions

`tonumber` converts its argument to a number value.

```
> tonumber("1")
1
> tonumber(9)
9
> tonumber("900")
900
>  
```

## For Expression

```
> [for s in ["a","b"] : upper(s)]
[
  "A",
  "B",
]
>
```

A `for` expression's input (given after the `in` keyword) can be a list, a set, a tuple, a map, or an object.

The type of brackets around the `for` expression decide what type of result it produces.

The above example uses `[` and `]`, which produces a tuple. If you use `{` and `}` instead, the result is an object and you must provide two result expressions that are separated by the `=>` symbol:

```
> {for s in ["a","b"] : s => upper(s)}
{
  "a" = "A"
  "b" = "B"
}
>  
```

## Splat Expression

A *splat expression* provides a more concise way to express a common operation that could otherwise be performed with a `for` expression.

The special `[*]` symbol iterates over all of the elements of the list given to its left and accesses from each one the attribute name given on its right. 

```
> ["a","b"][*]
[
  "a",
  "b",
]
> ["a","b"][0]
"a"
> ["a","b"][1]
"b"
>
```



Let’s put all these together and create a VNI pool in Juniper Apstra.

```
pradeep@juniper apstra-terraform-8 % cat main.tf 
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

# This example creates a VNI pool consisting of 5 ranges with random
# begin/end values. Because the ranges must not overlap, the randomly
# selected begin/end values are sorted (as text!) before being used
# in a VNI pool range. If we randomly select the same value more than
# once, we'll wind up with fewer than 5 ranges.
locals {
  vni_min        = 4096
  vni_max        = 16777214
  ranges_desired = 5
}

# generate 10 random values
resource "random_integer" "range_limits" {
  count = local.ranges_desired * 2
  min   = local.vni_min
  max   = local.vni_max
}

# unique-ify, count pairs, and sort
locals {
  unique_values    = distinct(random_integer.range_limits[*].result)
  pair_count       = floor(length(local.unique_values) / 2)
  unsorted_strings = formatlist("%08d", slice(local.unique_values, 0, local.pair_count * 2))
  sorted_strings   = sort(local.unsorted_strings)
  sorted_numbers   = [for s in local.sorted_strings : tonumber(s)]
}

# generate a VNI pool with ranges equal to the number
# of begin/end pairs available.
resource "apstra_vni_pool" "five_random_ranges" {
  name = "five random ranges"
  ranges = [for i in range(local.pair_count) : {
    first = local.sorted_numbers[i * 2]
    last  = local.sorted_numbers[(i * 2) + 1]
  }]
}


pradeep@juniper apstra-terraform-8 
```



## Terraform Init

```
pradeep@juniper apstra-terraform-8 % terraform init

Initializing the backend...

Initializing provider plugins...
- Finding juniper/apstra versions matching ">= 0.28.0"...
- Finding latest version of hashicorp/random...
- Installing juniper/apstra v0.42.0...
- Installed juniper/apstra v0.42.0 (signed by a HashiCorp partner, key ID CB9C922903A66F3F)
- Installing hashicorp/random v3.5.1...
- Installed hashicorp/random v3.5.1 (signed by HashiCorp)

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
pradeep@juniper apstra-terraform-8 % 
```

## Terraform Apply

```
pradeep@juniper apstra-terraform-8 % terraform apply

Terraform used the selected providers to generate the following execution plan. Resource actions are indicated with the following symbols:
  + create

Terraform will perform the following actions:

  # apstra_vni_pool.five_random_ranges will be created
  + resource "apstra_vni_pool" "five_random_ranges" {
      + id              = (known after apply)
      + name            = "five random ranges"
      + ranges          = (known after apply)
      + status          = (known after apply)
      + total           = (known after apply)
      + used            = (known after apply)
      + used_percentage = (known after apply)
    }

  # random_integer.range_limits[0] will be created
  + resource "random_integer" "range_limits" {
      + id     = (known after apply)
      + max    = 16777214
      + min    = 4096
      + result = (known after apply)
    }

  # random_integer.range_limits[1] will be created
  + resource "random_integer" "range_limits" {
      + id     = (known after apply)
      + max    = 16777214
      + min    = 4096
      + result = (known after apply)
    }

  # random_integer.range_limits[2] will be created
  + resource "random_integer" "range_limits" {
      + id     = (known after apply)
      + max    = 16777214
      + min    = 4096
      + result = (known after apply)
    }

  # random_integer.range_limits[3] will be created
  + resource "random_integer" "range_limits" {
      + id     = (known after apply)
      + max    = 16777214
      + min    = 4096
      + result = (known after apply)
    }

  # random_integer.range_limits[4] will be created
  + resource "random_integer" "range_limits" {
      + id     = (known after apply)
      + max    = 16777214
      + min    = 4096
      + result = (known after apply)
    }

  # random_integer.range_limits[5] will be created
  + resource "random_integer" "range_limits" {
      + id     = (known after apply)
      + max    = 16777214
      + min    = 4096
      + result = (known after apply)
    }

  # random_integer.range_limits[6] will be created
  + resource "random_integer" "range_limits" {
      + id     = (known after apply)
      + max    = 16777214
      + min    = 4096
      + result = (known after apply)
    }

  # random_integer.range_limits[7] will be created
  + resource "random_integer" "range_limits" {
      + id     = (known after apply)
      + max    = 16777214
      + min    = 4096
      + result = (known after apply)
    }

  # random_integer.range_limits[8] will be created
  + resource "random_integer" "range_limits" {
      + id     = (known after apply)
      + max    = 16777214
      + min    = 4096
      + result = (known after apply)
    }

  # random_integer.range_limits[9] will be created
  + resource "random_integer" "range_limits" {
      + id     = (known after apply)
      + max    = 16777214
      + min    = 4096
      + result = (known after apply)
    }

Plan: 11 to add, 0 to change, 0 to destroy.

Do you want to perform these actions?
  Terraform will perform the actions described above.
  Only 'yes' will be accepted to approve.

  Enter a value: yes

random_integer.range_limits[7]: Creating...
random_integer.range_limits[8]: Creating...
random_integer.range_limits[1]: Creating...
random_integer.range_limits[2]: Creating...
random_integer.range_limits[5]: Creating...
random_integer.range_limits[7]: Creation complete after 0s [id=701768]
random_integer.range_limits[6]: Creating...
random_integer.range_limits[9]: Creating...
random_integer.range_limits[3]: Creating...
random_integer.range_limits[4]: Creating...
random_integer.range_limits[1]: Creation complete after 0s [id=13773872]
random_integer.range_limits[0]: Creating...
random_integer.range_limits[8]: Creation complete after 0s [id=14349966]
random_integer.range_limits[9]: Creation complete after 0s [id=4782211]
random_integer.range_limits[2]: Creation complete after 0s [id=13501423]
random_integer.range_limits[3]: Creation complete after 0s [id=15658190]
random_integer.range_limits[5]: Creation complete after 0s [id=9377344]
random_integer.range_limits[0]: Creation complete after 0s [id=519737]
random_integer.range_limits[4]: Creation complete after 0s [id=8836472]
random_integer.range_limits[6]: Creation complete after 0s [id=15079159]
apstra_vni_pool.five_random_ranges: Creating...
apstra_vni_pool.five_random_ranges: Creation complete after 2s [id=25f63957-679b-4f45-b094-56c46ad3f9b4]

Apply complete! Resources: 11 added, 0 changed, 0 destroyed.
pradeep@juniper apstra-terraform-8 % 
```
Go to Apstra UI and verify the VNI pools.

![tf_apstra_34]({{ site.url }}{{ site.baseurl }}/assets/images/tf_apstra_34.png) 

```
pradeep@juniper apstra-terraform-8 % terraform show
# apstra_vni_pool.five_random_ranges:
resource "apstra_vni_pool" "five_random_ranges" {
    id              = "25f63957-679b-4f45-b094-56c46ad3f9b4"
    name            = "five random ranges"
    ranges          = [
        {
            first           = 13773872
            last            = 14349966
            status          = "pool_element_available"
            total           = 576095
            used            = 0
            used_percentage = 0
        },
        {
            first           = 15079159
            last            = 15658190
            status          = "pool_element_available"
            total           = 579032
            used            = 0
            used_percentage = 0
        },
        {
            first           = 4782211
            last            = 8836472
            status          = "pool_element_available"
            total           = 4054262
            used            = 0
            used_percentage = 0
        },
        {
            first           = 519737
            last            = 701768
            status          = "pool_element_available"
            total           = 182032
            used            = 0
            used_percentage = 0
        },
        {
            first           = 9377344
            last            = 13501423
            status          = "pool_element_available"
            total           = 4124080
            used            = 0
            used_percentage = 0
        },
    ]
    status          = "not_in_use"
    total           = 9515501
    used            = 0
    used_percentage = 0
}

# random_integer.range_limits[0]:
resource "random_integer" "range_limits" {
    id     = "519737"
    max    = 16777214
    min    = 4096
    result = 519737
}

# random_integer.range_limits[1]:
resource "random_integer" "range_limits" {
    id     = "13773872"
    max    = 16777214
    min    = 4096
    result = 13773872
}

# random_integer.range_limits[2]:
resource "random_integer" "range_limits" {
    id     = "13501423"
    max    = 16777214
    min    = 4096
    result = 13501423
}

# random_integer.range_limits[3]:
resource "random_integer" "range_limits" {
    id     = "15658190"
    max    = 16777214
    min    = 4096
    result = 15658190
}

# random_integer.range_limits[4]:
resource "random_integer" "range_limits" {
    id     = "8836472"
    max    = 16777214
    min    = 4096
    result = 8836472
}

# random_integer.range_limits[5]:
resource "random_integer" "range_limits" {
    id     = "9377344"
    max    = 16777214
    min    = 4096
    result = 9377344
}

# random_integer.range_limits[6]:
resource "random_integer" "range_limits" {
    id     = "15079159"
    max    = 16777214
    min    = 4096
    result = 15079159
}

# random_integer.range_limits[7]:
resource "random_integer" "range_limits" {
    id     = "701768"
    max    = 16777214
    min    = 4096
    result = 701768
}

# random_integer.range_limits[8]:
resource "random_integer" "range_limits" {
    id     = "14349966"
    max    = 16777214
    min    = 4096
    result = 14349966
}

# random_integer.range_limits[9]:
resource "random_integer" "range_limits" {
    id     = "4782211"
    max    = 16777214
    min    = 4096
    result = 4782211
}
pradeep@juniper apstra-terraform-8 % 
```

## Terraform Destroy

```
pradeep@juniper apstra-terraform-8 % terraform apply -destroy
random_integer.range_limits[1]: Refreshing state... [id=13773872]
random_integer.range_limits[2]: Refreshing state... [id=13501423]
random_integer.range_limits[4]: Refreshing state... [id=8836472]
random_integer.range_limits[6]: Refreshing state... [id=15079159]
random_integer.range_limits[9]: Refreshing state... [id=4782211]
random_integer.range_limits[5]: Refreshing state... [id=9377344]
random_integer.range_limits[3]: Refreshing state... [id=15658190]
random_integer.range_limits[7]: Refreshing state... [id=701768]
random_integer.range_limits[0]: Refreshing state... [id=519737]
random_integer.range_limits[8]: Refreshing state... [id=14349966]
apstra_vni_pool.five_random_ranges: Refreshing state... [id=25f63957-679b-4f45-b094-56c46ad3f9b4]

Terraform used the selected providers to generate the following execution plan. Resource actions are indicated with the following symbols:
  - destroy

Terraform will perform the following actions:

  # apstra_vni_pool.five_random_ranges will be destroyed
  - resource "apstra_vni_pool" "five_random_ranges" {
      - id              = "25f63957-679b-4f45-b094-56c46ad3f9b4" -> null
      - name            = "five random ranges" -> null
      - ranges          = [
          - {
              - first           = 13773872 -> null
              - last            = 14349966 -> null
              - status          = "pool_element_available" -> null
              - total           = 576095 -> null
              - used            = 0 -> null
              - used_percentage = 0 -> null
            },
          - {
              - first           = 15079159 -> null
              - last            = 15658190 -> null
              - status          = "pool_element_available" -> null
              - total           = 579032 -> null
              - used            = 0 -> null
              - used_percentage = 0 -> null
            },
          - {
              - first           = 4782211 -> null
              - last            = 8836472 -> null
              - status          = "pool_element_available" -> null
              - total           = 4054262 -> null
              - used            = 0 -> null
              - used_percentage = 0 -> null
            },
          - {
              - first           = 519737 -> null
              - last            = 701768 -> null
              - status          = "pool_element_available" -> null
              - total           = 182032 -> null
              - used            = 0 -> null
              - used_percentage = 0 -> null
            },
          - {
              - first           = 9377344 -> null
              - last            = 13501423 -> null
              - status          = "pool_element_available" -> null
              - total           = 4124080 -> null
              - used            = 0 -> null
              - used_percentage = 0 -> null
            },
        ] -> null
      - status          = "not_in_use" -> null
      - total           = 9515501 -> null
      - used            = 0 -> null
      - used_percentage = 0 -> null
    }

  # random_integer.range_limits[0] will be destroyed
  - resource "random_integer" "range_limits" {
      - id     = "519737" -> null
      - max    = 16777214 -> null
      - min    = 4096 -> null
      - result = 519737 -> null
    }

  # random_integer.range_limits[1] will be destroyed
  - resource "random_integer" "range_limits" {
      - id     = "13773872" -> null
      - max    = 16777214 -> null
      - min    = 4096 -> null
      - result = 13773872 -> null
    }

  # random_integer.range_limits[2] will be destroyed
  - resource "random_integer" "range_limits" {
      - id     = "13501423" -> null
      - max    = 16777214 -> null
      - min    = 4096 -> null
      - result = 13501423 -> null
    }

  # random_integer.range_limits[3] will be destroyed
  - resource "random_integer" "range_limits" {
      - id     = "15658190" -> null
      - max    = 16777214 -> null
      - min    = 4096 -> null
      - result = 15658190 -> null
    }

  # random_integer.range_limits[4] will be destroyed
  - resource "random_integer" "range_limits" {
      - id     = "8836472" -> null
      - max    = 16777214 -> null
      - min    = 4096 -> null
      - result = 8836472 -> null
    }

  # random_integer.range_limits[5] will be destroyed
  - resource "random_integer" "range_limits" {
      - id     = "9377344" -> null
      - max    = 16777214 -> null
      - min    = 4096 -> null
      - result = 9377344 -> null
    }

  # random_integer.range_limits[6] will be destroyed
  - resource "random_integer" "range_limits" {
      - id     = "15079159" -> null
      - max    = 16777214 -> null
      - min    = 4096 -> null
      - result = 15079159 -> null
    }

  # random_integer.range_limits[7] will be destroyed
  - resource "random_integer" "range_limits" {
      - id     = "701768" -> null
      - max    = 16777214 -> null
      - min    = 4096 -> null
      - result = 701768 -> null
    }

  # random_integer.range_limits[8] will be destroyed
  - resource "random_integer" "range_limits" {
      - id     = "14349966" -> null
      - max    = 16777214 -> null
      - min    = 4096 -> null
      - result = 14349966 -> null
    }

  # random_integer.range_limits[9] will be destroyed
  - resource "random_integer" "range_limits" {
      - id     = "4782211" -> null
      - max    = 16777214 -> null
      - min    = 4096 -> null
      - result = 4782211 -> null
    }

Plan: 0 to add, 0 to change, 11 to destroy.

Do you really want to destroy all resources?
  Terraform will destroy all your managed infrastructure, as shown above.
  There is no undo. Only 'yes' will be accepted to confirm.

  Enter a value: yes

apstra_vni_pool.five_random_ranges: Destroying... [id=25f63957-679b-4f45-b094-56c46ad3f9b4]
apstra_vni_pool.five_random_ranges: Destruction complete after 1s
random_integer.range_limits[7]: Destroying... [id=701768]
random_integer.range_limits[0]: Destroying... [id=519737]
random_integer.range_limits[5]: Destroying... [id=9377344]
random_integer.range_limits[6]: Destroying... [id=15079159]
random_integer.range_limits[2]: Destroying... [id=13501423]
random_integer.range_limits[9]: Destroying... [id=4782211]
random_integer.range_limits[3]: Destroying... [id=15658190]
random_integer.range_limits[4]: Destroying... [id=8836472]
random_integer.range_limits[1]: Destroying... [id=13773872]
random_integer.range_limits[8]: Destroying... [id=14349966]
random_integer.range_limits[3]: Destruction complete after 0s
random_integer.range_limits[4]: Destruction complete after 0s
random_integer.range_limits[8]: Destruction complete after 0s
random_integer.range_limits[1]: Destruction complete after 0s
random_integer.range_limits[0]: Destruction complete after 0s
random_integer.range_limits[9]: Destruction complete after 0s
random_integer.range_limits[7]: Destruction complete after 0s
random_integer.range_limits[2]: Destruction complete after 0s
random_integer.range_limits[6]: Destruction complete after 0s
random_integer.range_limits[5]: Destruction complete after 0s

Apply complete! Resources: 0 added, 0 changed, 11 destroyed.
pradeep@juniper apstra-terraform-8 % 
```

Go back to Apstra UI and verify that the VNI pools are no longer present

![tf_apstra_35]({{ site.url }}{{ site.baseurl }}/assets/images/tf_apstra_35.png) 
