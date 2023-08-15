---
layout: single
title: "Segment Routing"
date:   2023-08-15 01:55:04 +0530
categories: Networking
tags: MPLS
show_date: true
classes: wide
header:
  overlay_image: /assets/images/networking-2.jpg
  og_image: /assets/images/networking-2.jpg
  teaser: /assets/images/junos.png

author:
  name     : "Junos"
  avatar   : "/assets/images/junos.png"

sidebar:
  - title: "Topics"
    nav: my-sidebar

---
# Segment Routing

## Getting started with Segment Routing or SPRING or Source Packet Routing

In this article, let's replace LDP with SPRING to establish a full mesh of LSPs in a sample network with 8 routers (R1 through R8). R1 and R8 are the PE routers with a L3VPN configured on both.

IS-IS is the IGP protocol in use and Segment Routing extensions are enabled for IS-IS.

Only IPv4 is running, so IPv4 Node SID (node-segment IP4 index) is assigned. 

R1 has a node-segment ID of 41, R2 has 42, and so on.

On all routers SRGB is configured with the same start-label value of `100000` and index-range of `1000`.

## Configuration

```
pradeep@R1> show configuration protocols isis source-packet-routing 
srgb start-label 100000 index-range 1000;
node-segment ipv4-index 41;

pradeep@R1> 
```
```
pradeep@R2> show configuration protocols isis source-packet-routing 
srgb start-label 100000 index-range 1000;
node-segment ipv4-index 42;

pradeep@R2> 
```

```
pradeep@R3> show configuration protocols isis source-packet-routing 
srgb start-label 100000 index-range 1000;
node-segment ipv4-index 43;

pradeep@R3> 
```
```
pradeep@R4> show configuration protocols isis source-packet-routing 
srgb start-label 100000 index-range 1000;
node-segment ipv4-index 44;

pradeep@R4> 
```

```
pradeep@R5> show configuration protocols isis source-packet-routing 
srgb start-label 100000 index-range 1000;
node-segment ipv4-index 45;

pradeep@R5> 
```

```
pradeep@R6> show configuration protocols isis source-packet-routing 
srgb start-label 100000 index-range 1000;
node-segment ipv4-index 46;

pradeep@R6> 
```

```
pradeep@R7> show configuration protocols isis source-packet-routing 
srgb start-label 100000 index-range 1000;
node-segment ipv4-index 47;

pradeep@R7> 
```

```
pradeep@R8> show configuration protocols isis source-packet-routing 
srgb start-label 100000 index-range 1000;
node-segment ipv4-index 48;

pradeep@R8>
```
## Verification
```
pradeep@R1> show isis overview 
Instance: master
  Router ID: 192.168.1.1
  Hostname: R1
  Sysid: 1921.6800.1001
  Areaid: 49.0001
  Adjacency holddown: enabled
  Maximum Areas: 3
  LSP life time: 1200
  Attached bit evaluation: enabled
  SPF delay: 200 msec, SPF holddown: 5000 msec, SPF rapid runs: 3
  IPv4 is enabled, IPv6 is enabled, SPRING based MPLS is enabled
  Traffic engineering: enabled
  Traffic engineering v6: disabled
  Restart: Disabled
    Helper mode: Enabled
  Layer2-map: Disabled
  Source Packet Routing (SPRING): Enabled
    SRGB Config Range :
      SRGB Start-Label : 100000, SRGB Index-Range : 1000
    SRGB Block Allocation: Success
      SRGB Start Index : 100000, SRGB Size : 1000, Label-Range: [ 100000, 100999 ]
    Node Segments: Enabled
      Ipv4 Index : 41
    SRv6: Disabled
  Post Convergence Backup: Disabled
  Level 1
    Internal route preference: 15
    External route preference: 160
    Prefix export count: 0
    Wide metrics are enabled, Narrow metrics are enabled
    Source Packet Routing is enabled
  Level 2
    Internal route preference: 18
    External route preference: 165
    Prefix export count: 0
    Wide metrics are enabled, Narrow metrics are enabled
    Source Packet Routing is enabled

pradeep@R1> 
```

```
pradeep@R1> show isis database extensive | match Node   
  Node Segment Blocks Advertised:
      Node SID, Flags: 0x40(R:0,N:1,P:0,E:0,V:0,L:0), Algo: SPF(0), Value: 41
  Node Segment Blocks Advertised:
      Node SID, Flags: 0x40(R:0,N:1,P:0,E:0,V:0,L:0), Algo: SPF(0), Value: 42
  Node Segment Blocks Advertised:
      Node SID, Flags: 0x40(R:0,N:1,P:0,E:0,V:0,L:0), Algo: SPF(0), Value: 43
      Node SID, Flags: 0xe0(R:1,N:1,P:1,E:0,V:0,L:0), Algo: SPF(0), Value: 44
      Node SID, Flags: 0xe0(R:1,N:1,P:1,E:0,V:0,L:0), Algo: SPF(0), Value: 45
      Node SID, Flags: 0xe0(R:1,N:1,P:1,E:0,V:0,L:0), Algo: SPF(0), Value: 46
      Node SID, Flags: 0xe0(R:1,N:1,P:1,E:0,V:0,L:0), Algo: SPF(0), Value: 47
      Node SID, Flags: 0xe0(R:1,N:1,P:1,E:0,V:0,L:0), Algo: SPF(0), Value: 48

pradeep@R1> 
```

```
pradeep@R8> show isis overview 
Instance: master
  Router ID: 192.168.1.8
  Hostname: R8
  Sysid: 1921.6800.1008
  Areaid: 49.0003
  Adjacency holddown: enabled
  Maximum Areas: 3
  LSP life time: 1200
  Attached bit evaluation: enabled
  SPF delay: 200 msec, SPF holddown: 5000 msec, SPF rapid runs: 3
  IPv4 is enabled, IPv6 is enabled, SPRING based MPLS is enabled
  Traffic engineering: enabled
  Traffic engineering v6: disabled
  Restart: Disabled
    Helper mode: Enabled
  Layer2-map: Disabled
  Source Packet Routing (SPRING): Enabled
    SRGB Config Range :
      SRGB Start-Label : 100000, SRGB Index-Range : 1000
    SRGB Block Allocation: Success
      SRGB Start Index : 100000, SRGB Size : 1000, Label-Range: [ 100000, 100999 ]
    Node Segments: Enabled
      Ipv4 Index : 48
    SRv6: Disabled
  Post Convergence Backup: Disabled
  Level 1
    Internal route preference: 15
    External route preference: 160
    Prefix export count: 0
    Wide metrics are enabled, Narrow metrics are enabled
    Source Packet Routing is enabled
  Level 2
    Internal route preference: 18
    External route preference: 165
    Prefix export count: 0
    Wide metrics are enabled, Narrow metrics are enabled
    Source Packet Routing is enabled

pradeep@R8> 
```



## inet.3 on R1

```
pradeep@R1> show route table inet.3    

inet.3: 7 destinations, 7 routes (7 active, 0 holddown, 0 hidden)
+ = Active Route, - = Last Active, * = Both

192.168.1.2/32     *[L-ISIS/14] 08:51:11, metric 20
                    >  to 10.1.3.3 via ge-0/0/1.0, Push 100042
192.168.1.3/32     *[L-ISIS/14] 09:46:03, metric 10
                    >  to 10.1.3.3 via ge-0/0/1.0
192.168.1.4/32     *[L-ISIS/14] 09:35:21, metric 20
                    >  to 10.1.3.3 via ge-0/0/1.0, Push 100044
192.168.1.5/32     *[L-ISIS/14] 09:31:43, metric 20
                    >  to 10.1.3.3 via ge-0/0/1.0, Push 100045
192.168.1.6/32     *[L-ISIS/14] 09:28:16, metric 30
                    >  to 10.1.3.3 via ge-0/0/1.0, Push 100046
192.168.1.7/32     *[L-ISIS/14] 09:26:51, metric 40
                    >  to 10.1.3.3 via ge-0/0/1.0, Push 100047
192.168.1.8/32     *[L-ISIS/14] 08:52:18, metric 40
                    >  to 10.1.3.3 via ge-0/0/1.0, Push 100048

pradeep@R1> 
```

