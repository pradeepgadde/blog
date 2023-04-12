---


layout: single
title:  "Google Cloud Storage"
date:   2023-04-11 08:59:04 +0530
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

# Cloud Storage

In this lab, you learn how to perform the following tasks:

- Create and use buckets
- Set access control lists to restrict access
- Use your own encryption keys
- Implement version controls
- Use directory synchronization
- Share a bucket across projects using IAM



### **Download a sample file using CURL and make two copies**

```sh
Welcome to Cloud Shell! Type "help" to get started.
Your Cloud Platform project in this session is set to qwiklabs-gcp-00-c5e4bd8c538b.
Use “gcloud config set project [PROJECT_ID]” to change to a different project.
student_01_6caa97ae231a@cloudshell:~ (qwiklabs-gcp-00-c5e4bd8c538b)$ export BUCKET_NAME_1=gaddepradeep
student_01_6caa97ae231a@cloudshell:~ (qwiklabs-gcp-00-c5e4bd8c538b)$ echo $BUCKET_NAME_1
gaddepradeep
student_01_6caa97ae231a@cloudshell:~ (qwiklabs-gcp-00-c5e4bd8c538b)$ curl \
https://hadoop.apache.org/docs/current/\
hadoop-project-dist/hadoop-common/\
ClusterSetup.html > setup.html
  % Total    % Received % Xferd  Average Speed   Time    Time     Time  Current
                                 Dload  Upload   Total   Spent    Left  Speed
100 59422  100 59422    0     0  2001k      0 --:--:-- --:--:-- --:--:-- 2001k
student_01_6caa97ae231a@cloudshell:~ (qwiklabs-gcp-00-c5e4bd8c538b)$ cp setup.html setup2.html
cp setup.html setup3.html
student_01_6caa97ae231a@cloudshell:~ (qwiklabs-gcp-00-c5e4bd8c538b)$ ls
README-cloudshell.txt  setup2.html  setup3.html  setup.html
student_01_6caa97ae231a@cloudshell:~ (qwiklabs-gcp-00-c5e4bd8c538b)$
```

### **Copy the file to the bucket and configure the access control list**
Run the following command to copy the first file to the bucket:
```sh
student_01_6caa97ae231a@cloudshell:~ (qwiklabs-gcp-00-c5e4bd8c538b)$ gsutil cp setup.html gs://$BUCKET_NAME_1/
Copying file://setup.html [Content-Type=text/html]...
/ [1 files][ 58.0 KiB/ 58.0 KiB]
Operation completed over 1 objects/58.0 KiB.
student_01_6caa97ae231a@cloudshell:~ (qwiklabs-gcp-00-c5e4bd8c538b)$

```
To get the default access list that's been assigned to setup.html, run the following command:

```sh
student_01_6caa97ae231a@cloudshell:~ (qwiklabs-gcp-00-c5e4bd8c538b)$ gsutil acl get gs://$BUCKET_NAME_1/setup.html  > acl.txt
cat acl.txt
[
  {
    "entity": "project-owners-724889422008",
    "projectTeam": {
      "projectNumber": "724889422008",
      "team": "owners"
    },
    "role": "OWNER"
  },
  {
    "entity": "project-editors-724889422008",
    "projectTeam": {
      "projectNumber": "724889422008",
      "team": "editors"
    },
    "role": "OWNER"
  },
  {
    "entity": "project-viewers-724889422008",
    "projectTeam": {
      "projectNumber": "724889422008",
      "team": "viewers"
    },
    "role": "READER"
  },
  {
    "email": "student-01-6caa97ae231a@qwiklabs.net",
    "entity": "user-student-01-6caa97ae231a@qwiklabs.net",
    "role": "OWNER"
  }
]
student_01_6caa97ae231a@cloudshell:~ (qwiklabs-gcp-00-c5e4bd8c538b)$
```



To set the access list to private and verify the results, run the following commands:

```sh
student_01_6caa97ae231a@cloudshell:~ (qwiklabs-gcp-00-c5e4bd8c538b)$ gsutil acl set private gs://$BUCKET_NAME_1/setup.html
gsutil acl get gs://$BUCKET_NAME_1/setup.html  > acl2.txt
cat acl2.txt
Setting ACL on gs://gaddepradeep/setup.html...
/ [1 objects]
Operation completed over 1 objects.
[
  {
    "email": "student-01-6caa97ae231a@qwiklabs.net",
    "entity": "user-student-01-6caa97ae231a@qwiklabs.net",
    "role": "OWNER"
  }
]
student_01_6caa97ae231a@cloudshell:~ (qwiklabs-gcp-00-c5e4bd8c538b)$
```

To update the access list to make the file publicly readable, run the following commands:

```sh
student_01_6caa97ae231a@cloudshell:~ (qwiklabs-gcp-00-c5e4bd8c538b)$ gsutil acl ch -u AllUsers:R gs://$BUCKET_NAME_1/setup.html
gsutil acl get gs://$BUCKET_NAME_1/setup.html  > acl3.txt
cat acl3.txt
Updated ACL on gs://gaddepradeep/setup.html
[
  {
    "email": "student-01-6caa97ae231a@qwiklabs.net",
    "entity": "user-student-01-6caa97ae231a@qwiklabs.net",
    "role": "OWNER"
  },
  {
    "entity": "allUsers",
    "role": "READER"
  }
]
student_01_6caa97ae231a@cloudshell:~ (qwiklabs-gcp-00-c5e4bd8c538b)$
```

### **Examine the file in the Cloud Console**

