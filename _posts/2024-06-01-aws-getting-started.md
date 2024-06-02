---

layout: single
title:  "AWS: Getting Started"
categories: Cloud
tags: AWS
classes: wide
show_date: true
header:
  overlay_image: /assets/images/aws-banner.png
  og_image: /assets/images/aws-banner.png
  teaser: /assets/images/aws.png
author:
  name     : "Amazon Web Services"
  avatar   : "/assets/images/aws.png"

sidebar:
  - title: "Topics"
    nav: my-sidebar

---
# Getting Started with AWS

## Overview of Amazon Web Services

In 2006, Amazon Web Services (AWS) began offering IT infrastructure services to businesses as web
services—now commonly known as cloud computing. Amazon Web Services offers a broad set of global cloud-based products including compute, storage, databases, analytics, networking, mobile, developer tools, management tools, IoT, security, and
enterprise applications: on-demand, available in seconds, with pay-as-you-go pricing. 

## Six advanatages of Cloud Computing

- Trade capital expense for variable expense
- Benefit from massive economies of scale
- Stop guessing capacity
- Increase speed and agility
- Stop spending money running and maintaining data centers 
- Go global in minutes

## Cloud Computing Models

- Infrastructure as a Service (IaaS)
- Platform as a Service (PaaS)
- Software as a Service (SaaS)

## Cloud Computing Deployment Models

- Cloud
- Hybrid
- On-premises (Private cloud)

## Global Infrastructure

AWS serves over a million active customers in more than 240 countries and territories. The AWS Cloud infrastructure is built around AWS Regions and Availability Zones. An AWS Region is a physical location in the world where we have multiple Availability Zones. Availability Zones consist of one or more discrete data centers, each with redundant power, networking, and connectivity, housed in separate facilities. These Availability Zones offer you the ability to operate production applications and databases that are more highly available, fault tolerant, and scalable than would be possible from a single data center. The AWS Cloud operates in 80 Availability Zones within 25 geographic Regions around the world, with announced plans for more Availability Zones and Regions.

## Security and Compliance

### Security

- Keep Your Data Safe

- Meet Compliance Requirements

- Save Money

- Scale quickly

### Compliance
The IT infrastructure that AWS provides to its customers is designed and managed in alignment with
best security practices and a variety of IT security standards. The following is a partial list of assurance
programs with which AWS complies:
• SOC 1/ISAE 3402, SOC 2, SOC 3
• FISMA, DIACAP, and FedRAMP
• PCI DSS Level 1
• ISO 9001, ISO 27001, ISO 27017, ISO 27018

 ## Amazon Web Services Cloud

- AWS Management Console
- AWS Command Line Interface
- Software Development Kits

##  AWS Certified Cloud Practitioner Domains

• Domain 1: Cloud Concepts (24% of scored content)
• Domain 2: Security and Compliance (30% of scored content)
• Domain 3: Cloud Technology and Services (34% of scored content)
• Domain 4: Billing, Pricing, and Support (12% of scored content)

## Domain 1: Cloud Concepts

### Define the benefits of the AWS Cloud
Value proposition of the AWS Cloud

• Understanding the economies of scale (for example, cost savings)
• Understanding the benefits of global infrastructure (for example, speed of deployment, global reach)
• Understanding the advantages of high availability, elasticity, and agility

### Identify design principles of the AWS Cloud
AWS Well-Architected Framework

• Understanding the pillars of the Well-Architected Framework (for example, operational excellence, security, reliability, performance efficiency, cost optimization, sustainability)
• Identifying differences between the pillars of the Well-Architected Framework

### Understand the benefits of and strategies for migration to the AWS Cloud
Cloud adoption strategies
Resources to support the cloud migration journey

• Understanding the benefits of the AWS Cloud Adoption Framework (AWS CAF) (for example, reduced business risk; improved environmental, social, and governance (ESG) performance; increased revenue; increased operational efficiency)
• Identifying appropriate migration strategies (for example, database replication, use of AWS Snowball)

### Understand concepts of cloud economics
Aspects of cloud economics
Cost savings of moving to the cloud

• Understanding the role of fixed costs compared with variable costs
• Understanding costs that are associated with on-premises environments
• Understanding the differences between licensing strategies (for example, Bring Your Own License [BYOL] model compared with included licenses)
• Understanding the concept of rightsizing
• Identifying benefits of automation (for example, provisioning and configuration management with AWS CloudFormation)
• Identifying managed AWS services (for example, Amazon RDS, Amazon Elastic Container Service [Amazon ECS], Amazon Elastic Kubernetes Service [Amazon EKS], Amazon DynamoDB)

