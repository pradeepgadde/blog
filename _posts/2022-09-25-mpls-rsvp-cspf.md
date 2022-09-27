---

layout: single
title:  "Constrained Shortest Path First"
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

# CSPF
CSPF is an advanced version of SPF, using the TED links are pruned if they do not have enough BW or tagged to be avoided or don't contain mandatory tags (admin groups). On this constrained partial topology, strict/loose hops are considered. If equal path costs exist, pick one with the least hops. If still there is a tie, choose one at random.

This LSP computation happens one at a time starting with the high-priority LSPs. If LSPs are of equal priority, then they are setup in alphabetical order.

Reservable Bandwidth - total BW an interface can offer
Available Bandwidth = (total reservable BW) - (BW of all LSPs on that link)
Available Bandwidth Ratio = (Available BW % Reservable BW) * 100

By default, CSPF chooses one `random` path, incase of a tie (equal cost paths).
We can change this to choose the `most-fill` path or the `least-fill` path.
Fullness is judged by the Available BW Ratio.



`least-fill` means largest available bandwidth ratio , `most-fill` is tha path with the smallest available bandwidth ratio.

`Least-fill` balances load, while `most-fill` avoids the bin packing problem.

We configure these options under the LSP itself.

```sh
[edit]
root@juniper# set protocols mpls label-switched-path <lsp-name> <random|least-fill|most-fill>
```