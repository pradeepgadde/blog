---
layout: single
title:  "Troubleshooting L2Circuits"
date:   2022-10-25 01:40:04 +0530
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

# Troubleshooting L2Circuits

Use `show l2circuit connections`

`OL -- no outgoing label`
`LD -- local site signaled down`



The Pseudowire Status TLV - special TLV that both PEs can negotiate

```sh
set protocols l2circuit neighbor <X.X.X.X> interface <ge-0/0/0.0> pseudowire-status-tlv
```

Refer to `PW status code:` in the `show l2circuit connections` output.



- no LSP to remote PE
- Firewall filters blocking LDP
- Encap mismatch
- VLAN mismatch
- incorrect VC ID
- Incorrect local interface 

In L2Circuit, the VLAN ID is advertised in the FEC!

`VM -- vlan id mismatch`

`Requested VLAN ID` and `Input VLAN ID` in the L2Circuit traceoptions

`NP -- interface h/w not present` Incorrect local interface

