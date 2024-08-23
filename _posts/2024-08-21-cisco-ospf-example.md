---
layout: single
title:  "Cisco IOS OSPF Example"
categories: Networking
tags: OSPF
show_date: true
classes: wide
header:
  teaser: /assets/images/cisco.png
author:
  name     : "Cisco"
  avatar   : "/assets/images/cisco.png"

sidebar:
  - title: "Topics"
    nav: my-sidebar
---
# Cisco IOS OSPF Example
In this example, there are two routers connected by a single link and OSPF is enabled to advertise the static routes into OSPF.

## R0
Here is the R0 configuration:

```py
R0>
R0>en
R0#show run
R0#show running-config 
Building configuration...

Current configuration : 864 bytes
!
version 15.1
no service timestamps log datetime msec
no service timestamps debug datetime msec
no service password-encryption
!
hostname R0
!
!
!
!
!
!
!
!
ip cef
no ipv6 cef
!
!
!
!
license udi pid CISCO2911/K9 sn FTX1524A0OO-
!
!
!
!
!
!
!
!
!
no ip domain-lookup
!
!
spanning-tree mode pvst
!
!
!
!
!
!
interface GigabitEthernet0/0
 ip address 192.168.1.1 255.255.255.0
 duplex auto
 speed auto
!
interface GigabitEthernet0/1
 no ip address
 duplex auto
 speed auto
 shutdown
!
interface GigabitEthernet0/2
 no ip address
 duplex auto
 speed auto
 shutdown
!
interface Vlan1
 no ip address
 shutdown
!
router ospf 1
 log-adjacency-changes
 redistribute static subnets 
 network 192.168.1.0 0.0.0.255 area 0
!
ip classless
ip route 10.0.0.0 255.255.255.0 Null0 
!
ip flow-export version 9
!
!
!
!
!
!
!
line con 0
!
line aux 0
!
line vty 0 4
 login
!
!
!
end


R0#show ip int br
Interface              IP-Address      OK? Method Status                Protocol 
GigabitEthernet0/0     192.168.1.1     YES manual up                    up 
GigabitEthernet0/1     unassigned      YES unset  administratively down down 
GigabitEthernet0/2     unassigned      YES unset  administratively down down 
Vlan1                  unassigned      YES unset  administratively down down
R0#show ip route
Codes: L - local, C - connected, S - static, R - RIP, M - mobile, B - BGP
       D - EIGRP, EX - EIGRP external, O - OSPF, IA - OSPF inter area
       N1 - OSPF NSSA external type 1, N2 - OSPF NSSA external type 2
       E1 - OSPF external type 1, E2 - OSPF external type 2, E - EGP
       i - IS-IS, L1 - IS-IS level-1, L2 - IS-IS level-2, ia - IS-IS inter area
       * - candidate default, U - per-user static route, o - ODR
       P - periodic downloaded static route

Gateway of last resort is not set

     10.0.0.0/24 is subnetted, 1 subnets
S       10.0.0.0/24 is directly connected, Null0
     172.16.0.0/24 is subnetted, 1 subnets
O E2    172.16.1.0/24 [110/20] via 192.168.1.2, 00:12:54, GigabitEthernet0/0
     192.168.1.0/24 is variably subnetted, 2 subnets, 2 masks
C       192.168.1.0/24 is directly connected, GigabitEthernet0/0
L       192.168.1.1/32 is directly connected, GigabitEthernet0/0

R0#show ip prot
R0#show ip protocols 

Routing Protocol is "ospf 1"
  Outgoing update filter list for all interfaces is not set 
  Incoming update filter list for all interfaces is not set 
  Router ID 192.168.1.1
  It is an autonomous system boundary router
  Redistributing External Routes from,
    static 
  Number of areas in this router is 1. 1 normal 0 stub 0 nssa
  Maximum path: 4
  Routing for Networks:
    192.168.1.0 0.0.0.255 area 0
  Routing Information Sources:  
    Gateway         Distance      Last Update 
    192.168.1.1          110      00:21:37
    192.168.1.2          110      00:13:03
  Distance: (default is 110)

R0#show ip ospf ne
R0#show ip ospf neighbor 


Neighbor ID     Pri   State           Dead Time   Address         Interface
192.168.1.2       1   FULL/BDR        00:00:31    192.168.1.2     GigabitEthernet0/0
R0#show ip ospf data
R0#show ip ospf database 
            OSPF Router with ID (192.168.1.1) (Process ID 1)

                Router Link States (Area 0)

Link ID         ADV Router      Age         Seq#       Checksum Link count
192.168.1.1     192.168.1.1     1309        0x80000004 0x00f4ad 1
192.168.1.2     192.168.1.2     795         0x80000005 0x00f0ad 1

                Net Link States (Area 0)
Link ID         ADV Router      Age         Seq#       Checksum
192.168.1.1     192.168.1.1     888         0x80000003 0x004f5c

                Type-5 AS External Link States
Link ID         ADV Router      Age         Seq#       Checksum Tag
10.0.0.0        192.168.1.1     1309        0x80000001 0x002e1d 0
172.16.1.0      192.168.1.2     795         0x80000001 0x001a7c 0
R0#
```

