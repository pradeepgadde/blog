---

layout: single
title:  "RSVP LSP Bandwidth"
date:   2022-09-25 09:59:04 +0530
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

# Reserving BW on RSVP LSPs
By default, all types of traffic go over an LSP. If required, we can map specific traffic to specific LSPs using a policy.

Each LSP might have a different BW requirement. By default, it is possible to have many high-BW LSPs take a similar path. If that is the case, all LSPs on the path may experience packet loss.

RSVP LSPs can reserve BW along their path.
Also, we can mark certain LSPs as higher priority. This causes low-priority LSPs to finder alternate paths (may be longer ones).

This priority system comes into picture when there is not enough BW on a certain link.

RSVP BW reservations are used only while calculating the LSP path. It is not a policer that limits and drops traffic. Once the LSP is up, it can send as much traffic as it likes.

By using RSVP BW reservations, we can avoid BW congestion before it happens, by placing differen LSPs into different paths.

BW reservations can be either 
- manual 
- automatic

For example, if we need 500Mbps on a 1G link, we can addthe `bandwidth` statement to the LSP definition.

```sh
[edit]
root@juniper# set protocols mpls label-switched-path <lsp-name> to 10.20.30.9 bandwidth 500m
```
In the output of 
```sh
root@juniper> show mpls lsp name <lsp-name> detail
```
It shows , `Bandwidth: 500m`.

We can view BW reservations with

```sh
root@juniper> show rsvp interface
```

`Static BW` : 1000Mbps
`Available BW`: 500Mbps
`Reserved BW`: 500Mbps
`Highwatermark`: 500Mbps (Highest amount of total BW that has ever been reserved on this interface)

LSPs are unidirectional so the BW allocations happen in the direction of the LSP only.

If we look at the TED, an LSP reserves BW at every priority level.

By default, every single LSP is of equally best priority and reserves BW at every priority level (0 to 7).

Any link that cannot statisfy the BW requirements are pruned from the topology and then the metrically best path is chosen.

By default, an interface offers its full BW, however if required, we can oversubscribe or undersubscribe.

Two methods to change the interface BW

- Subscription : percentage
- Bandwidth : direct number

The change can be verified using the `show rsvp interface` command.

To monitor LSPs traffic levels, we can use the `monitor label-switched-path <lsp-name>` command. Other option is to use an external monitoring system like Paragon Pathfinder.





