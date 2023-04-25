---
layout: single
title:  "Managing Terraform State"
date:   2023-04-25 01:59:04 +0530
categories: Cloud
tags: GCP
classes: wide
show_date: true
header:
  overlay_image: /assets/images/gcp-banner-1.png
  og_image: /assets/images/gcp-banner-1.png
  teaser: /assets/images/gpcne.png
author:
  name     : "Cloud Network Engineer"
  avatar   : "/assets/images/gpcne.png"

sidebar:
  - title: "Topics"
    nav: my-sidebar

---

# Managing Terraform State

Perform the following tasks:

- Create a local backend.
- Create a Cloud Storage backend.
- Refresh your Terraform state.
- Import a Terraform configuration.
- Manage the imported configuration with Terraform.

Terraform must store the state about your managed infrastructure and  configuration. This state is used by Terraform to map real-world  resources to your configuration, keep track of metadata, and improve  performance for large infrastructures.

This state is stored by default in a local file named `terraform.tfstate`, but it can also be stored remotely, which works better in a team environment.

Terraform uses this local state to create plans and make changes to your infrastructure. Before any operation, Terraform does a [refresh](https://www.terraform.io/docs/commands/refresh.html) to update the state with the real infrastructure.

The primary purpose of Terraform state is to store bindings between  objects in a remote system and resource instances declared in your  configuration. When Terraform creates a remote object in response to a  change of configuration, it will record the identity of that remote  object against a particular resource instance and then potentially  update or delete that object in response to future configuration  changes.



## Purpose of Terraform state

State is a necessary requirement for Terraform to function. People  sometimes ask whether Terraform can work without state or not use state  and just inspect cloud resources on every run. In the scenarios where  Terraform may be able to get away without state, doing so would require  shifting massive amounts of complexity from one place (state) to another place (the replacement concept). This section will help explain why  Terraform state is required.

### Mapping to the real world

Terraform requires some sort of database to map Terraform config to the real world. When your configuration contains a `resource resource "google_compute_instance" "foo"`, Terraform uses this map to know that instance `i-abcd1234` is represented by that resource.

Terraform expects that each remote object is bound to only one  resource instance, which is normally guaranteed because Terraform is  responsible for creating the objects and recording their identities in  the state. If you instead import objects that were created outside of  Terraform, you must verify that each distinct object is imported to only one resource instance.

If one remote object is bound to two or more resource instances,  Terraform may take unexpected actions against those objects because the  mapping from configuration to the remote object state has become  ambiguous.

### Metadata

In addition to tracking the mappings between resources and remote  objects, Terraform must also track metadata such as resource  dependencies.

Terraform typically uses the configuration to determine dependency  order. However, when you remove a resource from a Terraform  configuration, Terraform must know how to delete that resource.  Terraform can see that a mapping exists for a resource that is not in  your configuration file and plan to destroy. However, because the  resource no longer exists, the order cannot be determined from the  configuration alone.

To ensure correct operation, Terraform retains a copy of the most  recent set of dependencies within the state. Now Terraform can still  determine the correct order for destruction from the state when you  delete one or more items from the configuration.

This could be avoided if Terraform knew a required ordering between  resource types. For example, Terraform could know that servers must be  deleted before the subnets they are a part of. The complexity for this  approach quickly becomes unmanageable, however: in addition to  understanding the ordering semantics of every resource for every cloud,  Terraform must also understand the ordering across providers.

Terraform also stores other metadata for similar reasons, such as a  pointer to the provider configuration that was most recently used with  the resource in situations where multiple aliased providers are present.

### Performance

In addition to basic mapping, Terraform stores a cache of the  attribute values for all resources in the state. This is an optional  feature of Terraform state and is used only as a performance  improvement.

When running a `terraform plan`, Terraform must know the  current state of resources in order to effectively determine the changes needed to reach your desired configuration.

For small infrastructures, Terraform can query your providers and  sync the latest attributes from all your resources. This is the default  behavior of Terraform: for every `plan` and `apply`, Terraform will sync all resources in your state.

For larger infrastructures, querying every resource is too slow. Many cloud providers do not provide APIs to query multiple resources at the  same time, and the round trip time for each resource is hundreds of  milliseconds. In addition, cloud providers almost always have API rate  limiting, so Terraform can only request a limited number of resources in a period of time. Larger users of Terraform frequently use both the `-refresh=false` flag and the `-target` flag in order to work around this. In these scenarios, the cached state is treated as the record of truth.

### Syncing

In the default configuration, Terraform stores the state in a file in the current working directory where Terraform was run. This works when  you are getting started, but when Terraform is used in a team, it is  important for everyone to be working with the same state so that  operations will be applied to the same remote objects.

[Remote state](https://www.terraform.io/docs/state/remote.html) is the recommended solution to this problem. With a fully featured  state backend, Terraform can use remote locking as a measure to avoid  multiple different users accidentally running Terraform at the same  time; this ensures that each Terraform run begins with the most recent  updated state.

### State locking

If supported by your [backend](https://www.terraform.io/docs/backends/), Terraform will lock your state for all operations that could write  state. This prevents others from acquiring the lock and potentially  corrupting your state.

State locking happens automatically on all operations that could  write state. You won't see any message that it is happening. If state  locking fails, Terraform will not continue. You can disable state  locking for most commands with the `-lock` flag, but it is not recommended.

If acquiring the lock is taking longer than expected, Terraform will  output a status message. If Terraform doesn't output a message, state  locking is still occurring.

Not all backends support locking. View the list of [backend types](https://www.terraform.io/language/settings/backends/configuration#backend-types) for details on whether a backend supports locking.

### Workspaces

Each Terraform configuration has an associated [backend](https://www.terraform.io/docs/backends/index.html) that defines how operations are executed and where persistent data such as the Terraform state is stored.

The persistent data stored in the backend belongs to a *workspace*. Initially the backend has only one workspace, called *default*, and thus only one Terraform state is associated with that configuration.

Certain backends support *multiple* named workspaces, which  allows multiple states to be associated with a single configuration. The configuration still has only one backend, but multiple distinct  instances of that configuration can be deployed without configuring a  new backend or changing authentication credentials



## 1. Working with backends

A *backend* in Terraform determines how state is loaded and how an operation such as `apply` is executed. This abstraction enables non-local file state storage, remote execution, etc.

By default, Terraform uses the "local" backend, which is the normal  behavior of Terraform you're used to. This is the backend that was being invoked throughout the previous labs.



### Add a local backend

In this section, you will configure a local backend.

When configuring a backend for the first time (moving from no defined backend to explicitly configuring one), Terraform will give you the  option to migrate your state to the new backend. This lets you adopt  backends without losing any existing state.

To be extra careful, we always recommend that you also manually back up your state. You can do this by simply copying your `terraform.tfstate` file to another location. The initialization process should also create a backup, but it never hurts to be safe!

Configuring a backend for the first time is no different from  changing a configuration in the future: create the new configuration and run `terraform init`. Terraform will guide you the rest of the way.

```sh
Welcome to Cloud Shell! Type "help" to get started.
Your Cloud Platform project in this session is set to qwiklabs-gcp-00-eea2f6cd711e.
Use “gcloud config set project [PROJECT_ID]” to change to a different project.
student_01_1996df2c1f49@cloudshell:~ (qwiklabs-gcp-00-eea2f6cd711e)$ touch main.tf
student_01_1996df2c1f49@cloudshell:~ (qwiklabs-gcp-00-eea2f6cd711e)$ gcloud config list --format 'value(core.project)'
qwiklabs-gcp-00-eea2f6cd711e
student_01_1996df2c1f49@cloudshell:~ (qwiklabs-gcp-00-eea2f6cd711e)$ cat main.tf
provider "google" {
  project     = "qwiklabs-gcp-00-eea2f6cd711e"
  region      = "us-central-1"
}
resource "google_storage_bucket" "test-bucket-for-state" {
  name        = "qwiklabs-gcp-00-eea2f6cd711e"
  location    = "US"
  uniform_bucket_level_access = true
}
terraform {
  backend "local" {
    path = "terraform/state/terraform.tfstate"
  }
}student_01_1996df2c1f49@cloudshell:~ (qwiklabs-gcp-00-eea2f6cd711e)$ terraform init

Initializing the backend...

Successfully configured the backend "local"! Terraform will automatically
use this backend unless the backend configuration changes.

Initializing provider plugins...
- Finding latest version of hashicorp/google...
- Installing hashicorp/google v4.63.0...
- Installed hashicorp/google v4.63.0 (signed by HashiCorp)

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
student_01_1996df2c1f49@cloudshell:~ (qwiklabs-gcp-00-eea2f6cd711e)$ terraform apply

Terraform used the selected providers to generate the following execution plan. Resource actions are indicated with the following symbols:
  + create

Terraform will perform the following actions:

  # google_storage_bucket.test-bucket-for-state will be created
  + resource "google_storage_bucket" "test-bucket-for-state" {
      + force_destroy               = false
      + id                          = (known after apply)
      + location                    = "US"
      + name                        = "qwiklabs-gcp-00-eea2f6cd711e"
      + project                     = (known after apply)
      + public_access_prevention    = (known after apply)
      + self_link                   = (known after apply)
      + storage_class               = "STANDARD"
      + uniform_bucket_level_access = true
      + url                         = (known after apply)
    }

Plan: 1 to add, 0 to change, 0 to destroy.

Do you want to perform these actions?
  Terraform will perform the actions described above.
  Only 'yes' will be accepted to approve.

  Enter a value: yes

google_storage_bucket.test-bucket-for-state: Creating...
google_storage_bucket.test-bucket-for-state: Creation complete after 2s [id=qwiklabs-gcp-00-eea2f6cd711e]

Apply complete! Resources: 1 added, 0 changed, 0 destroyed.
student_01_1996df2c1f49@cloudshell:~ (qwiklabs-gcp-00-eea2f6cd711e)$ terraform show
# google_storage_bucket.test-bucket-for-state:
resource "google_storage_bucket" "test-bucket-for-state" {
    default_event_based_hold    = false
    force_destroy               = false
    id                          = "qwiklabs-gcp-00-eea2f6cd711e"
    location                    = "US"
    name                        = "qwiklabs-gcp-00-eea2f6cd711e"
    project                     = "qwiklabs-gcp-00-eea2f6cd711e"
    public_access_prevention    = "inherited"
    requester_pays              = false
    self_link                   = "https://www.googleapis.com/storage/v1/b/qwiklabs-gcp-00-eea2f6cd711e"
    storage_class               = "STANDARD"
    uniform_bucket_level_access = true
    url                         = "gs://qwiklabs-gcp-00-eea2f6cd711e"
}
student_01_1996df2c1f49@cloudshell:~ (qwiklabs-gcp-00-eea2f6cd711e)$
```



### Add a Cloud Storage backend

A Cloud Storage backend stores the state as an object in a  configurable prefix in a given bucket on Cloud Storage. This backend  also supports [state locking](https://www.terraform.io/docs/state/locking.html). This will lock your state for all operations that could write state.  This prevents others from acquiring the lock and potentially corrupting  your state.

State locking happens automatically on all operations that could  write state. You won't see any message that it is happening. If state  locking fails, Terraform will not continue. You can disable state  locking for most commands with the `-lock` flag, but this is not recommended.

1. Navigate back to your `main.tf` file in the editor. You will now replace the current local backend with a `gcs` backend.
2. To change the existing local backend configuration, copy the following configuration into your file, replacing the `local` backend:

```sh
student_01_1996df2c1f49@cloudshell:~ (qwiklabs-gcp-00-eea2f6cd711e)$ cat main.tf
provider "google" {
  project     = "qwiklabs-gcp-00-eea2f6cd711e"
  region      = "us-central-1"
}
resource "google_storage_bucket" "test-bucket-for-state" {
  name        = "qwiklabs-gcp-00-eea2f6cd711e"
  location    = "US"
  uniform_bucket_level_access = true
}
terraform {
  backend "gcs" {
    bucket  = "qwiklabs-gcp-00-eea2f6cd711e"
    prefix  = "terraform/state"
  }
}student_01_1996df2c1f49@cloudshell:~ (qwiklabs-gcp-00-eea2f6cd711e)$
```



```sh
student_01_1996df2c1f49@cloudshell:~ (qwiklabs-gcp-00-eea2f6cd711e)$ terraform init -migrate-state

Initializing the backend...
Terraform detected that the backend type changed from "local" to "gcs".

Acquiring state lock. This may take a few moments...
Do you want to copy existing state to the new backend?
  Pre-existing state was found while migrating the previous "local" backend to the
  newly configured "gcs" backend. No existing state was found in the newly
  configured "gcs" backend. Do you want to copy this state to the new "gcs"
  backend? Enter "yes" to copy and "no" to start with an empty state.

  Enter a value: yes


Successfully configured the backend "gcs"! Terraform will automatically
use this backend unless the backend configuration changes.

Initializing provider plugins...
- Reusing previous version of hashicorp/google from the dependency lock file
- Using previously-installed hashicorp/google v4.63.0

Terraform has been successfully initialized!

You may now begin working with Terraform. Try running "terraform plan" to see
any changes that are required for your infrastructure. All Terraform commands
should now work.

If you ever set or change modules or backend configuration for Terraform,
rerun this command to reinitialize your working directory. If you forget, other
commands will detect it and remind you to do so if necessary.
student_01_1996df2c1f49@cloudshell:~ (qwiklabs-gcp-00-eea2f6cd711e)$
```



### Refresh the state

The `terraform refresh` command is used to reconcile the  state Terraform knows about (via its state file) with the real-world  infrastructure. This can be used to detect any drift from the last-known state and to update the state file.

This does not modify infrastructure, but does modify the state file.  If the state is changed, this may cause changes to occur during the next plan or apply.



```sh
student_01_1996df2c1f49@cloudshell:~ (qwiklabs-gcp-00-eea2f6cd711e)$ terraform refresh
Acquiring state lock. This may take a few moments...
google_storage_bucket.test-bucket-for-state: Refreshing state... [id=qwiklabs-gcp-00-eea2f6cd711e]
Releasing state lock. This may take a few moments...
student_01_1996df2c1f49@cloudshell:~ (qwiklabs-gcp-00-eea2f6cd711e)$
```

```sh
student_01_1996df2c1f49@cloudshell:~ (qwiklabs-gcp-00-eea2f6cd711e)$ terraform show
# google_storage_bucket.test-bucket-for-state:
resource "google_storage_bucket" "test-bucket-for-state" {
    default_event_based_hold    = false
    force_destroy               = false
    id                          = "qwiklabs-gcp-00-eea2f6cd711e"
    labels                      = {
        "key" = "value"
    }
    location                    = "US"
    name                        = "qwiklabs-gcp-00-eea2f6cd711e"
    project                     = "qwiklabs-gcp-00-eea2f6cd711e"
    public_access_prevention    = "inherited"
    requester_pays              = false
    self_link                   = "https://www.googleapis.com/storage/v1/b/qwiklabs-gcp-00-eea2f6cd711e"
    storage_class               = "STANDARD"
    uniform_bucket_level_access = true
    url                         = "gs://qwiklabs-gcp-00-eea2f6cd711e"
}
student_01_1996df2c1f49@cloudshell:~ (qwiklabs-gcp-00-eea2f6cd711e)$
```



### Clean up your workspace

Before continuing to the next task, destroy your provisioned infrastructure.

1. First, revert your backend to `local` so you can delete the storage bucket. Copy and replace the `gcs` configuration with the following:

```sh
student_01_1996df2c1f49@cloudshell:~ (qwiklabs-gcp-00-eea2f6cd711e)$ cat main.tf
provider "google" {
  project     = "qwiklabs-gcp-00-eea2f6cd711e"
  region      = "us-central-1"
}
resource "google_storage_bucket" "test-bucket-for-state" {
  name        = "qwiklabs-gcp-00-eea2f6cd711e"
  location    = "US"
  uniform_bucket_level_access = true
}
terraform {
  backend "local" {
    path = "terraform/state/terraform.tfstate"
  }
}student_01_1996df2c1f49@cloudshell:~ (qwiklabs-gcp-00-eea2f6cd711e)$
```

```sh
student_01_1996df2c1f49@cloudshell:~ (qwiklabs-gcp-00-eea2f6cd711e)$ terraform init -migrate-state

Initializing the backend...
Terraform detected that the backend type changed from "gcs" to "local".

Acquiring state lock. This may take a few moments...
Do you want to copy existing state to the new backend?
  Pre-existing state was found while migrating the previous "gcs" backend to the
  newly configured "local" backend. An existing non-empty state already exists in
  the new backend. The two states have been saved to temporary files that will be
  removed after responding to this query.

  Previous (type "gcs"): /tmp/terraform3978850200/1-gcs.tfstate
  New      (type "local"): /tmp/terraform3978850200/2-local.tfstate

  Do you want to overwrite the state in the new backend with the previous state?
  Enter "yes" to copy and "no" to start with the existing state in the newly
  configured "local" backend.

  Enter a value: yes


Successfully configured the backend "local"! Terraform will automatically
use this backend unless the backend configuration changes.

Initializing provider plugins...
- Reusing previous version of hashicorp/google from the dependency lock file
- Using previously-installed hashicorp/google v4.63.0

Terraform has been successfully initialized!

You may now begin working with Terraform. Try running "terraform plan" to see
any changes that are required for your infrastructure. All Terraform commands
should now work.

If you ever set or change modules or backend configuration for Terraform,
rerun this command to reinitialize your working directory. If you forget, other
commands will detect it and remind you to do so if necessary.
student_01_1996df2c1f49@cloudshell:~ (qwiklabs-gcp-00-eea2f6cd711e)$
```

In the `main.tf` file, add the `force_destroy = true` argument to your `google_storage_bucket` resource. When you delete a bucket, this boolean option will [delete all contained objects](https://www.terraform.io/docs/providers/google/r/storage_bucket#force_destroy). If you try to delete a bucket that contains objects, Terraform will  fail that run. 

```sh
student_01_1996df2c1f49@cloudshell:~ (qwiklabs-gcp-00-eea2f6cd711e)$ terraform apply
google_storage_bucket.test-bucket-for-state: Refreshing state... [id=qwiklabs-gcp-00-eea2f6cd711e]

Terraform used the selected providers to generate the following execution plan. Resource actions are indicated with the following symbols:
  ~ update in-place

Terraform will perform the following actions:

  # google_storage_bucket.test-bucket-for-state will be updated in-place
  ~ resource "google_storage_bucket" "test-bucket-for-state" {
        id                          = "qwiklabs-gcp-00-eea2f6cd711e"
      ~ labels                      = {
          - "key" = "value" -> null
        }
        name                        = "qwiklabs-gcp-00-eea2f6cd711e"
        # (10 unchanged attributes hidden)
    }

Plan: 0 to add, 1 to change, 0 to destroy.

Do you want to perform these actions?
  Terraform will perform the actions described above.
  Only 'yes' will be accepted to approve.

  Enter a value: yes

google_storage_bucket.test-bucket-for-state: Modifying... [id=qwiklabs-gcp-00-eea2f6cd711e]
google_storage_bucket.test-bucket-for-state: Modifications complete after 0s [id=qwiklabs-gcp-00-eea2f6cd711e]

Apply complete! Resources: 0 added, 1 changed, 0 destroyed.
student_01_1996df2c1f49@cloudshell:~ (qwiklabs-gcp-00-eea2f6cd711e)$ terraform destroy
google_storage_bucket.test-bucket-for-state: Refreshing state... [id=qwiklabs-gcp-00-eea2f6cd711e]

Terraform used the selected providers to generate the following execution plan. Resource actions are indicated with the following symbols:
  - destroy

Terraform will perform the following actions:

  # google_storage_bucket.test-bucket-for-state will be destroyed
  - resource "google_storage_bucket" "test-bucket-for-state" {
      - default_event_based_hold    = false -> null
      - force_destroy               = false -> null
      - id                          = "qwiklabs-gcp-00-eea2f6cd711e" -> null
      - labels                      = {} -> null
      - location                    = "US" -> null
      - name                        = "qwiklabs-gcp-00-eea2f6cd711e" -> null
      - project                     = "qwiklabs-gcp-00-eea2f6cd711e" -> null
      - public_access_prevention    = "inherited" -> null
      - requester_pays              = false -> null
      - self_link                   = "https://www.googleapis.com/storage/v1/b/qwiklabs-gcp-00-eea2f6cd711e" -> null
      - storage_class               = "STANDARD" -> null
      - uniform_bucket_level_access = true -> null
      - url                         = "gs://qwiklabs-gcp-00-eea2f6cd711e" -> null
    }

Plan: 0 to add, 0 to change, 1 to destroy.

Do you really want to destroy all resources?
  Terraform will destroy all your managed infrastructure, as shown above.
  There is no undo. Only 'yes' will be accepted to confirm.

  Enter a value: yes

google_storage_bucket.test-bucket-for-state: Destroying... [id=qwiklabs-gcp-00-eea2f6cd711e]
╷
│ Error: Error trying to delete bucket qwiklabs-gcp-00-eea2f6cd711e containing objects without `force_destroy` set to true
│
│
╵
student_01_1996df2c1f49@cloudshell:~ (qwiklabs-gcp-00-eea2f6cd711e)$
```



After setting `force_destroy = true` Your resource configuration should resemble the  following:

```sh
student_01_1996df2c1f49@cloudshell:~ (qwiklabs-gcp-00-eea2f6cd711e)$ cat main.tf
provider "google" {
  project     = "qwiklabs-gcp-00-eea2f6cd711e"
  region      = "us-central-1"
}
resource "google_storage_bucket" "test-bucket-for-state" {
  name        = "qwiklabs-gcp-00-eea2f6cd711e"
  location    = "US"
  uniform_bucket_level_access = true
  force_destroy = true
}
terraform {
  backend "local" {
    path = "terraform/state/terraform.tfstate"
  }
}student_01_1996df2c1f49@cloudshell:~ (qwiklabs-gcp-00-eea2f6cd711e)$
```

```sh
student_01_1996df2c1f49@cloudshell:~ (qwiklabs-gcp-00-eea2f6cd711e)$ terraform apply
google_storage_bucket.test-bucket-for-state: Refreshing state... [id=qwiklabs-gcp-00-eea2f6cd711e]

Terraform used the selected providers to generate the following execution plan. Resource actions are indicated with the following symbols:
  ~ update in-place

Terraform will perform the following actions:

  # google_storage_bucket.test-bucket-for-state will be updated in-place
  ~ resource "google_storage_bucket" "test-bucket-for-state" {
      ~ force_destroy               = false -> true
        id                          = "qwiklabs-gcp-00-eea2f6cd711e"
        name                        = "qwiklabs-gcp-00-eea2f6cd711e"
        # (10 unchanged attributes hidden)
    }

Plan: 0 to add, 1 to change, 0 to destroy.

Do you want to perform these actions?
  Terraform will perform the actions described above.
  Only 'yes' will be accepted to approve.

  Enter a value: yes

google_storage_bucket.test-bucket-for-state: Modifying... [id=qwiklabs-gcp-00-eea2f6cd711e]
google_storage_bucket.test-bucket-for-state: Modifications complete after 1s [id=qwiklabs-gcp-00-eea2f6cd711e]

Apply complete! Resources: 0 added, 1 changed, 0 destroyed.
student_01_1996df2c1f49@cloudshell:~ (qwiklabs-gcp-00-eea2f6cd711e)$ terraform destroy
google_storage_bucket.test-bucket-for-state: Refreshing state... [id=qwiklabs-gcp-00-eea2f6cd711e]

Terraform used the selected providers to generate the following execution plan. Resource actions are indicated with the following symbols:
  - destroy

Terraform will perform the following actions:

  # google_storage_bucket.test-bucket-for-state will be destroyed
  - resource "google_storage_bucket" "test-bucket-for-state" {
      - default_event_based_hold    = false -> null
      - force_destroy               = true -> null
      - id                          = "qwiklabs-gcp-00-eea2f6cd711e" -> null
      - labels                      = {} -> null
      - location                    = "US" -> null
      - name                        = "qwiklabs-gcp-00-eea2f6cd711e" -> null
      - project                     = "qwiklabs-gcp-00-eea2f6cd711e" -> null
      - public_access_prevention    = "inherited" -> null
      - requester_pays              = false -> null
      - self_link                   = "https://www.googleapis.com/storage/v1/b/qwiklabs-gcp-00-eea2f6cd711e" -> null
      - storage_class               = "STANDARD" -> null
      - uniform_bucket_level_access = true -> null
      - url                         = "gs://qwiklabs-gcp-00-eea2f6cd711e" -> null
    }

Plan: 0 to add, 0 to change, 1 to destroy.

Do you really want to destroy all resources?
  Terraform will destroy all your managed infrastructure, as shown above.
  There is no undo. Only 'yes' will be accepted to confirm.

  Enter a value: yes

google_storage_bucket.test-bucket-for-state: Destroying... [id=qwiklabs-gcp-00-eea2f6cd711e]
google_storage_bucket.test-bucket-for-state: Destruction complete after 2s

Destroy complete! Resources: 1 destroyed.
student_01_1996df2c1f49@cloudshell:~ (qwiklabs-gcp-00-eea2f6cd711e)$
```

## 2. Import Terraform configuration

In this section, you will import an existing Docker container and  image into an empty Terraform workspace. By doing so, you will learn  strategies and considerations for importing real-world infrastructure  into Terraform.

The default Terraform workflow involves creating and managing infrastructure entirely with Terraform.

- Write a Terraform configuration that defines the infrastructure you want to create.
- Review the Terraform plan to ensure that the configuration will result in the expected state and infrastructure.
- Apply the configuration to create your Terraform state and infrastructure.

After you create infrastructure with Terraform, you can update the  configuration and plan and apply those changes. Eventually you will use  Terraform to destroy the infrastructure when it is no longer needed.  This workflow assumes that Terraform will create an entirely new  infrastructure.

However, you may need to manage infrastructure that wasn’t created by Terraform. Terraform import solves this problem by loading supported  resources into your Terraform workspace’s state.

The import command doesn’t automatically generate the configuration  to manage the infrastructure, though. Because of this, importing  existing infrastructure into Terraform is a multi-step process.



Bringing existing infrastructure under Terraform’s control involves five main steps:

- Identify the existing infrastructure to be imported.
- Import the infrastructure into your Terraform state.
- Write a Terraform configuration that matches that infrastructure.
- Review the Terraform plan to ensure that the configuration matches the expected state and infrastructure.
- Apply the configuration to update your Terraform state.



In this section, first you will create a Docker container with the  Docker CLI. Next, you will import it into a new Terraform workspace.  Then you will update the container’s configuration using Terraform  before finally destroying it when you are done.



### Create a Docker container

1. Create a container named `hashicorp-learn` using the latest NGINX image from Docker Hub, and preview the container on the Cloud Shell virtual machine over port 80 (HTTP):

```sh
student_01_1996df2c1f49@cloudshell:~ (qwiklabs-gcp-00-eea2f6cd711e)$ docker run --name hashicorp-learn --detach --publish 8080:80 nginx:latest
Unable to find image 'nginx:latest' locally
latest: Pulling from library/nginx
26c5c85e47da: Pull complete
4f3256bdf66b: Pull complete
2019c71d5655: Pull complete
8c767bdbc9ae: Pull complete
78e14bb05fd3: Pull complete
75576236abf5: Pull complete
Digest: sha256:63b44e8ddb83d5dd8020327c1f40436e37a6fffd3ef2498a6204df23be6e7e94
Status: Downloaded newer image for nginx:latest
c09fcd6d528b0ebbad44702a6fbe18205d0f49975de113adcb20cd3a966d5684
student_01_1996df2c1f49@cloudshell:~ (qwiklabs-gcp-00-eea2f6cd711e)$ docker ps
CONTAINER ID   IMAGE          COMMAND                  CREATED         STATUS         PORTS                  NAMES
c09fcd6d528b   nginx:latest   "/docker-entrypoint.…"   4 seconds ago   Up 3 seconds   0.0.0.0:8080->80/tcp   hashicorp-learn
student_01_1996df2c1f49@cloudshell:~ (qwiklabs-gcp-00-eea2f6cd711e)$
```

Now you have a Docker image and container to import into your workspace and manage with Terraform.

Now you have a Docker image and container to import into your workspace and manage with Terraform.



```sh
student_01_1996df2c1f49@cloudshell:~ (qwiklabs-gcp-00-eea2f6cd711e)$ git clone https://github.com/hashicorp/learn-terraform-import.git
Cloning into 'learn-terraform-import'...
remote: Enumerating objects: 113, done.
remote: Counting objects: 100% (40/40), done.
remote: Compressing objects: 100% (25/25), done.
remote: Total 113 (delta 20), reused 22 (delta 15), pack-reused 73
Receiving objects: 100% (113/113), 38.01 KiB | 9.50 MiB/s, done.
Resolving deltas: 100% (45/45), done.
student_01_1996df2c1f49@cloudshell:~ (qwiklabs-gcp-00-eea2f6cd711e)$ cd learn-terraform-import
student_01_1996df2c1f49@cloudshell:~/learn-terraform-import (qwiklabs-gcp-00-eea2f6cd711e)$ ls
docker.tf  LICENSE  main.tf  README.md  terraform.tf
student_01_1996df2c1f49@cloudshell:~/learn-terraform-import (qwiklabs-gcp-00-eea2f6cd711e)$ cat main.tf
# Copyright (c) HashiCorp, Inc.
# SPDX-License-Identifier: MPL-2.0

provider "docker" {
  host = "unix:///var/run/docker.sock"
}
student_01_1996df2c1f49@cloudshell:~/learn-terraform-import (qwiklabs-gcp-00-eea2f6cd711e)$ cat docker.tf
# Copyright (c) HashiCorp, Inc.
# SPDX-License-Identifier: MPL-2.0

# Terraform configuration
student_01_1996df2c1f49@cloudshell:~/learn-terraform-import (qwiklabs-gcp-00-eea2f6cd711e)$ cat terraform.tf
# Copyright (c) HashiCorp, Inc.
# SPDX-License-Identifier: MPL-2.0

terraform {
  /* Uncomment this block to use Terraform Cloud for this tutorial
  cloud {
      organization = "organization-name"
      workspaces {
        name = "learn-terraform-import"
      }
  }
  */

  required_providers {
    docker = {
      source  = "kreuzwerker/docker"
      version = "~> 2.17.0"
    }
  }

  required_version = "~> 1.2"
}
student_01_1996df2c1f49@cloudshell:~/learn-terraform-import (qwiklabs-gcp-00-eea2f6cd711e)$
```

This directory contains two Terraform configuration files that make up the configuration you will use in this guide:

- `main.tf` file configures the Docker provider.
- `docker.tf` file will contain the configuration necessary to manage the Docker container you created in an earlier step.

1. Initialize your Terraform workspace:

```sh
student_01_1996df2c1f49@cloudshell:~/learn-terraform-import (qwiklabs-gcp-00-eea2f6cd711e)$ terraform init

Initializing the backend...

Initializing provider plugins...
- Reusing previous version of kreuzwerker/docker from the dependency lock file
- Installing kreuzwerker/docker v2.17.0...
- Installed kreuzwerker/docker v2.17.0 (self-signed, key ID BD080C4571C6104C)

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
student_01_1996df2c1f49@cloudshell:~/learn-terraform-import (qwiklabs-gcp-00-eea2f6cd711e)$
```



Find the `provider: docker` resource and **comment out or delete** the `host` argument:

1. Next, navigate to `learn-terraform-import/docker.tf`.
2. Under the commented-out code, define an empty `docker_container` resource in your `docker.tf` file, which represents a Docker container with the Terraform resource ID `docker_container.web`:

```sh
student_01_1996df2c1f49@cloudshell:~/learn-terraform-import (qwiklabs-gcp-00-eea2f6cd711e)$ cat main.tf
# Copyright (c) HashiCorp, Inc.
# SPDX-License-Identifier: MPL-2.0

provider "docker" {
  #host = "unix:///var/run/docker.sock"
}
student_01_1996df2c1f49@cloudshell:~/learn-terraform-import (qwiklabs-gcp-00-eea2f6cd711e)$ cat docker.tf
# Copyright (c) HashiCorp, Inc.
# SPDX-License-Identifier: MPL-2.0

# Terraform configuration
resource "docker_container" "web" {}

student_01_1996df2c1f49@cloudshell:~/learn-terraform-import (qwiklabs-gcp-00-eea2f6cd711e)$
```

```sh
student_01_1996df2c1f49@cloudshell:~/learn-terraform-import (qwiklabs-gcp-00-eea2f6cd711e)$ docker ps
CONTAINER ID   IMAGE          COMMAND                  CREATED         STATUS         PORTS                  NAMES
c09fcd6d528b   nginx:latest   "/docker-entrypoint.…"   5 minutes ago   Up 5 minutes   0.0.0.0:8080->80/tcp   hashicorp-learn
student_01_1996df2c1f49@cloudshell:~/learn-terraform-import (qwiklabs-gcp-00-eea2f6cd711e)$ terraform import docker_container.web $(docker inspect -f {{.ID}} hashicorp-learn)
docker_container.web: Importing from ID "c09fcd6d528b0ebbad44702a6fbe18205d0f49975de113adcb20cd3a966d5684"...
docker_container.web: Import prepared!
  Prepared docker_container for import
docker_container.web: Refreshing state... [id=c09fcd6d528b0ebbad44702a6fbe18205d0f49975de113adcb20cd3a966d5684]

Import successful!

The resources that were imported are shown above. These resources are now in
your Terraform state and will henceforth be managed by Terraform.

student_01_1996df2c1f49@cloudshell:~/learn-terraform-import (qwiklabs-gcp-00-eea2f6cd711e)$
```

```sh
student_01_1996df2c1f49@cloudshell:~/learn-terraform-import (qwiklabs-gcp-00-eea2f6cd711e)$ terraform show
# docker_container.web:
resource "docker_container" "web" {
    command           = [
        "nginx",
        "-g",
        "daemon off;",
    ]
    cpu_shares        = 0
    dns               = []
    dns_opts          = []
    dns_search        = []
    entrypoint        = [
        "/docker-entrypoint.sh",
    ]
    gateway           = "172.18.0.1"
    group_add         = []
    hostname          = "c09fcd6d528b"
    id                = "c09fcd6d528b0ebbad44702a6fbe18205d0f49975de113adcb20cd3a966d5684"
    image             = "sha256:6efc10a0510f143a90b69dc564a914574973223e88418d65c1f8809e08dc0a1f"
    init              = false
    ip_address        = "172.18.0.2"
    ip_prefix_length  = 16
    ipc_mode          = "private"
    links             = []
    log_driver        = "json-file"
    log_opts          = {}
    max_retry_count   = 0
    memory            = 0
    memory_swap       = 0
    name              = "hashicorp-learn"
    network_data      = [
        {
            gateway                   = "172.18.0.1"
            global_ipv6_address       = ""
            global_ipv6_prefix_length = 0
            ip_address                = "172.18.0.2"
            ip_prefix_length          = 16
            ipv6_gateway              = ""
            network_name              = "bridge"
        },
    ]
    network_mode      = "default"
    privileged        = false
    publish_all_ports = false
    read_only         = false
    restart           = "no"
    rm                = false
    security_opts     = []
    shm_size          = 64
    stdin_open        = false
    storage_opts      = {}
    sysctls           = {}
    tmpfs             = {}
    tty               = false

    ports {
        external = 8080
        internal = 80
        ip       = "0.0.0.0"
        protocol = "tcp"
    }
}
student_01_1996df2c1f49@cloudshell:~/learn-terraform-import (qwiklabs-gcp-00-eea2f6cd711e)$
```

### Create configuration

You’ll need to create Terraform configuration before you can use Terraform to manage this container.

1. Run the following code:

```sh
student_01_1996df2c1f49@cloudshell:~/learn-terraform-import (qwiklabs-gcp-00-eea2f6cd711e)$ terraform plan
╷
│ Error: Missing required argument
│
│   on docker.tf line 5, in resource "docker_container" "web":
│    5: resource "docker_container" "web" {}
│
│ The argument "image" is required, but no definition was found.
╵
╷
│ Error: Missing required argument
│
│   on docker.tf line 5, in resource "docker_container" "web":
│    5: resource "docker_container" "web" {}
│
│ The argument "name" is required, but no definition was found.
╵
student_01_1996df2c1f49@cloudshell:~/learn-terraform-import (qwiklabs-gcp-00-eea2f6cd711e)$
```

There are two approaches to update the configuration in `docker.tf` to match the state you imported. You can either accept the entire  current state of the resource into your configuration as-is or select  the required attributes into your configuration individually. Each of  these approaches can be useful in different circumstances.

- Using the current state is often faster, but can result in an overly  verbose configuration because every attribute is included in the state,  whether it is necessary to include in your configuration or not.
- Individually selecting the required attributes can lead to more  manageable configuration, but requires you to understand which  attributes need to be set in the configuration.

For this lab's purposes, you will use the current state as the resource.

```sh
student_01_1996df2c1f49@cloudshell:~/learn-terraform-import (qwiklabs-gcp-00-eea2f6cd711e)$ terraform show -no-color > docker.tf
student_01_1996df2c1f49@cloudshell:~/learn-terraform-import (qwiklabs-gcp-00-eea2f6cd711e)$ cat docker.tf
# docker_container.web:
resource "docker_container" "web" {
    command           = [
        "nginx",
        "-g",
        "daemon off;",
    ]
    cpu_shares        = 0
    dns               = []
    dns_opts          = []
    dns_search        = []
    entrypoint        = [
        "/docker-entrypoint.sh",
    ]
    gateway           = "172.18.0.1"
    group_add         = []
    hostname          = "c09fcd6d528b"
    id                = "c09fcd6d528b0ebbad44702a6fbe18205d0f49975de113adcb20cd3a966d5684"
    image             = "sha256:6efc10a0510f143a90b69dc564a914574973223e88418d65c1f8809e08dc0a1f"
    init              = false
    ip_address        = "172.18.0.2"
    ip_prefix_length  = 16
    ipc_mode          = "private"
    links             = []
    log_driver        = "json-file"
    log_opts          = {}
    max_retry_count   = 0
    memory            = 0
    memory_swap       = 0
    name              = "hashicorp-learn"
    network_data      = [
        {
            gateway                   = "172.18.0.1"
            global_ipv6_address       = ""
            global_ipv6_prefix_length = 0
            ip_address                = "172.18.0.2"
            ip_prefix_length          = 16
            ipv6_gateway              = ""
            network_name              = "bridge"
        },
    ]
    network_mode      = "default"
    privileged        = false
    publish_all_ports = false
    read_only         = false
    restart           = "no"
    rm                = false
    security_opts     = []
    shm_size          = 64
    stdin_open        = false
    storage_opts      = {}
    sysctls           = {}
    tmpfs             = {}
    tty               = false

    ports {
        external = 8080
        internal = 80
        ip       = "0.0.0.0"
        protocol = "tcp"
    }
}
student_01_1996df2c1f49@cloudshell:~/learn-terraform-import (qwiklabs-gcp-00-eea2f6cd711e)$
```

```sh
student_01_1996df2c1f49@cloudshell:~/learn-terraform-import (qwiklabs-gcp-00-eea2f6cd711e)$ terraform plan
╷
│ Warning: Argument is deprecated
│
│   with docker_container.web,
│   on docker.tf line 24, in resource "docker_container" "web":
│   24:     links             = []
│
│ The --link flag is a legacy feature of Docker. It may eventually be removed.
╵
╷
│ Error: Value for unconfigurable attribute
│
│   with docker_container.web,
│   on docker.tf line 15, in resource "docker_container" "web":
│   15:     gateway           = "172.18.0.1"
│
│ Can't configure a value for "gateway": its value will be decided automatically based on the result of applying this configuration.
╵
╷
│ Error: Invalid or unknown key
│
│   with docker_container.web,
│   on docker.tf line 18, in resource "docker_container" "web":
│   18:     id                = "c09fcd6d528b0ebbad44702a6fbe18205d0f49975de113adcb20cd3a966d5684"
│
╵
╷
│ Error: Value for unconfigurable attribute
│
│   with docker_container.web,
│   on docker.tf line 21, in resource "docker_container" "web":
│   21:     ip_address        = "172.18.0.2"
│
│ Can't configure a value for "ip_address": its value will be decided automatically based on the result of applying this configuration.
╵
╷
│ Error: Value for unconfigurable attribute
│
│   with docker_container.web,
│   on docker.tf line 22, in resource "docker_container" "web":
│   22:     ip_prefix_length  = 16
│
│ Can't configure a value for "ip_prefix_length": its value will be decided automatically based on the result of applying this configuration.
╵
╷
│ Error: Value for unconfigurable attribute
│
│   with docker_container.web,
│   on docker.tf line 31, in resource "docker_container" "web":
│   31:     network_data      = [
│   32:         {
│   33:             gateway                   = "172.18.0.1"
│   34:             global_ipv6_address       = ""
│   35:             global_ipv6_prefix_length = 0
│   36:             ip_address                = "172.18.0.2"
│   37:             ip_prefix_length          = 16
│   38:             ipv6_gateway              = ""
│   39:             network_name              = "bridge"
│   40:         },
│   41:     ]
│
│ Can't configure a value for "network_data": its value will be decided automatically based on the result of applying this configuration.
╵
student_01_1996df2c1f49@cloudshell:~/learn-terraform-import (qwiklabs-gcp-00-eea2f6cd711e)$
```

Terraform will show warnings and errors about a deprecated argument ('links'), and several read-only arguments (`ip_address`, `network_data`, `gateway`, `ip_prefix_length`, `id`).

These read-only arguments are values that Terraform stores in its  state for Docker containers but that it cannot set via configuration  because they are managed internally by Docker. Terraform can set the  links argument with configuration, but still displays a warning because  it is deprecated and might not be supported by future versions of the  Docker provider.

Because the approach shown here loads all of the attributes  represented in Terraform state, your configuration includes optional  attributes whose values are the same as their defaults. Which attributes are optional, and their default values, will vary from provider to  provider, and are listed in the [provider documentation](https://www.terraform.io/docs/providers/docker/r/container).

You can now selectively remove these optional attributes. **Remove** all of these attributes, *keeping only the required attributes*: `image`, `name`, and `ports`. After removing these optional attributes, your configuration should match the following:

```sh
student_01_1996df2c1f49@cloudshell:~/learn-terraform-import (qwiklabs-gcp-00-eea2f6cd711e)$ cat docker.tf
# docker_container.web:
resource "docker_container" "web" {

    image             = "sha256:6efc10a0510f143a90b69dc564a914574973223e88418d65c1f8809e08dc0a1f"

    name              = "hashicorp-learn"

    ports {
        external = 8080
        internal = 80
        ip       = "0.0.0.0"
        protocol = "tcp"
    }
}
student_01_1996df2c1f49@cloudshell:~/learn-terraform-import (qwiklabs-gcp-00-eea2f6cd711e)$
```

```sh
student_01_1996df2c1f49@cloudshell:~/learn-terraform-import (qwiklabs-gcp-00-eea2f6cd711e)$ terraform plan
docker_container.web: Refreshing state... [id=c09fcd6d528b0ebbad44702a6fbe18205d0f49975de113adcb20cd3a966d5684]

Terraform used the selected providers to generate the following execution plan. Resource actions are indicated with the following symbols:
-/+ destroy and then create replacement

Terraform will perform the following actions:

  # docker_container.web must be replaced
-/+ resource "docker_container" "web" {
      + attach            = false
      + bridge            = (known after apply)
      ~ command           = [
          - "nginx",
          - "-g",
          - "daemon off;",
        ] -> (known after apply)
      + container_logs    = (known after apply)
      - cpu_shares        = 0 -> null
      - dns               = [] -> null
      - dns_opts          = [] -> null
      - dns_search        = [] -> null
      ~ entrypoint        = [
          - "/docker-entrypoint.sh",
        ] -> (known after apply)
      + env               = (known after apply)
      + exit_code         = (known after apply)
      ~ gateway           = "172.18.0.1" -> (known after apply)
      - group_add         = [] -> null
      ~ hostname          = "c09fcd6d528b" -> (known after apply)
      ~ id                = "c09fcd6d528b0ebbad44702a6fbe18205d0f49975de113adcb20cd3a966d5684" -> (known after apply)
      ~ init              = false -> (known after apply)
      ~ ip_address        = "172.18.0.2" -> (known after apply)
      ~ ip_prefix_length  = 16 -> (known after apply)
      ~ ipc_mode          = "private" -> (known after apply)
      - links             = [] -> null
      ~ log_driver        = "json-file" -> (known after apply)
      - log_opts          = {} -> null
      + logs              = false
      - max_retry_count   = 0 -> null
      - memory            = 0 -> null
      - memory_swap       = 0 -> null
      + must_run          = true
        name              = "hashicorp-learn"
      ~ network_data      = [
          - {
              - gateway                   = "172.18.0.1"
              - global_ipv6_address       = ""
              - global_ipv6_prefix_length = 0
              - ip_address                = "172.18.0.2"
              - ip_prefix_length          = 16
              - ipv6_gateway              = ""
              - network_name              = "bridge"
            },
        ] -> (known after apply)
      - network_mode      = "default" -> null
      - privileged        = false -> null
      - publish_all_ports = false -> null
      + remove_volumes    = true
      ~ security_opts     = [] -> (known after apply)
      ~ shm_size          = 64 -> (known after apply)
      + start             = true
      - storage_opts      = {} -> null
      - sysctls           = {} -> null
      - tmpfs             = {} -> null
        # (6 unchanged attributes hidden)

        # (1 unchanged block hidden)
    }

Plan: 1 to add, 0 to change, 1 to destroy.

───────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────

Note: You didn't use the -out option to save this plan, so Terraform can't guarantee to take exactly these actions if you run "terraform apply" now.
student_01_1996df2c1f49@cloudshell:~/learn-terraform-import (qwiklabs-gcp-00-eea2f6cd711e)$
```

The plan should now execute successfully. Notice that the plan indicates that Terraform will update the container to add the `attach`, `logs`, `must_run`, and `start` attributes.

Terraform uses these attributes to create Docker containers, but Docker doesn’t store them. As a result, `terraform import` didn’t load their values into state. When you plan and apply your  configuration, the Docker provider will assign the default values for  these attributes and save them in state, but they won’t affect the  running container.

1. Apply the changes and finish the process of syncing your updated  Terraform configuration and state with the Docker container they  represent. Type **yes** at the prompt to confirm.

```sh
student_01_1996df2c1f49@cloudshell:~/learn-terraform-import (qwiklabs-gcp-00-eea2f6cd711e)$ terraform apply
docker_container.web: Refreshing state... [id=c09fcd6d528b0ebbad44702a6fbe18205d0f49975de113adcb20cd3a966d5684]

Terraform used the selected providers to generate the following execution plan. Resource actions are indicated with the following symbols:
-/+ destroy and then create replacement

Terraform will perform the following actions:

  # docker_container.web must be replaced
-/+ resource "docker_container" "web" {
      + attach            = false
      + bridge            = (known after apply)
      ~ command           = [
          - "nginx",
          - "-g",
          - "daemon off;",
        ] -> (known after apply)
      + container_logs    = (known after apply)
      - cpu_shares        = 0 -> null
      - dns               = [] -> null
      - dns_opts          = [] -> null
      - dns_search        = [] -> null
      ~ entrypoint        = [
          - "/docker-entrypoint.sh",
        ] -> (known after apply)
      + env               = (known after apply)
      + exit_code         = (known after apply)
      ~ gateway           = "172.18.0.1" -> (known after apply)
      - group_add         = [] -> null
      ~ hostname          = "c09fcd6d528b" -> (known after apply)
      ~ id                = "c09fcd6d528b0ebbad44702a6fbe18205d0f49975de113adcb20cd3a966d5684" -> (known after apply)
      ~ init              = false -> (known after apply)
      ~ ip_address        = "172.18.0.2" -> (known after apply)
      ~ ip_prefix_length  = 16 -> (known after apply)
      ~ ipc_mode          = "private" -> (known after apply)
      - links             = [] -> null
      ~ log_driver        = "json-file" -> (known after apply)
      - log_opts          = {} -> null
      + logs              = false
      - max_retry_count   = 0 -> null
      - memory            = 0 -> null
      - memory_swap       = 0 -> null
      + must_run          = true
        name              = "hashicorp-learn"
      ~ network_data      = [
          - {
              - gateway                   = "172.18.0.1"
              - global_ipv6_address       = ""
              - global_ipv6_prefix_length = 0
              - ip_address                = "172.18.0.2"
              - ip_prefix_length          = 16
              - ipv6_gateway              = ""
              - network_name              = "bridge"
            },
        ] -> (known after apply)
      - network_mode      = "default" -> null
      - privileged        = false -> null
      - publish_all_ports = false -> null
      + remove_volumes    = true
      ~ security_opts     = [] -> (known after apply)
      ~ shm_size          = 64 -> (known after apply)
      + start             = true
      - storage_opts      = {} -> null
      - sysctls           = {} -> null
      - tmpfs             = {} -> null
        # (6 unchanged attributes hidden)

        # (1 unchanged block hidden)
    }

Plan: 1 to add, 0 to change, 1 to destroy.

Do you want to perform these actions?
  Terraform will perform the actions described above.
  Only 'yes' will be accepted to approve.

  Enter a value: yes

docker_container.web: Destroying... [id=c09fcd6d528b0ebbad44702a6fbe18205d0f49975de113adcb20cd3a966d5684]
docker_container.web: Destruction complete after 0s
docker_container.web: Creating...
docker_container.web: Creation complete after 0s [id=4f4197de7d98397da72e04fc3e3f7c2112f3b4e5f7f34638289fd21f74a52562]

Apply complete! Resources: 1 added, 0 changed, 1 destroyed.
student_01_1996df2c1f49@cloudshell:~/learn-terraform-import (qwiklabs-gcp-00-eea2f6cd711e)$
```

Now your configuration file, Terraform state, and the container are  all in sync, and you can use Terraform to manage the Terraform container as you normally would.

### Create image resource

In some cases, you can bring resources under Terraform's control without using the `terraform import` command. This is often the case for resources that are defined by a single unique ID or tag, such as Docker images.

In your `docker.tf` file, the `docker_container.web` resource specifies the SHA256 hash ID of the image used to create the  container. This is how docker stores the image ID internally, and so `terraform import` loaded the image ID directly into your state. However the image ID is  not as human readable as the image tag or name, and it may not match  your intent. For example, you might want to use the latest version of  the "nginx" image.



To retrieve the image's tag name, run the following command, replacing `<IMAGE-ID>` with the image ID from `docker.tf`:



```sh
student_01_1996df2c1f49@cloudshell:~/learn-terraform-import (qwiklabs-gcp-00-eea2f6cd711e)$ docker image inspect 6efc10a0510f143a90b69dc564a914574973223e88418d65c1f8809e08dc0a1f -f {{.RepoTags}}
[nginx:latest]
student_01_1996df2c1f49@cloudshell:~/learn-terraform-import (qwiklabs-gcp-00-eea2f6cd711e)$
```

```sh
student_01_1996df2c1f49@cloudshell:~/learn-terraform-import (qwiklabs-gcp-00-eea2f6cd711e)$ cat docker.tf
# docker_container.web:
resource "docker_container" "web" {

    image             = "sha256:6efc10a0510f143a90b69dc564a914574973223e88418d65c1f8809e08dc0a1f"

    name              = "hashicorp-learn"

    ports {
        external = 8080
        internal = 80
        ip       = "0.0.0.0"
        protocol = "tcp"
    }
}
resource "docker_image" "nginx" {
  name         = "nginx:latest"
}
student_01_1996df2c1f49@cloudshell:~/learn-terraform-import (qwiklabs-gcp-00-eea2f6cd711e)$ terraform apply
docker_container.web: Refreshing state... [id=4f4197de7d98397da72e04fc3e3f7c2112f3b4e5f7f34638289fd21f74a52562]

Terraform used the selected providers to generate the following execution plan. Resource actions are indicated with the following symbols:
  + create

Terraform will perform the following actions:

  # docker_image.nginx will be created
  + resource "docker_image" "nginx" {
      + id          = (known after apply)
      + latest      = (known after apply)
      + name        = "nginx:latest"
      + output      = (known after apply)
      + repo_digest = (known after apply)
    }

Plan: 1 to add, 0 to change, 0 to destroy.

Do you want to perform these actions?
  Terraform will perform the actions described above.
  Only 'yes' will be accepted to approve.

  Enter a value: yes

docker_image.nginx: Creating...
docker_image.nginx: Creation complete after 0s [id=sha256:6efc10a0510f143a90b69dc564a914574973223e88418d65c1f8809e08dc0a1fnginx:latest]

Apply complete! Resources: 1 added, 0 changed, 0 destroyed.
student_01_1996df2c1f49@cloudshell:~/learn-terraform-import (qwiklabs-gcp-00-eea2f6cd711e)$
```

Now that Terraform has created a resource for the image, you can reference it in your container’s configuration.

Change the image value for `docker_container.web` to reference the new image resource:

```sh
student_01_1996df2c1f49@cloudshell:~/learn-terraform-import (qwiklabs-gcp-00-eea2f6cd711e)$ cat docker.tf
# docker_container.web:
resource "docker_container" "web" {

    image = docker_image.nginx.latest

    name              = "hashicorp-learn"

    ports {
        external = 8080
        internal = 80
        ip       = "0.0.0.0"
        protocol = "tcp"
    }
}
resource "docker_image" "nginx" {
  name         = "nginx:latest"
}
student_01_1996df2c1f49@cloudshell:~/learn-terraform-import (qwiklabs-gcp-00-eea2f6cd711e)$
```

```sh
student_01_1996df2c1f49@cloudshell:~/learn-terraform-import (qwiklabs-gcp-00-eea2f6cd711e)$ terraform apply
docker_image.nginx: Refreshing state... [id=sha256:6efc10a0510f143a90b69dc564a914574973223e88418d65c1f8809e08dc0a1fnginx:latest]
docker_container.web: Refreshing state... [id=4f4197de7d98397da72e04fc3e3f7c2112f3b4e5f7f34638289fd21f74a52562]

No changes. Your infrastructure matches the configuration.

Terraform has compared your real infrastructure against your configuration and found no differences, so no changes are needed.
╷
│ Warning: Deprecated attribute
│
│   on docker.tf line 4, in resource "docker_container" "web":
│    4:     image = docker_image.nginx.latest
│
│ The attribute "latest" is deprecated. Refer to the provider documentation for details.
│
│ (and one more similar warning elsewhere)
╵

Apply complete! Resources: 0 added, 0 changed, 0 destroyed.
student_01_1996df2c1f49@cloudshell:~/learn-terraform-import (qwiklabs-gcp-00-eea2f6cd711e)$
```

Because `docker_image.nginx.latest` will match the hardcoded image ID you replaced, running `terraform apply` at this point will show no changes.

### Manage the container with Terraform

Now that Terraform manages the Docker container, use Terraform to change the configuration.

In your `docker.tf` file, change the container's external port from `8080` to `8081`:

```sh
student_01_1996df2c1f49@cloudshell:~/learn-terraform-import (qwiklabs-gcp-00-eea2f6cd711e)$ cat docker.tf
# docker_container.web:
resource "docker_container" "web" {

    image = docker_image.nginx.latest

    name              = "hashicorp-learn"

    ports {
        external = 8081
        internal = 80
        ip       = "0.0.0.0"
        protocol = "tcp"
    }
}
resource "docker_image" "nginx" {
  name         = "nginx:latest"
}
student_01_1996df2c1f49@cloudshell:~/learn-terraform-import (qwiklabs-gcp-00-eea2f6cd711e)$
```

```sh
student_01_1996df2c1f49@cloudshell:~/learn-terraform-import (qwiklabs-gcp-00-eea2f6cd711e)$ terraform apply
docker_image.nginx: Refreshing state... [id=sha256:6efc10a0510f143a90b69dc564a914574973223e88418d65c1f8809e08dc0a1fnginx:latest]
docker_container.web: Refreshing state... [id=4f4197de7d98397da72e04fc3e3f7c2112f3b4e5f7f34638289fd21f74a52562]

Terraform used the selected providers to generate the following execution plan. Resource actions are indicated with the following symbols:
-/+ destroy and then create replacement

Terraform will perform the following actions:

  # docker_container.web must be replaced
-/+ resource "docker_container" "web" {
      + bridge            = (known after apply)
      ~ command           = [
          - "nginx",
          - "-g",
          - "daemon off;",
        ] -> (known after apply)
      + container_logs    = (known after apply)
      - cpu_shares        = 0 -> null
      - dns               = [] -> null
      - dns_opts          = [] -> null
      - dns_search        = [] -> null
      ~ entrypoint        = [
          - "/docker-entrypoint.sh",
        ] -> (known after apply)
      ~ env               = [] -> (known after apply)
      + exit_code         = (known after apply)
      ~ gateway           = "172.18.0.1" -> (known after apply)
      - group_add         = [] -> null
      ~ hostname          = "4f4197de7d98" -> (known after apply)
      ~ id                = "4f4197de7d98397da72e04fc3e3f7c2112f3b4e5f7f34638289fd21f74a52562" -> (known after apply)
      ~ init              = false -> (known after apply)
      ~ ip_address        = "172.18.0.2" -> (known after apply)
      ~ ip_prefix_length  = 16 -> (known after apply)
      ~ ipc_mode          = "private" -> (known after apply)
      - links             = [] -> null
      ~ log_driver        = "json-file" -> (known after apply)
      - log_opts          = {} -> null
      - max_retry_count   = 0 -> null
      - memory            = 0 -> null
      - memory_swap       = 0 -> null
        name              = "hashicorp-learn"
      ~ network_data      = [
          - {
              - gateway                   = "172.18.0.1"
              - global_ipv6_address       = ""
              - global_ipv6_prefix_length = 0
              - ip_address                = "172.18.0.2"
              - ip_prefix_length          = 16
              - ipv6_gateway              = ""
              - network_name              = "bridge"
            },
        ] -> (known after apply)
      - network_mode      = "default" -> null
      - privileged        = false -> null
      - publish_all_ports = false -> null
      ~ security_opts     = [] -> (known after apply)
      ~ shm_size          = 64 -> (known after apply)
      - storage_opts      = {} -> null
      - sysctls           = {} -> null
      - tmpfs             = {} -> null
        # (11 unchanged attributes hidden)

      ~ ports {
          ~ external = 8080 -> 8081 # forces replacement
            # (3 unchanged attributes hidden)
        }
    }

Plan: 1 to add, 0 to change, 1 to destroy.
╷
│ Warning: Deprecated attribute
│
│   on docker.tf line 4, in resource "docker_container" "web":
│    4:     image = docker_image.nginx.latest
│
│ The attribute "latest" is deprecated. Refer to the provider documentation for details.
│
│ (and one more similar warning elsewhere)
╵

Do you want to perform these actions?
  Terraform will perform the actions described above.
  Only 'yes' will be accepted to approve.

  Enter a value: yes

docker_container.web: Destroying... [id=4f4197de7d98397da72e04fc3e3f7c2112f3b4e5f7f34638289fd21f74a52562]
docker_container.web: Destruction complete after 1s
docker_container.web: Creating...
docker_container.web: Creation complete after 0s [id=a24c607b263816317b8888a45f8a2963c86ace12228705bf0c929815d9a70b6d]
╷
│ Warning: Deprecated attribute
│
│   on docker.tf line 4, in resource "docker_container" "web":
│    4:     image = docker_image.nginx.latest
│
│ The attribute "latest" is deprecated. Refer to the provider documentation for details.
│
│ (and one more similar warning elsewhere)
╵

Apply complete! Resources: 1 added, 0 changed, 1 destroyed.
student_01_1996df2c1f49@cloudshell:~/learn-terraform-import (qwiklabs-gcp-00-eea2f6cd711e)$
```



```sh
student_01_1996df2c1f49@cloudshell:~/learn-terraform-import (qwiklabs-gcp-00-eea2f6cd711e)$ docker ps
CONTAINER ID   IMAGE          COMMAND                  CREATED          STATUS          PORTS                  NAMES
a24c607b2638   6efc10a0510f   "/docker-entrypoint.…"   18 seconds ago   Up 17 seconds   0.0.0.0:8081->80/tcp   hashicorp-learn
student_01_1996df2c1f49@cloudshell:~/learn-terraform-import (qwiklabs-gcp-00-eea2f6cd711e)$
```

Notice that the container ID has changed. Because changing the port  configuration required destroying and recreating it, this is a  completely new container.



### Destroy infrastructure

You have now imported your Docker container and the image used to create it into Terraform.

1. Destroy the container and image:

```sh
student_01_1996df2c1f49@cloudshell:~/learn-terraform-import (qwiklabs-gcp-00-eea2f6cd711e)$ terraform destroy
docker_image.nginx: Refreshing state... [id=sha256:6efc10a0510f143a90b69dc564a914574973223e88418d65c1f8809e08dc0a1fnginx:latest]
docker_container.web: Refreshing state... [id=a24c607b263816317b8888a45f8a2963c86ace12228705bf0c929815d9a70b6d]

Terraform used the selected providers to generate the following execution plan. Resource actions are indicated with the following symbols:
  - destroy

Terraform will perform the following actions:

  # docker_container.web will be destroyed
  - resource "docker_container" "web" {
      - attach            = false -> null
      - command           = [
          - "nginx",
          - "-g",
          - "daemon off;",
        ] -> null
      - cpu_shares        = 0 -> null
      - dns               = [] -> null
      - dns_opts          = [] -> null
      - dns_search        = [] -> null
      - entrypoint        = [
          - "/docker-entrypoint.sh",
        ] -> null
      - env               = [] -> null
      - gateway           = "172.18.0.1" -> null
      - group_add         = [] -> null
      - hostname          = "a24c607b2638" -> null
      - id                = "a24c607b263816317b8888a45f8a2963c86ace12228705bf0c929815d9a70b6d" -> null
      - image             = "sha256:6efc10a0510f143a90b69dc564a914574973223e88418d65c1f8809e08dc0a1f" -> null
      - init              = false -> null
      - ip_address        = "172.18.0.2" -> null
      - ip_prefix_length  = 16 -> null
      - ipc_mode          = "private" -> null
      - links             = [] -> null
      - log_driver        = "json-file" -> null
      - log_opts          = {} -> null
      - logs              = false -> null
      - max_retry_count   = 0 -> null
      - memory            = 0 -> null
      - memory_swap       = 0 -> null
      - must_run          = true -> null
      - name              = "hashicorp-learn" -> null
      - network_data      = [
          - {
              - gateway                   = "172.18.0.1"
              - global_ipv6_address       = ""
              - global_ipv6_prefix_length = 0
              - ip_address                = "172.18.0.2"
              - ip_prefix_length          = 16
              - ipv6_gateway              = ""
              - network_name              = "bridge"
            },
        ] -> null
      - network_mode      = "default" -> null
      - privileged        = false -> null
      - publish_all_ports = false -> null
      - read_only         = false -> null
      - remove_volumes    = true -> null
      - restart           = "no" -> null
      - rm                = false -> null
      - security_opts     = [] -> null
      - shm_size          = 64 -> null
      - start             = true -> null
      - stdin_open        = false -> null
      - storage_opts      = {} -> null
      - sysctls           = {} -> null
      - tmpfs             = {} -> null
      - tty               = false -> null

      - ports {
          - external = 8081 -> null
          - internal = 80 -> null
          - ip       = "0.0.0.0" -> null
          - protocol = "tcp" -> null
        }
    }

  # docker_image.nginx will be destroyed
  - resource "docker_image" "nginx" {
      - id          = "sha256:6efc10a0510f143a90b69dc564a914574973223e88418d65c1f8809e08dc0a1fnginx:latest" -> null
      - latest      = "sha256:6efc10a0510f143a90b69dc564a914574973223e88418d65c1f8809e08dc0a1f" -> null
      - name        = "nginx:latest" -> null
      - repo_digest = "nginx@sha256:63b44e8ddb83d5dd8020327c1f40436e37a6fffd3ef2498a6204df23be6e7e94" -> null
    }

Plan: 0 to add, 0 to change, 2 to destroy.
╷
│ Warning: Deprecated attribute
│
│   on docker.tf line 4, in resource "docker_container" "web":
│    4:     image = docker_image.nginx.latest
│
│ The attribute "latest" is deprecated. Refer to the provider documentation for details.
╵

Do you really want to destroy all resources?
  Terraform will destroy all your managed infrastructure, as shown above.
  There is no undo. Only 'yes' will be accepted to confirm.

  Enter a value: yes

docker_container.web: Destroying... [id=a24c607b263816317b8888a45f8a2963c86ace12228705bf0c929815d9a70b6d]
docker_container.web: Destruction complete after 0s
docker_image.nginx: Destroying... [id=sha256:6efc10a0510f143a90b69dc564a914574973223e88418d65c1f8809e08dc0a1fnginx:latest]
docker_image.nginx: Destruction complete after 0s

Destroy complete! Resources: 2 destroyed.
student_01_1996df2c1f49@cloudshell:~/learn-terraform-import (qwiklabs-gcp-00-eea2f6cd711e)$
```

```sh
student_01_1996df2c1f49@cloudshell:~/learn-terraform-import (qwiklabs-gcp-00-eea2f6cd711e)$ docker ps --filter "name=hashicorp-learn"
CONTAINER ID   IMAGE     COMMAND   CREATED   STATUS    PORTS     NAMES
student_01_1996df2c1f49@cloudshell:~/learn-terraform-import (qwiklabs-gcp-00-eea2f6cd711e)$
```

## Limitations and other considerations

There are several important things to consider when importing resources into Terraform.

Terraform import can only know the current state of infrastructure as reported by the Terraform provider. It does not know:

- Whether the infrastructure is working correctly.
- The intent of the infrastructure.
- Changes you've made to the infrastructure that aren't controlled by  Terraform; for example, the state of a Docker container's filesystem.

Importing involves manual steps which can be error-prone, especially  if the person importing resources lacks the context of how and why those resources were created originally.

Importing manipulates the Terraform state file; you may want to create a backup before importing new infrastructure.

Terraform import doesn’t detect or generate relationships between infrastructure.

Terraform doesn’t detect default attributes that don’t need to be set in your configuration.

Not all providers and resources support Terraform import.



mporting infrastructure into Terraform does not mean that it can be  destroyed and recreated by Terraform. For example, the imported  infrastructure could rely on other unmanaged infrastructure or  configuration.

Following infrastructure as code (IaC) best practices such as [immutable infrastructure](https://www.hashicorp.com/resources/what-is-mutable-vs-immutable-infrastructure) can help prevent many of these problems, but infrastructure created manually is unlikely to follow IaC best practices.

Tools such as [Terraformer](https://github.com/GoogleCloudPlatform/terraformer) can automate some manual steps associated with importing  infrastructure. However, these tools are not part of Terraform itself  and are not endorsed or supported by HashiCorp.



In this lab, you learned how to manage backends and state with  Terraform. You created local and Cloud Storage backends to manage your  state file, refreshed your state, and imported configuration into  Terraform. You then updated the configuration and manually edited to  fully manage the Docker container with Terraform.



![]({{ site.url }}{{ site.baseurl }}/assets/images/gcp-tf-s1.png)

![]({{ site.url }}{{ site.baseurl }}/assets/images/gcp-tf-s2.png)

![]({{ site.url }}{{ site.baseurl }}/assets/images/gcp-tf-s3.png)

![]({{ site.url }}{{ site.baseurl }}/assets/images/gcp-tf-s4.png)

![]({{ site.url }}{{ site.baseurl }}/assets/images/gcp-tf-s5.png)

![]({{ site.url }}{{ site.baseurl }}/assets/images/gcp-tf-s6.png)



