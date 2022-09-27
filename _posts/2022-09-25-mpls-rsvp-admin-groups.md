---

layout: single
title:  "Administrative Groups"
date:   2022-09-25 10:59:04 +0530
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

# Tag links to be used or to be avoided
It is possible to tag links either to be avoided or used during the LSP path calculation. We configure admin groups on the individual links on each router. This information is then propagated via IS-IS/OSPF, and stored in the TED.

Configuring Admin Groups:
specify a name for each group, and assign a value from `0` to `31`. 
Note, only the number is advertised, not the name, 
Make sure, configuration is similar on all devices in the environment.

```sh
[edit]
root@juniper# set protocols mpls admin-groups <name> <0-31>
```

Once we define the admin group, tag each interface 

```sh
[edit]
root@juniper# set protocols mpls interface <interface-name> admin-group <admin-group-name>
```

It is not mandatory to tag both ends of the link with the same admin group (because LSPs are unidirectional!)

To see the admin groups on each interface

```sh
root@juniper> show mpls interface
```

In this output, we see `Administrative groups (x: extended)` column.

With extended admin groups, we can have more than `32` admin groups.

One Interface can belong to more than one admin groups. This information is carried by OSPF/IS-IS in a single 32-bit value, where each bit represents one group. Left most is group` 0` and right most is group `31`.

For example, if an interface belongs to admin groups `9` and `15`, then it is represented as `00000000000000001000001000000000`

To add admin-group constraints, we can use 

- `include-any`: at least one of the admin-groups must be present
- `include-all`: all of the configured admin-groups must be present
- `exclude`: does not contain atleast one of the groups (like exclude any)

Note with`exclude`, if any of the tags present (atleast one), then the link will be pruned from the topology.



If you tag a link after an LSP has come up, by default, nothing happens. Once an LSP is up, it stays up (for stability).

It is possible to change this behaviour with LSP optimization.



