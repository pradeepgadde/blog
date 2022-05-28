---

layout: single
title:  "MPLS RSVP LSPs in Junos"
date:   2011-01-22 07:59:04 +0530
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

MPLS RSVP LSPs

Here’s a simple setup to understand the basic operation of MPLS Dynamic Label Switched Path using signaling protocol RSVP (Resource Reservation Protocol) .

Note: We just need to configure LSP on the Ingress device only, In all other devices we simply need to enable RSVP and MPLS on the required interfaces.


## Ingress

```sh
root@Ingress# show | no-more 
## Last changed: 2011-01-22 00:23:15 UTC
version 10.4R1.9;
system {
    host-name Ingress;
    root-authentication {
        encrypted-password "$1$cYyoUQl.$blO1yJaM39r/7rGw.a8k.1"; ## SECRET-DATA
    }
}
interfaces {
    ge-0/0/0 {
        unit 0 {
            family inet {
                address 10.20.30.1/24;
            }
            family mpls;
        }
    }
    ge-0/0/1 {
        unit 0 {
            family inet {
                address 100.1.1.1/30;
            }
            family mpls;
        }
    }
}
routing-options {
    static {
        route 0.0.0.0/0 next-hop 100.1.1.2;
    }
}
protocols {
    rsvp {
        interface ge-0/0/1.0;
        interface ge-0/0/0.0;
    }
    mpls {
        traffic-engineering bgp-igp;
        traceoptions {
            file mplstest;
            flag all;
        }
        label-switched-path mylsp {
            to 192.168.1.100;
            install 172.16.1.1/32;
            no-cspf;
        }
        interface ge-0/0/1.0;
        interface ge-0/0/0.0;
    }
}
security {
    forwarding-options {
        family {
            mpls {
                mode packet-based;
            }
        }
    }
}
```

```sh
[edit]
root@Ingress# run show route | no-more 

inet.0: 7 destinations, 7 routes (7 active, 0 holddown, 0 hidden)
+ = Active Route, - = Last Active, * = Both

0.0.0.0/0          *[Static/5] 00:54:11
                    > to 100.1.1.2 via ge-0/0/1.0
10.20.30.0/24      *[Direct/0] 00:08:31
                    > via ge-0/0/0.0
10.20.30.1/32      *[Local/0] 00:54:11
                      Local via ge-0/0/0.0
100.1.1.0/30       *[Direct/0] 00:54:11
                    > via ge-0/0/1.0
100.1.1.1/32       *[Local/0] 00:54:11
                      Local via ge-0/0/1.0
172.16.1.1/32      *[RSVP/7/1] 00:29:53, metric 65535
                    > to 100.1.1.2 via ge-0/0/1.0, label-switched-path mylsp
192.168.1.100/32   *[RSVP/7/1] 00:29:53, metric 65535
                    > to 100.1.1.2 via ge-0/0/1.0, label-switched-path mylsp

mpls.0: 3 destinations, 3 routes (3 active, 0 holddown, 0 hidden)
+ = Active Route, - = Last Active, * = Both

0                  *[MPLS/0] 00:54:12, metric 1
                      Receive
1                  *[MPLS/0] 00:54:12, metric 1
                      Receive
2                  *[MPLS/0] 00:54:12, metric 1
                      Receive
```

```sh
[edit]
root@Ingress# run show mpls lsp | no-more 
Ingress LSP: 1 sessions
To              From            State Rt P     ActivePath       LSPname
192.168.1.100   100.1.1.1       Up     2 *                      mylsp
Total 1 displayed, Up 1, Down 0

Egress LSP: 1 sessions
To              From            State   Rt Style Labelin Labelout LSPname 
100.1.1.1       172.16.1.1      Up       0  1 FF       3        - myreturnlsp
Total 1 displayed, Up 1, Down 0

Transit LSP: 0 sessions
Total 0 displayed, Up 0, Down 0

[edit]
root@Ingress# run show mpls lsp extensive | no-more 
Ingress LSP: 1 sessions

192.168.1.100
  From: 100.1.1.1, State: Up, ActiveRoute: 2, LSPname: mylsp
  ActivePath:  (primary)
  LSPtype: Static Configured
  LoadBalance: Random
  Encoding type: Packet, Switching type: Packet, GPID: IPv4
 *Primary                    State: Up
    Priorities: 7 0
    SmartOptimizeTimer: 180
    Received RRO (ProtectionFlag 1=Available 2=InUse 4=B/W 8=Node 10=SoftPreempt 20=Node-ID):
          100.1.1.2 200.1.1.1 192.168.1.100
    5 Jan 22 00:48:43.764 Selected as active path
    4 Jan 22 00:48:43.762 Record Route:  100.1.1.2 200.1.1.1 192.168.1.100
    3 Jan 22 00:48:43.759 Up
    2 Jan 22 00:44:58.728 200.1.1.1: No Route toward dest[3 times]
    1 Jan 22 00:24:25.548 Originate Call
  Created: Sat Jan 22 00:24:24 2011
Total 1 displayed, Up 1, Down 0

Egress LSP: 1 sessions

100.1.1.1
  From: 172.16.1.1, LSPstate: Up, ActiveRoute: 0
  LSPname: myreturnlsp, LSPpath: Primary
  Suggested label received: -, Suggested label sent: -
  Recovery label received: -, Recovery label sent: -
  Resv style: 1 FF, Label in: 3, Label out: -
  Time left:  139, Since: Sat Jan 22 01:13:46 2011
  Tspec: rate 0bps size 0bps peak Infbps m 20 M 1500
  Port number: sender 2 receiver 53845 protocol 0
  PATH rcvfrom: 100.1.1.2 (ge-0/0/1.0) 9 pkts
  Adspec: received MTU 1500 
  PATH sentto: localclient
  RESV rcvfrom: localclient 
  Record route: 192.168.1.100 200.1.1.1 100.1.1.2 <self>  
Total 1 displayed, Up 1, Down 0

Transit LSP: 0 sessions
Total 0 displayed, Up 0, Down 0

```

