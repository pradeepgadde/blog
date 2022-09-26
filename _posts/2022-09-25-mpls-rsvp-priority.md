---

layout: single
title:  "RSVP LSP Priorities"
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
# Using RSVP Priority Levels

By default, The first router to boot up gets access to the best paths and LSPs in one router come up in alphabetical order.

So, the order in which these LSPs come up has a huge impact on which LSPs get access to which resources. And sometimes, this cause some LSPs to take a longer path or fail to come up.

When you pack things, the order matters! (The Bin Packing Problem)!

We can solve this problem by configuring priorities for the LSPs.
With Priority, we can move less important LSPs to different paths and make room for more important LSPs. Also, it is possible to tear down less important LSPs if no BW is available.

The stratgey is to pack big things first, the small things will fit in the gaps.
If we pack small LSPs first, we might artificially starve large LSPs. 
LSP priorities can prevent this problem;

Priority : 0 to 7 
numberically smallest is the best priority!

If there is enough BW, low and high priority LSPs can share the same path.

When there is no enough BW on the best path,
High priority LSPs force low priority LSPs to find an alternate path
Low priority LSPs may tear down completely if there is no BW 

Within a single router that has many LSPs, 
High priority LSPs are signalled first
If the priority is same, they are signalled in alphabetical order.

There are two type: `Setup` and `Hold` Priority.

New LSPs setup priority is compared to the existing LSP's hold priority.
If the new LSPs setup priority is better than the existing LSPs hold priority, the new LSP wins.In that case, existing LSP is kicked off and forced to find an alternate path.

Incase of draw, the existing LSP stays on the same path.
We cant configure a setup priority that is better than the hold priority. This is to prevent a situation where two LSPs constantly preempt each other (infinite loop!)

Defaults:
setup : 7 (weakest priority), will never replace any other LSP
hold: 0 (best priority), no other LSP can preempt it

To change the defaults, for example setup 5, hold 4
```sh
[edit]
root@juniper# set protocols mpls label-switched-path <lsp-name> priority 5 4
root@juniper# set protocols mpls label-switched-path <lsp-name> bandwidth 400m
```

The change of priority may cause LSP to flap.

In the TED, since we have configured a hold priority of 4, the configured BW is available at priorites 4 to 7.

Better priorites (0 to 3) still have the full BW available.

Higher priority LSPs only care about their priority level.

```sh

Available BW [priority] bps:

[0] 1000Mbps

[1] 1000Mbps

[2] 1000Mbps

[3] 1000Mbps

[4] 600Mbps

[5] 600Mbps

[6] 600Mbps

[7] 600Mbps

```

We can make use of `apply-groups` feature of Junos to change all LSP priorities. 

Note that when LSPs are moved because of priority, the rerouting is not graceful.

LSP is torn down before it signalled a new path.

To avoid this downtime, two options:

- soft-preemption
- adaptive



```sh
[edit]
root@juniper# set protocols mpls label-switched-path <lsp-name> soft-preemption
```

We will add this statement to the low priority LSP.