## Traceroute from R1 to R8
```
pradeep@R1> traceroute mpls segment-routing isis 192.168.1.8 
  Probe options: ttl 64, retries 3, wait 10, paths 16, exp 7, fanout 16

  ttl    Label  Protocol    Address          Previous Hop     Probe Status
    1   100048  ISIS        10.1.3.3         (null)           Success           
  FEC-Stack-Sent: ISIS 
  ttl    Label  Protocol    Address          Previous Hop     Probe Status
    2   100048  ISIS        10.3.4.4         10.1.3.3         Success           
  FEC-Stack-Sent: ISIS 
  ttl    Label  Protocol    Address          Previous Hop     Probe Status
    3   100048  ISIS        10.4.6.6         10.3.4.4         Success           
  FEC-Stack-Sent: ISIS 
  ttl    Label  Protocol    Address          Previous Hop     Probe Status
    4        3  ISIS        10.6.8.8         10.4.6.6         Egress            
  FEC-Stack-Sent: ISIS 

  Path 1 via ge-0/0/1.0 destination 127.0.0.64


pradeep@R1> 
```
## Traceroute from R1 to R7
```
pradeep@R1> traceroute mpls segment-routing isis 192.168.1.7    
  Probe options: ttl 64, retries 3, wait 10, paths 16, exp 7, fanout 16

  ttl    Label  Protocol    Address          Previous Hop     Probe Status
    1   100047  ISIS        10.1.3.3         (null)           Success           
  FEC-Stack-Sent: ISIS 
  ttl    Label  Protocol    Address          Previous Hop     Probe Status
    2   100047  ISIS        10.3.4.4         10.1.3.3         Success           
  FEC-Stack-Sent: ISIS 
  ttl    Label  Protocol    Address          Previous Hop     Probe Status
    3   100047  ISIS        10.4.6.6         10.3.4.4         Success           
  FEC-Stack-Sent: ISIS 
  ttl    Label  Protocol    Address          Previous Hop     Probe Status
    4        3  ISIS        10.6.7.7         10.4.6.6         Egress            
  FEC-Stack-Sent: ISIS 

  Path 1 via ge-0/0/1.0 destination 127.0.0.64


pradeep@R1> 
```
## Traceroute from R1 to R6
```
pradeep@R1> traceroute mpls segment-routing isis 192.168.1.6    
  Probe options: ttl 64, retries 3, wait 10, paths 16, exp 7, fanout 16

  ttl    Label  Protocol    Address          Previous Hop     Probe Status
    1   100046  ISIS        10.1.3.3         (null)           Success           
  FEC-Stack-Sent: ISIS 
  ttl    Label  Protocol    Address          Previous Hop     Probe Status
    2   100046  ISIS        10.3.4.4         10.1.3.3         Success           
  FEC-Stack-Sent: ISIS 
  ttl    Label  Protocol    Address          Previous Hop     Probe Status
    3        3  ISIS        10.4.6.6         10.3.4.4         Egress            
  FEC-Stack-Sent: ISIS 

  Path 1 via ge-0/0/1.0 destination 127.0.0.64


pradeep@R1>
```
## Traceroute from R1 to R5
```
pradeep@R1> traceroute mpls segment-routing isis 192.168.1.5    
  Probe options: ttl 64, retries 3, wait 10, paths 16, exp 7, fanout 16

  ttl    Label  Protocol    Address          Previous Hop     Probe Status
    1   100045  ISIS        10.1.3.3         (null)           Success           
  FEC-Stack-Sent: ISIS 
  ttl    Label  Protocol    Address          Previous Hop     Probe Status
    2        3  ISIS        10.3.5.5         10.1.3.3         Egress            
  FEC-Stack-Sent: ISIS 

  Path 1 via ge-0/0/1.0 destination 127.0.0.64


pradeep@R1> 
```
## Traceroute from R1 to R4
```
pradeep@R1> traceroute mpls segment-routing isis 192.168.1.4    
  Probe options: ttl 64, retries 3, wait 10, paths 16, exp 7, fanout 16

  ttl    Label  Protocol    Address          Previous Hop     Probe Status
    1   100044  ISIS        10.1.3.3         (null)           Success           
  FEC-Stack-Sent: ISIS 
  ttl    Label  Protocol    Address          Previous Hop     Probe Status
    2        3  ISIS        10.3.4.4         10.1.3.3         Egress            
  FEC-Stack-Sent: ISIS 

  Path 1 via ge-0/0/1.0 destination 127.0.0.64


pradeep@R1>
```
## Traceroute from R1 to R3
```
pradeep@R1> traceroute mpls segment-routing isis 192.168.1.3    
  Probe options: ttl 64, retries 3, wait 10, paths 16, exp 7, fanout 16

  ttl    Label  Protocol    Address          Previous Hop     Probe Status
    1                       10.1.3.3         (null)           Egress            
  FEC-Stack-Sent: ISIS 

  Path 1 via ge-0/0/1.0 destination 127.0.0.64


pradeep@R1>
```
## Traceroute from R1 to R2
```
pradeep@R1> traceroute mpls segment-routing isis 192.168.1.2    
  Probe options: ttl 64, retries 3, wait 10, paths 16, exp 7, fanout 16

  ttl    Label  Protocol    Address          Previous Hop     Probe Status
    1   100042  ISIS        10.1.3.3         (null)           Success           
  FEC-Stack-Sent: ISIS 
  ttl    Label  Protocol    Address          Previous Hop     Probe Status
    2        3  ISIS        10.2.3.2         10.1.3.3         Egress            
  FEC-Stack-Sent: ISIS 

  Path 1 via ge-0/0/1.0 destination 127.0.0.64


pradeep@R1>
```

## Route/Label Lookup from R1 to R8

```
pradeep@R1> show route 192.168.1.8 

inet.0: 25 destinations, 25 routes (25 active, 0 holddown, 0 hidden)
+ = Active Route, - = Last Active, * = Both

192.168.1.8/32     *[IS-IS/18] 08:59:28, metric 40
                    >  to 10.1.3.3 via ge-0/0/1.0

inet.3: 7 destinations, 7 routes (7 active, 0 holddown, 0 hidden)
+ = Active Route, - = Last Active, * = Both

192.168.1.8/32     *[L-ISIS/14] 08:59:28, metric 40
                    >  to 10.1.3.3 via ge-0/0/1.0, Push 100048

pradeep@R1>
```

```
pradeep@R2> show route label 100048 

mpls.0: 16 destinations, 16 routes (16 active, 0 holddown, 0 hidden)
+ = Active Route, - = Last Active, * = Both

100048             *[L-ISIS/14] 08:58:49, metric 40
                    >  to 10.2.3.3 via ge-0/0/2.0, Swap 100048

pradeep@R2> 
```

```
pradeep@R3> show route label 100048 

mpls.0: 25 destinations, 25 routes (25 active, 0 holddown, 0 hidden)
+ = Active Route, - = Last Active, * = Both

100048             *[L-ISIS/14] 09:00:24, metric 30
                    >  to 10.3.4.4 via ge-0/0/4.0, Swap 100048
                       to 10.3.5.5 via ge-0/0/5.0, Swap 100048

pradeep@R3> 
```

```
pradeep@R4> show route label 100048  

mpls.0: 19 destinations, 19 routes (19 active, 0 holddown, 0 hidden)
+ = Active Route, - = Last Active, * = Both

100048             *[L-ISIS/14] 09:00:51, metric 20
                    >  to 10.4.6.6 via ge-0/0/1.0, Swap 100048

pradeep@R4> 
```

```
pradeep@R5> show route label 100048       

mpls.0: 19 destinations, 19 routes (19 active, 0 holddown, 0 hidden)
+ = Active Route, - = Last Active, * = Both

100048             *[L-ISIS/14] 09:01:33, metric 20
                    >  to 10.5.6.6 via ge-0/0/2.0, Swap 100048

pradeep@R5>
```

```
pradeep@R6> show route label 100048 

mpls.0: 25 destinations, 25 routes (25 active, 0 holddown, 0 hidden)
+ = Active Route, - = Last Active, * = Both

100048             *[L-ISIS/14] 09:01:47, metric 10
                    >  to 10.6.8.8 via ge-0/0/5.0, Pop      
100048(S=0)        *[L-ISIS/14] 09:01:47, metric 10
                    >  to 10.6.8.8 via ge-0/0/5.0, Pop      

pradeep@R6> 
```

```
pradeep@R8> show route 192.168.1.8   

inet.0: 24 destinations, 24 routes (24 active, 0 holddown, 0 hidden)
+ = Active Route, - = Last Active, * = Both

192.168.1.8/32     *[Direct/0] 09:03:24
                    >  via lo0.0

pradeep@R8> 
```

## R1 L3VPN table