```sh
root@Ingress# run show rsvp session 
Ingress RSVP: 1 sessions
To              From            State   Rt Style Labelin Labelout LSPname 
192.168.1.100   100.1.1.1       Up       2  1 FF       -   299856 mylsp
Total 1 displayed, Up 1, Down 0

Egress RSVP: 1 sessions
To              From            State   Rt Style Labelin Labelout LSPname 
100.1.1.1       172.16.1.1      Up       0  1 FF       3        - myreturnlsp
Total 1 displayed, Up 1, Down 0

Transit RSVP: 0 sessions
Total 0 displayed, Up 0, Down 0

[edit]
root@Ingress# run show rsvp session detail | no-more 
Ingress RSVP: 1 sessions

192.168.1.100
  From: 100.1.1.1, LSPstate: Up, ActiveRoute: 2
  LSPname: mylsp, LSPpath: Primary
  LSPtype: Static Configured
  Suggested label received: -, Suggested label sent: -
  Recovery label received: -, Recovery label sent: 299856
  Resv style: 1 FF, Label in: -, Label out: 299856
  Time left:    -, Since: Sat Jan 22 00:24:25 2011
  Tspec: rate 0bps size 0bps peak Infbps m 20 M 1500
  Port number: sender 1 receiver 53029 protocol 0
  PATH rcvfrom: localclient 
  Adspec: sent MTU 1500
  Path MTU: received 1500
  PATH sentto: 100.1.1.2 (ge-0/0/1.0) 77 pkts
  RESV rcvfrom: 100.1.1.2 (ge-0/0/1.0) 43 pkts
  Record route: <self> 100.1.1.2 200.1.1.1 192.168.1.100  
Total 1 displayed, Up 1, Down 0

Egress RSVP: 1 sessions

100.1.1.1
  From: 172.16.1.1, LSPstate: Up, ActiveRoute: 0
  LSPname: myreturnlsp, LSPpath: Primary
  Suggested label received: -, Suggested label sent: -
  Recovery label received: -, Recovery label sent: -
  Resv style: 1 FF, Label in: 3, Label out: -
  Time left:  128, Since: Sat Jan 22 01:13:46 2011
  Tspec: rate 0bps size 0bps peak Infbps m 20 M 1500
  Port number: sender 2 receiver 53845 protocol 0
  PATH rcvfrom: 100.1.1.2 (ge-0/0/1.0) 9 pkts
  Adspec: received MTU 1500 
  PATH sentto: localclient
  RESV rcvfrom: localclient 
  Record route: 192.168.1.100 200.1.1.1 100.1.1.2 <self>  
Total 1 displayed, Up 1, Down 0

Transit RSVP: 0 sessions
Total 0 displayed, Up 0, Down 0

[edit]
root@Ingress# run traceroute 172.16.1.1 
traceroute to 172.16.1.1 (172.16.1.1), 30 hops max, 40 byte packets
 1  100.1.1.2 (100.1.1.2)  4.679 ms  3.640 ms  3.857 ms
     MPLS Label=299856 CoS=0 TTL=1 S=1
 2  200.1.1.1 (200.1.1.1)  4.259 ms  4.501 ms  4.961 ms
     MPLS Label=299776 CoS=0 TTL=1 S=1
 3  172.16.1.1 (172.16.1.1)  4.972 ms  4.560 ms  10.311 ms
```

