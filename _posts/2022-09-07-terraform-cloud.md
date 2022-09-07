---
layout: single
title:  "Getting Started with Terraform Cloud"
date:   2022-09-07 010:58:04 +0530
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

According to HashiCorp documentation, Terraform Cloud is an application that helps teams use Terraform together. It manages Terraform runs in a consistent and reliable environment, and includes easy access to shared state and secret data, access controls for approving changes to infrastructure, a private registry for sharing Terraform modules, detailed policy controls for governing the contents of Terraform configurations, and more.

Run the following command in your terminal and follow the prompts to fetch an API token for Terraform to use. 

```sh
(base) pradeep:~$terraform login
Terraform will request an API token for app.terraform.io using your browser.

If login is successful, Terraform will store the token in plain text in
the following file for use by subsequent commands:
    /Users/pradeep/.terraform.d/credentials.tfrc.json

Do you want to proceed?
  Only 'yes' will be accepted to confirm.

  Enter a value: yes


---------------------------------------------------------------------------------

Terraform must now open a web browser to the tokens page for app.terraform.io.

If a browser does not open this automatically, open the following URL to proceed:
    https://app.terraform.io/app/settings/tokens?source=terraform-login


---------------------------------------------------------------------------------

Generate a token using your browser, and copy-paste it into this prompt.

Terraform will store the token in plain text in the following file
for use by subsequent commands:
    /Users/pradeep/.terraform.d/credentials.tfrc.json

Token for app.terraform.io:
  Enter a value: 


Retrieved token for user gaddepradeep


---------------------------------------------------------------------------------

                                          -                                
                                          -----                           -
                                          ---------                      --
                                          ---------  -                -----
                                           ---------  ------        -------
                                             -------  ---------  ----------
                                                ----  ---------- ----------
                                                  --  ---------- ----------
   Welcome to Terraform Cloud!                     -  ---------- -------
                                                      ---  ----- ---
   Documentation: terraform.io/docs/cloud             --------   -
                                                      ----------
                                                      ----------
                                                       ---------
                                                           -----
                                                               -


   New to TFC? Follow these steps to instantly apply an example configuration:

   $ git clone https://github.com/hashicorp/tfc-getting-started.git
   $ cd tfc-getting-started
   $ scripts/setup.sh


(base) pradeep:~$
```

```sh
(base) pradeep:~$git clone https://github.com/hashicorp/tfc-getting-started.git
Cloning into 'tfc-getting-started'...
remote: Enumerating objects: 167, done.
remote: Counting objects: 100% (43/43), done.
remote: Compressing objects: 100% (16/16), done.
remote: Total 167 (delta 31), reused 27 (delta 27), pack-reused 124
Receiving objects: 100% (167/167), 38.55 KiB | 487.00 KiB/s, done.
Resolving deltas: 100% (77/77), done.
(base) pradeep:~$
```

