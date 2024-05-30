---

layout: single
title:  "IAM Custom Roles"
categories: Cloud
tags: GCP
classes: wide
show_date: true
header:
  overlay_image: /assets/images/gcp-banner.png
  og_image: /assets/images/gcp-banner.png
  teaser: /assets/images/pca-gcp.png
author:
  name     : "Professional Cloud Architect"
  avatar   : "/assets/images/pca-gcp.png"

sidebar:
  - title: "Topics"
    nav: my-sidebar

---
# IAM Custom Roles

Cloud IAM provides the right tools to manage resource permissions  with minimum fuss and high automation. You don't directly grant users  permissions. Instead, you grant them roles, which bundle one or more  permissions. This allows you to map job functions within your company to groups and roles. Users get access only to what they need to get the  job done, and admins can easily grant default permissions to entire  groups of users.

There are two kinds of roles in Cloud IAM:

- Predefined Roles
- Custom Roles

**Predefined roles** are created and maintained by  Google. Their permissions are automatically updated as necessary, such  as when new features or services are added to Google Cloud.

**Custom roles** are user-defined, and allow you to  bundle one or more supported permissions to meet your specific needs.  Custom roles are not maintained by Google; when new permissions,  features, or services are added to Google Cloud, your custom roles will  not be updated automatically.You create a custom role by combining one  or more of the available Cloud IAM permissions. Permissions allow users  to perform specific actions on Google Cloud resources.

### 

In the Cloud IAM world, permissions are represented in the form `<service>.<resource>.<verb>`. For example, the `compute.instances.list` permission allows a user to list the Compute Engine instances they own, while `compute.instances.stop` allows a user to stop a VM.

Permissions usually, but not always, correspond 1:1 with REST  methods. That is, each Google Cloud service has an associated permission for each REST method that it has. To call a method, the caller needs  that permission. For example, the caller of `topic.publish()` needs the `pubsub.topics.publish` permission.

Custom roles can only be used to grant permissions in policies for  the same project or organization that owns the roles or resources under  them. You cannot grant custom roles from one project or organization on a resource owned by a different project or organization.

### Required permissions and roles

To create a custom role, a caller must have the `iam.roles.create` permission.

Users who are not owners, including organization administrators, must be assigned either the Organization Role Administrator role  (roles/iam.organizationRoleAdmin) or the IAM Role Administrator role  (roles/iam.roleAdmin). The IAM Security Reviewer role  (roles/iam.securityReviewer) enables the ability to view custom roles  but not administer them.

The custom roles user interface is in the Cloud Console under IAM  Roles. It is only available to users who have permissions to create or  manage custom roles. By default, only project owners can create new  roles. Project owners can control access to this feature by granting IAM Role Administrator role to others on the same project; for  organizations, only Organization Administrators can grant the  Organization Role, Administrator role.

```sh
student_01_3019b4c4e30b@cloudshell:~ (qwiklabs-gcp-02-680d2fa80cd5)$ gcloud iam list-testable-permissions //cloudresourcemanager.googleapis.com/projects/$DEVSHELL_PROJECT_ID
---
apiDisabled: true
name: accessapproval.requests.approve
stage: GA
---
apiDisabled: true
name: accessapproval.requests.dismiss
stage: GA
---
apiDisabled: true
name: accessapproval.requests.get
stage: GA
---
apiDisabled: true
name: accessapproval.requests.invalidate
stage: GA
---
apiDisabled: true
name: accessapproval.requests.list
stage: GA
---
apiDisabled: true
name: accessapproval.serviceAccounts.get
stage: GA
---
apiDisabled: true
name: accessapproval.settings.delete
stage: GA
---
apiDisabled: true
name: accessapproval.settings.get
stage: GA
---
apiDisabled: true
name: accessapproval.settings.update
stage: GA
---
name: actions.agent.claimContentProvider
stage: GA
title: Claim a content provider using an AoG project
---
name: actions.agent.get
stage: GA
title: Get action data and metadata
---
name: actions.agent.update
stage: GA
title: Update action data and metadata
---
name: actions.agentVersions.create
stage: GA
title: Create an action version
---
name: actions.agentVersions.delete
stage: GA
title: Delete an action version
---
name: actions.agentVersions.deploy
stage: GA
title: Deploy an action version
---
name: actions.agentVersions.get
stage: GA
title: Get an action version
---
name: actions.agentVersions.list
stage: GA
title: List all the action versions
---
apiDisabled: true
name: advisorynotifications.notifications.get
stage: GA
title: Get notification
---
apiDisabled: true
name: advisorynotifications.notifications.list
stage: GA
title: List notifications
---
apiDisabled: true
name: advisorynotifications.settings.get
stage: GA
title: Get settings
---
apiDisabled: true
name: advisorynotifications.settings.update
stage: GA
title: Update settings
---
description: Create agent example
name: aiplatform.agentExamples.create
stage: BETA
title: Create agent example
---
description: Delete agent example
name: aiplatform.agentExamples.delete
stage: BETA
title: Delete agent example
---
description: Get agent example
name: aiplatform.agentExamples.get
stage: BETA
title: Get agent example
---
^C

Command killed by keyboard interrupt


student_01_3019b4c4e30b@cloudshell:~ (qwiklabs-gcp-02-680d2fa80cd5)$ 
```



