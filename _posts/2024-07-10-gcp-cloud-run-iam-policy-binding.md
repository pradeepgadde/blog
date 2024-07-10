---

layout: single
title:  "Implementing Least Privilege IAM Policy Bindings in Cloud Run"
categories: Cloud
tags: GCP
classes: wide
show_date: true
header:
  overlay_image: /assets/images/gcp-banner-2.png
  og_image: /assets/images/gcp-banner-2.png
  teaser: /assets/images/pca-gcp.png
author:
  name     : "Professional Cloud Architect"
  avatar   : "/assets/images/pca-gcp.png"

sidebar:
  - title: "Topics"
    nav: my-sidebar

---
# Implementing Least Privilege IAM Policy Bindings in Cloud Run

In Cloud Run, if a service is deployed without specifying a service  account, a default service account is used. The default service account  used is the Compute Engine service account which has the broad Editor  role on the project. Because of policy binding inheritance, this service account has read and write permissions on most resources in your  project. While convenient, it's an inherent security risk as resources  can be created, modified, or deleted with this service account.

To mitigate this risk and implement the principle of least privilege, you should create a service account that serves as the service's  identity, and grant the minimum set of permissions to the account that  are required for the service's functionality.

- Configure your environment and enable the Cloud Run API.
- Create and deploy a public Cloud Run service.
- Test the service with unauthenticated requests.
- Create a service account with minimum permissions.
- Use the gcloud CLI to authenticate with the service account, and invoke a Cloud Run service.
- Implement least privilege by granting the minimum set of permissions required to invoke a service on Cloud Run.

## Configure the environment

Set up environment variables in Cloud Shell to make the provisioning process more flexible.

```sh
Welcome to Cloud Shell! Type "help" to get started.
Your Cloud Platform project in this session is set to qwiklabs-gcp-00-cbf0411caa40.
Use “gcloud config set project [PROJECT_ID]” to change to a different project.
student_01_e488ef31cb8a@cloudshell:~ (qwiklabs-gcp-00-cbf0411caa40)$  gcloud services enable run.googleapis.com
Operation "operations/acf.p2-27047558188-c2de9f89-b190-46fd-99b6-84aed27da351" finished successfully.
student_01_e488ef31cb8a@cloudshell:~ (qwiklabs-gcp-00-cbf0411caa40)$  LOCATION=us-east1
student_01_e488ef31cb8a@cloudshell:~ (qwiklabs-gcp-00-cbf0411caa40)$ gcloud config set run/region $LOCATION
Updated property [run/region].
student_01_e488ef31cb8a@cloudshell:~ (qwiklabs-gcp-00-cbf0411caa40)$ 

```



## Create and deploy a public service

### Requirements

Quickway parking has a Cloud Run billing service that they would like to be made more secure. In this task, you:

- Deploy the *billing service* from an image.
- Test the service by invoking it without any authentication.

### Deploying with Cloud Run

The Quickway development team already has an image of the billing application available on Google Cloud.

```sh
student_01_e488ef31cb8a@cloudshell:~ (qwiklabs-gcp-00-cbf0411caa40)$  gcloud run deploy billing-service \
  --image gcr.io/qwiklabs-resources/gsp723-parking-service \
  --region $LOCATION \
  --allow-unauthenticated
Deploying container to Cloud Run service [billing-service] in project [qwiklabs-gcp-00-cbf0411caa40] region [us-east1]
OK Deploying new service... Done.                                                                                                                                                  
  OK Creating Revision...                                                                                                                                                          
  OK Routing traffic...                                                                                                                                                            
  OK Setting IAM Policy...                                                                                                                                                         
Done.                                                                                                                                                                              
Service [billing-service] revision [billing-service-00001-j7d] has been deployed and is serving 100 percent of traffic.
Service URL: https://billing-service-kdshgpmrmq-ue.a.run.app
student_01_e488ef31cb8a@cloudshell:~ (qwiklabs-gcp-00-cbf0411caa40)$ 
```

Assign the URL of the new service to an environment variable:

```sh
student_01_e488ef31cb8a@cloudshell:~ (qwiklabs-gcp-00-cbf0411caa40)$  BILLING_SERVICE_URL=$(gcloud run services list \
  --format='value(URL)' \
  --filter="billing-service")
student_01_e488ef31cb8a@cloudshell:~ (qwiklabs-gcp-00-cbf0411caa40)$ 
```

Invoke the service without any authorization:

```sh
student_01_e488ef31cb8a@cloudshell:~ (qwiklabs-gcp-00-cbf0411caa40)$ curl -X POST -H "Content-Type: application/json" $BILLING_SERVICE_URL -d '{"userid": "1234", "minBalance": 100}'
student_01_e488ef31cb8a@cloudshell:~ (qwiklabs-gcp-00-cbf0411caa40)$ 
```

The service does not generate any output when invoked.

In the Google Cloud console **Navigation menu** (![navmenu](https://cdn.qwiklabs.com/tkgw1TDgj4Q%2BYKQUW4jUFd0O5OEKlUMBRYbhlCrF0WY%3D)), click **Cloud Run**.

Click the link to the *billing-service*.

To view the service logs, click **Logs**.

Add the log filter `minBalance` to view the minimum balance received in the request made to the service.

```json
{
  "textPayload": "minBalance: 100",
  "insertId": "668df765000bd9dd49e91d6b",
  "resource": {
    "type": "cloud_run_revision",
    "labels": {
      "service_name": "billing-service",
      "revision_name": "billing-service-00001-j7d",
      "project_id": "qwiklabs-gcp-00-cbf0411caa40",
      "configuration_name": "billing-service",
      "location": "us-east1"
    }
  },
  "timestamp": "2024-07-10T02:52:21.776669Z",
  "labels": {
    "instanceId": "0087244a80d6a0653f36af6da0457d25f72ec8ead4c5f2fb2511d218b9322fb77618433df7e97d495d46b2bf4641cddce91b45ce3fb995efa56a86fe98b44a9e"
  },
  "logName": "projects/qwiklabs-gcp-00-cbf0411caa40/logs/run.googleapis.com%2Fstdout",
  "receiveTimestamp": "2024-07-10T02:52:21.989710705Z"
}
```



To go back to the service details page, click **<- Service details**.

Select the *billing-service* by checking the box to the left of the green check mark.

```sh
This resource is public and can be accessed by anyone on the internet. To remove public access, remove "allUsers" and "allAuthenticatedUsers" from the resource's principals. 
```

The Security team has spotted something in the security settings. Can  you see what part of the above configuration has them so concerned?

Take a closer look at the authentication applied. Currently **anyone on the internet** can call the billing service. This is indicated by the *allUsers* identity that has the *Cloud Run Invoker* role.

When the Billing service was originally deployed, it used the `--allow-unauthenticated` permission, which means that the service is publicly accessible, and can be invoked without any authentication.

By removing the `--allow-unauthenticated` permission you can use the Cloud Run default permissions to secure the service, or you can explicitly specify the `no-allow-unauthenticated` permission.

By making these changes, the Security team will be a lot happier with the overall design.



## Authenticating service requests

The team updates the application design to show how the changes will work:

The main changes are:

- Remove unauthenticated public access to the *billing service*.
- Create a new service account with appropriate permissions to invoke the *billing service*.

### Update the service to require authentication

Delete the existing deployed service:

```sh
student_01_e488ef31cb8a@cloudshell:~ (qwiklabs-gcp-00-cbf0411caa40)$  gcloud run services delete billing-service
Service [billing-service] will be deleted.

Do you want to continue (Y/n)?  y

Deleting [billing-service]...done.                                                                                                                                                 
Deleted service [billing-service].
student_01_e488ef31cb8a@cloudshell:~ (qwiklabs-gcp-00-cbf0411caa40)$ 
```

Redeploy the *billing service* with the `--no-allow-authenticated` permission:

```sh
student_01_e488ef31cb8a@cloudshell:~ (qwiklabs-gcp-00-cbf0411caa40)$  gcloud run deploy billing-service \
  --image gcr.io/qwiklabs-resources/gsp723-parking-service \
  --region $LOCATION \
  --no-allow-unauthenticated
Deploying container to Cloud Run service [billing-service] in project [qwiklabs-gcp-00-cbf0411caa40] region [us-east1]
OK Deploying new service... Done.                                                                                                                                                  
  OK Creating Revision...                                                                                                                                                          
  OK Routing traffic...                                                                                                                                                            
Done.                                                                                                                                                                              
Service [billing-service] revision [billing-service-00001-wdh] has been deployed and is serving 100 percent of traffic.
Service URL: https://billing-service-kdshgpmrmq-ue.a.run.app
student_01_e488ef31cb8a@cloudshell:~ (qwiklabs-gcp-00-cbf0411caa40)$ 
```

Redeploying the service means it no longer allows unauthenticated access at its service URL. In addition, the access permission to invoke the  service has been removed.

Wait a few seconds, and then invoke the *billing service* again as before:

```sh
student_01_e488ef31cb8a@cloudshell:~ (qwiklabs-gcp-00-cbf0411caa40)$  curl -X POST -H "Content-Type: application/json" $BILLING_SERVICE_URL -d '{"userid": "1234", "minBalance": 100}'

<html><head>
<meta http-equiv="content-type" content="text/html;charset=utf-8">
<title>403 Forbidden</title>
</head>
<body text=#000000 bgcolor=#ffffff>
<h1>Error: Forbidden</h1>
<h2>Your client does not have permission to get URL <code>/</code> from this server.</h2>
<h2></h2>
</body></html>
student_01_e488ef31cb8a@cloudshell:~ (qwiklabs-gcp-00-cbf0411caa40)$ 
```

As expected, the output is a permissions error since the service now requires authentication.

Now that you understand more about the permissions used with Cloud  Run, correct the authentication permissions applied to the Billing  service:

### Create a service account

To invoke the *billing service* you will need an identity or service account with appropriate permissions, and bind that identity to the service.

This can be done in the Google Cloud console, or with the gcloud  command line interface. In this lab, you use the Google Cloud console to create the service account and set up the new policy binding for the *billing service*.

1. In the Google Cloud console **Navigation menu** (![navmenu](https://cdn.qwiklabs.com/tkgw1TDgj4Q%2BYKQUW4jUFd0O5OEKlUMBRYbhlCrF0WY%3D)), select **IAM & Admin > Service Accounts**.

2. To create a new service account that will provide authenticated access, click **Create Service Account** near the top.

3. Name the service account: `Billing Initiator`.

4. To create the account, click **Create and Continue**, and then advance to the **Grant Access** step.

   To give the *Billing Initiator* service account permissions to invoke the *billing service*, select the **Role** drop-down, scroll the left side to `Cloud Run`, and then select the role `Cloud Run Invoker`.

5. To complete the setup of the service account, click **Continue**, and then click **Done**.

   You will see the new service account at the top of the list of service accounts in the console.

The service account **Billing Initiator** has been created with the authorization to invoke a Cloud Run service, using an IAM policy binding on your project.

## Invoke the service with authentication

Now that you have a service account with the appropriate permission, you can use it to invoke your Cloud Run service.

### Authenticate with gcloud

The first step is to set the service account in gcloud so it can be used to authenticate with the service.

The first step is to set the service account in gcloud so it can be used to authenticate with the service.

1. In the Cloud Shell terminal menu, open a new shell in a separate tab by clicking **Add** (![add](https://cdn.qwiklabs.com/Ku8WDO0HmyEnieuJFNYyzbvTz2VUPldrUYH2ORP6XYk%3D)).

   Execute the remaining commands of this task in this Cloud Shell window.

2. Get the service account identity email and save it in an environment variable:

```sh
Welcome to Cloud Shell! Type "help" to get started.
Your Cloud Platform project in this session is set to qwiklabs-gcp-00-cbf0411caa40.
Use “gcloud config set project [PROJECT_ID]” to change to a different project.
student_01_e488ef31cb8a@cloudshell:~ (qwiklabs-gcp-00-cbf0411caa40)$  BILLING_INITIATOR_EMAIL=$(gcloud iam service-accounts list --filter="Billing Initiator" --format="value(EMAIL)"); echo $BILLING_INITIATOR_EMAIL
billing-initiator@qwiklabs-gcp-00-cbf0411caa40.iam.gserviceaccount.com
student_01_e488ef31cb8a@cloudshell:~ (qwiklabs-gcp-00-cbf0411caa40)$ 

```

In this Cloud Shell terminal, assign the URL of the *billing service* to an environment variable:

```sh
student_01_e488ef31cb8a@cloudshell:~ (qwiklabs-gcp-00-cbf0411caa40)$  BILLING_SERVICE_URL=$(gcloud run services list \
  --format='value(URL)' \
  --filter="billing-service")
student_01_e488ef31cb8a@cloudshell:~ (qwiklabs-gcp-00-cbf0411caa40)$ echo $BILLING_SERVICE_URL 
https://billing-service-kdshgpmrmq-ue.a.run.app
student_01_e488ef31cb8a@cloudshell:~ (qwiklabs-gcp-00-cbf0411caa40)$ 

```

To authenticate gcloud using the service account, generate a key file:

```sh
student_01_e488ef31cb8a@cloudshell:~ (qwiklabs-gcp-00-cbf0411caa40)$ gcloud iam service-accounts keys create key.json --iam-account=${BILLING_INITIATOR_EMAIL}
created key [ce9a7921bd950f3eada4bf2191e647887feb0346] of type [json] as [key.json] for [billing-initiator@qwiklabs-gcp-00-cbf0411caa40.iam.gserviceaccount.com]
student_01_e488ef31cb8a@cloudshell:~ (qwiklabs-gcp-00-cbf0411caa40)$ 
```

Authorize access to Cloud Run with a service account:

```sh
student_01_e488ef31cb8a@cloudshell:~ (qwiklabs-gcp-00-cbf0411caa40)$ gcloud auth activate-service-account --key-file=key.json
Activated service account credentials for: [billing-initiator@qwiklabs-gcp-00-cbf0411caa40.iam.gserviceaccount.com]
student_01_e488ef31cb8a@cloudshell:~ (qwiklabs-gcp-00-cbf0411caa40)$ 
```



### Invoke the service

1. Invoke the Cloud Run *billing service* with an identity token generated from the service account:

```sh
student_01_e488ef31cb8a@cloudshell:~ (qwiklabs-gcp-00-cbf0411caa40)$  curl -X POST -H "Content-Type: application/json" \
  -H "Authorization: Bearer $(gcloud auth print-identity-token)" \
  $BILLING_SERVICE_URL -d '{"userid": "1234", "minBalance": 500}'
student_01_e488ef31cb8a@cloudshell:~ (qwiklabs-gcp-00-cbf0411caa40)$ 
```

In the Google Cloud console **Navigation menu** (![navmenu](https://cdn.qwiklabs.com/tkgw1TDgj4Q%2BYKQUW4jUFd0O5OEKlUMBRYbhlCrF0WY%3D)), click **Cloud Run**.

Click the link to the *billing-service*.

To view the service logs, click **Logs**.

Add the log filter `minBalance` to view the updated minimum balance received in the request made to the service.

```json
{
  "textPayload": "minBalance: 500",
  "insertId": "668dfae70008d14a5e127904",
  "resource": {
    "type": "cloud_run_revision",
    "labels": {
      "project_id": "qwiklabs-gcp-00-cbf0411caa40",
      "service_name": "billing-service",
      "configuration_name": "billing-service",
      "location": "us-east1",
      "revision_name": "billing-service-00001-wdh"
    }
  },
  "timestamp": "2024-07-10T03:07:19.577866Z",
  "labels": {
    "instanceId": "0087244a80f951561e9ab853af137cc82d496a8311c679fe37ba1bc05c1f59bccc2f277983f0c6af8e57a79201f516e07882901466c26f8867e0e0f5afeeff7f"
  },
  "logName": "projects/qwiklabs-gcp-00-cbf0411caa40/logs/run.googleapis.com%2Fstdout",
  "receiveTimestamp": "2024-07-10T03:07:19.645458977Z"
}
```



## Implement least privilege

You've used a service account with the appropriate permissions to  invoke a Cloud Run service that was previously accessible by anyone.  But, have you used the absolute minimum privileges needed to call this  specific service?

To determine if this is true, deploy a second billing service which  we will assume should be accessible only by other internal private  services, such as Cloud Scheduler.

### Deploy a second service

1. Open a third Cloud Shell terminal window or tab.
2. Create a LOCATION environment variable:
3. To simulate a second service, deploy the billing application image to Cloud Run:

```sh
student_01_e488ef31cb8a@cloudshell:~ (qwiklabs-gcp-00-cbf0411caa40)$  LOCATION=us-east1
student_01_e488ef31cb8a@cloudshell:~ (qwiklabs-gcp-00-cbf0411caa40)$  gcloud run deploy billing-service-2 \
  --image gcr.io/qwiklabs-resources/gsp723-parking-service \
  --region $LOCATION \
  --no-allow-unauthenticated
Deploying container to Cloud Run service [billing-service-2] in project [qwiklabs-gcp-00-cbf0411caa40] region [us-east1]
OK Deploying new service... Done.                                                                                                                                                  
  OK Creating Revision...                                                                                                                                                          
  OK Routing traffic...                                                                                                                                                            
Done.                                                                                                                                                                              
Service [billing-service-2] revision [billing-service-2-00001-pbf] has been deployed and is serving 100 percent of traffic.
Service URL: https://billing-service-2-kdshgpmrmq-ue.a.run.app
student_01_e488ef31cb8a@cloudshell:~ (qwiklabs-gcp-00-cbf0411caa40)$ 
```

Assign the URL of the new service to an environment variable:

```sh
student_01_e488ef31cb8a@cloudshell:~ (qwiklabs-gcp-00-cbf0411caa40)$  BILLING_SERVICE_2_URL=$(gcloud run services list \
  --format='value(URL)' \
  --filter="billing-service-2")
student_01_e488ef31cb8a@cloudshell:~ (qwiklabs-gcp-00-cbf0411caa40)$ 
```

### Invoke the second service with the service account identity

1. In this third Cloud Shell terminal, authorize access to Cloud Run with the same service account:

```sh
student_01_e488ef31cb8a@cloudshell:~ (qwiklabs-gcp-00-cbf0411caa40)$ gcloud auth activate-service-account --key-file=key.json
Activated service account credentials for: [billing-initiator@qwiklabs-gcp-00-cbf0411caa40.iam.gserviceaccount.com]
student_01_e488ef31cb8a@cloudshell:~ (qwiklabs-gcp-00-cbf0411caa40)$  curl -X POST -H "Content-Type: application/json" \
  -H "Authorization: Bearer $(gcloud auth print-identity-token)" \
  $BILLING_SERVICE_2_URL -d '{"userid": "1234", "minBalance": 900}'
student_01_e488ef31cb8a@cloudshell:~ (qwiklabs-gcp-00-cbf0411caa40)$ 
```

Why was this successful!? It's because when you created the service  account, the Cloud Run Invoker permissions were granted to this account  on the project. Because of inheritance, resources in the project such as the two Cloud Run services inherit those permissions, and as a result,  the service account can be used to invoke the services.



### Restrict service account permissions

To fully implement least privilege, the service account should only be granted permissions on the service that it needs.

In this subtask, you remove the permission previously granted to the  service account on the project, and then add the appropriate permissions required to invoke the original billing service.

1. Switch to the first Cloud Shell terminal window.
2. In this Cloud Shell terminal, get the service account identity email and save it in an environment variable:
3. Remove the permission on the service account for the project:
4. Add the permission to the service account on the *billing service*:

```sh
student_01_e488ef31cb8a@cloudshell:~ (qwiklabs-gcp-00-cbf0411caa40)$  BILLING_INITIATOR_EMAIL=$(gcloud iam service-accounts list --filter="Billing Initiator" --format="value(EMAIL)"); echo $BILLING_INITIATOR_EMAIL
billing-initiator@qwiklabs-gcp-00-cbf0411caa40.iam.gserviceaccount.com
student_01_e488ef31cb8a@cloudshell:~ (qwiklabs-gcp-00-cbf0411caa40)$
```
```sh
student_01_e488ef31cb8a@cloudshell:~ (qwiklabs-gcp-00-cbf0411caa40)$ gcloud projects remove-iam-policy-binding $GOOGLE_CLOUD_PROJECT \
  --member=serviceAccount:${BILLING_INITIATOR_EMAIL} \
  --role=roles/run.invoker
Updated IAM policy for project [qwiklabs-gcp-00-cbf0411caa40].
bindings:
- members:
  - serviceAccount:qwiklabs-gcp-00-cbf0411caa40@qwiklabs-gcp-00-cbf0411caa40.iam.gserviceaccount.com
  role: roles/bigquery.admin
- members:
  - serviceAccount:27047558188@cloudbuild.gserviceaccount.com
  role: roles/cloudbuild.builds.builder
- members:
  - serviceAccount:service-27047558188@gcp-sa-cloudbuild.iam.gserviceaccount.com
  role: roles/cloudbuild.serviceAgent
- members:
  - serviceAccount:service-27047558188@compute-system.iam.gserviceaccount.com
  role: roles/compute.serviceAgent
- members:
  - serviceAccount:service-27047558188@container-engine-robot.iam.gserviceaccount.com
  role: roles/container.serviceAgent
- members:
  - serviceAccount:27047558188-compute@developer.gserviceaccount.com
  - serviceAccount:27047558188@cloudservices.gserviceaccount.com
  role: roles/editor
- members:
  - serviceAccount:admiral@qwiklabs-services-prod.iam.gserviceaccount.com
  - serviceAccount:qwiklabs-gcp-00-cbf0411caa40@qwiklabs-gcp-00-cbf0411caa40.iam.gserviceaccount.com
  - user:student-01-e488ef31cb8a@qwiklabs.net
  role: roles/owner
- members:
  - serviceAccount:service-27047558188@serverless-robot-prod.iam.gserviceaccount.com
  role: roles/run.serviceAgent
- members:
  - serviceAccount:qwiklabs-gcp-00-cbf0411caa40@qwiklabs-gcp-00-cbf0411caa40.iam.gserviceaccount.com
  role: roles/storage.admin
- members:
  - user:student-01-e488ef31cb8a@qwiklabs.net
  role: roles/viewer
etag: BwYc3Ak1xZo=
version: 1
```

```sh
student_01_e488ef31cb8a@cloudshell:~ (qwiklabs-gcp-00-cbf0411caa40)$ gcloud run services add-iam-policy-binding billing-service --region $LOCATION \
  --member=serviceAccount:${BILLING_INITIATOR_EMAIL} \
  --role=roles/run.invoker --platform managed
Updated IAM policy for service [billing-service].
bindings:
- members:
  - serviceAccount:billing-initiator@qwiklabs-gcp-00-cbf0411caa40.iam.gserviceaccount.com
  role: roles/run.invoker
etag: BwYc3An6smw=
version: 1
student_01_e488ef31cb8a@cloudshell:~ (qwiklabs-gcp-00-cbf0411caa40)$ 
```

### Invoke the services

1. Switch back to the second Cloud Shell terminal window or tab.
2. Wait a few seconds, and then invoke the first Cloud Run *billing service* with an identity token generated from the service account:

```sh
student_01_e488ef31cb8a@cloudshell:~ (qwiklabs-gcp-00-cbf0411caa40)$  curl -X POST -H "Content-Type: application/json" \
  -H "Authorization: Bearer $(gcloud auth print-identity-token)" \
  $BILLING_SERVICE_URL -d '{"userid": "1234", "minBalance": 700}'
student_01_e488ef31cb8a@cloudshell:~ (qwiklabs-gcp-00-cbf0411caa40)$ 
```

It takes a few seconds for the updated permissions to propagate, after which this invocation should be successful.

Switch to the third Cloud Shell terminal window.

Try to invoke the second Cloud Run service with an identity token generated from the same service account:

```sh
student_01_e488ef31cb8a@cloudshell:~ (qwiklabs-gcp-00-cbf0411caa40)$  curl -X POST -H "Content-Type: application/json" \
  -H "Authorization: Bearer $(gcloud auth print-identity-token)" \
  $BILLING_SERVICE_2_URL -d '{"userid": "1234", "minBalance": 500}'

<html><head>
<meta http-equiv="content-type" content="text/html;charset=utf-8">
<title>403 Forbidden</title>
</head>
<body text=#000000 bgcolor=#ffffff>
<h1>Error: Forbidden</h1>
<h2>Your client does not have permission to get URL <code>/</code> from this server.</h2>
<h2></h2>
</body></html>
student_01_e488ef31cb8a@cloudshell:~ (qwiklabs-gcp-00-cbf0411caa40)$ 
```

You should now receive a permissions error indicating that the service  account has only the minimum set of permissions required to invoke the  first service.

In this lab, you have seen how to reconfigure a deployed service to  secure access to it, and implemented the principle of least privilege  when granting permissions to access resources on Google Cloud. You:

- Deployed a service to Cloud Run.
- Used the gcloud CLI to update the service to require authentication.
- Created a service account with the required permissions to invoke the service.
- Set the minimum permissions required to invoke a specific service on Cloud Run, implementing least privilege.

Here's a diagram of this requirement:

![Requirement diagram showing access to a second internal billing service.](https://cdn.qwiklabs.com/XjqZiYDWe1PDhnQSuHLLLQgH7DLK%2BR5ZWXX381c34Lk%3D)

## History

```sh
student_01_e488ef31cb8a@cloudshell:~ (qwiklabs-gcp-00-cbf0411caa40)$ history 
    1   gcloud services enable run.googleapis.com
    2   LOCATION=us-east1
    3  gcloud config set run/region $LOCATION
    4   gcloud run deploy billing-service   --image gcr.io/qwiklabs-resources/gsp723-parking-service   --region $LOCATION   --allow-unauthenticated
    5   BILLING_SERVICE_URL=$(gcloud run services list \
  --format='value(URL)' \
  --filter="billing-service")
    6  curl -X POST -H "Content-Type: application/json" $BILLING_SERVICE_URL -d '{"userid": "1234", "minBalance": 100}'
    7   gcloud run services delete billing-service
    8   gcloud run deploy billing-service   --image gcr.io/qwiklabs-resources/gsp723-parking-service   --region $LOCATION   --no-allow-unauthenticated
    9   curl -X POST -H "Content-Type: application/json" $BILLING_SERVICE_URL -d '{"userid": "1234", "minBalance": 100}'
   10   BILLING_INITIATOR_EMAIL=$(gcloud iam service-accounts list --filter="Billing Initiator" --format="value(EMAIL)"); echo $BILLING_INITIATOR_EMAIL
   11  gcloud projects remove-iam-policy-binding $GOOGLE_CLOUD_PROJECT   --member=serviceAccount:${BILLING_INITIATOR_EMAIL}   --role=roles/run.invoker
   12  gcloud run services add-iam-policy-binding billing-service --region $LOCATION   --member=serviceAccount:${BILLING_INITIATOR_EMAIL}   --role=roles/run.invoker --platform managed
   13  history 
student_01_e488ef31cb8a@cloudshell:~ (qwiklabs-gcp-00-cbf0411caa40)$ 
```

```sh
student_01_e488ef31cb8a@cloudshell:~ (qwiklabs-gcp-00-cbf0411caa40)$ history 
    1   gcloud services enable run.googleapis.com
    2   LOCATION=us-east1
    3  gcloud config set run/region $LOCATION
    4   gcloud run deploy billing-service   --image gcr.io/qwiklabs-resources/gsp723-parking-service   --region $LOCATION   --allow-unauthenticated
    5   BILLING_SERVICE_URL=$(gcloud run services list \
    6    --format='value(URL)' \
    7    --filter="billing-service")
    8  curl -X POST -H "Content-Type: application/json" $BILLING_SERVICE_URL -d '{"userid": "1234", "minBalance": 100}'
    9   gcloud run services delete billing-service
   10   gcloud run deploy billing-service   --image gcr.io/qwiklabs-resources/gsp723-parking-service   --region $LOCATION   --no-allow-unauthenticated
   11   curl -X POST -H "Content-Type: application/json" $BILLING_SERVICE_URL -d '{"userid": "1234", "minBalance": 100}'
   12   BILLING_INITIATOR_EMAIL=$(gcloud iam service-accounts list --filter="Billing Initiator" --format="value(EMAIL)"); echo $BILLING_INITIATOR_EMAIL
   13   BILLING_SERVICE_URL=$(gcloud run services list \
  --format='value(URL)' \
  --filter="billing-service")
   14  echo $BILLING_SERVICE_URL 
   15  gcloud iam service-accounts keys create key.json --iam-account=${BILLING_INITIATOR_EMAIL}
   16  gcloud auth activate-service-account --key-file=key.json
   17   curl -X POST -H "Content-Type: application/json"   -H "Authorization: Bearer $(gcloud auth print-identity-token)"   $BILLING_SERVICE_URL -d '{"userid": "1234", "minBalance": 500}'
   18   LOCATION=us-east1
   19   curl -X POST -H "Content-Type: application/json"   -H "Authorization: Bearer $(gcloud auth print-identity-token)"   $BILLING_SERVICE_URL -d '{"userid": "1234", "minBalance": 700}'
   20  history 
student_01_e488ef31cb8a@cloudshell:~ (qwiklabs-gcp-00-cbf0411caa40)$ 
```

```sh
student_01_e488ef31cb8a@cloudshell:~ (qwiklabs-gcp-00-cbf0411caa40)$ history 
    1   gcloud services enable run.googleapis.com
    2   LOCATION=us-east1
    3  gcloud config set run/region $LOCATION
    4   gcloud run deploy billing-service   --image gcr.io/qwiklabs-resources/gsp723-parking-service   --region $LOCATION   --allow-unauthenticated
    5   BILLING_SERVICE_URL=$(gcloud run services list \
    6    --format='value(URL)' \
    7    --filter="billing-service")
    8  curl -X POST -H "Content-Type: application/json" $BILLING_SERVICE_URL -d '{"userid": "1234", "minBalance": 100}'
    9   gcloud run services delete billing-service
   10   gcloud run deploy billing-service   --image gcr.io/qwiklabs-resources/gsp723-parking-service   --region $LOCATION   --no-allow-unauthenticated
   11   curl -X POST -H "Content-Type: application/json" $BILLING_SERVICE_URL -d '{"userid": "1234", "minBalance": 100}'
   12   BILLING_INITIATOR_EMAIL=$(gcloud iam service-accounts list --filter="Billing Initiator" --format="value(EMAIL)"); echo $BILLING_INITIATOR_EMAIL
   13   BILLING_SERVICE_URL=$(gcloud run services list \
   14    --format='value(URL)' \
   15    --filter="billing-service")
   16  echo $BILLING_SERVICE_URL 
   17  gcloud iam service-accounts keys create key.json --iam-account=${BILLING_INITIATOR_EMAIL}
   18  gcloud auth activate-service-account --key-file=key.json
   19   curl -X POST -H "Content-Type: application/json"   -H "Authorization: Bearer $(gcloud auth print-identity-token)"   $BILLING_SERVICE_URL -d '{"userid": "1234", "minBalance": 500}'
   20   LOCATION=us-east1
   21   LOCATION=us-east1
   22   gcloud run deploy billing-service-2   --image gcr.io/qwiklabs-resources/gsp723-parking-service   --region $LOCATION   --no-allow-unauthenticated
   23   BILLING_SERVICE_2_URL=$(gcloud run services list \
  --format='value(URL)' \
  --filter="billing-service-2")
   24  gcloud auth activate-service-account --key-file=key.json
   25   curl -X POST -H "Content-Type: application/json"   -H "Authorization: Bearer $(gcloud auth print-identity-token)"   $BILLING_SERVICE_2_URL -d '{"userid": "1234", "minBalance": 900}'
   26   curl -X POST -H "Content-Type: application/json"   -H "Authorization: Bearer $(gcloud auth print-identity-token)"   $BILLING_SERVICE_2_URL -d '{"userid": "1234", "minBalance": 500}'
   27  history 
student_01_e488ef31cb8a@cloudshell:~ (qwiklabs-gcp-00-cbf0411caa40)$ 
```