```sh
(base) pradeep:~$ls
LICENSE		README.md	backend.tf	main.tf		provider.tf	scripts
(base) pradeep:~$
```
Let's view the setup script
```sh
(base) pradeep:~$cat scripts/setup.sh
#!/usr/bin/env bash
set -euo pipefail

info() {
  printf "\r\033[00;35m$1\033[0m\n"
}

success() {
  printf "\r\033[00;32m$1\033[0m\n"
}

fail() {
  printf "\r\033[0;31m$1\033[0m\n"
}

divider() {
  printf "\r\033[0;1m========================================================================\033[0m\n"
}

pause_for_confirmation() {
  read -rsp $'Press any key to continue (ctrl-c to quit):\n' -n1 key
}

# Set up an interrupt handler so we can exit gracefully
interrupt_count=0
interrupt_handler() {
  ((interrupt_count += 1))

  echo ""
  if [[ $interrupt_count -eq 1 ]]; then
    fail "Really quit? Hit ctrl-c again to confirm."
  else
    echo "Goodbye!"
    exit
  fi
}
trap interrupt_handler SIGINT SIGTERM

# This setup script does all the magic.

# Check for required tools
declare -a req_tools=("terraform" "sed" "curl" "jq")
for tool in "${req_tools[@]}"; do
  if ! command -v "$tool" > /dev/null; then
    fail "It looks like '${tool}' is not installed; please install it and run this setup script again."
    exit 1
  fi
done

# Get the minimum required version of Terraform
minimumTerraformMajorVersion=0
minimumTerraformMinorVersion=14
minimumTerraformVersion=$(($minimumTerraformMajorVersion * 1000 + $minimumTerraformMinorVersion))

# Get the current version of Terraform
installedTerraformMajorVersion=$(terraform version -json | jq -r '.terraform_version' | cut -d '.' -f 1)
installedTerraformMinorVersion=$(terraform version -json | jq -r '.terraform_version' | cut -d '.' -f 2)
installedTerraformVersion=$(($installedTerraformMajorVersion * 1000 + $installedTerraformMinorVersion))

# Check we meet the minimum required version
if [ $installedTerraformVersion -lt $minimumTerraformVersion ]; then
  echo
  fail "Terraform $minimumTerraformMajorVersion.$minimumTerraformMinorVersion.x or later is required for this setup script!"
  echo "You are currently running:"
  terraform version
  exit 1
fi

# Set up some variables we'll need
HOST="${1:-app.terraform.io}"
BACKEND_TF=$(dirname ${BASH_SOURCE[0]})/../backend.tf
PROVIDER_TF=$(dirname ${BASH_SOURCE[0]})/../provider.tf
TERRAFORM_VERSION=$(terraform version -json | jq -r '.terraform_version')

# Check that we've already authenticated via Terraform in the static credentials
# file.  Note that if you configure your token via a credentials helper or any
# other method besides the static file, this script will not take that in to
# account - but we do this to avoid embedding a Go binary in this simple script
# and you hopefully do not need this Getting Started project if you're using one
# already!
CREDENTIALS_FILE="$HOME/.terraform.d/credentials.tfrc.json"
TOKEN=$(jq -j --arg h "$HOST" '.credentials[$h].token' "$CREDENTIALS_FILE")
if [[ ! -f $CREDENTIALS_FILE || $TOKEN == null ]]; then
  fail "We couldn't find a token in the Terraform credentials file at $CREDENTIALS_FILE."
  fail "Please run 'terraform login', then run this setup script again."
  exit 1
fi

# Check that this is your first time running this script. If not, we'll reset
# all local state and restart from scratch!
if ! git diff-index --quiet --no-ext-diff HEAD --; then
  echo "It looks like you may have run this script before! Re-running it will reset any
  changes you've made to backend.tf and provider.tf."
  echo
  pause_for_confirmation

  git checkout HEAD backend.tf provider.tf
  rm -rf .terraform
  rm -f *.lock.hcl
fi

echo
printf "\r\033[00;35;1m
--------------------------------------------------------------------------
Getting Started with Terraform Cloud
-------------------------------------------------------------------------\033[0m"
echo
echo
echo "Terraform Cloud offers secure, easy-to-use remote state management and allows
you to run Terraform remotely in a controlled environment. Terraform Cloud runs
can be performed on demand or triggered automatically by various events."
echo
echo "This script will set up everything you need to get started. You'll be
applying some example infrastructure - for free - in less than a minute."
echo
info "First, we'll do some setup and configure Terraform to use Terraform Cloud."
echo
pause_for_confirmation

# Create a Terraform Cloud organization
echo
echo "Creating an organization and workspace..."
sleep 1
setup() {
  curl https://$HOST/api/getting-started/setup \
    --request POST \
    --silent \
    --header "Content-Type: application/vnd.api+json" \
    --header "Authorization: Bearer $TOKEN" \
    --header "User-Agent: tfc-getting-started" \
    --data @- << REQUEST_BODY
{
	"workflow": "remote-operations",
  "terraform-version": "$TERRAFORM_VERSION"
}
REQUEST_BODY
}

response=$(setup)
err=$(echo $response | jq -r '.errors')

if [[ $err != null ]]; then
  err_msg=$(echo $err | jq -r '.[0].detail')
  if [[ $err_msg != null ]]; then
    fail "An error occurred: ${err_msg}"
  else 
    fail "An unknown error occurred: ${err}"
  fi
  exit 1
fi

# TODO: If there's an active trial, we should just retrieve that and configure
# it instead (especially if it has no state yet)
info=$(echo $response | jq -r '.info')
if [[ $info != null ]]; then
  info "\n${info}"
  exit 0
fi

organization_name=$(echo $response | jq -r '.data."organization-name"')
workspace_name=$(echo $response | jq -r '.data."workspace-name"')

echo
echo "Writing remote backend configuration to backend.tf..."
sleep 2

# We don't sed -i because MacOS's sed has problems with it.
TEMP=$(mktemp)
cat $BACKEND_TF |
  # add backend config for the hostname if necessary
  if [[ "$HOST" != "app.terraform.io" ]]; then sed "5a\\
\    hostname = \"$HOST\"
    "; else cat; fi |
  # replace the organization and workspace names
  sed "s/{{ORGANIZATION_NAME}}/${organization_name}/" |
  sed "s/{{WORKSPACE_NAME}}/${workspace_name}/" \
    > $TEMP
mv $TEMP $BACKEND_TF

# add extra provider config for the hostname if necessary
if [[ "$HOST" != "app.terraform.io" ]]; then
  TEMP=$(mktemp)
  cat $PROVIDER_TF |
    sed "11a\\
  \  hostname = var.provider_hostname
      " \
      > $TEMP
  echo "
variable \"provider_hostname\" {
  type = string
}" >> $TEMP
  mv $TEMP $PROVIDER_TF
fi

echo
divider
echo
success "Ready to go; the example configuration is set up to use Terraform Cloud!"
echo
echo "An example workspace named '${workspace_name}' was created for you."
echo "You can view this workspace in the Terraform Cloud UI here:"
echo "https://$HOST/app/${organization_name}/workspaces/${workspace_name}"
echo
info "Next, we'll run 'terraform init' to initialize the backend and providers:"
echo
echo "$ terraform init"
echo
pause_for_confirmation

echo
terraform init
echo
echo "..."
sleep 2
echo
divider
echo
info "Now it’s time for 'terraform plan', to see what changes Terraform will perform:"
echo
echo "$ terraform plan"
echo
pause_for_confirmation

echo
terraform plan
echo
echo "..."
sleep 3
echo
divider
echo
success "The plan is complete!"
echo
echo "This plan was initiated from your local machine, but executed within
Terraform Cloud!"
echo
echo "Terraform Cloud runs Terraform on disposable virtual machines in
its own cloud infrastructure. This 'remote execution' helps provide consistency
and visibility for critical provisioning operations. It also enables notifications,
version control integration, and powerful features like Sentinel policy enforcement
and cost estimation (shown in the output above)."
echo
info "To actually make changes, we'll run 'terraform apply'. We'll also auto-approve
the result, since this is an example:"
echo
echo "$ terraform apply -auto-approve"
echo
pause_for_confirmation

echo
terraform apply -auto-approve

echo
echo "..."
sleep 3
echo
divider
echo
success "You did it! You just provisioned infrastructure with Terraform Cloud!"
echo
info "The organization we created here has a 30-day free trial of the Team &
Governance tier features. After the trial ends, you'll be moved to the Free tier."
echo
echo "You now have:"
echo
echo "  * Workspaces for organizing your infrastructure. Terraform Cloud manages"
echo "    infrastructure collections with workspaces instead of directories. You"
echo "    can view your workspace here:"
echo "    https://$HOST/app/$organization_name/workspaces/$workspace_name"
echo "  * Remote state management, with the ability to share outputs across"
echo "    workspaces. We've set up state management for you in your current"
echo "    workspace, and you can reference state from other workspaces using"
echo "    the 'terraform_remote_state' data source."
echo "  * Much more!"
echo
info "To see the mock infrastructure you just provisioned and continue exploring
Terraform Cloud, visit:
https://$HOST/fake-web-services"
echo
exit 0
(base) pradeep:~$

```

