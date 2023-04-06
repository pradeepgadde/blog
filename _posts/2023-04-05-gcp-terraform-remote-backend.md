---


layout: single
title:  "GCP Terraform: Creating a Remote Backend"
date:   2023-04-05 11:59:04 +0530
categories: Cloud
tags: GCP
classes: wide
show_date: true
header:
  overlay_image: /assets/images/gcp-banner-2.png
  og_image: /assets/images/gcp-banner-2.png
  teaser: /assets/images/gpcne.png
author:
  name     : "Cloud Network Engineer"
  avatar   : "/assets/images/gpcne.png"

sidebar:
  - title: "Topics"
    nav: my-sidebar

---
# Creating a Remote Backend




In this lab, you will create a local backend and then create a Cloud Storage bucket to migrate the state to a remote backend

- Create a local backend.
- Create a Cloud Storage backend.
- Refresh your Terraform state.



```sh
Welcome to Cloud Shell! Type "help" to get started.
Your Cloud Platform project in this session is set to qwiklabs-gcp-00-0244f2a4320d.
Use “gcloud config set project [PROJECT_ID]” to change to a different project.
student_03_1336992abf14@cloudshell:~ (qwiklabs-gcp-00-0244f2a4320d)$ terraform --version
Terraform v1.4.4
on linux_amd64
student_03_1336992abf14@cloudshell:~ (qwiklabs-gcp-00-0244f2a4320d)$ gcloud config list --format 'value(core.project)'
qwiklabs-gcp-00-0244f2a4320d
student_03_1336992abf14@cloudshell:~ (qwiklabs-gcp-00-0244f2a4320d)$ vi main.tf
student_03_1336992abf14@cloudshell:~ (qwiklabs-gcp-00-0244f2a4320d)$ cat main.tf
provider "google" {
  project     = "qwiklabs-gcp-00-0244f2a4320d"
  region      = "us-central-1"
}
resource "google_storage_bucket" "test-bucket-for-state" {
  name        = "qwiklabs-gcp-00-0244f2a4320d"
  location    = "US"
  uniform_bucket_level_access = true
}
terraform {
  backend "local" {
    path = "terraform/state/terraform.tfstate"
  }
}
student_03_1336992abf14@cloudshell:~ (qwiklabs-gcp-00-0244f2a4320d)$
```



```sh
student_03_1336992abf14@cloudshell:~ (qwiklabs-gcp-00-0244f2a4320d)$ terraform init

Initializing the backend...

Successfully configured the backend "local"! Terraform will automatically
use this backend unless the backend configuration changes.

Initializing provider plugins...
- Finding latest version of hashicorp/google...
- Installing hashicorp/google v4.60.1...
- Installed hashicorp/google v4.60.1 (signed by HashiCorp)

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
student_03_1336992abf14@cloudshell:~ (qwiklabs-gcp-00-0244f2a4320d)$
```

```sh
student_03_1336992abf14@cloudshell:~ (qwiklabs-gcp-00-0244f2a4320d)$ terraform apply

Terraform used the selected providers to generate the following execution plan. Resource actions are indicated with the following symbols:
  + create

Terraform will perform the following actions:

  # google_storage_bucket.test-bucket-for-state will be created
  + resource "google_storage_bucket" "test-bucket-for-state" {
      + force_destroy               = false
      + id                          = (known after apply)
      + location                    = "US"
      + name                        = "qwiklabs-gcp-00-0244f2a4320d"
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
google_storage_bucket.test-bucket-for-state: Creation complete after 2s [id=qwiklabs-gcp-00-0244f2a4320d]

Apply complete! Resources: 1 added, 0 changed, 0 destroyed.
student_03_1336992abf14@cloudshell:~ (qwiklabs-gcp-00-0244f2a4320d)$
```



```sh
student_03_1336992abf14@cloudshell:~ (qwiklabs-gcp-00-0244f2a4320d)$ terraform show
# google_storage_bucket.test-bucket-for-state:
resource "google_storage_bucket" "test-bucket-for-state" {
    default_event_based_hold    = false
    force_destroy               = false
    id                          = "qwiklabs-gcp-00-0244f2a4320d"
    location                    = "US"
    name                        = "qwiklabs-gcp-00-0244f2a4320d"
    project                     = "qwiklabs-gcp-00-0244f2a4320d"
    public_access_prevention    = "inherited"
    requester_pays              = false
    self_link                   = "https://www.googleapis.com/storage/v1/b/qwiklabs-gcp-00-0244f2a4320d"
    storage_class               = "STANDARD"
    uniform_bucket_level_access = true
    url                         = "gs://qwiklabs-gcp-00-0244f2a4320d"
}
student_03_1336992abf14@cloudshell:~ (qwiklabs-gcp-00-0244f2a4320d)$
```

