---
layout: single
title:  "GCP DevOps Pipeline"
categories: Cloud
tags: GCP
classes: wide
show_date: true
header:
  overlay_image: /assets/images/gcp-devops.png
  og_image: /assets/images/gcp-devops.png
  teaser: /assets/images/pca-gcp.png
author:
  name     : "Professional Cloud Architect"
  avatar   : "/assets/images/pca-gcp.png"

sidebar:
  - title: "Topics"
    nav: my-sidebar

---
## GCP DevOps Pipeline

```sh
Welcome to Cloud Shell! Type "help" to get started.
Your Cloud Platform project in this session is set to qwiklabs-gcp-01-33eb1b4849d5.
Use “gcloud config set project [PROJECT_ID]” to change to a different project.
student_01_87494247c649@cloudshell:~ (qwiklabs-gcp-01-33eb1b4849d5)$ mkdir gcp-course
student_01_87494247c649@cloudshell:~ (qwiklabs-gcp-01-33eb1b4849d5)$ cd gcp-course
student_01_87494247c649@cloudshell:~/gcp-course (qwiklabs-gcp-01-33eb1b4849d5)$ gcloud source repos clone devops-repo
Cloning into '/home/student_01_87494247c649/gcp-course/devops-repo'...
warning: You appear to have cloned an empty repository.
Project [qwiklabs-gcp-01-33eb1b4849d5] repository [devops-repo] was cloned to [/home/student_01_87494247c649/gcp-course/devops-repo].
student_01_87494247c649@cloudshell:~/gcp-course (qwiklabs-gcp-01-33eb1b4849d5)$ cd devops-repo
student_01_87494247c649@cloudshell:~/gcp-course/devops-repo (qwiklabs-gcp-01-33eb1b4849d5)$ cd ~/gcp-course/devops-repo
git add --all
student_01_87494247c649@cloudshell:~/gcp-course/devops-repo (qwiklabs-gcp-01-33eb1b4849d5)$ git config --global user.email "you@example.com"
git config --global user.name "Pradeep"
student_01_87494247c649@cloudshell:~/gcp-course/devops-repo (qwiklabs-gcp-01-33eb1b4849d5)$ git commit -a -m "Initial Commit"
[master (root-commit) e803a32] Initial Commit
 4 files changed, 37 insertions(+)
 create mode 100644 main.py
 create mode 100644 templates/index.html
 create mode 100644 templates/layout.html
 create mode 100644 templates/requirements.txt
student_01_87494247c649@cloudshell:~/gcp-course/devops-repo (qwiklabs-gcp-01-33eb1b4849d5)$ git push origin master
Enumerating objects: 7, done.
Counting objects: 100% (7/7), done.
Delta compression using up to 2 threads
Compressing objects: 100% (6/6), done.
Writing objects: 100% (7/7), 943 bytes | 471.00 KiB/s, done.
Total 7 (delta 0), reused 0 (delta 0), pack-reused 0
remote: Waiting for private key checker: 4/4 objects left
To https://source.developers.google.com/p/qwiklabs-gcp-01-33eb1b4849d5/r/devops-repo
 * [new branch]      master -> master
student_01_87494247c649@cloudshell:~/gcp-course/devops-repo (qwiklabs-gcp-01-33eb1b4849d5)$ cd ~/gcp-course/devops-repo
student_01_87494247c649@cloudshell:~/gcp-course/devops-repo (qwiklabs-gcp-01-33eb1b4849d5)$ echo $DEVSHELL_PROJECT_ID
qwiklabs-gcp-01-33eb1b4849d5
student_01_87494247c649@cloudshell:~/gcp-course/devops-repo (qwiklabs-gcp-01-33eb1b4849d5)$
```