Let us run the script

```sh
(base) pradeep:~$scripts/setup.sh 


--------------------------------------------------------------------------
Getting Started with Terraform Cloud
-------------------------------------------------------------------------

Terraform Cloud offers secure, easy-to-use remote state management and allows
you to run Terraform remotely in a controlled environment. Terraform Cloud runs
can be performed on demand or triggered automatically by various events.

This script will set up everything you need to get started. You'll be
applying some example infrastructure - for free - in less than a minute.

First, we'll do some setup and configure Terraform to use Terraform Cloud.

Press any key to continue (ctrl-c to quit):

Creating an organization and workspace...

Writing remote backend configuration to backend.tf...

========================================================================

Ready to go; the example configuration is set up to use Terraform Cloud!

An example workspace named 'getting-started' was created for you.
You can view this workspace in the Terraform Cloud UI here:
https://app.terraform.io/app/example-org-48af70/workspaces/getting-started

Next, we'll run 'terraform init' to initialize the backend and providers:

$ terraform init

Press any key to continue (ctrl-c to quit):


Initializing the backend...

Successfully configured the backend "remote"! Terraform will automatically
use this backend unless the backend configuration changes.

Initializing provider plugins...
- Finding latest version of hashicorp/fakewebservices...
- Installing hashicorp/fakewebservices v0.2.3...
- Installed hashicorp/fakewebservices v0.2.3 (signed by HashiCorp)

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

...

========================================================================

Now it’s time for 'terraform plan', to see what changes Terraform will perform:

$ terraform plan

Press any key to continue (ctrl-c to quit):

Running plan in the remote backend. Output will stream here. Pressing Ctrl-C
will stop streaming the logs, but will not stop the plan running remotely.

Preparing the remote plan...

To view this run in a browser, visit:
https://app.terraform.io/app/example-org-48af70/getting-started/runs/run-Lp9bfan2mmC47Jf3

Waiting for the plan to start...

Terraform v1.2.8
on linux_amd64
Initializing plugins and modules...

Terraform used the selected providers to generate the following execution
plan. Resource actions are indicated with the following symbols:
  + create

Terraform will perform the following actions:

  # fakewebservices_database.prod_db will be created
  + resource "fakewebservices_database" "prod_db" {
      + id   = (known after apply)
      + name = "Production DB"
      + size = 256
    }

  # fakewebservices_load_balancer.primary_lb will be created
  + resource "fakewebservices_load_balancer" "primary_lb" {
      + id      = (known after apply)
      + name    = "Primary Load Balancer"
      + servers = [
          + "Server 1",
          + "Server 2",
        ]
    }

  # fakewebservices_server.servers[0] will be created
  + resource "fakewebservices_server" "servers" {
      + id   = (known after apply)
      + name = "Server 1"
      + type = "t2.micro"
      + vpc  = "Primary VPC"
    }

  # fakewebservices_server.servers[1] will be created
  + resource "fakewebservices_server" "servers" {
      + id   = (known after apply)
      + name = "Server 2"
      + type = "t2.micro"
      + vpc  = "Primary VPC"
    }

  # fakewebservices_vpc.primary_vpc will be created
  + resource "fakewebservices_vpc" "primary_vpc" {
      + cidr_block = "0.0.0.0/1"
      + id         = (known after apply)
      + name       = "Primary VPC"
    }

Plan: 5 to add, 0 to change, 0 to destroy.


------------------------------------------------------------------------

Cost estimation:

Resources: 0 of 5 estimated
           $0.0/mo +$0.0

...

========================================================================

The plan is complete!

This plan was initiated from your local machine, but executed within
Terraform Cloud!

Terraform Cloud runs Terraform on disposable virtual machines in
its own cloud infrastructure. This 'remote execution' helps provide consistency
and visibility for critical provisioning operations. It also enables notifications,
version control integration, and powerful features like Sentinel policy enforcement
and cost estimation (shown in the output above).

To actually make changes, we'll run 'terraform apply'. We'll also auto-approve
the result, since this is an example:

$ terraform apply -auto-approve

Press any key to continue (ctrl-c to quit):

Running apply in the remote backend. Output will stream here. Pressing Ctrl-C
will cancel the remote apply if it's still pending. If the apply started it
will stop streaming the logs, but will not stop the apply running remotely.

Preparing the remote apply...

To view this run in a browser, visit:
https://app.terraform.io/app/example-org-48af70/getting-started/runs/run-2GRy1he3EMXf1HNt

Waiting for the plan to start...

Terraform v1.2.8
on linux_amd64
Initializing plugins and modules...

Terraform used the selected providers to generate the following execution
plan. Resource actions are indicated with the following symbols:
  + create

Terraform will perform the following actions:

  # fakewebservices_database.prod_db will be created
  + resource "fakewebservices_database" "prod_db" {
      + id   = (known after apply)
      + name = "Production DB"
      + size = 256
    }

  # fakewebservices_load_balancer.primary_lb will be created
  + resource "fakewebservices_load_balancer" "primary_lb" {
      + id      = (known after apply)
      + name    = "Primary Load Balancer"
      + servers = [
          + "Server 1",
          + "Server 2",
        ]
    }

  # fakewebservices_server.servers[0] will be created
  + resource "fakewebservices_server" "servers" {
      + id   = (known after apply)
      + name = "Server 1"
      + type = "t2.micro"
      + vpc  = "Primary VPC"
    }

  # fakewebservices_server.servers[1] will be created
  + resource "fakewebservices_server" "servers" {
      + id   = (known after apply)
      + name = "Server 2"
      + type = "t2.micro"
      + vpc  = "Primary VPC"
    }

  # fakewebservices_vpc.primary_vpc will be created
  + resource "fakewebservices_vpc" "primary_vpc" {
      + cidr_block = "0.0.0.0/1"
      + id         = (known after apply)
      + name       = "Primary VPC"
    }

Plan: 5 to add, 0 to change, 0 to destroy.


------------------------------------------------------------------------

Cost estimation:

Resources: 0 of 5 estimated
           $0.0/mo +$0.0

------------------------------------------------------------------------

fakewebservices_database.prod_db: Creating...
fakewebservices_database.prod_db: Creation complete after 0s [id=fakedb-t6DMaR31hJbzGE9D]
fakewebservices_vpc.primary_vpc: Creation complete after 0s [id=fakevpc-zhujv7iQD9TzH5ix]
fakewebservices_server.servers[0]: Creating...
fakewebservices_server.servers[1]: Creating...
fakewebservices_server.servers[1]: Creation complete after 0s [id=fakeserver-jFWW2D1CDucvZsMX]
fakewebservices_server.servers[0]: Creation complete after 1s [id=fakeserver-42BKWQaUrjF8kT8S]
fakewebservices_load_balancer.primary_lb: Creating...
fakewebservices_load_balancer.primary_lb: Creation complete after 1s [id=fakelb-7Zt8T5neYdGqRKp9]

Apply complete! Resources: 5 added, 0 changed, 0 destroyed.


...

========================================================================

You did it! You just provisioned infrastructure with Terraform Cloud!

The organization we created here has a 30-day free trial of the Team &
Governance tier features. After the trial ends, you'll be moved to the Free tier.

You now have:

  * Workspaces for organizing your infrastructure. Terraform Cloud manages
    infrastructure collections with workspaces instead of directories. You
    can view your workspace here:
    https://app.terraform.io/app/example-org-48af70/workspaces/getting-started
  * Remote state management, with the ability to share outputs across
    workspaces. We've set up state management for you in your current
    workspace, and you can reference state from other workspaces using
    the 'terraform_remote_state' data source.
  * Much more!

To see the mock infrastructure you just provisioned and continue exploring
Terraform Cloud, visit:
https://app.terraform.io/fake-web-services

(base) pradeep:~$

```

