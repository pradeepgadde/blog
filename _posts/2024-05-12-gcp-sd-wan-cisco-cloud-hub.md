---

layout: single
title:  "Cisco: SD-WAN Cloud Hub with Google Cloud"
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

# Cisco: SD-WAN Cloud Hub with Google Cloud

[Service Directory](https://cloud.google.com/service-directory) helps reduce the complexity of management and operations by providing a single place to publish, discover, and connect services. It is a  managed service that enhances service inventory management at scale so  you don’t have to. Service Directory provides real-time service  information, whether you have a few service endpoints or thousands. This helps ensure that your applications only resolve the most updated  information of their resources, increasing the reachability of your  services.

With Service Directory, you can easily understand all your services  across multi-cloud environments. This includes workloads running in  Compute Engine VMs, Google Kubernetes Engine (GKE), as well as external  services running on-prem and third-party clouds. It improves application reachability by maintaining the endpoint information for all your  services.

Service Directory solves the following problems:

1. **Interoperability**: Service Directory is a universal  naming service that works across Google Cloud, multi-cloud, and  on-premises. You can migrate services between these environments and  still use the same service name to register and resolve endpoints.
2. **Service management**: Service Directory is a managed  service. Your organization doesn't have to worry about the high  availability, redundancy, scaling, or maintenance concerns of  maintaining your own service registry.
3. **Access Control**: With Service Directory, you can  control who can register and resolve your services using IAM. Assign  Service Directory roles to teams, service accounts, and organizations.
4. **Limitations of pure DNS**: DNS resolvers can be  unreliable in terms of respecting TTLs and caching, cannot handle larger record sizes, and do not offer an easy way to serve metadata to users.  In addition to DNS support, Service Directory offers HTTP and gRPC APIs  to query and resolve services.



- Configure Service Directory, with a namespace, service, and endpoint
- Configure a Service Directory DNS zone
- Use Cloud Logging with Service Directory

## Configuring Service Directory

This section shows how to set up a Service Directory namespace, add a service to the namespace, and add endpoints to a service.

1. In the Console, search for "network services", then select **Service Directory**.
2. Click **Enable** to enable the Service Directory API.

> Service Directory is a platform for discovering, publishing, and connecting services. 

### Configure a namespace

Register a service that you can assign endpoints to. Use annotations for defining properties of the service (e.g. version number, location,  environment). When resolving a service, applications receive both the  endpoints and annotations associated with that service. 

1. In the Service Directory page, click **+REGISTER SERVICE**.

2. On the Register service page select **Standard** for Service type. 

3. Click **Next**.

4. In the **Region** pull-down menu, select a region for your namespace. For this lab, use .us-west1

5. In the **Namespace** field, select **CREATE NAMESPACE**.

6. In the **Namespace name** field give your namespace a name. For this lab, you can use `example-namespace`. A namespace is a way to group services within a  region. Each namespace can optionally be associated with a private Cloud DNS zone. 

    You are now adding a namespace to location "us-west1" 

7. Enter a **Service name**. For this lab use `example-service`.

8. Click **Create**.

### Configuring an endpoint

Once the service is registered, add some endpoints. An endpoint  consists of a unique name and the optional fields of address, port, and  key/value metadata. The address, if specified, must be valid IPv4 or  IPv6. An endpoint is an individual instance that is able to handle requests  for a given service. Endpoints consist of an IP address, port, and  annotations. Any service can have one or more endpoints. 

1. In the Service Directory page, click on your namespace, then click the service you just created.
2. Click **+Add Endpoint**.
3. Provide an **Endpoint name**. For this example, you can use `example-endpoint`.
4. Enter an IPv4 or IPv6 **IP address**. For this example, you can use `0.0.0.0`.
5. Enter a **Port number**. For this example, you can use `80`.
6. Click **Create**.

## Configuring a Service Directory DNS zone

You can create a Service Directory zone that allows your Google  Cloud-based services to query your Service Directory namespace via DNS.

1. From the **Network Services** menu, select **Cloud DNS**.

2. Click **Create zone**.

3. In the Zone type section, select **Private**.

4. Give the zone a name. For this example, you can use: `example-zone-name`.

5. Give the zone a DNS name. For this example, you can use: `myzone.example.com`.

6. Under **Options**, select `Use a service directory namespace`.

7. Under **Networks**, select one or more networks that can use the Service Directory zone. You should use the `default` network here, then click **OK**.

8. Select the **Region** where the namespace you want to link lives. Start typing  then select it.

   Select the **Namespace** you want to link. This should be the namespace you created earlier `example-namespace`. 

   Click **Create**.

> You can associate only one Service Directory zone with a given namespace, and you cannot associate a given zone with multiple  namespaces. You must create the Cloud DNS zone and the Service Directory namespace in the same project.

```sh
Welcome to Cloud Shell! Type "help" to get started.
Your Cloud Platform project in this session is set to qwiklabs-gcp-00-7ef88f0476fc.
Use “gcloud config set project [PROJECT_ID]” to change to a different project.
gcloud dns --project=qwiklabs-gcp-00-7ef88f0476fc managed-zones create example-zone-name --description="" --dns-name="myzone.example.com." --visibility="private" --networks="https://compute.googleapis.com/compute/v1/projects/qwiklabs-gcp-00-7ef88f0476fc/global/networks/default" --service-directory-namespace="https://servicedirectory.googleapis.com/v1/project
student_01_aa8f3a74b714@cloudshell:~ (qwiklabs-gcp-00-7ef88f0476fc)$ gcloud dns --project=qwiklabs-gcp-00-7ef88f0476fc managed-zones create example-zone-name --description="" --dns-name="myzone.example.com." --visibility="private" --networks="https://compute.googleapis.com/compute/v1/projects/qwiklabs-gcp-00-7ef88f0476fc/global/networks/default" --service-directory-namespace="https://servicedirectory.googleapis.com/v1/projects/qwiklabs-gcp-00-7ef88f0476fc/locations/us-west1/namespaces/example-namespace"
Created [https://dns.googleapis.com/dns/v1/projects/qwiklabs-gcp-00-7ef88f0476fc/managedZones/example-zone-name].
student_01_aa8f3a74b714@cloudshell:~ (qwiklabs-gcp-00-7ef88f0476fc)$ 
```



## Securing Service Directory in a service perimeter

[VPC Service Controls](https://cloud.google.com/vpc-service-controls/docs/overview) improves your ability to mitigate the risk of data exfiltration from  Google Cloud services. With VPC Service Controls, you can configure  security perimeters around the resources and data of services that you  explicitly specify.

## Querying using DNS

This section covers DNS querying, but there are no tasks you need to complete.

DNS queries for the following record types are supported:

- **A/AAAA/SRV** records for a service or an endpoint
- **SOA/NS** records for the private zone origin

## Logging and Monitoring

You can use [Cloud Monitoring](https://cloud.google.com/monitoring) and [Cloud Logging](https://cloud.google.com/logging) with Service Directory.

### Logging

Service Directory produces audit logs that can be viewed through Logging.

#### Audit logs

[Audit logs](https://cloud.google.com/logging/docs/audit) can help you answer the questions "Who did what, where, and when?". Service Directory writes two types of audit logs: **admin activity** and **data access**. Admin activity logs are always enabled and apply to the following Service Directory operations:

- CreateNamespace
- UpdateNamespace
- DeleteNamespace
- SetIamPolicy

All other Service Directory operations are considered data access  logs and are not enabled by default. Data access logs are also subject  to Logging pricing and quota, whereas neither applies to admin activity  logs. To enable data access logging, see [Configuring Data Access logs](https://cloud.google.com/logging/docs/audit/configure-data-access#config-console).

1. To see these logs in [Logging](https://cloud.google.com/logging/docs/view/overview#getting_started), in the Cloud Console, search for "logging" then select **Logging**. You'll be on the **Logs Explorer** page.
2. From the **Resource** dropdown select `Service Directory Namespace`, then select your region and expand the log for the namespace you created earlier.
3. Select `activity` from the **Log name** dropdown. You should see one `CreateNamespace` log.

```json
{
  "protoPayload": {
    "@type": "type.googleapis.com/google.cloud.audit.AuditLog",
    "authenticationInfo": {
      "principalEmail": "student-01-aa8f3a74b714@qwiklabs.net"
    },
    "requestMetadata": {
      "callerIp": "122.171.21.43",
      "callerSuppliedUserAgent": "Mozilla/5.0 (Macintosh; Intel Mac OS X 10.15; rv:125.0) Gecko/20100101 Firefox/125.0,gzip(gfe),gzip(gfe)",
      "requestAttributes": {
        "time": "2024-05-14T01:58:02.262379896Z",
        "auth": {}
      },
      "destinationAttributes": {}
    },
    "serviceName": "servicedirectory.googleapis.com",
    "methodName": "google.cloud.servicedirectory.v1beta1.RegistrationService.CreateNamespace",
    "authorizationInfo": [
      {
        "resource": "projects/00000011308f6a1a",
        "permission": "servicedirectory.namespaces.create",
        "granted": true,
        "resourceAttributes": {},
        "permissionType": "ADMIN_WRITE"
      }
    ],
    "resourceName": "projects/00000011308f6a1a",
    "request": {
      "parent": "projects/qwiklabs-gcp-00-7ef88f0476fc/locations/us-west1",
      "@type": "type.googleapis.com/google.cloud.servicedirectory.v1beta1.CreateNamespaceRequest",
      "namespace": {},
      "namespaceId": "example-namespace"
    },
    "response": {
      "name": "projects/qwiklabs-gcp-00-7ef88f0476fc/locations/us-west1/namespaces/example-namespace",
      "uid": "7cc0dba0fa124dbaa7a6074156068c93",
      "createTime": "2024-05-14T01:58:02.280888Z",
      "@type": "type.googleapis.com/google.cloud.servicedirectory.v1beta1.Namespace",
      "updateTime": "2024-05-14T01:58:02.280888Z"
    }
  },
  "insertId": "b41avkd6sdg",
  "resource": {
    "type": "servicedirectory_namespace",
    "labels": {
      "namespace_name": "example-namespace",
      "location": "us-west1",
      "project_id": "qwiklabs-gcp-00-7ef88f0476fc"
    }
  },
  "timestamp": "2024-05-14T01:58:02.256349627Z",
  "severity": "NOTICE",
  "logName": "projects/qwiklabs-gcp-00-7ef88f0476fc/logs/cloudaudit.googleapis.com%2Factivity",
  "receiveTimestamp": "2024-05-14T01:58:03.118380571Z"
}
```



1. In the Cloud Console return to  **Network Services** and select **Service Directory**.
2. For the namespace you created, click the three dots on the right side of the row. Click **Delete**, then **Delete** again.
3. Now go back to **Logging** and go to the **Logs Explorer** page.
4. You should now see a `DeleteNamespace` log.

```json
{
  "protoPayload": {
    "@type": "type.googleapis.com/google.cloud.audit.AuditLog",
    "authenticationInfo": {
      "principalEmail": "student-01-aa8f3a74b714@qwiklabs.net"
    },
    "requestMetadata": {
      "callerIp": "122.171.21.43",
      "callerSuppliedUserAgent": "Mozilla/5.0 (Macintosh; Intel Mac OS X 10.15; rv:125.0) Gecko/20100101 Firefox/125.0,gzip(gfe),gzip(gfe)",
      "requestAttributes": {
        "time": "2024-05-14T02:06:22.203405248Z",
        "auth": {}
      },
      "destinationAttributes": {}
    },
    "serviceName": "servicedirectory.googleapis.com",
    "methodName": "google.cloud.servicedirectory.v1beta1.RegistrationService.DeleteNamespace",
    "authorizationInfo": [
      {
        "resource": "projects/00000011308f6a1a/locations/us-west1/namespaces/example-namespace",
        "permission": "servicedirectory.namespaces.delete",
        "granted": true,
        "resourceAttributes": {},
        "permissionType": "ADMIN_WRITE"
      }
    ],
    "resourceName": "projects/00000011308f6a1a/locations/us-west1/namespaces/example-namespace",
    "request": {
      "@type": "type.googleapis.com/google.cloud.servicedirectory.v1beta1.DeleteNamespaceRequest",
      "name": "projects/qwiklabs-gcp-00-7ef88f0476fc/locations/us-west1/namespaces/example-namespace"
    }
  },
  "insertId": "n1eko3d67jo",
  "resource": {
    "type": "servicedirectory_namespace",
    "labels": {
      "location": "us-west1",
      "project_id": "qwiklabs-gcp-00-7ef88f0476fc",
      "namespace_name": "example-namespace"
    }
  },
  "timestamp": "2024-05-14T02:06:22.197103965Z",
  "severity": "NOTICE",
  "logName": "projects/qwiklabs-gcp-00-7ef88f0476fc/logs/cloudaudit.googleapis.com%2Factivity",
  "receiveTimestamp": "2024-05-14T02:06:22.846838945Z"
}
```



### Monitoring

Monitoring allows you to create dashboards or set up alerts and can be accessed by visiting Monitoring in Cloud Console.

To view basic monitoring metrics (request count, size and latency), you can go to the Metrics Explorer and filter by `resource_type:consumed_api` and `service:servicedirectory.googleapis.com`.
