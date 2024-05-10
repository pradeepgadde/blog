---

layout: single
title:  "Working with Artifact Registry"
categories: Cloud
tags: GCP
toc: true
toc_sticky: true
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

# Working with Artifact Registry

As the evolution of Container Registry, [Artifact Registry](https://cloud.google.com/artifact-registry) is a single place for your organization to manage container images and  language packages (such as Maven and npm). It is fully integrated with  Google Cloud's tooling and runtimes and comes with support for native  artifact protocols. This makes it simple to integrate it with your CI/CD tooling to set up automated pipelines.

- Create repositories for Containers and Language Packages
- Manage container images with Artifact Registry
- Integrate Artifact Registry with Cloud Code
- Configure Maven to use Artifact Registry for Java Dependencies

## Prepare the lab environment

### Set up variables

- In Cloud Shell, set your project ID and project number. Save them as `PROJECT_ID` and `PROJECT_NUMBER` variables:

```sh
Welcome to Cloud Shell! Type "help" to get started.
Your Cloud Platform project in this session is set to qwiklabs-gcp-00-1176b71068a0.
Use “gcloud config set project [PROJECT_ID]” to change to a different project.
student_01_0fb7ab211dcf@cloudshell:~ (qwiklabs-gcp-00-1176b71068a0)$ export PROJECT_ID=$(gcloud config get-value project)
export PROJECT_NUMBER=$(gcloud projects describe $PROJECT_ID --format='value(projectNumber)')
export REGION=us-east1
gcloud config set compute/region $REGION
Your active configuration is: [cloudshell-9112]
WARNING: Property validation for compute/region was skipped.
Updated property [compute/region].
student_01_0fb7ab211dcf@cloudshell:~ (qwiklabs-gcp-00-1176b71068a0)$ 

```

### Enable Google services

```sh
student_01_0fb7ab211dcf@cloudshell:~ (qwiklabs-gcp-00-1176b71068a0)$ gcloud services enable \
  cloudresourcemanager.googleapis.com \
  container.googleapis.com \
  artifactregistry.googleapis.com \
  containerregistry.googleapis.com \
  containerscanning.googleapis.com
Operation "operations/acat.p2-610761045646-51bcfca2-43a2-4c5d-8d1f-8898546f8371" finished successfully.
student_01_0fb7ab211dcf@cloudshell:~ (qwiklabs-gcp-00-1176b71068a0)$ 
```



### Get the source code

```sh
student_01_0fb7ab211dcf@cloudshell:~ (qwiklabs-gcp-00-1176b71068a0)$ git clone https://github.com/GoogleCloudPlatform/cloud-code-samples/
cd ~/cloud-code-samples
Cloning into 'cloud-code-samples'...
remote: Enumerating objects: 16993, done.
remote: Counting objects: 100% (85/85), done.
remote: Compressing objects: 100% (53/53), done.
remote: Total 16993 (delta 38), reused 74 (delta 32), pack-reused 16908
Receiving objects: 100% (16993/16993), 27.62 MiB | 13.60 MiB/s, done.
Resolving deltas: 100% (10661/10661), done.
student_01_0fb7ab211dcf@cloudshell:~/cloud-code-samples (qwiklabs-gcp-00-1176b71068a0)$ 
```



### Provision the infrastructure

```sh
student_01_0fb7ab211dcf@cloudshell:~/cloud-code-samples (qwiklabs-gcp-00-1176b71068a0)$ gcloud container clusters create container-dev-cluster --zone=us-east1-d
Default change: VPC-native is the default mode during cluster creation for versions greater than 1.21.0-gke.1500. To create advanced routes based clusters, please pass the `--no-enable-ip-alias` flag
Note: Your Pod address range (`--cluster-ipv4-cidr`) can accommodate at most 1008 node(s).
Creating cluster container-dev-cluster in us-east1-d... Cluster is being health-checked (master is healthy)...done.                                                                
Created [https://container.googleapis.com/v1/projects/qwiklabs-gcp-00-1176b71068a0/zones/us-east1-d/clusters/container-dev-cluster].
To inspect the contents of your cluster, go to: https://console.cloud.google.com/kubernetes/workload_/gcloud/us-east1-d/container-dev-cluster?project=qwiklabs-gcp-00-1176b71068a0
kubeconfig entry generated for container-dev-cluster.
NAME: container-dev-cluster
LOCATION: us-east1-d
MASTER_VERSION: 1.28.7-gke.1026000
MASTER_IP: 34.148.232.63
MACHINE_TYPE: e2-medium
NODE_VERSION: 1.28.7-gke.1026000
NUM_NODES: 3
STATUS: RUNNING
student_01_0fb7ab211dcf@cloudshell:~/cloud-code-samples (qwiklabs-gcp-00-1176b71068a0)$ 
```



## Working with container images

### Create a Docker Repository on Artifact registry

Artifact Registry supports managing container images and language  packages. Different artifact types require different specifications. For example, the requests for Maven dependencies are different from  requests for Node dependencies.

To support the different API specifications, Artifact Registry needs  to know what format you want the API responses to follow. To do this you will create a repository and pass in the `--repository-format` flag indicating the type of repository desired.

```sh
student_01_0fb7ab211dcf@cloudshell:~/cloud-code-samples (qwiklabs-gcp-00-1176b71068a0)$ gcloud artifacts repositories create container-dev-repo --repository-format=docker \
  --location=$REGION \
  --description="Docker repository for Container Dev Workshop"
Create request issued for: [container-dev-repo]
Waiting for operation [projects/qwiklabs-gcp-00-1176b71068a0/locations/us-east1/operations/977f0f2c-7897-4348-be27-02e3db83c9d1] to complete...done.                               
Created repository [container-dev-repo].
student_01_0fb7ab211dcf@cloudshell:~/cloud-code-samples (qwiklabs-gcp-00-1176b71068a0)$ 
```



### Configure Docker Authentication to Artifact Registry

When connecting to Artifact Registry credentials are required in  order to provide access. Rather than set up separate credentials, Docker can be configured to use your `gcloud` credentials seamlessly.

```sh
student_01_0fb7ab211dcf@cloudshell:~/cloud-code-samples (qwiklabs-gcp-00-1176b71068a0)$ gcloud auth configure-docker us-east1-docker.pkg.dev
WARNING: Your config file at [/home/student_01_0fb7ab211dcf/.docker/config.json] contains these credential helper entries:

{
  "credHelpers": {
    "africa-south1-docker.pkg.dev": "gcloud",
    "asia-docker.pkg.dev": "gcloud",
    "asia-east1-docker.pkg.dev": "gcloud",
    "asia-east2-docker.pkg.dev": "gcloud",
    "asia-northeast1-docker.pkg.dev": "gcloud",
    "asia-northeast2-docker.pkg.dev": "gcloud",
    "asia-northeast3-docker.pkg.dev": "gcloud",
    "asia-south1-docker.pkg.dev": "gcloud",
    "asia-south2-docker.pkg.dev": "gcloud",
    "asia-southeast1-docker.pkg.dev": "gcloud",
    "asia-southeast2-docker.pkg.dev": "gcloud",
    "australia-southeast1-docker.pkg.dev": "gcloud",
    "australia-southeast2-docker.pkg.dev": "gcloud",
    "europe-docker.pkg.dev": "gcloud",
    "europe-central2-docker.pkg.dev": "gcloud",
    "europe-north1-docker.pkg.dev": "gcloud",
    "europe-southwest1-docker.pkg.dev": "gcloud",
    "europe-west1-docker.pkg.dev": "gcloud",
    "europe-west10-docker.pkg.dev": "gcloud",
    "europe-west12-docker.pkg.dev": "gcloud",
    "europe-west2-docker.pkg.dev": "gcloud",
    "europe-west3-docker.pkg.dev": "gcloud",
    "europe-west4-docker.pkg.dev": "gcloud",
    "europe-west6-docker.pkg.dev": "gcloud",
    "europe-west8-docker.pkg.dev": "gcloud",
    "europe-west9-docker.pkg.dev": "gcloud",
    "me-central1-docker.pkg.dev": "gcloud",
    "me-central2-docker.pkg.dev": "gcloud",
    "me-west1-docker.pkg.dev": "gcloud",
    "northamerica-northeast1-docker.pkg.dev": "gcloud",
    "northamerica-northeast2-docker.pkg.dev": "gcloud",
    "southamerica-east1-docker.pkg.dev": "gcloud",
    "us-docker.pkg.dev": "gcloud",
    "us-central1-docker.pkg.dev": "gcloud",
    "us-central2-docker.pkg.dev": "gcloud",
    "us-east1-docker.pkg.dev": "gcloud",
    "us-east4-docker.pkg.dev": "gcloud",
    "us-east5-docker.pkg.dev": "gcloud",
    "us-east7-docker.pkg.dev": "gcloud",
    "us-south1-docker.pkg.dev": "gcloud",
    "us-west1-docker.pkg.dev": "gcloud",
    "us-west2-docker.pkg.dev": "gcloud",
    "us-west3-docker.pkg.dev": "gcloud",
    "us-west4-docker.pkg.dev": "gcloud"
  }
}
Adding credentials for: us-east1-docker.pkg.dev
gcloud credential helpers already registered correctly.
student_01_0fb7ab211dcf@cloudshell:~/cloud-code-samples (qwiklabs-gcp-00-1176b71068a0)$ 
```

### Explore the sample Application

A sample application is provided in the git repository you cloned.

- Change into the java directory and review the application code:

```sh
student_01_0fb7ab211dcf@cloudshell:~/cloud-code-samples (qwiklabs-gcp-00-1176b71068a0)$ cd ~/cloud-code-samples/java/java-hello-world
student_01_0fb7ab211dcf@cloudshell:~/cloud-code-samples/java/java-hello-world (qwiklabs-gcp-00-1176b71068a0)$ ls
checkstyle.xml  Dockerfile  img  kubernetes-manifests  pom.xml  README.md  skaffold.yaml  src
student_01_0fb7ab211dcf@cloudshell:~/cloud-code-samples/java/java-hello-world (qwiklabs-gcp-00-1176b71068a0)$ cat Dockerfile 
# Use maven to compile the java application.
FROM maven:3-jdk-11-slim AS build-env

# Set the working directory to /app
WORKDIR /app

# copy the pom.xml file to download dependencies
COPY pom.xml ./

# download dependencies as specified in pom.xml
# building dependency layer early will speed up compile time when pom is unchanged
RUN mvn verify --fail-never

# Copy the rest of the working directory contents into the container
COPY . ./

# Compile the application.
RUN mvn -Dmaven.test.skip=true package

# Build runtime image.
FROM openjdk:11.0.16-jre-slim

# Copy the compiled files over.
COPY --from=build-env /app/target/ /app/

# Starts java app with debugging server at port 5005.
CMD ["java", "-jar", "/app/hello-world-1.0.0.jar"]
student_01_0fb7ab211dcf@cloudshell:~/cloud-code-samples/java/java-hello-world (qwiklabs-gcp-00-1176b71068a0)$ 
```

The folder contains an example Java application that renders a simple  web page: in addition to various files not relevant for this specific  lab, it contains the source code, under the `src` folder, and a Dockerfile you will use to build a container image locally.

Build the Container Image

Before you can store container images in Artifact Registry you need to create one.

- Run the following command to build the container image and tag it properly:

```sh
student_01_0fb7ab211dcf@cloudshell:~/cloud-code-samples/java/java-hello-world (qwiklabs-gcp-00-1176b71068a0)$ docker build -t us-east1-docker.pkg.dev/qwiklabs-gcp-00-1176b71068a0/container-dev-repo/java-hello-world:tag1 .
[+] Building 49.5s (14/14) FINISHED                                                                                                                                  docker:default
 => [internal] load build definition from Dockerfile                                                                                                                           0.0s
 => => transferring dockerfile: 779B                                                                                                                                           0.0s
 => [internal] load metadata for docker.io/library/maven:3-jdk-11-slim                                                                                                         2.6s
 => [internal] load metadata for docker.io/library/openjdk:11.0.16-jre-slim                                                                                                    1.8s
 => [internal] load .dockerignore                                                                                                                                              0.0s
 => => transferring context: 76B                                                                                                                                               0.0s
 => [build-env 1/6] FROM docker.io/library/maven:3-jdk-11-slim@sha256:2cb7c73ba2fd0f7ae64cfabd99180030ec85841a1197b4ae821d21836cb0aa3b                                        10.3s
 => => resolve docker.io/library/maven:3-jdk-11-slim@sha256:2cb7c73ba2fd0f7ae64cfabd99180030ec85841a1197b4ae821d21836cb0aa3b                                                   0.0s
 => => sha256:a2f2f93da48276873890ac821b3c991d53a7e864791aaf82c39b7863c908b93b 1.58MB / 1.58MB                                                                                 0.2s
 => => sha256:12cca292b13cb58fadde25af113ddc4ac3b0c5e39ab3f1290a6ba62ec8237afd 212B / 212B                                                                                     0.2s
 => => sha256:2cb7c73ba2fd0f7ae64cfabd99180030ec85841a1197b4ae821d21836cb0aa3b 549B / 549B                                                                                     0.0s
 => => sha256:d12b61d6ea1938fd3f8b194f06276a2b70b4f1f5b2a87304a7c0c585db64b242 2.00kB / 2.00kB                                                                                 0.0s
 => => sha256:62643abbdb7b732a1267a2acabef7b08a57240914c7def0c4503f9bc6d95845f 8.49kB / 8.49kB                                                                                 0.0s
 => => sha256:1efc276f4ff952c055dea726cfc96ec6a4fdb8b62d9eed816bd2b788f2860ad7 31.37MB / 31.37MB                                                                               0.6s
 => => sha256:578079fbf7e898fa411cf4a42cefaae0619ad9ab9de4835933f6dd416a4f66b5 2.46MB / 2.46MB                                                                                 0.5s
 => => sha256:69e15dccd787ba2cfe67f6abf5970ed88a5e019efbb499e499da3ab20b85fcc7 202.34MB / 202.34MB                                                                             3.7s
 => => sha256:b0d7f601ed347b0973a73476491b4a59956272a478e6d7ade323b3187a2718f3 8.74MB / 8.74MB                                                                                 0.9s
 => => sha256:4ea345e90dc33448725856890f2f43f903bfd156fe2b75f8df2f35376e28e3a0 854B / 854B                                                                                     0.8s
 => => extracting sha256:1efc276f4ff952c055dea726cfc96ec6a4fdb8b62d9eed816bd2b788f2860ad7                                                                                      3.3s
 => => sha256:605b2bfb5ae9e78e2eab41e0246efb1bee52ac9f64f8331e22654553cc1f5dec 362B / 362B                                                                                     1.0s
 => => extracting sha256:a2f2f93da48276873890ac821b3c991d53a7e864791aaf82c39b7863c908b93b                                                                                      0.2s
 => => extracting sha256:12cca292b13cb58fadde25af113ddc4ac3b0c5e39ab3f1290a6ba62ec8237afd                                                                                      0.0s
 => => extracting sha256:69e15dccd787ba2cfe67f6abf5970ed88a5e019efbb499e499da3ab20b85fcc7                                                                                      5.4s
 => => extracting sha256:578079fbf7e898fa411cf4a42cefaae0619ad9ab9de4835933f6dd416a4f66b5                                                                                      0.1s
 => => extracting sha256:b0d7f601ed347b0973a73476491b4a59956272a478e6d7ade323b3187a2718f3                                                                                      0.2s
 => => extracting sha256:4ea345e90dc33448725856890f2f43f903bfd156fe2b75f8df2f35376e28e3a0                                                                                      0.0s
 => => extracting sha256:605b2bfb5ae9e78e2eab41e0246efb1bee52ac9f64f8331e22654553cc1f5dec                                                                                      0.0s
 => [stage-1 1/2] FROM docker.io/library/openjdk:11.0.16-jre-slim@sha256:93af7df2308c5141a751c4830e6b6c5717db102b3b31f012ea29d842dc4f2b02                                      7.4s
 => => resolve docker.io/library/openjdk:11.0.16-jre-slim@sha256:93af7df2308c5141a751c4830e6b6c5717db102b3b31f012ea29d842dc4f2b02                                              0.0s
 => => sha256:93af7df2308c5141a751c4830e6b6c5717db102b3b31f012ea29d842dc4f2b02 549B / 549B                                                                                     0.0s
 => => sha256:884c08d0f406a81ae1b5786932abaf399c335b997da7eea6a30cc51529220b66 1.16kB / 1.16kB                                                                                 0.0s
 => => sha256:764a04af3eff09cc6a29bcc19cf6315dbea455d7392c1a588a5deb331a929c29 7.55kB / 7.55kB                                                                                 0.0s
 => => sha256:1efc276f4ff952c055dea726cfc96ec6a4fdb8b62d9eed816bd2b788f2860ad7 31.37MB / 31.37MB                                                                               0.5s
 => => sha256:a2f2f93da48276873890ac821b3c991d53a7e864791aaf82c39b7863c908b93b 1.58MB / 1.58MB                                                                                 0.2s
 => => sha256:12cca292b13cb58fadde25af113ddc4ac3b0c5e39ab3f1290a6ba62ec8237afd 212B / 212B                                                                                     0.2s
 => => sha256:d73cf48caaac2e45ad76a2a9eb3b311d0e4eb1d804e3d2b9cf075a1fa31e6f92 46.04MB / 46.04MB                                                                               1.8s
 => => extracting sha256:1efc276f4ff952c055dea726cfc96ec6a4fdb8b62d9eed816bd2b788f2860ad7                                                                                     46.2s
 => => extracting sha256:d73cf48caaac2e45ad76a2a9eb3b311d0e4eb1d804e3d2b9cf075a1fa31e6f92                                                                                      2.9s
 => [internal] load build context                                                                                                                                              0.0s
 => => transferring context: 982.38kB                                                                                                                                          0.0s
 => [build-env 2/6] WORKDIR /app                                                                                                                                               2.3s
 => [build-env 3/6] COPY pom.xml ./                                                                                                                                            0.0s
 => [build-env 4/6] RUN mvn verify --fail-never                                                                                                                               24.7s
 => [build-env 5/6] COPY . ./                                                                                                                                                  0.0s
 => [build-env 6/6] RUN mvn -Dmaven.test.skip=true package                                                                                                                     8.6s
 => [stage-1 2/2] COPY --from=build-env /app/target/ /app/                                                                                                                     0.2s
 => exporting to image                                                                                                                                                         0.4s
 => => exporting layers                                                                                                                                                        0.3s
 => => writing image sha256:4c72ae5977ef20514f9010c040cc3a629d1c66603bf91e7f67f2ed875d7def02                                                                                   0.0s
 => => naming to us-east1-docker.pkg.dev/qwiklabs-gcp-00-1176b71068a0/container-dev-repo/java-hello-world:tag1                                                                 0.0s
student_01_0fb7ab211dcf@cloudshell:~/cloud-code-samples/java/java-hello-world (qwiklabs-gcp-00-1176b71068a0)$ 
```

### Push the Container Image to Artifact Registry

- Run the following command to push the container image to the repository you created:

```sh
student_01_0fb7ab211dcf@cloudshell:~/cloud-code-samples/java/java-hello-world (qwiklabs-gcp-00-1176b71068a0)$ docker push us-east1-docker.pkg.dev/qwiklabs-gcp-00-1176b71068a0/container-dev-repo/java-hello-world:tag1
The push refers to repository [us-east1-docker.pkg.dev/qwiklabs-gcp-00-1176b71068a0/container-dev-repo/java-hello-world]
a30faf36c63f: Pushed 
d7802b8508af: Pushed 
e3abdc2e9252: Pushed 
eafe6e032dbd: Pushed 
92a4e8a3140f: Pushed 
tag1: digest: sha256:b75959e667c2c7db6929995a2c9f547d3a45081e03b4877bcfb302064b4b048f size: 1371
student_01_0fb7ab211dcf@cloudshell:~/cloud-code-samples/java/java-hello-world (qwiklabs-gcp-00-1176b71068a0)$ 
```

### Review the image in Artifact Registry

1. In  **Artifact Registry > Repositories**, click into `container-dev-repo` and check that the `java-hello-world` image is there.
2. Click on the image and note the image tagged `tag1`. You can see that Vulnerability Scanning is running or already completed and the number of vulnerabilities detected is visible.

Click on the number of vulnerabilities and you will see the list of  vulnerabilities detected in the image, with the CVE bulletin name and  the severity. Click **VIEW** on each listed vulnerability to get more details:

## Integration with Cloud Code

In this section you use the Artifact Registry Docker image repository with [Cloud Code](https://cloud.google.com/code).

### Deploy the Application to GKE Cluster from Cloud Code

1. From the `java-hello-world` folder run the following command to open Cloud Shell Editor and add the application folder to this workspace.

```sh
cloudshell workspace .
```

   Click on **View > Command Palette...** and type **Run on Kubernetes** and select **Cloud Code: Run on Kubernetes**.

Choose **cloud-code-samples/java/java-hello-world/skaffold.yaml** and then **dockerfile**.

```sh
Starting to debug the app using configuration 'Kubernetes: Run/Debug' from .vscode/launch.json...
To view more detailed logs, go to Output channel : "Kubernetes: Run/Debug - Detailed"
Input image registry us-east1-docker.pkg.dev/qwiklabs-gcp-00-1176b71068a0/container-dev-repo does not match the expected image registry gcr.io/qwiklabs-gcp-00-1176b71068a0 based on the GCP project associated with the current context. Please ensure that the cluster is authorized to pull images from the input registry.

Update initiated
Build started for artifact java-hello-world
Build completed for artifact java-hello-world

Deploy started
Status check started
Resource pod/java-hello-world-79bfd8c959-wsl7p status updated to In Progress
Resource deployment/java-hello-world status updated to In Progress
Resource deployment/java-hello-world status updated to In Progress
Resource deployment/java-hello-world status completed successfully
Status check succeeded

**************URLs*****************
Deploy completed

Forwarded URL from service java-hello-world-external: http://localhost:4503
Debuggable container started pod/java-hello-world-79bfd8c959-wsl7p:server (default)
Update succeeded
***********************************
Watching for changes...
To disable watch mode for subsequent runs, set watch to false in your launch configuration /home/student_01_0fb7ab211dcf/.vscode/launch.json and relaunch the application.

```

1. When you execute **Run on Kubernetes** for the first  time Cloud Code prompts you for the target image repository location.  Once provided, the repository url is stored in the file `.vscode/launch.json` which is created in the application folder.

In the output pane you see that the build starts for the application image `java-hello-world,` the image is uploaded to the Artifact Registry repository configured previously.

1. In **Artifact Registry > Repositories** click into `container-dev-repo` and check that the `java-hello-world` image and note a new image tagged `latest`.

### Review the Deployed Application

1. Go back to Cloud Shell Editor. When deployment is complete  Skaffold/Cloud Code will print the exposed url where the service have  been forwarded, click on the link - **Open Web Preview**:

![A group celebrating](https://4503-cs-69aafbb9-a5b5-4ee1-af76-c7f272bcf4e5.ql-asia-southeast1-mijh.cloudshell.dev/img/KE-hello-world.svg) 			 		
```sh
   It's running!

   Congratulations, you successfully deployed a Kubernetes application with Cloud Code!
```


### Update application code

Now update the application to see the change implemented immediately in the deployment on the cluster:

1. Open the `HelloWorldController.java` by clicking on **View > Command Palette...** and then click one backspace and then enter the path **src/main/java/cloudcode/helloworld/web** and click the option starting with `Hello..` .
2. Change the text in row 20 from "It's running!" to "It's updated!".  You should see the build and deployment process starting immediately.
3. At the end of the deploy click again on the forwarded url or refresh  the browser window with the application to see your change deployed:

```sh

It's updated!
Congratulations, you successfully deployed a Kubernetes application with Cloud Code!

```

In the Cloud console go to **Navigation Menu > Artifact Registry > Repositories** and click into `container-dev-repo` to check that the `java-hello-world` image and note the new image.

## Working with language packages

In this section you will set up an Artifact Registry Java repository  and upload packages to it, leveraging them in different applications.

### Create a Java package repository

```sh
student_01_0fb7ab211dcf@cloudshell:~ (qwiklabs-gcp-00-1176b71068a0)$ gcloud artifacts repositories create container-dev-java-repo \
    --repository-format=maven \
    --location=us-east1 \
    --description="Java package repository for Container Dev Workshop"
Create request issued for: [container-dev-java-repo]
Waiting for operation [projects/qwiklabs-gcp-00-1176b71068a0/locations/us-east1/operations/55a307df-d054-43ae-b4c1-2e6b3eed4ef3] to complete...done.                               
Created repository [container-dev-java-repo].
student_01_0fb7ab211dcf@cloudshell:~ (qwiklabs-gcp-00-1176b71068a0)$ 
```

In the Cloud console go to  **Artifact Registry > Repositories** and notice your newly created Maven repository named `container-dev-java-repo`, if you click on it you can see that it's empty at the moment.

### Set up authentication to Artifact Repository

- Use the following command to update the well-known location for  Application Default Credentials (ADC) with your user account credentials so that the Artifact Registry credential helper can authenticate using  them when connecting with repositories:

```sh
student_01_0fb7ab211dcf@cloudshell:~ (qwiklabs-gcp-00-1176b71068a0)$ gcloud auth login --update-adc

You are already authenticated with gcloud when running
inside the Cloud Shell and so do not need to run this
command. Do you wish to proceed anyway?

Do you want to continue (Y/n)?  y

Go to the following link in your browser, and complete the sign-in prompts:

    https://accounts.google.com/o/oauth2/auth?response_type=code&client_id=32555940559.apps.googleusercontent.com&redirect_uri=https%3A%2F%2Fsdk.cloud.google.com%2Fauthcode.html&scope=openid+https%3A%2F%2Fwww.googleapis.com%2Fauth%2Fuserinfo.email+https%3A%2F%2Fwww.googleapis.com%2Fauth%2Fcloud-platform+https%3A%2F%2Fwww.googleapis.com%2Fauth%2Fappengine.admin+https%3A%2F%2Fwww.googleapis.com%2Fauth%2Fsqlservice.login+https%3A%2F%2Fwww.googleapis.com%2Fauth%2Fcompute+https%3A%2F%2Fwww.googleapis.com%2Fauth%2Faccounts.reauth&state=kMWqRaJeVFqkK1xaICcvKm8BCrEQxS&prompt=consent&token_usage=remote&access_type=offline&code_challenge=ngJXZhe4QXt42ErD8uvsufz-9tp6XUHtosxl_JojLZk&code_challenge_method=S256

Once finished, enter the verification code provided in your browser: 4/0AdLIrYc6HS3gOTFLPpEMG-EwEhxZ35C4xDo4jDSbqK2oaCXvIZVSzXrxMnvYTXrcHhcCRg

Application Default Credentials (ADC) were updated.

You are now logged in as [student-01-0fb7ab211dcf@qwiklabs.net].
Your current project is [qwiklabs-gcp-00-1176b71068a0].  You can change this setting by running:
  $ gcloud config set project PROJECT_ID
student_01_0fb7ab211dcf@cloudshell:~ (qwiklabs-gcp-00-1176b71068a0)$ 
```

### Configure Maven for Artifact Registry

1. Run the following command to print the repository configuration to add to your Java project:

```sh
student_01_0fb7ab211dcf@cloudshell:~ (qwiklabs-gcp-00-1176b71068a0)$ gcloud artifacts print-settings mvn \
    --repository=container-dev-java-repo \
    --location=us-east1
<!-- Insert following snippet into your pom.xml -->

<project>
  <distributionManagement>
    <snapshotRepository>
      <id>artifact-registry</id>
      <url>artifactregistry://us-east1-maven.pkg.dev/qwiklabs-gcp-00-1176b71068a0/container-dev-java-repo</url>
    </snapshotRepository>
    <repository>
      <id>artifact-registry</id>
      <url>artifactregistry://us-east1-maven.pkg.dev/qwiklabs-gcp-00-1176b71068a0/container-dev-java-repo</url>
    </repository>
  </distributionManagement>

  <repositories>
    <repository>
      <id>artifact-registry</id>
      <url>artifactregistry://us-east1-maven.pkg.dev/qwiklabs-gcp-00-1176b71068a0/container-dev-java-repo</url>
      <releases>
        <enabled>true</enabled>
      </releases>
      <snapshots>
        <enabled>true</enabled>
      </snapshots>
    </repository>
  </repositories>

  <build>
    <extensions>
      <extension>
        <groupId>com.google.cloud.artifactregistry</groupId>
        <artifactId>artifactregistry-maven-wagon</artifactId>
        <version>2.2.0</version>
      </extension>
    </extensions>
  </build>
</project>

student_01_0fb7ab211dcf@cloudshell:~ (qwiklabs-gcp-00-1176b71068a0)$ 
```

Open the `pom.xml` in Cloud Shell Editor and add the returned settings to the appropriate sections in the file:

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
 <modelVersion>4.0.0</modelVersion>

 <artifactId>hello-world</artifactId>
 <packaging>jar</packaging>
 <name>Cloud Code Hello World</name>
 <description>Getting started with Cloud Code</description>
 <version>1.0.0</version>
<distributionManagement>
   <snapshotRepository>
     <id>artifact-registry</id>
     <url>artifactregistry://us-east1-maven.pkg.dev/qwiklabs-gcp-00-1176b71068a0/container-dev-java-repo</url>
   </snapshotRepository>
   <repository>
     <id>artifact-registry</id>
     <url>artifactregistry://us-east1-maven.pkg.dev/qwiklabs-gcp-00-1176b71068a0/container-dev-java-repo</url>
   </repository>
 </distributionManagement>

 <repositories>
   <repository>
     <id>artifact-registry</id>
     <url>artifactregistry://us-east1-maven.pkg.dev/qwiklabs-gcp-00-1176b71068a0/container-dev-java-repo</url>
     <releases>
       <enabled>true</enabled>
     </releases>
     <snapshots>
       <enabled>true</enabled>
     </snapshots>
   </repository>
 </repositories>

 <parent>
   <groupId>org.springframework.boot</groupId>
   <artifactId>spring-boot-starter-parent</artifactId>
   <version>2.6.3</version>
 </parent>

 <properties>
   <java.version>1.8</java.version>
   <checkstyle.config.location>./checkstyle.xml</checkstyle.config.location>
 </properties>

 <build>
   <plugins>
     <plugin>
       <groupId>com.google.cloud.tools</groupId>
       <artifactId>jib-maven-plugin</artifactId>
       <version>3.2.0</version>
     </plugin>
     <plugin>
       <groupId>org.springframework.boot</groupId>
       <artifactId>spring-boot-maven-plugin</artifactId>
     </plugin>
     <plugin>
       <groupId>org.apache.maven.plugins</groupId>
       <artifactId>maven-checkstyle-plugin</artifactId>
       <version>3.1.2</version>
     </plugin>
   </plugins>
   <extensions>
     <extension>
       <groupId>com.google.cloud.artifactregistry</groupId>
       <artifactId>artifactregistry-maven-wagon</artifactId>
       <version>2.1.0</version>
     </extension>
   </extensions>
 </build>

 <!-- The Spring Cloud GCP BOM will manage spring-cloud-gcp version numbers for you. -->
 <dependencyManagement>
   <dependencies>
     <dependency>
       <groupId>org.springframework.cloud</groupId>
       <artifactId>spring-cloud-gcp-dependencies</artifactId>
       <version>1.2.8.RELEASE</version>
       <type>pom</type>
       <scope>import</scope>
     </dependency>
   </dependencies>
 </dependencyManagement>

 <dependencies>

   <dependency>
     <groupId>org.springframework.boot</groupId>
     <artifactId>spring-boot-starter</artifactId>
   </dependency>

   <dependency>
     <groupId>org.springframework.boot</groupId>
     <artifactId>spring-boot-starter-jetty</artifactId>
   </dependency>

   <dependency>
     <groupId>org.springframework</groupId>
     <artifactId>spring-webmvc</artifactId>
   </dependency>

   <dependency>
     <groupId>org.springframework.boot</groupId>
     <artifactId>spring-boot-starter-thymeleaf</artifactId>
   </dependency>

   <dependency>
     <groupId>org.springframework.boot</groupId>
     <artifactId>spring-boot-starter-test</artifactId>
     <scope>test</scope>
   </dependency>

   <dependency>
     <groupId>org.springframework.cloud</groupId>
     <artifactId>spring-cloud-gcp-starter-logging</artifactId>
   </dependency>

 </dependencies>

</project>

```

### Upload your Java package to Artifact Registry

With Artifact Registry configured in Maven, you can now use Artifact  Registry to store Java Jars for use by other projects in your  organization.

- Run the following command to upload your Java package to Artifact Registry:

```sh
student_01_0fb7ab211dcf@cloudshell:~/cloud-code-samples/java/java-hello-world (qwiklabs-gcp-00-1176b71068a0)$ mvn deploy

{snip}
[INFO] --- deploy:2.8.2:deploy (default-deploy) @ hello-world ---
Uploading to artifact-registry: artifactregistry://us-east1-maven.pkg.dev/qwiklabs-gcp-00-1176b71068a0/container-dev-java-repo/org/springframework/boot/hello-world/1.0.0/hello-world-1.0.0.jar
[INFO] ArtifactRegistry Maven Wagon: Retrieving credentials...
[INFO] Trying Application Default Credentials...
[INFO] Using Application Default Credentials.
Uploaded to artifact-registry: artifactregistry://us-east1-maven.pkg.dev/qwiklabs-gcp-00-1176b71068a0/container-dev-java-repo/org/springframework/boot/hello-world/1.0.0/hello-world-1.0.0.jar (44 MB at 6.3 MB/s)
Uploading to artifact-registry: artifactregistry://us-east1-maven.pkg.dev/qwiklabs-gcp-00-1176b71068a0/container-dev-java-repo/org/springframework/boot/hello-world/1.0.0/hello-world-1.0.0.pom
Uploaded to artifact-registry: artifactregistry://us-east1-maven.pkg.dev/qwiklabs-gcp-00-1176b71068a0/container-dev-java-repo/org/springframework/boot/hello-world/1.0.0/hello-world-1.0.0.pom (3.6 kB at 1.9 kB/s)
Downloading from artifact-registry: artifactregistry://us-east1-maven.pkg.dev/qwiklabs-gcp-00-1176b71068a0/container-dev-java-repo/org/springframework/boot/hello-world/maven-metadata.xml
Uploading to artifact-registry: artifactregistry://us-east1-maven.pkg.dev/qwiklabs-gcp-00-1176b71068a0/container-dev-java-repo/org/springframework/boot/hello-world/maven-metadata.xml
Uploaded to artifact-registry: artifactregistry://us-east1-maven.pkg.dev/qwiklabs-gcp-00-1176b71068a0/container-dev-java-repo/org/springframework/boot/hello-world/maven-metadata.xml (315 B at 173 B/s)
[INFO] ------------------------------------------------------------------------
[INFO] BUILD SUCCESS
[INFO] ------------------------------------------------------------------------
[INFO] Total time:  01:25 min
[INFO] Finished at: 2024-05-09T16:17:50Z
[INFO] ------------------------------------------------------------------------
student_01_0fb7ab211dcf@cloudshell:~/cloud-code-samples/java/java-hello-world (qwiklabs-gcp-00-1176b71068a0)$ 
```



### Check the Java package in Artifact Registry

In the Cloud console go to **Artifact Registry > Repositories** and click into `container-dev-java-repo` to check that the `hello-world` binary artifact is there:



```sh
student_01_0fb7ab211dcf@cloudshell:~/cloud-code-samples/java/java-hello-world (qwiklabs-gcp-00-1176b71068a0)$ history 
    1  export PROJECT_ID=$(gcloud config get-value project)
    2  export PROJECT_NUMBER=$(gcloud projects describe $PROJECT_ID --format='value(projectNumber)')
    3  export REGION=us-east1
    4  gcloud config set compute/region $REGION
    5  gcloud services enable   cloudresourcemanager.googleapis.com   container.googleapis.com   artifactregistry.googleapis.com   containerregistry.googleapis.com   containerscanning.googleapis.com
    6  git clone https://github.com/GoogleCloudPlatform/cloud-code-samples/
    7  cd ~/cloud-code-samples
    8  gcloud container clusters create container-dev-cluster --zone=us-east1-d
    9  gcloud artifacts repositories create container-dev-repo --repository-format=docker   --location=$REGION   --description="Docker repository for Container Dev Workshop"
   10  gcloud auth configure-docker us-east1-docker.pkg.dev
   11  cd ~/cloud-code-samples/java/java-hello-world
   12  ls
   13  cat Dockerfile 
   14  docker build -t us-east1-docker.pkg.dev/qwiklabs-gcp-00-1176b71068a0/container-dev-repo/java-hello-world:tag1 .
   15  docker push us-east1-docker.pkg.dev/qwiklabs-gcp-00-1176b71068a0/container-dev-repo/java-hello-world:tag1
   16  cloudshell workspace .
   17  gcloud artifacts repositories create container-dev-java-repo     --repository-format=maven     --location=us-east1     --description="Java package repository for Container Dev Workshop"
   18  gcloud auth login --update-adc
   19  gcloud artifacts print-settings mvn     --repository=container-dev-java-repo     --location=us-east1
   20  mvn deploy
   21  ls
   22  cd cloud-code-samples/
   23  ls
   24  cd java/
   25  ls
   26  cd java-hello-world/
   27  mvn deploy
   28  history 
student_01_0fb7ab211dcf@cloudshell:~/cloud-code-samples/java/java-hello-world (qwiklabs-gcp-00-1176b71068a0)$ 
```



Congratulations! In this lab you learned about some of the features  available in Artifact Registry. You first created repositories for  containers and language packages. You then managed container images with Artifact Registry and integrated it with Cloud Code. Finally, you  configured Maven to use Artifact Registry for Java dependencies. You now have a solid understanding of features available in Artifact Registry.
