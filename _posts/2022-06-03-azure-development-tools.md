---
layout: single
title:  "Azure Software Development Process Tools and Services "
date:   2022-06-03 10:59:04 +0530
categories: Cloud
tags: Azure
show_date: true
classes: wide
header:
  teaser: /assets/images/microsoft-certified-fundamentals-badge.svg
author:
  name     : "Microsoft"
  avatar   : "/assets/images/azure.png"

sidebar:
  - title: "Blog"
   
    text: "Checkout other topics"
    nav: my-sidebar

---

Microsoft tools enable source-code management, continuous integration and continuous delivery (CI/CD), and automate the creation of testing environments.

There are three primary offerings:

## Azure DevOps Services

Azure DevOps Services is a suite of services that address every stage of the software development lifecycle.

- Azure Repos—a centralized source-code repository 
- Azure Boards— an agile project management suite that includes Kanban boards, reporting, and tracking ideas and work from high-level epics to work items and issues.
- Azure Pipelines—a CI/CD pipeline automation tool.
- Azure Artifacts— a repository for hosting artifacts, such as compiled source code
- Azure Test Plans—an automated test tool that can be used in a CI/CD pipeline

## GitHub and GitHub Actions
- `Git` is a decentralized source-code management tool.
- GitHub is a hosted version of `Git` that serves as the primary remote.

GitHub Actions enables workflow automation with triggers for many lifecycle events. 

Both Azure DevOps and GitHub allow public and private code repositories.

GitHub is a lighter-weight tool than Azure DevOps, with a focus on individual developers contributing to the open-source code.

Azure DevOps, on the other hand, is more focused on enterprise development, with heavier project-management and planning tools, and finer-grained access control.

## Azure DevTest Labs
Azure DevTest Labs provides an automated means of managing the process of building, setting up, and tearing down virtual machines (VMs) that contain builds of your software projects.

Anything you can deploy in Azure via an ARM template can be provisioned through DevTest Labs. 