## View the available permissions for a resource

Before you create a custom role, you might want to know what  permissions can be applied to a resource. You can get all permissions  that can be applied to a resource, and the resources below that in the  hierarchy, using the gcloud command-line tool, the Cloud Console, or the IAM API. For example, you can get all permissions that you can apply on an organization and on projects in that organization.





## Get the role metadata

Before you create a custom role, you might want to get the metadata  for both predefined and custom roles. Role metadata includes the role ID and permissions contained in the role. You can view the metadata using  the Cloud Console or the IAM API.

```sh

student_01_3019b4c4e30b@cloudshell:~ (qwiklabs-gcp-02-680d2fa80cd5)$ gcloud iam roles describe roles/editor | more
description: View, create, update, and delete most Google Cloud resources. See the
  list of included permissions.
etag: AA==
includedPermissions:
- accessapproval.requests.get
- accessapproval.requests.list
- accessapproval.serviceAccounts.get
- accessapproval.settings.get
- accesscontextmanager.accessLevels.create
- accesscontextmanager.accessLevels.delete
- accesscontextmanager.accessLevels.get
- accesscontextmanager.accessLevels.list
- accesscontextmanager.accessLevels.replaceAll
- accesscontextmanager.accessLevels.update
- accesscontextmanager.authorizedOrgsDescs.create
- accesscontextmanager.authorizedOrgsDescs.delete
- accesscontextmanager.authorizedOrgsDescs.get
- accesscontextmanager.authorizedOrgsDescs.list
- accesscontextmanager.authorizedOrgsDescs.update
- accesscontextmanager.gcpUserAccessBindings.create
- accesscontextmanager.gcpUserAccessBindings.delete
- accesscontextmanager.gcpUserAccessBindings.get
- accesscontextmanager.gcpUserAccessBindings.list
- accesscontextmanager.gcpUserAccessBindings.update
- accesscontextmanager.policies.create
- accesscontextmanager.policies.delete
- accesscontextmanager.policies.get
- accesscontextmanager.policies.getIamPolicy
- accesscontextmanager.policies.list
- accesscontextmanager.policies.update
- accesscontextmanager.servicePerimeters.commit
- accesscontextmanager.servicePerimeters.create
--More--
```



## View the grantable roles on resources

Use the `gcloud iam list-grantable-roles` command to return a list of all roles that can be applied to a given resource.

```sh
student_01_3019b4c4e30b@cloudshell:~ (qwiklabs-gcp-02-680d2fa80cd5)$ gcloud iam list-grantable-roles //cloudresourcemanager.googleapis.com/projects/$DEVSHELL_PROJECT_ID
---
description: Access to edit and deploy an action
name: roles/actions.Admin
title: Actions Admin
---
description: Access to view an action
name: roles/actions.Viewer
title: Actions Viewer
---
description: Gives Vertex AI Colab the proper permissions to function.
name: roles/aiplatform.colabServiceAgent
title: Vertex AI Colab Service Agent
---
description: Gives Vertex AI Custom Code the proper permissions.
name: roles/aiplatform.customCodeServiceAgent
title: Vertex AI Custom Code Service Agent
---
description: Gives Vertex AI Extension that executes custom code the permissions it
  needs to function.
name: roles/aiplatform.extensionCustomCodeServiceAgent
title: Vertex AI Extension Custom Code Service Agent
---
description: Gives Vertex AI Extension the permissions it needs to function.
name: roles/aiplatform.extensionServiceAgent
title: Vertex AI Extension Service Agent
---
description: Deprecated. Use featurestoreAdmin instead.
name: roles/aiplatform.featurestoreUser
title: Vertex AI Feature Store User
---
description: Gives Vertex AI Model Monitoring the permissions it needs to function.
name: roles/aiplatform.modelMonitoringServiceAgent
title: Vertex AI Model Monitoring Service Agent
---
description: Grants users full access to schedules and notebook execution jobs.
name: roles/aiplatform.notebookExecutorUser
title: Notebook Executor User
---
description: Vertex AI Service Agent used to run Notebook managed resources in user
  project with restricted permissions.
name: roles/aiplatform.notebookServiceAgent
title: Vertex AI Notebook Service Agent
---
description: Vertex AI Service Agent used by Vertex RAG to access user imported data
  and Vertex AI in the project
name: roles/aiplatform.ragServiceAgent
title: Vertex AI RAG Data Service Agent
{snip}
```