## Domain 2: Security and Compliance 
### Understand the AWS shared responsibility model
AWS shared responsibility model
• Recognizing the components of the AWS shared responsibility model
• Describing the customer’s responsibilities on AWS
• Describing AWS responsibilities
• Describing responsibilities that the customer and AWS share
• Describing how AWS responsibilities and customer responsibilities can shift,
depending on the service used (for example, Amazon RDS, AWS Lambda, Amazon EC2)

### Understand AWS Cloud security, governance, and compliance concepts
AWS compliance and governance concepts
Benefits of cloud security (for example, encryption)
Where to capture and locate logs that are associated with cloud security
Identifying where to find AWS compliance information (for example, AWS Artifact)
• Understanding compliance needs among geographic locations or industries (for example, AWS Compliance)
• Describing how customers secure resources on AWS (for example, Amazon Inspector, AWS Security Hub, Amazon GuardDuty, AWS Shield)
• Identifying different encryption options (for example, encryption in transit, encryption at rest)
• Recognizing services that aid in governance and compliance (for example,
monitoring with Amazon CloudWatch; auditing with AWS CloudTrail, AWS
Audit Manager, and AWS Config; reporting with access reports)
• Recognizing compliance requirements that vary among AWS services

### Identify AWS access management capabilities
Identity and access management (for example, AWS Identity and Access Management [IAM])
Importance of protecting the AWS root user account
Principle of least privilege
• AWS IAM Identity Center (AWS Single Sign-On)
• Understanding access keys, password policies, and credential storage (for example, AWS Secrets Manager, AWS Systems Manager)
• Identifying authentication methods in AWS (for example, multi-factor authentication [MFA], IAM Identity Center, cross-account IAM roles)
• Defining groups, users, custom policies, and managed policies in compliance with the principle of least privilege
• Identifying tasks that only the account root user can perform
• Understanding which methods can achieve root user protection
• Understanding the types of identity management (for example, federated)

### Identify components and resources for security
Security capabilities that AWS provides
Security-related documentation that AWS provides

• Describing AWS security features and services (for example, security groups, network ACLs, AWS WAF)
• Understanding that third-party security products are available from AWS Marketplace
• Identifying where AWS security information is available (for example, AWS Knowledge Center, AWS Security Center, AWS Security Blog)
• Understanding the use of AWS services for identifying security issues (for example, AWS Trusted Advisor)

## Domain 3: Cloud Technology and Services 
### Define methods of deploying and operating in the AWS Cloud
• Different ways of provisioning and operating in the AWS Cloud
• Different ways to access AWS services
• Types of cloud deployment models
• Connectivity options

• Deciding between options such as programmatic access (for example, APIs, SDKs, CLI), the AWS Management Console, and infrastructure as code (IaC)
• Evaluating requirements to determine whether to use one-time operations or repeatable processes
• Identifying different deployment models (for example, cloud, hybrid, on- premises)
• Identifying connectivity options (for example, AWS VPN, AWS Direct Connect, public internet)

### Define the AWS global infrastructure
• AWS Regions, Availability Zones, and edge locations
• High availability
• Use of multiple Regions
• Benefits of edge locations
• AWS Wavelength Zones and AWS Local Zones

Describing relationships among Regions, Availability Zones, and edge locations
• Describing how to achieve high availability by using multiple Availability Zones
• Recognizing that Availability Zones do not share single points of failure
• Describing when to use multiple Regions (for example, disaster recovery, business continuity, low latency for end users, data sovereignty)
• Describing at a high level the benefits of edge locations (for example,
Amazon CloudFront, AWS Global Accelerator)

### Identify AWS compute services
AWS compute services
• Recognizing the appropriate use of different EC2 instance types (for example, compute optimized, storage optimized)
• Recognizing the appropriate use of different container options (for example, Amazon ECS, Amazon EKS)
• Recognizing the appropriate use of different serverless compute options (for example, AWS Fargate, Lambda)
• Recognizing that auto scaling provides elasticity
• Identifying the purposes of load balancers

### Identify AWS database services
AWS database services
• Database migration
• Deciding when to use EC2 hosted databases or AWS managed databases
• Identifying relational databases (for example, Amazon RDS, Amazon Aurora)
• Identifying NoSQL databases (for example, DynamoDB)
• Identifying memory-based databases
• Identifying database migration tools (for example AWS Database Migration Service [AWS DMS], AWS Schema Conversion Tool [AWS SCT])

### Identify AWS network services
• Identifying the components of a VPC (for example, subnets, gateways)
• Understanding security in a VPC (for example, network ACLs, security groups)
• Understanding the purpose of Amazon Route 53
• Identifying edge services (for example, CloudFront, Global Accelerator)
• Identifying network connectivity options to AWS (for example AWS VPN,
Direct Connect)

