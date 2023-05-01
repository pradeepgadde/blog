---
layout: single
title:  "Cloud Storage: Qwik Start"
date:   2023-05-01 01:59:05 +0530
categories: Cloud
tags: GCP
classes: wide
show_date: true
header:
  overlay_image: /assets/images/gcp-banner-1.png
  og_image: /assets/images/gcp-banner-1.png
  teaser: /assets/images/gpcne.png
author:
  name     : "Cloud Network Engineer"
  avatar   : "/assets/images/gpcne.png"

sidebar:
  - title: "Topics"
    nav: my-sidebar

---

# GCP Cloud Storage: Qwik Start

## Overview

Cloud Storage allows world-wide storage and retrieval of any amount  of data at any time. You can use Cloud Storage for a range of scenarios  including serving website content, storing data for archival and  disaster recovery, or distributing large data objects to users via  direct download.

In this hands-on lab we will learn how to create a storage bucket,  upload objects to it, create folders and subfolders in it, and make  objects publicly accessible using the Google Cloud command line.

Throughout this lab we'll be able to verify your work in the console by going to **Navigation menu** > **Cloud Storage**. We'll just need to refresh your browser after each command is run to see the new items we've created



## Task 1. Create a bucket

The Cloud Storage utility tool, [gsutil](https://cloud.google.com/storage/docs/gsutil), is installed and ready to use in Google Cloud. In this lab we use `gsutil` in Cloud Shell.

When we create a bucket we must follow the universal bucket naming rules, below.

**Bucket naming rules**

- Do not include sensitive information in the bucket name, because the bucket namespace is global and publicly visible.
- Bucket names must contain only lowercase letters, numbers, dashes  (-), underscores (_), and dots (.). Names containing dots require  [verification](https://cloud.google.com/storage/docs/domain-name-verification).
- Bucket names must start and end with a number or letter.
- Bucket names must contain 3 to 63 characters. Names containing dots  can contain up to 222 characters, but each dot-separated component can  be no longer than 63 characters.
- Bucket names cannot be represented as an IP address in dotted-decimal notation (for example, 192.168.5.4).
- Bucket names cannot begin with the "goog" prefix.
- Bucket names cannot contain "google" or close misspellings of "google".
- Also, for DNS compliance and future compatibility, you should not  use underscores (_) or have a period adjacent to another period or dash. For example, ".." or "-." or ".-" are not valid in DNS names.

Use the make bucket (`mb`) command to make a bucket, replacing `<YOUR_BUCKET_NAME>` with a unique name that follows the bucket naming rules:

This command is creating a bucket with default settings. To see what those default settings are, use the Cloud console **Navigation menu** > **Cloud Storage**, then click on your bucket name, and click on the **Configuration** tab.

That's it — we've just created a Cloud Storage bucket!

```sh
Welcome to Cloud Shell! Type "help" to get started.
Your Cloud Platform project in this session is set to qwiklabs-gcp-02-1087d640b711.
Use “gcloud config set project [PROJECT_ID]” to change to a different project.
student_01_a0075c49c713@cloudshell:~ (qwiklabs-gcp-02-1087d640b711)$ gsutil mb gs://pradeepgadde
Creating gs://pradeepgadde/...
student_01_a0075c49c713@cloudshell:~ (qwiklabs-gcp-02-1087d640b711)$
```





## Task 2. Upload an object into your bucket

Use Cloud Shell to upload an object into a bucket.

1. To download this image (ada.jpg) into your bucket, enter this command into Cloud Shell:

```sh
student_01_a0075c49c713@cloudshell:~ (qwiklabs-gcp-02-1087d640b711)$ curl https://upload.wikimedia.org/wikipedia/commons/thumb/a/a4/Ada_Lovelace_portrait.jpg/800px-Ada_Lovelace_portrait.jpg --output ada.jpg
  % Total    % Received % Xferd  Average Speed   Time    Time     Time  Current
                                 Dload  Upload   Total   Spent    Left  Speed
100  360k  100  360k    0     0   606k      0 --:--:-- --:--:-- --:--:--  606k
student_01_a0075c49c713@cloudshell:~ (qwiklabs-gcp-02-1087d640b711)$ gsutil cp ada.jpg gs://pradeepgadde
Copying file://ada.jpg [Content-Type=image/jpeg]...
- [1 files][360.1 KiB/360.1 KiB]
Operation completed over 1 objects/360.1 KiB.
student_01_a0075c49c713@cloudshell:~ (qwiklabs-gcp-02-1087d640b711)$
```

```sh
student_01_a0075c49c713@cloudshell:~ (qwiklabs-gcp-02-1087d640b711)$ rm ada.jpg
student_01_a0075c49c713@cloudshell:~ (qwiklabs-gcp-02-1087d640b711)$
```



## Task 3. Download an object from your bucket

- Use the `gsutil cp` command to download the image you stored in your bucket to Cloud Shell:

```sh
student_01_a0075c49c713@cloudshell:~ (qwiklabs-gcp-02-1087d640b711)$ gsutil cp -r gs://pradeepgadde/ada.jpg .
Copying gs://pradeepgadde/ada.jpg...
- [1 files][360.1 KiB/360.1 KiB]
Operation completed over 1 objects/360.1 KiB.
student_01_a0075c49c713@cloudshell:~ (qwiklabs-gcp-02-1087d640b711)$
```



## Task 4. Copy an object to a folder in the bucket

- Use the `gsutil cp` command to create a folder called `image-folder` and copy the image (ada.jpg) into it:

  ```sh
  student_01_a0075c49c713@cloudshell:~ (qwiklabs-gcp-02-1087d640b711)$ gsutil cp -r gs://pradeepgadde/ada.jpg gs://pradeepgadde/image-folder/
  Copying gs://pradeepgadde/ada.jpg [Content-Type=image/jpeg]...
  / [1 files][360.1 KiB/360.1 KiB]
  Operation completed over 1 objects/360.1 KiB.
  student_01_a0075c49c713@cloudshell:~ (qwiklabs-gcp-02-1087d640b711)$
  ```

  



## Task 5. List details for an object

- Use the `gsutil ls` command, with the `-l` flag to get some details about the image file you uploaded to your bucket:

```sh
student_01_a0075c49c713@cloudshell:~ (qwiklabs-gcp-02-1087d640b711)$ gsutil ls gs://pradeepgadde
gs://pradeepgadde/ada.jpg
gs://pradeepgadde/image-folder/
student_01_a0075c49c713@cloudshell:~ (qwiklabs-gcp-02-1087d640b711)$
```

```sh
student_01_a0075c49c713@cloudshell:~ (qwiklabs-gcp-02-1087d640b711)$ gsutil ls -l gs://pradeepgadde/ada.jpg
    368723  2023-05-01T08:55:49Z  gs://pradeepgadde/ada.jpg
TOTAL: 1 objects, 368723 bytes (360.08 KiB)
student_01_a0075c49c713@cloudshell:~ (qwiklabs-gcp-02-1087d640b711)$
```



## Task 6. Make your object publicly accessible

- Use the `gsutil acl ch` command to grant all users read permission for the object stored in your bucket:

```sh
student_01_a0075c49c713@cloudshell:~ (qwiklabs-gcp-02-1087d640b711)$ gsutil acl ch -u AllUsers:R gs://pradeepgadde/ada.jpg
Updated ACL on gs://pradeepgadde/ada.jpg
student_01_a0075c49c713@cloudshell:~ (qwiklabs-gcp-02-1087d640b711)$
```



## Task 7. Remove public access

1. To remove this permission, use the command:

```sh
student_01_a0075c49c713@cloudshell:~ (qwiklabs-gcp-02-1087d640b711)$ gsutil acl ch -d AllUsers gs://pradeepgadde/ada.jpg
Updated ACL on gs://pradeepgadde/ada.jpg
student_01_a0075c49c713@cloudshell:~ (qwiklabs-gcp-02-1087d640b711)$
```

```sh
student_01_a0075c49c713@cloudshell:~ (qwiklabs-gcp-02-1087d640b711)$ gsutil rm gs://pradeepgadde/ada.jpg
Removing gs://pradeepgadde/ada.jpg...
/ [1 objects]
Operation completed over 1 objects.
student_01_a0075c49c713@cloudshell:~ (qwiklabs-gcp-02-1087d640b711)$
```

![]({{ site.url }}{{ site.baseurl }}/assets/images/gcp-cs-1.png)

![]({{ site.url }}{{ site.baseurl }}/assets/images/gcp-cs-2.png)

![]({{ site.url }}{{ site.baseurl }}/assets/images/gcp-cs-3.png)

![]({{ site.url }}{{ site.baseurl }}/assets/images/gcp-cs-4.png)



```sh
student_01_a0075c49c713@cloudshell:~ (qwiklabs-gcp-02-1087d640b711)$ gsutil acl -h
CommandException: Incorrect option(s) specified. Usage:

  gsutil acl set [-f] [-r] [-a] <file-or-canned_acl_name> url...
  gsutil acl get url
  gsutil acl ch [-f] [-r] <grant>... url...

  where each <grant> is one of the following forms:

    -u <id>|<email>:<permission>
    -g <id>|<email>|<domain>|All|AllAuth:<permission>
    -p (viewers|editors|owners)-<project number>:<permission>
    -d <id>|<email>|<domain>|All|AllAuth|(viewers|editors|owners)-<project number>



For additional help run:
  gsutil help acl
student_01_a0075c49c713@cloudshell:~ (qwiklabs-gcp-02-1087d640b711)$ gsutil help acl
NAME
  acl - Get, set, or change bucket and/or object ACLs


SYNOPSIS
  gsutil acl set [-f] [-r] [-a] <file-or-canned_acl_name> url...
  gsutil acl get url
  gsutil acl ch [-f] [-r] <grant>... url...

  where each <grant> is one of the following forms:

    -u <id>|<email>:<permission>
    -g <id>|<email>|<domain>|All|AllAuth:<permission>
    -p (viewers|editors|owners)-<project number>:<permission>
    -d <id>|<email>|<domain>|All|AllAuth|(viewers|editors|owners)-<project number>



DESCRIPTION
  The acl command has three sub-commands:

GET
  The "acl get" command gets the ACL text for a bucket or object, which you can
  save and edit for the acl set command.


SET
  The "acl set" command allows you to set an Access Control List on one or
  more buckets and objects. The file-or-canned_acl_name parameter names either
  a canned ACL or the path to a file that contains ACL text. The simplest way
  to use the "acl set" command is to specify one of the canned ACLs, e.g.,:

    gsutil acl set private gs://bucket

  If you want to make an object or bucket publicly readable or writable, it is
  recommended to use "acl ch", to avoid accidentally removing OWNER permissions.
  See the "acl ch" section for details.

  See `Predefined ACLs
  <https://cloud.google.com/storage/docs/access-control/lists#predefined-acl>`_
  for a list of canned ACLs.

  If you want to define more fine-grained control over your data, you can
  retrieve an ACL using the "acl get" command, save the output to a file, edit
  the file, and then use the "acl set" command to set that ACL on the buckets
  and/or objects. For example:

    gsutil acl get gs://bucket/file.txt > acl.txt

  Make changes to acl.txt such as adding an additional grant, then:

    gsutil acl set acl.txt gs://cats/file.txt

  Note that you can set an ACL on multiple buckets or objects at once. For
  example, to set ACLs on all .jpg files found in a bucket:

    gsutil acl set acl.txt gs://bucket/**.jpg

  If you have a large number of ACLs to update you might want to use the
  gsutil -m option, to perform a parallel (multi-threaded/multi-processing)
  update:

    gsutil -m acl set acl.txt gs://bucket/**.jpg

  Note that multi-threading/multi-processing is only done when the named URLs
  refer to objects, which happens either if you name specific objects or
  if you enumerate objects by using an object wildcard or specifying
  the acl -r flag.


SET OPTIONS
  The "set" sub-command has the following options

  -R, -r      Performs "acl set" request recursively, to all objects under
              the specified URL.

  -a          Performs "acl set" request on all object versions.

  -f          Normally gsutil stops at the first error. The -f option causes
              it to continue when it encounters errors. If some of the ACLs
              couldn't be set, gsutil's exit status will be non-zero even if
              this flag is set. This option is implicitly set when running
              "gsutil -m acl...".


CH
  The "acl ch" (or "acl change") command updates access control lists, similar
  in spirit to the Linux chmod command. You can specify multiple access grant
  additions and deletions in a single command run; all changes will be made
  atomically to each object in turn. For example, if the command requests
  deleting one grant and adding a different grant, the ACLs being updated will
  never be left in an intermediate state where one grant has been deleted but
  the second grant not yet added. Each change specifies a user or group grant
  to add or delete, and for grant additions, one of R, W, O (for the
  permission to be granted). A more formal description is provided in a later
  section; below we provide examples.

CH EXAMPLES
  Examples for "ch" sub-command:

  Grant anyone on the internet READ access to the object example-object:

    gsutil acl ch -u AllUsers:R gs://example-bucket/example-object

  NOTE: By default, publicly readable objects are served with a Cache-Control
  header allowing such objects to be cached for 3600 seconds. If you need to
  ensure that updates become visible immediately, you should set a
  Cache-Control header of "Cache-Control:private, max-age=0, no-transform" on
  such objects. For help doing this, see "gsutil help setmeta".

  Grant the user john.doe@example.com READ access to all objects
  in example-bucket that begin with folder/:

    gsutil acl ch -r -u john.doe@example.com:R gs://example-bucket/folder/

  Grant the group admins@example.com OWNER access to all jpg files in
  example-bucket:

    gsutil acl ch -g admins@example.com:O gs://example-bucket/**.jpg

  Grant the owners of project example-project WRITE access to the bucket
  example-bucket:

    gsutil acl ch -p owners-example-project:W gs://example-bucket

  NOTE: You can replace 'owners' with 'viewers' or 'editors' to grant access
  to a project's viewers/editors respectively.

  Remove access to the bucket example-bucket for the viewers of project number
  12345:

    gsutil acl ch -d viewers-12345 gs://example-bucket

  NOTE: You cannot remove the project owners group from ACLs of gs:// buckets in
  the given project. Attempts to do so will appear to succeed, but the service
  will add the project owners group into the new set of ACLs before applying it.

  Note that removing a project requires you to reference the project by
  its number (which you can see with the acl get command) as opposed to its
  project ID string.

  Grant the service account foo@developer.gserviceaccount.com WRITE access to
  the bucket example-bucket:

    gsutil acl ch -u foo@developer.gserviceaccount.com:W gs://example-bucket

  Grant all users from the `G Suite
  <https://www.google.com/work/apps/business/>`_ domain my-domain.org READ
  access to the bucket gcs.my-domain.org:

    gsutil acl ch -g my-domain.org:R gs://gcs.my-domain.org

  Remove any current access by john.doe@example.com from the bucket
  example-bucket:

    gsutil acl ch -d john.doe@example.com gs://example-bucket

  If you have a large number of objects to update, enabling multi-threading
  with the gsutil -m flag can significantly improve performance. The
  following command adds OWNER for admin@example.org using
  multi-threading:

    gsutil -m acl ch -r -u admin@example.org:O gs://example-bucket

  Grant READ access to everyone from my-domain.org and to all authenticated
  users, and grant OWNER to admin@mydomain.org, for the buckets
  my-bucket and my-other-bucket, with multi-threading enabled:

    gsutil -m acl ch -r -g my-domain.org:R -g AllAuth:R \
      -u admin@mydomain.org:O gs://my-bucket/ gs://my-other-bucket

CH ROLES
  You may specify the following roles with either their shorthand or
  their full name:

    R: READ
    W: WRITE
    O: OWNER

  For more information on these roles and the access they grant, see the
  permissions section of the `Access Control Lists page
  <https://cloud.google.com/storage/docs/access-control/lists#permissions>`_.

CH ENTITIES
  There are four different entity types: Users, Groups, All Authenticated Users,
  and All Users.

  Users are added with -u and a plain ID or email address, as in
  "-u john-doe@gmail.com:r". Note: Service Accounts are considered to be users.

  Groups are like users, but specified with the -g flag, as in
  "-g power-users@example.com:O". Groups may also be specified as a full
  domain, as in "-g my-company.com:r".

  AllAuthenticatedUsers and AllUsers are specified directly, as
  in "-g AllUsers:R" or "-g AllAuthenticatedUsers:O". These are case
  insensitive, and may be shortened to "all" and "allauth", respectively.

  Removing roles is specified with the -d flag and an ID, email
  address, domain, or one of AllUsers or AllAuthenticatedUsers.

  Many entities' roles can be specified on the same command line, allowing
  bundled changes to be executed in a single run. This will reduce the number of
  requests made to the server.

CH OPTIONS
  The "ch" sub-command has the following options

  -d          Remove all roles associated with the matching entity.

  -f          Normally gsutil stops at the first error. The -f option causes
              it to continue when it encounters errors. With this option the
              gsutil exit status will be 0 even if some ACLs couldn't be
              changed.

  -g          Add or modify a group entity's role.

  -p          Add or modify a project viewers/editors/owners role.

  -R, -r      Performs acl ch request recursively, to all objects under the
              specified URL.

  -u          Add or modify a user entity's role.
student_01_a0075c49c713@cloudshell:~ (qwiklabs-gcp-02-1087d640b711)$
```



We created a storage bucket, organized it by creating folders and  subfolders, then uploaded objects to it. We also made objects in our  bucket publicly accessible using Cloud Shell.