## Add a Cloud Storage backend

1. Navigate back to your `main.tf` file in the editor. You will now replace the current local backend with a `gcs` backend.
2. To change the existing local backend configuration, replace the code for local backend with the following configuration in the `main.tf` file.

```sh
student_03_1336992abf14@cloudshell:~ (qwiklabs-gcp-00-0244f2a4320d)$ cat main.tf
provider "google" {
  project     = "qwiklabs-gcp-00-0244f2a4320d"
  region      = "us-central-1"
}
resource "google_storage_bucket" "test-bucket-for-state" {
  name        = "qwiklabs-gcp-00-0244f2a4320d"
  location    = "US"
  uniform_bucket_level_access = true
}
terraform {
  backend "gcs" {
    bucket  = "qwiklabs-gcp-00-0244f2a4320d"
    prefix  = "terraform/state"
  }
}
student_03_1336992abf14@cloudshell:~ (qwiklabs-gcp-00-0244f2a4320d)$
```

![]({{ site.url }}{{ site.baseurl }}/assets/images/gcp-tf-10.png)

```sh
student_03_1336992abf14@cloudshell:~ (qwiklabs-gcp-00-0244f2a4320d)$ terraform init -migrate-state

Initializing the backend...
Terraform detected that the backend type changed from "local" to "gcs".

Acquiring state lock. This may take a few moments...
Do you want to copy existing state to the new backend?
  Pre-existing state was found while migrating the previous "local" backend to the
  newly configured "gcs" backend. No existing state was found in the newly
  configured "gcs" backend. Do you want to copy this state to the new "gcs"
  backend? Enter "yes" to copy and "no" to start with an empty state.

  Enter a value: yes

Releasing state lock. This may take a few moments...

Successfully configured the backend "gcs"! Terraform will automatically
use this backend unless the backend configuration changes.

Initializing provider plugins...
- Reusing previous version of hashicorp/google from the dependency lock file
- Using previously-installed hashicorp/google v4.60.1

Terraform has been successfully initialized!

You may now begin working with Terraform. Try running "terraform plan" to see
any changes that are required for your infrastructure. All Terraform commands
should now work.

If you ever set or change modules or backend configuration for Terraform,
rerun this command to reinitialize your working directory. If you forget, other
commands will detect it and remind you to do so if necessary.
student_03_1336992abf14@cloudshell:~ (qwiklabs-gcp-00-0244f2a4320d)$
```

![]({{ site.url }}{{ site.baseurl }}/assets/images/gcp-tf-11.png)

![]({{ site.url }}{{ site.baseurl }}/assets/images/gcp-tf-12.png)

```json
{
  "version": 4,
  "terraform_version": "1.4.4",
  "serial": 2,
  "lineage": "f727cc57-2210-13a8-1caf-2b84170d8eb7",
  "outputs": {},
  "resources": [
    {
      "mode": "managed",
      "type": "google_storage_bucket",
      "name": "test-bucket-for-state",
      "provider": "provider[\"registry.terraform.io/hashicorp/google\"]",
      "instances": [
        {
          "schema_version": 0,
          "attributes": {
            "autoclass": [],
            "cors": [],
            "custom_placement_config": [],
            "default_event_based_hold": false,
            "encryption": [],
            "force_destroy": false,
            "id": "qwiklabs-gcp-00-0244f2a4320d",
            "labels": null,
            "lifecycle_rule": [],
            "location": "US",
            "logging": [],
            "name": "qwiklabs-gcp-00-0244f2a4320d",
            "project": "qwiklabs-gcp-00-0244f2a4320d",
            "public_access_prevention": "inherited",
            "requester_pays": false,
            "retention_policy": [],
            "self_link": "https://www.googleapis.com/storage/v1/b/qwiklabs-gcp-00-0244f2a4320d",
            "storage_class": "STANDARD",
            "timeouts": null,
            "uniform_bucket_level_access": true,
            "url": "gs://qwiklabs-gcp-00-0244f2a4320d",
            "versioning": [],
            "website": []
          },
          "sensitive_attributes": [],
          "private": "eyJlMmJmYjczMC1lY2FhLTExZTYtOGY4OC0zNDM2M2JjN2M0YzAiOnsiY3JlYXRlIjo2MDAwMDAwMDAwMDAsInJlYWQiOjI0MDAwMDAwMDAwMCwidXBkYXRlIjoyNDAwMDAwMDAwMDB9fQ=="
        }
      ]
    }
  ],
  "check_results": null
}
```