```sh
[edit]
root@Ingress# run show rsvp interface 
RSVP interface: 2 active
                  Active Subscr- Static      Available   Reserved    Highwater
Interface   State resv   iption  BW          BW          BW          mark
ge-0/0/0.0  Up         0   100%  1000Mbps    1000Mbps    0bps        0bps       
ge-0/0/1.0  Up         1   100%  1000Mbps    1000Mbps    0bps        0bps       

[edit]
root@Ingress# run show rsvp neighbor     
RSVP neighbor: 1 learned
Address            Idle Up/Dn LastChange HelloInt HelloTx/Rx MsgRcvd
100.1.1.2             0  1/0       33:53        9   227/227  61

[edit]
root@Ingress# run show rsvp neighbor detail 
RSVP neighbor: 1 learned
Address: 100.1.1.2 via: ge-0/0/1.0 status: Up
  Last changed time: 33:58, Idle: 5 sec, Up cnt: 1, Down cnt: 0
  Message received: 61
  Hello: sent 227, received: 227, interval: 9 sec
  Remote instance: 0x72ab8bb1, Local instance: 0x2bdff369
  Refresh reduction:  not operational

[edit]
root@Ingress#
```

## Transit
```sh
root@Transit# show | no-more 
## Last changed: 2011-01-22 03:48:27 UTC
version 10.2R3.10;
system {
    host-name Transit;
    root-authentication {
        encrypted-password "$1$uLjcwqNG$Elo7yksUMhtaopeQUj0Uf0"; ## SECRET-DATA
    }
}
interfaces {
    fe-0/0/0 {
        unit 0 {
            family inet {
                address 100.1.1.2/30;
            }
            family mpls;
        }
    }
    fe-0/0/1 {
        unit 0 {
            family inet {
                address 200.1.1.2/30;
            }
            family mpls;
        }
    }
}
routing-options {
    static {
        route 172.16.1.1/32 next-hop 200.1.1.1;
        route 192.168.1.0/24 next-hop 200.1.1.1;
    }
}
protocols {
    rsvp {
        interface fe-0/0/0.0;
        interface fe-0/0/1.0;
    }
    mpls {
        traceoptions {
            file mplstest;
            flag all;
        }
        interface fe-0/0/0.0;
        interface fe-0/0/1.0;
    }
}
security {
    forwarding-options {
        family {
            mpls {
                mode packet-based;
            }
        }
    }
}
```

```sh
[edit]
root@Transit# run show route | no-more 

inet.0: 6 destinations, 6 routes (6 active, 0 holddown, 0 hidden)
+ = Active Route, - = Last Active, * = Both

100.1.1.0/30       *[Direct/0] 1d 02:05:32
                    > via fe-0/0/0.0
100.1.1.2/32       *[Local/0] 1d 02:05:58
                      Local via fe-0/0/0.0
172.16.1.1/32      *[Static/5] 01:08:36
                    > to 200.1.1.1 via fe-0/0/1.0
192.168.1.0/24     *[Static/5] 00:59:18
                    > to 200.1.1.1 via fe-0/0/1.0
200.1.1.0/30       *[Direct/0] 1d 02:04:40
                    > via fe-0/0/1.0
200.1.1.2/32       *[Local/0] 1d 02:05:57
                      Local via fe-0/0/1.0

mpls.0: 6 destinations, 6 routes (6 active, 0 holddown, 0 hidden)
+ = Active Route, - = Last Active, * = Both

0                  *[MPLS/0] 1d 02:06:30, metric 1
                      Receive
1                  *[MPLS/0] 1d 02:06:30, metric 1
                      Receive
2                  *[MPLS/0] 1d 02:06:30, metric 1
                      Receive
299856             *[RSVP/7/1] 00:34:47, metric 1
                    > to 200.1.1.1 via fe-0/0/1.0, label-switched-path mylsp
299888             *[RSVP/7/1] 00:09:44, metric 1
                    > to 100.1.1.1 via fe-0/0/0.0, label-switched-path myreturnlsp
299888(S=0)        *[RSVP/7/1] 00:09:44, metric 1
                    > to 100.1.1.1 via fe-0/0/0.0, label-switched-path myreturnlsp
```

