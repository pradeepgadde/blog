---

layout: single
title:  "RSVP Label Switched Path"
date:   2022-09-25 08:59:04 +0530
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

# Using the Traffic Engineering Database (TED)

Traffic Engineering extensions allow OSPF/IS-IS to advertise additional information like admin groups, BW related info.

Admin groups/link colors/tags
Maximum BW of the link, available BW by priority level, TE metric

TE Extensions are enabled by default in IS-IS, no extra config required.
The following Traffic engineering information is automatically advertised:
- Local IP Address
- Neighbor Address
- Current reservable BW (8 priority levels)
- Maximum reservable BW
- Administrative Groups

In OSPF, we need to manually enable TE with the following command. 

Existing neighbors are not disturbed. This command generates Type 10 (opaque) LSAs and they stay within a single area (area scope).

```sh
[edit]
root@juniper# set protocols ospf traffic-engineering
```

```sh
root@Juniper> show ospf database opaque-area
```

```sh
root@Juniper> show isis database <routername> extensive 
```

TE information is extracted from OSPF/IS-IS and reformatted for consistency, and placed into Traffic Engineering Database (TED).

The TED on each router is almost identical. 

```sh
root@Juniper> show ted database
```
To see all routers, their neighbors, and interface IP info, name 
```sh
root@Juniper> show ted database | except index
```

```sh
root@Juniper> show ted database extensive <routername>
```
Same information as in IS-IS Link state PDU but presented in a different format.

For example, instead of Administrative Group, we see `Color`.

TED nodes with a `0.0.0.0` address indicate a LAN.
A psuedonode added to make it easier for topology calculations.

To create an LSP that uses TED, we dont need any extra config;  Earlier when we dont need TED, we used `no-cspf`. Now, its not required. Thats the only difference in the LSP config.

```sh
[edit]
root@juniper# set protocols mpls label-switched-path <lsp-name> to 10.20.30.9
```
Now the ingress router calculates the exact path that this LSP should take.

The Ingress router sends ERO (Explicit Route Objects) in the Path message which contains the exact path that the router calculated using the TED.

Transit routers just follow this path, or reject the Path message if they can't follow.

```sh
root@Juniper> show mpls lsp name <lsp-name> detail
```
We will see `Computed ERO` and `Received RRO`.

EXPLICIT ROUTE object, lists each hop in order and also shows as Strict or loose.
Strict- listed hop should be directly connected. (loopback IPs also can be used)
Loose- can be many hops away

`Computed ERO (S [L] denotes strict [loose])`

To configure loose and strict hops, use a named path in the `edit protocols mpls path` hierarchy.

```sh
[edit]
root@juniper# set protocols mpls path <path-name> X.X.X.X strict
[edit]
root@juniper# set protocols mpls path <path-name> Y.Y.Y.Y loose
```

To apply the loose/strict hops, create an explicit `Primary` path, using your named path under the LSP defintion.

```sh
[edit]
root@juniper# set protocols mpls label-switched-path <lsp-name> to 10.20.30.9 primary <path-name>
```