## R1

Here is the R1 configuration:
```py
R1#show running-config 
Building configuration...

Current configuration : 866 bytes
!
version 15.1
no service timestamps log datetime msec
no service timestamps debug datetime msec
no service password-encryption
!
hostname R1
!
!
!
!
!
!
!
!
ip cef
no ipv6 cef
!
!
!
!
license udi pid CISCO2911/K9 sn FTX1524NTR6-
!
!
!
!
!
!
!
!
!
no ip domain-lookup
!
!
spanning-tree mode pvst
!
!
!
!
!
!
interface GigabitEthernet0/0
 ip address 192.168.1.2 255.255.255.0
 duplex auto
 speed auto
!
interface GigabitEthernet0/1
 no ip address
 duplex auto
 speed auto
 shutdown
!
interface GigabitEthernet0/2
 no ip address
 duplex auto
 speed auto
 shutdown
!
interface Vlan1
 no ip address
 shutdown
!
router ospf 1
 log-adjacency-changes
 redistribute static subnets 
 network 192.168.1.0 0.0.0.255 area 0
!
ip classless
ip route 172.16.1.0 255.255.255.0 Null0 
!
ip flow-export version 9
!
!
!
!
!
!
!
line con 0
!
line aux 0
!
line vty 0 4
 login
!
!
!
end


R1# 
```

```sh
R1#show ip int br
Interface              IP-Address      OK? Method Status                Protocol 
GigabitEthernet0/0     192.168.1.2     YES manual up                    up 
GigabitEthernet0/1     unassigned      YES unset  administratively down down 
GigabitEthernet0/2     unassigned      YES unset  administratively down down 
Vlan1                  unassigned      YES unset  administratively down down
R1#show ip route
Codes: L - local, C - connected, S - static, R - RIP, M - mobile, B - BGP
       D - EIGRP, EX - EIGRP external, O - OSPF, IA - OSPF inter area
       N1 - OSPF NSSA external type 1, N2 - OSPF NSSA external type 2
       E1 - OSPF external type 1, E2 - OSPF external type 2, E - EGP
       i - IS-IS, L1 - IS-IS level-1, L2 - IS-IS level-2, ia - IS-IS inter area
       * - candidate default, U - per-user static route, o - ODR
       P - periodic downloaded static route

Gateway of last resort is not set

     10.0.0.0/24 is subnetted, 1 subnets
O E2    10.0.0.0/24 [110/20] via 192.168.1.1, 00:17:50, GigabitEthernet0/0
     172.16.0.0/24 is subnetted, 1 subnets
S       172.16.1.0/24 is directly connected, Null0
     192.168.1.0/24 is variably subnetted, 2 subnets, 2 masks
C       192.168.1.0/24 is directly connected, GigabitEthernet0/0
L       192.168.1.2/32 is directly connected, GigabitEthernet0/0

R1#show ip pro
R1#show ip protocols 

Routing Protocol is "ospf 1"
  Outgoing update filter list for all interfaces is not set 
  Incoming update filter list for all interfaces is not set 
  Router ID 192.168.1.2
  It is an autonomous system boundary router
  Redistributing External Routes from,
    static 
  Number of areas in this router is 1. 1 normal 0 stub 0 nssa
  Maximum path: 4
  Routing for Networks:
    192.168.1.0 0.0.0.255 area 0
  Routing Information Sources:  
    Gateway         Distance      Last Update 
    192.168.1.1          110      00:17:56
    192.168.1.2          110      00:09:22
  Distance: (default is 110)

R1#show ip ospf ne 


Neighbor ID     Pri   State           Dead Time   Address         Interface
192.168.1.1       1   FULL/DR         00:00:38    192.168.1.1     GigabitEthernet0/0
R1#show ip ospf da
R1#show ip ospf database 
            OSPF Router with ID (192.168.1.2) (Process ID 1)

                Router Link States (Area 0)

Link ID         ADV Router      Age         Seq#       Checksum Link count
192.168.1.1     192.168.1.1     1093        0x80000004 0x00f4ad 1
192.168.1.2     192.168.1.2     579         0x80000005 0x00f0ad 1

                Net Link States (Area 0)
Link ID         ADV Router      Age         Seq#       Checksum
192.168.1.1     192.168.1.1     672         0x80000003 0x004f5c

                Type-5 AS External Link States
Link ID         ADV Router      Age         Seq#       Checksum Tag
10.0.0.0        192.168.1.1     1093        0x80000001 0x002e1d 0
172.16.1.0      192.168.1.2     579         0x80000001 0x001a7c 0
R1#
```

