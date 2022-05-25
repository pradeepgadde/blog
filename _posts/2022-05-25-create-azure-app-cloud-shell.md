---

layout: single
title:  "Create a static HTML web app by using Azure Cloud Shell"
date:   2022-05-25 05:59:04 +0530
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

In this post. let us look at how to deploy a basic HTML+CSS site to Azure App Service by using the Azure CLI `az webapp up` command. We will then update the code and redeploy it by using the same command.

The `az webapp up` command makes it easy to create and update web apps. When executed it performs the following actions:
Create a default resource group.
Create a default app service plan.
Create an app with the specified name.
Zip deploy files from the current working directory to the web app.

In the Cloud Shell, create a directory and then navigate to it.

```sh
pradeep@Azure:~$ mkdir htmlapp
pradeep@Azure:~$ cd htmlapp/
pradeep@Azure:~/htmlapp$ 
```
Run the following `git` command to clone the sample app repository to your htmlapp directory.

```sh
pradeep@Azure:~/htmlapp$ git clone https://github.com/Azure-Samples/html-docs-hello-world.git
Cloning into 'html-docs-hello-world'...
remote: Enumerating objects: 50, done.
remote: Counting objects: 100% (4/4), done.
remote: Compressing objects: 100% (4/4), done.
remote: Total 50 (delta 0), reused 2 (delta 0), pack-reused 46
Unpacking objects: 100% (50/50), done.
pradeep@Azure:~/htmlapp$ 
```

Change to the directory that contains the sample code and run the `az webapp up` command. 

```sh
pradeep@Azure:~/htmlapp$ cd html-docs-hello-world
pradeep@Azure:~/htmlapp/html-docs-hello-world$ az webapp up --location southasia --name pradeep-webapp  --html
The webapp 'pradeep-webapp' doesn't exist
Creating Resource group 'gaddepradeep_rg_6671' ...
(LocationNotAvailableForResourceGroup) The provided location 'southasia' is not available for resource group. List of available regions is 'centralus,eastasia,southeastasia,eastus,eastus2,westus,westus2,northcentralus,southcentralus,westcentralus,northeurope,westeurope,japaneast,japanwest,brazilsouth,australiasoutheast,australiaeast,westindia,southindia,centralindia,canadacentral,canadaeast,uksouth,ukwest,koreacentral,koreasouth,francecentral,southafricanorth,uaenorth,australiacentral,switzerlandnorth,germanywestcentral,norwayeast,jioindiawest,westus3,swedencentral,australiacentral2'.
Code: LocationNotAvailableForResourceGroup
Message: The provided location 'southasia' is not available for resource group. List of available regions is 'centralus,eastasia,southeastasia,eastus,eastus2,westus,westus2,northcentralus,southcentralus,westcentralus,northeurope,westeurope,japaneast,japanwest,brazilsouth,australiasoutheast,australiaeast,westindia,southindia,centralindia,canadacentral,canadaeast,uksouth,ukwest,koreacentral,koreasouth,francecentral,southafricanorth,uaenorth,australiacentral,switzerlandnorth,germanywestcentral,norwayeast,jioindiawest,westus3,swedencentral,australiacentral2'.
pradeep@Azure:~/htmlapp/html-docs-hello-world$ 
```
I entered the incorrect location `southasia` so an error message is shown along with the solution. It lists all possible values.

Let us run this time with correct location `southindia`.

```sh
pradeep@Azure:~/htmlapp/html-docs-hello-world$ az webapp up --location southindia --name pradeep-webapp  --html
The webapp 'pradeep-webapp' doesn't exist
Creating Resource group 'gaddepradeep_rg_1844' ...
Resource group creation complete
Creating AppServicePlan 'gaddepradeep_asp_9952' ...
Creating webapp 'pradeep-webapp' ...
Configuring default logging for the app, if not already enabled
Creating zip with contents of dir /home/pradeep/htmlapp/html-docs-hello-world ...
Getting scm site credentials for zip deployment
Starting zip deployment. This operation can take a while to complete ...
Deployment endpoint responded with status code 202
You can launch the app at http://pradeep-webapp.azurewebsites.net
Setting 'az webapp up' default arguments for current directory. Manage defaults with 'az configure --scope local'
--resource-group/-g default: gaddepradeep_rg_1844
--sku default: F1
--plan/-p default: gaddepradeep_asp_9952
--location/-l default: southindia
--name/-n default: pradeep-webapp
{
  "URL": "http://pradeep-webapp.azurewebsites.net",
  "appserviceplan": "gaddepradeep_asp_9952",
  "location": "southindia",
  "name": "pradeep-webapp",
  "os": "Windows",
  "resourcegroup": "gaddepradeep_rg_1844",
  "runtime_version": "-",
  "runtime_version_detected": "-",
  "sku": "FREE",
  "src_path": "//home//pradeep//htmlapp//html-docs-hello-world"
}
pradeep@Azure:~/htmlapp/html-docs-hello-world$ 
```

