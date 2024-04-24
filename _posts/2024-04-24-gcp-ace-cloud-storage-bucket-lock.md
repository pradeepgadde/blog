---
layout: single
title:  "Google Cloud Storage - Bucket Lock"
categories: Cloud
tags: GCP
classes: wide
show_date: true
header:
  teaser: /assets/images/ace-gcp.png
author:
  name     : "Associate Cloud Engineer"
  avatar   : "/assets/images/ace-gcp.png"

sidebar:
  - title: "Topics"
    nav: my-sidebar

---

# Google Cloud Storage - Bucket Lock 
Cloud Storage Bucket Lock feature  allows you to configure a data retention policy for a Cloud Storage bucket that governs how long objects in the bucket must be retained. The feature also allows you to lock the data retention policy, permanently preventing the policy from being reduced or removed.

Create a new bucket

Define an environment variable named Cloud Storage_BUCKET and use your Project ID as the bucket name. Use the following command which uses the Cloud SDK to get your Project ID:

```sh
Welcome to Cloud Shell! Type "help" to get started.
Your Cloud Platform project in this session is set to qwiklabs-gcp-02-01f28432cf1c.
Use “gcloud config set project [PROJECT_ID]” to change to a different project.
student_01_9b51dda9fd9c@cloudshell:~ (qwiklabs-gcp-02-01f28432cf1c)$ 
student_01_9b51dda9fd9c@cloudshell:~ (qwiklabs-gcp-02-01f28432cf1c)$ export BUCKET=$(gcloud config get-value project)
Your active configuration is: [cloudshell-15560]
student_01_9b51dda9fd9c@cloudshell:~ (qwiklabs-gcp-02-01f28432cf1c)$ gsutil mb "gs://$BUCKET"
Creating gs://qwiklabs-gcp-02-01f28432cf1c/...
student_01_9b51dda9fd9c@cloudshell:~ (qwiklabs-gcp-02-01f28432cf1c)$ 
```

Define a Retention Policy

Consider a financial institution branch with a SEC Rule 17a-4 requirement to retain financial transaction records for 6 years (10 seconds for this lab). The Branch IT Administrator, a technical administrator responsible for day-to-day records management and retention within the branch, wants to create a Cloud Storage bucket with a 10 second Retention Policy to help with this requirement.

Look at how to set up a Retention Policy on a bucket.

You can define the Retention Policy using seconds, days, months, and years with the Cloud Storage gsutil tool. As an example, create a Retention Policy for 10 seconds:

> Note: You can also use 10d for 10 days, 10m for 10 months or 10y for 10 years. To learn more, use the command: gsutil help retention set. 

```sh
student_01_9b51dda9fd9c@cloudshell:~ (qwiklabs-gcp-02-01f28432cf1c)$ gsutil retention set 10s "gs://$BUCKET"
Setting Retention Policy on gs://qwiklabs-gcp-02-01f28432cf1c/...
student_01_9b51dda9fd9c@cloudshell:~ (qwiklabs-gcp-02-01f28432cf1c)$ 
```

Verify the Retention Policy for a bucket:

```sh
student_01_9b51dda9fd9c@cloudshell:~ (qwiklabs-gcp-02-01f28432cf1c)$ gsutil retention get "gs://$BUCKET"
  Retention Policy (UNLOCKED):
    Duration: 10 Second(s)
    Effective Time: Wed, 24 Apr 2024 17:42:00 GMT
student_01_9b51dda9fd9c@cloudshell:~ (qwiklabs-gcp-02-01f28432cf1c)$ 
```

The `Effective Time` defines when the policy took effect on the bucket.