```sh

root@Transit# run show mpls lsp 
Ingress LSP: 0 sessions
Total 0 displayed, Up 0, Down 0

Egress LSP: 0 sessions
Total 0 displayed, Up 0, Down 0

Transit LSP: 2 sessions
To              From            State   Rt Style Labelin Labelout LSPname 
100.1.1.1       172.16.1.1      Up       1  1 FF  299888        3 myreturnlsp
192.168.1.100   100.1.1.1       Up       1  1 FF  299856   299776 mylsp
Total 2 displayed, Up 2, Down 0


root@Transit# run show mpls lsp detail | no-more 
Ingress LSP: 0 sessions
Total 0 displayed, Up 0, Down 0

Egress LSP: 0 sessions
Total 0 displayed, Up 0, Down 0

Transit LSP: 2 sessions

100.1.1.1
  From: 172.16.1.1, LSPstate: Up, ActiveRoute: 1
  LSPname: myreturnlsp, LSPpath: Primary
  Suggested label received: -, Suggested label sent: -
  Recovery label received: -, Recovery label sent: 3
  Resv style: 1 FF, Label in: 299888, Label out: 3
  Time left:  157, Since: Sat Jan 22 04:37:54 2011
  Tspec: rate 0bps size 0bps peak Infbps m 20 M 1500
  Port number: sender 2 receiver 53845 protocol 0
  PATH rcvfrom: 200.1.1.1 (fe-0/0/1.0) 16 pkts
  Adspec: received MTU 1500 sent MTU 1500
  PATH sentto: 100.1.1.1 (fe-0/0/0.0) 16 pkts
  RESV rcvfrom: 100.1.1.1 (fe-0/0/0.0) 16 pkts
  Record route: 192.168.1.100 200.1.1.1 <self> 100.1.1.1  

192.168.1.100
  From: 100.1.1.1, LSPstate: Up, ActiveRoute: 1
  LSPname: mylsp, LSPpath: Primary
  Suggested label received: -, Suggested label sent: -
  Recovery label received: -, Recovery label sent: 299776
  Resv style: 1 FF, Label in: 299856, Label out: 299776
  Time left:  139, Since: Sat Jan 22 03:50:12 2011
  Tspec: rate 0bps size 0bps peak Infbps m 20 M 1500
  Port number: sender 1 receiver 53029 protocol 0
  PATH rcvfrom: 100.1.1.1 (fe-0/0/0.0) 79 pkts
  Adspec: received MTU 1500 sent MTU 1500
  PATH sentto: 200.1.1.1 (fe-0/0/1.0) 81 pkts
  RESV rcvfrom: 200.1.1.1 (fe-0/0/1.0) 49 pkts
  Record route: 100.1.1.1 <self> 200.1.1.1 192.168.1.100  
Total 2 displayed, Up 2, Down 0

```

```sh
root@Transit# run show rsvp session 
Ingress RSVP: 0 sessions
Total 0 displayed, Up 0, Down 0

Egress RSVP: 0 sessions
Total 0 displayed, Up 0, Down 0

Transit RSVP: 2 sessions
To              From            State   Rt Style Labelin Labelout LSPname 
100.1.1.1       172.16.1.1      Up       1  1 FF  299888        3 myreturnlsp
192.168.1.100   100.1.1.1       Up       1  1 FF  299856   299776 mylsp
Total 2 displayed, Up 2, Down 0

[edit]
root@Transit# run show rsvp session detail | no-more 
Ingress RSVP: 0 sessions
Total 0 displayed, Up 0, Down 0

Egress RSVP: 0 sessions
Total 0 displayed, Up 0, Down 0

Transit RSVP: 2 sessions

100.1.1.1
  From: 172.16.1.1, LSPstate: Up, ActiveRoute: 1
  LSPname: myreturnlsp, LSPpath: Primary
  Suggested label received: -, Suggested label sent: -
  Recovery label received: -, Recovery label sent: 3
  Resv style: 1 FF, Label in: 299888, Label out: 3
  Time left:  147, Since: Sat Jan 22 04:37:54 2011
  Tspec: rate 0bps size 0bps peak Infbps m 20 M 1500
  Port number: sender 2 receiver 53845 protocol 0
  PATH rcvfrom: 200.1.1.1 (fe-0/0/1.0) 16 pkts
  Adspec: received MTU 1500 sent MTU 1500
  PATH sentto: 100.1.1.1 (fe-0/0/0.0) 16 pkts
  RESV rcvfrom: 100.1.1.1 (fe-0/0/0.0) 16 pkts
  Record route: 192.168.1.100 200.1.1.1 <self> 100.1.1.1  

192.168.1.100
  From: 100.1.1.1, LSPstate: Up, ActiveRoute: 1
  LSPname: mylsp, LSPpath: Primary
  Suggested label received: -, Suggested label sent: -
  Recovery label received: -, Recovery label sent: 299776
  Resv style: 1 FF, Label in: 299856, Label out: 299776
  Time left:  129, Since: Sat Jan 22 03:50:12 2011
  Tspec: rate 0bps size 0bps peak Infbps m 20 M 1500
  Port number: sender 1 receiver 53029 protocol 0
  PATH rcvfrom: 100.1.1.1 (fe-0/0/0.0) 79 pkts
  Adspec: received MTU 1500 sent MTU 1500
  PATH sentto: 200.1.1.1 (fe-0/0/1.0) 81 pkts
  RESV rcvfrom: 200.1.1.1 (fe-0/0/1.0) 49 pkts
  Record route: 100.1.1.1 <self> 200.1.1.1 192.168.1.100  
Total 2 displayed, Up 2, Down 0

[edit]
```