![TFC]({{ site.url }}{{ site.baseurl }}/assets/images/tfc-1.png)

![TFC]({{ site.url }}{{ site.baseurl }}/assets/images/tfc-2.png)


```sh
base) pradeep:~$cat main.tf 
# The following configuration uses a provider which provisions [fake] resources
# to a fictitious cloud vendor called "Fake Web Services". Configuration for
# the fakewebservices provider can be found in provider.tf.
#
# After running the setup script (./scripts/setup.sh), feel free to change these
# resources and 'terraform apply' as much as you'd like! These resources are
# purely for demonstration and created in Terraform Cloud, scoped to your TFC
# user account.
#
# To review the provider and documentation for the available resources and
# schemas, see: https://registry.terraform.io/providers/hashicorp/fakewebservices
#
# If you're looking for the configuration for the remote backend, you can find that
# in backend.tf.


resource "fakewebservices_vpc" "primary_vpc" {
  name       = "Primary VPC"
  cidr_block = "0.0.0.0/1"
}

resource "fakewebservices_server" "servers" {
  count = 2

  name = "Server ${count.index + 1}"
  type = "t2.micro"
  vpc  = fakewebservices_vpc.primary_vpc.name
}

resource "fakewebservices_load_balancer" "primary_lb" {
  name    = "Primary Load Balancer"
  servers = fakewebservices_server.servers[*].name
}

resource "fakewebservices_database" "prod_db" {
  name = "Production DB"
  size = 256
}
(base) pradeep:~$
```