1. Now that the bucket has a Retention Policy, add a transaction record object to test it:

   ```sh
   student_01_9b51dda9fd9c@cloudshell:~ (qwiklabs-gcp-02-01f28432cf1c)$ gsutil cp gs://spls/gsp297/dummy_transactions "gs://$BUCKET/"
   Copying gs://spls/gsp297/dummy_transactions [Content-Type=text/plain]...
   / [1 files][   45.0 B/   45.0 B]                                                
   Operation completed over 1 objects/45.0 B.                                       
   student_01_9b51dda9fd9c@cloudshell:~ (qwiklabs-gcp-02-01f28432cf1c)$ 
   ```

   Review the retention expiration:

   ```sh
   student_01_9b51dda9fd9c@cloudshell:~ (qwiklabs-gcp-02-01f28432cf1c)$ gsutil ls -L "gs://$BUCKET/dummy_transactions"
   gs://qwiklabs-gcp-02-01f28432cf1c/dummy_transactions:
       Creation time:          Wed, 24 Apr 2024 17:42:24 GMT
       Update time:            Wed, 24 Apr 2024 17:42:24 GMT
       Storage class:          STANDARD
       Retention Expiration:   Wed, 24 Apr 2024 17:42:34 GMT
       Content-Language:       en
       Content-Length:         45
       Content-Type:           text/plain
       Hash (crc32c):          4PdEyA==
       Hash (md5):             loAwRzgwWU+JCSjGGildQQ==
       ETag:                   CP2uiYyz24UDEAE=
       Generation:             1713980544538493
       Metageneration:         1
       ACL:                    [
     {
       "entity": "project-owners-869650039119",
       "projectTeam": {
         "projectNumber": "869650039119",
         "team": "owners"
       },
       "role": "OWNER"
     },
     {
       "entity": "project-editors-869650039119",
       "projectTeam": {
         "projectNumber": "869650039119",
         "team": "editors"
       },
       "role": "OWNER"
     },
     {
       "entity": "project-viewers-869650039119",
       "projectTeam": {
         "projectNumber": "869650039119",
         "team": "viewers"
       },
       "role": "READER"
     },
     {
       "email": "student-01-9b51dda9fd9c@qwiklabs.net",
       "entity": "user-student-01-9b51dda9fd9c@qwiklabs.net",
       "role": "OWNER"
     }
   ]
   TOTAL: 1 objects, 45 bytes (45 B)
   student_01_9b51dda9fd9c@cloudshell:~ (qwiklabs-gcp-02-01f28432cf1c)$ 
   ```

   When the Retention Policy expires for the given object, it can then be deleted.

   To extend a Retention Policy, use the `gsutil retention set` command to update the Retention Policy.


Lock the Retention Policy

While unlocked, you can remove the Retention Policy from the bucket or reduce the retention time. After you lock the Retention Policy, it cannot be removed from the bucket or the retention time reduced.

Lock the Retention Policy:

```sh
student_01_9b51dda9fd9c@cloudshell:~ (qwiklabs-gcp-02-01f28432cf1c)$ gsutil retention lock "gs://$BUCKET/"
This will PERMANENTLY set the Retention Policy on gs://qwiklabs-gcp-02-01f28432cf1c to:

  Retention Policy (UNLOCKED):
    Duration: 10 Second(s)
    Effective Time: Wed, 24 Apr 2024 17:42:00 GMT

This setting cannot be reverted!  Continue? [y|N]: y
Locking Retention Policy on gs://qwiklabs-gcp-02-01f28432cf1c/...
student_01_9b51dda9fd9c@cloudshell:~ (qwiklabs-gcp-02-01f28432cf1c)$ 
```

```sh
student_01_9b51dda9fd9c@cloudshell:~ (qwiklabs-gcp-02-01f28432cf1c)$ gsutil retention get "gs://$BUCKET/"
  Retention Policy (LOCKED):
    Duration: 10 Second(s)
    Effective Time: Wed, 24 Apr 2024 17:42:00 GMT
student_01_9b51dda9fd9c@cloudshell:~ (qwiklabs-gcp-02-01f28432cf1c)$ 
```

Once locked the Retention Policy can't be unlocked and can only be  extended. The Effective Time is updated if the amount of time since it  was set or last updated has exceeded the Retention Policy.



Temporary hold

Continuing the example above, consider the bucket already configured with a 10 second locked Retention Policy.

Financial regulators decide to perform an audit of one of the branch's customers, and require that those records are retained for the duration of the audit. Some of the Cloud Storage objects for this customer are close to their expiration date, and will soon be automatically deleted.

To handle this, when regulatory investigation begins, the Branch IT Administrator sets the temporary hold flag for each of the objects related to the audit. While that flag is set, the objects will continue to be protected from deletion, even if they are older than 10 seconds.



