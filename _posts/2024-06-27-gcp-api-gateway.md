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

### Enable the required APIs

1. In the Cloud Console, on the Navigation menu (![Navigation menu icon](https://cdn.qwiklabs.com/tkgw1TDgj4Q%2BYKQUW4jUFd0O5OEKlUMBRYbhlCrF0WY%3D)) click  **APIs & Services > Library**.
2. Start typing "api gateway" in the **Search** bar, then select the **API Gateway API** tile.
3. Now click the **Enable** button on the next screen.

## Deploying an API backend

API Gateway sits in front of a deployed backend service and handles all  incoming requests. In this lab, API Gateway routes incoming calls to a  Cloud Function backend named **helloGET** that contains the function shown below:

```sh
Welcome to Cloud Shell! Type "help" to get started.
Your Cloud Platform project in this session is set to qwiklabs-gcp-02-c45fb2422cb2.
Use “gcloud config set project [PROJECT_ID]” to change to a different project.
student_01_932d053b64d1@cloudshell:~ (qwiklabs-gcp-02-c45fb2422cb2)$ gcloud config set compute/region europe-west1
WARNING: Property validation for compute/region was skipped.
Updated property [compute/region].
student_01_932d053b64d1@cloudshell:~ (qwiklabs-gcp-02-c45fb2422cb2)$ git clone https://github.com/GoogleCloudPlatform/nodejs-docs-samples.git
Cloning into 'nodejs-docs-samples'...
remote: Enumerating objects: 55080, done.
remote: Counting objects: 100% (96/96), done.
remote: Compressing objects: 100% (50/50), done.
remote: Total 55080 (delta 37), reused 85 (delta 32), pack-reused 54984
Receiving objects: 100% (55080/55080), 77.61 MiB | 12.92 MiB/s, done.
Resolving deltas: 100% (37789/37789), done.
student_01_932d053b64d1@cloudshell:~ (qwiklabs-gcp-02-c45fb2422cb2)$ cd nodejs-docs-samples/functions/helloworld/helloworldGet
student_01_932d053b64d1@cloudshell:~/nodejs-docs-samples/functions/helloworld/helloworldGet (qwiklabs-gcp-02-c45fb2422cb2)$ ls
index.js  package.json  test
student_01_932d053b64d1@cloudshell:~/nodejs-docs-samples/functions/helloworld/helloworldGet (qwiklabs-gcp-02-c45fb2422cb2)$
```

```js
student_01_932d053b64d1@cloudshell:~/nodejs-docs-samples/functions/helloworld/helloworldGet (qwiklabs-gcp-02-c45fb2422cb2)$ cat index.js 
// Copyright 2022 Google LLC
//
// Licensed under the Apache License, Version 2.0 (the "License");
// you may not use this file except in compliance with the License.
// You may obtain a copy of the License at
//
//      http://www.apache.org/licenses/LICENSE-2.0
//
// Unless required by applicable law or agreed to in writing, software
// distributed under the License is distributed on an "AS IS" BASIS,
// WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
// See the License for the specific language governing permissions and
// limitations under the License.

'use strict';

// [START functions_helloworld_get]
const functions = require('@google-cloud/functions-framework');

// Register an HTTP function with the Functions Framework that will be executed
// when you make an HTTP request to the deployed function's endpoint.
functions.http('helloGET', (req, res) => {
  res.send('Hello World!');
});
// [END functions_helloworld_get]
student_01_932d053b64d1@cloudshell:~/nodejs-docs-samples/functions/helloworld/helloworldGet (qwiklabs-gcp-02-c45fb2422cb2)$ 
```

To deploy the function with an HTTP trigger, run the following command in the directory containing your function:

```sh
student_01_932d053b64d1@cloudshell:~/nodejs-docs-samples/functions/helloworld/helloworldGet (qwiklabs-gcp-02-c45fb2422cb2)$ gcloud functions deploy helloGET --runtime nodejs14 --trigger-http --allow-unauthenticated --region europe-west1
In a future Cloud SDK release, new functions will be deployed as 2nd gen  functions by default. This is equivalent to currently deploying new  with the --gen2 flag. Existing 1st gen functions will not be impacted and will continue to deploy as 1st gen functions.
You can preview this behavior in beta. Alternatively, you can disable this behavior by explicitly specifying the --no-gen2 flag or by setting the functions/gen2 config property to 'off'.
To learn more about the differences between 1st gen and 2nd gen functions, visit:
https://cloud.google.com/functions/docs/concepts/version-comparison
WARNING: Node.js 14 is no longer supported by the Node.js community as of 30 April, 2023. Runtime nodejs14 is currently deprecated for Cloud Functions. We recommend you to upgrade to the latest version of Node.js as soon as possible.
ERROR: (gcloud.functions.deploy) ResponseError: status=[403], code=[Ok], message=[Unable to retrieve the repository metadata for projects/qwiklabs-gcp-02-c45fb2422cb2/locations/europe-west1/repositories/gcf-artifacts. Ensure that the Cloud Functions service account has 'artifactregistry.repositories.list' and 'artifactregistry.repositories.get' permissions. You can add the permissions by granting the role 'roles/artifactregistry.reader'.]
student_01_932d053b64d1@cloudshell:~/nodejs-docs-samples/functions/helloworld/helloworldGet (qwiklabs-gcp-02-c45fb2422cb2)$
```

If you receive an Error as **IamPermissionDeniedException** rerun the above command. 

```sh
student_01_932d053b64d1@cloudshell:~/nodejs-docs-samples/functions/helloworld/helloworldGet (qwiklabs-gcp-02-c45fb2422cb2)$ gcloud functions deploy helloGET --runtime nodejs14 --trigger-http --allow-unauthenticated --region europe-west1
In a future Cloud SDK release, new functions will be deployed as 2nd gen  functions by default. This is equivalent to currently deploying new  with the --gen2 flag. Existing 1st gen functions will not be impacted and will continue to deploy as 1st gen functions.
You can preview this behavior in beta. Alternatively, you can disable this behavior by explicitly specifying the --no-gen2 flag or by setting the functions/gen2 config property to 'off'.
To learn more about the differences between 1st gen and 2nd gen functions, visit:
https://cloud.google.com/functions/docs/concepts/version-comparison
WARNING: Node.js 14 is no longer supported by the Node.js community as of 30 April, 2023. Runtime nodejs14 is currently deprecated for Cloud Functions. We recommend you to upgrade to the latest version of Node.js as soon as possible.
Deploying function (may take a while - up to 2 minutes)...working...                                                                                                               
For Cloud Build Logs, visit: https://console.cloud.google.com/cloud-build/builds;region=europe-west1/c0797b03-fd9d-4386-9f5a-bb84b57602c6?project=241556485876
Deploying function (may take a while - up to 2 minutes)...done.                                                                                                                    
automaticUpdatePolicy: {}
availableMemoryMb: 256
buildId: c0797b03-fd9d-4386-9f5a-bb84b57602c6
buildName: projects/241556485876/locations/europe-west1/builds/c0797b03-fd9d-4386-9f5a-bb84b57602c6
dockerRegistry: ARTIFACT_REGISTRY
entryPoint: helloGET
httpsTrigger:
  securityLevel: SECURE_ALWAYS
  url: https://europe-west1-qwiklabs-gcp-02-c45fb2422cb2.cloudfunctions.net/helloGET
ingressSettings: ALLOW_ALL
labels:
  deployment-tool: cli-gcloud
maxInstances: 5
name: projects/qwiklabs-gcp-02-c45fb2422cb2/locations/europe-west1/functions/helloGET
runtime: nodejs14
serviceAccountEmail: qwiklabs-gcp-02-c45fb2422cb2@appspot.gserviceaccount.com
sourceUploadUrl: https://storage.googleapis.com/uploads-777776350147.europe-west1.cloudfunctions.appspot.com/552cd0a4-b64d-43c1-a435-d30af3d522d5.zip
status: ACTIVE
timeout: 60s
updateTime: '2024-06-27T11:35:33.350Z'
versionId: '1'
student_01_932d053b64d1@cloudshell:~/nodejs-docs-samples/functions/helloworld/helloworldGet (qwiklabs-gcp-02-c45fb2422cb2)$ 
```



## Test the API backend

1. When the function finishes deploying, take note of the `httpsTrigger`'s url property or find it using the following command:

```sh
student_01_932d053b64d1@cloudshell:~/nodejs-docs-samples/functions/helloworld/helloworldGet (qwiklabs-gcp-02-c45fb2422cb2)$ gcloud functions describe helloGET --region europe-west1automaticUpdatePolicy: {}
availableMemoryMb: 256
buildId: c0797b03-fd9d-4386-9f5a-bb84b57602c6
buildName: projects/241556485876/locations/europe-west1/builds/c0797b03-fd9d-4386-9f5a-bb84b57602c6
dockerRegistry: ARTIFACT_REGISTRY
entryPoint: helloGET
httpsTrigger:
  securityLevel: SECURE_ALWAYS
  url: https://europe-west1-qwiklabs-gcp-02-c45fb2422cb2.cloudfunctions.net/helloGET
ingressSettings: ALLOW_ALL
labels:
  deployment-tool: cli-gcloud
maxInstances: 5
name: projects/qwiklabs-gcp-02-c45fb2422cb2/locations/europe-west1/functions/helloGET
runtime: nodejs14
serviceAccountEmail: qwiklabs-gcp-02-c45fb2422cb2@appspot.gserviceaccount.com
sourceUploadUrl: https://storage.googleapis.com/uploads-777776350147.europe-west1.cloudfunctions.appspot.com/552cd0a4-b64d-43c1-a435-d30af3d522d5.zip
status: ACTIVE
timeout: 60s
updateTime: '2024-06-27T11:35:33.350Z'
versionId: '1'
student_01_932d053b64d1@cloudshell:~/nodejs-docs-samples/functions/helloworld/helloworldGet (qwiklabs-gcp-02-c45fb2422cb2)$ 
```
Visit the URL to invoke the Cloud Function. You should see the message Hello World! as the response:

```sh
student_01_932d053b64d1@cloudshell:~/nodejs-docs-samples/functions/helloworld/helloworldGet (qwiklabs-gcp-02-c45fb2422cb2)$ export PROJECT_ID=qwiklabs-gcp-02-c45fb2422cb2
student_01_932d053b64d1@cloudshell:~/nodejs-docs-samples/functions/helloworld/helloworldGet (qwiklabs-gcp-02-c45fb2422cb2)$ curl -v https://europe-west1-qwiklabs-gcp-02-c45fb2422cb2.cloudfunctions.net/helloGET
*   Trying 216.239.36.54:443...
* Connected to europe-west1-qwiklabs-gcp-02-c45fb2422cb2.cloudfunctions.net (216.239.36.54) port 443 (#0)
* ALPN, offering h2
* ALPN, offering http/1.1
*  CAfile: /etc/ssl/certs/ca-certificates.crt
*  CApath: /etc/ssl/certs
* TLSv1.0 (OUT), TLS header, Certificate Status (22):
* TLSv1.3 (OUT), TLS handshake, Client hello (1):
* TLSv1.2 (IN), TLS header, Certificate Status (22):
* TLSv1.3 (IN), TLS handshake, Server hello (2):
* TLSv1.2 (IN), TLS header, Finished (20):
* TLSv1.2 (IN), TLS header, Supplemental data (23):
* TLSv1.3 (IN), TLS handshake, Encrypted Extensions (8):
* TLSv1.3 (IN), TLS handshake, Certificate (11):
* TLSv1.3 (IN), TLS handshake, CERT verify (15):
* TLSv1.3 (IN), TLS handshake, Finished (20):
* TLSv1.2 (OUT), TLS header, Finished (20):
* TLSv1.3 (OUT), TLS change cipher, Change cipher spec (1):
* TLSv1.2 (OUT), TLS header, Supplemental data (23):
* TLSv1.3 (OUT), TLS handshake, Finished (20):
* SSL connection using TLSv1.3 / TLS_AES_256_GCM_SHA384
* ALPN, server accepted to use h2
* Server certificate:
*  subject: CN=misc.google.com
*  start date: Jun 13 15:35:12 2024 GMT
*  expire date: Sep  5 15:35:11 2024 GMT
*  subjectAltName: host "europe-west1-qwiklabs-gcp-02-c45fb2422cb2.cloudfunctions.net" matched cert's "*.cloudfunctions.net"
*  issuer: C=US; O=Google Trust Services; CN=WR2
*  SSL certificate verify ok.
* Using HTTP2, server supports multiplexing
* Connection state changed (HTTP/2 confirmed)
* Copying HTTP/2 data in stream buffer to connection buffer after upgrade: len=0
* TLSv1.2 (OUT), TLS header, Supplemental data (23):
* TLSv1.2 (OUT), TLS header, Supplemental data (23):
* TLSv1.2 (OUT), TLS header, Supplemental data (23):
* Using Stream ID: 1 (easy handle 0x5ba7d1a24eb0)
* TLSv1.2 (OUT), TLS header, Supplemental data (23):
> GET /helloGET HTTP/2
> Host: europe-west1-qwiklabs-gcp-02-c45fb2422cb2.cloudfunctions.net
> user-agent: curl/7.81.0
> accept: */*
> 
* TLSv1.2 (IN), TLS header, Supplemental data (23):
* TLSv1.2 (OUT), TLS header, Supplemental data (23):
* TLSv1.2 (IN), TLS header, Supplemental data (23):
* TLSv1.2 (IN), TLS header, Supplemental data (23):
< HTTP/2 200 
< content-type: text/html; charset=utf-8
< function-execution-id: f9w1xts7agka
< x-cloud-trace-context: 8c1687d03d08e0142832687516d44f02;o=1
< date: Thu, 27 Jun 2024 11:37:13 GMT
< server: Google Frontend
< content-length: 12
< 
* TLSv1.2 (IN), TLS header, Supplemental data (23):
* TLSv1.2 (IN), TLS header, Supplemental data (23):
* Connection #0 to host europe-west1-qwiklabs-gcp-02-c45fb2422cb2.cloudfunctions.net left intact
Hello World!student_01_932d053b64d1@cloudshell:~/nodejs-docs-samples/functions/helloworld/helloworldGet (qwiklabs-gcp-02-c45fb2422cb2)$ 
```

### Create the API definition

API Gateway uses an API definition to route calls to the backend  service. You can use an OpenAPI spec that contains specialized  annotations to define the desired API Gateway behavior. The OpenAPI spec for this quickstart contains routing instructions to the Cloud Function backend.
```yaml
student_01_932d053b64d1@cloudshell:~/nodejs-docs-samples/functions/helloworld/helloworldGet (qwiklabs-gcp-02-c45fb2422cb2)$ cd ~
student_01_932d053b64d1@cloudshell:~ (qwiklabs-gcp-02-c45fb2422cb2)$ touch openapi2-functions.yaml
student_01_932d053b64d1@cloudshell:~ (qwiklabs-gcp-02-c45fb2422cb2)$ vi openapi2-functions.yaml 
student_01_932d053b64d1@cloudshell:~ (qwiklabs-gcp-02-c45fb2422cb2)$ cat openapi2-functions.yaml 
# openapi2-functions.yaml
swagger: '2.0'
info:
  title: API_ID description
  description: Sample API on API Gateway with a Google Cloud Functions backend
  version: 1.0.0
schemes:
  - https
produces:
  - application/json
paths:
  /hello:
    get:
      summary: Greet a user
      operationId: hello
      x-google-backend:
        address: https://europe-west1-qwiklabs-gcp-02-c45fb2422cb2.cloudfunctions.net/helloGET
      responses:
       '200':
          description: A successful response
          schema:
            type: string

student_01_932d053b64d1@cloudshell:~ (qwiklabs-gcp-02-c45fb2422cb2)$ 
```

```sh
student_01_932d053b64d1@cloudshell:~ (qwiklabs-gcp-02-c45fb2422cb2)$ export API_ID="hello-world-$(cat /dev/urandom | tr -dc 'a-z' | fold -w ${1:-8} | head -n 1)"
student_01_932d053b64d1@cloudshell:~ (qwiklabs-gcp-02-c45fb2422cb2)$ sed -i "s/API_ID/${API_ID}/g" openapi2-functions.yaml
sed -i "s/PROJECT_ID/$PROJECT_ID/g" openapi2-functions.yaml
student_01_932d053b64d1@cloudshell:~ (qwiklabs-gcp-02-c45fb2422cb2)$ 
```



## Creating a gateway

Now you are ready to create and deploy a gateway on API Gateway.

1. In the top search bar enter **API Gateway** and select it from the options that appear.
2. Click **Create Gateway**. Then, in the **APIs** section:

- Ensure the **Select an API** input is set to **Create new API**.
- For **Display Name** enter `Hello World API`
- For **API ID**, run the following command to once again obtain the API ID and enter it into the **API ID** field:

```sh
student_01_932d053b64d1@cloudshell:~ (qwiklabs-gcp-02-c45fb2422cb2)$ export API_ID="hello-world-$(cat /dev/urandom | tr -dc 'a-z' | fold -w ${1:-8} | head -n 1)"
echo $API_ID
hello-world-vwhmpmln
student_01_932d053b64d1@cloudshell:~ (qwiklabs-gcp-02-c45fb2422cb2)$ 
```



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

**It will take several minutes (~10 minutes) for the Create Gateway operation to complete.** To check the status of the creation and deployment process, you can  click the Notification icon in the main navigation bar to display a  status notification, as shown in the image below. Please ensure that the icon status has a green check next to it before proceeding.

### Testing your API Deployment

Now you can send requests to your API using the URL generated upon deployment of your gateway.

1. In Cloud Shell, enter the following command to retrieve the `GATEWAY_URL` of the newly created API hosted by API Gateway:

```sh
student_01_932d053b64d1@cloudshell:~ (qwiklabs-gcp-02-c45fb2422cb2)$ export GATEWAY_URL=$(gcloud api-gateway gateways describe hello-gateway --location europe-west1 --format json | jq -r .defaultHostname)
student_01_932d053b64d1@cloudshell:~ (qwiklabs-gcp-02-c45fb2422cb2)$ echo $GATEWAY_URL
hello-gateway-32ywhtdg.ew.gateway.dev
student_01_932d053b64d1@cloudshell:~ (qwiklabs-gcp-02-c45fb2422cb2)$ 
```

If it is not, that means you will need to **wait longer** for the API Gateway to be deployed.

1. Run the following curl command and validate that the response returned is `Hello World!`

   ```
   student_01_932d053b64d1@cloudshell:~ (qwiklabs-gcp-02-c45fb2422cb2)$ curl -s -w "\n" https://$GATEWAY_URL/hello
   Hello World!
   student_01_932d053b64d1@cloudshell:~ (qwiklabs-gcp-02-c45fb2422cb2)$ 
   ```

   

## Securing access by using an API key

To secure access to your API backend, you can generate an API key  associated with your project and grant that key access to call your API. To create an API Key you must do the following:

1. In the Cloud Console, navigate to **APIs & Services** > **Credentials**.
2. Select **Create credentials**, then select **API Key** from the dropdown menu. The **API key created** dialog box displays your newly created key.

```sh
AIzaSyC62hnEn0DwEsRv7qG6EHVqORvC8eLSBMU
```

1. Copy the API Key from the dialog, then click on **close**.
2. Store the API Key value in Cloud Shell by running the following command:

Now, enable the API Key support for your service.

1. In Cloud Shell, obtain the name of the `Managed Service` you just created using the following command:

2. Then, using the `Managed Service` name of the API you just created, run this command to **enable** API key support for the service:

   ```sh
   student_01_932d053b64d1@cloudshell:~ (qwiklabs-gcp-02-c45fb2422cb2)$ export API_KEY=AIzaSyC62hnEn0DwEsRv7qG6EHVqORvC8eLSBMU
   student_01_932d053b64d1@cloudshell:~ (qwiklabs-gcp-02-c45fb2422cb2)$ MANAGED_SERVICE=$(gcloud api-gateway apis list --format json | jq -r .[0].managedService | cut -d'/' -f6)
   echo $MANAGED_SERVICE
   hello-world-vwhmpmln-26xsj53t5wl5p.apigateway.qwiklabs-gcp-02-c45fb2422cb2.cloud.goog
   student_01_932d053b64d1@cloudshell:~ (qwiklabs-gcp-02-c45fb2422cb2)$ gcloud services enable $MANAGED_SERVICE
   Operation "operations/acat.p2-241556485876-b28c47e6-4aa6-419c-be66-37a981a570ae" finished successfully.
   student_01_932d053b64d1@cloudshell:~ (qwiklabs-gcp-02-c45fb2422cb2)$ 
   ```

   ### Modify the OpenAPI Spec to leverage API Key Security

   In this section, modify the API config of the deployed API to enforce an API key validation security policy on all traffic.

   1. Add the `security` type and `securityDefinitions` sections to a new file called `openapi2-functions2.yaml` file as shown below:

   ```yaml
   student_01_932d053b64d1@cloudshell:~ (qwiklabs-gcp-02-c45fb2422cb2)$ touch openapi2-functions2.yaml
   student_01_932d053b64d1@cloudshell:~ (qwiklabs-gcp-02-c45fb2422cb2)$ vi openapi2-functions2.yaml 
   student_01_932d053b64d1@cloudshell:~ (qwiklabs-gcp-02-c45fb2422cb2)$ cat openapi2-functions2.yaml 
   # openapi2-functions.yaml
   swagger: '2.0'
   info:
     title: API_ID description
     description: Sample API on API Gateway with a Google Cloud Functions backend
     version: 1.0.0
   schemes:
     - https
   produces:
     - application/json
   paths:
     /hello:
       get:
         summary: Greet a user
         operationId: hello
         x-google-backend:
           address: https://europe-west1-qwiklabs-gcp-02-c45fb2422cb2.cloudfunctions.net/helloGET
         security:
           - api_key: []
         responses:
          '200':
             description: A successful response
             schema:
               type: string
   securityDefinitions:
     api_key:
       type: "apiKey"
       name: "key"
       in: "query"
   student_01_932d053b64d1@cloudshell:~ (qwiklabs-gcp-02-c45fb2422cb2)$ 
   ```

   Run the following commands to replace the variables set in the last step in the OpenAPI spec file:

   ```sh
   student_01_932d053b64d1@cloudshell:~ (qwiklabs-gcp-02-c45fb2422cb2)$ sed -i "s/API_ID/${API_ID}/g" openapi2-functions2.yaml
   sed -i "s/PROJECT_ID/$PROJECT_ID/g" openapi2-functions2.yaml
   student_01_932d053b64d1@cloudshell:~ (qwiklabs-gcp-02-c45fb2422cb2)$ 
   ```

   ```yaml
   student_01_932d053b64d1@cloudshell:~ (qwiklabs-gcp-02-c45fb2422cb2)$ cat openapi2-functions2.yaml 
   # openapi2-functions.yaml
   swagger: '2.0'
   info:
     title: hello-world-vwhmpmln description
     description: Sample API on API Gateway with a Google Cloud Functions backend
     version: 1.0.0
   schemes:
     - https
   produces:
     - application/json
   paths:
     /hello:
       get:
         summary: Greet a user
         operationId: hello
         x-google-backend:
           address: https://europe-west1-qwiklabs-gcp-02-c45fb2422cb2.cloudfunctions.net/helloGET
         security:
           - api_key: []
         responses:
          '200':
             description: A successful response
             schema:
               type: string
   securityDefinitions:
     api_key:
       type: "apiKey"
       name: "key"
       in: "query"
   student_01_932d053b64d1@cloudshell:~ (qwiklabs-gcp-02-c45fb2422cb2)$ 
   ```

   Download the updated API spec file, you will use it to update the Gateway config in the next step:

   Click **Download**.

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

**Note:** **It may take a few minutes for the Update Gateway operation to complete.**

## Testing calls using your API key

1. To test using your API key run the following command:

```sh
student_01_932d053b64d1@cloudshell:~ (qwiklabs-gcp-02-c45fb2422cb2)$ export GATEWAY_URL=$(gcloud api-gateway gateways describe hello-gateway --location europe-west1 --format json | jq -r .defaultHostname)
curl -sL $GATEWAY_URL/hello
{"message":"UNAUTHENTICATED: Method doesn't allow unregistered callers (callers without established identity). Please use API Key or other form of API consumer identity to call this API.","code":401}
student_01_932d053b64d1@cloudshell:~ (qwiklabs-gcp-02-c45fb2422cb2)$ 
```

You should see a response similar to the following error as an API key was not supplied with the `curl` call: `UNAUTHENTICATED:Method doesn't allow unregistered callers (callers without established identity). Please use API Key or other form of API  consumer identity to call this API.`

1. Run the following curl command with the `key` query parameter and use the API key previously created to call the API:

```sh
student_01_932d053b64d1@cloudshell:~ (qwiklabs-gcp-02-c45fb2422cb2)$ curl -sL -w "\n" $GATEWAY_URL/hello?key=$API_KEY
Hello World!
student_01_932d053b64d1@cloudshell:~ (qwiklabs-gcp-02-c45fb2422cb2)$ 
```

If you do not have the `API_KEY` environment variable set you can get your API key from the left menu by navigating **APIs & Services** > **Credentials**. The key will be available under the **API Keys** section.

The response returned from the API should now be `Hello World!`.

You have successfully protected an API backend with API Gateway. Now you can start onboarding new API clients by generating additional API keys.

## History

```sh
student_01_932d053b64d1@cloudshell:~ (qwiklabs-gcp-02-c45fb2422cb2)$ history 
    1  gcloud config set compute/region europe-west1
    2  git clone https://github.com/GoogleCloudPlatform/nodejs-docs-samples.git
    3  cd nodejs-docs-samples/functions/helloworld/helloworldGet
    4  ls
    5  cat index.js 
    6  gcloud functions deploy helloGET --runtime nodejs14 --trigger-http --allow-unauthenticated --region europe-west1
    7  gcloud functions deploy helloGET --runtime nodejs14 --trigger-http --allow-unauthenticated --region europe-west1
    8  gcloud functions describe helloGET --region europe-west1
    9  export PROJECT_ID=qwiklabs-gcp-02-c45fb2422cb2
   10  curl -v https://europe-west1-qwiklabs-gcp-02-c45fb2422cb2.cloudfunctions.net/helloGET
   11  cd ~
   12  touch openapi2-functions.yaml
   13  vi openapi2-functions.yaml 
   14  cat openapi2-functions.yaml 
   15  export API_ID="hello-world-$(cat /dev/urandom | tr -dc 'a-z' | fold -w ${1:-8} | head -n 1)"
   16  sed -i "s/API_ID/${API_ID}/g" openapi2-functions.yaml
   17  sed -i "s/PROJECT_ID/$PROJECT_ID/g" openapi2-functions.yaml
   18  export API_ID="hello-world-$(cat /dev/urandom | tr -dc 'a-z' | fold -w ${1:-8} | head -n 1)"
   19  echo $API_ID
   20  cloudshell download $HOME/openapi2-functions.yaml
   21  export GATEWAY_URL=$(gcloud api-gateway gateways describe hello-gateway --location europe-west1 --format json | jq -r .defaultHostname)
   22  echo $GATEWAY_URL
   23  curl -s -w "\n" https://$GATEWAY_URL/hello
   24  export API_KEY=AIzaSyC62hnEn0DwEsRv7qG6EHVqORvC8eLSBMU
   25  MANAGED_SERVICE=$(gcloud api-gateway apis list --format json | jq -r .[0].managedService | cut -d'/' -f6)
   26  echo $MANAGED_SERVICE
   27  gcloud services enable $MANAGED_SERVICE
   28  touch openapi2-functions2.yaml
   29  vi openapi2-functions2.yaml 
   30  cat openapi2-functions2.yaml 
   31  sed -i "s/API_ID/${API_ID}/g" openapi2-functions2.yaml
   32  sed -i "s/PROJECT_ID/$PROJECT_ID/g" openapi2-functions2.yaml
   33  cat openapi2-functions2.yaml 
   34  cloudshell download $HOME/openapi2-functions2.yaml
   35  export GATEWAY_URL=$(gcloud api-gateway gateways describe hello-gateway --location europe-west1 --format json | jq -r .defaultHostname)
   36  curl -sL $GATEWAY_URL/hello
   37  curl -sL -w "\n" $GATEWAY_URL/hello?key=$API_KEY
   38  history 
student_01_932d053b64d1@cloudshell:~ (qwiklabs-gcp-02-c45fb2422cb2)$ 
```