```sh
(base) pradeep:~$cat provider.tf 
# The following variable is used to configure the provider's authentication
# token. You don't need to provide a token on the command line to apply changes,
# though: using the remote backend, Terraform will execute remotely in Terraform
# Cloud where your token is already securely stored in your workspace!

variable "provider_token" {
  type = string
  sensitive = true
}

provider "fakewebservices" {
  token = var.provider_token
}
(base) pradeep:~$
```

```sh
(base) pradeep:~$cat backend.tf 
# The block below configures Terraform to use the 'remote' backend with Terraform Cloud.
# For more information, see https://www.terraform.io/docs/backends/types/remote.html
terraform {
  backend "remote" {
    organization = "example-org-48af70"

    workspaces {
      name = "getting-started"
    }
  }

  required_version = ">= 0.14.0"
}
(base) pradeep:~$
```

As shown on the web page, let us play around with your example configuration

We can change our example configuration to provision more mock resources. 
Here is the modified configuration file.

```sh
base) pradeep:~$cat main.tf    
# The following configuration uses a provider which provisions [fake] resources
# to a fictitious cloud vendor called "Fake Web Services". Configuration for
# the fakewebservices provider can be found in provider.tf.
#
# After running the setup script (./scripts/setup.sh), feel free to change these
# resources and 'terraform apply' as much as you'd like! These resources are
# purely for demonstration and created in Terraform Cloud, scoped to your TFC
# user account.
#
# To review the provider and documentation for the available resources and
# schemas, see: https://registry.terraform.io/providers/hashicorp/fakewebservices
#
# If you're looking for the configuration for the remote backend, you can find that
# in backend.tf.


resource "fakewebservices_vpc" "primary_vpc" {
  name       = "Primary VPC"
  cidr_block = "0.0.0.0/1"
}

resource "fakewebservices_server" "servers" {
  count = 2

  name = "Server ${count.index + 1}"
  type = "t2.micro"
  vpc  = fakewebservices_vpc.primary_vpc.name
}

resource "fakewebservices_load_balancer" "primary_lb" {
  name    = "Primary Load Balancer"
  servers = fakewebservices_server.servers[*].name
}

resource "fakewebservices_database" "prod_db" {
  name = "Production DB"
  size = 256
}

resource "fakewebservices_server" "server-3" {
  name = "Server 3"
  type = "t2.macro"
}

resource "fakewebservices_server" "server-4" {
  name = "Server 4"
  type = "t2.macro"
}
(base) pradeep:~$
```