```sh
student_01_9b51dda9fd9c@cloudshell:~ (qwiklabs-gcp-02-01f28432cf1c)$ gsutil retention temp set "gs://$BUCKET/dummy_transactions"
Setting Temporary Hold on gs://qwiklabs-gcp-02-01f28432cf1c/dummy_transactions...
/ [1 objects]                                                                   
Operation completed over 1 objects.                                              
student_01_9b51dda9fd9c@cloudshell:~ (qwiklabs-gcp-02-01f28432cf1c)$ 
```

By placing a temporary hold on the object, delete operations are not  possible unless the object is released from the hold. As an example,  attempt to delete the object:

```sh
student_01_9b51dda9fd9c@cloudshell:~ (qwiklabs-gcp-02-01f28432cf1c)$ gsutil rm "gs://$BUCKET/dummy_transactions"
Removing gs://qwiklabs-gcp-02-01f28432cf1c/dummy_transactions...
AccessDeniedException: 403 Object 'qwiklabs-gcp-02-01f28432cf1c/dummy_transactions' is under active Temporary hold and cannot be deleted, overwritten or archived until hold is removed.
student_01_9b51dda9fd9c@cloudshell:~ (qwiklabs-gcp-02-01f28432cf1c)$ 
```

Once regulators conclude their audit, the Branch IT Administrator  removes the temporary hold. Use the following command to release the  hold:

```sh
student_01_9b51dda9fd9c@cloudshell:~ (qwiklabs-gcp-02-01f28432cf1c)$ gsutil retention temp release "gs://$BUCKET/dummy_transactions"
Releasing Temporary Hold on gs://qwiklabs-gcp-02-01f28432cf1c/dummy_transactions...
/ [1 objects]                                                                   
Operation completed over 1 objects.                                              
student_01_9b51dda9fd9c@cloudshell:~ (qwiklabs-gcp-02-01f28432cf1c)$ 
```

Now you can delete the file unless the Retention Policy for the file hasn't expired. Otherwise wait a few moments and try again.

```sh
student_01_9b51dda9fd9c@cloudshell:~ (qwiklabs-gcp-02-01f28432cf1c)$ gsutil rm "gs://$BUCKET/dummy_transactions"
Removing gs://qwiklabs-gcp-02-01f28432cf1c/dummy_transactions...
/ [1 objects]                                                                   
Operation completed over 1 objects.                                              
student_01_9b51dda9fd9c@cloudshell:~ (qwiklabs-gcp-02-01f28432cf1c)$ 
```



Event-based holds

Continuing the example above, consider the bucket already configured with a 10 second locked Retention Policy.

In addition to retaining financial transaction records for 10 seconds, the branch also needs to retain loan records for 10 seconds starting from the date that the loan is paid off. To accomplish this, the Branch IT Administrator uploads loan records as new Cloud Storage objects with the event-based hold flag set.

Each day, as the loans are paid off, the Branch IT Administrator unsets the event-based hold flag on the corresponding Cloud Storage objects. No further action is necessary, since Cloud Storage automatically calculates a new Retention Policy expiring 10 seconds from that day.

Event-based holds allow you to delay a Retention Policy from counting down until the hold is released. Event-based holds can be managed per object and set by default in bucket properties when new objects are added to a bucket.



Enable the default event-based hold for your bucket using the following command:

```sh
student_01_9b51dda9fd9c@cloudshell:~ (qwiklabs-gcp-02-01f28432cf1c)$ gsutil retention event-default set "gs://$BUCKET/"
Setting default Event-Based Hold on gs://qwiklabs-gcp-02-01f28432cf1c/...
student_01_9b51dda9fd9c@cloudshell:~ (qwiklabs-gcp-02-01f28432cf1c)$ 
```

Add an example loan into the bucket using the following command:

```sh
student_01_9b51dda9fd9c@cloudshell:~ (qwiklabs-gcp-02-01f28432cf1c)$ gsutil cp gs://spls/gsp297/dummy_loan "gs://$BUCKET/"
Copying gs://spls/gsp297/dummy_loan [Content-Type=text/plain]...
/ [1 files][   10.0 B/   10.0 B]                                                
Operation completed over 1 objects/10.0 B.                                       
student_01_9b51dda9fd9c@cloudshell:~ (qwiklabs-gcp-02-01f28432cf1c)$ 
```

Verify that the event-based hold is enabled for your newly added loan using the following command:

```sh
student_01_9b51dda9fd9c@cloudshell:~ (qwiklabs-gcp-02-01f28432cf1c)$ gsutil ls -L "gs://$BUCKET/dummy_loan"
gs://qwiklabs-gcp-02-01f28432cf1c/dummy_loan:
    Creation time:          Wed, 24 Apr 2024 17:49:05 GMT
    Update time:            Wed, 24 Apr 2024 17:49:05 GMT
    Storage class:          STANDARD
    Event-Based Hold:       Enabled
    Content-Length:         10
    Content-Type:           text/plain
    Hash (crc32c):          rQpq8Q==
    Hash (md5):             /a7B2PeMXZYfqgfLEqYGHg==
    ETag:                   CMaChcu024UDEAE=
    Generation:             1713980945023302
    Metageneration:         1
    ACL:                    [
  {
    "entity": "project-owners-869650039119",
    "projectTeam": {
      "projectNumber": "869650039119",
      "team": "owners"
    },
    "role": "OWNER"
  },
  {
    "entity": "project-editors-869650039119",
    "projectTeam": {
      "projectNumber": "869650039119",
      "team": "editors"
    },
    "role": "OWNER"
  },
  {
    "entity": "project-viewers-869650039119",
    "projectTeam": {
      "projectNumber": "869650039119",
      "team": "viewers"
    },
    "role": "READER"
  },
  {
    "email": "student-01-9b51dda9fd9c@qwiklabs.net",
    "entity": "user-student-01-9b51dda9fd9c@qwiklabs.net",
    "role": "OWNER"
  }
]
TOTAL: 1 objects, 10 bytes (10 B)
student_01_9b51dda9fd9c@cloudshell:~ (qwiklabs-gcp-02-01f28432cf1c)$ 
```

Notice that Retention Expiration isn't defined. The Retention Expiration time is not available until the Event-Based hold is released on the  object.

When the loan is paid off, the Branch IT Administrator then releases the event-based hold using the following command:

```sh
student_01_9b51dda9fd9c@cloudshell:~ (qwiklabs-gcp-02-01f28432cf1c)$ gsutil retention event release "gs://$BUCKET/dummy_loan"
Releasing Event-Based Hold on gs://qwiklabs-gcp-02-01f28432cf1c/dummy_loan...
/ [1 objects]                                                                   
Operation completed over 1 objects.                                              
student_01_9b51dda9fd9c@cloudshell:~ (qwiklabs-gcp-02-01f28432cf1c)$ 
```

After an event-based hold is released, the bucket Retention Policy takes effect. Verify that the example loan now has a Retention Expiration  field using the following command:

```sh
student_01_9b51dda9fd9c@cloudshell:~ (qwiklabs-gcp-02-01f28432cf1c)$ gsutil ls -L "gs://$BUCKET/dummy_loan"
gs://qwiklabs-gcp-02-01f28432cf1c/dummy_loan:
    Creation time:          Wed, 24 Apr 2024 17:49:05 GMT
    Update time:            Wed, 24 Apr 2024 17:50:25 GMT
    Storage class:          STANDARD
    Retention Expiration:   Wed, 24 Apr 2024 17:50:35 GMT
    Content-Length:         10
    Content-Type:           text/plain
    Hash (crc32c):          rQpq8Q==
    Hash (md5):             /a7B2PeMXZYfqgfLEqYGHg==
    ETag:                   CMaChcu024UDEAI=
    Generation:             1713980945023302
    Metageneration:         2
    ACL:                    [
  {
    "entity": "project-owners-869650039119",
    "projectTeam": {
      "projectNumber": "869650039119",
      "team": "owners"
    },
    "role": "OWNER"
  },
  {
    "entity": "project-editors-869650039119",
    "projectTeam": {
      "projectNumber": "869650039119",
      "team": "editors"
    },
    "role": "OWNER"
  },
  {
    "entity": "project-viewers-869650039119",
    "projectTeam": {
      "projectNumber": "869650039119",
      "team": "viewers"
    },
    "role": "READER"
  },
  {
    "email": "student-01-9b51dda9fd9c@qwiklabs.net",
    "entity": "user-student-01-9b51dda9fd9c@qwiklabs.net",
    "role": "OWNER"
  }
]
TOTAL: 1 objects, 10 bytes (10 B)
student_01_9b51dda9fd9c@cloudshell:~ (qwiklabs-gcp-02-01f28432cf1c)$ 
```

