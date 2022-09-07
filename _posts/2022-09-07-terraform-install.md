---
layout: single
title:  "Installing the latest Terraform Package"
date:   2022-09-07 04:56:04 +0530
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
# Updating the Terraform Version
First let us verify the current version of the Terraform.

```sh
(base) pradeep:~$terraform version
Terraform v1.1.9
on darwin_amd64

Your version of Terraform is out of date! The latest version
is 1.2.8. You can update by downloading from https://www.terraform.io/downloads.html
(base) pradeep:~$
```

```sh
(base) pradeep:~$brew install terraform
Running `brew update --auto-update`...
==> Auto-updated Homebrew!
Updated 4 taps (hashicorp/tap, azure/functions, homebrew/core and homebrew/cask).
==> New Formulae
{snip}
==> New Casks
{snip}

You have 20 outdated formulae and 2 outdated casks installed.
You can upgrade them with brew upgrade
or list them with brew outdated.

==> Downloading https://ghcr.io/v2/homebrew/core/terraform/manifests/1.2.8
######################################################################## 100.0%
==> Downloading https://ghcr.io/v2/homebrew/core/terraform/blobs/sha256:b706c616ec77c065678f18fb3b4c3c1d9b5498c27f4e11c0db653e9893f1984d
==> Downloading from https://pkg-containers.githubusercontent.com/ghcr1/blobs/sha256:b706c616ec77c065678f18fb3b4c3c1d9b5498c27f4e11c0db653e9893f1984d?se=2022-09-07T15%3A00%3A00Z&sig=wgm57PsgutrtdBDhmDc2U7
######################################################################## 100.0%
==> Pouring terraform--1.2.8.monterey.bottle.tar.gz
Error: The `brew link` step did not complete successfully
The formula built, but is not symlinked into /usr/local
Could not symlink bin/terraform
Target /usr/local/bin/terraform
already exists. You may want to remove it:
  rm '/usr/local/bin/terraform'

To force the link and overwrite all conflicting files:
  brew link --overwrite terraform

To list all files that would be deleted:
  brew link --overwrite --dry-run terraform

Possible conflicting files are:
/usr/local/bin/terraform
==> Summary
🍺  /usr/local/Cellar/terraform/1.2.8: 6 files, 67.4MB
==> Running `brew cleanup terraform`...
Disable this behaviour by setting HOMEBREW_NO_INSTALL_CLEANUP.
Hide these hints with HOMEBREW_NO_ENV_HINTS (see `man brew`).
Removing: /usr/local/Cellar/terraform/1.1.9... (3 files, 69.2MB)
==> `brew cleanup` has not been run in the last 30 days, running now...
Disable this behaviour by setting HOMEBREW_NO_INSTALL_CLEANUP.
Hide these hints with HOMEBREW_NO_ENV_HINTS (see `man brew`).
{snip}
(base) pradeep:~$
```
As we have the previous version of Terraform already, the symlink creation failed.

```sh
(base) pradeep:~$rm  /usr/local/bin/terraform
(base) pradeep:~$
```
After removing the previous package,  tried installing terraform again.

```sh
(base) pradeep:~$brew install terraform      
Warning: terraform 1.2.8 is already installed, it's just not linked.
To link this version, run:
  brew link terraform
(base) pradeep:~$
```

Home Brew says that `terraform 1.2.8 is already installed, it's just not linked.`

Let us link it using the command shown by the Home Brew.
```sh
(base) pradeep:~$brew link terraform
Linking /usr/local/Cellar/terraform/1.2.8... 1 symlinks created.
(base) pradeep:~$
```

Finally verify the current version
```sh
(base) pradeep:~$terraform version
Terraform v1.2.8
on darwin_amd64
(base) pradeep:~$
```
Success! We have updated the Terraform version from v1.1.9 to v1.2.8.