The `terraform refresh` command is used to reconcile the  state Terraform knows about (via its state file) with the real-world  infrastructure. This can be used to detect any drift from the last-known state and to update the state file. This does not modify infrastructure, but does modify the state file. If  the state is changed, this may cause changes to occur during the next  plan or apply.





Return to your storage bucket in the Cloud Console. Select the check box next to the name, and  click the **Labels** button on the top. The info panel with **Permissions** and **Labels** tabs will open up.

Click the **Labels** tab.

Click **+ADD LABEL**. Set the **Key1** = `key` and **Value1** = `value`.

Click **Save**.



```sh
google_storage_bucket.test-bucket-for-state: Refreshing state... [id=qwiklabs-gcp-00-0244f2a4320d]
Releasing state lock. This may take a few moments...
student_03_1336992abf14@cloudshell:~ (qwiklabs-gcp-00-0244f2a4320d)$
```

![]({{ site.url }}{{ site.baseurl }}/assets/images/gcp-tf-13.png)

## Clean up the workspace

1. First, revert your backend to `local` so you can delete the storage bucket. Copy and replace the `gcs` configuration with the following:

```sh
student_03_1336992abf14@cloudshell:~ (qwiklabs-gcp-00-0244f2a4320d)$ cat main.tf
provider "google" {
  project     = "qwiklabs-gcp-00-0244f2a4320d"
  region      = "us-central-1"
}
resource "google_storage_bucket" "test-bucket-for-state" {
  name        = "qwiklabs-gcp-00-0244f2a4320d"
  location    = "US"
  uniform_bucket_level_access = true
}
terraform {
  backend "local" {
    path = "terraform/state/terraform.tfstate"
  }
}
student_03_1336992abf14@cloudshell:~ (qwiklabs-gcp-00-0244f2a4320d)$
```

```sh
student_03_1336992abf14@cloudshell:~ (qwiklabs-gcp-00-0244f2a4320d)$ terraform init -migrate-state

Initializing the backend...
Terraform detected that the backend type changed from "gcs" to "local".

Acquiring state lock. This may take a few moments...
Do you want to copy existing state to the new backend?
  Pre-existing state was found while migrating the previous "gcs" backend to the
  newly configured "local" backend. An existing non-empty state already exists in
  the new backend. The two states have been saved to temporary files that will be
  removed after responding to this query.

  Previous (type "gcs"): /tmp/terraform2775610862/1-gcs.tfstate
  New      (type "local"): /tmp/terraform2775610862/2-local.tfstate

  Do you want to overwrite the state in the new backend with the previous state?
  Enter "yes" to copy and "no" to start with the existing state in the newly
  configured "local" backend.

  Enter a value: yes

Releasing state lock. This may take a few moments...

Successfully configured the backend "local"! Terraform will automatically
use this backend unless the backend configuration changes.

Initializing provider plugins...
- Reusing previous version of hashicorp/google from the dependency lock file
- Using previously-installed hashicorp/google v4.60.1

Terraform has been successfully initialized!

You may now begin working with Terraform. Try running "terraform plan" to see
any changes that are required for your infrastructure. All Terraform commands
should now work.

If you ever set or change modules or backend configuration for Terraform,
rerun this command to reinitialize your working directory. If you forget, other
commands will detect it and remind you to do so if necessary.
student_03_1336992abf14@cloudshell:~ (qwiklabs-gcp-00-0244f2a4320d)$
```