```sh
(base) pradeep:~$terraform apply
Running apply in the remote backend. Output will stream here. Pressing Ctrl-C
will cancel the remote apply if it's still pending. If the apply started it
will stop streaming the logs, but will not stop the apply running remotely.

Preparing the remote apply...

To view this run in a browser, visit:
https://app.terraform.io/app/example-org-48af70/getting-started/runs/run-t84w8Z1vF3zzmuft

Waiting for the plan to start...

Terraform v1.2.8
on linux_amd64
Initializing plugins and modules...
fakewebservices_vpc.primary_vpc: Refreshing state... [id=fakevpc-zhujv7iQD9TzH5ix]
fakewebservices_database.prod_db: Refreshing state... [id=fakedb-t6DMaR31hJbzGE9D]
fakewebservices_server.servers[0]: Refreshing state... [id=fakeserver-42BKWQaUrjF8kT8S]
fakewebservices_server.servers[1]: Refreshing state... [id=fakeserver-jFWW2D1CDucvZsMX]
fakewebservices_load_balancer.primary_lb: Refreshing state... [id=fakelb-7Zt8T5neYdGqRKp9]

Terraform used the selected providers to generate the following execution
plan. Resource actions are indicated with the following symbols:
  + create

Terraform will perform the following actions:

  # fakewebservices_server.server-3 will be created
  + resource "fakewebservices_server" "server-3" {
      + id   = (known after apply)
      + name = "Server 3"
      + type = "t2.macro"
    }

  # fakewebservices_server.server-4 will be created
  + resource "fakewebservices_server" "server-4" {
      + id   = (known after apply)
      + name = "Server 4"
      + type = "t2.macro"
    }

Plan: 2 to add, 0 to change, 0 to destroy.


------------------------------------------------------------------------

Cost estimation:

Resources: 0 of 7 estimated
           $0.0/mo +$0.0

------------------------------------------------------------------------

Do you want to perform these actions in workspace "getting-started"?
  Terraform will perform the actions described above.
  Only 'yes' will be accepted to approve.

  Enter a value: yes

fakewebservices_server.server-3: Creating...
fakewebservices_server.server-4: Creation complete after 0s [id=fakeserver-566MKyKjSo8ji326]
fakewebservices_server.server-3: Creation complete after 0s [id=fakeserver-pCaHFWMS52bfa1Cq]

Apply complete! Resources: 2 added, 0 changed, 0 destroyed.

(base) pradeep:~$
```

