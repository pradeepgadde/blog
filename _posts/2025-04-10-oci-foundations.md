---

layout: single
title:  "Oracle Cloud Infrastructure Foundation"
categories: Cloud
tags: OCI
show_date: true
classes: wide
header:
  teaser: /assets/images/oci.png
author:
  name     : "Oracle"
  avatar   : "/assets/images/oci.png"

sidebar:
  - title: "Blog"
   
    text: "Checkout other topics"
    nav: my-sidebar

---

# OCI Fundamentals

## Compute

- The primary goal of distributing resources across multiple **availability  domains** in Oracle Cloud Infrastructure is to improve fault tolerance and high availability. By spreading resources across isolated,  fault-tolerant data centers within a region, applications can continue  operating even if one availability domain experiences an outage.

- It is important to choose a **region** close to users to reduce network latency and increase application performance. 

- The Oracle Cloud Infrastructure Compute service offers **Virtual Machines  (VMs)**, **Bare Metal instances**, and **Dedicated Virtual Machine Hosts**. 

- **Vertical scaling** in the Oracle Cloud Infrastructure Compute service  refers to the process of increasing the performance of an instance by  adding more OCPUs and memory. This allows for improved processing  capabilities and resource allocation, resulting in enhanced performance  for the compute instance.

- **Horizontal scaling** is achieved by adding or removing instances within an instance pool in Oracle Cloud Infrastructure Compute. This type of  scaling helps manage workloads and maintain optimal resource utilization by distributing traffic across multiple instances.

- **Oracle Roving Edge Infrastructure** is a service designed to host  workloads and use cases that operate at the edge, especially the far  edge and tactical edge. It is designed to function as an extension of  your Oracle Cloud tenancy.

- **OCI Dedicated Region** enables customers to deploy an **Oracle Cloud region in their own data center**. 

- Oracle Cloud Infrastructure's **Bring Your Own License (BYOL)** feature  helps customers save on costs by allowing them to use their existing  software licenses in OCI. This means customers can leverage their  investment in Oracle software licenses when migrating or deploying new  applications in OCI, without the need to purchase additional licenses.

- When creating an Oracle Cloud Infrastructure Compute **flexible shape**  instance, users can customize the number of OCPUs and the amount of  memory according to their needs. Flexible shapes provide more control  over resource allocation and help users optimize costs and performance.

- In the Oracle Cloud Infrastructure Compute service, an **Instance  Configuration** is a **predefined configuration** that includes the instance's shape, base image, and metadata. It allows users to quickly create new  instances with the same configuration, streamlining the deployment  process.

- **Live Migration** is a feature in the Oracle Cloud Infrastructure Compute  service that enables users to **migrate running instances** between  different fault domains without any downtime. It allows users to perform maintenance or balance workloads across fault domains while maintaining the availability and performance of their applications.

## Networking
- Oracle Cloud Infrastructure **Load Balancer** supports three types of load balancing algorithms: **Round Robin**, **Least Connections**, and **IP Hash**. The Round Robin algorithm distributes incoming traffic evenly across instances. This algorithm helps ensure efficient distribution of network traffic and maintain the availability and performance of applications. 
- The main difference between a **Load Balancer** and a **Network Load Balancer**  in Oracle Cloud Infrastructure is that a Load Balancer works at the  application layer (**layer 7**) and can handle application-specific traffic, while a Network Load Balancer works at the transport layer (**layer 4**)  and can handle any type of TCP or UDP traffic.
- A **Dynamic Routing Gateway (DRG)** provides a path for traffic between a  VCN and an on-premises network or another VCN in the same or different  region.
- The key difference between **Security Lists** and **Network Security Groups** in Oracle Cloud Infrastructure is that Security Lists apply to **subnets**,  while Network Security Groups apply to individual instance **VNICs**. This  allows for more granular control of traffic in and out of instances.
- A **Route Table** in OCI defines how traffic leaving a subnet is routed. You create route rules to send traffic to specific targets, such as an  **Internet Gateway (for public traffic)** or a **NAT Gateway (for instances in a private subnet requiring outbound internet access)**. 
- A **Route Table** is a component in Oracle Cloud Infrastructure Networking  Service that defines rules for packet forwarding to **destinations outside the Virtual Cloud Network (VCN)**. Route Tables have rules to route  traffic from subnets to destinations outside the VCN by way of gateways  or specially configured instances.
- A **Service Gateway** in Oracle Cloud Infrastructure networking service  enables access to **Oracle services within the same region without the  traffic going through the public internet**. This provides a more secure  and reliable connection for accessing Oracle services like Object  Storage, Autonomous Database, and others.




## Storage