## Egress
```sh
root@Egress# show | no-more 
## Last changed: 2011-01-21 21:43:25 UTC
version 10.4R1.9;
system {
    host-name Egress;
    root-authentication {
        encrypted-password "$1$zBf6qAyP$Q9Emr5Fjyr464GRNb1tf31"; ## SECRET-DATA
    }
}
interfaces {
    ge-0/0/0 {
        unit 0 {
            family inet {
                address 192.168.1.1/24;
            }
            family mpls;
        }
    }
    ge-0/0/1 {
        unit 0 {
            family inet {
                address 200.1.1.1/30;
            }
            family mpls;
        }
    }
}
routing-options {
    static {
        route 100.1.1.0/30 next-hop 200.1.1.2;
        route 0.0.0.0/0 next-hop 192.168.1.100;
    }
}
protocols {
    rsvp {
        interface ge-0/0/1.0;
        interface ge-0/0/0.0;
    }
    mpls {
        interface ge-0/0/1.0;
        interface ge-0/0/0.0;
    }
}
security {
    forwarding-options {
        family {
            mpls {
                mode packet-based;
            }
        }
    }
}
```

```sh
[edit]
root@Egress# run show route | no-more 

inet.0: 6 destinations, 6 routes (6 active, 0 holddown, 0 hidden)
+ = Active Route, - = Last Active, * = Both

0.0.0.0/0          *[Static/5] 00:32:44
                    > to 192.168.1.100 via ge-0/0/0.0
100.1.1.0/30       *[Static/5] 02:18:11
                    > to 200.1.1.2 via ge-0/0/1.0
192.168.1.0/24     *[Direct/0] 00:32:44
                    > via ge-0/0/0.0
192.168.1.1/32     *[Local/0] 02:18:11
                      Local via ge-0/0/0.0
200.1.1.0/30       *[Direct/0] 02:18:11
                    > via ge-0/0/1.0
200.1.1.1/32       *[Local/0] 02:18:11
                      Local via ge-0/0/1.0

mpls.0: 6 destinations, 6 routes (6 active, 0 holddown, 0 hidden)
+ = Active Route, - = Last Active, * = Both

0                  *[MPLS/0] 02:18:11, metric 1
                      Receive
1                  *[MPLS/0] 02:18:11, metric 1
                      Receive
2                  *[MPLS/0] 02:18:11, metric 1
                      Receive
299776             *[RSVP/7/1] 00:32:10, metric 1
                    > to 192.168.1.100 via ge-0/0/0.0, label-switched-path mylsp
299776(S=0)        *[RSVP/7/1] 00:32:10, metric 1
                    > to 192.168.1.100 via ge-0/0/0.0, label-switched-path mylsp
299808             *[RSVP/7/1] 00:07:07, metric 1
                    > to 200.1.1.2 via ge-0/0/1.0, label-switched-path myreturnlsp
```