```
pradeep@R1> show route table XYZ 

XYZ.inet.0: 7 destinations, 7 routes (7 active, 0 holddown, 0 hidden)
+ = Active Route, - = Last Active, * = Both

172.16.1.0/24      *[Direct/0] 5w3d 05:42:45
                    >  via ge-0/0/2.0
172.16.1.1/32      *[Local/0] 5w3d 05:42:45
                       Local via ge-0/0/2.0
172.16.101.0/24    *[BGP/170] 09:16:16, localpref 100, from 192.168.1.8
                      AS path: I, validation-state: unverified
                    >  to 10.1.3.3 via ge-0/0/1.0, Push 16, Push 100048(top)
192.168.11.0/24    *[BGP/170] 09:16:16, localpref 100, from 192.168.1.8
                      AS path: I, validation-state: unverified
                    >  to 10.1.3.3 via ge-0/0/1.0, Push 16, Push 100048(top)
192.168.12.0/24    *[BGP/170] 09:16:16, localpref 100, from 192.168.1.8
                      AS path: I, validation-state: unverified
                    >  to 10.1.3.3 via ge-0/0/1.0, Push 16, Push 100048(top)
192.168.150.0/24   *[Static/5] 5w3d 05:42:45
                    >  to 172.16.1.101 via ge-0/0/2.0
192.168.250.0/24   *[Static/5] 5w3d 05:42:45
                    >  to 172.16.1.101 via ge-0/0/2.0

XYZ.inet6.0: 1 destinations, 1 routes (1 active, 0 holddown, 0 hidden)
+ = Active Route, - = Last Active, * = Both

ff02::2/128        *[INET6/0] 5w3d 05:42:45
                       MultiRecv

pradeep@R1> 
```

## R8 inet.3 table

```
pradeep@R8> show route table inet.3 

inet.3: 7 destinations, 7 routes (7 active, 0 holddown, 0 hidden)
+ = Active Route, - = Last Active, * = Both

192.168.1.1/32     *[L-ISIS/14] 09:03:11, metric 40
                    >  to 10.6.8.6 via ge-0/0/5.0, Push 100041
192.168.1.2/32     *[L-ISIS/14] 09:02:03, metric 40
                    >  to 10.6.8.6 via ge-0/0/5.0, Push 100042
192.168.1.3/32     *[L-ISIS/14] 09:03:11, metric 30
                    >  to 10.6.8.6 via ge-0/0/5.0, Push 100043
192.168.1.4/32     *[L-ISIS/14] 09:03:11, metric 20
                    >  to 10.6.8.6 via ge-0/0/5.0, Push 100044
192.168.1.5/32     *[L-ISIS/14] 09:03:11, metric 20
                    >  to 10.6.8.6 via ge-0/0/5.0, Push 100045
192.168.1.6/32     *[L-ISIS/14] 09:03:11, metric 10
                    >  to 10.6.8.6 via ge-0/0/5.0
192.168.1.7/32     *[L-ISIS/14] 09:03:01, metric 20
                    >  to 10.6.8.6 via ge-0/0/5.0, Push 100047

pradeep@R8> 
```

## Traceroute from R8 to R1

```
pradeep@R8> traceroute mpls segment-routing isis 192.168.1.1 
  Probe options: ttl 64, retries 3, wait 10, paths 16, exp 7, fanout 16

  ttl    Label  Protocol    Address          Previous Hop     Probe Status
    1   100041  ISIS        10.6.8.6         (null)           Success           
  FEC-Stack-Sent: ISIS 
  ttl    Label  Protocol    Address          Previous Hop     Probe Status
    2   100041  ISIS        10.4.6.4         10.6.8.6         Success           
  FEC-Stack-Sent: ISIS 
  ttl    Label  Protocol    Address          Previous Hop     Probe Status
    3   100041  ISIS        10.3.4.3         10.4.6.4         Success           
  FEC-Stack-Sent: ISIS 
  ttl    Label  Protocol    Address          Previous Hop     Probe Status
    4        3  ISIS        10.1.3.1         10.3.4.3         Egress            
  FEC-Stack-Sent: ISIS 

  Path 1 via ge-0/0/5.0 destination 127.0.0.64


pradeep@R8> 
```

## R8 L3VPN table

```
pradeep@R8> show route table XYZ.inet   

XYZ.inet.0: 7 destinations, 7 routes (7 active, 0 holddown, 0 hidden)
+ = Active Route, - = Last Active, * = Both

172.16.1.0/24      *[BGP/170] 09:13:20, localpref 100, from 192.168.1.1
                      AS path: I, validation-state: unverified
                    >  to 10.6.8.6 via ge-0/0/5.0, Push 49, Push 100041(top)
172.16.101.0/24    *[Direct/0] 09:13:28
                    >  via ge-0/0/0.0
172.16.101.1/32    *[Local/0] 09:13:28
                       Local via ge-0/0/0.0
192.168.11.0/24    *[Static/5] 09:13:28
                    >  to 172.16.101.101 via ge-0/0/0.0
192.168.12.0/24    *[Static/5] 09:13:28
                    >  to 172.16.101.101 via ge-0/0/0.0
192.168.150.0/24   *[BGP/170] 09:13:20, localpref 100, from 192.168.1.1
                      AS path: I, validation-state: unverified
                    >  to 10.6.8.6 via ge-0/0/5.0, Push 49, Push 100041(top)
192.168.250.0/24   *[BGP/170] 09:13:20, localpref 100, from 192.168.1.1
                      AS path: I, validation-state: unverified
                    >  to 10.6.8.6 via ge-0/0/5.0, Push 49, Push 100041(top)

XYZ.inet6.0: 1 destinations, 1 routes (1 active, 0 holddown, 0 hidden)
+ = Active Route, - = Last Active, * = Both

ff02::2/128        *[INET6/0] 09:14:14
                       MultiRecv

pradeep@R8> 
```



## MPLS table on R3

```
pradeep@R3> show route table mpls.0 

mpls.0: 25 destinations, 25 routes (25 active, 0 holddown, 0 hidden)
+ = Active Route, - = Last Active, * = Both

0                  *[MPLS/0] 7w3d 14:56:40, metric 1
                       to table inet.0
0(S=0)             *[MPLS/0] 7w3d 14:56:40, metric 1
                       to table mpls.0
1                  *[MPLS/0] 7w3d 14:56:40, metric 1
                       Receive
2                  *[MPLS/0] 7w3d 14:56:40, metric 1
                       to table inet6.0
2(S=0)             *[MPLS/0] 7w3d 14:56:40, metric 1
                       to table mpls.0
13                 *[MPLS/0] 7w3d 14:56:40, metric 1
                       Receive
50                 *[L-ISIS/14] 09:58:14, metric 0
                    >  to 10.3.5.5 via ge-0/0/5.0, Pop      
50(S=0)            *[L-ISIS/14] 09:58:14, metric 0
                    >  to 10.3.5.5 via ge-0/0/5.0, Pop      
51                 *[L-ISIS/14] 09:58:14, metric 0
                    >  to 10.3.4.4 via ge-0/0/4.0, Pop      
51(S=0)            *[L-ISIS/14] 09:58:14, metric 0
                    >  to 10.3.4.4 via ge-0/0/4.0, Pop      
53                 *[L-ISIS/14] 09:58:14, metric 0
                    >  to 10.1.3.1 via ge-0/0/1.0, Pop      
53(S=0)            *[L-ISIS/14] 09:58:14, metric 0
                    >  to 10.1.3.1 via ge-0/0/1.0, Pop      
54                 *[L-ISIS/14] 09:03:32, metric 0
                    >  to 10.2.3.2 via ge-0/0/2.0, Pop      
54(S=0)            *[L-ISIS/14] 09:03:32, metric 0
                    >  to 10.2.3.2 via ge-0/0/2.0, Pop      
100041             *[L-ISIS/14] 09:58:14, metric 10
                    >  to 10.1.3.1 via ge-0/0/1.0, Pop      
100041(S=0)        *[L-ISIS/14] 09:58:14, metric 10
                    >  to 10.1.3.1 via ge-0/0/1.0, Pop      
100042             *[L-ISIS/14] 09:03:22, metric 10
                    >  to 10.2.3.2 via ge-0/0/2.0, Pop      
100042(S=0)        *[L-ISIS/14] 09:03:22, metric 10
                    >  to 10.2.3.2 via ge-0/0/2.0, Pop      
100044             *[L-ISIS/14] 09:47:32, metric 10
                    >  to 10.3.4.4 via ge-0/0/4.0, Pop      
100044(S=0)        *[L-ISIS/14] 09:47:32, metric 10
                    >  to 10.3.4.4 via ge-0/0/4.0, Pop      
100045             *[L-ISIS/14] 09:43:55, metric 10
                    >  to 10.3.5.5 via ge-0/0/5.0, Pop      
100045(S=0)        *[L-ISIS/14] 09:43:55, metric 10
                    >  to 10.3.5.5 via ge-0/0/5.0, Pop      
100046             *[L-ISIS/14] 09:40:27, metric 20
                    >  to 10.3.4.4 via ge-0/0/4.0, Swap 100046
                       to 10.3.5.5 via ge-0/0/5.0, Swap 100046
100047             *[L-ISIS/14] 09:39:03, metric 30
                    >  to 10.3.4.4 via ge-0/0/4.0, Swap 100047
                       to 10.3.5.5 via ge-0/0/5.0, Swap 100047
100048             *[L-ISIS/14] 09:04:29, metric 30
                    >  to 10.3.4.4 via ge-0/0/4.0, Swap 100048
                       to 10.3.5.5 via ge-0/0/5.0, Swap 100048

pradeep@R3> 
```


