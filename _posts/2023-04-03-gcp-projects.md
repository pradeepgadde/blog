---


layout: single
title:  "Google Cloud Projects"
date:   2023-04-03 08:59:04 +0530
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

# Google Cloud Projects

Google Cloud projects form the basis for creating, enabling, and using all Google Cloud services including managing APIs, enabling billing, adding and removing collaborators, and managing permissions for Google Cloud resources.

he following are used to identify your project:

- **Project name**: A human-readable name for your project.

  The project name isn't used by any Google APIs. You can edit the project name at any time during or after project creation. Project names do not need to be unique.

- **Project ID**: A globally unique identifier for your project.

  A project ID is a unique string used to differentiate your project from all others in Google Cloud. You can use the Google Cloud console to generate a project ID, or you can choose your own. You can only modify the project ID when you're creating the project.

  Project ID requirements:

  - Must be 6 to 30 characters in length.
  - Can only contain lowercase letters, numbers, and hyphens.
  - Must start with a letter.
  - Cannot end with a hyphen.
  - Cannot be in use or previously used; this includes deleted projects.
  - Cannot contain restricted strings, such as `google` and `ssl`.

- **Project number**: An automatically generated unique identifier for your project.

```sh
gaddepradeep@cloudshell:~$ gcloud projects create -h
Usage: gcloud projects create [PROJECT_ID] [optional flags]
  optional flags may be  --enable-cloud-apis | --folder | --help | --labels |
                         --name | --organization | --set-as-default

For detailed information on this command and its flags, run:
  gcloud projects create --help
gaddepradeep@cloudshell:~$
```

```sh
gcloud projects create PROJECT_ID
```

```sh
gcloud projects create PROJECT_ID --organization=ORGANIZATION_ID

gcloud projects create PROJECT_ID --folder=FOLDER_ID
```

To create a project with an organization or a folder as parent, use the `--organization` or `--folder` flags. As a resource can only have one parent, only one of these flags can be used:

```sh
gcloud projects describe PROJECT_ID
```



![]({{ site.url }}{{ site.baseurl }}/assets/images/gcp-500.png)

![]({{ site.url }}{{ site.baseurl }}/assets/images/gcp-501.png)

![]({{ site.url }}{{ site.baseurl }}/assets/images/gcp-502.png)

![]({{ site.url }}{{ site.baseurl }}/assets/images/gcp-503.png)



## Shutting down (deleting) projects

You can shut down projects using the Google Cloud console or the [`projects.delete`](https://cloud.google.com/resource-manager/reference/rest/v3/projects/delete) method in the API. A project must have a lifecycle state of `ACTIVE` to be shut down in this way.



This method immediately marks a project to be deleted. A  notification email is sent to the user who initiated the delete  operation and the Technical category contacts that are listed in [Essential Contacts](https://cloud.google.com/resource-manager/docs/managing-notification-contacts) on a best effort basis; if the notification fails to send, the project  is still marked to be deleted. If there's no contact in the Technical  category, the fallback contact isn't notified.



![]({{ site.url }}{{ site.baseurl }}/assets/images/gcp-504.png)

![]({{ site.url }}{{ site.baseurl }}/assets/images/gcp-505.png)

![]({{ site.url }}{{ site.baseurl }}/assets/images/gcp-506.png)

A project that is marked for deletion isn't usable. If the project has a billing account associated with it, that association is broken and isn't reinstated if the project delete operation is canceled. After 30 days, the project is fully deleted. Until it is fully deleted, the project might still be visible, although it isn't usable.



To stop the project delete process during the 30-day period, see the [steps to restore a project](https://cloud.google.com/resource-manager/docs/creating-managing-projects#restoring_a_project). You can verify the number of days left in the 30-day period by using the [`gcloud projects describe`](https://cloud.google.com/sdk/gcloud/reference/projects/describe) Google Cloud CLI method.

```sh
Welcome to Cloud Shell! Type "help" to get started.
To set your Cloud Platform project in this session use “gcloud config set project [PROJECT_ID]”
gaddepradeep@cloudshell:~$ gcloud projects list
Listed 0 items.
gaddepradeep@cloudshell:~$ gcloud projects describe pradeepgadde
createTime: '2023-04-05T02:23:10.714Z'
lifecycleState: DELETE_REQUESTED
name: pradeepgadde
projectId: pradeepgadde
projectNumber: '616276308247'
gaddepradeep@cloudshell:~$
```

