---

layout: single
title:  "RSVP LSP Optimization"
date:   2022-09-25 15:59:04 +0530
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

# Optimizing RSVP LSPs

RSVP LSPs can automatically find and signal better, more optimal paths, but this is not by default.

By default, RSVP LSPs stay on their path until they are torn down or preempted. They wont change even if a better path becomes available.

This default behaviour can be changed with LSP optimization which runs CSPF at regular intervals (specified by the opimization timer).

We can enable optimization globally or on individual LSPs. Both methods make use of the `optimize-timer` statement.

```sh
[edit]
root@juniper# set protocols mpls optimize-timer <value>
```

```sh
[edit]
root@juniper# set protocols mpls label-switched-path <lsp-name> to X.X.X.X optimize-timer <value>
```

The default value for `optimize-timer` is `0` seconds, which means optimization never happens.

We can also optimize manually, with the `clear mpls lsp name <lsp-name> optimize` command. Traffic moves to new paths gracefully.



Conditions for LSP optimization

1. The new CSPF metric must not be higher than the old path
2. If the metrics are same, the new path must not have more hops
3. The new path must not cause preemption of other LSPs
4. The new path must not worsen the congestion overall 

Large topologies benefit from shorter optimize-timers but more frequent opitimiztion increses the CPU usage.

If we want to optimize purely on IGP metric (condition 1 above) and ignore the remaining all conditions (condition 2, 3, and 4), we can specify `optimize-aggressive` either globally or on individual LSP.



```sh
[edit]
root@juniper# set protocols mpls optimize-aggressive
```

```sh
[edit]
root@juniper# set protocols mpls label-switched-path <lsp-name> to X.X.X.X optimize-aggressive
```

We can also optimize-aggressivemanually, with the `clear mpls lsp name <lsp-name> optimize-aggressive` command.

Also, we can optimize detour and bypass LSPs, both configured under `edit protocols rsvp`.

```sh
[edit]
root@juniper# set protocols rsvp fast-reroute optimize-timer <value>
```

```sh
[edit]
root@juniper# set protocols rsvp interface <interface-name> link-protection optimize-timer <value>
```

