---

layout: single
title:  "Segment Routing"
date:   2022-09-26 17:59:04 +0530
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
# Source Packet Routing in Networking (SPRING)

Segment routing also known as SPRING is a new way to create LSPs, offers a lot of features like shortest path routing (like LDP), traffic engineering (like RSVP), local repair (like both LDP and RSVP) etc.

There are multiple methods of deploying Segment Routing.
1. Using MPLS labels, called `SR-MPLS`
2. Using IPv6 headers, instead of MPLS labels, called `SRv6`

SR advertised label information using IS-IS/OSPF, so no extra adjacencies to maintain, no need to signal traffic-engineered LSPs.

One block of labels can represent every router in the network.

Every single router in the network is aware of what label each router expects to receive, when there is a need to forward traffic in a particular direction.



SR is becoming a popular replacement for LDP.

SR divides the network into segments, and each segment is given a `Segment ID(SID)`. 

Links and routers (nodes) are two examples of segments.

SID itself is used as the MPLS label or used to calculate the MPLS label.

All SIDs advertised in IS-IS/OSPF. OSPF and IS-IS were extended to carry this segment ID information.

- Adjacency SID: label allocated to each link running IS-IS/OSPF; One for each address family (IPv4, IPv6)
- Node SID: a unique number representing a router on the network. Node SIDs populate the `inet.3` table. Automatic full mesh of LSPs, just like LDP

Configuring Segment Routing:

1. Enable MPLs in both control and data plane
2. Add `source-packet-routing` to either OSPF/IS-IS

```sh
[edit]
root@juniper# set protocols ospf source-packet-routing
```

```sh
[edit]
root@juniper# set protocols isis source-packet-routing
```

Incase of Juniper MX devices, we need to make sure they are in `enhanced-ip` mode.

With the above configuration, devices start advertising Adjacency SIDs.

In the OSPF/IS-IS database we can see those SIDs, for example `Level 2 IPv4 Adj-SID: XX`. 

The Adjacency SID number is also the MPLS label, we can verify this in the `mpls.0` table, they are shown a `[L-ISIS/14]`. 

Each router knows exactly what label every router has allocated.

**Any router can build a stack of labels to specify an exact path.**

Instead of signalling and maintaining an explicit LSP, any router now has the ability to calculate a path, identify the labels that each hop in that path expects to receive, and then push a stack of labels onto the packet.

Each router in the path, can inspect just the top label, pop it, and forward the remaining stack to the next hop in the path.

Ingress router makes the decision of how to route a packet, and every other router in the network obeys that decision.

Manual static LSPs can be setup under `[edit protocols source-packet-routing]` by creating a named `segment-list` and apply it a manual LSP (`source-routing-path`).

Instead of manual setup, usually an external controller like Juniper Paragon Pathfinder is used to build stack of labels.



Each SID has some flags

- F: Family, set for IPv6, unset for IPv4
- V: Value
- L: Locally significant

