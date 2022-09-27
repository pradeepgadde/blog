---

layout: single
title:  "RSVP Primary and Secondary LSPs"
date:   2022-09-25 12:59:04 +0530
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

# Primary and Secondary Paths
We can create multiple named paths for an LSP, each path with its own constraints like BW, Admin groups, strict and loose hops etc.
We can designate them as `Primary` and `Secondary`. 
Multiple secondary paths possible but only one Primary path.

Primary Paths are not Mandatory!

When we assign primary and secondary paths to an LSP, primary path comes up first. Secondary path is used only if the primary fails (it is not even calculated in advance and is `down` by default).
When the primary path( need not be exactly same hop-by-hop previous path, any path that meets primary constraints) is available, traffic reverts after a minute.

Secondary paths do not actively try to avoid links on the Primary path, so try to add more specific constraints when possible.

Configure paths, we can also add constraints under each path (like BW, adming groups etc)
```sh
[edit]
root@juniper# set protocols mpls path <path-name> <constraints like strict/loose hops>
```

Bind your paths to the LSP using primary and secondary

```
[edit]
root@juniper# set protocols mpls label-switched-path <lsp-name> primary <path-name>
[edit]
root@juniper# set protocols mpls label-switched-path <lsp-name> secondary <path-name>
```

Any constraint set at the `label-switched-path` will affect both Primary and Secondary paths. They inherit those constraints.

If a link goes down on the primary path, ingress router brings up the secondary path and immediately used it for forwarding.

And ingress router also tries to calculate a new primary path, if it can't it tries again every 30 seconds.

If it can, the new primary path comes up, but not used until a minute for stability testing.

```sh
root@juniper> show mpls lsp ingress detail
```

In this command output, look for `ActivePath: <path-name> (primary|secondary)`.

Three timers:

- `retry-limit`, default is `0`, means unlimited , number of times to try, to find a new Primary path
- `retry-timer`, default is `30` seconds, the time to wait before trying to find a new path
- `revert-timer`, default is `60` seconds, the wait time before traffic is moved to new primary path

```sh
[edit]
root@juniper# set protocols mpls label-switched-path <lsp-name> retry-limit <value>
[edit]
root@juniper# set protocols mpls label-switched-path <lsp-name> retry-timer <value>
[edit]
root@juniper# set protocols mpls label-switched-path <lsp-name> revert-timer <value>
```

We can configure two or more secondary paths, without any primary path.

If traffic moves to another secondary path, it stays on that path. 

Anothe option to achieve similar result is using `revert-timer` of `0`.



It is possible to move to a secondary path manually at any time,

```sh
[edit]
root@juniper# set protocols mpls label-switched-path <lsp-name> secondary <path-name> select manual
```

With this secondary path comes up immediately and is used for forwarding. During the switchover, no packets are lost.

Primary path remains up but not used. 

There is another option, `select unconditional` that forces the move, even if the secondary path can not be formed.



Always-up standby secondary paths: a solution for defining secondary constraints which is manual and tedious job.

`Standby` paths are pre-calculated, pre-signalled, always-up. Adds a temporary metric of 8,000,000 to each link used by the primary path.

```sh
[edit]
root@juniper# set protocols mpls label-switched-path <lsp-name> secondary <path-name> standby
```

We can also create an empty path, that does not have any constraints. This will take the metrically best path.

Note, standby secondary paths double the RSVP state.

Also, the standby secondary paths are in the routing table, however they do not get installed in the forwarding table by default.

In the `show route detail` command output, we can see the `weight` of each next hop. The numerically smallest weight is always preferred.

So, if the primary goes down, there will be a delay while the secondary standby is installed in the FIB.



Enabling load-balancing (`load-balance per-packet or per-flow [>21.4 Junos] `) also preinstalles the backup paths into the FIB.
