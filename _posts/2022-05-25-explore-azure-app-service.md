---

layout: single
title:  "Explore Azure App Service"
date:   2022-05-25 04:59:04 +0530
categories: Cloud
tags: Azure
show_date: true
classes: wide
header:
  teaser: /assets/images/ms_build_cloud_skills.svg
author:
  name     : "Microsoft"
  avatar   : "/assets/images/azure.png"

sidebar:
  - title: "Blog"
   
    text: "Checkout other topics"
    nav: my-sidebar

---
This is the first one in the Build: Azure Developer Challenge.

In this post, let us learn about

Azure App Service key components and value.
How Azure App Service manages authentication and authorization.
Methods to control inbound and outbound traffic to your web app.
Deployong an app to App Service using Azure CLI commands.

Azure App Service is an HTTP-based service for hosting web applications, REST APIs, and mobile back ends.

Built-in auto scale support
Continuous integration/deployment support
App Service can also host web apps natively on Linux for supported application stacks. 

The languages, and their supported versions, are updated on a regular basis. You can retrieve the current list by using the following command in the Cloud Shell.

Let us launch the Cloud Shell from Azure Portal.

![Cloud Shell]({{ site.url }}{{ site.baseurl }}/assets/images/azure-cloud-shell-1.png)

![Cloud Shell]({{ site.url }}{{ site.baseurl }}/assets/images/azure-cloud-shell-2.png)

![Cloud Shell]({{ site.url }}{{ site.baseurl }}/assets/images/azure-cloud-shell-3.png)

```sh
Requesting a Cloud Shell.Succeeded. 
Connecting terminal...

Welcome to Azure Cloud Shell

Type "az" to use Azure CLI
Type "help" to learn about Cloud Shell

pradeep@Azure:~$ az

Welcome to Azure CLI!
---------------------
Use `az -h` to see available commands or go to https://aka.ms/cli.

Telemetry
---------
The Azure CLI collects usage data in order to improve your experience.
The data is anonymous and does not include commandline argument values.
The data is collected by Microsoft.

You can change your telemetry settings with `az configure`.


     /\
    /  \    _____   _ _  ___ _
   / /\ \  |_  / | | | \'__/ _\
  / ____ \  / /| |_| | | |  __/
 /_/    \_\/___|\__,_|_|  \___|


Welcome to the cool new Azure CLI!

Use `az --version` to display the current version.
Here are the base commands:

    account             : Manage Azure subscription information.
    acr                 : Manage private registries with Azure Container Registries.
    ad                  : Manage Azure Active Directory Graph entities needed for Role Based Access
                         Control.
    advisor             : Manage Azure Advisor.
    afd                 : Manage Azure Front Door Standard/Premium. For classical Azure Front Door,
                         please refer https://docs.microsoft.com/en-us/cli/azure/network/front-
                         door?view=azure-cli-latest.
    ai-examples         : Add AI powered examples to help content.
    aks                 : Manage Azure Kubernetes Services.
    ams                 : Manage Azure Media Services resources.
    apim                : Manage Azure API Management services.
    appconfig           : Manage App Configurations.
    appservice          : Manage App Service plans.
    aro                 : Manage Azure Red Hat OpenShift clusters.
    backup              : Manage Azure Backups.
    batch               : Manage Azure Batch.
    bicep               : Bicep CLI command group.
    billing             : Manage Azure Billing.
    bot                 : Manage Microsoft Azure Bot Service.
    cache               : Commands to manage CLI objects cached using the `--defer` argument.
    capacity            : Manage capacity.
    cdn                 : Manage Azure Content Delivery Networks (CDNs).
    cloud               : Manage registered Azure clouds.
    cognitiveservices   : Manage Azure Cognitive Services accounts.
    config              : Manage Azure CLI configuration.
    configure           : Manage Azure CLI configuration. This command is interactive.
    consumption         : Manage consumption of Azure resources.
    container           : Manage Azure Container Instances.
    cosmosdb            : Manage Azure Cosmos DB database accounts.
    databoxedge         : Support data box edge device and management.
    deployment          : Manage Azure Resource Manager template deployment at subscription scope.
    deployment-scripts  : Manage deployment scripts at subscription or resource group scope.
    deploymentmanager   : Create and manage rollouts for your service.
    disk                : Manage Azure Managed Disks.
    disk-access         : Manage disk access resources.
    disk-encryption-set : Disk Encryption Set resource.
    dla                 : Manage Data Lake Analytics accounts, jobs, and catalogs.
    dls                 : Manage Data Lake Store accounts and filesystems.
    dms                 : Manage Azure Data Migration Service (DMS) instances.
    eventgrid           : Manage Azure Event Grid topics, domains, domain topics, system topics
                         partner topics, event subscriptions, system topic event subscriptions and
                         partner topic event subscriptions.
    eventhubs           : Manage Azure Event Hubs namespaces, eventhubs, consumergroups and geo
                         recovery configurations - Alias.
    extension           : Manage and update CLI extensions.
    feature             : Manage resource provider features.
    feedback            : Send feedback to the Azure CLI Team.
    find                : I'm an AI robot, my advice is based on our Azure documentation as well as
                         the usage patterns of Azure CLI and Azure ARM users. Using me improves
                         Azure products and documentation.
    functionapp         : Manage function apps. To install the Azure Functions Core tools see
                         https://github.com/Azure/azure-functions-core-tools.
    group               : Manage resource groups and template deployments.
    hdinsight           : Manage HDInsight resources.
    identity            : Managed Identities.
    image               : Manage custom virtual machine images.
    interactive         : Start interactive mode. Installs the Interactive extension if not
                         installed already.
    iot                 : Manage Internet of Things (IoT) assets.
    keyvault            : Manage KeyVault keys, secrets, and certificates.
    kusto               : Manage Azure Kusto resources.
    lab                 : Manage Azure DevTest Labs.
    local-context       : Manage Local Context.
    lock                : Manage Azure locks.
    logicapp            : Manage logic apps.
    login               : Log in to Azure.
    logout              : Log out to remove access to Azure subscriptions.
    managed-cassandra   : Azure Managed Cassandra.
    managedapp          : Manage template solutions provided and maintained by Independent Software
                         Vendors (ISVs).
    managedservices     : Manage the registration assignments and definitions in Azure.
    maps                : Manage Azure Maps.
    mariadb             : Manage Azure Database for MariaDB servers.
    monitor             : Manage the Azure Monitor Service.
    mysql               : Manage Azure Database for MySQL servers.
    netappfiles         : Manage Azure NetApp Files (ANF) Resources.
    network             : Manage Azure Network resources.
    policy              : Manage resource policies.
    postgres            : Manage Azure Database for PostgreSQL servers.
    ppg                 : Manage Proximity Placement Groups.
    provider            : Manage resource providers.
    redis               : Manage dedicated Redis caches for your Azure applications.
    relay               : Manage Azure Relay Service namespaces, WCF relays, hybrid connections, and
                         rules.
    reservations        : Manage Azure Reservations.
    resource            : Manage Azure resources.
    rest                : Invoke a custom request.
    restore-point       : Manage restore point with res.
    role                : Manage user roles for access control with Azure Active Directory and
                         service principals.
    search              : Manage Azure Search services, admin keys and query keys.
    security            : Manage your security posture with Azure Security Center.
    servicebus          : Manage Azure Service Bus namespaces, queues, topics, subscriptions, rules
                         and geo-disaster recovery configuration alias.
    sf                  : Manage and administer Azure Service Fabric clusters.
    sig                 : Manage shared image gallery.
    signalr             : Manage Azure SignalR Service.
    snapshot            : Manage point-in-time copies of managed disks, native blobs, or other
                         snapshots.
    sql                 : Manage Azure SQL Databases and Data Warehouses.
    ssh                 : SSH into resources (Azure VMs, Arc servers, etc) using AAD issued openssh
                         certificates.
    sshkey              : Manage ssh public key with vm.
    staticwebapp        : Manage static apps.
    storage             : Manage Azure Cloud Storage resources.
    synapse             : Manage and operate Synapse Workspace, Spark Pool, SQL Pool.
    tag                 : Tag Management on a resource.
    term                : Manage marketplace agreement with marketplaceordering.
    ts                  : Manage template specs at subscription or resource group scope.
    upgrade             : Upgrade Azure CLI and extensions.
    version             : Show the versions of Azure CLI modules and extensions in JSON format by
                         default or format configured by --output.
    vm                  : Manage Linux or Windows virtual machines.
    vmss                : Manage groupings of virtual machines in an Azure Virtual Machine Scale Set
                         (VMSS).
    webapp              : Manage web apps.
pradeep@Azure:~$ 
```



```sh
pradeep@Azure:~$ az webapp list-runtimes --linux
Argument 'linux' has been deprecated and will be removed in a future release. Use '--os-type' instead.
[
  "DOTNETCORE:6.0",
  "DOTNETCORE:3.1",
  "NODE:16-lts",
  "NODE:14-lts",
  "PYTHON:3.9",
  "PYTHON:3.8",
  "PYTHON:3.7",
  "PHP:8.0",
  "PHP:7.4",
  "RUBY:2.7",
  "RUBY:2.7.3",
  "JAVA:11-java11",
  "JAVA:8-jre8",
  "JBOSSEAP:7-java11",
  "JBOSSEAP:7-java8",
  "TOMCAT:10.0-java11",
  "TOMCAT:10.0-jre8",
  "TOMCAT:9.0-java11",
  "TOMCAT:9.0-jre8",
  "TOMCAT:8.5-java11",
  "TOMCAT:8.5-jre8"
]
pradeep@Azure:~$ 
```

App Service on Linux does have some limitations:

- App Service on Linux is not supported on Shared pricing tier.
- You can't mix Windows and Linux apps in the same App Service plan.
- Historically, you could not mix Windows and Linux apps in the same resource group. However, all resource groups created on or after January 21, 2021 do support this scenario. Support for resource groups created before January 21, 2021 will be rolled out across Azure regions (including National cloud regions) soon.
- The Azure portal shows only features that currently work for Linux apps. As features are enabled, they're activated on the portal.

Azure App Service Plans

When you create an App Service plan in a certain region (for example, West Europe), a set of compute resources is created for that plan in that region. Whatever apps you put into this App Service plan run on these compute resources as defined by your App Service plan. Each App Service plan defines:

- Region (West US, East US, etc.)
- Number of VM instances
- Size of VM instances (Small, Medium, Large)
- Pricing tier (Free, Shared, Basic, Standard, Premium, PremiumV2, PremiumV3, Isolated)



The *pricing tier* of an App Service plan determines what App Service features you get and how much you pay for the plan. There are a few categories of pricing tiers:

Shared compute
Dedicated compute
Isolated
Consumption

In the **Free** and **Shared** tiers, an app receives CPU minutes on a shared VM instance and can't scale out. 

App Service supports both automated and manual deployment.

Automated deployment, or continuous integration, is a process used to push out new features and bug fixes in a fast and repetitive pattern with minimal impact on end users.

Azure supports automated deployment directly from several sources. The following options are available:

## Automated deployment

- **Azure DevOps**: You can push your code to Azure DevOps, build your code in the cloud, run the tests, generate a release from the code, and finally, push your code to an Azure Web App.
- **GitHub**: Azure supports automated deployment directly from GitHub. When you connect your GitHub repository to Azure for automated deployment, any changes you push to your production branch on GitHub will be automatically deployed for you.
- **Bitbucket**: With its similarities to GitHub, you can configure an automated deployment with Bitbucket.

## Manual deployment

There are a few options that you can use to manually push your code to Azure:

- **Git**: App Service web apps feature a Git URL that you can add as a remote repository. Pushing to the remote repository will deploy your app.
- **CLI**: `webapp up` is a feature of the `az` command-line interface that packages your app and deploys it. Unlike other deployment methods, `az webapp up` can create a new App Service web app for you if you haven't already created one.
- **Zip deploy**: Use `curl` or a similar HTTP utility to send a ZIP of your application files to App Service.
- **FTP/S**: FTP or FTPS is a traditional way of pushing your code to many hosting environments, including App Service.

Azure App Service provides built-in authentication and authorization support, so you can sign in users and access data by writing minimal or no code in your web app, API, and mobile back end, and also Azure Functions.

App Service uses federated identity, in which a third-party identity provider manages the user identities and authentication flow for you. The following identity providers are available by default:

Microsoft Identity Platform
Facebook
Google
Twitter
Any OpenID Connect provider

The authentication and authorization module runs in the same sandbox as your application code. When it's enabled, every incoming HTTP request passes through it before being handled by your application code.

By default, apps hosted in App Service are accessible directly through the internet and can reach only internet-hosted endpoints. But for many applications, you need to control the inbound and outbound network traffic.



