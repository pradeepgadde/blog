---

layout: single
title:  "Cloud Source Repositories"
date:   2023-03-29 11:59:04 +0530
categories: Cloud
tags: GCP
classes: wide
show_date: true
header:
  overlay_image: /assets/images/gpcne.png
  og_image: /assets/images/gpcne.png
  teaser: /assets/images/gpcne.png
author:
  name     : "Cloud Network Engineer"
  avatar   : "/assets/images/gpcne.png"

sidebar:
  - title: "Topics"
    nav: my-sidebar

---

# Cloud Source Repositories

[Google Cloud Source Repositories](https://cloud.google.com/source-repositories/) provides Git version control to support collaborative development of  any application or service. In this lab, you will create a local Git  repository that contains a sample file, add a Google Source Repository  as a remote, and push the contents of the local repository. You will use the source browser included in Source Repositories to view your  repository files from within the Cloud Console.



- gcloud source repos clone REPO_DEMO
- cd REPO_DEMO
- echo 'Hello World!' > myfile.txt
- git config --global user.email "you@example.com"
- git config --global user.name "Your Name"
- git add myfile.txt
- git commit -m "First file using Cloud Source Repositories" myfile.txt
- git push origin master
- gcloud source repos list

```sh
Welcome to Cloud Shell! Type "help" to get started.
Your Cloud Platform project in this session is set to qwiklabs-gcp-00-b39bf9625237.
Use “gcloud config set project [PROJECT_ID]” to change to a different project.
student_01_2dc359053fcc@cloudshell:~ (qwiklabs-gcp-00-b39bf9625237)$ gcloud source repos create REPO_DEMO
Created [REPO_DEMO].
WARNING: You may be billed for this repository. See https://cloud.google.com/source-repositories/docs/pricing for details.
student_01_2dc359053fcc@cloudshell:~ (qwiklabs-gcp-00-b39bf9625237)$

```



```sh
student_01_2dc359053fcc@cloudshell:~ (qwiklabs-gcp-00-b39bf9625237)$ gcloud source repos clone REPO_DEMO
Cloning into '/home/student_01_2dc359053fcc/REPO_DEMO'...
warning: You appear to have cloned an empty repository.
Project [qwiklabs-gcp-00-b39bf9625237] repository [REPO_DEMO] was cloned to [/home/student_01_2dc359053fcc/REPO_DEMO].
student_01_2dc359053fcc@cloudshell:~ (qwiklabs-gcp-00-b39bf9625237)$
```





```sh
Welcome to Cloud Shell! Type "help" to get started.
Your Cloud Platform project in this session is set to qwiklabs-gcp-00-b39bf9625237.
Use “gcloud config set project [PROJECT_ID]” to change to a different project.
student_01_2dc359053fcc@cloudshell:~ (qwiklabs-gcp-00-b39bf9625237)$ gcloud source repos create REPO_DEMO
Created [REPO_DEMO].
WARNING: You may be billed for this repository. See https://cloud.google.com/source-repositories/docs/pricing for details.
student_01_2dc359053fcc@cloudshell:~ (qwiklabs-gcp-00-b39bf9625237)$ gcloud source repos clone REPO_DEMO
Cloning into '/home/student_01_2dc359053fcc/REPO_DEMO'...
warning: You appear to have cloned an empty repository.
Project [qwiklabs-gcp-00-b39bf9625237] repository [REPO_DEMO] was cloned to [/home/student_01_2dc359053fcc/REPO_DEMO].
student_01_2dc359053fcc@cloudshell:~ (qwiklabs-gcp-00-b39bf9625237)$ cd REPO_DEMO
student_01_2dc359053fcc@cloudshell:~/REPO_DEMO (qwiklabs-gcp-00-b39bf9625237)$ echo 'Hello World!' > myfile.txt
student_01_2dc359053fcc@cloudshell:~/REPO_DEMO (qwiklabs-gcp-00-b39bf9625237)$ git config --global user.email "you@example.com"
student_01_2dc359053fcc@cloudshell:~/REPO_DEMO (qwiklabs-gcp-00-b39bf9625237)$ git config --global user.name "Your Name"
student_01_2dc359053fcc@cloudshell:~/REPO_DEMO (qwiklabs-gcp-00-b39bf9625237)$ git add myfile.txt
student_01_2dc359053fcc@cloudshell:~/REPO_DEMO (qwiklabs-gcp-00-b39bf9625237)$ git commit -m "First file using Cloud Source Repositories" myfile.txt
[master (root-commit) 2e7c195] First file using Cloud Source Repositories
 1 file changed, 1 insertion(+)
 create mode 100644 myfile.txt
student_01_2dc359053fcc@cloudshell:~/REPO_DEMO (qwiklabs-gcp-00-b39bf9625237)$ git push origin master
Enumerating objects: 3, done.
Counting objects: 100% (3/3), done.
Writing objects: 100% (3/3), 245 bytes | 245.00 KiB/s, done.
Total 3 (delta 0), reused 0 (delta 0), pack-reused 0
To https://source.developers.google.com/p/qwiklabs-gcp-00-b39bf9625237/r/REPO_DEMO
 * [new branch]      master -> master
student_01_2dc359053fcc@cloudshell:~/REPO_DEMO (qwiklabs-gcp-00-b39bf9625237)$
```



```sh
https://source.cloud.google.com/qwiklabs-gcp-00-b39bf9625237/REPO_DEMO/+/master:myfile.txt
```

![]({{ site.url }}{{ site.baseurl }}/assets/images/gcp-211.png)

![]({{ site.url }}{{ site.baseurl }}/assets/images/gcp-212.png)