Open a browser and navigate to the app URL (http://http://pradeep-webapp.azurewebsites.net) and verify the app is running - take note of the title at the top of the page. Leave the browser open on the app for the next section.

![Azure Web App]({{ site.url }}{{ site.baseurl }}/assets/images/azure-webapp-1.png)

In the Cloud Shell, type `code index.html` to open the editor. In the <h1> heading tag, change Azure App Service - Sample Static HTML Site to Azure App Service Updated - or to anything else that you'd like.

```sh
pradeep@Azure:~/htmlapp/html-docs-hello-world$ code index.html
```

```sh
pradeep@Azure:~/htmlapp/html-docs-hello-world$ cat index.html | grep Pradeep
    <title>Azure App Service - Sample Static HTML Site Updated by Pradeep</title>
          <h1>Azure App Service - Sample Static HTML Site Updated by Pradeep</h1>
pradeep@Azure:~/htmlapp/html-docs-hello-world$ 
```
Redeploy the app with the same `az webapp up` command. 

```sh
pradeep@Azure:~/htmlapp/html-docs-hello-world$ az webapp up --location southindia --name pradeep-webapp --html
Webapp 'pradeep-webapp' already exists. The command will deploy contents to the existing app.
Creating AppServicePlan 'gaddepradeep_asp_9952' ...
Creating zip with contents of dir /home/pradeep/htmlapp/html-docs-hello-world ...
Getting scm site credentials for zip deployment
Starting zip deployment. This operation can take a while to complete ...
Deployment endpoint responded with status code 202
You can launch the app at http://pradeep-webapp.azurewebsites.net
Setting 'az webapp up' default arguments for current directory. Manage defaults with 'az configure --scope local'
--resource-group/-g default: gaddepradeep_rg_1844
--sku default: F1
--plan/-p default: gaddepradeep_asp_9952
--location/-l default: southindia
--name/-n default: pradeep-webapp
{
  "URL": "http://pradeep-webapp.azurewebsites.net",
  "appserviceplan": "gaddepradeep_asp_9952",
  "location": "southindia",
  "name": "pradeep-webapp",
  "os": "Windows",
  "resourcegroup": "gaddepradeep_rg_1844",
  "runtime_version": "-",
  "runtime_version_detected": "-",
  "sku": "FREE",
  "src_path": "//home//pradeep//htmlapp//html-docs-hello-world"
}
pradeep@Azure:~/htmlapp/html-docs-hello-world$ 
```

![Azure Web App]({{ site.url }}{{ site.baseurl }}/assets/images/azure-webapp-2.png)



Azure App Service is a distributed system. The roles that handle incoming HTTP or HTTPS requests are called front ends. The roles that host the customer workload are called workers. All the roles in an App Service deployment exist in a multi-tenant network. Because there are many different customers in the same App Service scale unit, you can't connect the App Service network directly to your network.

Instead of connecting the networks, you need features to handle the various aspects of application communication. The features that handle requests to your app can't be used to solve problems when you're making calls from your app. Likewise, the features that solve problems for calls from your app can't be used to solve problems to your app.

There are a number of addresses that are used for outbound calls. The outbound addresses used by your app for making outbound calls are listed in the properties for your app. These addresses are shared by all the apps running on the same worker VM family in the App Service deployment. If you want to see all the addresses that your app might use in a scale unit, there's property called possibleOutboundAddresses that will list them.

To find the outbound IP addresses currently used by your app in the Azure portal, click Properties in your app's left-hand navigation.

![Azure Web App]({{ site.url }}{{ site.baseurl }}/assets/images/azure-webapp-3.png)

You can find the same information by running the following command in the Cloud Shell. They are listed in the Additional Outbound IP Addresses field.

```sh
pradeep@Azure:~/htmlapp/html-docs-hello-world$ az webapp show --resource-group gaddepradeep_rg_1844 --name pradeep-webapp  --query outboundIpAddresses --output tsv
104.211.225.167,104.211.228.24,104.211.224.166,104.211.223.200,104.211.227.251
pradeep@Azure:~/htmlapp/html-docs-hello-world$ 
```

To find all possible outbound IP addresses for your app, regardless of pricing tiers, run the following command in the Cloud Shell.
```sh
pradeep@Azure:~/htmlapp/html-docs-hello-world$ az webapp show --resource-group gaddepradeep_rg_1844 --name pradeep-webapp  --query possibleOutboundIpAddresses --output tsv
104.211.225.167,104.211.228.24,104.211.224.166,104.211.223.200,104.211.227.251,104.211.231.155,104.211.227.42,104.211.217.144
pradeep@Azure:~/htmlapp/html-docs-hello-world$ 
```
