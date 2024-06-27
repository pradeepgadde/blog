---

layout: single
title:  "API Gateway: Qwik Start"
categories: Cloud
tags: GCP
classes: wide
show_date: true
header:
  overlay_image: /assets/images/gcp-banner-1.png
  og_image: /assets/images/gcp-banner-1.png
  teaser: /assets/images/pca-gcp.png
author:
  name     : "Professional Cloud Architect"
  avatar   : "/assets/images/pca-gcp.png"

sidebar:
  - title: "Topics"
    nav: my-sidebar

---
# API Gateway: Qwik Start

API Gateway enables you to provide secure access to your services  through a well-defined REST API that is consistent across all of your  services, regardless of service implementation. A consistent API:

- Makes it easy for app developers to consume your services.
- Enables you to change the backend service implementation without affecting the public API.
- Enables you to take advantage of the scaling, monitoring, and security features built into the Google Cloud.

## Deploying an API backend

API Gateway sits in front of a deployed backend service and handles all  incoming requests. In this lab, API Gateway routes incoming calls to a  Cloud Function backend named **helloGET** that contains the function shown below:



## Test the API backend

1. When the function finishes deploying, take note of the `httpsTrigger`'s url property or find it using the following command:

### Create the API definition

API Gateway uses an API definition to route calls to the backend  service. You can use an OpenAPI spec that contains specialized  annotations to define the desired API Gateway behavior. The OpenAPI spec for this quickstart contains routing instructions to the Cloud Function backend.



## Creating a gateway

Now you are ready to create and deploy a gateway on API Gateway.

1. In the top search bar enter **API Gateway** and select it from the options that appear.
2. Click **Create Gateway**. Then, in the **APIs** section:

- Ensure the **Select an API** input is set to **Create new API**.
- For **Display Name** enter `Hello World API`
- For **API ID**, run the following command to once again obtain the API ID and enter it into the **API ID** field:
- In the **API Config** section:

- Ensure the **Select a Config** input is set to **Create new API config**.
- Do the following to upload the `openapi2-functions.yaml` file previously created.

1. In Cloud Shell, run the following command:
2. Click **Download**.
3. Select **Browse** and select the file from the browser's download location:

- Enter `Hello World Config` in the **Display Name** field.
- Ensure the **Select a Service Account** input is set to **Compute Engine default service account**.

1. In the **Gateway details** Section:

- Enter `Hello Gateway` in the **Display Name** field.
- Set the **Location** drop down to .

1. Click **Create Gateway**.

### Testing your API Deployment

Now you can send requests to your API using the URL generated upon deployment of your gateway.

1. In Cloud Shell, enter the following command to retrieve the `GATEWAY_URL` of the newly created API hosted by API Gateway:



## Securing access by using an API key

To secure access to your API backend, you can generate an API key  associated with your project and grant that key access to call your API. To create an API Key you must do the following:

1. In the Cloud Console, navigate to **APIs & Services** > **Credentials**.
2. Select **Create credentials**, then select **API Key** from the dropdown menu. The **API key created** dialog box displays your newly created key.





## Create and deploy a new API config to your existing gateway

1. Open the **API Gateway** page in Cloud Console. (Click **Navigation Menu > API Gateway**.)
2. Select your API from the list to view details.
3. Select the **Gateways** tab.
4. Select `Hello Gateway` from the list of available **Gateways**.
5. Click on `Edit` at the top of the Gateway page.
6. Under **API Config** change the drop down to `Create new API config`.
7. Click **Browse** in the **Upload an API Spec** input box and select the `openapi2-functions2.yaml` file.
8. Enter `Hello Config` for **Display Name**.
9. Select `Qwiklabs User Service Account` for **Select a Service Account**.
10. Click **Update**.



## Testing calls using your API key

1. To test using your API key run the following command:
