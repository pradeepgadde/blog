---

layout: single
title:  "Moving Traffic Gracefully to New LSPs"
date:   2022-09-25 16:59:04 +0530
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

# Make-before-Break

An LSPs path can take advantage of better links that become available in the network and traffic moves over to this new path, without first tearing down the old path.

Temporarily there will be two live copies of the same LSP: same name, same Tunnel ID, but different LSP IDs. This is how a router is able to move traffic over gracefully, with no packet loss.

This is not visible in `show mpls lsp` but in `show rsvp session`. 

Once the new path proves to be stable, old path is removed.

Optimization makes a new LSP before it breaks the old one.

We can find logs related to this:

`CSPF: Reroute due to re-optimization`

`Originate make-before-break call`

`Make-before-break: Switched to new instance`

When two copies of the same LSP share a link, they are treated as two different LSPs and this has an impact on LSP optimization.

By default, these two copies of the same LSP don't share BW reservation,  because they are treated as two different.

LSP Reservation Styles:

1. `Fixed Filter` (FF): default , two of the same LSP are treated as separate LSPs. Can not share BW reservations
2. `Shared Explicit` (SE): enabled with the `adaptive` command. Two of the same LSPs can share a BW reservation

The style is requested in the `Path` Session Attribute object and `Resv` message confirms it in a Style object.

To change the style, simply add `adaptive` to the LSP

```sh
[edit]
root@juniper# set protocols mpls label-switched-path <lsp-name> to X.X.X.X adaptive
```

In the `show rsvp session` output, we can see the `Resv style:` as either `FF` or `SE`.