```sh
student_01_87494247c649@cloudshell:~/gcp-course/devops-repo (qwiklabs-gcp-01-33eb1b4849d5)$ gcloud artifacts repositories create devops-repo \
    --repository-format=docker \
    --location=us-east1
Create request issued for: [devops-repo]
Waiting for operation [projects/qwiklabs-gcp-01-33eb1b4849d5/locations/us-east1/operations/f463a973-bebe-4548-9cf8-9d047e61ed45] to complete...done.                               
Created repository [devops-repo].
student_01_87494247c649@cloudshell:~/gcp-course/devops-repo (qwiklabs-gcp-01-33eb1b4849d5)$
```
```sh
student_01_87494247c649@cloudshell:~/gcp-course/devops-repo (qwiklabs-
gcp-01-33eb1b4849d5)$ gcloud auth configure-docker us-east1-docker.pkg.dev
WARNING: Your config file at [/home/student_01_87494247c649/.docker/config.json] contains these credential helper entries:

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
student_01_87494247c649@cloudshell:~/gcp-course/devops-repo (qwiklabs-gcp-01-33eb1b4849d5)$
```
## Failed Build
```sh
student_01_87494247c649@cloudshell:~/gcp-course/devops-repo (qwiklabs-gcp-01-33eb1b4849d5)$ gcloud builds submit --tag us-east1-docker.pkg.dev/$DEVSHELL_PROJECT_ID/devops-repo/devops-image:v0.1 .
Creating temporary tarball archive of 5 file(s) totalling 983 bytes before compression.
Uploading tarball of [.] to [gs://qwiklabs-gcp-01-33eb1b4849d5_cloudbuild/source/1712542108.520821-b310c46bca9144a08b3cdd33adeea868.tgz]
Created [https://cloudbuild.googleapis.com/v1/projects/qwiklabs-gcp-01-33eb1b4849d5/locations/global/builds/ef0da099-6b06-45be-bbc9-340aa00a34cf].
Logs are available at [ https://console.cloud.google.com/cloud-build/builds/ef0da099-6b06-45be-bbc9-340aa00a34cf?project=1090604652612 ].
------------------------------------------------------------------------------- REMOTE BUILD OUTPUT --------------------------------------------------------------------------------
starting build "ef0da099-6b06-45be-bbc9-340aa00a34cf"

FETCHSOURCE
Fetching storage object: gs://qwiklabs-gcp-01-33eb1b4849d5_cloudbuild/source/1712542108.520821-b310c46bca9144a08b3cdd33adeea868.tgz#1712542110965843
Copying gs://qwiklabs-gcp-01-33eb1b4849d5_cloudbuild/source/1712542108.520821-b310c46bca9144a08b3cdd33adeea868.tgz#1712542110965843...
/ [1 files][   1012 B/   1012 B]                                                
Operation completed over 1 objects/1012.0 B.
BUILD
Already have image (with digest): gcr.io/cloud-builders/docker
Sending build context to Docker daemon  6.144kB
Step 1/7 : FROM python:3.9
3.9: Pulling from library/python
71215d55680c: Already exists
3cb8f9c23302: Already exists
5f899db30843: Already exists
567db630df8d: Already exists
d68cd2123173: Already exists
dade860a2d46: Pulling fs layer
ae4968fa8c5e: Pulling fs layer
0fb178dfc7b2: Pulling fs layer
ae4968fa8c5e: Download complete
0fb178dfc7b2: Verifying Checksum
0fb178dfc7b2: Download complete
dade860a2d46: Verifying Checksum
dade860a2d46: Download complete
dade860a2d46: Pull complete
ae4968fa8c5e: Pull complete
0fb178dfc7b2: Pull complete
Digest: sha256:5c72dd8986db8c289ad1a6514319c144ed8f72c6dd702873c4358445473b6f54
Status: Downloaded newer image for python:3.9
 61d1a8851fa0
Step 2/7 : WORKDIR /app
 Running in fb8756e20b09
Removing intermediate container fb8756e20b09
 36aa1da750bb
Step 3/7 : COPY . .
 ba3e16607c28
Step 4/7 : RUN pip install gunicorn
 Running in 31c0f1ebed9e
Collecting gunicorn
  Downloading gunicorn-21.2.0-py3-none-any.whl (80 kB)
     ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━ 80.2/80.2 kB 3.4 MB/s eta 0:00:00
Collecting packaging
  Downloading packaging-24.0-py3-none-any.whl (53 kB)
     ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━ 53.5/53.5 kB 9.3 MB/s eta 0:00:00
Installing collected packages: packaging, gunicorn
Successfully installed gunicorn-21.2.0 packaging-24.0
WARNING: Running pip as the 'root' user can result in broken permissions and conflicting behaviour with the system package manager. It is recommended to use a virtual environment instead: https://pip.pypa.io/warnings/venv

[notice] A new release of pip is available: 23.0.1 -> 24.0
[notice] To update, run: pip install --upgrade pip
Removing intermediate container 31c0f1ebed9e
 7d48711db7c1
Step 5/7 : RUN pip install -r requirements.txt
 Running in 5ae8eb5b9cf3
ERROR: Could not open requirements file: [Errno 2] No such file or directory: 'requirements.txt'

[notice] A new release of pip is available: 23.0.1 -> 24.0
[notice] To update, run: pip install --upgrade pip
The command '/bin/sh -c pip install -r requirements.txt' returned a non-zero code: 1
ERROR
ERROR: build step 0 "gcr.io/cloud-builders/docker" failed: step exited with non-zero status: 1

------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------

BUILD FAILURE: Build step failure: build step 0 "gcr.io/cloud-builders/docker" failed: step exited with non-zero status: 1
ERROR: (gcloud.builds.submit) build ef0da099-6b06-45be-bbc9-340aa00a34cf completed with status "FAILURE"
```

