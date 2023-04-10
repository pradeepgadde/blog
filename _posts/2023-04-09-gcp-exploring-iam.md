---


layout: single
title:  "Exploring IAM"
date:   2023-04-09 08:59:04 +0530
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

# Exploring IAM

In this lab, you learn how to perform the following tasks:

- Use IAM to implement access control
- Restrict access to specific features or resources
- Use the Service Account User role

## Task 1. Setup for two users

### Sign in to the Cloud Console as the first user

1. This lab provisions you with two user names available in the **Connection Details** dialog. Sign in to the Cloud Console in an Incognito window as usual with the **Username 1** provided in Qwiklabs. Note that both user names use the same single password.

### Sign in to the Cloud Console as the second user

1. Open another tab in your incognito window.
2. Browse to  [console.cloud.google.com](http://console.cloud.google.com).
3. Click on the user icon in the top-right corner of the screen, and then click **Add account**.
4. Sign in to the Cloud Console with the **Username 2** provided in Qwiklabs.



```sh
Welcome to Cloud Shell! Type "help" to get started.
Your Cloud Platform project in this session is set to qwiklabs-gcp-04-3c899ff07c67.
Use “gcloud config set project [PROJECT_ID]” to change to a different project.
student_01_55032dd35b93@cloudshell:~ (qwiklabs-gcp-04-3c899ff07c67)$ gsutil ls gs://pradeepgadde
gs://pradeepgadde/sample.txt
student_01_55032dd35b93@cloudshell:~ (qwiklabs-gcp-04-3c899ff07c67)$
```

```sh
gcloud compute instances create demoiam --project=qwiklabs-gcp-04-3c899ff07c67 --zone=us-east1-d --machine-type=e2-micro --network-interface=network-tier=PREMIUM,subnet=default --metadata=enable-oslogin=true --maintenance-policy=MIGRATE --provisioning-model=STANDARD --service-account=read-bucket-objects@qwiklabs-gcp-04-3c899ff07c67.iam.gserviceaccount.com --scopes=https://www.googleapis.com/auth/cloud-platform --create-disk=auto-delete=yes,boot=yes,device-name=demoiam,image=projects/debian-cloud/global/images/debian-11-bullseye-v20230306,mode=rw,size=10,type=projects/qwiklabs-gcp-04-3c899ff07c67/zones/us-east1-d/diskTypes/pd-balanced --no-shielded-secure-boot --shielded-vtpm --shielded-integrity-monitoring --labels=ec-src=vm_add-gcloud --reservation-affinity=any
```

At this point, you might have the user test access by connecting via SSH to the VM and performing the next actions. As the owner of the project, you already possess the Service Account User role. So you can simulate  what the user would experience by just using SSH to access the VM from  the Cloud Console

```sh
Linux demoiam 5.10.0-21-cloud-amd64 #1 SMP Debian 5.10.162-1 (2023-01-21) x86_64

The programs included with the Debian GNU/Linux system are free software;
the exact distribution terms for each program are described in the
individual files in /usr/share/doc/*/copyright.

Debian GNU/Linux comes with ABSOLUTELY NO WARRANTY, to the extent
permitted by applicable law.
Creating directory '/home/student-01-cd9de465be5c'.
student-01-cd9de465be5c@demoiam:~$ gcloud compute instances list
ERROR: (gcloud.compute.instances.list) Some requests did not succeed:
 - Required 'compute.instances.list' permission for 'projects/qwiklabs-gcp-04-3c899ff07c67'

student-01-cd9de465be5c@demoiam:~$ 

```

```sh
student-01-cd9de465be5c@demoiam:~$ gsutil cp gs://pradeepgadde/sample.txt .
Copying gs://pradeepgadde/sample.txt...
/ [1 files][ 87.7 KiB/ 87.7 KiB]                                                
Operation completed over 1 objects/87.7 KiB.                                     
student-01-cd9de465be5c@demoiam:~$ 
```

```sh
student-01-cd9de465be5c@demoiam:~$ mv sample.txt sample2.txt
student-01-cd9de465be5c@demoiam:~$ gsutil cp sample2.txt gs://pradeepgadde
Copying file://sample2.txt [Content-Type=text/plain]...
AccessDeniedException: 403 read-bucket-objects@qwiklabs-gcp-04-3c899ff07c67.iam.gserviceaccount.com does not have storage.objects.create access to the Google Cloud Storage object. Permission 'storage.objects.create' denied on resource (or it may not exist).
student-01-cd9de465be5c@demoiam:~$ 
```

```sh
student-01-cd9de465be5c@demoiam:~$ gsutil cp sample2.txt gs://pradeepgadde
Copying file://sample2.txt [Content-Type=text/plain]...
/ [1 files][ 87.7 KiB/ 87.7 KiB]                                                
Operation completed over 1 objects/87.7 KiB.                                     
student-01-cd9de465be5c@demoiam:~$ 
```

## Review

In this lab you exercised granting and revoking IAM roles, first to a user, **Username 2**, and then to a Service Account User. You could allocate Service Account  User credentials and "bake" them into a VM to create specific-purpose  authorized bastion hosts.

![]({{ site.url }}{{ site.baseurl }}/assets/images/iam-1.png)

![]({{ site.url }}{{ site.baseurl }}/assets/images/iam-2.png)

![]({{ site.url }}{{ site.baseurl }}/assets/images/iam-3.png)

![]({{ site.url }}{{ site.baseurl }}/assets/images/iam-4.png)

![]({{ site.url }}{{ site.baseurl }}/assets/images/iam-5.png)

![]({{ site.url }}{{ site.baseurl }}/assets/images/iam-6.png)

![]({{ site.url }}{{ site.baseurl }}/assets/images/iam-7.png)

![]({{ site.url }}{{ site.baseurl }}/assets/images/iam-8.png)

![]({{ site.url }}{{ site.baseurl }}/assets/images/iam-9.png)

![]({{ site.url }}{{ site.baseurl }}/assets/images/iam-10.png)

![]({{ site.url }}{{ site.baseurl }}/assets/images/iam-11.png)

![]({{ site.url }}{{ site.baseurl }}/assets/images/iam-12.png)

![]({{ site.url }}{{ site.baseurl }}/assets/images/iam-13.png)

![]({{ site.url }}{{ site.baseurl }}/assets/images/iam-14.png)

![]({{ site.url }}{{ site.baseurl }}/assets/images/iam-15.png)

![]({{ site.url }}{{ site.baseurl }}/assets/images/iam-16.png)

![]({{ site.url }}{{ site.baseurl }}/assets/images/iam-17.png)

![]({{ site.url }}{{ site.baseurl }}/assets/images/iam-18.png)

![]({{ site.url }}{{ site.baseurl }}/assets/images/iam-19.png)

![]({{ site.url }}{{ site.baseurl }}/assets/images/iam-20.png)

![]({{ site.url }}{{ site.baseurl }}/assets/images/iam-21.png)

![]({{ site.url }}{{ site.baseurl }}/assets/images/iam-22.png)

![]({{ site.url }}{{ site.baseurl }}/assets/images/iam-23.png)

![]({{ site.url }}{{ site.baseurl }}/assets/images/iam-24.png)

![]({{ site.url }}{{ site.baseurl }}/assets/images/iam-25.png)







