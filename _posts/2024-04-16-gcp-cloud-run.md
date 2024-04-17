---
layout: single
title:  "GCP Cloud Run Demo"
categories: Cloud
tags: GCP
classes: wide
show_date: true
header:
  teaser: /assets/images/pca-gcp.png
author:
  name     : "Professional Cloud Architect"
  avatar   : "/assets/images/pca-gcp.png"

sidebar:
  - title: "Topics"
    nav: my-sidebar

---
## GCP Cloud Run Demo 

Build a PDF converter web app on Cloud Run, which is a serverless  service, that automatically converts files stored in Google Drive into  PDFs stored in segregated Google Drive folders.

- Convert a Go application to a container
- Learn how to build containers with Google Cloud Build
- Create a Cloud Run service that converts files to PDF files in the cloud.
- Understand how to create Service Accounts and add permissions
- Use event processing with Cloud Storage

## Get the source code

```sh
Welcome to Cloud Shell! Type "help" to get started.
Your Cloud Platform project in this session is set to qwiklabs-gcp-04-c62450155f24.
Use “gcloud config set project [PROJECT_ID]” to change to a different project.
student_02_853e6baf1823@cloudshell:~ (qwiklabs-gcp-04-c62450155f24)$ gcloud auth list --filter=status:ACTIVE --format="value(account)"
student-02-853e6baf1823@qwiklabs.net
student_02_853e6baf1823@cloudshell:~ (qwiklabs-gcp-04-c62450155f24)$ git clone https://github.com/Deleplace/pet-theory.git
Cloning into 'pet-theory'...
remote: Enumerating objects: 403, done.
remote: Counting objects: 100% (45/45), done.
remote: Compressing objects: 100% (11/11), done.
remote: Total 403 (delta 35), reused 34 (delta 34), pack-reused 358
Receiving objects: 100% (403/403), 1.24 MiB | 15.83 MiB/s, done.
Resolving deltas: 100% (225/225), done.
student_02_853e6baf1823@cloudshell:~ (qwiklabs-gcp-04-c62450155f24)$ cd pet-theory/lab03
student_02_853e6baf1823@cloudshell:~/pet-theory/lab03 (qwiklabs-gcp-04-c62450155f24)$ ls
Dockerfile  gcs.go  go.mod  go.sum  notification.go  README.md  server.go  solution
student_02_853e6baf1823@cloudshell:~/pet-theory/lab03 (qwiklabs-gcp-04-c62450155f24)$ 
```

## Creating an invoice microservice

In this section you will create a Go application to process requests. As outlined in the architecture diagram, you will integrate Cloud Storage as part of the solution.

