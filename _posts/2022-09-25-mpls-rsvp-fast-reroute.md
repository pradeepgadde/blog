---

layout: single
title:  "RSVP FastReroute"
date:   2022-09-25 13:59:04 +0530
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

# One-to-One Local Repair (Fast Reroute)

To protect against link failure and node failure, RSVP LSPs can create always-on backup LSPs. These backup paths start at the point of local repair and are only used for a very short duration (till the headend calculates the new path for the main LSP).

There are two ways of doing this.

1. One-to-one backup (`fast-reroute`)
2. Facility backup (`link-protection` or `node-link-protection`)

In one-to-one backup approach, each LSP creates a `detour` path at each hop. This causes RSVP state to increase by huge amount.

In facility backup approach, one single `bypass` LSP can protect many LSPs, scales much better in large networks.

Note, these local repair paths are only used temporarily. The device detecting the error, sends a PathErr to the ingress to calculates and signal a new path. Also, it indicates that the current LSP can continue to forward traffic unitl the new path is live.



One-to-One Backup paths:

for every one LSP, there is one detour, at every hop. Detours offer node protection, and fallback to link protection if the topology forces. Also, detours take the shortest path to the egress router (not mandatory to go back to the main LSP's path).

Detours for the same LSP can merge together to reduce some state.

To enable this one-to-one backups, add `fast-reroute` to the LSP. No additional configuration required on other routers.

```sh
[edit]
root@juniper# set protocols mpls label-switched-path <lsp-name> to X.X.X.X fast-reroute
```

Also, add an additional load-balancing policy to preinstall detour paths into the FIB.

To verify, use `show mpls lip ingress extensive` command and look for `FastReroute desired` and in the logs, look for `Fast-reroute Detour Up`.

To see Detour information, use the `show rsvp session detail` command and look for `Detour Explicit route`

Note, the detour does not have a unique name, uses the same name as the main LSP.

On tranist routers, we see something like this`Detour branch from X.X.X.X, to skip Y.Y.Y.Y, Up`

Detour count shows how many detours have been merged into one.

Default behaviour :

Admin groups are inherited but BW configuration is not.

Hop count has a  default value of six. 

We can change these defaults within the `fast-reroute` hierarchy of the LSP.