## R6 MPLS table

```
pradeep@R6> show route table mpls.0 

mpls.0: 25 destinations, 25 routes (25 active, 0 holddown, 0 hidden)
+ = Active Route, - = Last Active, * = Both

0                  *[MPLS/0] 7w3d 15:17:00, metric 1
                       to table inet.0
0(S=0)             *[MPLS/0] 7w3d 15:17:00, metric 1
                       to table mpls.0
1                  *[MPLS/0] 7w3d 15:17:00, metric 1
                       Receive
2                  *[MPLS/0] 7w3d 15:17:00, metric 1
                       to table inet6.0
2(S=0)             *[MPLS/0] 7w3d 15:17:00, metric 1
                       to table mpls.0
13                 *[MPLS/0] 7w3d 15:17:00, metric 1
                       Receive
47                 *[L-ISIS/14] 09:33:21, metric 0
                    >  to 10.6.8.8 via ge-0/0/5.0, Pop      
47(S=0)            *[L-ISIS/14] 09:33:21, metric 0
                    >  to 10.6.8.8 via ge-0/0/5.0, Pop      
48                 *[L-ISIS/14] 10:09:18, metric 0
                    >  to 10.6.7.7 via ge-0/0/4.0, Pop      
48(S=0)            *[L-ISIS/14] 10:09:18, metric 0
                    >  to 10.6.7.7 via ge-0/0/4.0, Pop      
49                 *[L-ISIS/14] 10:09:18, metric 0
                    >  to 10.5.6.5 via ge-0/0/2.0, Pop      
49(S=0)            *[L-ISIS/14] 10:09:18, metric 0
                    >  to 10.5.6.5 via ge-0/0/2.0, Pop      
50                 *[L-ISIS/14] 10:09:18, metric 0
                    >  to 10.4.6.4 via ge-0/0/1.0, Pop      
50(S=0)            *[L-ISIS/14] 10:09:18, metric 0
                    >  to 10.4.6.4 via ge-0/0/1.0, Pop      
100041             *[L-ISIS/14] 10:09:18, metric 30
                    >  to 10.4.6.4 via ge-0/0/1.0, Swap 100041
                       to 10.5.6.5 via ge-0/0/2.0, Swap 100041
100042             *[L-ISIS/14] 09:32:13, metric 30
                    >  to 10.4.6.4 via ge-0/0/1.0, Swap 100042
                       to 10.5.6.5 via ge-0/0/2.0, Swap 100042
100043             *[L-ISIS/14] 10:09:18, metric 20
                    >  to 10.4.6.4 via ge-0/0/1.0, Swap 100043
                       to 10.5.6.5 via ge-0/0/2.0, Swap 100043
100044             *[L-ISIS/14] 10:09:18, metric 10
                    >  to 10.4.6.4 via ge-0/0/1.0, Pop      
100044(S=0)        *[L-ISIS/14] 10:09:18, metric 10
                    >  to 10.4.6.4 via ge-0/0/1.0, Pop      
100045             *[L-ISIS/14] 10:09:18, metric 10
                    >  to 10.5.6.5 via ge-0/0/2.0, Pop      
100045(S=0)        *[L-ISIS/14] 10:09:18, metric 10
                    >  to 10.5.6.5 via ge-0/0/2.0, Pop      
100047             *[L-ISIS/14] 10:07:54, metric 10
                    >  to 10.6.7.7 via ge-0/0/4.0, Pop      
100047(S=0)        *[L-ISIS/14] 10:07:54, metric 10
                    >  to 10.6.7.7 via ge-0/0/4.0, Pop      
100048             *[L-ISIS/14] 09:33:11, metric 10
                    >  to 10.6.8.8 via ge-0/0/5.0, Pop      
100048(S=0)        *[L-ISIS/14] 09:33:11, metric 10
                    >  to 10.6.8.8 via ge-0/0/5.0, Pop      

pradeep@R6> 
```



## R3 inet.3 table

```
pradeep@R3> show route table inet.3 

inet.3: 7 destinations, 7 routes (7 active, 0 holddown, 0 hidden)
+ = Active Route, - = Last Active, * = Both

192.168.1.1/32     *[L-ISIS/14] 09:59:20, metric 10
                    >  to 10.1.3.1 via ge-0/0/1.0
192.168.1.2/32     *[L-ISIS/14] 09:04:28, metric 10
                    >  to 10.2.3.2 via ge-0/0/2.0
192.168.1.4/32     *[L-ISIS/14] 09:48:38, metric 10
                    >  to 10.3.4.4 via ge-0/0/4.0
192.168.1.5/32     *[L-ISIS/14] 09:45:01, metric 10
                    >  to 10.3.5.5 via ge-0/0/5.0
192.168.1.6/32     *[L-ISIS/14] 09:41:33, metric 20
                       to 10.3.4.4 via ge-0/0/4.0, Push 100046
                    >  to 10.3.5.5 via ge-0/0/5.0, Push 100046
192.168.1.7/32     *[L-ISIS/14] 09:40:09, metric 30
                       to 10.3.4.4 via ge-0/0/4.0, Push 100047
                    >  to 10.3.5.5 via ge-0/0/5.0, Push 100047
192.168.1.8/32     *[L-ISIS/14] 09:05:35, metric 30
                    >  to 10.3.4.4 via ge-0/0/4.0, Push 100048
                       to 10.3.5.5 via ge-0/0/5.0, Push 100048

pradeep@R3> 
```

