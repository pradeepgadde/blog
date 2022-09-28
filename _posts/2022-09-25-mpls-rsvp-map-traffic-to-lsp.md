---

layout: single
title:  "Map specific traffic to  an LSP"
date:   2022-09-25 17:59:04 +0530
categories: Networking
tags: MPLS
show_date: true
classes: wide
header:
  teaser: /assets/images/junos.png
author:
  name     : "Junos"
  avatar   : "/assets/images/junos.png"

sidebar:
  - title: "Blog"
   
    text: "Checkout other topics"
    nav: my-sidebar

---
# Use a policy to map specific traffic to an LSP
We need a way to tell our router what traffic should be mapped to which LSP. In Junos, this is done through routing policy.

Use any match condition for your policy, and use `install-nexthop lsp` action to map traffic to a particular LSP.

Apply the routing policy to the forwarding table as an export policy. It is done under `[edit routing-options forwarding-table]` hierarchy, similar to how we apply a load-balancing policy.