```sh
student_01_87494247c649@cloudshell:~/gcp-course/devops-repo (qwiklabs-gcp-01-33eb1b4849d5)$ ls
Dockerfile  main.py  templates
student_01_87494247c649@cloudshell:~/gcp-course/devops-repo (qwiklabs-gcp-01-33eb1b4849d5)$ ls templates/
index.html  layout.html  requirements.txt
student_01_87494247c649@cloudshell:~/gcp-course/devops-repo (qwiklabs-gcp-01-33eb1b4849d5)$ mv templates/requirements.txt .
student_01_87494247c649@cloudshell:~/gcp-course/devops-repo (qwiklabs-gcp-01-33eb1b4849d5)$ ls
Dockerfile  main.py  requirements.txt  templates
```
## Successful Build
```sh
student_01_87494247c649@cloudshell:~/gcp-course/devops-repo (qwiklabs-gcp-01-33eb1b4849d5)$ gcloud builds submit --tag us-east1-docker.pkg.dev/$DEVSHELL_PROJECT_ID/devops-repo/devops-image:v0.1 .
Creating temporary tarball archive of 5 file(s) totalling 983 bytes before compression.
Uploading tarball of [.] to [gs://qwiklabs-gcp-01-33eb1b4849d5_cloudbuild/source/1712542250.554263-c8bf56c10a514d18aa293ee302fe3f4f.tgz]
Created [https://cloudbuild.googleapis.com/v1/projects/qwiklabs-gcp-01-33eb1b4849d5/locations/global/builds/8f6a7213-b321-4975-9c24-8c2b28f35cd1].
Logs are available at [ https://console.cloud.google.com/cloud-build/builds/8f6a7213-b321-4975-9c24-8c2b28f35cd1?project=1090604652612 ].
------------------------------------------------------------------------------- REMOTE BUILD OUTPUT --------------------------------------------------------------------------------
starting build "8f6a7213-b321-4975-9c24-8c2b28f35cd1"

FETCHSOURCE
Fetching storage object: gs://qwiklabs-gcp-01-33eb1b4849d5_cloudbuild/source/1712542250.554263-c8bf56c10a514d18aa293ee302fe3f4f.tgz#1712542252419310
Copying gs://qwiklabs-gcp-01-33eb1b4849d5_cloudbuild/source/1712542250.554263-c8bf56c10a514d18aa293ee302fe3f4f.tgz#1712542252419310...
/ [1 files][   1017 B/   1017 B]                                                
Operation completed over 1 objects/1017.0 B.
BUILD
Already have image (with digest): gcr.io/cloud-builders/docker
Sending build context to Docker daemon  6.144kB
Step 1/7 : FROM python:3.9
3.9: Pulling from library/python
71215d55680c: Already exists
3cb8f9c23302: Already exists
5f899db30843: Already exists
567db630df8d: Already exists
d68cd2123173: Already exists
dade860a2d46: Pulling fs layer
ae4968fa8c5e: Pulling fs layer
0fb178dfc7b2: Pulling fs layer
ae4968fa8c5e: Verifying Checksum
ae4968fa8c5e: Download complete
0fb178dfc7b2: Verifying Checksum
0fb178dfc7b2: Download complete
dade860a2d46: Verifying Checksum
dade860a2d46: Download complete
dade860a2d46: Pull complete
ae4968fa8c5e: Pull complete
0fb178dfc7b2: Pull complete
Digest: sha256:5c72dd8986db8c289ad1a6514319c144ed8f72c6dd702873c4358445473b6f54
Status: Downloaded newer image for python:3.9
 61d1a8851fa0
Step 2/7 : WORKDIR /app
 Running in 215840b30173
Removing intermediate container 215840b30173
 7d758244f13b
Step 3/7 : COPY . .
 22cbf3bcd891
Step 4/7 : RUN pip install gunicorn
 Running in 684a99be2246
Collecting gunicorn
  Downloading gunicorn-21.2.0-py3-none-any.whl (80 kB)
     ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━ 80.2/80.2 kB 3.2 MB/s eta 0:00:00
Collecting packaging
  Downloading packaging-24.0-py3-none-any.whl (53 kB)
     ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━ 53.5/53.5 kB 7.2 MB/s eta 0:00:00
Installing collected packages: packaging, gunicorn
Successfully installed gunicorn-21.2.0 packaging-24.0
WARNING: Running pip as the 'root' user can result in broken permissions and conflicting behaviour with the system package manager. It is recommended to use a virtual environment instead: https://pip.pypa.io/warnings/venv

[notice] A new release of pip is available: 23.0.1 -> 24.0
[notice] To update, run: pip install --upgrade pip
Removing intermediate container 684a99be2246
 ababd35f44b4
Step 5/7 : RUN pip install -r requirements.txt
 Running in c972d0245861
WARNING: Running pip as the 'root' user can result in broken permissions and conflicting behaviour with the system package manager. It is recommended to use a virtual environment instead: https://pip.pypa.io/warnings/venv

[notice] A new release of pip is available: 23.0.1 -> 24.0
[notice] To update, run: pip install --upgrade pip
Removing intermediate container c972d0245861
 4edb78b60012
Step 6/7 : ENV PORT=80
 Running in e2cf19f37439
Removing intermediate container e2cf19f37439
 56d44c30aadd
Step 7/7 : CMD exec gunicorn --bind :$PORT --workers 1 --threads 8 main:app
 Running in e62b650ac1e3
Removing intermediate container e62b650ac1e3
 ae0ae36ba955
Successfully built ae0ae36ba955
Successfully tagged us-east1-docker.pkg.dev/qwiklabs-gcp-01-33eb1b4849d5/devops-repo/devops-image:v0.1
PUSH
Pushing us-east1-docker.pkg.dev/qwiklabs-gcp-01-33eb1b4849d5/devops-repo/devops-image:v0.1
The push refers to repository [us-east1-docker.pkg.dev/qwiklabs-gcp-01-33eb1b4849d5/devops-repo/devops-image]
2bb3969d687e: Preparing
690d718e8be0: Preparing
22a0dfbf0dec: Preparing
6240dfd1479f: Preparing
17b02461857a: Preparing
5e7745c5bee2: Preparing
3aff9f9c9f44: Preparing
e077e19b6682: Preparing
21e1c4948146: Preparing
68866beb2ed2: Preparing
e6e2ab10dba6: Preparing
0238a1790324: Preparing
5e7745c5bee2: Waiting
3aff9f9c9f44: Waiting
e077e19b6682: Waiting
21e1c4948146: Waiting
68866beb2ed2: Waiting
e6e2ab10dba6: Waiting
0238a1790324: Waiting
6240dfd1479f: Pushed
22a0dfbf0dec: Pushed
2bb3969d687e: Pushed
690d718e8be0: Pushed
17b02461857a: Pushed
5e7745c5bee2: Pushed
3aff9f9c9f44: Pushed
e077e19b6682: Pushed
e6e2ab10dba6: Pushed
68866beb2ed2: Pushed
0238a1790324: Pushed
21e1c4948146: Pushed
v0.1: digest: sha256:7c8120ff7f7e67adad88b297068848f8c1d773eb61a25cfbf514c98454306e31 size: 2840
DONE
------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------
ID: 8f6a7213-b321-4975-9c24-8c2b28f35cd1
CREATE_TIME: 2024-04-08T02:10:53+00:00
DURATION: 1M27S
SOURCE: gs://qwiklabs-gcp-01-33eb1b4849d5_cloudbuild/source/1712542250.554263-c8bf56c10a514d18aa293ee302fe3f4f.tgz
IMAGES: us-east1-docker.pkg.dev/qwiklabs-gcp-01-33eb1b4849d5/devops-repo/devops-image:v0.1
STATUS: SUCCESS
student_01_87494247c649@cloudshell:~/gcp-course/devops-repo (qwiklabs-gcp-01-33eb1b4849d5)$ cd ~/gcp-course/devops-repo
git add --all
student_01_87494247c649@cloudshell:~/gcp-course/devops-repo (qwiklabs-gcp-01-33eb1b4849d5)$ git commit -am "Added Docker Support"
[master c2512ce] Added Docker Support
 3 files changed, 7 insertions(+), 1 deletion(-)
 create mode 100644 Dockerfile
 create mode 100644 requirements.txt
 delete mode 100644 templates/requirements.txt
student_01_87494247c649@cloudshell:~/gcp-course/devops-repo (qwiklabs-gcp-01-33eb1b4849d5)$ git push origin master
Enumerating objects: 7, done.
Counting objects: 100% (7/7), done.
Delta compression using up to 2 threads
Compressing objects: 100% (4/4), done.
Writing objects: 100% (5/5), 541 bytes | 541.00 KiB/s, done.
Total 5 (delta 1), reused 0 (delta 0), pack-reused 0
remote: Resolving deltas: 100% (1/1)
remote: Waiting for private key checker: 1/2 objects left
To https://source.developers.google.com/p/qwiklabs-gcp-01-33eb1b4849d5/r/devops-repo
   e803a32..c2512ce  master -> master
student_01_87494247c649@cloudshell:~/gcp-course/devops-repo (qwiklabs-gcp-01-33eb1b4849d5)$
```
```sh
student_01_87494247c649@cloudshell:~/gcp-course/devops-repo (qwiklabs-gcp-01-33eb1b4849d5)$ cd ~/gcp-course/devops-repo
git commit -a -m "Testing Build Trigger"
[master 8cde062] Testing Build Trigger
 1 file changed, 1 insertion(+), 1 deletion(-)
```

```sh
student_01_87494247c649@cloudshell:~/gcp-course/devops-repo (qwiklabs-gcp-01-33eb1b4849d5)$ git push origin master
Enumerating objects: 5, done.
Counting objects: 100% (5/5), done.
Delta compression using up to 2 threads
Compressing objects: 100% (3/3), done.
Writing objects: 100% (3/3), 305 bytes | 305.00 KiB/s, done.
Total 3 (delta 2), reused 0 (delta 0), pack-reused 0
remote: Resolving deltas: 100% (2/2)
remote: Waiting for private key checker: 1/1 objects left
To https://source.developers.google.com/p/qwiklabs-gcp-01-33eb1b4849d5/r/devops-repo
   c2512ce..8cde062  master -> master
student_01_87494247c649@cloudshell:~/gcp-course/devops-repo (qwiklabs-gcp-01-33eb1b4849d5)$ 
```