```go
student_02_853e6baf1823@cloudshell:~/pet-theory/lab03 (qwiklabs-gcp-04-c62450155f24)$ cat server.go 
package main

import (
      "fmt"
      "io/ioutil"
      "log"
      "net/http"
      "os"
      "os/exec"
      "regexp"
      "strings"
)

func main() {
      http.HandleFunc("/", process)

      port := os.Getenv("PORT")
      if port == "" {
              port = "8080"
              log.Printf("Defaulting to port %s", port)
      }

      log.Printf("Listening on port %s", port)
      err := http.ListenAndServe(fmt.Sprintf(":%s", port), nil)
      log.Fatal(err)
}

func process(w http.ResponseWriter, r *http.Request) {
      log.Println("Serving request")

      if r.Method == "GET" {
              fmt.Fprintln(w, "Ready to process POST requests from Cloud Storage trigger")
              return
      }

      //
      // Read request body containing Cloud Storage object metadata
      //
      gcsInputFile, err1 := readBody(r)
      if err1 != nil {
              log.Printf("Error reading POST data: %v", err1)
              w.WriteHeader(http.StatusBadRequest)
              fmt.Fprintf(w, "Problem with POST data: %v \n", err1)
              return
      }

      //
      // Working directory (concurrency-safe)
      //
      localDir, errDir := ioutil.TempDir("", "")
      if errDir != nil {
              log.Printf("Error creating local temp dir: %v", errDir)
              w.WriteHeader(http.StatusInternalServerError)
              fmt.Fprintf(w, "Could not create a temp directory on server. \n")
              return
      }
      defer os.RemoveAll(localDir)

      //
      // Download input file from Cloud Storage
      //
      localInputFile, err2 := download(gcsInputFile, localDir)
      if err2 != nil {
              log.Printf("Error downloading Cloud Storage file [%s] from bucket [%s]: %v",
gcsInputFile.Name, gcsInputFile.Bucket, err2)
              w.WriteHeader(http.StatusInternalServerError)
              fmt.Fprintf(w, "Error downloading Cloud Storage file [%s] from bucket [%s]",
gcsInputFile.Name, gcsInputFile.Bucket)
              return
      }

      //
      // Use LibreOffice to convert local input file to local PDF file.
      //
      localPDFFilePath, err3 := convertToPDF(localInputFile.Name(), localDir)
      if err3 != nil {
              log.Printf("Error converting to PDF: %v", err3)
              w.WriteHeader(http.StatusInternalServerError)
              fmt.Fprintf(w, "Error converting to PDF.")
              return
      }

      //
      // Upload the freshly generated PDF to Cloud Storage
      //
      targetBucket := os.Getenv("PDF_BUCKET")
      err4 := upload(localPDFFilePath, targetBucket)
      if err4 != nil {
              log.Printf("Error uploading PDF file to bucket [%s]: %v", targetBucket, err4)
              w.WriteHeader(http.StatusInternalServerError)
              fmt.Fprintf(w, "Error downloading Cloud Storage file [%s] from bucket [%s]",
gcsInputFile.Name, gcsInputFile.Bucket)
              return
      }

      //
      // Delete the original input file from Cloud Storage.
      //
      err5 := deleteGCSFile(gcsInputFile.Bucket, gcsInputFile.Name)
      if err5 != nil {
              log.Printf("Error deleting file [%s] from bucket [%s]: %v", gcsInputFile.Name,
gcsInputFile.Bucket, err5)
         // This is not a blocking error.
         // The PDF was successfully generated and uploaded.
      }

      log.Println("Successfully produced PDF")
      fmt.Fprintln(w, "Successfully produced PDF")
}

func convertToPDF(localFilePath string, localDir string) (resultFilePath string, err error) {
      log.Printf("Converting [%s] to PDF", localFilePath)
      cmd := exec.Command("libreoffice", "--headless", "--convert-to", "pdf",
              "--outdir", localDir,
              localFilePath)
      cmd.Stdout, cmd.Stderr = os.Stdout, os.Stderr
      log.Println(cmd)
      err = cmd.Run()
      if err != nil {
              return "", err
      }

      pdfFilePath := regexp.MustCompile(`\.\w+$`).ReplaceAllString(localFilePath, ".pdf")
      if !strings.HasSuffix(pdfFilePath, ".pdf") {
              pdfFilePath += ".pdf"
      }
      log.Printf("Converted %s to %s", localFilePath, pdfFilePath)
      return pdfFilePath, nil
}student_02_853e6baf1823@cloudshell:~/pet-theory/lab03 (qwiklabs-gcp-04-c62450155f24)$ 
```



```go
student_02_853e6baf1823@cloudshell:~/pet-theory/lab03 (qwiklabs-gcp-04-c62450155f24)$ go build -o server
student_02_853e6baf1823@cloudshell:~/pet-theory/lab03 (qwiklabs-gcp-04-c62450155f24)$ 
```



The functions called by this top-level code are in source files:

- server.go
- notification.go
- gcs.go

With the application has been successfully built, you can create the pdf-conversion service.