## OSPF Database view from IOS

```py
R0#show ip ospf database ?
  asbr-summary  ASBR summary link states
  external      External link states
  network       Network link states
  router        Router link states
  summary       Network summary link states
  <cr>
R0#show ip ospf database ro
R0#show ip ospf database router 

            OSPF Router with ID (192.168.1.1) (Process ID 1)

                Router Link States (Area 0)

  LS age: 1364
  Options: (No TOS-capability, DC)
  LS Type: Router Links
  Link State ID: 192.168.1.1
  Advertising Router: 192.168.1.1
  LS Seq Number: 80000004
  Checksum: 0xf4ad
  Length: 36
  AS Boundary Router
  Number of Links: 1

    Link connected to: a Transit Network
     (Link ID) Designated Router address: 192.168.1.1
     (Link Data) Router Interface address: 192.168.1.1
      Number of TOS metrics: 0
       TOS 0 Metrics: 1

  LS age: 850
  Options: (No TOS-capability, DC)
  LS Type: Router Links
  Link State ID: 192.168.1.2
  Advertising Router: 192.168.1.2
  LS Seq Number: 80000005
  Checksum: 0xf0ad
  Length: 36
  AS Boundary Router
  Number of Links: 1

    Link connected to: a Transit Network
     (Link ID) Designated Router address: 192.168.1.1
     (Link Data) Router Interface address: 192.168.1.2
      Number of TOS metrics: 0
       TOS 0 Metrics: 1
R0#show ip ospf database n
R0#show ip ospf database network 

            OSPF Router with ID (192.168.1.1) (Process ID 1)

                Net Link States (Area 0)

  Routing Bit Set on this LSA
  LS age: 952
  Options: (No TOS-capability, DC)
  LS Type: Network Links
  Link State ID: 192.168.1.1  (address of Designated Router)
  Advertising Router: 192.168.1.1
  LS Seq Number: 80000003
  Checksum: 0x4f5c
  Length: 32
  Network Mask: /24
        Attached Router: 192.168.1.2
        Attached Router: 192.168.1.1
R0#show ip ospf database su
R0#show ip ospf database summary 

            OSPF Router with ID (192.168.1.1) (Process ID 1)
R0#show ip ospf database ex
R0#show ip ospf database external 

            OSPF Router with ID (192.168.1.1) (Process ID 1)

                Type-5 AS External Link States

  Routing Bit Set on this LSA
  LS age: 1384
  Options: (No TOS-capability, DC)
  LS Type: AS External Link
  Link State ID: 10.0.0.0 (External Network Number )
  Advertising Router: 192.168.1.1
  LS Seq Number: 80000001
  Checksum: 0x2e1d
  Length: 36
  Network Mask: /24
        Metric Type: 2 (Larger than any link state path)
        TOS: 0
        Metric: 20
        Forward Address: 0.0.0.0
        External Route Tag: 0

  Routing Bit Set on this LSA
  LS age: 870
  Options: (No TOS-capability, DC)
  LS Type: AS External Link
  Link State ID: 172.16.1.0 (External Network Number )
  Advertising Router: 192.168.1.2
  LS Seq Number: 80000001
  Checksum: 0x1a7c
  Length: 36
  Network Mask: /24
        Metric Type: 2 (Larger than any link state path)
        TOS: 0
        Metric: 20
        Forward Address: 0.0.0.0
        External Route Tag: 0
R0#show ip ospf database external ?
  <cr>
R0#show ip ospf database ?
  asbr-summary  ASBR summary link states
  external      External link states
  network       Network link states
  router        Router link states
  summary       Network summary link states
  <cr>
R0#show ip ospf database as
R0#show ip ospf database asbr-summary 

            OSPF Router with ID (192.168.1.1) (Process ID 1)
R0#
```