```
pradeep@R3> show isis database extensive | match SID    
     P2P IPv4 Adj-SID:      50, Weight:   0, Flags: --VL--
    NLPID: 0x83, Fixed length: 27 bytes, Version: 1, Sysid length: 0 bytes
      Node SID, Flags: 0x40(R:0,N:1,P:0,E:0,V:0,L:0), Algo: SPF(0), Value: 41
      SPRING Capability - Flags: 0xc0(I:1,V:1), Range: 1000, SID-Label: 100000
      P2P IPV4 Adj-SID - Flags:0x30(F:0,B:0,V:1,L:1,S:0,P:0), Weight:0, Label: 50
      P2P IPv4 Adj-SID:      50, Weight:   0, Flags: --VL--
     P2P IPv4 Adj-SID:      16, Weight:   0, Flags: --VL--
    NLPID: 0x83, Fixed length: 27 bytes, Version: 1, Sysid length: 0 bytes
      Node SID, Flags: 0x40(R:0,N:1,P:0,E:0,V:0,L:0), Algo: SPF(0), Value: 42
      SPRING Capability - Flags: 0xc0(I:1,V:1), Range: 1000, SID-Label: 100000
      P2P IPV4 Adj-SID - Flags:0x30(F:0,B:0,V:1,L:1,S:0,P:0), Weight:0, Label: 16
      P2P IPv4 Adj-SID:      16, Weight:   0, Flags: --VL--
     P2P IPv4 Adj-SID:      53, Weight:   0, Flags: --VL--
     P2P IPv4 Adj-SID:      54, Weight:   0, Flags: --VL--
    NLPID: 0x83, Fixed length: 27 bytes, Version: 1, Sysid length: 0 bytes
      Node SID, Flags: 0x40(R:0,N:1,P:0,E:0,V:0,L:0), Algo: SPF(0), Value: 43
      Node SID, Flags: 0xe0(R:1,N:1,P:1,E:0,V:0,L:0), Algo: SPF(0), Value: 44
      Node SID, Flags: 0xe0(R:1,N:1,P:1,E:0,V:0,L:0), Algo: SPF(0), Value: 45
      Node SID, Flags: 0xe0(R:1,N:1,P:1,E:0,V:0,L:0), Algo: SPF(0), Value: 46
      Node SID, Flags: 0xe0(R:1,N:1,P:1,E:0,V:0,L:0), Algo: SPF(0), Value: 47
      Node SID, Flags: 0xe0(R:1,N:1,P:1,E:0,V:0,L:0), Algo: SPF(0), Value: 48
      SPRING Capability - Flags: 0xc0(I:1,V:1), Range: 1000, SID-Label: 100000
      P2P IPV4 Adj-SID - Flags:0x30(F:0,B:0,V:1,L:1,S:0,P:0), Weight:0, Label: 53
      P2P IPv4 Adj-SID:      53, Weight:   0, Flags: --VL--
      P2P IPV4 Adj-SID - Flags:0x30(F:0,B:0,V:1,L:1,S:0,P:0), Weight:0, Label: 54
      P2P IPv4 Adj-SID:      54, Weight:   0, Flags: --VL--
     P2P IPv4 Adj-SID:      51, Weight:   0, Flags: --VL--
     P2P IPv4 Adj-SID:      50, Weight:   0, Flags: --VL--
    NLPID: 0x83, Fixed length: 27 bytes, Version: 1, Sysid length: 0 bytes
      Node SID, Flags: 0x40(R:0,N:1,P:0,E:0,V:0,L:0), Algo: SPF(0), Value: 43
      Node SID, Flags: 0xe0(R:1,N:1,P:1,E:0,V:0,L:0), Algo: SPF(0), Value: 41
      Node SID, Flags: 0xe0(R:1,N:1,P:1,E:0,V:0,L:0), Algo: SPF(0), Value: 42
      SPRING Capability - Flags: 0xc0(I:1,V:1), Range: 1000, SID-Label: 100000
      P2P IPV4 Adj-SID - Flags:0x30(F:0,B:0,V:1,L:1,S:0,P:0), Weight:0, Label: 50
      P2P IPv4 Adj-SID:      50, Weight:   0, Flags: --VL--
      P2P IPV4 Adj-SID - Flags:0x30(F:0,B:0,V:1,L:1,S:0,P:0), Weight:0, Label: 51
      P2P IPv4 Adj-SID:      51, Weight:   0, Flags: --VL--
     P2P IPv4 Adj-SID:      45, Weight:   0, Flags: --VL--
     P2P IPv4 Adj-SID:      46, Weight:   0, Flags: --VL--
    NLPID: 0x83, Fixed length: 27 bytes, Version: 1, Sysid length: 0 bytes
      Node SID, Flags: 0x40(R:0,N:1,P:0,E:0,V:0,L:0), Algo: SPF(0), Value: 44
      SPRING Capability - Flags: 0xc0(I:1,V:1), Range: 1000, SID-Label: 100000
      P2P IPV4 Adj-SID - Flags:0x30(F:0,B:0,V:1,L:1,S:0,P:0), Weight:0, Label: 45
      P2P IPv4 Adj-SID:      45, Weight:   0, Flags: --VL--
      P2P IPV4 Adj-SID - Flags:0x30(F:0,B:0,V:1,L:1,S:0,P:0), Weight:0, Label: 46
      P2P IPv4 Adj-SID:      46, Weight:   0, Flags: --VL--
     P2P IPv4 Adj-SID:      47, Weight:   0, Flags: --VL--
     P2P IPv4 Adj-SID:      48, Weight:   0, Flags: --VL--
    NLPID: 0x83, Fixed length: 27 bytes, Version: 1, Sysid length: 0 bytes
      Node SID, Flags: 0x40(R:0,N:1,P:0,E:0,V:0,L:0), Algo: SPF(0), Value: 45
      SPRING Capability - Flags: 0xc0(I:1,V:1), Range: 1000, SID-Label: 100000
      P2P IPV4 Adj-SID - Flags:0x30(F:0,B:0,V:1,L:1,S:0,P:0), Weight:0, Label: 47
      P2P IPv4 Adj-SID:      47, Weight:   0, Flags: --VL--
      P2P IPV4 Adj-SID - Flags:0x30(F:0,B:0,V:1,L:1,S:0,P:0), Weight:0, Label: 48
      P2P IPv4 Adj-SID:      48, Weight:   0, Flags: --VL--
     P2P IPv4 Adj-SID:      50, Weight:   0, Flags: --VL--
     P2P IPv4 Adj-SID:      49, Weight:   0, Flags: --VL--
    NLPID: 0x83, Fixed length: 27 bytes, Version: 1, Sysid length: 0 bytes
      Node SID, Flags: 0x40(R:0,N:1,P:0,E:0,V:0,L:0), Algo: SPF(0), Value: 46
      Node SID, Flags: 0xe0(R:1,N:1,P:1,E:0,V:0,L:0), Algo: SPF(0), Value: 47
      Node SID, Flags: 0xe0(R:1,N:1,P:1,E:0,V:0,L:0), Algo: SPF(0), Value: 48
      SPRING Capability - Flags: 0xc0(I:1,V:1), Range: 1000, SID-Label: 100000
      P2P IPV4 Adj-SID - Flags:0x30(F:0,B:0,V:1,L:1,S:0,P:0), Weight:0, Label: 49
      P2P IPv4 Adj-SID:      49, Weight:   0, Flags: --VL--
      P2P IPV4 Adj-SID - Flags:0x30(F:0,B:0,V:1,L:1,S:0,P:0), Weight:0, Label: 50
      P2P IPv4 Adj-SID:      50, Weight:   0, Flags: --VL--

pradeep@R3> 
```

## IS-IS Adjacencies

```
pradeep@R1> show isis adjacency extensive 
R3
  Interface: ge-0/0/1.0, Level: 1, State: Up, Expires in 20 secs
  Priority: 0, Up/Down transitions: 1, Last transition: 10:24:37 ago
  Circuit type: 1, Speaks: IP, IPv6
  Topologies: Unicast
  Restart capable: Yes, Adjacency advertisement: Advertise
  IP addresses: 10.1.3.3
  Level 1 IPv4 Adj-SID: 50
  Transition log:
  When                  State        Event           Down reason
  Tue Aug 15 00:33:21   Up           Seenself        

pradeep@R1> exit 
```

```
pradeep@R2> show isis adjacency extensive 
R3
  Interface: ge-0/0/2.0, Level: 1, State: Up, Expires in 25 secs
  Priority: 0, Up/Down transitions: 1, Last transition: 09:20:52 ago
  Circuit type: 1, Speaks: IP, IPv6
  Topologies: Unicast
  Restart capable: Yes, Adjacency advertisement: Advertise
  IP addresses: 10.2.3.3
  Level 1 IPv4 Adj-SID: 16
  Transition log:
  When                  State        Event           Down reason
  Tue Aug 15 01:36:00   Up           Seenself        

pradeep@R2> exit
```

```
pradeep@R3> show isis adjacency extensive 
R1
  Interface: ge-0/0/1.0, Level: 1, State: Up, Expires in 21 secs
  Priority: 0, Up/Down transitions: 1, Last transition: 10:29:25 ago
  Circuit type: 1, Speaks: IP, IPv6
  Topologies: Unicast
  Restart capable: Yes, Adjacency advertisement: Advertise
  IP addresses: 10.1.3.1
  Level 1 IPv4 Adj-SID: 53
  Transition log:
  When                  State        Event           Down reason
  Tue Aug 15 00:28:09   Up           Seenself        

R2
  Interface: ge-0/0/2.0, Level: 1, State: Up, Expires in 24 secs
  Priority: 0, Up/Down transitions: 1, Last transition: 09:21:57 ago
  Circuit type: 1, Speaks: IP, IPv6
  Topologies: Unicast
  Restart capable: Yes, Adjacency advertisement: Advertise
  IP addresses: 10.2.3.2
  Level 1 IPv4 Adj-SID: 54
  Transition log:
  When                  State        Event           Down reason
  Tue Aug 15 01:35:37   Up           Seenself        

R4
  Interface: ge-0/0/4.0, Level: 2, State: Up, Expires in 21 secs
  Priority: 0, Up/Down transitions: 1, Last transition: 10:29:09 ago
  Circuit type: 3, Speaks: IP, IPv6
  Topologies: Unicast
  Restart capable: Yes, Adjacency advertisement: Advertise
  IP addresses: 10.3.4.4
  Level 2 IPv4 Adj-SID: 51
  Transition log:
  When                  State        Event           Down reason
  Tue Aug 15 00:28:24   Up           Seenself        

R5
  Interface: ge-0/0/5.0, Level: 2, State: Up, Expires in 22 secs
  Priority: 0, Up/Down transitions: 1, Last transition: 10:28:04 ago
  Circuit type: 3, Speaks: IP, IPv6
  Topologies: Unicast
  Restart capable: Yes, Adjacency advertisement: Advertise
  IP addresses: 10.3.5.5
  Level 2 IPv4 Adj-SID: 50
  Transition log:
  When                  State        Event           Down reason
  Tue Aug 15 00:29:30   Up           Seenself        

pradeep@R3> 
```