```go
student_02_853e6baf1823@cloudshell:~/pet-theory/lab03 (qwiklabs-gcp-04-c62450155f24)$ cat gcs.go 
package main

import (
        "context"
        "io"
        "log"
        "os"
        "path/filepath"

        "cloud.google.com/go/storage"
)

func download(gcsFile GCSEvent, localDir string) (*os.File, error) {
        log.Printf("Downloading input file from GCS: bucket [%s], object [%s]", gcsFile.Bucket, gcsFile.Name)
        localPath := filepath.Join(localDir, gcsFile.Name)
        tmpInputFile, err1 := os.OpenFile(localPath, os.O_CREATE|os.O_WRONLY, 0777)
        if err1 != nil {
                return nil, err1
        }
        defer tmpInputFile.Close()
        ctx := context.Background()
        reader, err2 := storageClient.Bucket(gcsFile.Bucket).Object(gcsFile.Name).NewReader(ctx)
        if err2 != nil {
                return nil, err2
        }
        defer reader.Close()
        n, err3 := io.Copy(tmpInputFile, reader)
        if err3 != nil {
                return nil, err3
        }
        log.Printf("Downloaded %d bytes from gs://%s/%s to %s", n, gcsFile.Bucket, gcsFile.Name, localPath)
        return tmpInputFile, nil
}

func upload(localFilePath string, bucket string) error {
        objectName := filepath.Base(localFilePath)
        log.Printf("Uploading PDF result [%s] to GCS bucket [%s]", objectName, bucket)
        localFile, err1 := os.Open(localFilePath)
        if err1 != nil {
                return err1
        }
        ctx := context.Background()
        writer := storageClient.Bucket(bucket).Object(objectName).NewWriter(ctx)
        n, err2 := io.Copy(writer, localFile)
        if err2 != nil {
                return err2
        }
        err3 := writer.Close()
        if err3 != nil {
                return err3
        }
        log.Printf("Uploaded %d bytes from %s to gs://%s/%s", n, localFilePath, bucket, objectName)

        return nil
}

func deleteGCSFile(bucket, name string) error {
        log.Println("Deleting input file from GCS")
        ctx := context.Background()
        return storageClient.Bucket(bucket).Object(name).Delete(ctx)
}

var storageClient *storage.Client

func init() {
        ctx := context.Background()
        var err error
        storageClient, err = storage.NewClient(ctx)
        if err != nil {
                log.Fatal(err)
        }

        if os.Getenv("PDF_BUCKET") == "" {
                log.Fatal("Need env var PDF_BUCKET: target GCS bucket where generated PDF will be written")
        }
}
student_02_853e6baf1823@cloudshell:~/pet-theory/lab03 (qwiklabs-gcp-04-c62450155f24)$ 
```

```go
student_02_853e6baf1823@cloudshell:~/pet-theory/lab03 (qwiklabs-gcp-04-c62450155f24)$ cat notification.go 
package main

import (
        "bytes"
        "encoding/base64"
        "encoding/json"
        "fmt"
        "io/ioutil"
        "log"
        "net/http"
        "time"
)

// GCSEvent has only a few fields from the event, that we're
// interested in.
type GCSEvent struct {
        Bucket string `json:"bucket"`
        Name   string `json:"name"`
        // PubSubGCSNotification.Message.Data contains other fields,
        // that we ignore.
}

// PubSubGCSNotification matches the JSON payload of the notification
// provided by GCS via PubSub, for a "new file created" event.
type PubSubGCSNotification struct {
        Message struct {
                Attributes  map[string]interface{} `json:"attributes"`
                MessageID   string                 `json:"messageId"`
                PublishTime time.Time              `json:"publishTime"`
                // Data is a base64 encoded GCSEvent
                Data string `json:"data"`
        } `json:"message"`
        Subscription string `json:"subscription"`
}

func (notif PubSubGCSNotification) decodeGCSEvent() (GCSEvent, error) {
        decoded, err1 := base64.StdEncoding.DecodeString(notif.Message.Data)
        if err1 != nil {
                return GCSEvent{}, fmt.Errorf("decoding notification message base64 encoded data %q: %v", notif.Message.Data, err1)
        }
        var fileEvent GCSEvent
        err2 := json.Unmarshal(decoded, &fileEvent)
        if err2 != nil {
                return GCSEvent{}, fmt.Errorf("unmarshalling GCSEvent data: %v. Could not parse %q", err2, string(decoded))
        }
        return fileEvent, nil
}

func readBody(r *http.Request) (GCSEvent, error) {
        log.Println("Reading POST data")
        body, err1 := ioutil.ReadAll(r.Body)
        if err1 != nil {
                return GCSEvent{}, fmt.Errorf("Error reading POST data: %v", err1)
        }
        log.Println("POST data =", string(body))
        var notification PubSubGCSNotification
        var file GCSEvent
        r.Body.Close()
        if len(bytes.TrimSpace(body)) == 0 {
                return GCSEvent{}, fmt.Errorf("Empty request body. Expecting a PubSub notification of a GCS event.")
        }
        err2 := json.Unmarshal(body, &notification)
        if err2 != nil {
                return GCSEvent{}, fmt.Errorf("Error unmarshalling POST data: %v. Could not parse %q", err2, string(body))
        }
        file, err3 := notification.decodeGCSEvent()
        if err3 != nil {
                return GCSEvent{}, fmt.Errorf("Error extracting notification encoded data: %v", err3)
        }
        return file, nil
}
student_02_853e6baf1823@cloudshell:~/pet-theory/lab03 (qwiklabs-gcp-04-c62450155f24)$ 
```