In the `main.tf` file, add the `force_destroy = true` argument to your `google_storage_bucket` resource. When you delete a bucket, this boolean option will [delete all contained objects](https://www.terraform.io/docs/providers/google/r/storage_bucket.html#force_destroy).

```sh
student_03_1336992abf14@cloudshell:~ (qwiklabs-gcp-00-0244f2a4320d)$ cat main.tf
provider "google" {
  project     = "qwiklabs-gcp-00-0244f2a4320d"
  region      = "us-central-1"
}
resource "google_storage_bucket" "test-bucket-for-state" {
  name        = "qwiklabs-gcp-00-0244f2a4320d"
  location    = "US"
  uniform_bucket_level_access = true
  force_destroy = true
}
terraform {
  backend "local" {
    path = "terraform/state/terraform.tfstate"
  }
}
student_03_1336992abf14@cloudshell:~ (qwiklabs-gcp-00-0244f2a4320d)$
```

```sh
student_03_1336992abf14@cloudshell:~ (qwiklabs-gcp-00-0244f2a4320d)$ terraform apply
google_storage_bucket.test-bucket-for-state: Refreshing state... [id=qwiklabs-gcp-00-0244f2a4320d]

Terraform used the selected providers to generate the following execution plan. Resource actions are indicated with the following symbols:
  ~ update in-place

Terraform will perform the following actions:

  # google_storage_bucket.test-bucket-for-state will be updated in-place
  ~ resource "google_storage_bucket" "test-bucket-for-state" {
      ~ force_destroy               = false -> true
        id                          = "qwiklabs-gcp-00-0244f2a4320d"
      ~ labels                      = {
          - "key" = "value" -> null
        }
        name                        = "qwiklabs-gcp-00-0244f2a4320d"
        # (9 unchanged attributes hidden)
    }

Plan: 0 to add, 1 to change, 0 to destroy.

Do you want to perform these actions?
  Terraform will perform the actions described above.
  Only 'yes' will be accepted to approve.

  Enter a value: yes

google_storage_bucket.test-bucket-for-state: Modifying... [id=qwiklabs-gcp-00-0244f2a4320d]
google_storage_bucket.test-bucket-for-state: Modifications complete after 0s [id=qwiklabs-gcp-00-0244f2a4320d]

Apply complete! Resources: 0 added, 1 changed, 0 destroyed.
student_03_1336992abf14@cloudshell:~ (qwiklabs-gcp-00-0244f2a4320d)$
```

```sh
student_03_1336992abf14@cloudshell:~ (qwiklabs-gcp-00-0244f2a4320d)$ terraform destroy
google_storage_bucket.test-bucket-for-state: Refreshing state... [id=qwiklabs-gcp-00-0244f2a4320d]

Terraform used the selected providers to generate the following execution plan. Resource actions are indicated with the following symbols:
  - destroy

Terraform will perform the following actions:

  # google_storage_bucket.test-bucket-for-state will be destroyed
  - resource "google_storage_bucket" "test-bucket-for-state" {
      - default_event_based_hold    = false -> null
      - force_destroy               = true -> null
      - id                          = "qwiklabs-gcp-00-0244f2a4320d" -> null
      - labels                      = {} -> null
      - location                    = "US" -> null
      - name                        = "qwiklabs-gcp-00-0244f2a4320d" -> null
      - project                     = "qwiklabs-gcp-00-0244f2a4320d" -> null
      - public_access_prevention    = "inherited" -> null
      - requester_pays              = false -> null
      - self_link                   = "https://www.googleapis.com/storage/v1/b/qwiklabs-gcp-00-0244f2a4320d" -> null
      - storage_class               = "STANDARD" -> null
      - uniform_bucket_level_access = true -> null
      - url                         = "gs://qwiklabs-gcp-00-0244f2a4320d" -> null
    }

Plan: 0 to add, 0 to change, 1 to destroy.

Do you really want to destroy all resources?
  Terraform will destroy all your managed infrastructure, as shown above.
  There is no undo. Only 'yes' will be accepted to confirm.

  Enter a value: yes

google_storage_bucket.test-bucket-for-state: Destroying... [id=qwiklabs-gcp-00-0244f2a4320d]
google_storage_bucket.test-bucket-for-state: Destruction complete after 2s

Destroy complete! Resources: 1 destroyed.
student_03_1336992abf14@cloudshell:~ (qwiklabs-gcp-00-0244f2a4320d)$
```

![]({{ site.url }}{{ site.baseurl }}/assets/images/gcp-tf-14.png)

In this lab, we learned how to manage backends and state with  Terraform. We created local and Cloud Storage backends to manage our  state file, and also refreshed the state.