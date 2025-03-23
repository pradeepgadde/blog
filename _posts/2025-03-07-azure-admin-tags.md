---

layout: single
title:  "Azure Resource Tags"
categories: Cloud
tags: Azure
show_date: true
classes: wide
header:
  teaser: /assets/images/az104.png
author:
  name     : "Microsoft"
  avatar   : "/assets/images/az104.png"

sidebar:
  - title: "Blog"
   
    text: "Checkout other topics"
    nav: my-sidebar

---

# Azure Resource Tags

Tags are name/value pairs that enable you to categorize resources  and view consolidated billing by applying the same tag to multiple  resources and resource groups. Tag names are case insensitive, but tag  values are case sensitive.

Do not enter names or values that could make your resources less secure  or that contain personal/sensitive information because tag data will be  replicated globally.

You can apply tags to your Azure resources, resource groups, and subscriptions but not to management groups.

Resources don't inherit the tags you apply to a resource group or a subscription. 

Use tags to group your billing data. For example, if you're running  multiple virtual machines for different organizations, use the tags to  group usage by cost center. You can also use tags to categorize costs by runtime environment, including the billing usage for virtual machines  running in the production environment.

Not all resource types support tags.

Each resource type might have specific requirements when working with tags.

Each resource, resource group, and subscription can have a maximum of 50 tag name-value pairs. If you need to apply more tags than the maximum  allowed number, use a JSON string for the tag value. The JSON string can contain many of the values that you apply to a single tag name. A  resource group or subscription can contain many resources that each have 50 tag name-value pairs.



Use [Azure Policy](https://learn.microsoft.com/en-us/azure/governance/policy/overview) to enforce tagging rules and conventions. By creating a policy, you  avoid the scenario of resources being deployed to your subscription that don't have the expected tags for your organization. Instead of manually applying tags or searching for resources that aren't compliant, create a policy that automatically applies the needed tags during deployment.  With the new [Modify](https://learn.microsoft.com/en-us/azure/governance/policy/concepts/effects#modify) effect and a [remediation task](https://learn.microsoft.com/en-us/azure/governance/policy/how-to/remediate-resources), you can also apply tags to existing resources. 