## Create a pdf-conversion service

The PDF service will use Cloud Run and Cloud Storage to initiate a process each time a file is uploaded to the designated storage.

To achieve this you will use a common pattern of event notifications together with Cloud Pub/Sub. Doing this enables the application to concentrate only on processing information. Transporting and passing information is performed by other services, which allows you to keep the application simple.

```dockerfile
student_02_853e6baf1823@cloudshell:~/pet-theory/lab03 (qwiklabs-gcp-04-c62450155f24)$ cat Dockerfile 
FROM debian:buster
RUN apt-get update -y \
  && apt-get install -y libreoffice \
  && apt-get clean
WORKDIR /usr/src/app
COPY server .
CMD [ "./server" ]student_02_853e6baf1823@cloudshell:~/pet-theory/lab03 (qwiklabs-gcp-04-c62450155f24)$ 
```

Initiate a rebuild of the `pdf-converter` image using Cloud Build:

```sh
student_02_853e6baf1823@cloudshell:~/pet-theory/lab03 (qwiklabs-gcp-04-c62450155f24)$ gcloud builds submit   --tag gcr.io/$GOOGLE_CLOUD_PROJECT/pdf-converter

unning hooks in /etc/ca-certificates/update.d...
{snip}
done.
done.
Processing triggers for libgdk-pixbuf2.0-0:amd64 (2.38.1+dfsg-1) ...
Removing intermediate container 49483171f84f
 1f5ae80c3a1d
Step 3/5 : WORKDIR /usr/src/app
 Running in 572d429bb535
Removing intermediate container 572d429bb535
 1c46a2f3f2e7
Step 4/5 : COPY server .
 a722727ee0ca
Step 5/5 : CMD [ "./server" ]
 Running in f3460def47cf
Removing intermediate container f3460def47cf
 6d5688ef47f8
Successfully built 6d5688ef47f8
Successfully tagged gcr.io/qwiklabs-gcp-04-c62450155f24/pdf-converter:latest
PUSH
Pushing gcr.io/qwiklabs-gcp-04-c62450155f24/pdf-converter
The push refers to repository [gcr.io/qwiklabs-gcp-04-c62450155f24/pdf-converter]
1811cf7c8b0b: Preparing
e233d66e67d6: Preparing
920c87495d59: Preparing
f74c00a8e5c3: Preparing
e233d66e67d6: Pushed
1811cf7c8b0b: Pushed
f74c00a8e5c3: Pushed
920c87495d59: Pushed
latest: digest: sha256:4b4b649e829c57ccbd3087dd43fcb4537c4c6568bb101cb0a5b21df74a261a32 size: 1160
DONE
------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------
ID: eb5ee966-b036-44ba-b5c9-7f12ba3a9245
CREATE_TIME: 2024-04-17T05:13:53+00:00
DURATION: 3M56S
SOURCE: gs://qwiklabs-gcp-04-c62450155f24_cloudbuild/source/1713330822.735678-ef537c3015ad432693f2ab42066aec43.tgz
IMAGES: gcr.io/qwiklabs-gcp-04-c62450155f24/pdf-converter (+1 more)
STATUS: SUCCESS
student_02_853e6baf1823@cloudshell:~/pet-theory/lab03 (qwiklabs-gcp-04-c62450155f24)$
```

