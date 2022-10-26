---
layout: single
title: "VPLS Advanced Tpologies"
date:   2022-10-26 01:55:04 +0530
categories: Networking
tags: MPLS
show_date: true
classes: wide
header:
  overlay_image: /assets/images/networking-4.jpg
  og_image: /assets/images/networking-4.jpg
  teaser: /assets/images/junos.png

author:
  name     : "Junos"
  avatar   : "/assets/images/junos.png"

sidebar:
  - title: "Topics"
    nav: my-sidebar

---

# VPLS Advanced Topologies

Hub and Spoke VPLS instead of full mesh

Use `no-local-switching` to prevent VPLS local switching when two spoke CEs are connected to the same PE

```sh
set routing-instances <name> no-local-switching
```

By default, a BGP VPLS can host 65,534 sites

we can configure a `site-range` but not that useful!

LDP VPLS

Hierarchical VPLS

On the wire, LDP VPLS and LDP L2Circuit signaling is identical.

If we configure spokes as L2Circuit, and hub as VPLS, it works!



## BGP VPLS Multihoming



`bgp.vpls.0`

`<name>.vpls.0`

there is no `.l2id.0` table in VPLS unlike L2VPN

Same philosophy as L2VPN multihoming

<span style="color:red">VPLS Multihoming</span>

