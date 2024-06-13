---
layout: single
title:  "Juniper Mist Wired Assurance—Intro"
categories: Networking
tags: Mist
show_date: true
toc: true
toc_sticky: true
header:
  teaser: /assets/images/mist.png
author:
  name     : "Mist"
  avatar   : "/assets/images/mist.png"

sidebar:
  - title: "Blog"
    nav: my-sidebar

---

# Juniper Mist Wired Assurance—Intro
Mist Onboarding Process

- Day 0 Provision (ZTP for greenfield and Adop for brownfield)

- Day 1 Deploy (Template-based config, port/interface profiles, colour/colorless ports)

- Day 2+ Operate (Alerts, Monitor, Troubleshoot)

## Wired Assurance Options

- Wired Visibility (requires Marvis for Wired Network license) - including third-party switches

- Wired Assurance  (required Wired Assurance license, Marvis for Wired Network license is optional), only Juniper switches

  1. Cloud Monitored

  2. Cloud Managed

     Requirements:

     1. Onboarding to Mist Cloud

     2. Minimum Junos Version

     3. Wired Assurance Subscription


## Wired Visibility
Multivendor, requires Mist APs and Marvis VNA. Free. Uses LLDP/LLDP-MED connectivity between the switch and the AP.
 - Switch Firmware Compliance
 - Switch-AP Affinity
 - Power Compliance
 - Inactive Wired VLANs
 - Switch Outage blast radius to mobile users

## Wired Assurance
Wired SLEs - Throughput, Successful Connects, Switch Health

## Campus Fabric Architectures
- EVPN Multihoming
- Campus Fabric Core-Distribution
- Campus Fabric IP Clos

## Subscriptions
Under `Organization > Subscriptions`, two options: `Apply Activation Code` or `Add/Renew Subscriptions`

- Summary
- Orders

Types:

- sub-man (Wireless Management)
- sub-eng (vBLE Engagement)
- sub-ast (Asset Visibility)
- sub-ex (Wired Assurance)
- sub-me (Mist Edge)
- s-srx-1s (WAN Assurance)
- s-client-s (IoT Assurance)
- sub-pma (Premium Analytics)
- sub-vna (Marvis/Virtual Network Assistant)
- sub-ex-2s (Marvis)
- sub-srx-2s (Marvis)

## Integration

- Day 0 Seamless, simplified onboarding (Greenfield Claim/Brownfield Adoption)
- Day 1 Deploy template-based config with device and port profiles
- Day 2+ Operate with Wired User SLEs
- Day 2+ Correlate with Switch Events
- Day 2+ Marvis VNA with natural language processing

## Greenfield Claims

The Mist AI App

Switch Claim Code (for individual switch)

Activation Code (Bulk claim switches)

Automated APIs to claim and autoprovision switches

For ZTP, Outbound TCP over SSH port 2200

PHC on the switch (Switch senda PHC request with its serial number)

PHS (Juniper Redirect Server) on the Cloud (PHS redirects with switch with the MIst server info and certificate)

Mist response with Initial Config and then outbound TCP connection on port 2200

Assign the device to site

Device Management with NETCONF/SSH

Switch should be able to resolve `oc-term.mistsys.net`.

The device ID is of the format <org-ID>.<mac_addr>

## Brownfield Adoptopn

Using `Organization > Inventory > Switches`

Doesnt require any Mist APs, but wired assurance subscription required.

## Telemetry Collection

MAC Address Table, STP Info, Alarms, Chassis Inventory, DHCP Binding Info, Dot1X Info, Interface Stats, LLDP Info, PoE Controller Info will be sent to the Cloud.

Event Scripts (Pull) > PyAgent (Push) > CloudX (JMA) (Push)

JMA: Junos Mist Agent

`show version | match mist`

JMA does not use port 2200, instead it uses port 443.

Both PyAgent and JMA leverage JTI (Juniper Telemetry Interface)