```sh

root@Egress# run show mpls lsp  
Ingress LSP: 0 sessions
Total 0 displayed, Up 0, Down 0

Egress LSP: 0 sessions
Total 0 displayed, Up 0, Down 0

Transit LSP: 2 sessions
To              From            State   Rt Style Labelin Labelout LSPname 
100.1.1.1       172.16.1.1      Up       1  1 FF  299808   299888 myreturnlsp
192.168.1.100   100.1.1.1       Up       1  1 FF  299776        3 mylsp
Total 2 displayed, Up 2, Down 0

[edit]
root@Egress# run show mpls lsp extensive | no-more 
Ingress LSP: 0 sessions
Total 0 displayed, Up 0, Down 0

Egress LSP: 0 sessions
Total 0 displayed, Up 0, Down 0

Transit LSP: 2 sessions

100.1.1.1
  From: 172.16.1.1, LSPstate: Up, ActiveRoute: 1
  LSPname: myreturnlsp, LSPpath: Primary
  Suggested label received: -, Suggested label sent: -
  Recovery label received: -, Recovery label sent: 299888
  Resv style: 1 FF, Label in: 299808, Label out: 299888
  Time left:  134, Since: Fri Jan 21 22:37:54 2011
  Tspec: rate 0bps size 0bps peak Infbps m 20 M 1500
  Port number: sender 2 receiver 53845 protocol 0
  PATH rcvfrom: 192.168.1.100 (ge-0/0/0.0) 12 pkts
  Adspec: received MTU 1500 sent MTU 1500
  PATH sentto: 200.1.1.2 (ge-0/0/1.0) 12 pkts
  RESV rcvfrom: 200.1.1.2 (ge-0/0/1.0) 12 pkts
  Record route: 192.168.1.100 <self> 200.1.1.2 100.1.1.1  

192.168.1.100
  From: 100.1.1.1, LSPstate: Up, ActiveRoute: 1
  LSPname: mylsp, LSPpath: Primary
  Suggested label received: -, Suggested label sent: -
  Recovery label received: -, Recovery label sent: 3
  Resv style: 1 FF, Label in: 299776, Label out: 3
  Time left:  116, Since: Fri Jan 21 22:12:51 2011
  Tspec: rate 0bps size 0bps peak Infbps m 20 M 1500
  Port number: sender 1 receiver 53029 protocol 0
  PATH rcvfrom: 200.1.1.2 (ge-0/0/1.0) 45 pkts
  Adspec: received MTU 1500 sent MTU 1500
  PATH sentto: 192.168.1.100 (ge-0/0/0.0) 45 pkts
  RESV rcvfrom: 192.168.1.100 (ge-0/0/0.0) 45 pkts
  Record route: 100.1.1.1 200.1.1.2 <self> 192.168.1.100  
Total 2 displayed, Up 2, Down 0
```

```sh
[edit]
root@Egress# run show rsvp session 
Ingress RSVP: 0 sessions
Total 0 displayed, Up 0, Down 0

Egress RSVP: 0 sessions
Total 0 displayed, Up 0, Down 0

Transit RSVP: 2 sessions
To              From            State   Rt Style Labelin Labelout LSPname 
100.1.1.1       172.16.1.1      Up       1  1 FF  299808   299888 myreturnlsp
192.168.1.100   100.1.1.1       Up       1  1 FF  299776        3 mylsp
Total 2 displayed, Up 2, Down 0

[edit]
root@Egress# run show rsvp session detail | no-more 
Ingress RSVP: 0 sessions
Total 0 displayed, Up 0, Down 0

Egress RSVP: 0 sessions
Total 0 displayed, Up 0, Down 0

Transit RSVP: 2 sessions

100.1.1.1
  From: 172.16.1.1, LSPstate: Up, ActiveRoute: 1
  LSPname: myreturnlsp, LSPpath: Primary
  Suggested label received: -, Suggested label sent: -
  Recovery label received: -, Recovery label sent: 299888
  Resv style: 1 FF, Label in: 299808, Label out: 299888
  Time left:  126, Since: Fri Jan 21 22:37:54 2011
  Tspec: rate 0bps size 0bps peak Infbps m 20 M 1500
  Port number: sender 2 receiver 53845 protocol 0
  PATH rcvfrom: 192.168.1.100 (ge-0/0/0.0) 12 pkts
  Adspec: received MTU 1500 sent MTU 1500
  PATH sentto: 200.1.1.2 (ge-0/0/1.0) 12 pkts
  RESV rcvfrom: 200.1.1.2 (ge-0/0/1.0) 12 pkts
  Record route: 192.168.1.100 <self> 200.1.1.2 100.1.1.1  

192.168.1.100
  From: 100.1.1.1, LSPstate: Up, ActiveRoute: 1
  LSPname: mylsp, LSPpath: Primary
  Suggested label received: -, Suggested label sent: -
  Recovery label received: -, Recovery label sent: 3
  Resv style: 1 FF, Label in: 299776, Label out: 3
  Time left:  153, Since: Fri Jan 21 22:12:51 2011
  Tspec: rate 0bps size 0bps peak Infbps m 20 M 1500
  Port number: sender 1 receiver 53029 protocol 0
  PATH rcvfrom: 200.1.1.2 (ge-0/0/1.0) 46 pkts
  Adspec: received MTU 1500 sent MTU 1500
  PATH sentto: 192.168.1.100 (ge-0/0/0.0) 46 pkts
  RESV rcvfrom: 192.168.1.100 (ge-0/0/0.0) 46 pkts
  Record route: 100.1.1.1 200.1.1.2 <self> 192.168.1.100  
Total 2 displayed, Up 2, Down 0

[edit]
```

## End

