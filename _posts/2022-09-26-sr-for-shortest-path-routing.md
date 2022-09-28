---

layout: single
title:  "Segment Routing as a replacement to LDP"
date:   2022-09-26 18:59:04 +0530
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
# SR for Shortest-Path Routing

If want to use SR as a replacement for LDP. there is no need for a controller, and no need for stacks of labels. It is very easy to create segment routed shortest-path LSPs.

When we use SR for shortest-path routing, there is just one transport label on the packet, which instructs how the packet should be forwarded to an egress router, via the metrically best path, as decided by OSPF/IS-IS.

Also, with SR it is possible to use the exact same transport label along the entire path.

For shortest-path routing using SR, we need a block of labels, either assigned automatically or manually. 

Node SID together with the starting label decides the MPLS label.

SRGB- Segment Routing Global Block, block advertised globally. (Starting label and block size)

Junos reserves a block of 4096 labels.

Also, we can configure every router to use the same block of labels. With this, same transport label is used for shortest-path routing for any node in the network.

Assign Node SIDs

```sh
[edit]
root@juniper# set protocols isis source-packet-routing node-segment <ipv4-index | ipv6-index> <value>
```

Node SID is a unique global number, that represents a particular router within the network.

When we configure Node SIDs on each router, `inet.3` table is populated automatically with a full mesh of shortest-path LSPs to every other router in the network, just like in LDP.

If we configure a Node SID, SRGB is created automatically by Junos.

`Node Segment Blocks Advertised: ` `Label-Range : [ X, Y ]`, `Size: Z`



Instead of advertising individual labels for each FEC in the network, ingress and transit routers can just look at the desitnation node SID, look at their next hop router's label block, and calculate the label that this next-hop router expects to receive.

For example, if Router-A wants to send a packet to Router-C, and Router-B is the physical next hop,

Router-B's starting label + Router-C's Node SID = MPLS label used by Router-A.



We can manually configure an SRGB, by choosing our own starting label and block size. Also, we can use the same label block on all routers. With this, each router uses the same transport label to get to any device.

```sh
[edit]
root@juniper# set protocols isis source-packet-routing srgb start-label <value>
[edit]
root@juniper# set protocols isis source-packet-routing srgb index-range <value>
```

Here is short demo of SR from Juniper Learning Bytes.

<iframe width="560" height="315" src="https://www.youtube.com/embed/HXs6keKpkK0" title="YouTube video player" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture" allowfullscreen></iframe>