```
pradeep@R4> show isis adjacency extensive 
R6
  Interface: ge-0/0/1.0, Level: 2, State: Up, Expires in 21 secs
  Priority: 0, Up/Down transitions: 1, Last transition: 10:29:13 ago
  Circuit type: 2, Speaks: IP, IPv6
  Topologies: Unicast
  Restart capable: Yes, Adjacency advertisement: Advertise
  IP addresses: 10.4.6.6
  Level 2 IPv4 Adj-SID: 46
  Transition log:
  When                  State        Event           Down reason
  Tue Aug 15 00:28:51   Up           Seenself        

R3
  Interface: ge-0/0/4.0, Level: 2, State: Up, Expires in 19 secs
  Priority: 0, Up/Down transitions: 1, Last transition: 10:29:42 ago
  Circuit type: 2, Speaks: IP, IPv6
  Topologies: Unicast
  Restart capable: Yes, Adjacency advertisement: Advertise
  IP addresses: 10.3.4.3
  Level 2 IPv4 Adj-SID: 45
  Transition log:
  When                  State        Event           Down reason
  Tue Aug 15 00:28:23   Up           Seenself        

pradeep@R4> 
```

```
pradeep@R5> show isis adjacency extensive 
R6
  Interface: ge-0/0/2.0, Level: 2, State: Up, Expires in 21 secs
  Priority: 0, Up/Down transitions: 1, Last transition: 10:29:12 ago
  Circuit type: 2, Speaks: IP, IPv6
  Topologies: Unicast
  Restart capable: Yes, Adjacency advertisement: Advertise
  IP addresses: 10.5.6.6
  Level 2 IPv4 Adj-SID: 48
  Transition log:
  When                  State        Event           Down reason
  Tue Aug 15 00:29:19   Up           Seenself        

R3
  Interface: ge-0/0/5.0, Level: 2, State: Up, Expires in 21 secs
  Priority: 0, Up/Down transitions: 1, Last transition: 10:29:01 ago
  Circuit type: 2, Speaks: IP, IPv6
  Topologies: Unicast
  Restart capable: Yes, Adjacency advertisement: Advertise
  IP addresses: 10.3.5.3
  Level 2 IPv4 Adj-SID: 47
  Transition log:
  When                  State        Event           Down reason
  Tue Aug 15 00:29:30   Up           Seenself        

pradeep@R5> 
```

```
pradeep@R6> show isis adjacency extensive 
R4
  Interface: ge-0/0/1.0, Level: 2, State: Up, Expires in 22 secs
  Priority: 0, Up/Down transitions: 1, Last transition: 10:30:14 ago
  Circuit type: 3, Speaks: IP, IPv6
  Topologies: Unicast
  Restart capable: Yes, Adjacency advertisement: Advertise
  IP addresses: 10.4.6.4
  Level 2 IPv4 Adj-SID: 50
  Transition log:
  When                  State        Event           Down reason
  Tue Aug 15 00:28:25   Up           Seenself        

R5
  Interface: ge-0/0/2.0, Level: 2, State: Up, Expires in 21 secs
  Priority: 0, Up/Down transitions: 1, Last transition: 10:29:48 ago
  Circuit type: 3, Speaks: IP, IPv6
  Topologies: Unicast
  Restart capable: Yes, Adjacency advertisement: Advertise
  IP addresses: 10.5.6.5
  Level 2 IPv4 Adj-SID: 49
  Transition log:
  When                  State        Event           Down reason
  Tue Aug 15 00:28:50   Up           Seenself        

R7
  Interface: ge-0/0/4.0, Level: 1, State: Up, Expires in 19 secs
  Priority: 0, Up/Down transitions: 1, Last transition: 10:30:06 ago
  Circuit type: 1, Speaks: IP, IPv6
  Topologies: Unicast
  Restart capable: Yes, Adjacency advertisement: Advertise
  IP addresses: 10.6.7.7
  Level 1 IPv4 Adj-SID: 48
  Transition log:
  When                  State        Event           Down reason
  Tue Aug 15 00:28:33   Up           Seenself        

R8
  Interface: ge-0/0/5.0, Level: 1, State: Up, Expires in 24 secs
  Priority: 0, Up/Down transitions: 1, Last transition: 09:24:28 ago
  Circuit type: 1, Speaks: IP, IPv6
  Topologies: Unicast
  Restart capable: Yes, Adjacency advertisement: Advertise
  IP addresses: 10.6.8.8
  Level 1 IPv4 Adj-SID: 47
  Transition log:
  When                  State        Event           Down reason
  Tue Aug 15 01:34:11   Up           Seenself        

pradeep@R6> 
```

```
pradeep@R7> show isis adjacency extensive 
R6
  Interface: ge-0/0/4.0, Level: 1, State: Up, Expires in 24 secs
  Priority: 0, Up/Down transitions: 1, Last transition: 10:30:46 ago
  Circuit type: 1, Speaks: IP, IPv6
  Topologies: Unicast
  Restart capable: Yes, Adjacency advertisement: Advertise
  IP addresses: 10.6.7.6
  Level 1 IPv4 Adj-SID: 16
  Transition log:
  When                  State        Event           Down reason
  Tue Aug 15 00:29:04   Up           Seenself        

pradeep@R7> 
```

```
pradeep@R8> show isis adjacency extensive 
R6
  Interface: ge-0/0/5.0, Level: 1, State: Up, Expires in 23 secs
  Priority: 0, Up/Down transitions: 1, Last transition: 09:25:33 ago
  Circuit type: 1, Speaks: IP, IPv6
  Topologies: Unicast
  Restart capable: Yes, Adjacency advertisement: Advertise
  IP addresses: 10.6.8.6
  Level 1 IPv4 Adj-SID: 17
  Transition log:
  When                  State        Event           Down reason
  Tue Aug 15 01:40:58   Up           Seenself        

pradeep@R8> 
```

## Adjacency SIDs

```
pradeep@R1> show isis database extensive | match adj 
     P2P IPv4 Adj-SID:      50, Weight:   0, Flags: --VL--
      P2P IPV4 Adj-SID - Flags:0x30(F:0,B:0,V:1,L:1,S:0,P:0), Weight:0, Label: 50
      P2P IPv4 Adj-SID:      50, Weight:   0, Flags: --VL--
     P2P IPv4 Adj-SID:      16, Weight:   0, Flags: --VL--
      P2P IPV4 Adj-SID - Flags:0x30(F:0,B:0,V:1,L:1,S:0,P:0), Weight:0, Label: 16
      P2P IPv4 Adj-SID:      16, Weight:   0, Flags: --VL--
     P2P IPv4 Adj-SID:      53, Weight:   0, Flags: --VL--
     P2P IPv4 Adj-SID:      54, Weight:   0, Flags: --VL--
      P2P IPV4 Adj-SID - Flags:0x30(F:0,B:0,V:1,L:1,S:0,P:0), Weight:0, Label: 53
      P2P IPv4 Adj-SID:      53, Weight:   0, Flags: --VL--
      P2P IPV4 Adj-SID - Flags:0x30(F:0,B:0,V:1,L:1,S:0,P:0), Weight:0, Label: 54
      P2P IPv4 Adj-SID:      54, Weight:   0, Flags: --VL--

pradeep@R1> 
```