```sh
root@End# show | no-more 
## Last changed: 2011-01-22 01:12:04 UTC
version 10.4R1.9;
system {
    host-name End;
    root-authentication {
        encrypted-password "$1$0Qvx.RMD$ki0qDPBmNXrg8dVHSi/rR0"; ## SECRET-DATA
    }
    name-server {
        208.67.222.222;
        208.67.220.220;
    }
    login {
        class test {
            allow-configuration " zones | policies | utm ";
            deny-configuration security;
        }
        user lab {
            uid 2000;
            class test;
            authentication {
                encrypted-password "$1$71tuO5YL$QPsIszs/9AdEay5.enTbb1"; ## SECRET-DATA
            }
        }
    }
    services {
        ssh;
        telnet;
        xnm-clear-text;
        web-management {
            http {
                interface vlan.0;
            }
            https {
                system-generated-certificate;
                interface vlan.0;
            }
        }
        dhcp {
            router {
                192.168.1.1;
            }
            pool 192.168.1.0/24 {
                address-range low 192.168.1.2 high 192.168.1.254;
            }
            propagate-settings ge-0/0/0.0;
        }
    }
    syslog {
        archive size 100k files 3;
        user * {
            any emergency;
        }
        file messages {
            any critical;
            authorization info;
        }
        file interactive-commands {
            interactive-commands error;
        }
    }
    max-configurations-on-flash 5;
    max-configuration-rollbacks 5;
    license {
        autoupdate {
            url https://ae1.juniper.net/junos/key_retrieval;
        }
    }
}
interfaces {
    ge-0/0/0 {
        unit 0 {
            family inet {
                address 192.168.1.100/24;
            }
            family mpls;
        }
    }
    lo0 {
        unit 0 {
            family inet {
                address 172.16.1.1/32;
            }
        }
    }
}
routing-options {
    static {
        route 0.0.0.0/0 next-hop 192.168.1.1;
    }
}
protocols {
    rsvp {
        interface ge-0/0/0.0;
    }
    mpls {
        traffic-engineering bgp-igp;
        label-switched-path myreturnlsp {
            to 100.1.1.1;
            install 10.20.30.5/32 active;
            no-cspf;
        }
        interface ge-0/0/0.0;
    }
    stp;
}
security {
    zones {
        security-zone test {
            interfaces {
                all {
                    host-inbound-traffic {
                        system-services {
                            all;
                        }
                        protocols {
                            all;
                        }
                    }
                }
            }
        }
    }
    policies {
        default-policy {
            permit-all;
        }
    }
}
```

```sh
root@End# run show route | no-more 

inet.0: 6 destinations, 6 routes (6 active, 0 holddown, 0 hidden)
+ = Active Route, - = Last Active, * = Both

0.0.0.0/0          *[Static/5] 00:38:31
                    > to 192.168.1.1 via ge-0/0/0.0
10.20.30.5/32      *[RSVP/7/1] 00:12:54, metric 65535
                    > to 192.168.1.1 via ge-0/0/0.0, label-switched-path myreturnlsp
100.1.1.1/32       *[RSVP/7/1] 00:12:54, metric 65535
                    > to 192.168.1.1 via ge-0/0/0.0, label-switched-path myreturnlsp
172.16.1.1/32      *[Direct/0] 00:39:06
                    > via lo0.0
192.168.1.0/24     *[Direct/0] 00:38:31
                    > via ge-0/0/0.0
192.168.1.100/32   *[Local/0] 00:38:35
                      Local via ge-0/0/0.0

mpls.0: 3 destinations, 3 routes (3 active, 0 holddown, 0 hidden)
+ = Active Route, - = Last Active, * = Both

0                  *[MPLS/0] 00:39:07, metric 1
                      Receive
1                  *[MPLS/0] 00:39:07, metric 1
                      Receive
2                  *[MPLS/0] 00:39:07, metric 1
                      Receive
```