1. Deploy the updated pdf-converter service. It's a good idea to give LibreOffice 2GB of RAM to work with,   see the line with the `--memory` option. Run these commands to build the container and to deploy it:

   ```sh
   student_02_853e6baf1823@cloudshell:~/pet-theory/lab03 (qwiklabs-gcp-04-c62450155f24)$ gcloud run deploy pdf-converter \
     --image gcr.io/$GOOGLE_CLOUD_PROJECT/pdf-converter \
     --platform managed \
     --region us-east4 \
     --memory=2Gi \
     --no-allow-unauthenticated \
     --set-env-vars PDF_BUCKET=$GOOGLE_CLOUD_PROJECT-processed \
     --max-instances=3
   Deploying container to Cloud Run service [pdf-converter] in project [qwiklabs-gcp-04-c62450155f24] region [us-east4]
   OK Deploying new service... Done.                                                                                                                                                  
     OK Creating Revision...                                                                                                                                                          
     OK Routing traffic...                                                                                                                                                            
   Done.                                                                                                                                                                              
   Service [pdf-converter] revision [pdf-converter-00001-xp5] has been deployed and is serving 100 percent of traffic.
   Service URL: https://pdf-converter-on4bdltgoq-uk.a.run.app
   student_02_853e6baf1823@cloudshell:~/pet-theory/lab03 (qwiklabs-gcp-04-c62450155f24)$ 
   ```

   The Cloud Run service has now been successfully deployed. However we deployed an application that requires the correct permissions to access it.

## Create a Service Account