### Identify AWS storage services
• Identifying the uses for object storage
• Recognizing the differences in Amazon S3 storage classes
• Identifying block storage solutions (for example, Amazon Elastic Block Store [Amazon EBS], instance store)
• Identifying file services (for example, Amazon Elastic File System [Amazon EFS], Amazon FSx)
• Identifying cached file systems (for example, AWS Storage Gateway)
• Understanding use cases for lifecycle policies
• Understanding use cases for AWS Backup

### Identify AWS artificial intelligence and machine learning (AI/ML) services and analytics services

Understanding the different AI/ML services and the tasks that they
accomplish (for example, Amazon SageMaker, Amazon Lex, Amazon Kendra)
• Identifying the services for data analytics (for example, Amazon Athena,
Amazon Kinesis, AWS Glue, Amazon QuickSight)

### Identify services from other in-scope AWS service categories
Application integration services of Amazon EventBridge, Amazon Simple
Notification Service (Amazon SNS), and Amazon Simple Queue Service
(Amazon SQS)
• Business application services of Amazon Connect and Amazon Simple Email Service (Amazon SES)
• Customer engagement services of AWS Activate for Startups, AWS IQ, AWS Managed Services (AMS), and AWS Support
• Developer tool services and capabilities of AWS AppConfig, AWS Cloud9, AWS CloudShell, AWS CodeArtifact, AWS CodeBuild, AWS CodeCommit, AWS CodeDeploy, AWS CodePipeline, AWS CodeStar, and AWS X-Ray
• End-user computing services of Amazon AppStream 2.0, Amazon WorkSpaces, and Amazon WorkSpaces Web
• Frontend web and mobile services of AWS Amplify and AWS AppSync
• IoT services of AWS IoT Core and AWS IoT Greengrass

Choosing the appropriate service to deliver messages and to send alerts and
notifications
• Choosing the appropriate service to meet business application needs
• Choosing the appropriate service for AWS customer support
• Choosing the appropriate option for business support assistance
• Identifying the tools to develop, deploy, and troubleshoot applications
• Identifying the services that can present the output of virtual machines
(VMs) on end-user machines
• Identifying the services that can create and deploy frontend and mobile
services
• Identifying the services that manage IoT devices

## Domain 4: Billing, Pricing, and Support 
### Compare AWS pricing models
Compute purchasing options (for example, On-Demand Instances, Reserved
Instances, Spot Instances, Savings Plans, Dedicated Hosts, Dedicated
Instances, Capacity Reservations)
• Data transfer charges
• Storage options and tiers
• Identifying and comparing when to use various compute purchasing options
• Describing Reserved Instance flexibility
• Describing Reserved Instance behavior in AWS Organizations
• Understanding incoming data transfer costs and outgoing data transfer costs
(for example, from one Region to another Region, within the same Region)
• Understanding different pricing options for various storage options and
tiers

### Understand resources for billing, budget, and cost management
• Billing support and information
• Pricing information for AWS services
• AWS Organizations
• AWS cost allocation tags

• Understanding the appropriate uses and capabilities of AWS Budgets, AWS Cost Explorer, and AWS Billing Conductor
• Understanding the appropriate uses and capabilities of AWS Pricing Calculator
• Understanding AWS Organizations consolidated billing and allocation of costs
• Understanding various types of cost allocation tags and their relation to billing reports (for example, AWS Cost and Usage Report)

### Identify AWS technical resources and AWS Support options
• Resources and documentation available on official AWS websites
• AWS Support plans
• Role of the AWS Partner Network, including independent software vendors and system integrators
• AWS Support Center

• Locating AWS whitepapers, blogs, and documentation on official AWS websites
• Identifying and locating AWS technical resources (for example AWS Prescriptive Guidance, AWS Knowledge Center, AWS re:Post)
• Identifying AWS Support options for AWS customers (for example, customer service and communities, AWS Developer Support, AWS Business Support, AWS Enterprise On-Ramp Support, AWS Enterprise Support)
• Identifying the role of Trusted Advisor, AWS Health Dashboard, and the
AWS Health API to help manage and monitor environments for cost
optimization
• Identifying the role of the AWS Trust and Safety team to report abuse of AWS resources
• Understanding the role of AWS Partners (for example AWS Marketplace,
independent software vendors, system integrators)
• Identifying the benefits of being an AWS Partner (for example, partner training and certification, partner events, partner volume discounts)
• Identifying the key services that AWS Marketplace offers (for example, cost management, governance and entitlement)
• Identifying technical assistance options available at AWS (for example, AWS Professional Services, AWS Solutions Architects)