```sh
root@End# run show mpls lsp 
Ingress LSP: 1 sessions
To              From            State Rt P     ActivePath       LSPname
100.1.1.1       172.16.1.1      Up     2 *                      myreturnlsp
Total 1 displayed, Up 1, Down 0

Egress LSP: 1 sessions
To              From            State   Rt Style Labelin Labelout LSPname 
192.168.1.100   100.1.1.1       Up       0  1 FF       3        - mylsp
Total 1 displayed, Up 1, Down 0

Transit LSP: 0 sessions
Total 0 displayed, Up 0, Down 0


root@End# run show mpls lsp detail | no-more 
Ingress LSP: 1 sessions

100.1.1.1
  From: 172.16.1.1, State: Up, ActiveRoute: 2, LSPname: myreturnlsp
  ActivePath:  (primary)
  LSPtype: Static Configured
  LoadBalance: Random
  Encoding type: Packet, Switching type: Packet, GPID: IPv4
 *Primary                    State: Up
    Priorities: 7 0
    SmartOptimizeTimer: 180
    Received RRO (ProtectionFlag 1=Available 2=InUse 4=B/W 8=Node 10=SoftPreempt 20=Node-ID):
          192.168.1.1 200.1.1.2 100.1.1.1
Total 1 displayed, Up 1, Down 0

Egress LSP: 1 sessions

192.168.1.100
  From: 100.1.1.1, LSPstate: Up, ActiveRoute: 0
  LSPname: mylsp, LSPpath: Primary
  Suggested label received: -, Suggested label sent: -
  Recovery label received: -, Recovery label sent: -
  Resv style: 1 FF, Label in: 3, Label out: -
  Time left:  124, Since: Sat Jan 22 00:47:01 2011
  Tspec: rate 0bps size 0bps peak Infbps m 20 M 1500
  Port number: sender 1 receiver 53029 protocol 0
  PATH rcvfrom: 192.168.1.1 (ge-0/0/0.0) 53 pkts
  Adspec: received MTU 1500 
  PATH sentto: localclient
  RESV rcvfrom: localclient 
  Record route: 100.1.1.1 200.1.1.2 192.168.1.1 <self>  
Total 1 displayed, Up 1, Down 0

Transit LSP: 0 sessions
Total 0 displayed, Up 0, Down 0

[edit]
root@End# run show rsvp session 
Ingress RSVP: 1 sessions
To              From            State   Rt Style Labelin Labelout LSPname 
100.1.1.1       172.16.1.1      Up       2  1 FF       -   299808 myreturnlsp
Total 1 displayed, Up 1, Down 0

Egress RSVP: 1 sessions
To              From            State   Rt Style Labelin Labelout LSPname 
192.168.1.100   100.1.1.1       Up       0  1 FF       3        - mylsp
Total 1 displayed, Up 1, Down 0

Transit RSVP: 0 sessions
Total 0 displayed, Up 0, Down 0

[edit]
root@End# run show rsvp session detail | no-more 
Ingress RSVP: 1 sessions

100.1.1.1
  From: 172.16.1.1, LSPstate: Up, ActiveRoute: 2
  LSPname: myreturnlsp, LSPpath: Primary
  LSPtype: Static Configured
  Suggested label received: -, Suggested label sent: -
  Recovery label received: -, Recovery label sent: 299808
  Resv style: 1 FF, Label in: -, Label out: 299808
  Time left:    -, Since: Sat Jan 22 01:12:04 2011
  Tspec: rate 0bps size 0bps peak Infbps m 20 M 1500
  Port number: sender 2 receiver 53845 protocol 0
  PATH rcvfrom: localclient 
  Adspec: sent MTU 1500
  Path MTU: received 1500
  PATH sentto: 192.168.1.1 (ge-0/0/0.0) 20 pkts
  RESV rcvfrom: 192.168.1.1 (ge-0/0/0.0) 20 pkts
  Record route: <self> 192.168.1.1 200.1.1.2 100.1.1.1  
Total 1 displayed, Up 1, Down 0

Egress RSVP: 1 sessions

192.168.1.100
  From: 100.1.1.1, LSPstate: Up, ActiveRoute: 0
  LSPname: mylsp, LSPpath: Primary
  Suggested label received: -, Suggested label sent: -
  Recovery label received: -, Recovery label sent: -
  Resv style: 1 FF, Label in: 3, Label out: -
  Time left:  116, Since: Sat Jan 22 00:47:01 2011
  Tspec: rate 0bps size 0bps peak Infbps m 20 M 1500
  Port number: sender 1 receiver 53029 protocol 0
  PATH rcvfrom: 192.168.1.1 (ge-0/0/0.0) 53 pkts
  Adspec: received MTU 1500 
  PATH sentto: localclient
  RESV rcvfrom: localclient 
  Record route: 100.1.1.1 200.1.1.2 192.168.1.1 <self>  
Total 1 displayed, Up 1, Down 0

Transit RSVP: 0 sessions
Total 0 displayed, Up 0, Down 0

[edit]
root@End# run traceroute 10.20.30.5 
traceroute to 10.20.30.5 (10.20.30.5), 30 hops max, 40 byte packets
 1  192.168.1.1 (192.168.1.1)  3.771 ms  9.823 ms  9.914 ms
     MPLS Label=299808 CoS=0 TTL=1 S=1
 2  200.1.1.2 (200.1.1.2)  9.657 ms  3.632 ms  3.135 ms
     MPLS Label=299888 CoS=0 TTL=1 S=1
 3  100.1.1.1 (100.1.1.1)  9.007 ms  9.975 ms  10.049 ms
 4  100.1.1.2 (100.1.1.2)  9.463 ms !N  9.503 ms !N  9.017 ms !N

[edit]
```