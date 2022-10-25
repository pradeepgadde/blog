---
layout: single
title:  "FEC 129 Pseudowires"
date:   2022-10-25 02:50:04 +0530
categories: Networking
tags: MPLS
show_date: true
classes: wide
header:
  teaser: /assets/images/junos.png
header:
  overlay_image: /assets/images/networking-3.jpg
  og_image: /assets/images/networking-3.jpg
  teaser: /assets/images/junos.png

author:
  name     : "Junos"
  avatar   : "/assets/images/junos.png"

sidebar:
  - title: "Topics"
    nav: my-sidebar

---

# FEC 129 Pseudowires

To get the best of both worlds (BGP +LDP) and have features like Auto discovery

It combines BGP and LDP
BGP for autodiscovery
LDP for signalling 

Defined in RFC 4447

FEC 128: PWid FEC Element, contains Virtual Circuit ID, requires explicit configuration of the remote PE; Does not understand Multi-tenancy;

FEC 129: The Generalized PWid FEC Element, works more like L2VPN
Source/Target Attachment Individual Identifier (SAII/TAII)
like source/remote Site ID of L2VPN
Attachment Group Identifier (AGI) is a general VPN Identifier, to denote customer/tenant