> You can also set an event-based hold for a specific object by using the following command:

 ```
gsutil retention event set "gs://bucket-name/object-name"
 ```

How to remove a Retention Policy

Unfortunately, the branch shuts down its lending operations. The Branch IT Administrator still needs to maintain the existing records for their full duration, but no longer expects to produce records in the bucket. After the last loan Retention Period has expired and no longer subject to a hold, the Branch IT Administrator can then delete the empty bucket. The bucket can be deleted even though it has a locked Retention Policy because it contains no data subject to retention.

Delete an empty bucket using the following command:

```sh
student_01_9b51dda9fd9c@cloudshell:~ (qwiklabs-gcp-02-01f28432cf1c)$ gsutil rb "gs://$BUCKET/"
Removing gs://qwiklabs-gcp-02-01f28432cf1c/...
NotEmptyException: 409 BucketNotEmpty (qwiklabs-gcp-02-01f28432cf1c)
student_01_9b51dda9fd9c@cloudshell:~ (qwiklabs-gcp-02-01f28432cf1c)$ 
```

```sh
student_01_9b51dda9fd9c@cloudshell:~ (qwiklabs-gcp-02-01f28432cf1c)$ gsutil ls "gs://$BUCKET/"
gs://qwiklabs-gcp-02-01f28432cf1c/dummy_loan
student_01_9b51dda9fd9c@cloudshell:~ (qwiklabs-gcp-02-01f28432cf1c)$ gsutil rm gs://qwiklabs-gcp-02-01f28432cf1c/dummy_loan
Removing gs://qwiklabs-gcp-02-01f28432cf1c/dummy_loan...
/ [1 objects]                                                                   
Operation completed over 1 objects.                                              
student_01_9b51dda9fd9c@cloudshell:~ (qwiklabs-gcp-02-01f28432cf1c)$ gsutil ls "gs://$BUCKET/"
student_01_9b51dda9fd9c@cloudshell:~ (qwiklabs-gcp-02-01f28432cf1c)$ gsutil rb "gs://$BUCKET/"
Removing gs://qwiklabs-gcp-02-01f28432cf1c/...
student_01_9b51dda9fd9c@cloudshell:~ (qwiklabs-gcp-02-01f28432cf1c)$ 
```
###  Summary
```sh
student_01_9b51dda9fd9c@cloudshell:~ (qwiklabs-gcp-02-01f28432cf1c)$ history 
    1  export BUCKET=$(gcloud config get-value project)
    2  gsutil mb "gs://$BUCKET"
    3  gsutil retention set 10s "gs://$BUCKET"
    4  gsutil retention get "gs://$BUCKET"
    5  gsutil cp gs://spls/gsp297/dummy_transactions "gs://$BUCKET/"
    6  gsutil ls -L "gs://$BUCKET/dummy_transactions"
    7  gsutil retention lock "gs://$BUCKET/"
    8  gsutil retention get "gs://$Cloud Storage_BUCKET/"
    9  gsutil retention get "gs://$BUCKET/"
   10  gsutil retention temp set "gs://$BUCKET/dummy_transactions"
   11  gsutil rm "gs://$BUCKET/dummy_transactions"
   12  gsutil retention temp release "gs://$BUCKET/dummy_transactions"
   13  gsutil rm "gs://$BUCKET/dummy_transactions"
   14  gsutil retention event-default set "gs://$BUCKET/"
   15  gsutil cp gs://spls/gsp297/dummy_loan "gs://$BUCKET/"
   16  gsutil ls -L "gs://$BUCKET/dummy_loan"
   17  gsutil retention event release "gs://$BUCKET/dummy_loan"
   18  gsutil ls -L "gs://$BUCKET/dummy_loan"
   19  gsutil rb "gs://$BUCKET/"
   20  gsutil ls "gs://$BUCKET/"
   21  gsutil rm gs://qwiklabs-gcp-02-01f28432cf1c/dummy_loan
   22  gsutil ls "gs://$BUCKET/"
   23  gsutil rb "gs://$BUCKET/"
   24  history 
student_01_9b51dda9fd9c@cloudshell:~ (qwiklabs-gcp-02-01f28432cf1c)$ 
```