```
pradeep@R2> show isis database extensive | match adj 
     P2P IPv4 Adj-SID:      50, Weight:   0, Flags: --VL--
      P2P IPV4 Adj-SID - Flags:0x30(F:0,B:0,V:1,L:1,S:0,P:0), Weight:0, Label: 50
      P2P IPv4 Adj-SID:      50, Weight:   0, Flags: --VL--
     P2P IPv4 Adj-SID:      16, Weight:   0, Flags: --VL--
      P2P IPV4 Adj-SID - Flags:0x30(F:0,B:0,V:1,L:1,S:0,P:0), Weight:0, Label: 16
      P2P IPv4 Adj-SID:      16, Weight:   0, Flags: --VL--
     P2P IPv4 Adj-SID:      53, Weight:   0, Flags: --VL--
     P2P IPv4 Adj-SID:      54, Weight:   0, Flags: --VL--
      P2P IPV4 Adj-SID - Flags:0x30(F:0,B:0,V:1,L:1,S:0,P:0), Weight:0, Label: 53
      P2P IPv4 Adj-SID:      53, Weight:   0, Flags: --VL--
      P2P IPV4 Adj-SID - Flags:0x30(F:0,B:0,V:1,L:1,S:0,P:0), Weight:0, Label: 54
      P2P IPv4 Adj-SID:      54, Weight:   0, Flags: --VL--

pradeep@R2> 
```

```
pradeep@R3> show isis database extensive | match adj 
     P2P IPv4 Adj-SID:      50, Weight:   0, Flags: --VL--
      P2P IPV4 Adj-SID - Flags:0x30(F:0,B:0,V:1,L:1,S:0,P:0), Weight:0, Label: 50
      P2P IPv4 Adj-SID:      50, Weight:   0, Flags: --VL--
     P2P IPv4 Adj-SID:      16, Weight:   0, Flags: --VL--
      P2P IPV4 Adj-SID - Flags:0x30(F:0,B:0,V:1,L:1,S:0,P:0), Weight:0, Label: 16
      P2P IPv4 Adj-SID:      16, Weight:   0, Flags: --VL--
     P2P IPv4 Adj-SID:      53, Weight:   0, Flags: --VL--
     P2P IPv4 Adj-SID:      54, Weight:   0, Flags: --VL--
      P2P IPV4 Adj-SID - Flags:0x30(F:0,B:0,V:1,L:1,S:0,P:0), Weight:0, Label: 53
      P2P IPv4 Adj-SID:      53, Weight:   0, Flags: --VL--
      P2P IPV4 Adj-SID - Flags:0x30(F:0,B:0,V:1,L:1,S:0,P:0), Weight:0, Label: 54
      P2P IPv4 Adj-SID:      54, Weight:   0, Flags: --VL--
     P2P IPv4 Adj-SID:      51, Weight:   0, Flags: --VL--
     P2P IPv4 Adj-SID:      50, Weight:   0, Flags: --VL--
      P2P IPV4 Adj-SID - Flags:0x30(F:0,B:0,V:1,L:1,S:0,P:0), Weight:0, Label: 50
      P2P IPv4 Adj-SID:      50, Weight:   0, Flags: --VL--
      P2P IPV4 Adj-SID - Flags:0x30(F:0,B:0,V:1,L:1,S:0,P:0), Weight:0, Label: 51
      P2P IPv4 Adj-SID:      51, Weight:   0, Flags: --VL--
     P2P IPv4 Adj-SID:      45, Weight:   0, Flags: --VL--
     P2P IPv4 Adj-SID:      46, Weight:   0, Flags: --VL--
      P2P IPV4 Adj-SID - Flags:0x30(F:0,B:0,V:1,L:1,S:0,P:0), Weight:0, Label: 45
      P2P IPv4 Adj-SID:      45, Weight:   0, Flags: --VL--
      P2P IPV4 Adj-SID - Flags:0x30(F:0,B:0,V:1,L:1,S:0,P:0), Weight:0, Label: 46
      P2P IPv4 Adj-SID:      46, Weight:   0, Flags: --VL--
     P2P IPv4 Adj-SID:      47, Weight:   0, Flags: --VL--
     P2P IPv4 Adj-SID:      48, Weight:   0, Flags: --VL--
      P2P IPV4 Adj-SID - Flags:0x30(F:0,B:0,V:1,L:1,S:0,P:0), Weight:0, Label: 47
      P2P IPv4 Adj-SID:      47, Weight:   0, Flags: --VL--
      P2P IPV4 Adj-SID - Flags:0x30(F:0,B:0,V:1,L:1,S:0,P:0), Weight:0, Label: 48
      P2P IPv4 Adj-SID:      48, Weight:   0, Flags: --VL--
     P2P IPv4 Adj-SID:      50, Weight:   0, Flags: --VL--
     P2P IPv4 Adj-SID:      49, Weight:   0, Flags: --VL--
      P2P IPV4 Adj-SID - Flags:0x30(F:0,B:0,V:1,L:1,S:0,P:0), Weight:0, Label: 49
      P2P IPv4 Adj-SID:      49, Weight:   0, Flags: --VL--
      P2P IPV4 Adj-SID - Flags:0x30(F:0,B:0,V:1,L:1,S:0,P:0), Weight:0, Label: 50
      P2P IPv4 Adj-SID:      50, Weight:   0, Flags: --VL--

pradeep@R3> 
```

```
pradeep@R4> show isis database extensive | match adj 
     P2P IPv4 Adj-SID:      51, Weight:   0, Flags: --VL--
     P2P IPv4 Adj-SID:      50, Weight:   0, Flags: --VL--
      P2P IPV4 Adj-SID - Flags:0x30(F:0,B:0,V:1,L:1,S:0,P:0), Weight:0, Label: 50
      P2P IPv4 Adj-SID:      50, Weight:   0, Flags: --VL--
      P2P IPV4 Adj-SID - Flags:0x30(F:0,B:0,V:1,L:1,S:0,P:0), Weight:0, Label: 51
      P2P IPv4 Adj-SID:      51, Weight:   0, Flags: --VL--
     P2P IPv4 Adj-SID:      45, Weight:   0, Flags: --VL--
     P2P IPv4 Adj-SID:      46, Weight:   0, Flags: --VL--
      P2P IPV4 Adj-SID - Flags:0x30(F:0,B:0,V:1,L:1,S:0,P:0), Weight:0, Label: 45
      P2P IPv4 Adj-SID:      45, Weight:   0, Flags: --VL--
      P2P IPV4 Adj-SID - Flags:0x30(F:0,B:0,V:1,L:1,S:0,P:0), Weight:0, Label: 46
      P2P IPv4 Adj-SID:      46, Weight:   0, Flags: --VL--
     P2P IPv4 Adj-SID:      47, Weight:   0, Flags: --VL--
     P2P IPv4 Adj-SID:      48, Weight:   0, Flags: --VL--
      P2P IPV4 Adj-SID - Flags:0x30(F:0,B:0,V:1,L:1,S:0,P:0), Weight:0, Label: 47
      P2P IPv4 Adj-SID:      47, Weight:   0, Flags: --VL--
      P2P IPV4 Adj-SID - Flags:0x30(F:0,B:0,V:1,L:1,S:0,P:0), Weight:0, Label: 48
      P2P IPv4 Adj-SID:      48, Weight:   0, Flags: --VL--
     P2P IPv4 Adj-SID:      50, Weight:   0, Flags: --VL--
     P2P IPv4 Adj-SID:      49, Weight:   0, Flags: --VL--
      P2P IPV4 Adj-SID - Flags:0x30(F:0,B:0,V:1,L:1,S:0,P:0), Weight:0, Label: 49
      P2P IPv4 Adj-SID:      49, Weight:   0, Flags: --VL--
      P2P IPV4 Adj-SID - Flags:0x30(F:0,B:0,V:1,L:1,S:0,P:0), Weight:0, Label: 50
      P2P IPv4 Adj-SID:      50, Weight:   0, Flags: --VL--

pradeep@R4> 
```