## Create a custom role

To create a custom role, a caller must possess `iam.roles.create` permission. By default, the owner of a project or an organization has this permission and can create and manage custom roles.

Users who are not owners, including organization admins, must be  assigned either the Organization Role Administrator role, or the IAM  Role Administrator role.

Use the `gcloud iam roles create` command to create new custom roles in two ways:

- Provide a YAML file that contains the role definition
- Specify the role definition using flags

When creating a custom role, you must specify whether it applies to the organization level or project level by using the `--organization [ORGANIZATION_ID]` or `--project [PROJECT_ID]` flags. Each example below creates a custom role at the project level.



### Create a custom role using a YAML file

Create a YAML file that contains the definition for your custom role. The file must be structured in the following way:

```yaml
student_01_3019b4c4e30b@cloudshell:~ (qwiklabs-gcp-02-680d2fa80cd5)$ cat role-definition.yaml
title: "Role Editor"
description: "Edit access for App Versions"
stage: "ALPHA"
includedPermissions:
- appengine.versions.create
- appengine.versions.delete
student_01_3019b4c4e30b@cloudshell:~ (qwiklabs-gcp-02-680d2fa80cd5)$ 
```

```sh
student_01_3019b4c4e30b@cloudshell:~ (qwiklabs-gcp-02-680d2fa80cd5)$ gcloud iam roles create editor --project $DEVSHELL_PROJECT_ID \
--file role-definition.yaml
Created role [editor].
description: Edit access for App Versions
etag: BwYZpr7w6RM=
includedPermissions:
- appengine.versions.create
- appengine.versions.delete
name: projects/qwiklabs-gcp-02-680d2fa80cd5/roles/editor
stage: ALPHA
title: Role Editor
student_01_3019b4c4e30b@cloudshell:~ (qwiklabs-gcp-02-680d2fa80cd5)$ 
```



### Create a custom role using flags

Now you'll use the flag method to create a new custom role. The flags take a similar form to the YAML file, so you’ll recognize how the  command is being built.

```sh
student_01_3019b4c4e30b@cloudshell:~ (qwiklabs-gcp-02-680d2fa80cd5)$ gcloud iam roles create viewer --project $DEVSHELL_PROJECT_ID \
--title "Role Viewer" --description "Custom role description." \
--permissions compute.instances.get,compute.instances.list --stage ALPHA
Created role [viewer].
description: Custom role description.
etag: BwYZpsJWKJ0=
includedPermissions:
- compute.instances.get
- compute.instances.list
name: projects/qwiklabs-gcp-02-680d2fa80cd5/roles/viewer
stage: ALPHA
title: Role Viewer
student_01_3019b4c4e30b@cloudshell:~ (qwiklabs-gcp-02-680d2fa80cd5)$ 
```



## List the custom roles

Execute the following `gcloud` command to list custom roles, specifying either project-level or organization-level custom roles:

```sh
student_01_3019b4c4e30b@cloudshell:~ (qwiklabs-gcp-02-680d2fa80cd5)$ gcloud iam roles list --project $DEVSHELL_PROJECT_ID
---
description: Edit access for App Versions
etag: BwYZpr7w6RM=
name: projects/qwiklabs-gcp-02-680d2fa80cd5/roles/editor
title: Role Editor
---
description: Custom role description.
etag: BwYZpsJWKJ0=
name: projects/qwiklabs-gcp-02-680d2fa80cd5/roles/viewer
title: Role Viewer
student_01_3019b4c4e30b@cloudshell:~ (qwiklabs-gcp-02-680d2fa80cd5)$ 
```



## Update an existing custom role

A common pattern for updating a resource's metadata, such as a custom role, is to read its current state, update the data locally, and then  send the modified data for writing. This pattern could cause a conflict  if two or more independent processes attempt the sequence  simultaneously.

