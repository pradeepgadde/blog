---
layout: single
title:  "Jenkins Distributed Builds"
date:   2022-09-08 011:58:04 +0530
categories: Automation
tags: Jenkins
show_date: true
classes: wide
header:
  teaser: /assets/images/jenkins.svg
author:
  name     : "Jenkins"
  avatar   : "/assets/images/jenkins.svg"

sidebar:
  - title: "Blog"
   
    text: "Checkout other topics"
    nav: my-sidebar

---

Distributed builds are builds that run on nodes other than the built-in controller node. The Jenkins controller serves HTTP requests and stores all important information related to the builds. Agents that run on nodes manage the task execution on behalf of the Jenkins controller and supply most of the computing power required to build and test the software.

![Jenkins]({{ site.url }}{{ site.baseurl }}/assets/images/jenkins-distributed-builds.png)

Image Courtesy: CloudBees University

Distributed Jenkins components

A distributed Jenkins system uses the following components:

The Jenkins controller is the Jenkins service itself, which is a webserver that also acts as a "brain" for deciding how, when, and where to run tasks
    
A node is a server where Jenkins runs build jobs on executors. Note that the Jenkins controller also runs on a node.
    
An executor is effectively a thread for execution of tasks
    
The agent is the tool that manages the executors on a remote node, on behalf of Jenkins.

On production systems, the node where the Jenkins controller runs should be configured with 0 executors to ensure that no builds run on the controller.