![TFC]({{ site.url }}{{ site.baseurl }}/assets/images/tfc-3.png)

![TFC]({{ site.url }}{{ site.baseurl }}/assets/images/tfc-4.png)

```sh
(base) pradeep:~$terraform destroy
Running apply in the remote backend. Output will stream here. Pressing Ctrl-C
will cancel the remote apply if it's still pending. If the apply started it
will stop streaming the logs, but will not stop the apply running remotely.

Preparing the remote apply...

To view this run in a browser, visit:
https://app.terraform.io/app/example-org-48af70/getting-started/runs/run-Bn37og2Pcw2y8FQd

Waiting for the plan to start...

Terraform v1.2.8
on linux_amd64
Initializing plugins and modules...
fakewebservices_vpc.primary_vpc: Refreshing state... [id=fakevpc-zhujv7iQD9TzH5ix]
fakewebservices_server.server-3: Refreshing state... [id=fakeserver-pCaHFWMS52bfa1Cq]
fakewebservices_database.prod_db: Refreshing state... [id=fakedb-t6DMaR31hJbzGE9D]
fakewebservices_server.server-4: Refreshing state... [id=fakeserver-566MKyKjSo8ji326]
fakewebservices_server.servers[0]: Refreshing state... [id=fakeserver-42BKWQaUrjF8kT8S]
fakewebservices_server.servers[1]: Refreshing state... [id=fakeserver-jFWW2D1CDucvZsMX]
fakewebservices_load_balancer.primary_lb: Refreshing state... [id=fakelb-7Zt8T5neYdGqRKp9]

Terraform used the selected providers to generate the following execution
plan. Resource actions are indicated with the following symbols:
  - destroy

Terraform will perform the following actions:

  # fakewebservices_database.prod_db will be destroyed
  - resource "fakewebservices_database" "prod_db" {
      - id   = "fakedb-t6DMaR31hJbzGE9D" -> null
      - name = "Production DB" -> null
      - size = 256 -> null
    }

  # fakewebservices_load_balancer.primary_lb will be destroyed
  - resource "fakewebservices_load_balancer" "primary_lb" {
      - id      = "fakelb-7Zt8T5neYdGqRKp9" -> null
      - name    = "Primary Load Balancer" -> null
      - servers = [
          - "Server 1",
          - "Server 2",
        ] -> null
    }

  # fakewebservices_server.server-3 will be destroyed
  - resource "fakewebservices_server" "server-3" {
      - id   = "fakeserver-pCaHFWMS52bfa1Cq" -> null
      - name = "Server 3" -> null
      - type = "t2.macro" -> null
    }

  # fakewebservices_server.server-4 will be destroyed
  - resource "fakewebservices_server" "server-4" {
      - id   = "fakeserver-566MKyKjSo8ji326" -> null
      - name = "Server 4" -> null
      - type = "t2.macro" -> null
    }

  # fakewebservices_server.servers[0] will be destroyed
  - resource "fakewebservices_server" "servers" {
      - id   = "fakeserver-42BKWQaUrjF8kT8S" -> null
      - name = "Server 1" -> null
      - type = "t2.micro" -> null
      - vpc  = "Primary VPC" -> null
    }

  # fakewebservices_server.servers[1] will be destroyed
  - resource "fakewebservices_server" "servers" {
      - id   = "fakeserver-jFWW2D1CDucvZsMX" -> null
      - name = "Server 2" -> null
      - type = "t2.micro" -> null
      - vpc  = "Primary VPC" -> null
    }

  # fakewebservices_vpc.primary_vpc will be destroyed
  - resource "fakewebservices_vpc" "primary_vpc" {
      - cidr_block = "0.0.0.0/1" -> null
      - id         = "fakevpc-zhujv7iQD9TzH5ix" -> null
      - name       = "Primary VPC" -> null
    }

Plan: 0 to add, 0 to change, 7 to destroy.


------------------------------------------------------------------------

Cost estimation:

Resources: 0 of 0 estimated
           $0.0/mo +$0.0

------------------------------------------------------------------------

Do you really want to destroy all resources in workspace "getting-started"?
  Terraform will destroy all your managed infrastructure, as shown above.
  There is no undo. Only 'yes' will be accepted to confirm.

  Enter a value: yes

fakewebservices_load_balancer.primary_lb: Destroying... [id=fakelb-7Zt8T5neYdGqRKp9]
fakewebservices_server.server-4: Destroying... [id=fakeserver-566MKyKjSo8ji326]
fakewebservices_server.server-3: Destroying... [id=fakeserver-pCaHFWMS52bfa1Cq]
fakewebservices_server.server-4: Destruction complete after 0s
fakewebservices_load_balancer.primary_lb: Destruction complete after 0s
fakewebservices_server.servers[0]: Destroying... [id=fakeserver-42BKWQaUrjF8kT8S]
fakewebservices_server.servers[1]: Destroying... [id=fakeserver-jFWW2D1CDucvZsMX]
fakewebservices_server.server-3: Destruction complete after 0s
fakewebservices_database.prod_db: Destruction complete after 0s
fakewebservices_server.servers[1]: Destruction complete after 0s
fakewebservices_server.servers[0]: Destruction complete after 0s
fakewebservices_vpc.primary_vpc: Destroying... [id=fakevpc-zhujv7iQD9TzH5ix]
fakewebservices_vpc.primary_vpc: Destruction complete after 0s

Apply complete! Resources: 0 added, 0 changed, 7 destroyed.

(base) pradeep:~$
```

![TFC]({{ site.url }}{{ site.baseurl }}/assets/images/tfc-5.png)

Finally, delete the example organization from the Terraform Cloud.