For example, if two owners for a project try to make conflicting changes to a role at the same time, some changes could fail.

Cloud IAM solves this problem using an `etag` property in  custom roles. This property is used to verify if the custom role has  changed since the last request. When you make a request to Cloud IAM  with an etag value, Cloud IAM compares the etag value in the request  with the existing etag value associated with the custom role. It writes  the change only if the etag values match.

### Update a custom role using a YAML file

1. Get the current definition for the role by executing the following `gcloud` command, replacing `[ROLE_ID]` with **editor**.

```sh
student_01_3019b4c4e30b@cloudshell:~ (qwiklabs-gcp-02-680d2fa80cd5)$ gcloud iam roles describe editor --project $DEVSHELL_PROJECT_ID
description: Edit access for App Versions
etag: BwYZpr7w6RM=
includedPermissions:
- appengine.versions.create
- appengine.versions.delete
name: projects/qwiklabs-gcp-02-680d2fa80cd5/roles/editor
stage: ALPHA
title: Role Editor
student_01_3019b4c4e30b@cloudshell:~ (qwiklabs-gcp-02-680d2fa80cd5)$ 
```


Copy the output to use to create a new YAML file in the next steps.

Create a new-role-definition.yaml file with your editor:

```sh
student_01_3019b4c4e30b@cloudshell:~ (qwiklabs-gcp-02-680d2fa80cd5)$ cat new-role-definition.yaml 
description: Edit access for App Versions
etag: BwYZpr7w6RM=
includedPermissions:
- appengine.versions.create
- appengine.versions.delete
- storage.buckets.get
- storage.buckets.list
name: projects/qwiklabs-gcp-02-680d2fa80cd5/roles/editor
stage: ALPHA
title: Role Editor
student_01_3019b4c4e30b@cloudshell:~ (qwiklabs-gcp-02-680d2fa80cd5)$ 
```

```sh
student_01_3019b4c4e30b@cloudshell:~ (qwiklabs-gcp-02-680d2fa80cd5)$ gcloud iam roles update editor --project $DEVSHELL_PROJECT_ID \
--file new-role-definition.yaml
description: Edit access for App Versions
etag: BwYZptJknhc=
includedPermissions:
- appengine.versions.create
- appengine.versions.delete
- storage.buckets.get
- storage.buckets.list
name: projects/qwiklabs-gcp-02-680d2fa80cd5/roles/editor
stage: ALPHA
title: Role Editor
student_01_3019b4c4e30b@cloudshell:~ (qwiklabs-gcp-02-680d2fa80cd5)$ 
```



### Update a custom role using flags