1. In the Cloud Console, on the **Navigation menu** (![Navigation menu icon](https://cdn.qwiklabs.com/tkgw1TDgj4Q%2BYKQUW4jUFd0O5OEKlUMBRYbhlCrF0WY%3D)), click **Cloud Storage** > **Buckets**.
2. Click [BUCKET_NAME_1].
3. Verify that for file setup.html, **Public access** has a **Public link** available.





### **Delete the local file and copy back from Cloud Storage**

```sh
student_01_6caa97ae231a@cloudshell:~ (qwiklabs-gcp-00-c5e4bd8c538b)$
student_01_6caa97ae231a@cloudshell:~ (qwiklabs-gcp-00-c5e4bd8c538b)$ ls
acl2.txt  acl3.txt  acl.txt  README-cloudshell.txt  setup2.html  setup3.html  setup.html
student_01_6caa97ae231a@cloudshell:~ (qwiklabs-gcp-00-c5e4bd8c538b)$ rm setup.html
student_01_6caa97ae231a@cloudshell:~ (qwiklabs-gcp-00-c5e4bd8c538b)$ ls
acl2.txt  acl3.txt  acl.txt  README-cloudshell.txt  setup2.html  setup3.html
student_01_6caa97ae231a@cloudshell:~ (qwiklabs-gcp-00-c5e4bd8c538b)$ gsutil cp gs://$BUCKET_NAME_1/setup.html setup.html
Copying gs://gaddepradeep/setup.html...
/ [1 files][ 58.0 KiB/ 58.0 KiB]
Operation completed over 1 objects/58.0 KiB.
student_01_6caa97ae231a@cloudshell:~ (qwiklabs-gcp-00-c5e4bd8c538b)$ ls
acl2.txt  acl3.txt  acl.txt  README-cloudshell.txt  setup2.html  setup3.html  setup.html
student_01_6caa97ae231a@cloudshell:~ (qwiklabs-gcp-00-c5e4bd8c538b)$
```



## Customer-supplied encryption keys (CSEK)

### **Generate a CSEK key**

For the next step, you need an AES-256 base-64 key.

```python
student_01_6caa97ae231a@cloudshell:~ (qwiklabs-gcp-00-c5e4bd8c538b)$ python3 -c 'import base64; import os; print(base64.encodebytes(os.urandom(32)))'
b'pAIeK1+GFVuL9CjLkUIeuXTboJE6PzqXO2LzqySYM40=\n'
student_01_6caa97ae231a@cloudshell:~ (qwiklabs-gcp-00-c5e4bd8c538b)$
```

### **Modify the boto file**

The encryption controls are contained in a gsutil configuration file named `.boto`.



```sh
student_01_6caa97ae231a@cloudshell:~ (qwiklabs-gcp-00-c5e4bd8c538b)$ ls -al
total 232
drwxr-xr-x 5 student_01_6caa97ae231a student_01_6caa97ae231a  4096 Apr 11 17:21 .
drwxr-xr-x 4 root                    root                     4096 Apr 11 17:11 ..
-rw-r--r-- 1 student_01_6caa97ae231a student_01_6caa97ae231a   144 Apr 11 17:18 acl2.txt
-rw-r--r-- 1 student_01_6caa97ae231a student_01_6caa97ae231a   200 Apr 11 17:19 acl3.txt
-rw-r--r-- 1 student_01_6caa97ae231a student_01_6caa97ae231a   641 Apr 11 17:16 acl.txt
-rw------- 1 student_01_6caa97ae231a student_01_6caa97ae231a   740 Apr 11 17:22 .bash_history
-rw-r--r-- 1 student_01_6caa97ae231a student_01_6caa97ae231a   220 Jan  1  1970 .bash_logout
-rw-r--r-- 1 student_01_6caa97ae231a student_01_6caa97ae231a  3564 Apr  2 07:14 .bashrc
drwxr-xr-x 3 student_01_6caa97ae231a student_01_6caa97ae231a  4096 Apr  2 07:04 .config
drwxr-xr-x 2 student_01_6caa97ae231a student_01_6caa97ae231a  4096 Apr 11 17:11 .docker
drwxr-xr-x 3 student_01_6caa97ae231a student_01_6caa97ae231a  4096 Apr 11 17:21 .gsutil
-rw-r--r-- 1 student_01_6caa97ae231a student_01_6caa97ae231a   807 Jan  1  1970 .profile
-rw-r--r-- 1 student_01_6caa97ae231a student_01_6caa97ae231a   913 Apr 11 17:11 README-cloudshell.txt
-rw-r--r-- 1 student_01_6caa97ae231a student_01_6caa97ae231a 59422 Apr 11 17:15 setup2.html
-rw-r--r-- 1 student_01_6caa97ae231a student_01_6caa97ae231a 59422 Apr 11 17:15 setup3.html
-rw-r--r-- 1 student_01_6caa97ae231a student_01_6caa97ae231a 59422 Apr 11 17:21 setup.html
student_01_6caa97ae231a@cloudshell:~ (qwiklabs-gcp-00-c5e4bd8c538b)$
```



```sh
student_01_6caa97ae231a@cloudshell:~ (qwiklabs-gcp-00-c5e4bd8c538b)$ gsutil version -l
gsutil version: 5.21
checksum: 74f2cab932617e89e1a88b1b346339f7 (OK)
boto version: 2.49.0
python version: 3.9.2 (default, Feb 28 2021, 17:03:44) [GCC 10.2.1 20210110]
OS: Linux 5.15.89+
multiprocessing available: True
using cloud sdk: True
pass cloud sdk credentials to gsutil: True
config path(s): No config found
gsutil path: /usr/lib/google-cloud-sdk/bin/gsutil
compiled crcmod: False
installed via package manager: False
editable install: False
shim enabled: False
student_01_6caa97ae231a@cloudshell:~ (qwiklabs-gcp-00-c5e4bd8c538b)$
```



```sh
student_01_6caa97ae231a@cloudshell:~ (qwiklabs-gcp-00-c5e4bd8c538b)$ gsutil config -n
This command will create a boto config file at
/home/student_01_6caa97ae231a/.boto containing your credentials, based
on your responses to the following questions.

Boto config file "/home/student_01_6caa97ae231a/.boto" created. If you
need to use a proxy to access the Internet please see the instructions
in that file.
student_01_6caa97ae231a@cloudshell:~ (qwiklabs-gcp-00-c5e4bd8c538b)$ ls -la
total 256
drwxr-xr-x 5 student_01_6caa97ae231a student_01_6caa97ae231a  4096 Apr 11 17:25 .
drwxr-xr-x 4 root                    root                     4096 Apr 11 17:11 ..
-rw-r--r-- 1 student_01_6caa97ae231a student_01_6caa97ae231a   144 Apr 11 17:18 acl2.txt
-rw-r--r-- 1 student_01_6caa97ae231a student_01_6caa97ae231a   200 Apr 11 17:19 acl3.txt
-rw-r--r-- 1 student_01_6caa97ae231a student_01_6caa97ae231a   641 Apr 11 17:16 acl.txt
-rw------- 1 student_01_6caa97ae231a student_01_6caa97ae231a   782 Apr 11 17:25 .bash_history
-rw-r--r-- 1 student_01_6caa97ae231a student_01_6caa97ae231a   220 Jan  1  1970 .bash_logout
-rw-r--r-- 1 student_01_6caa97ae231a student_01_6caa97ae231a  3564 Apr  2 07:14 .bashrc
-rw------- 1 student_01_6caa97ae231a student_01_6caa97ae231a 21485 Apr 11 17:25 .boto
drwxr-xr-x 3 student_01_6caa97ae231a student_01_6caa97ae231a  4096 Apr  2 07:04 .config
drwxr-xr-x 2 student_01_6caa97ae231a student_01_6caa97ae231a  4096 Apr 11 17:11 .docker
drwxr-xr-x 3 student_01_6caa97ae231a student_01_6caa97ae231a  4096 Apr 11 17:21 .gsutil
-rw-r--r-- 1 student_01_6caa97ae231a student_01_6caa97ae231a   807 Jan  1  1970 .profile
-rw-r--r-- 1 student_01_6caa97ae231a student_01_6caa97ae231a   913 Apr 11 17:11 README-cloudshell.txt
-rw-r--r-- 1 student_01_6caa97ae231a student_01_6caa97ae231a 59422 Apr 11 17:15 setup2.html
-rw-r--r-- 1 student_01_6caa97ae231a student_01_6caa97ae231a 59422 Apr 11 17:15 setup3.html
-rw-r--r-- 1 student_01_6caa97ae231a student_01_6caa97ae231a 59422 Apr 11 17:21 setup.html
student_01_6caa97ae231a@cloudshell:~ (qwiklabs-gcp-00-c5e4bd8c538b)$
```

### **Upload the remaining setup files (encrypted) and verify in the Cloud Console**

```sh
student_01_6caa97ae231a@cloudshell:~ (qwiklabs-gcp-00-c5e4bd8c538b)$ gsutil cp setup2.html gs://$BUCKET_NAME_1/
gsutil cp setup3.html gs://$BUCKET_NAME_1/
Copying file://setup2.html [Content-Type=text/html]...
- [1 files][ 58.0 KiB/ 58.0 KiB]
Operation completed over 1 objects/58.0 KiB.
Copying file://setup3.html [Content-Type=text/html]...
/ [1 files][ 58.0 KiB/ 58.0 KiB]
Operation completed over 1 objects/58.0 KiB.
student_01_6caa97ae231a@cloudshell:~ (qwiklabs-gcp-00-c5e4bd8c538b)$
```



### **Delete local files, copy new files, and verify encryption**

```sh
student_01_6caa97ae231a@cloudshell:~ (qwiklabs-gcp-00-c5e4bd8c538b)$ rm setup*
student_01_6caa97ae231a@cloudshell:~ (qwiklabs-gcp-00-c5e4bd8c538b)$ ls
acl2.txt  acl3.txt  acl.txt  README-cloudshell.txt
student_01_6caa97ae231a@cloudshell:~ (qwiklabs-gcp-00-c5e4bd8c538b)$ gsutil cp gs://$BUCKET_NAME_1/setup* ./
Copying gs://gaddepradeep/setup.html...
Copying gs://gaddepradeep/setup2.html...
Copying gs://gaddepradeep/setup3.html...
- [3 files][174.1 KiB/174.1 KiB]
Operation completed over 3 objects/174.1 KiB.
student_01_6caa97ae231a@cloudshell:~ (qwiklabs-gcp-00-c5e4bd8c538b)$ ls
acl2.txt  acl3.txt  acl.txt  README-cloudshell.txt  setup2.html  setup3.html  setup.html
student_01_6caa97ae231a@cloudshell:~ (qwiklabs-gcp-00-c5e4bd8c538b)$
```

To cat the encrypted files to see whether they made it back, run the following commands:

```sh
cat setup.html
 cat setup2.html
 cat setup3.html
```



### **Move the current CSEK encrypt key to decrypt key**



### **Generate another CSEK key and add to the boto file**

```python
student_01_6caa97ae231a@cloudshell:~ (qwiklabs-gcp-00-c5e4bd8c538b)$ python3 -c 'import base64; import os; print(base64.encodebytes(os.urandom(32)))'
b'jD9EzOuqojLVBY5ZeuEBGq0OiureRj6Kq02E4lp81cI=\n'
student_01_6caa97ae231a@cloudshell:~ (qwiklabs-gcp-00-c5e4bd8c538b)$
```

### **Rewrite the key for file 1 and comment out the old decrypt key**

When a file is encrypted, rewriting the file decrypts it using the  decryption_key1 that you previously set, and encrypts the file with the  new encryption_key.

You are rewriting the key for setup2.html, but not for setup3.html,  so that you can see what happens if you don't rotate the keys properly.



```sh
student_01_6caa97ae231a@cloudshell:~ (qwiklabs-gcp-00-c5e4bd8c538b)$ gsutil rewrite -k gs://$BUCKET_NAME_1/setup2.html
- [1 files][ 58.0 KiB/ 58.0 KiB]                                                 0.0 B/ 58.0 KiB]
Operation completed over 1 objects/58.0 KiB.
student_01_6caa97ae231a@cloudshell:~ (qwiklabs-gcp-00-c5e4bd8c538b)$
```

### **Download setup 2 and setup3**



```sh
student_01_6caa97ae231a@cloudshell:~ (qwiklabs-gcp-00-c5e4bd8c538b)$ gsutil rewrite -k gs://$BUCKET_NAME_1/setup2.html
- [1 files][ 58.0 KiB/ 58.0 KiB]                                                 0.0 B/ 58.0 KiB]
Operation completed over 1 objects/58.0 KiB.
student_01_6caa97ae231a@cloudshell:~ (qwiklabs-gcp-00-c5e4bd8c538b)$ nano .boto
student_01_6caa97ae231a@cloudshell:~ (qwiklabs-gcp-00-c5e4bd8c538b)$ gsutil cp  gs://$BUCKET_NAME_1/setup2.html recover2.html
Copying gs://gaddepradeep/setup2.html...
/ [1 files][ 58.0 KiB/ 58.0 KiB]
Operation completed over 1 objects/58.0 KiB.
student_01_6caa97ae231a@cloudshell:~ (qwiklabs-gcp-00-c5e4bd8c538b)$ gsutil cp  gs://$BUCKET_NAME_1/setup3.html recover3.html
Traceback (most recent call last):
  File "/usr/lib/google-cloud-sdk/platform/gsutil/gsutil", line 21, in <module>
    gsutil.RunMain()
  File "/usr/lib/google-cloud-sdk/platform/gsutil/gsutil.py", line 151, in RunMain
    sys.exit(gslib.__main__.main())
  File "/usr/lib/google-cloud-sdk/platform/gsutil/gslib/__main__.py", line 436, in main
    return _RunNamedCommandAndHandleExceptions(
  File "/usr/lib/google-cloud-sdk/platform/gsutil/gslib/__main__.py", line 785, in _RunNamedCommandAndHandleExceptions
    _HandleUnknownFailure(e)
  File "/usr/lib/google-cloud-sdk/platform/gsutil/gslib/__main__.py", line 633, in _RunNamedCommandAndHandleExceptions
    return command_runner.RunNamedCommand(command_name,
  File "/usr/lib/google-cloud-sdk/platform/gsutil/gslib/command_runner.py", line 421, in RunNamedCommand
    return_code = command_inst.RunCommand()
  File "/usr/lib/google-cloud-sdk/platform/gsutil/gslib/commands/cp.py", line 1100, in RunCommand
    self.Apply(_CopyFuncWrapper,
  File "/usr/lib/google-cloud-sdk/platform/gsutil/gslib/command.py", line 1575, in Apply
    self._SequentialApply(func, args_iterator, exception_handler, caller_id,
  File "/usr/lib/google-cloud-sdk/platform/gsutil/gslib/command.py", line 1627, in _SequentialApply
    args = next(args_iterator)
  File "/usr/lib/google-cloud-sdk/platform/gsutil/gslib/name_expansion.py", line 687, in __next__
    name_expansion_result = next(self.current_expansion_iter)
  File "/usr/lib/google-cloud-sdk/platform/gsutil/gslib/plurality_checkable_iterator.py", line 96, in __next__
    raise six.reraise(item_tuple[1].__class__, item_tuple[1], item_tuple[2])
  File "/usr/lib/google-cloud-sdk/platform/gsutil/third_party/six/six.py", line 693, in reraise
    raise value
  File "/usr/lib/google-cloud-sdk/platform/gsutil/gslib/plurality_checkable_iterator.py", line 70, in _PopulateHead
    e = next(self.base_iterator)
  File "/usr/lib/google-cloud-sdk/platform/gsutil/gslib/name_expansion.py", line 291, in __iter__
    for (names_container, blr) in post_step3_iter:
  File "/usr/lib/google-cloud-sdk/platform/gsutil/gslib/plurality_checkable_iterator.py", line 96, in __next__
    raise six.reraise(item_tuple[1].__class__, item_tuple[1], item_tuple[2])
  File "/usr/lib/google-cloud-sdk/platform/gsutil/third_party/six/six.py", line 693, in reraise
    raise value
  File "/usr/lib/google-cloud-sdk/platform/gsutil/gslib/plurality_checkable_iterator.py", line 70, in _PopulateHead
    e = next(self.base_iterator)
  File "/usr/lib/google-cloud-sdk/platform/gsutil/gslib/name_expansion.py", line 533, in __iter__
    for (names_container, blr) in self.tuple_iter:
  File "/usr/lib/google-cloud-sdk/platform/gsutil/gslib/plurality_checkable_iterator.py", line 96, in __next__
    raise six.reraise(item_tuple[1].__class__, item_tuple[1], item_tuple[2])
  File "/usr/lib/google-cloud-sdk/platform/gsutil/third_party/six/six.py", line 693, in reraise
    raise value
  File "/usr/lib/google-cloud-sdk/platform/gsutil/gslib/plurality_checkable_iterator.py", line 70, in _PopulateHead
    e = next(self.base_iterator)
  File "/usr/lib/google-cloud-sdk/platform/gsutil/gslib/name_expansion.py", line 497, in __iter__
    for blr in self.blr_iter:
  File "/usr/lib/google-cloud-sdk/platform/gsutil/gslib/plurality_checkable_iterator.py", line 96, in __next__
    raise six.reraise(item_tuple[1].__class__, item_tuple[1], item_tuple[2])
  File "/usr/lib/google-cloud-sdk/platform/gsutil/third_party/six/six.py", line 693, in reraise
    raise value
  File "/usr/lib/google-cloud-sdk/platform/gsutil/gslib/plurality_checkable_iterator.py", line 70, in _PopulateHead
    e = next(self.base_iterator)
  File "/usr/lib/google-cloud-sdk/platform/gsutil/gslib/wildcard_iterator.py", line 538, in IterAll
    for blr in self.__iter__(bucket_listing_fields=bucket_listing_fields,
  File "/usr/lib/google-cloud-sdk/platform/gsutil/gslib/wildcard_iterator.py", line 189, in __iter__
    get_object = self.gsutil_api.GetObjectMetadata(
  File "/usr/lib/google-cloud-sdk/platform/gsutil/gslib/cloud_api_delegator.py", line 319, in GetObjectMetadata
    return self._GetApi(provider).GetObjectMetadata(bucket_name,
  File "/usr/lib/google-cloud-sdk/platform/gsutil/gslib/gcs_json_api.py", line 1058, in GetObjectMetadata
    raise EncryptionException(
gslib.cloud_api.EncryptionException: Missing decryption key with SHA256 hash b'wj/YxEOVmZiHr0fpDYNxOrva00A1iOyPYtN6/eb+wIg='. No decryption key matches object gs://gaddepradeep/setup3.html
student_01_6caa97ae231a@cloudshell:~ (qwiklabs-gcp-00-c5e4bd8c538b)$
```



## 5. Enable lifecycle management

### **View the current lifecycle policy for the bucket**

```sh
student_01_6caa97ae231a@cloudshell:~ (qwiklabs-gcp-00-c5e4bd8c538b)$ gsutil lifecycle get gs://$BUCKET_NAME_1
gs://gaddepradeep/ has no lifecycle configuration.
student_01_6caa97ae231a@cloudshell:~ (qwiklabs-gcp-00-c5e4bd8c538b)$
```



### **Create a JSON lifecycle policy file**

1. To create a file named *life.json*, run the following command:

   ```json
   student_01_6caa97ae231a@cloudshell:~ (qwiklabs-gcp-00-c5e4bd8c538b)$ cat life.json
   {
     "rule":
     [
       {
         "action": {"type": "Delete"},
         "condition": {"age": 31}
       }
     ]
   }
   student_01_6caa97ae231a@cloudshell:~ (qwiklabs-gcp-00-c5e4bd8c538b)$
   ```

### **Set the policy and verify**

1. To set the policy, run the following command:

   ```sh
   student_01_6caa97ae231a@cloudshell:~ (qwiklabs-gcp-00-c5e4bd8c538b)$ gsutil lifecycle set life.json gs://$BUCKET_NAME_1
   Setting lifecycle configuration on gs://gaddepradeep/...
   student_01_6caa97ae231a@cloudshell:~ (qwiklabs-gcp-00-c5e4bd8c538b)$
   ```

```json
student_01_6caa97ae231a@cloudshell:~ (qwiklabs-gcp-00-c5e4bd8c538b)$ gsutil lifecycle get gs://$BUCKET_NAME_1
{"rule": [{"action": {"type": "Delete"}, "condition": {"age": 31}}]}
student_01_6caa97ae231a@cloudshell:~ (qwiklabs-gcp-00-c5e4bd8c538b)$
```

### **View the versioning status for the bucket and enable versioning**

```sh
student_01_6caa97ae231a@cloudshell:~ (qwiklabs-gcp-00-c5e4bd8c538b)$ gsutil versioning get gs://$BUCKET_NAME_1
gs://gaddepradeep: Suspended
student_01_6caa97ae231a@cloudshell:~ (qwiklabs-gcp-00-c5e4bd8c538b)$
```

To enable versioning, run the following command:

```sh
student_01_6caa97ae231a@cloudshell:~ (qwiklabs-gcp-00-c5e4bd8c538b)$ gsutil versioning set on gs://$BUCKET_NAME_1
Enabling versioning for gs://gaddepradeep/...
student_01_6caa97ae231a@cloudshell:~ (qwiklabs-gcp-00-c5e4bd8c538b)$
```



```sh
student_01_6caa97ae231a@cloudshell:~ (qwiklabs-gcp-00-c5e4bd8c538b)$ gsutil versioning get gs://$BUCKET_NAME_1
gs://gaddepradeep: Enabled
student_01_6caa97ae231a@cloudshell:~ (qwiklabs-gcp-00-c5e4bd8c538b)$
```



### **Create several versions of the sample file in the bucket**

```sh
student_01_6caa97ae231a@cloudshell:~ (qwiklabs-gcp-00-c5e4bd8c538b)$ ls -al setup.html
-rw-r--r-- 1 student_01_6caa97ae231a student_01_6caa97ae231a 59422 Apr 11 17:31 setup.html
student_01_6caa97ae231a@cloudshell:~ (qwiklabs-gcp-00-c5e4bd8c538b)$ nano setup.html
student_01_6caa97ae231a@cloudshell:~ (qwiklabs-gcp-00-c5e4bd8c538b)$ ls -al setup.html
-rw-r--r-- 1 student_01_6caa97ae231a student_01_6caa97ae231a 58953 Apr 11 17:47 setup.html
student_01_6caa97ae231a@cloudshell:~ (qwiklabs-gcp-00-c5e4bd8c538b)$
```

```sh
student_01_6caa97ae231a@cloudshell:~ (qwiklabs-gcp-00-c5e4bd8c538b)$ gsutil cp -v setup.html gs://$BUCKET_NAME_1
Copying file://setup.html [Content-Type=text/html]...
Created: gs://gaddepradeep/setup.html#1681235275148256

Operation completed over 1 objects/57.6 KiB.
student_01_6caa97ae231a@cloudshell:~ (qwiklabs-gcp-00-c5e4bd8c538b)$
```

```sh
student_01_6caa97ae231a@cloudshell:~ (qwiklabs-gcp-00-c5e4bd8c538b)$ ls -al setup.html
-rw-r--r-- 1 student_01_6caa97ae231a student_01_6caa97ae231a 58604 Apr 11 17:48 setup.html
student_01_6caa97ae231a@cloudshell:~ (qwiklabs-gcp-00-c5e4bd8c538b)$
```

```sh
student_01_6caa97ae231a@cloudshell:~ (qwiklabs-gcp-00-c5e4bd8c538b)$ gsutil cp -v setup.html gs://$BUCKET_NAME_1
Copying file://setup.html [Content-Type=text/html]...
Created: gs://gaddepradeep/setup.html#1681235354685318

Operation completed over 1 objects/57.2 KiB.
student_01_6caa97ae231a@cloudshell:~ (qwiklabs-gcp-00-c5e4bd8c538b)$
```

### **List all versions of the file**

1. To list all versions of the file, run the following command:

   ```sh
   student_01_6caa97ae231a@cloudshell:~ (qwiklabs-gcp-00-c5e4bd8c538b)$ gsutil ls -a gs://$BUCKET_NAME_1/setup.html
   gs://gaddepradeep/setup.html#1681233374017388
   gs://gaddepradeep/setup.html#1681235275148256
   gs://gaddepradeep/setup.html#1681235354685318
   student_01_6caa97ae231a@cloudshell:~ (qwiklabs-gcp-00-c5e4bd8c538b)$
   ```

   ```sh
   student_01_6caa97ae231a@cloudshell:~ (qwiklabs-gcp-00-c5e4bd8c538b)$ export VERSION_NAME=gs://gaddepradeep/setup.html#1681233374017388
   student_01_6caa97ae231a@cloudshell:~ (qwiklabs-gcp-00-c5e4bd8c538b)$ echo $VERSION_NAME
   gs://gaddepradeep/setup.html#1681233374017388
   student_01_6caa97ae231a@cloudshell:~ (qwiklabs-gcp-00-c5e4bd8c538b)$
   ```

### **Download the oldest, original version of the file and verify recovery**

1. Download the original version of the file:

```sh
student_01_6caa97ae231a@cloudshell:~ (qwiklabs-gcp-00-c5e4bd8c538b)$ gsutil cp $VERSION_NAME recovered.txt
Copying gs://gaddepradeep/setup.html#1681233374017388...
/ [1 files][ 58.0 KiB/ 58.0 KiB]
Operation completed over 1 objects/58.0 KiB.
student_01_6caa97ae231a@cloudshell:~ (qwiklabs-gcp-00-c5e4bd8c538b)$
```

```sh
student_01_6caa97ae231a@cloudshell:~ (qwiklabs-gcp-00-c5e4bd8c538b)$ ls -al setup.html
ls -al recovered.txt
-rw-r--r-- 1 student_01_6caa97ae231a student_01_6caa97ae231a 58604 Apr 11 17:48 setup.html
-rw-r--r-- 1 student_01_6caa97ae231a student_01_6caa97ae231a 59422 Apr 11 17:52 recovered.txt
student_01_6caa97ae231a@cloudshell:~ (qwiklabs-gcp-00-c5e4bd8c538b)$
```



### Make a nested directory and sync with a bucket

Make a nested directory structure so that you can examine what happens when it is recursively copied to a bucket.

```sh
student_01_6caa97ae231a@cloudshell:~ (qwiklabs-gcp-00-c5e4bd8c538b)$ mkdir firstlevel
mkdir ./firstlevel/secondlevel
cp setup.html firstlevel
cp setup.html firstlevel/secondlevel
student_01_6caa97ae231a@cloudshell:~ (qwiklabs-gcp-00-c5e4bd8c538b)$ ls
acl2.txt  acl3.txt  acl.txt  firstlevel  life.json  README-cloudshell.txt  recover2.html  recovered.txt  setup2.html  setup3.html  setup.html
student_01_6caa97ae231a@cloudshell:~ (qwiklabs-gcp-00-c5e4bd8c538b)$
```



```sh
student_01_6caa97ae231a@cloudshell:~ (qwiklabs-gcp-00-c5e4bd8c538b)$ gsutil rsync -r ./firstlevel gs://$BUCKET_NAME_1/firstlevel

WARNING: gsutil rsync uses hashes when modification time is not available at
both the source and destination. Your crcmod installation isn't using the
module's C extension, so checksumming will run very slowly. If this is your
first rsync since updating gsutil, this rsync can take significantly longer than
usual. For help installing the extension, please see "gsutil help crcmod".

Building synchronization state...
Starting synchronization...
Copying file://./firstlevel/secondlevel/setup.html [Content-Type=text/html]...
Copying file://./firstlevel/setup.html [Content-Type=text/html]...
- [2 files][114.5 KiB/114.5 KiB]
Operation completed over 2 objects/114.5 KiB.
student_01_6caa97ae231a@cloudshell:~ (qwiklabs-gcp-00-c5e4bd8c538b)$
```



```sh
student_01_6caa97ae231a@cloudshell:~ (qwiklabs-gcp-00-c5e4bd8c538b)$ gsutil ls -r gs://$BUCKET_NAME_1/firstlevel
gs://gaddepradeep/firstlevel/:
gs://gaddepradeep/firstlevel/setup.html

gs://gaddepradeep/firstlevel/secondlevel/:
gs://gaddepradeep/firstlevel/secondlevel/setup.html
student_01_6caa97ae231a@cloudshell:~ (qwiklabs-gcp-00-c5e4bd8c538b)$
```

## Cross-project sharing

### **Switch to the second project**

1. Open a new incognito tab.
2. Navigate to [console.cloud.google.com](http://console.cloud.google.com) to open a Cloud Console.
3. Click the project selector dropdown in the title bar.
4. Click **All**, and then click the second project  provided for you in the Qwiklabs Connection Details dialog. Remember  that the Project ID is a unique name across all Google Cloud projects.  The second project ID will be referred to as [PROJECT_ID_2].

### **Upload a text file to the bucket**

1. Upload a file to [BUCKET_NAME_2]. Any small example file or text file will do.
2. Note the file name (referred to as [FILE_NAME]); you will use it later.



### **Create an IAM Service Account**



```json
{
  "type": "service_account",
  "project_id": "qwiklabs-gcp-02-3fb348e11e59",
  "private_key_id": "24024affeab3d0674761c21c125af78d78ed287",
  "private_key": "-----BEGIN PRIVATE KEY-----\nMIIEvAIBADANBgkqhkiG9w0QEFAASCBKYwggSiAgEAAoIBAQC88aP3SCbdWvS9\nx1qe7jd2I5P83v8lvX40d58fkEn7NswxvvjlI1kWsUm2xFpYjw+soLQvhu/S68g/\nQ8ARDU1arEdJPvfwOfNRp1LqFUnzgUw3If8kFcyYzzXeqXrsEMh1AnCYDxQDtU3s\nk4cebMheVDscWGAqkTqG+rq+ey4sdOrGF53I0MG83q45BmiiYmqOZmYO2JauIKo9\n2QbelKkKSUeEmMz5cCV9p1y8PqD5D2K3UY1dzV+2a+7S7VgzdBcARXCRXuQoRA/U\nmJobP+OO43ru+i7SusgqXTa70hbH8Qq1n7DHlk6RP39G6K3Z2kSel3Z1fYjGlg7w\nvzj3gru3AgMBAAECggEAAOsbqSjBBHMBEtrC0Ox5jYnY/wLLa+hbLT03S7WjY3kG\nRGE2i5qU7xdUaUOsERC1y2ZAQYqRf5erv+aNbjVDAJlTgxAGr8+UgpZv9pry4q1o\nm8WTIxfYI0w4cEJyKKSxm+bwUW4os1np6X+MtYGLfSa7L/scMxSO8z5TpUIQLcmb\nHenUZUREc4Eif0qAxrLJP0n4HmaqTdhUpiKey/DM8uoE3aGHRRGrj2OglMZ8ploH\nJlEyxTtd3paTXgkIwArjoPq0hXIuzQUoEwdZC1dA36scH//6lfYB0qD+/Kxm6O0L\nlbgbimwQW/jX7Ipgta/7HkvD18Soqlw85xN70j8qmQKBgQD9tVpR6a6vVPZCOKlb\nLve3wYdlDWj/DJavS8koHlItNnuF5IXjG0TS8I+Xl6ONp+m3Z/HBXi5FOI9umORx\njLfOZ/4EN7ezt+XMqldCMQlBmEV26jQFnk/ktxhyv/dD6S81bJ27oRFIdnaA4Di6\nAUFCFIyXqBIWyb8R8xDP8VwgbwKBgQC+poiO2vvkso5D8VkWC1wF37WgoaN5MLtC\ntTwPs9+OZGYzbjpUQAHU3Itei9xFt1dCxYvMqvtKvvqLyk2NMiOEyGXa6zjO6Lx3\nKvkws9A6vOqyPA6Lzdu4HSZ+avOK5Iup6LaAz58w1mtDdXsyquQdk6yQdrBxJcE6\nEZFECnQtOQKBgEiV4BdbYgzro+DiUSGzWFAMYG463fVKZroUVqLRufURh1mRPfTx\n0kj/ZjWavsJCbg15AaOvDFHlkrOzrnHg3LGeMgVQ7otVhTsGzQdCwff89llCd\nMIJhF++MmHOnfUgtoRsTQ9yVd+X7QH+G6GK9elPRVAuNAMtj3UWA6jilAoGAPQ1h\nb7XqsmsHqfRQ3gFXP75LEJySmA2l+g/FoxWsApJeNBwZa79vlrXln6pUKLM0q3pN\ncYZToLUV0MxBF3U18KCoFXn8IC5hpBvL3u/GP/kdg2Q+GEEdGpGjMRqY0SKtIwUV\n5JwYU3BmuxyVDj2xfVM8EKshh6pafXkAtvRODekCgYBz3orYqJsr2YHkMeZSrDcu\n1V4ZIHezJcHOq+KYbgvVE5hLe39MB19kYeXOzOV1pBZ4M532p8XflaukdoemqUpw\nYt76NoWV2qAxKaxpxuWbTlHojGQiiOuphyka3zYb2ZHHMVB8D1+LJmSCES+Y7svX\nXeBwvS+BIx8Kk6gTf8kjvA==\n-----END PRIVATE KEY-----\n",
  "client_email": "cross-project-storage@qwiklabs-gcp-02-3fb348e11e59.iam.gserviceaccount.com",
  "client_id": "104000233834814584403",
  "auth_uri": "https://accounts.google.com/o/oauth2/auth",
  "token_uri": "https://oauth2.googleapis.com/token",
  "auth_provider_x509_cert_url": "https://www.googleapis.com/oauth2/v1/certs",
  "client_x509_cert_url": "https://www.googleapis.com/robot/v1/metadata/x509/cross-project-storage%40qwiklabs-gcp-02-3fb348e11e59.iam.gserviceaccount.com"
}

```

### **Create a VM**



```sh
Linux crossproject 4.19.0-23-cloud-amd64 #1 SMP Debian 4.19.269-1 (2022-12-20) x86_64

The programs included with the Debian GNU/Linux system are free software;
the exact distribution terms for each program are described in the
individual files in /usr/share/doc/*/copyright.

Debian GNU/Linux comes with ABSOLUTELY NO WARRANTY, to the extent
permitted by applicable law.
Creating directory '/home/student-01-6caa97ae231a'.
student-01-6caa97ae231a@crossproject:~$ export BUCKET_NAME_2=pradeepgadde2
student-01-6caa97ae231a@crossproject:~$ echo $BUCKET_NAME_2
pradeepgadde2
student-01-6caa97ae231a@crossproject:~$ 
```



```sh
student-01-6caa97ae231a@crossproject:~$ gsutil ls gs://$BUCKET_NAME_2/
AccessDeniedException: 403 724889422008-compute@developer.gserviceaccount.com does not have storage.objects.list access to the Google Cloud Storage bucket. Permission 'storage.objects.list' denied on resource (or it may not exist).
student-01-6caa97ae231a@crossproject:~$ 

```

### **Authorize the VM**

```sh
student-01-6caa97ae231a@crossproject:~$ ls
credentials.json
student-01-6caa97ae231a@crossproject:~$ 
```

```sh
student-01-6caa97ae231a@crossproject:~$ gcloud auth activate-service-account --key-file credentials.json
Activated service account credentials for: [cross-project-storage@qwiklabs-gcp-02-3fb348e11e59.iam.gserviceaccount.com]
student-01-6caa97ae231a@crossproject:~$ 

```

```sh
student-01-6caa97ae231a@crossproject:~$ gsutil ls gs://$BUCKET_NAME_2/
gs://pradeepgadde2/graph.svg
student-01-6caa97ae231a@crossproject:~$ 
```

```sh
student-01-6caa97ae231a@crossproject:~$ gsutil cat gs://$BUCKET_NAME_2/$FILE_NAME
<?xml version="1.0" encoding="UTF-8" standalone="no"?>
<!DOCTYPE svg PUBLIC "-//W3C//DTD SVG 1.1//EN"
 "http://www.w3.org/Graphics/SVG/1.1/DTD/svg11.dtd">
<!-- Generated by graphviz version 2.43.0 (0)
 -->
<!-- Title: %3 Pages: 1 -->
<svg width="1346pt" height="332pt"
 viewBox="0.00 0.00 1345.61 332.00" xmlns="http://www.w3.org/2000/svg" xmlns:xlink="http://www.w3.org/1999/xlink">
<g id="graph0" class="graph" transform="scale(1 1) rotate(0) translate(4 328)">
<title>%3</title>
<polygon fill="white" stroke="transparent" points="-4,4 -4,-328 1341.61,-328 1341.61,4 -4,4"/>
<!-- [root] google_compute_address.vm_static_ip (expand) -->
<g id="node1" class="node">
<title>[root] google_compute_address.vm_static_ip (expand)</title>
<polygon fill="none" stroke="black" points="377.78,-108 89.78,-108 89.78,-72 377.78,-72 377.78,-108"/>
<text text-anchor="middle" x="233.78" y="-86.3" font-family="Times,serif" font-size="14.00">google_compute_address.vm_static_ip</text>
</g>
<!-- [root] provider[&quot;registry.terraform.io/hashicorp/google&quot;] -->
<g id="node5" class="node">
<title>[root] provider[&quot;registry.terraform.io/hashicorp/google&quot;]</title>
<polygon fill="none" stroke="black" points="632.78,-36 289.25,-18 632.78,0 976.31,-18 632.78,-36"/>
<text text-anchor="middle" x="632.78" y="-14.3" font-family="Times,serif" font-size="14.00">provider[&quot;registry.terraform.io/hashicorp/google&quot;]</text>
</g>
<!-- [root] google_compute_address.vm_static_ip (expand)&#45;&gt;[root] provider[&quot;registry.terraform.io/hashicorp/google&quot;] -->
<g id="edge1" class="edge">
<title>[root] google_compute_address.vm_static_ip (expand)&#45;&gt;[root] provider[&quot;registry.terraform.io/hashicorp/google&quot;]</title>
<path fill="none" stroke="black" d="M330.88,-71.97C397.23,-60.33 484.3,-45.05 547.88,-33.9"/>
<polygon fill="black" stroke="black" points="548.63,-37.32 557.88,-32.14 547.42,-30.42 548.63,-37.32"/>
</g>
<!-- [root] google_compute_instance.another_instance (expand) -->
<g id="node2" class="node">
<title>[root] google_compute_instance.another_instance (expand)</title>
<polygon fill="none" stroke="black" points="1195.78,-180 869.78,-180 869.78,-144 1195.78,-144 1195.78,-180"/>
<text text-anchor="middle" x="1032.78" y="-158.3" font-family="Times,serif" font-size="14.00">google_compute_instance.another_instance</text>
</g>
<!-- [root] google_storage_bucket.example_bucket (expand) -->
<g id="node4" class="node">
<title>[root] google_storage_bucket.example_bucket (expand)</title>
<polygon fill="none" stroke="black" points="1182.28,-108 883.28,-108 883.28,-72 1182.28,-72 1182.28,-108"/>
<text text-anchor="middle" x="1032.78" y="-86.3" font-family="Times,serif" font-size="14.00">google_storage_bucket.example_bucket</text>
</g>
<!-- [root] google_compute_instance.another_instance (expand)&#45;&gt;[root] google_storage_bucket.example_bucket (expand) -->
<g id="edge2" class="edge">
<title>[root] google_compute_instance.another_instance (expand)&#45;&gt;[root] google_storage_bucket.example_bucket (expand)</title>
<path fill="none" stroke="black" d="M1032.78,-143.7C1032.78,-135.98 1032.78,-126.71 1032.78,-118.11"/>
<polygon fill="black" stroke="black" points="1036.28,-118.1 1032.78,-108.1 1029.28,-118.1 1036.28,-118.1"/>
</g>
<!-- [root] google_compute_instance.vm_instance (expand) -->
<g id="node3" class="node">
<title>[root] google_compute_instance.vm_instance (expand)</title>
<polygon fill="none" stroke="black" points="686.78,-180 394.78,-180 394.78,-144 686.78,-144 686.78,-180"/>
<text text-anchor="middle" x="540.78" y="-158.3" font-family="Times,serif" font-size="14.00">google_compute_instance.vm_instance</text>
</g>
<!-- [root] google_compute_instance.vm_instance (expand)&#45;&gt;[root] google_compute_address.vm_static_ip (expand) -->
<g id="edge3" class="edge">
<title>[root] google_compute_instance.vm_instance (expand)&#45;&gt;[root] google_compute_address.vm_static_ip (expand)</title>
<path fill="none" stroke="black" d="M466.08,-143.97C421.51,-133.8 364.8,-120.87 318.56,-110.33"/>
<polygon fill="black" stroke="black" points="319.09,-106.86 308.56,-108.05 317.53,-113.69 319.09,-106.86"/>
</g>
<!-- [root] var.instance_name -->
<g id="node6" class="node">
<title>[root] var.instance_name</title>
<polygon fill="none" stroke="black" points="539.28,-108 396.28,-108 396.28,-72 545.28,-72 545.28,-102 539.28,-108"/>
<polyline fill="none" stroke="black" points="539.28,-108 539.28,-102 "/>
<polyline fill="none" stroke="black" points="545.28,-102 539.28,-102 "/>
<text text-anchor="middle" x="470.78" y="-86.3" font-family="Times,serif" font-size="14.00">var.instance_name</text>
</g>
<!-- [root] google_compute_instance.vm_instance (expand)&#45;&gt;[root] var.instance_name -->
<g id="edge4" class="edge">
<title>[root] google_compute_instance.vm_instance (expand)&#45;&gt;[root] var.instance_name</title>
<path fill="none" stroke="black" d="M523.48,-143.7C514.92,-135.14 504.44,-124.66 495.09,-115.3"/>
<polygon fill="black" stroke="black" points="497.43,-112.7 487.89,-108.1 492.48,-117.65 497.43,-112.7"/>
</g>
<!-- [root] var.instance_type -->
<g id="node7" class="node">
<title>[root] var.instance_type</title>
<polygon fill="none" stroke="black" points="698.28,-108 563.28,-108 563.28,-72 704.28,-72 704.28,-102 698.28,-108"/>
<polyline fill="none" stroke="black" points="698.28,-108 698.28,-102 "/>
<polyline fill="none" stroke="black" points="704.28,-102 698.28,-102 "/>
<text text-anchor="middle" x="633.78" y="-86.3" font-family="Times,serif" font-size="14.00">var.instance_type</text>
</g>
<!-- [root] google_compute_instance.vm_instance (expand)&#45;&gt;[root] var.instance_type -->
<g id="edge5" class="edge">
<title>[root] google_compute_instance.vm_instance (expand)&#45;&gt;[root] var.instance_type</title>
<path fill="none" stroke="black" d="M563.77,-143.7C575.6,-134.8 590.18,-123.82 602.96,-114.2"/>
<polygon fill="black" stroke="black" points="605.18,-116.91 611.06,-108.1 600.97,-111.32 605.18,-116.91"/>
</g>
<!-- [root] var.instance_zone -->
<g id="node8" class="node">
<title>[root] var.instance_zone</title>
<polygon fill="none" stroke="black" points="859.28,-108 722.28,-108 722.28,-72 865.28,-72 865.28,-102 859.28,-108"/>
<polyline fill="none" stroke="black" points="859.28,-108 859.28,-102 "/>
<polyline fill="none" stroke="black" points="865.28,-102 859.28,-102 "/>
<text text-anchor="middle" x="793.78" y="-86.3" font-family="Times,serif" font-size="14.00">var.instance_zone</text>
</g>
<!-- [root] google_compute_instance.vm_instance (expand)&#45;&gt;[root] var.instance_zone -->
<g id="edge6" class="edge">
<title>[root] google_compute_instance.vm_instance (expand)&#45;&gt;[root] var.instance_zone</title>
<path fill="none" stroke="black" d="M602.35,-143.97C638.47,-133.97 684.28,-121.3 722.03,-110.85"/>
<polygon fill="black" stroke="black" points="723.24,-114.15 731.94,-108.11 721.37,-107.4 723.24,-114.15"/>
</g>
<!-- [root] google_storage_bucket.example_bucket (expand)&#45;&gt;[root] provider[&quot;registry.terraform.io/hashicorp/google&quot;] -->
<g id="edge7" class="edge">
<title>[root] google_storage_bucket.example_bucket (expand)&#45;&gt;[root] provider[&quot;registry.terraform.io/hashicorp/google&quot;]</title>
<path fill="none" stroke="black" d="M935.44,-71.97C868.82,-60.31 781.35,-45 717.59,-33.84"/>
<polygon fill="black" stroke="black" points="718.02,-30.36 707.56,-32.09 716.81,-37.26 718.02,-30.36"/>
</g>
<!-- [root] output.instance_link (expand) -->
<g id="node9" class="node">
<title>[root] output.instance_link (expand)</title>
<ellipse fill="none" stroke="black" cx="176.78" cy="-234" rx="176.57" ry="18"/>
<text text-anchor="middle" x="176.78" y="-230.3" font-family="Times,serif" font-size="14.00">[root] output.instance_link (expand)</text>
</g>
<!-- [root] output.instance_link (expand)&#45;&gt;[root] google_compute_instance.vm_instance (expand) -->
<g id="edge8" class="edge">
<title>[root] output.instance_link (expand)&#45;&gt;[root] google_compute_instance.vm_instance (expand)</title>
<path fill="none" stroke="black" d="M255.77,-217.81C310.52,-207.28 383.82,-193.19 442.37,-181.93"/>
<polygon fill="black" stroke="black" points="443.22,-185.33 452.38,-180 441.9,-178.45 443.22,-185.33"/>
</g>
<!-- [root] output.network_IP (expand) -->
<g id="node10" class="node">
<title>[root] output.network_IP (expand)</title>
<ellipse fill="none" stroke="black" cx="540.78" cy="-234" rx="168.97" ry="18"/>
<text text-anchor="middle" x="540.78" y="-230.3" font-family="Times,serif" font-size="14.00">[root] output.network_IP (expand)</text>
</g>
<!-- [root] output.network_IP (expand)&#45;&gt;[root] google_compute_instance.vm_instance (expand) -->
<g id="edge9" class="edge">
<title>[root] output.network_IP (expand)&#45;&gt;[root] google_compute_instance.vm_instance (expand)</title>
<path fill="none" stroke="black" d="M540.78,-215.7C540.78,-207.98 540.78,-198.71 540.78,-190.11"/>
<polygon fill="black" stroke="black" points="544.28,-190.1 540.78,-180.1 537.28,-190.1 544.28,-190.1"/>
</g>
<!-- [root] provider[&quot;registry.terraform.io/hashicorp/google&quot;] (close) -->
<g id="node11" class="node">
<title>[root] provider[&quot;registry.terraform.io/hashicorp/google&quot;] (close)</title>
<ellipse fill="none" stroke="black" cx="1032.78" cy="-234" rx="304.65" ry="18"/>
<text text-anchor="middle" x="1032.78" y="-230.3" font-family="Times,serif" font-size="14.00">[root] provider[&quot;registry.terraform.io/hashicorp/google&quot;] (close)</text>
</g>
<!-- [root] provider[&quot;registry.terraform.io/hashicorp/google&quot;] (close)&#45;&gt;[root] google_compute_instance.another_instance (expand) -->
<g id="edge10" class="edge">
<title>[root] provider[&quot;registry.terraform.io/hashicorp/google&quot;] (close)&#45;&gt;[root] google_compute_instance.another_instance (expand)</title>
<path fill="none" stroke="black" d="M1032.78,-215.7C1032.78,-207.98 1032.78,-198.71 1032.78,-190.11"/>
<polygon fill="black" stroke="black" points="1036.28,-190.1 1032.78,-180.1 1029.28,-190.1 1036.28,-190.1"/>
</g>
<!-- [root] provider[&quot;registry.terraform.io/hashicorp/google&quot;] (close)&#45;&gt;[root] google_compute_instance.vm_instance (expand) -->
<g id="edge11" class="edge">
<title>[root] provider[&quot;registry.terraform.io/hashicorp/google&quot;] (close)&#45;&gt;[root] google_compute_instance.vm_instance (expand)</title>
<path fill="none" stroke="black" d="M921.76,-217.2C847.15,-206.59 748.55,-192.56 670.36,-181.43"/>
<polygon fill="black" stroke="black" points="670.83,-177.97 660.44,-180.02 669.84,-184.9 670.83,-177.97"/>
</g>
<!-- [root] root -->
<g id="node12" class="node">
<title>[root] root</title>
<ellipse fill="none" stroke="black" cx="540.78" cy="-306" rx="58.49" ry="18"/>
<text text-anchor="middle" x="540.78" y="-302.3" font-family="Times,serif" font-size="14.00">[root] root</text>
</g>
<!-- [root] root&#45;&gt;[root] output.instance_link (expand) -->
<g id="edge12" class="edge">
<title>[root] root&#45;&gt;[root] output.instance_link (expand)</title>
<path fill="none" stroke="black" d="M492.45,-295.7C434.73,-284.6 337.04,-265.82 265.59,-252.08"/>
<polygon fill="black" stroke="black" points="266.17,-248.63 255.69,-250.17 264.85,-255.5 266.17,-248.63"/>
</g>
<!-- [root] root&#45;&gt;[root] output.network_IP (expand) -->
<g id="edge13" class="edge">
<title>[root] root&#45;&gt;[root] output.network_IP (expand)</title>
<path fill="none" stroke="black" d="M540.78,-287.7C540.78,-279.98 540.78,-270.71 540.78,-262.11"/>
<polygon fill="black" stroke="black" points="544.28,-262.1 540.78,-252.1 537.28,-262.1 544.28,-262.1"/>
</g>
<!-- [root] root&#45;&gt;[root] provider[&quot;registry.terraform.io/hashicorp/google&quot;] (close) -->
<g id="edge14" class="edge">
<title>[root] root&#45;&gt;[root] provider[&quot;registry.terraform.io/hashicorp/google&quot;] (close)</title>
<path fill="none" stroke="black" d="M592.97,-297.58C668.24,-286.87 809.45,-266.78 911.59,-252.24"/>
<polygon fill="black" stroke="black" points="912.26,-255.68 921.66,-250.81 911.27,-248.75 912.26,-255.68"/>
</g>
</g>
</svg>
student-01-6caa97ae231a@crossproject:~$ 
```



```sh
student-01-6caa97ae231a@crossproject:~$ gsutil cp credentials.json gs://$BUCKET_NAME_2/
Copying file://credentials.json [Content-Type=application/json]...
/ [1 files][  2.3 KiB/  2.3 KiB]                                                
Operation completed over 1 objects/2.3 KiB.                                      
student-01-6caa97ae231a@crossproject:~$ 
```



![]({{ site.url }}{{ site.baseurl }}/assets/images/gs-1.png)

![]({{ site.url }}{{ site.baseurl }}/assets/images/gs-2.png)

![]({{ site.url }}{{ site.baseurl }}/assets/images/gs-3.png)

![]({{ site.url }}{{ site.baseurl }}/assets/images/gs-4.png)

![]({{ site.url }}{{ site.baseurl }}/assets/images/gs-5.png)

![]({{ site.url }}{{ site.baseurl }}/assets/images/gs-6.png)

![]({{ site.url }}{{ site.baseurl }}/assets/images/gs-7.png)

![]({{ site.url }}{{ site.baseurl }}/assets/images/gs-8.png)

![]({{ site.url }}{{ site.baseurl }}/assets/images/gs-9.png)

![]({{ site.url }}{{ site.baseurl }}/assets/images/gs-10.png)

![]({{ site.url }}{{ site.baseurl }}/assets/images/gs-11.png)







