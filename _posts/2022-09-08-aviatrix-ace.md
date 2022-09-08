---
layout: single
title:  "Getting Started with Aviatrix"
date:   2022-09-07 011:58:04 +0530
categories: Cloud
tags: Aviatrix
show_date: true
toc: true
toc_sticky: true
header:
  teaser: /assets/images/aviatrix.png
author:
  name     : "Aviatrix"
  avatar   : "/assets/images/aviatrix.png"

sidebar:
  - title: "Blog"
   
    text: "Checkout other topics"
    nav: my-sidebar

---
# Aviatrix ACE Prep Notes
According to Aviatrix documentation, The Aviatrix Certified Engineer (ACE) program is the industry’s first and only multi-cloud networking and security certification program. 

The ACE certification is designed for individuals who already understand basic networking concepts and prepares them with the working knowledge of native networking constructs in AWS, Azure, Google Cloud, and Oracle Cloud Infrastructure as well as the proficiency to build, automate, scale, and secure multi-cloud network architectures.

## Reference

[https://community.aviatrix.com/t/y4hh4ml/ace-associate-self-paced-learning-resources#about-aviatrix](https://community.aviatrix.com/t/y4hh4ml/ace-associate-self-paced-learning-resources#about-aviatrix) 

The following notes are a brief excerpts from this reference.

## Multi-Cloud Network Architecture

Aviatrix creates a purpose-built Multi-Cloud Network Architecture (MCNA) by implementing a data plane through dynamic and software-defined routing with a centralized control plane. 

Security is built into the network architecture through segmentation, encryption, ingress and egress filtering, and security services insertion. 

Aviatrix also leverages orchestrating cloud-native constructs, where necessary, in building and controlling the enterprise network and life-cycle management of the overall architecture. 

MCNA architecture defines four distinct layers at a high level. 
- Cloud Core 
- Cloud Security
- Cloud Access 
- Cloud Operations

Within the cloud core, there are two subdivisions: The applications layer and the global transit layer. 

MCNA showcases a centralized controller to manage single or multiple clouds with a global, distributed, unified and normalized data plane. 

## An Introduction to the Public Cloud and Public Cloud Networking

Public Cloud
On-Prem DC
Private Cloud

How Networking Works in the Public Cloud

The CSP’s pick a region to build their DC.
Next, they pick an Availability Zone located in the region.
The AZ’s hold the actual, physical data centers.
This prevents issues because there are multiple data centers, so one can take over if another has a problem.

## Public Cloud Networking

IaaS, PaaS, SaaS 

On Prem (physical/virtual) 

- Underlay, hardware, software, day0, day1 and day 2, everything is the user’s responsibility 

Infrastructure as a Service

- CSP’s manage hardware, software, storage, and networking 

- Users are responsible for running the virtual machines and patching the O/S 

Platform as a Service 

- Users only consume as a platform 

- CSP’s manage everything, you only manage applications and data 

Software as a Service 

- Users consume the service 

- All aspects managed by CSP’s 



Hybrid Cloud

- Cloud connectivity with On-Prem DC 

Public Cloud Basics 

- Known to be resilient, highly available, multiple regions 
- Just as data centers have issues, the CSP’s data centers have issues as well 
- Users however have no control/visibility of these issues 

Data Center 

- Cloud service providers use data centers to house cloud services and cloud-based resources 

Region 

- Data centers are grouped in regions and geographic areas to provide regional service availability 

Availability Zone 



Public Cloud vs On-Prem DC

- Distinct locations within the cloud provider’s network that are engineered to be isolated from failures 

- Public Cloud Network tries to provide the same services as the On-Prem DC 
- Provides concept of VPC (virtual private cloud) 
- The most important part of the VPC is the application/virtual machine 
- Virtual machines are sitting on different subnets, so they need a routing entity 
- Some security constructs are provided but are often very primitive 
- VPC required connectivity to internet (sends traffic to internet) 
- Users coming in/trying to get access to virtual machines 
- Private link to connect to data center needed 
- Limitations 
  - 100 BGP route limit in AWS-TGW 
  - No routing controls 
  - No service insertion 
  - Poor visibility 

## AWS Networking
AWS Services

EC2

- Is the virtual machine, or as AWS calls them, instances.

IAM

- This is Identity and Access Management, and it allows users to get access to the instances or applications.

VPC

- This is a Virtual Private Cloud, which is essentially a data center in the cloud.

S3

- This allows users to access storage using a Public IP Address.

Direct Connect

- Helps users connect their on-premise Data Center to AWS. 

Route 53

- It’s the DNS System that allows users to build applications and different services using a -domain name resolution.

Global Accelerator

- Allows users to connect their remote branches to the closest point in the AWS System.

CloudFront

- This caches frequently used data. This is the CDN service.



### The AWS Cloud Computing Structure and Components

1. In a region…

   a. The first thing to do is to create a VPC using a specified CIDR. VPC’s are regional concepts in AWS.

2. Specify an AZ within the VPC.

3. Specify a subnet in the AZ.

   a. This will be part of the CIDR that was already created.
   b. There is now an AZ to subnet affinity.
   
4. Define or deploy the instances within the subnet.

5. Once the instances are defined, users must now define a level of security.

   a. Create a security group and apply it to multiple instances, or have one security group per instance.

6. Network ACL

   a. This is a stateless Access Control List that is applied at a subnet level.

7. Route-table and router

   a. Users have basic access to the route-table, but do not have access to the actual router.



### Some AWS Gateways

Internet Gateway (IGW)

- A service that provides internet connection to the Virtual Machine. 

Virtual Private Gateway (VGW)

- A service that allows users to build IPSec tunnels. 

NAT Gateway

- This gateway allows private subnets to connect to the Internet. 

Transit Gateway (TGW)

- This provides hub and spoke connectivity for the VPCs in the system. 

VPN Gateway

- This connects the on prem network to the VPC or creates a hub and spoke technology between third party VPN devices and the AWS VPN Gateway.

Customer Gateway (CGW)

- This is allows users to create a shell or definition for the gateway sitting on the on-prem site and then apply the configuration on the actual router. 

Direct Connect Gateway (DXGW)

- This is a service that allows multiple regions to connect to a physical Direct Connect circuit. 

The Architecture of the AWS Gateways

1. The region holds the VPC’s.
2. These VPC’s connect to their own TGW’s.
3. The CGW connects to the DXGW.
4. The DXGW connects to the TGW’s.

### AWS Transit Gateway (AWS TGW)

This allows many VPCs to talk to each other without VPC Peering.
The difference between using VPC Peering or using the TGW is that the VPC peering doesn’t allow transitive routing while the TGW does.
This native service supports multiple route tables and also has a VPN attachment type.
However, this has its own limitations.

The enterprises are responsible for programming the VPC routes manually when using the AWS platform without Aviatrix.
The IPSec Tunnel throughput is only about 1.25Gbps.
The scalability is a problem.
There is no overlapping IP support.

### How Aviatrix Solves for the Limitations of the AWS Transit Gateway

Aviatrix manages and controls the AWS TGW which removes the routing limitations.
Aviatrix takes care of the initial configuration of the routes and any updates.
It helps to simplify the BGP over Direct Connect.
Aviatrix provides network correctness and propagates all the on-prem routes to the VPCs.



## Azure Networking

The Azure Architecture

AD (Active Directory) Tenant

- This is a top-level domain that contains several subdomains. 

Subscriptions

- These subscriptions are located in the subdomains.
- They allow users to access the services in Azure.
- A subscription is similar to an account in AWS.

Resource Groups

- Resource groups are located inside of the subscriptions.
- The resources are located inside of the resources groups, essentially as instances of the services.
- Some resources include Virtual Machines, Addresses, and Disks.
- There are no similar concepts to resource groups in AWS.

### The Azure Networking Components

VNet

- This is the fundamental building block in Azure.
- It is similar to a VPC in AWS.

Availability Zone

- Essentially a Data Center located in a region.

Network Security Group

- A stateful firewall layer.
- Similar to an AWS Security Group

Public and Private IP Addresses

- In Azure, when users create VMs and specify a Public IP Address, it is automatically able to talk to the outside world.
- AWS requires an Internet Gateway in order to achieve this.

Virtual Network Gateway

There are two types of these in Azure. 

VPN Gateway

- This connects on-prem to the VNet using the public internet.
- It must be deployed in a Gateway Subnet.
- A VPN Gateway is similar to the VGW in AWS.
- There is also a local network gateway which is the on-prem entity. This local network gateway is similar to the CGW in AWS.

Express Route Gateway

- This is popular amongst enterprises connecting their on-prem to the cloud.
- This is essentially a dedicated private line for connecting VNets to the Data Centers.

VNet Peering

- This is a native construct provided by Azure.
- However, it is not scalable.

Routing: UDR, BGP and System Routes

- User Defined Routes are static and user defined.
- System Routes are programmed routes that users can't see in the Azure portal. 

Azure Load Balancer

- This is a L4 load balancer. Users can also use the Application Gateway for L7 load balancing.



### Transit in Azure

Azure also provides options for transit. It has three native options for inter and intra region transit. 

ExpressRoute Edge Router

- This is a common solution but has a lot of limitations.
- It's default is Any to Any for all spokes, meaning that they can all communicate.
- There is also a lack of visibility and control.
- This feature is not officially documented.
- It is a shared resource.

Network Virtual Appliance (NVA)

- Customers usually deploy a firewall or third part appliance in the NVA and use it as a transit solution.
- This solution isn't easily scalable and requires a lot of manual work.
- There is limited visibility and control.
- The management of the static routes is challenging for enterprises.

VNet Peering

- This is Microsoft's preferred option. However, even this solution is difficult to use.
- Users have to create 1-to-1 mapping.
- This solution also doesn't scale and the VNet peering needs to be broken to add components.

### Azure Virtual WAN
This is Microsoft's proposed solution for the limitations of the native transit solutions. The Azure Virtual WAN has some drawbacks, however.

This solution is not Multi-Cloud friendly.
Users must buy all the features in order to use this solution.
The hub location doesn't support third party devices.
Troubleshooting and visibility are limited.
There is no way to control the routing policy.
There is a 200 BGP route limit from Azure to on-prem.

## GCP Networking

Resource in GCP

Global 

- Can be accessed by any other resource, across regions and zones 

- Creating VPC is a global operation because a network is a global resource 

- Different from AWS and Azure because the VPC and routing is global, not within a region 

Regional 

- Can be accessed only by resources in the same region 

- Reserving an IP address is a regional operation 

Zonal  

- Can be accessed only by resources in the same zone 

- Disks can be attached to computers in the same zone



### GCP Projects

- Projects are the fundamental organizational structure 
- GCP resources must belong to a project 
- Made up of settings, permissions, and other metadata that describe applications 
- Contains the computing, storage, and networking resources 
- A project can’t access other projects resources unless you use 
  - Shared VPC 
  - VPC Network Peering 

- GCP Regions and Zones 
- VPC/Subnets 
- VPC Peering 
- Implicit Routing 
- VPN Gateway 

### Basic GCP Network Components 
VPC Network 
- Global Routing: 
  - VPC is a global resource 
  - All the subnets irrespective of region are inherently routable within a VPC 
- Subnets/CIDR are a regional resource  
- Projects can contain multiple VPC networks 

Routes in GCP 
- Routes created by GCP for users are system generated routes 
  - Default route 
  - Subnet gateway 
- User Defined Route 
  - Static Routing 
  - Dynamic Routing 

Transit (Inter-VPC) Networking

- Lacks native transit solution to interconnect VPC’s 
  - VPC peering preferred 
  - Preaching single VPC 
- VPC Peering 
  - Same qualities as other CSP’s 
  - All preprogrammed routes from the two VPC’s are announced to each other 
  - Used to connect multiple VPC’s  
  - Non-transitive 

Cloud Interconnect 

- Connect your On-Prem network to your VPC network through a private connection 
- Limitation: Not encrypted 
- **Dedicated Interconnect** 
  - Enables users to connect existing network to the VPC network through a highly available, low latency, enterprise grade connection 

## OCI
OCI Hierarchy
Tenancy
Compartments 
Resources

### OCI Important Services
Compute Instances
IAM
VCN Virtual Cloud Network
Block Volume
FastConnect - private peering, public peering
DNS Zone Management

Security Lists- default, custom
Network Security Group
Route Table- default, custom

DRG - Dynamic Routing Gateway (DRG 2.0)
Internet Gateway
Service Gateway
NAT Gateway


## Multi-Cloud Transit Routing and Networking
The cloud providers allow VNet Peering when it comes to native VPCs or the VNet, which leads to problems with scale, as this model promotes a full mesh structure. This is because the VPC’s are not transitive, and thus leads to more complexities and difficulties when managing and updating the routes. Ultimately, there is no network correctness.

Currently, only AWS and Azure have transit solutions, and even those come with a handful of limitations.

AWS

- AWS provides the AWS Transit Gateway as its solution. There is a lack of visibility as users are not able to log into the software and control the security or even create two TGW’s in one region.

Azure

- Azure has multiple transit solutions, such as the ER Edge Router, the Azure Firewall, and the Virtual WAN, but each solution has its own specific limitations as well, such as a lack of visibility.

Non-Aviatrix 3rd Party Transit Network solutions

Despite there being other 3rd party transit network solutions on the market, such as Cisco and Palo Alto Networks, none of them come close to making transitive routing easier.

Users must manage the IPSec Tunnel with a throughput dependent on 1.25GBps per tunnel. Furthermore, instance sizes are not in the enterprise’s control and the BGP must also be managed by the user. All of these additional burdens just continue to increase the complexities and decrease the network correctness.

### Characteristics of Aviatrix Transit Architecture

Similar to a house, the foundation of this architecture must be secure and strong. This means that the architecture must be well rounded, secure and encrypted, and centrally managed.

The Aviatrix controller manages the interactions and builds the transit in a matter of minutes.
This transit can be made throughout all CSPs.
Enterprises can use CoPilot to get the maximum visibility in one centralized area.

BGP Route Appoval - This feature allows users to approve any BGP-learned route from on-prem into the cloud network. 



## Aviatrix Platform Components

- Aviatrix Controller 
  - Your instance- you use, keep, and manage 
  - Management, orchestration, and control plane 
- Aviatrix Gateways 
  - Provides normalized data plane 

- Native Cloud Constructs 
  - Control plane allows management of native cloud constructs 
  - If they fall short, Aviatrix can provide advanced network and security capabilities through gateways 

### Core Features  

- Controller 
  - Provides intelligent orchestration & control 
  - Multi account / Multi subscription 
- Advanced Networking Multi-Region and Multi-Cloud  
- High Performance Encryption  
  - In the cloud  
  - With on-prem DC using cloud hardware appliance 
- Site to Cloud/ On-Prem 
- CloudWAN 
- Smart SAML User VPN 
- Secure Egress and Ingress 
- Firewall Network (FireNet) 
- Operational Tools 
  - Ex: Flight Path, Packet Capture, Trace Route, etc. 
- Licensing options 
  - Pay as you go 
  - Bring your own license 

## Cloud Security

### High Performance Encryption
A typical data center or cloud deployment can be divided into different planes. For an enterprise, using AWS Direct Connect and Azure Express Route works to connect the data center to the cloud without encryption. To encrypt this connection, users have the option to create an IPSec Tunnel which limits the throughput to only 1.25Gbps. This is a problem in cases where the customer purchases a 10G Direct Connect Circuit/ExpressRoute, but is only able to utilize 1.25Gbps of it, which is immensely smaller than it should be.

Aviatrix has a method of encryption without the limitations of the IPSec Tunnel, called High Performance Encryption. HPE is able to accomplish this by utilizing all the CPU cores, which achieves line rate encryption throughput.

This HPE is integrated into the Transit Network solution by building a high performance encryption tunnel over private network links.

### Firewall Network - FireNet
Aviatrix provides a solution that allows enterprises to apply the firewall in any possible area or direction.

- In VPCs/VNets inside a cloud.
- If the traffic is going towards the Internet (Egress).
- If the traffic is coming from the Internet (Ingress).
- Traffic coming in from the Data Center.
- Any branch and partner site connected to the cloud.
- The traffic sent to another cloud.


This solution is unique because there are many problems that come with inserting firewalls in Cloud Networks:

The Firewall Vendors typically provide the firewalls but the customer is responsible for managing and routing the traffic to the firewalls themselves and managing it through and through.

The Cloud Providers often don’t have L7 Firewalls, and if they do, the features are limited in terms of those required for an enterprise. Similar to the solutions provided by the Firewall Vendors, the customer is responsible for routing the traffic to the firewalls themselves and managing it through and through.



### The Azure Native Solution

Azure has a native firewall, but the problem is that when users want to send the traffic from one spoke VNET to another, they must make sure the UDR’s are done properly so that it can point to the VIP of the load balancer correctly, go into the Firewall, and then to the destination. Furthermore, when running in Active Active mode, they must also deploy the SNAT so that the return traffic doesn’t hit the wrong firewall. Essentially, manual configuration is required and it takes high complexity to manage changes in Azure. The same challenges can be seen in a 3rd party firewall solution, with the additional challenge of having to configure the SNAT as well.

### The AWS Native Solutions

The VPC Attachment Model

One of the problems with AWS’s Native Solution is that the Active Standby can only be deployed in one AZ, even though the solution is quite expensive with only one VM-Series being active.

Because AWS has a TGW, it is possible to attach to the VPC to get more throughput. However, all of these routing tables also have to be manually configured so that the traffic can go where it needs to be.

The IPSec VPN Model

In this model, the IPSec Tunnel is built towards the Firewall, giving users 1.25Gbps throughput, which effectively gives users about 600MBps because it has to go back on the same VPN link. The throughput will reduce as more advanced features are used. Source NAT is another challenge.

### Aviatrix Transit FireNet Solutions

Aviatrix solves for the shortcomings of the native solutions provided by the CSPs by deploying the firewalls into the Transit VPC. This helps to utilize the full performance of the firewalls, simplifies the management of all the networking elements because Aviatrix takes care of it, and allows enterprises to scale with ease and no compromise in visibility.

This architecture consists of Active Active Gateways in an active mesh, and also provides a scale up option. This solution has a throughput of 70 Gbps, compared to the AWS TGW throughput of 1.25Gbps.

### Aviatrix AWS TGW FireNet

When deploying the Aviatrix solution with the AWS TGW, all users have to do is attach the Transit VPC to the AWS TGW.

This automatically sends the traffic to the Fire Wall without any additional components, such as an IPSec tunnel, or a SNAT. This also doesn’t require a BGP.

Some benefits of this approach compared to the native one include the maintenance of the session stickiness, load balancing provided by Aviatrix, and integration with Panorama.

### Aviatrix Security Domain
This allows enterprises to create policies and then implement network segmentation. For example, users can put all their production workloads in one production domain and all their developer workloads in another developer domain. This will allow the enterprise to disable traffic between the two domains, or set security policies to manage the communication.

### Aviatrix Azure Native Peering FireNet

The Aviatrix FireNet Solution works in Azure almost exactly how it works in AWS, with the only difference being that there will be no encryption without the Aviatrix Gateways in the Spoke VNets.

### FQDN/URL based Egress Filtering
Because Workloads require Internet access, it is essential to keep the rest of the cloud secure when these applications and instances/VMs pull this information from the Internet. 

A NAT Gateway is supposed to perform this task safely and efficiently. However, this gateway has limitations because it only allows enterprises to configure rules based on the IP (Source, destination, port, and protocol). There is no way to write rules based on the URL or FQDN instead.

Aviatrix provides the FQDN Egress Filter service to make writing rules based on the URL or FQDN possible.

Fully Qualified Domain Name (FQDN) Egress Filter

- Go on Controller
- Create profiles
- Create rules
- Push it to the gateways that you want to have access to the Internet

This can be used on public or private subnets to get the enhanced security that the cloud needs. This works by replacing the NAT Gateway with an Aviatrix Gateway inside of the VPC.

### Ingress Security (Guard Duty + VPC Ingress Routing)

The AWS GuardDuty is one example of a service that provides ingress security. It is a threat detection service that uses a database to detect malicious IP addresses. However, this service doesn’t take any action on the malicious activity that it finds.

The Aviatrix Guard Duty Enforcement service extends the capabilities of AWS GuardDuty by taking the information from AWS GuardDuty and then automatically triggering the controller into blocking the IP Address, thus contributing to an updated filtering table. Simply, the AWS GuardDuty service is an intrusion detection service, while the Aviatrix Guard Duty Enforcement service is an intrusion prevention service.

## Cloud Access
### Smart SAML User VPN
User VPN Features: 

- Connects users to public cloud resources 

- No need to backhaul to On-Prem DC first 

- Least latency accessing the cloud resources 

In the smart SAML VPN architecture, you deploy a VPC, or VNET in the cloud access layer. From there you enter the transit network, and then based on the routing you have, the destination can be reached.

This solution is profile based. The partners, contractors, and employees have their own profiles and go to only their respective VPC’s/VNETS. 

Aviatrix provides isolation amongst the personas. This solution also allows connection to different Enterprise Identity Provider (IDP). The security rules apply automatically when the user is active but otherwise are removed from the gateway. The solution also supports both split and full tunnel modes. Furthermore, we provide a client for authentication with IDP, but any OpenVPN client is also supported.  

### Site to Cloud
Aviatrix supports connectivity between its Gateways in the cloud and on-premise routers using a feature called Site to Cloud.

Without this feature, the standard way of doing this includes only building an IPSec Tunnel, which is not easy to configure and doesn’t have a central place to operate and manage branches. Site to Cloud, in addition to the Aviatrix CloudN hardware appliance, makes it easier to connect to the on-prem data centers and allows connections to any site or cloud. Site to Cloud supports TCP and UDP tunnels, as well as overlapping IPs.

How to Use Site to Cloud:

Build an encrypted connection to the gateway by managing the end of the tunnel.
Provide a configuration template.

## Operations, Visibility, and Troubleshooting

### Operational Challenges within the Public Cloud

Evidential Data 

- When working with Cloud Service Providers customers often struggle to prove the providers faults/issues 

Unfamiliar Toolset

- Native cloud lacks familiar tools like ping, packet capture, and trace route 

Blackbox – No visibility 

- Native cloud constructs want users to believe that everything is always under control, and provide no visibility into logs, current state, routing tables, etc. 

Infrastructure as code 

- Solves agility problem but creates a support issue as tier-1 is not able to trouble shoot code problems  

A Flat World in Public Cloud

- Lack of hierarchy in the cloud, which means it’s hard to insert security, control, and visibility 

Tier-3 Becomes Tier-1 

- Frontline support teams don’t have the skill and tools in public cloud, requiring senior network engineers to assist with most support issues 

Scaling Out 

- Problems occur when the architecture scales out because it grows complex and becomes hard to troubleshoot 



### What is Aviatrix doing to solve these issues?

#### Aviatrix CoPilot

Aviatrix CoPilot provides a global operational view of your multi-cloud network. Enterprise IT teams use CoPilot’s dynamic topology mapping to maintain an accurate topology of their global multi-cloud networks, FlowIQ to analyze global network traffic flows and global heat maps and time series trend charts to easily pinpoint and troubleshoot traffic anomalies.

CoPilot is deployed as an all-in-one virtual appliance and is available on AWS, Azure, GCP, and OCI MarketPlaces. CoPilot works in tandem with Aviatrix Controller; in order to use CoPilot, you must have an operational Aviatrix Controller. Aviatrix Controller and CoPilot are not required to be collocated. It is possible to run them in separate VPCs/VNets or separate cloud providers (in multi-cloud environments).


- CoPilot dashboard provides complete visibility into your cloud operations 
- Shows which gateways are down 
- Provides map of where resources are being deployed 

- Virtual data center rundown (regions) 
- Percentage of gateways deployed per cloud 
- Gives ability to visualize topology  
  - Where the resources are 
  - How resources are connected 
  - Are they under compliance? 
  - Users can customize what they want to see within their cloud 
  - Can see all the information about gateways and run diagnostics 
- Flow IQ 
  - Categorizes and filters traffic 
  - Graphs provide ability to drill down into specific issues 
  - Geolocation puts all the traffic intelligence onto a map so users can easily visualize 

#### FlightPath 

Even for very simple issues such as connectivity between instances, the troubleshooting process is extremely long. FlightPath is a feature in the controller that provides users with a report of what happens between the instances. 
FlightPath is a troubleshooting tool. It retrieves and displays, in a side by side fashion, cloud provider’s network related information such as Security Groups, Route table and route table entries and network ACL. This helps you to identify connectivity problems.

You do not need to launch Aviatrix gateways to use this tool, but you need to create Aviatrix accounts so that the Controller can use the account credentials to execute cloud provider’s APIs to retrieve relevant information.

The information is arranged in three sections: Security Groups, Route table entries and Network ACL. The Security Group is what is associated with the instance, and both the route table and the Network ACL are associated with the subnet that the instance is deployed.

#### Packet Capture 
- You can select the gateway and the host, and you receive a report of what is happening with the transfer 
- We provide the option to download this as a pcap file 

#### Role Based Access Control 

- You can create different roles based on different personas 
- You can assign access to specific resources for different teams 

#### Multi-Cloud & Multi-Account 

- Single pane of glass to manage all cloud accounts 
- Support for AWS, Azure, GCP, etc using the same workflows, technology, and tools 
- Periodic account audits VPC Tracker  
- VPC report  

- Helps users manage network CIDR ranges in one place 
- No gateway launches required 
- On demand test to detect overlapping CIDR’s before creating new one  

#### Showback Functionality
- Shows deployment per account 
- Use case is to gain visibility of the Aviatrix usage per each account and helps to charge back to teams who are part of deployment 



#### VPC Tracker

The VPC Tracker collects and helps you manage your network CIDR ranges at a central place. This feature eliminates the need to keep an Excel sheet on all your AWS VPC/Azure VNet network address allocations.

No gateway launches are required.

#### Inline and hitless software upgrade
Aviatrix software upgrade happens inline without taking down the controller.

When upgrading a controller’s software, all gateways are upgraded with the new software at the same time. This is done by the controller pushing new software to gateways directly and automatically once requested.