Each part of a role definition can be updated using a corresponding  flag. For a list of all possible flags from the SDK reference  documentation, see the topic [gcloud iam roles update](https://cloud.google.com/sdk/gcloud/reference/iam/roles/update).

Use the following flags to add or remove permissions:

- `--add-permissions`: Adds one or more comma-separated permissions to the role.
- `--remove-permissions`: Removes one or more comma-separated permissions from the role.

Alternatively, you can simply specify the new permissions using the `--permissions [PERMISSIONS]` flag and providing a comma-separated list of permissions to replace the existing permissions list.

```sh
student_01_3019b4c4e30b@cloudshell:~ (qwiklabs-gcp-02-680d2fa80cd5)$ gcloud iam roles update viewer --project $DEVSHELL_PROJECT_ID \
--add-permissions storage.buckets.get,storage.buckets.list
description: Custom role description.
etag: BwYZptptT0M=
includedPermissions:
- compute.instances.get
- compute.instances.list
- storage.buckets.get
- storage.buckets.list
name: projects/qwiklabs-gcp-02-680d2fa80cd5/roles/viewer
stage: ALPHA
title: Role Viewer
student_01_3019b4c4e30b@cloudshell:~ (qwiklabs-gcp-02-680d2fa80cd5)$ 
```



## Disable a custom role

When a role is disabled, any policy bindings related to the role are  inactivated, meaning that the permissions in the role will not be  granted, even if you grant the role to a user.

The easiest way to disable an existing custom role is to use the `--stage` flag and set it to DISABLED.

```sh
student_01_3019b4c4e30b@cloudshell:~ (qwiklabs-gcp-02-680d2fa80cd5)$ gcloud iam roles update viewer --project $DEVSHELL_PROJECT_ID \
--stage DISABLED
description: Custom role description.
etag: BwYZptyXXkQ=
includedPermissions:
- compute.instances.get
- compute.instances.list
- storage.buckets.get
- storage.buckets.list
name: projects/qwiklabs-gcp-02-680d2fa80cd5/roles/viewer
stage: DISABLED
title: Role Viewer
student_01_3019b4c4e30b@cloudshell:~ (qwiklabs-gcp-02-680d2fa80cd5)$ 
```



## Delete a custom role

- Use the `gcloud iam roles delete` command to delete a custom role. Once deleted the role is inactive and cannot be used to create new IAM policy bindings:

```sh
student_01_3019b4c4e30b@cloudshell:~ (qwiklabs-gcp-02-680d2fa80cd5)$ gcloud iam roles delete viewer --project $DEVSHELL_PROJECT_ID
deleted: true
description: Custom role description.
etag: BwYZpt-Scgo=
includedPermissions:
- compute.instances.get
- compute.instances.list
- storage.buckets.get
- storage.buckets.list
name: projects/qwiklabs-gcp-02-680d2fa80cd5/roles/viewer
stage: DISABLED
title: Role Viewer
student_01_3019b4c4e30b@cloudshell:~ (qwiklabs-gcp-02-680d2fa80cd5)$ 
```

After the role has been deleted, existing bindings remain, but are  inactive. The role can be undeleted within 7 days. After 7 days, the  role enters a permanent deletion process that lasts 30 days. After 37  days, the Role ID is available to be used again.

## Restore a custom role

- Within the 7 days window you can restore a role. Deleted roles are in a **DISABLED** state. To make it available again, update the `--stage` flag:

```sh
student_01_3019b4c4e30b@cloudshell:~ (qwiklabs-gcp-02-680d2fa80cd5)$ gcloud iam roles undelete viewer --project $DEVSHELL_PROJECT_ID
description: Custom role description.
etag: BwYZpuS-894=
includedPermissions:
- compute.instances.get
- compute.instances.list
- storage.buckets.get
- storage.buckets.list
name: projects/qwiklabs-gcp-02-680d2fa80cd5/roles/viewer
stage: DISABLED
title: Role Viewer
student_01_3019b4c4e30b@cloudshell:~ (qwiklabs-gcp-02-680d2fa80cd5)$ 
```

## Summary

```sh
student_01_3019b4c4e30b@cloudshell:~ (qwiklabs-gcp-02-680d2fa80cd5)$ history 
    1  gcloud iam list-testable-permissions //cloudresourcemanager.googleapis.com/projects/$DEVSHELL_PROJECT_ID
    2  gcloud iam list-testable-permissions //cloudresourcemanager.googleapis.com/projects/$DEVSHELL_PROJECT_ID | less
    3  gcloud iam list-testable-permissions //cloudresourcemanager.googleapis.com/projects/$DEVSHELL_PROJECT_ID
    4  gcloud iam roles describe roles/editor
    5  gcloud iam roles describe roles/viewer
    6  gcloud iam roles describe roles/editor | more
    7  gcloud iam list-grantable-roles //cloudresourcemanager.googleapis.com/projects/$DEVSHELL_PROJECT_ID
    8  nano role-definition.yaml
    9  cat role-definition.yaml
   10  gcloud iam roles create editor --project $DEVSHELL_PROJECT_ID --file role-definition.yaml
   11  gcloud iam roles create viewer --project $DEVSHELL_PROJECT_ID --title "Role Viewer" --description "Custom role description." --permissions compute.instances.get,compute.instances.list --stage ALPHA
   12  gcloud iam roles list --project $DEVSHELL_PROJECT_ID
   13  gcloud iam roles list
   14  gcloud iam roles describe editor --project $DEVSHELL_PROJECT_ID
   15  nano new-role-definition.yaml
   16  cat new-role-definition.yaml 
   17  gcloud iam roles update editor --project $DEVSHELL_PROJECT_ID --file new-role-definition.yaml
   18  gcloud iam roles update viewer --project $DEVSHELL_PROJECT_ID --add-permissions storage.buckets.get,storage.buckets.list
   19  gcloud iam roles update viewer --project $DEVSHELL_PROJECT_ID --stage DISABLED
   20  gcloud iam roles delete viewer --project $DEVSHELL_PROJECT_ID
   21  gcloud iam roles undelete viewer --project $DEVSHELL_PROJECT_ID
   22  history 
student_01_3019b4c4e30b@cloudshell:~ (qwiklabs-gcp-02-680d2fa80cd5)$ 
```