- **Archive Storage** is a storage tier in the Oracle Cloud Infrastructure  Object Storage service designed for rarely or seldom accessed data that  can be restored within hours. It offers the lowest cost per stored  gigabyte and is suitable for long-term storage of data that is not  needed for immediate access, such as backups or historical data.
- **Block storage** is the type of storage associated with instances in the  Oracle Cloud Infrastructure Compute service. It provides low-latency,  high-performance storage volumes that can be attached to instances to  store data and applications.
- **Auto Tiering** for **Object Storage** is a feature that helps you automatically move objects between **Standard** and **Infrequent Access tiers** based on  their access patterns.
- **Auto-Tiering** uses lifecycle policies to move infrequently accessed data  from the Standard tier to a lower-cost storage tier, such as Archive or  Infrequent Access. This helps reduce costs by keeping rarely accessed  objects in less expensive storage.
- OCI Block Volumes are automatically replicated within an availability  domain for high durability, ensuring data redundancy and protection  against hardware failures.
- In the Oracle Cloud Infrastructure Block Volume service, **Online Resizing** enables you to increase the size of a block volume without any  downtime. This feature allows you to scale storage capacity on the fly  to accommodate growing data needs or application requirements, ensuring  continuous availability and performance.
- The Oracle Cloud Infrastructure **File Storage** service uses the Network  File System (NFS) protocol for file access. NFS allows clients to access files over a network in a manner that appears as though they are part  of the local file system.

## IAM

- An IAM policy statement in Oracle Cloud Infrastructure typically  consists of these components: **Location** (compartment or tenancy), **Action  Verb** (the specific action to be allowed), **Resource** (the resources the  action can be performed on), Principal (group the policy applies to),  and a set of optional **Conditions**. 
- In Oracle Cloud Infrastructure, the **Principal** component of an IAM  policy statement defines the user or group the policy applies to. It  specifies the groups that the policy statement affects, granting them  access to resources and actions defined in the policy statement.
- **Compartments** in Oracle Cloud Infrastructure are a global resource; they  can be nested to create a hierarchy; and IAM policies can be written to  grant access to resources in specific compartments.
- In Oracle Cloud Infrastructure, **compartment quotas** are applied on a  **per-compartment basis**. This allows administrators to set different  resource limits for each compartment, ensuring that resource usage  aligns with the organization's policies and requirements.
- Storing and managing encryption keys and secrets is a function of the Oracle Cloud Infrastructure **Vault** service.
- **Dynamic Groups** allow OCI Compute instances and other cloud resources to  act as principals and receive permissions via IAM policies. This means  that Compute instances, Functions, and other resources can have  permissions without requiring a specific user to authenticate.
- In the Oracle Cloud Infrastructure **shared security responsibility model**, the customer is responsible for securing their data, applications, and  access control. This includes implementing appropriate security measures such as encryption, user access control, and monitoring to protect  sensitive data and ensure the overall security of their cloud  environment.

## Security

- Oracle Cloud Infrastructure **Security Zones** help to enforce best practice security configurations for your resources. By using Security Zones,  you can ensure that resources are created and managed with security best practices, minimizing potential security risks and improving the  overall security posture of your infrastructure.
- OCI **Security Zones** enforce security best practices and policies by  preventing risky actions that could lead to security vulnerabilities. If a **policy violation** occurs, the **operation is denied** to ensure compliance with security standards.
- Oracle Cloud Infrastructure **Web Application Firewall (WAF)** is designed  to protect your web applications from various types of malicious  attacks, such as SQL injection and cross-site scripting. WAF inspects  incoming web traffic and filters out any requests that match predefined  security rules, ensuring the security and availability of your web  applications.
- In Oracle Cloud Infrastructure, **Security Lists** are responsible for  controlling traffic **between subnets** within a virtual cloud network  (VCN). They define ingress and egress rules to determine the allowed  traffic at the subnet level.

## Monitoring

- Oracle Cloud Infrastructure **Cloud Guard** continuously monitors your cloud  resources and configurations to detect, assess, and remediate security  risks. Cloud Guard helps maintain a strong security posture by  identifying and addressing potential security issues before they become  critical.
- **Ingress data transfer** refers to the data transferred into Oracle Cloud  Infrastructure from other sources. Generally, ingress data transfer is  free of charge, while **egress data transfe**r can have associated costs  depending on the destination (to a different cloud provider like AWS or  GCP, to a different OCI region, or to the internet).

- The primary purpose of setting up **budgets** in Oracle Cloud Infrastructure is to monitor and control spending on OCI services. Budgets allow  customers to track their spending and receive alerts when their spending approaches or exceeds the budget limits they have set, enabling them to manage costs effectively.
- The **Cost Analysis** tool in Oracle Cloud Infrastructure enables users to  visualize and analyze their **cloud usage** and **spending patterns** over time. This helps users better understand their consumption of resources and  services and make informed decisions about cost optimization.