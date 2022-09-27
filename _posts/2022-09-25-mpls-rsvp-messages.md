---

layout: single
title:  "Tearing down RSVP LSPs"
date:   2022-09-25 11:59:04 +0530
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
# RSVP Messages
To deal with RSVP failures, problems, and errors
- PathTear- travel in the same direction as Path (towards egress)
- ResvTear- travel in the same direction as Resv (towards ingress)
- PathErr- travel in the opposite direction to Path (towards ingress)
- ResvErr-travel in the opposite direction to Resv (towards egress)

PathTear and ResvTear are used to tear down an LSP. They removed the LSP from the network along with any resource reservations related to the LSP.

RSVP messages contain a numerical Tunnel ID and ls LSP ID. The combination of these two uniquely identify an LSP.

There is a downtime associated with LSP failure as these messages take time to reach the ingress and the router takes some more time to process and re-establish new paths. If there are more LSPs, there would be even more downtime.

To significantly reduce the downtime associated, we can make use of 

- backup local repair LSPs (pre-siganlled to route around nodes/links)
- secondary paths for an LSP at the ingress (headend)



PathErr and ResvErr don't tear down LSPs, they just report errors, using Error object

Errors in response to Path messages

- BW guarantees that cannot be guaranteed
- Strict hops that cannot be met
- Unknown loose hops
- MTU problems
- Routing loop in the RRO

Also, after an LSP is setup, there could be some errors like link down.

ResvTear and PathErr are often sent at same time.

If we look at Error messages, we see `Error node`, `Error code` , and `Error value`.



RSVP was originally soft-state protocol, with no adjacencies and no acknowledgements. RSVP messages sent every 30 second, for each LSP. Also, PathTear/ResvTear are sent only once.

To customize some of these,

```sh
[edit]
root@juniper# set protocols rsvp refresh-time <value>
[edit]
root@juniper# set protocols rsvp keep-multiplier <value>
```

Original RSVP was enhanced to reduce overhead. A new bit added to the RSVP header - `Refresh-Reduction-Capable` bit to support three new features

- RSVP bundle messages (more than one RSVP message in a packet)
- Message IDs and Acknowledgements (to make RSVP reliable)
- Summary Refresh Extension (possible to refresh multiple LSPs in one single message)

```sh
root@juniper> show rsvp neighbor detail
```
In the above output, if we see `Refresh reduction: operational`, RSVP extensions are enabled.

If not, we can enable manually, esp in old Junos releases
```sh
[edit]
root@juniper# set protocols rsvp interface <interface-name> aggregate
[edit]
root@juniper# set protocols rsvp interface <interface-name> reliable
```

RSVP hello messages are optional (as per RFC), so some vendore might not send RSVP hello messages.

In Junos, they are sent every `9` seconds by default. If the peer does not send hellos, Juniper devices fall back to soft-state method.