A [Service Account](https://cloud.google.com/iam/docs/understanding-service-accounts) is a special type of account with access to Google APIs.

In this lab uses a Service Account to access Cloud Run when a Cloud Storage event is processed. Cloud Storage supports a rich set of notifications that can be used to trigger events.

Next, update the code to notify the application when a file has been uploaded.

1. Click the **Navigation menu** > **Cloud Storage**, and verify that two buckets have been created. You should see:

- PROJECT_ID-processed
- PROJECT_ID-upload

Create a Pub/Sub notification to indicate a new file has been uploaded  to the docs bucket ("uploaded"). The notifications will be labeled with  the topic "new-doc".



```sh
student_02_853e6baf1823@cloudshell:~/pet-theory/lab03 (qwiklabs-gcp-04-c62450155f24)$ gsutil notification create -t new-doc -f json -e OBJECT_FINALIZE gs://$GOOGLE_CLOUD_PROJECT-upload
Created Cloud Pub/Sub topic projects/qwiklabs-gcp-04-c62450155f24/topics/new-doc
Created notification config projects/_/buckets/qwiklabs-gcp-04-c62450155f24-upload/notificationConfigs/2
student_02_853e6baf1823@cloudshell:~/pet-theory/lab03 (qwiklabs-gcp-04-c62450155f24)$ 
```

Create a new service account to trigger the Cloud Run services:

```sh
student_02_853e6baf1823@cloudshell:~/pet-theory/lab03 (qwiklabs-gcp-04-c62450155f24)$ gcloud iam service-accounts create pubsub-cloud-run-invoker --display-name "PubSub Cloud Run Invoker"
Created service account [pubsub-cloud-run-invoker].
student_02_853e6baf1823@cloudshell:~/pet-theory/lab03 (qwiklabs-gcp-04-c62450155f24)$ 
```

Give the service account permission to invoke the PDF converter service:

```sh
student_02_853e6baf1823@cloudshell:~/pet-theory/lab03 (qwiklabs-gcp-04-c62450155f24)$ gcloud run services add-iam-policy-binding pdf-converter \
  --member=serviceAccount:pubsub-cloud-run-invoker@$GOOGLE_CLOUD_PROJECT.iam.gserviceaccount.com \
  --role=roles/run.invoker \
  --region us-east4 \
  --platform managed
Updated IAM policy for service [pdf-converter].
bindings:
- members:
  - serviceAccount:pubsub-cloud-run-invoker@qwiklabs-gcp-04-c62450155f24.iam.gserviceaccount.com
  role: roles/run.invoker
etag: BwYWRB-I0eQ=
version: 1
student_02_853e6baf1823@cloudshell:~/pet-theory/lab03 (qwiklabs-gcp-04-c62450155f24)$ 
```

```sh
student_02_853e6baf1823@cloudshell:~/pet-theory/lab03 (qwiklabs-gcp-04-c62450155f24)$ PROJECT_NUMBER=$(gcloud projects list \
 --format="value(PROJECT_NUMBER)" \
 --filter="$GOOGLE_CLOUD_PROJECT")
student_02_853e6baf1823@cloudshell:~/pet-theory/lab03 (qwiklabs-gcp-04-c62450155f24)$ 
```

Enable your project to create Cloud Pub/Sub authentication tokens:

```sh
student_02_853e6baf1823@cloudshell:~/pet-theory/lab03 (qwiklabs-gcp-04-c62450155f24)$ gcloud projects add-iam-policy-binding $GOOGLE_CLOUD_PROJECT \
  --member=serviceAccount:service-$PROJECT_NUMBER@gcp-sa-pubsub.iam.gserviceaccount.com \
  --role=roles/iam.serviceAccountTokenCreator
Updated IAM policy for project [qwiklabs-gcp-04-c62450155f24].
bindings:
- members:
  - serviceAccount:qwiklabs-gcp-04-c62450155f24@qwiklabs-gcp-04-c62450155f24.iam.gserviceaccount.com
  role: roles/bigquery.admin
- members:
  - serviceAccount:997127409631@cloudbuild.gserviceaccount.com
  role: roles/cloudbuild.builds.builder
- members:
  - serviceAccount:service-997127409631@gcp-sa-cloudbuild.iam.gserviceaccount.com
  role: roles/cloudbuild.serviceAgent
- members:
  - serviceAccount:service-997127409631@compute-system.iam.gserviceaccount.com
  role: roles/compute.serviceAgent
- members:
  - serviceAccount:service-997127409631@container-engine-robot.iam.gserviceaccount.com
  role: roles/container.serviceAgent
- members:
  - serviceAccount:service-997127409631@containerregistry.iam.gserviceaccount.com
  role: roles/containerregistry.ServiceAgent
- members:
  - serviceAccount:997127409631-compute@developer.gserviceaccount.com
  - serviceAccount:997127409631@cloudservices.gserviceaccount.com
  role: roles/editor
- members:
  - user:student-02-853e6baf1823@qwiklabs.net
  role: roles/iam.serviceAccountAdmin
- members:
  - serviceAccount:service-997127409631@gcp-sa-pubsub.iam.gserviceaccount.com
  role: roles/iam.serviceAccountTokenCreator
- members:
  - user:student-02-853e6baf1823@qwiklabs.net
  role: roles/monitoring.admin
- members:
  - serviceAccount:admiral@qwiklabs-services-prod.iam.gserviceaccount.com
  - serviceAccount:qwiklabs-gcp-04-c62450155f24@qwiklabs-gcp-04-c62450155f24.iam.gserviceaccount.com
  - user:student-02-853e6baf1823@qwiklabs.net
  role: roles/owner
- members:
  - serviceAccount:service-997127409631@gcp-sa-pubsub.iam.gserviceaccount.com
  role: roles/pubsub.serviceAgent
- members:
  - serviceAccount:service-997127409631@serverless-robot-prod.iam.gserviceaccount.com
  role: roles/run.serviceAgent
- members:
  - serviceAccount:qwiklabs-gcp-04-c62450155f24@qwiklabs-gcp-04-c62450155f24.iam.gserviceaccount.com
  role: roles/storage.admin
- members:
  - user:student-02-853e6baf1823@qwiklabs.net
  role: roles/viewer
etag: BwYWRCN6P_A=
version: 1
student_02_853e6baf1823@cloudshell:~/pet-theory/lab03 (qwiklabs-gcp-04-c62450155f24)$ 
```

With the Service Account created it can be used to invoke the Cloud Run Service.

## Testing the Cloud Run service

Before progressing further, test the deployed service. Remember the service requires authentication, so test that to ensure it is actually private.

Save the URL of your service in the environment variable **$SERVICE_URL**:

```sh
student_02_853e6baf1823@cloudshell:~/pet-theory/lab03 (qwiklabs-gcp-04-c62450155f24)$ SERVICE_URL=$(gcloud run services describe pdf-converter \
  --platform managed \
  --region us-east4 \
  --format "value(status.url)")
student_02_853e6baf1823@cloudshell:~/pet-theory/lab03 (qwiklabs-gcp-04-c62450155f24)$ echo $SERVICE_URL
https://pdf-converter-on4bdltgoq-uk.a.run.app
student_02_853e6baf1823@cloudshell:~/pet-theory/lab03 (qwiklabs-gcp-04-c62450155f24)$ 
```

Make an anonymous GET request to your new service:

```sh
student_02_853e6baf1823@cloudshell:~/pet-theory/lab03 (qwiklabs-gcp-04-c62450155f24)$ curl -X GET $SERVICE_URL

<html><head>
<meta http-equiv="content-type" content="text/html;charset=utf-8">
<title>403 Forbidden</title>
</head>
<body text=#000000 bgcolor=#ffffff>
<h1>Error: Forbidden</h1>
<h2>Your client does not have permission to get URL <code>/</code> from this server.</h2>
<h2></h2>
</body></html>
student_02_853e6baf1823@cloudshell:~/pet-theory/lab03 (qwiklabs-gcp-04-c62450155f24)$ 
```

Now try invoking the service as an authorized user:

```sh
student_02_853e6baf1823@cloudshell:~/pet-theory/lab03 (qwiklabs-gcp-04-c62450155f24)$ curl -X GET -H "Authorization: Bearer $(gcloud auth print-identity-token)" $SERVICE_URL
Ready to process POST requests from Cloud Storage trigger
student_02_853e6baf1823@cloudshell:~/pet-theory/lab03 (qwiklabs-gcp-04-c62450155f24)$ 
```

Great work, you have successfully deployed an authenticated Cloud Run service.



## Cloud Storage trigger

To initiate a notification when new content is uploaded to Cloud Storage, add a subscription to your existing Pub/Sub Topic.

Cloud Storage notifications will automatically push a message to your  Topic queue when new content is uploaded. Using notifications allows you to create powerful applications that respond to events without needing  to write additional code.

Create a Pub/Sub subscription so that the PDF converter will be run whenever a message is published to the topic `new-doc`:

```sh
student_02_853e6baf1823@cloudshell:~/pet-theory/lab03 (qwiklabs-gcp-04-c62450155f24)$ gcloud pubsub subscriptions create pdf-conv-sub \
  --topic new-doc \
  --push-endpoint=$SERVICE_URL \
  --push-auth-service-account=pubsub-cloud-run-invoker@$GOOGLE_CLOUD_PROJECT.iam.gserviceaccount.com
Created subscription [projects/qwiklabs-gcp-04-c62450155f24/subscriptions/pdf-conv-sub].
student_02_853e6baf1823@cloudshell:~/pet-theory/lab03 (qwiklabs-gcp-04-c62450155f24)$ 
```

Now whenever a file is uploaded the Pub/Sub subscription will interact with your Service Account. The Service Account will then initiate your PDF Converter Cloud Run service.



## Testing Cloud Storage notification

To test the Cloud Run service, use the example files available.

1. Copy the test files into your upload bucket:

```sh
student_02_853e6baf1823@cloudshell:~/pet-theory/lab03 (qwiklabs-gcp-04-c62450155f24)$ gsutil -m cp -r gs://spls/gsp762/* gs://$GOOGLE_CLOUD_PROJECT-upload
Copying gs://spls/gsp762/file_example_XLSX_50.xlsx [Content-Type=application/vnd.openxmlformats-officedocument.spreadsheetml.sheet]...
Copying gs://spls/gsp762/file-sample_100kB.doc [Content-Type=application/msword]...
Copying gs://spls/gsp762/cat-and-mouse.jpg [Content-Type=image/jpeg]...
Copying gs://spls/gsp762/file-sample_1MB.docx [Content-Type=application/vnd.openxmlformats-officedocument.wordprocessingml.document]...
Copying gs://spls/gsp762/file_example_XLS_100.xls [Content-Type=application/vnd.ms-excel]...
Copying gs://spls/gsp762/file-sample_500kB.docx [Content-Type=application/vnd.openxmlformats-officedocument.wordprocessingml.document]...
Copying gs://spls/gsp762/file_example_XLS_10.xls [Content-Type=application/vnd.ms-excel]...
Copying gs://spls/gsp762/file_example_XLS_50.xls [Content-Type=application/vnd.ms-excel]...
Copying gs://spls/gsp762//Copy of cat-and-mouse.jpg [Content-Type=image/jpeg]...
student_02_853e6baf1823@cloudshell:~/pet-theory/lab03 (qwiklabs-gcp-04-c62450155f24)$ 
```

In the Cloud Console, click **Cloud Storage > Buckets** followed by the bucket name whose name ends in "**-upload**" and click the **Refresh** button a few times and see how the files are deleted, one by one, as they are converted to PDFs.

Then click **Buckets**, followed by the bucket whose name ends in "**-processed**". It should contain PDF versions of all files. Feel free to open the PDF files to make sure they were properly converted.



1. Once the upload is done, click **Navigation menu > Cloud Run** and click on the **pdf-converter** service.
2. Select the **LOGS** tab and add a filter of "Converting" to see the converted files.
3. Navigate to **Navigation menu > Cloud Storage** and open the bucket name ending in "**-upload**" to confirm all files uploaded have been processed.

Excellent work, you have successfully built a new service to create a PDF using files uploaded to Cloud Storage.



In this lab, you've learned how to convert a Go application into a  container, learned how to construct containers utilizing Google Cloud  Build, and launched a Cloud Run service. You've also gained skills in  enabling permissions through a Service Account and leveraging Cloud  Storage event processing, all of which are integral to the operation of  the pdf-converter service that transforms documents into PDFs and stores them in the "processed" bucket.

```sh
student_02_853e6baf1823@cloudshell:~/pet-theory/lab03 (qwiklabs-gcp-04-c62450155f24)$ history 
    1  gcloud auth list --filter=status:ACTIVE --format="value(account)"
    2  git clone https://github.com/Deleplace/pet-theory.git
    3  cd pet-theory/lab03
    4  ls
    5  cat server.go 
    6  go build -o server
    7  ls
    8  cat gcs.go 
    9  ls
   10  cat notification.go 
   11  ls
   12  cat Dockerfile 
   13  gcloud builds submit   --tag gcr.io/$GOOGLE_CLOUD_PROJECT/pdf-converter
   14  history 
   15  gcloud run deploy pdf-converter   --image gcr.io/$GOOGLE_CLOUD_PROJECT/pdf-converter   --platform managed   --region us-east4   --memory=2Gi   --no-allow-unauthenticated   --set-env-vars PDF_BUCKET=$GOOGLE_CLOUD_PROJECT-processed   --max-instances=3
   16  gsutil notification create -t new-doc -f json -e OBJECT_FINALIZE gs://$GOOGLE_CLOUD_PROJECT-upload
   17  gcloud iam service-accounts create pubsub-cloud-run-invoker --display-name "PubSub Cloud Run Invoker"
   18  gcloud run services add-iam-policy-binding pdf-converter   --member=serviceAccount:pubsub-cloud-run-invoker@$GOOGLE_CLOUD_PROJECT.iam.gserviceaccount.com   --role=roles/run.invoker   --region us-east4   --platform managed
   19  PROJECT_NUMBER=$(gcloud projects list \
 --format="value(PROJECT_NUMBER)" \
 --filter="$GOOGLE_CLOUD_PROJECT")
   20  gcloud projects add-iam-policy-binding $GOOGLE_CLOUD_PROJECT   --member=serviceAccount:service-$PROJECT_NUMBER@gcp-sa-pubsub.iam.gserviceaccount.com   --role=roles/iam.serviceAccountTokenCreator
   21  SERVICE_URL=$(gcloud run services describe pdf-converter \
  --platform managed \
  --region us-east4 \
  --format "value(status.url)")
   22  echo $SERVICE_URL
   23  curl -X GET $SERVICE_URL
   24  curl -X GET -H "Authorization: Bearer $(gcloud auth print-identity-token)" $SERVICE_URL
   25  gcloud pubsub subscriptions create pdf-conv-sub   --topic new-doc   --push-endpoint=$SERVICE_URL   --push-auth-service-account=pubsub-cloud-run-invoker@$GOOGLE_CLOUD_PROJECT.iam.gserviceaccount.com
   26  gsutil -m cp -r gs://spls/gsp762/* gs://$GOOGLE_CLOUD_PROJECT-upload
   27  history 
student_02_853e6baf1823@cloudshell:~/pet-theory/lab03 (qwiklabs-gcp-04-c62450155f24)$ 
```

