---

layout: single
title:  "RSVP Node-Link-Protection"
date:   2022-09-25 14:59:04 +0530
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
# Facility Backup (Node-Link-Protection)
To protect against link failure and node failure, RSVP LSPs can create always-on backup LSPs. These backup paths start at the point of local repair and are only used for a very short duration (till the headend calculates the new path for the main LSP).

There are two ways of doing this.

    One-to-one backup (fast-reroute)
    Facility backup (link-protection or node-link-protection)

In one-to-one backup approach, each LSP creates a detour path at each hop. This causes RSVP state to increase by huge amount.

In facility backup approach, one single bypass LSP can protect many LSPs, scales much better in large networks.

Bypass LSPs are like a tunnel, traffic from multiple LSPs can be tunneled over the bypass to some remote hop.

Within Facility Backup, there are two ways
1. `link-protection`, to protect from link failure
2. `node-link-protection`, to protect from both node and link failure (fallsback to link protection when the topology forces it)


Unlike detour LSPs (from fast-reroute), the bypass LSP is actually a separate, standalone LSP with its own name. 

The bypass LSP name is automatically deribed by the system, indicating what is being protected.

Facility backup builds a stack of labels to protect multiple LSPs.
In additiona to the transport label, a bypass label is pushed to the top of the stack. 
The routers look at only the bypass label, as the packet travels down the bypass LSP.

Node protecting LSPs are usually longer, so may affect delay-sensitive traffic.

To enable facility backup,
1. configure `link-protection` on any interface that should offer protection
2. Add `link-protection` or `node-link-protection` on the LSP

```sh
[edit]
root@juniper# set protocols rsvp interface <interface-name> link-protection
[edit]
root@juniper# set protocols mpls label-switched-path <lsp-name> to X.X.X.X <link-protection | node-link-protection>
```

Junos names the backup bypass LSPs automatically, for example, `Bypass->X.X.X.X->Y.Y.Y.Y` indicating that this bypass skips `X.X.X.X` and goes directly to `Y.Y.Y.Y`.

If its a link-protection bypass LSP, it is named like `Bypass->Z.Z.Z.Z`

Use `show route detail` to see the label stack pushed to the backup path.

Use `show mpls lsp bypass`  to see ingress bypass LSPs. The `show rsvp session ingress` also shows them.



Some devices can not push more than three labels on a packet.

MPLS VPNs-> one label, Main RSVP LSP-> one label, Bypass RSVP LSP-> one label, also some applications require an extra label.

In such scenarios, fast-reroute may be a better option.

By default, facility backup creates only one bypass LSP. To prevent a link from being overwhelmed, we can configure two or more bypass LSPs.

```sh
[edit]
root@juniper# set protocols rsvp interface <interface-name> link-protection max-bypasses <value>
[edit]
root@juniper# set protocols rsvp interface <interface-name> link-protection bandwidth <value>
```

