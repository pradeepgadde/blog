---

layout: single
title:  "Service Accounts and Roles: Fundamentals"
categories: Cloud
tags: GCP
classes: wide
show_date: true
header:
  overlay_image: /assets/images/gcp-banner.png
  og_image: /assets/images/gcp-banner.png
  teaser: /assets/images/ace-gcp.png
author:
  name     : "Associate Cloud Engineer"
  avatar   : "/assets/images/ace-gcp.png"

sidebar:
  - title: "Topics"
    nav: my-sidebar

---
# Service Accounts and Roles: Fundamentals

Service accounts are a special type of Google account that grant  permissions to virtual machines instead of end users. Service accounts  are primarily used to ensure safe, managed connections to APIs and  Google Cloud services. Granting access to trusted connections and  rejecting malicious ones is a must-have security feature for any Google  Cloud project. 

- Create and manage service accounts.
- Create a virtual machine and associate it with a service account.
- Use client libraries to access BigQuery from a service account.
- Run a query on a BigQuery public dataset from a Compute Engine instance.

## What are service accounts?

A service account is a special Google account that belongs to your application or a [virtual machine](https://cloud.google.com/compute/docs/instances/) (VM) instead of an individual end user. Your application uses the service account to [call the Google API of a service](https://developers.google.com/identity/protocols/OAuth2ServiceAccount#authorizingrequests), so that the users aren't directly involved.

For example, a Compute Engine VM may run as a service account, and  that account can be given permissions to access the resources it needs.  This way the service account is the identity of the service, and the  service account's permissions control which resources the service can  access.

A service account is identified by its email address, which is unique to the account.

#### User-managed service accounts

When you create a new Cloud project using Google Cloud console and if Compute Engine API is enabled for your project, a Compute Engine  Service account is created for you by default. It is identifiable using  the email `PROJECT_NUMBER-compute@developer.gserviceaccount.com`.

f your project contains an App Engine application, the default App  Engine service account is created in your project by default. It is  identifiable using the email `PROJECT_ID@appspot.gserviceaccount.com`.

#### Google-managed service accounts

In addition to the user-managed service accounts, you might see some  additional service accounts in your project’s IAM policy or in the  console. These service accounts are created and owned by Google. These  accounts represent different Google services and each account is  automatically granted IAM roles to access your Google Cloud project.

#### Google APIs service account

An example of a Google-managed service account is a Google API service account identifiable using the email `PROJECT_NUMBER@cloudservices.gserviceaccount.com`. This service account is designed specifically to run internal Google processes on your behalf and is not listed in the **Service Accounts** section of the console. By default, the account is automatically  granted the project editor role on the project and is listed in the **IAM** section of the console. This service account is deleted only when the project is deleted.

## Understanding IAM roles

When an identity calls a Google Cloud API, Google Cloud Identity and  Access Management requires that the identity has the appropriate  permissions to use the resource. You can grant permissions by granting  roles to a user, a group, or a service account.

### Types of roles

There are three types of roles in Cloud IAM:

- **Primitive roles**, which include the Owner, Editor, and Viewer roles that existed prior to the introduction of Cloud IAM.
- **Predefined roles**, which provide granular access for a specific service and are managed by Google Cloud.
- **Custom roles**, which provide granular access according to a user-specified list of permissions.

## Create and manage service accounts

When you create a new Cloud project, Google Cloud automatically  creates one Compute Engine service account and one App Engine service  account under that project. You can create up to 98 additional service  accounts to your project to control access to your resources.

### Creating a service account

Creating a service account is similar to adding a member to your  project, but the service account belongs to your applications rather  than an individual end user.

- To create a service account, run the following command in Cloud Shell:

```sh
Welcome to Cloud Shell! Type "help" to get started.
Your Cloud Platform project in this session is set to qwiklabs-gcp-03-de64d7fc6015.
Use “gcloud config set project [PROJECT_ID]” to change to a different project.
student_01_41d0c1af2373@cloudshell:~ (qwiklabs-gcp-03-de64d7fc6015)$ gcloud config set compute/region us-east4
WARNING: Property validation for compute/region was skipped.
Updated property [compute/region].
student_01_41d0c1af2373@cloudshell:~ (qwiklabs-gcp-03-de64d7fc6015)$ gcloud iam service-accounts create my-sa-123 --display-name "my service account"
Created service account [my-sa-123].
student_01_41d0c1af2373@cloudshell:~ (qwiklabs-gcp-03-de64d7fc6015)$

```



### Granting roles to service accounts

When granting IAM roles, you can treat a service account either as a [resource](https://cloud.google.com/iam/docs/overview#resource) or as an [identity](https://cloud.google.com/iam/docs/overview#concepts_related_to_identity).

Your application uses a service account as an identity to  authenticate to Google Cloud services. For example, if you have a  Compute Engine Virtual Machine (VM) running as a service account, you  can grant the editor role to the service account (the identity) for a  project (the resource).

At the same time, you might also want to control who can start the VM. You can do this by granting a user (the identity) the [serviceAccountUser](https://cloud.google.com/iam/docs/service-accounts#the_service_account_user_role) role for the service account (the resource).



#### Granting roles to a service account for specific resources

You grant roles to a service account so that the service account has  permission to complete specific actions on the resources in your Cloud  Platform project. For example, you might grant the `storage.admin` role to a service account so that it has control over objects and buckets in Cloud Storage.

- Run the following in Cloud Shell to grant roles to the service account you just made:

```sh
student_01_41d0c1af2373@cloudshell:~ (qwiklabs-gcp-03-de64d7fc6015)$ gcloud projects add-iam-policy-binding $DEVSHELL_PROJECT_ID \
    --member serviceAccount:my-sa-123@$DEVSHELL_PROJECT_ID.iam.gserviceaccount.com --role roles/editor
Updated IAM policy for project [qwiklabs-gcp-03-de64d7fc6015].
bindings:
- members:
  - serviceAccount:qwiklabs-gcp-03-de64d7fc6015@qwiklabs-gcp-03-de64d7fc6015.iam.gserviceaccount.com
  role: roles/bigquery.admin
- members:
  - serviceAccount:840906118713@cloudbuild.gserviceaccount.com
  role: roles/cloudbuild.builds.builder
- members:
  - serviceAccount:service-840906118713@gcp-sa-cloudbuild.iam.gserviceaccount.com
  role: roles/cloudbuild.serviceAgent
- members:
  - serviceAccount:service-840906118713@compute-system.iam.gserviceaccount.com
  role: roles/compute.serviceAgent
- members:
  - serviceAccount:service-840906118713@container-engine-robot.iam.gserviceaccount.com
  role: roles/container.serviceAgent
- members:
  - serviceAccount:840906118713-compute@developer.gserviceaccount.com
  - serviceAccount:840906118713@cloudservices.gserviceaccount.com
  - serviceAccount:my-sa-123@qwiklabs-gcp-03-de64d7fc6015.iam.gserviceaccount.com
  - user:student-01-41d0c1af2373@qwiklabs.net
  role: roles/editor
- members:
  - serviceAccount:admiral@qwiklabs-services-prod.iam.gserviceaccount.com
  - serviceAccount:qwiklabs-gcp-03-de64d7fc6015@qwiklabs-gcp-03-de64d7fc6015.iam.gserviceaccount.com
  - user:student-01-41d0c1af2373@qwiklabs.net
  role: roles/owner
- members:
  - serviceAccount:qwiklabs-gcp-03-de64d7fc6015@qwiklabs-gcp-03-de64d7fc6015.iam.gserviceaccount.com
  role: roles/storage.admin
- members:
  - user:student-01-41d0c1af2373@qwiklabs.net
  role: roles/viewer
etag: BwYZdkTyBk0=
version: 1
student_01_41d0c1af2373@cloudshell:~ (qwiklabs-gcp-03-de64d7fc6015)$ 
```



## Use the client libraries to access BigQuery using a service account

In this section, you query the BigQuery public datasets from an  instance with the help of a service account that has the necessary  roles.

### Create a service account

First create a new service account from the console.

1. Go to **Navigation menu** > **IAM & Admin**, select **Service accounts** and click on **+ Create Service Account**.
2. Fill necessary details with:

- **Service account name:**  bigquery-qwiklab

1. Now click **Create and Continue** and then add the following roles:

- **Role:** Bigquery > BigQuery Data Viewer and BigQuery > BigQuery User
- Click **Continue** and then click **Done**.

### Create a VM instance

1. In the console, go to **Compute Engine > VM Instances**, and click **Create Instance**.
2. Create your VM with the following information:

```sh
student_01_41d0c1af2373@cloudshell:~ (qwiklabs-gcp-03-de64d7fc6015)$ gcloud compute instances create bigquery-instance --project=qwiklabs-gcp-03-de64d7fc6015 --zone=us-east4-b --machine-type=e2-medium --network-interface=network-tier=PREMIUM,stack-type=IPV4_ONLY,subnet=default --metadata=enable-oslogin=true --maintenance-policy=MIGRATE --provisioning-model=STANDARD --service-account=bigquery-qwiklab@qwiklabs-gcp-03-de64d7fc6015.iam.gserviceaccount.com --scopes=https://www.googleapis.com/auth/cloud-platform --create-disk=auto-delete=yes,boot=yes,device-name=bigquery-instance,image=projects/debian-cloud/global/images/debian-11-bullseye-v20240515,mode=rw,size=10,type=projects/qwiklabs-gcp-03-de64d7fc6015/zones/us-east4-b/diskTypes/pd-balanced --no-shielded-secure-boot --shielded-vtpm --shielded-integrity-monitoring --labels=goog-ec-src=vm_add-gcloud --reservation-affinity=any
Created [https://www.googleapis.com/compute/v1/projects/qwiklabs-gcp-03-de64d7fc6015/zones/us-east4-b/instances/bigquery-instance].
NAME: bigquery-instance
ZONE: us-east4-b
MACHINE_TYPE: e2-medium
PREEMPTIBLE: 
INTERNAL_IP: 10.150.0.2
EXTERNAL_IP: 35.199.11.63
STATUS: RUNNING
student_01_41d0c1af2373@cloudshell:~ (qwiklabs-gcp-03-de64d7fc6015)$ 
```

### Put the example code on a Compute Engine instance

1. In the console, go to **Compute Engine** > **VM Instances**.
2. SSH into `bigquery-instance` by clicking on the **SSH** button.

```sh
student-01-41d0c1af2373@bigquery-instance:~$ history 
    1  sudo apt-get update
    2  sudo apt-get install -y git python3-pip
    3  pip3 install --upgrade pip
    4  pip3 install google-cloud-bigquery
    5  pip3 install pyarrow
    6  pip3 install pandas
    7  pip3 install db-dtypes
    8  history 
student-01-41d0c1af2373@bigquery-instance:~$ 
```

```py
student-01-41d0c1af2373@bigquery-instance:~$ echo "
from google.auth import compute_engine
from google.cloud import bigquery

credentials = compute_engine.Credentials(
    service_account_email='YOUR_SERVICE_ACCOUNT')

query = '''
SELECT
  year,
  COUNT(1) as num_babies
FROM
  publicdata.samples.natality
WHERE
" > query.py.query(query).to_dataframe())',
student-01-41d0c1af2373@bigquery-instance:~$ sed -i -e "s/qwiklabs-gcp-03-de64d7fc6015/$(gcloud config get-value project)/g" query.py
student-01-41d0c1af2373@bigquery-instance:~$ cat query.py

from google.auth import compute_engine
from google.cloud import bigquery

credentials = compute_engine.Credentials(
    service_account_email='YOUR_SERVICE_ACCOUNT')

query = '''
SELECT
  year,
  COUNT(1) as num_babies
FROM
  publicdata.samples.natality
WHERE
  year > 2000
GROUP BY
  year
'''

client = bigquery.Client(
    project='qwiklabs-gcp-03-de64d7fc6015',
    credentials=credentials)
print(client.query(query).to_dataframe())

student-01-41d0c1af2373@bigquery-instance:~$ 
```

```sh
student-01-41d0c1af2373@bigquery-instance:~$ sed -i -e "s/YOUR_SERVICE_ACCOUNT/bigquery-qwiklab@$(gcloud config get-value project).iam.gserviceaccount.com/g" query.py
student-01-41d0c1af2373@bigquery-instance:~$ cat query.py

from google.auth import compute_engine
from google.cloud import bigquery

credentials = compute_engine.Credentials(
    service_account_email='bigquery-qwiklab@qwiklabs-gcp-03-de64d7fc6015.iam.gserviceaccount.com')

query = '''
SELECT
  year,
  COUNT(1) as num_babies
FROM
  publicdata.samples.natality
WHERE
  year > 2000
GROUP BY
  year
'''

client = bigquery.Client(
    project='qwiklabs-gcp-03-de64d7fc6015',
    credentials=credentials)
print(client.query(query).to_dataframe())

student-01-41d0c1af2373@bigquery-instance:~$ 
```

```py
student-01-41d0c1af2373@bigquery-instance:~$ python3 query.py
   year  num_babies
0  2007     4324008
1  2005     4145619
2  2008     4255156
3  2001     4031531
4  2002     4027376
5  2006     4273225
6  2004     4118907
7  2003     4096092
student-01-41d0c1af2373@bigquery-instance:~$ 
```

## History

```sh
student_01_41d0c1af2373@cloudshell:~ (qwiklabs-gcp-03-de64d7fc6015)$ history 
    1  gcloud config set compute/region us-east4
    2  gcloud iam service-accounts create my-sa-123 --display-name "my service account"
    3  gcloud projects add-iam-policy-binding $DEVSHELL_PROJECT_ID     --member serviceAccount:my-sa-123@$DEVSHELL_PROJECT_ID.iam.gserviceaccount.com --role roles/editor
    4  gcloud compute instances create bigquery-instance --project=qwiklabs-gcp-03-de64d7fc6015 --zone=us-east4-b --machine-type=e2-medium --network-interface=network-tier=PREMIUM,stack-type=IPV4_ONLY,subnet=default --metadata=enable-oslogin=true --maintenance-policy=MIGRATE --provisioning-model=STANDARD --service-account=bigquery-qwiklab@qwiklabs-gcp-03-de64d7fc6015.iam.gserviceaccount.com --scopes=https://www.googleapis.com/auth/cloud-platform --create-disk=auto-delete=yes,boot=yes,device-name=bigquery-instance,image=projects/debian-cloud/global/images/debian-11-bullseye-v20240515,mode=rw,size=10,type=projects/qwiklabs-gcp-03-de64d7fc6015/zones/us-east4-b/diskTypes/pd-balanced --no-shielded-secure-boot --shielded-vtpm --shielded-integrity-monitoring --labels=goog-ec-src=vm_add-gcloud --reservation-affinity=any
    5  gcloud compute ssh --zone "us-east4-b" "bigquery-instance" --project "qwiklabs-gcp-03-de64d7fc6015"
    6  history 
student_01_41d0c1af2373@cloudshell:~ (qwiklabs-gcp-03-de64d7fc6015)$ 
```