```
pradeep@R5> show isis database extensive | match adj 
     P2P IPv4 Adj-SID:      51, Weight:   0, Flags: --VL--
     P2P IPv4 Adj-SID:      50, Weight:   0, Flags: --VL--
      P2P IPV4 Adj-SID - Flags:0x30(F:0,B:0,V:1,L:1,S:0,P:0), Weight:0, Label: 50
      P2P IPv4 Adj-SID:      50, Weight:   0, Flags: --VL--
      P2P IPV4 Adj-SID - Flags:0x30(F:0,B:0,V:1,L:1,S:0,P:0), Weight:0, Label: 51
      P2P IPv4 Adj-SID:      51, Weight:   0, Flags: --VL--
     P2P IPv4 Adj-SID:      45, Weight:   0, Flags: --VL--
     P2P IPv4 Adj-SID:      46, Weight:   0, Flags: --VL--
      P2P IPV4 Adj-SID - Flags:0x30(F:0,B:0,V:1,L:1,S:0,P:0), Weight:0, Label: 45
      P2P IPv4 Adj-SID:      45, Weight:   0, Flags: --VL--
      P2P IPV4 Adj-SID - Flags:0x30(F:0,B:0,V:1,L:1,S:0,P:0), Weight:0, Label: 46
      P2P IPv4 Adj-SID:      46, Weight:   0, Flags: --VL--
     P2P IPv4 Adj-SID:      47, Weight:   0, Flags: --VL--
     P2P IPv4 Adj-SID:      48, Weight:   0, Flags: --VL--
      P2P IPV4 Adj-SID - Flags:0x30(F:0,B:0,V:1,L:1,S:0,P:0), Weight:0, Label: 47
      P2P IPv4 Adj-SID:      47, Weight:   0, Flags: --VL--
      P2P IPV4 Adj-SID - Flags:0x30(F:0,B:0,V:1,L:1,S:0,P:0), Weight:0, Label: 48
      P2P IPv4 Adj-SID:      48, Weight:   0, Flags: --VL--
     P2P IPv4 Adj-SID:      50, Weight:   0, Flags: --VL--
     P2P IPv4 Adj-SID:      49, Weight:   0, Flags: --VL--
      P2P IPV4 Adj-SID - Flags:0x30(F:0,B:0,V:1,L:1,S:0,P:0), Weight:0, Label: 49
      P2P IPv4 Adj-SID:      49, Weight:   0, Flags: --VL--
      P2P IPV4 Adj-SID - Flags:0x30(F:0,B:0,V:1,L:1,S:0,P:0), Weight:0, Label: 50
      P2P IPv4 Adj-SID:      50, Weight:   0, Flags: --VL--

pradeep@R5> 
```

```
pradeep@R6> show isis database extensive | match adj 
     P2P IPv4 Adj-SID:      48, Weight:   0, Flags: --VL--
     P2P IPv4 Adj-SID:      47, Weight:   0, Flags: --VL--
      P2P IPV4 Adj-SID - Flags:0x30(F:0,B:0,V:1,L:1,S:0,P:0), Weight:0, Label: 48
      P2P IPv4 Adj-SID:      48, Weight:   0, Flags: --VL--
      P2P IPV4 Adj-SID - Flags:0x30(F:0,B:0,V:1,L:1,S:0,P:0), Weight:0, Label: 47
      P2P IPv4 Adj-SID:      47, Weight:   0, Flags: --VL--
     P2P IPv4 Adj-SID:      16, Weight:   0, Flags: --VL--
      P2P IPV4 Adj-SID - Flags:0x30(F:0,B:0,V:1,L:1,S:0,P:0), Weight:0, Label: 16
      P2P IPv4 Adj-SID:      16, Weight:   0, Flags: --VL--
     P2P IPv4 Adj-SID:      17, Weight:   0, Flags: --VL--
      P2P IPV4 Adj-SID - Flags:0x30(F:0,B:0,V:1,L:1,S:0,P:0), Weight:0, Label: 17
      P2P IPv4 Adj-SID:      17, Weight:   0, Flags: --VL--
     P2P IPv4 Adj-SID:      51, Weight:   0, Flags: --VL--
     P2P IPv4 Adj-SID:      50, Weight:   0, Flags: --VL--
      P2P IPV4 Adj-SID - Flags:0x30(F:0,B:0,V:1,L:1,S:0,P:0), Weight:0, Label: 50
      P2P IPv4 Adj-SID:      50, Weight:   0, Flags: --VL--
      P2P IPV4 Adj-SID - Flags:0x30(F:0,B:0,V:1,L:1,S:0,P:0), Weight:0, Label: 51
      P2P IPv4 Adj-SID:      51, Weight:   0, Flags: --VL--
     P2P IPv4 Adj-SID:      45, Weight:   0, Flags: --VL--
     P2P IPv4 Adj-SID:      46, Weight:   0, Flags: --VL--
      P2P IPV4 Adj-SID - Flags:0x30(F:0,B:0,V:1,L:1,S:0,P:0), Weight:0, Label: 45
      P2P IPv4 Adj-SID:      45, Weight:   0, Flags: --VL--
      P2P IPV4 Adj-SID - Flags:0x30(F:0,B:0,V:1,L:1,S:0,P:0), Weight:0, Label: 46
      P2P IPv4 Adj-SID:      46, Weight:   0, Flags: --VL--
     P2P IPv4 Adj-SID:      47, Weight:   0, Flags: --VL--
     P2P IPv4 Adj-SID:      48, Weight:   0, Flags: --VL--
      P2P IPV4 Adj-SID - Flags:0x30(F:0,B:0,V:1,L:1,S:0,P:0), Weight:0, Label: 47
      P2P IPv4 Adj-SID:      47, Weight:   0, Flags: --VL--
      P2P IPV4 Adj-SID - Flags:0x30(F:0,B:0,V:1,L:1,S:0,P:0), Weight:0, Label: 48
      P2P IPv4 Adj-SID:      48, Weight:   0, Flags: --VL--
     P2P IPv4 Adj-SID:      50, Weight:   0, Flags: --VL--
     P2P IPv4 Adj-SID:      49, Weight:   0, Flags: --VL--
      P2P IPV4 Adj-SID - Flags:0x30(F:0,B:0,V:1,L:1,S:0,P:0), Weight:0, Label: 49
      P2P IPv4 Adj-SID:      49, Weight:   0, Flags: --VL--
      P2P IPV4 Adj-SID - Flags:0x30(F:0,B:0,V:1,L:1,S:0,P:0), Weight:0, Label: 50
      P2P IPv4 Adj-SID:      50, Weight:   0, Flags: --VL--

pradeep@R6> 
```

```
pradeep@R7> show isis database extensive | match adj 
     P2P IPv4 Adj-SID:      48, Weight:   0, Flags: --VL--
     P2P IPv4 Adj-SID:      47, Weight:   0, Flags: --VL--
      P2P IPV4 Adj-SID - Flags:0x30(F:0,B:0,V:1,L:1,S:0,P:0), Weight:0, Label: 48
      P2P IPv4 Adj-SID:      48, Weight:   0, Flags: --VL--
      P2P IPV4 Adj-SID - Flags:0x30(F:0,B:0,V:1,L:1,S:0,P:0), Weight:0, Label: 47
      P2P IPv4 Adj-SID:      47, Weight:   0, Flags: --VL--
     P2P IPv4 Adj-SID:      16, Weight:   0, Flags: --VL--
      P2P IPV4 Adj-SID - Flags:0x30(F:0,B:0,V:1,L:1,S:0,P:0), Weight:0, Label: 16
      P2P IPv4 Adj-SID:      16, Weight:   0, Flags: --VL--
     P2P IPv4 Adj-SID:      17, Weight:   0, Flags: --VL--
      P2P IPV4 Adj-SID - Flags:0x30(F:0,B:0,V:1,L:1,S:0,P:0), Weight:0, Label: 17
      P2P IPv4 Adj-SID:      17, Weight:   0, Flags: --VL--

pradeep@R7> 
```

```
pradeep@R8> show isis database extensive | match adj 
     P2P IPv4 Adj-SID:      48, Weight:   0, Flags: --VL--
     P2P IPv4 Adj-SID:      47, Weight:   0, Flags: --VL--
      P2P IPV4 Adj-SID - Flags:0x30(F:0,B:0,V:1,L:1,S:0,P:0), Weight:0, Label: 48
      P2P IPv4 Adj-SID:      48, Weight:   0, Flags: --VL--
      P2P IPV4 Adj-SID - Flags:0x30(F:0,B:0,V:1,L:1,S:0,P:0), Weight:0, Label: 47
      P2P IPv4 Adj-SID:      47, Weight:   0, Flags: --VL--
     P2P IPv4 Adj-SID:      16, Weight:   0, Flags: --VL--
      P2P IPV4 Adj-SID - Flags:0x30(F:0,B:0,V:1,L:1,S:0,P:0), Weight:0, Label: 16
      P2P IPv4 Adj-SID:      16, Weight:   0, Flags: --VL--
     P2P IPv4 Adj-SID:      17, Weight:   0, Flags: --VL--
      P2P IPV4 Adj-SID - Flags:0x30(F:0,B:0,V:1,L:1,S:0,P:0), Weight:0, Label: 17
      P2P IPv4 Adj-SID:      17, Weight:   0, Flags: --VL--

pradeep@R8> 
```

With this we conclude that the labels can be consistent and easier to follow (or troubleshoot) the LSPs with Segment routing.
