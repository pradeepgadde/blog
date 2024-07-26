---

layout: single
title:  "Nokia SR Linux Streaming Telemetry Lab using GitHub Codespaces"
categories: Networking
tags: Nokia
classes: wide
show_date: true
header:
  overlay_image: /assets/images/clab.png
  og_image: /assets/images/clab.png
  teaser: /assets/images/clab.png
author:
  name     : "CONTAINERlab"
  avatar   : "/assets/images/clab.png"

sidebar:
  - title: "Topics"
    nav: my-sidebar

---
# Nokia SR Linux Streaming Telemetry Lab using GitHub Codespaces

GitHub Codespaces gets you up and coding faster with fully configured, secure cloud development environments native to GitHub. Individuals can use Codespaces for free each month for 60 hours, with  pay-as-you-go pricing after. Teams or Enterprises pay for Codespaces. A  maximum monthly cap can also be set for extra pricing control.

```sh
👋 Welcome to Codespaces! You are on a custom image defined in your devcontainer.json file.

🔍 To explore VS Code to its fullest, search using the Command Palette (Cmd/Ctrl + Shift + P)

📝 Edit away, then run your build command to see your code running in the browser.
@pradeepgadde ➜ /workspaces/srl-telemetry-lab (main) $ 

```
```json
{
    "image": "ghcr.io/srl-labs/containerlab/clab-devcontainer:0.55.0",
    "hostRequirements": {
        "cpus": 4,
        "memory": "16gb",
        "storage": "32gb"
    }
}
```

https://urban-happiness-xpjvx6544rpcpp7p.github.dev


This lab represents a small Clos fabric with [Nokia SR Linux](https://learn.srlinux.dev/) switches running as containers. The lab topology consists of a Clos topology, plus a Streaming Telemetry stack comprised of [gnmic](https://gnmic.openconfig.net/), prometheus and grafana applications.



In addition to the telemetry stack, the lab also includes a modern logging stack comprised of [promtail](https://grafana.com/docs/loki/latest/clients/promtail/) and [loki](https://grafana.com/oss/loki/).

Goals of this lab:

1. Demonstrate how a telemetry stack can be incorporated into the containerlab topology file.
2. Explain SR Linux holistic telemetry support.
3. Provide practical configuration examples for the gnmic collector to subscribe to fabric nodes and export metrics to Prometheus TSDB.
4. Introduce advanced Grafana dashboarding with [FlowChart](https://grafana.com/grafana/plugins/agenty-flowcharting-panel/) plugin rendering port speeds and statuses.
5. Give a sneak peek of the modern logging telemetry stack with Loki and Promtail to consume Syslog data from SR Linux nodes.

## Deploying the lab

The lab is deployed with the [containerlab](https://containerlab.dev/) project, where [`st.clab.yml`](https://vscode-remote+codespaces-002burban-002dhappiness-002dxpjvx6544rpcpp7p.vscode-resource.vscode-cdn.net/workspaces/srl-telemetry-lab/st.clab.yml) file declaratively describes the lab topology.



```bash
# change into the cloned directory
# and execute
containerlab deploy --reconfigure
```

```yaml
@pradeepgadde ➜ /workspaces/srl-telemetry-lab (main) $ cat st.clab.yml 
# Copyright 2020 Nokia
# Licensed under the BSD 3-Clause License.
# SPDX-License-Identifier: BSD-3-Clause

name: st # short for streaming telemetry ;)
prefix: ""

mgmt:
  network: st
  ipv4-subnet: 172.80.80.0/24

topology:
  defaults:
    kind: nokia_srlinux

  kinds:
    nokia_srlinux:
      image: ghcr.io/nokia/srlinux:24.7.1
      type: ixrd2l
    linux:
      image: ghcr.io/srl-labs/network-multitool

  nodes:
    ### SPINES ###
    spine1:
      type: ixrd3l
      group: spine
      startup-config: configs/fabric/spine1.cfg
      mgmt-ipv4: 172.80.80.21
    spine2:
      type: ixrd3l
      group: spine
      startup-config: configs/fabric/spine2.cfg
      mgmt-ipv4: 172.80.80.22

    ### LEAFS ###
    leaf1:
      startup-config: configs/fabric/leaf1.cfg
      mgmt-ipv4: 172.80.80.11
      group: leaf
    leaf2:
      startup-config: configs/fabric/leaf2.cfg
      mgmt-ipv4: 172.80.80.12
      group: leaf
    leaf3:
      startup-config: configs/fabric/leaf3.cfg
      mgmt-ipv4: 172.80.80.13
      group: leaf

    ### CLIENTS ###
    client1:
      kind: linux
      mgmt-ipv4: 172.80.80.31
      exec:
        - ip address add 172.17.0.1/24 dev eth1
        - ip -6 address add 2002::172:17:0:1/96 dev eth1
        - iperf3 -s -p 5201 -D > iperf3_1.log
        - iperf3 -s -p 5202 -D > iperf3_2.log
      group: server
    client2:
      kind: linux
      mgmt-ipv4: 172.80.80.32
      binds:
        - configs/client2:/config
      exec:
        - ip address add 172.17.0.2/24 dev eth1
        - ip -6 address add 2002::172:17:0:2/96 dev eth1
      group: server
    client3:
      kind: linux
      mgmt-ipv4: 172.80.80.33
      binds:
        - configs/client3:/config
      exec:
        - ip address add 172.17.0.3/24 dev eth1
        - ip -6 address add 2002::172:17:0:3/96 dev eth1
      group: server

    ### TELEMETRY STACK ###
    gnmic:
      kind: linux
      mgmt-ipv4: 172.80.80.41
      image: ghcr.io/openconfig/gnmic:0.38.1
      binds:
        - configs/gnmic/gnmic-config.yml:/gnmic-config.yml:ro
      cmd: --config /gnmic-config.yml --log subscribe
      group: "10" # group 10 is assigned to the nodes of a telemetry stack

    prometheus:
      kind: linux
      mgmt-ipv4: 172.80.80.42
      image: quay.io/prometheus/prometheus:v2.47.2
      binds:
        - configs/prometheus/prometheus.yml:/etc/prometheus/prometheus.yml:ro
      cmd: --config.file=/etc/prometheus/prometheus.yml
      ports:
        - 9090:9090
      group: "10"

    grafana:
      kind: linux
      mgmt-ipv4: 172.80.80.43
      image: grafana/grafana:10.2.1
      binds:
        - configs/grafana/datasource.yml:/etc/grafana/provisioning/datasources/datasource.yaml:ro
        - configs/grafana/dashboards.yml:/etc/grafana/provisioning/dashboards/dashboards.yaml:ro
        - configs/grafana/dashboards:/var/lib/grafana/dashboards
      ports:
        - 3000:3000
      env:
        GF_INSTALL_PLUGINS: https://algenty.github.io/flowcharting-repository/archives/agenty-flowcharting-panel-1.0.0d.220606199-SNAPSHOT.zip;agenty-flowcharting-panel
        # env vars to enable anonymous access
        GF_ORG_ROLE: "Admin"
        GF_ORG_NAME: "Main Org"
        GF_AUTH_ANONYMOUS_ENABLED: "true"
        GF_AUTH_ANONYMOUS: "true"
      group: "10"

    ### LOGGING STACK ###
    promtail:
      kind: linux
      mgmt-ipv4: 172.80.80.45
      image: grafana/promtail:2.9.2
      binds:
        - configs/promtail:/etc/promtail
      cmd: --config.file=/etc/promtail/promtail-config.yml
      ports:
        - 9080:9080

    loki:
      kind: linux
      mgmt-ipv4: 172.80.80.46
      image: grafana/loki:2.9.2
      binds:
        - configs/loki:/etc/loki
      cmd: --config.file=/etc/loki/loki-config.yml
      ports:
        - 3100:3100

  links:
    - endpoints: ["spine1:e1-1", "leaf1:e1-49"]
    - endpoints: ["spine1:e1-2", "leaf2:e1-49"]
    - endpoints: ["spine1:e1-3", "leaf3:e1-49"]
    - endpoints: ["spine2:e1-1", "leaf1:e1-50"]
    - endpoints: ["spine2:e1-2", "leaf2:e1-50"]
    - endpoints: ["spine2:e1-3", "leaf3:e1-50"]
    - endpoints: ["leaf1:e1-1", "client1:eth1"]
    - endpoints: ["leaf2:e1-1", "client2:eth1"]
    - endpoints: ["leaf3:e1-1", "client3:eth1"]
@pradeepgadde ➜ /workspaces/srl-telemetry-lab (main) $ 
```

```sh
@pradeepgadde ➜ /workspaces/srl-telemetry-lab (main) $ sudo containerlab deploy --reconfigure
INFO[0000] Containerlab v0.55.0 started                 
INFO[0000] Parsing & checking topology file: st.clab.yml 
INFO[0000] Removing /workspaces/srl-telemetry-lab/clab-st directory... 
INFO[0000] Creating docker network: Name="st", IPv4Subnet="172.80.80.0/24", IPv6Subnet="", MTU=1500 
INFO[0000] Pulling ghcr.io/openconfig/gnmic:0.38.1 Docker image 
INFO[0002] Done pulling ghcr.io/openconfig/gnmic:0.38.1 
INFO[0002] Pulling ghcr.io/nokia/srlinux:24.7.1 Docker image 
INFO[0039] Done pulling ghcr.io/nokia/srlinux:24.7.1    
INFO[0039] Pulling ghcr.io/srl-labs/network-multitool:latest Docker image 
INFO[0054] Done pulling ghcr.io/srl-labs/network-multitool:latest 
INFO[0054] Pulling docker.io/grafana/promtail:2.9.2 Docker image 
INFO[0059] Done pulling docker.io/grafana/promtail:2.9.2 
INFO[0059] Pulling docker.io/grafana/loki:2.9.2 Docker image 
INFO[0062] Done pulling docker.io/grafana/loki:2.9.2    
INFO[0062] Pulling docker.io/grafana/grafana:10.2.1 Docker image 
INFO[0071] Done pulling docker.io/grafana/grafana:10.2.1 
INFO[0071] Pulling quay.io/prometheus/prometheus:v2.47.2 Docker image 
INFO[0077] Done pulling quay.io/prometheus/prometheus:v2.47.2 
WARN[0077] Unable to init module loader: stat /lib/modules/6.5.0-1022-azure/modules.dep: no such file or directory. Skipping... 
INFO[0077] Creating lab directory: /workspaces/srl-telemetry-lab/clab-st 
INFO[0077] Creating container: "loki"                   
INFO[0077] Creating container: "grafana"                
INFO[0077] Creating container: "client1"                
INFO[0077] Creating container: "leaf1"                  
INFO[0079] Creating container: "client2"                
INFO[0079] Creating container: "leaf3"                  
INFO[0079] Created link: leaf1:e1-1 <--> client1:eth1   
INFO[0079] Running postdeploy actions for Nokia SR Linux 'leaf1' node 
INFO[0080] Creating container: "prometheus"             
INFO[0081] Running postdeploy actions for Nokia SR Linux 'leaf3' node 
INFO[0082] Creating container: "client3"                
INFO[0082] Creating container: "spine2"                 
INFO[0083] Created link: leaf3:e1-1 <--> client3:eth1   
INFO[0084] Creating container: "gnmic"                  
INFO[0084] Created link: spine2:e1-1 <--> leaf1:e1-50   
INFO[0084] Created link: spine2:e1-3 <--> leaf3:e1-50   
INFO[0084] Running postdeploy actions for Nokia SR Linux 'spine2' node 
INFO[0085] Creating container: "leaf2"                  
INFO[0087] Created link: spine2:e1-2 <--> leaf2:e1-50   
INFO[0087] Created link: leaf2:e1-1 <--> client2:eth1   
INFO[0087] Running postdeploy actions for Nokia SR Linux 'leaf2' node 
INFO[0131] Creating container: "promtail"               
INFO[0132] Creating container: "spine1"                 
INFO[0133] Created link: spine1:e1-1 <--> leaf1:e1-49   
INFO[0133] Created link: spine1:e1-2 <--> leaf2:e1-49   
INFO[0133] Created link: spine1:e1-3 <--> leaf3:e1-49   
INFO[0133] Running postdeploy actions for Nokia SR Linux 'spine1' node 
INFO[0160] Executed command "ip address add 172.17.0.1/24 dev eth1" on the node "client1". stdout: 
INFO[0160] Executed command "ip -6 address add 2002::172:17:0:1/96 dev eth1" on the node "client1". stdout: 
INFO[0160] Executed command "iperf3 -s -p 5201 -D > iperf3_1.log" on the node "client1". stdout: 
INFO[0160] Executed command "iperf3 -s -p 5202 -D > iperf3_2.log" on the node "client1". stdout: 
INFO[0160] Executed command "ip address add 172.17.0.2/24 dev eth1" on the node "client2". stdout: 
INFO[0160] Executed command "ip -6 address add 2002::172:17:0:2/96 dev eth1" on the node "client2". stdout: 
INFO[0160] Executed command "ip address add 172.17.0.3/24 dev eth1" on the node "client3". stdout: 
INFO[0160] Executed command "ip -6 address add 2002::172:17:0:3/96 dev eth1" on the node "client3". stdout: 
INFO[0160] Adding containerlab host entries to /etc/hosts file 
INFO[0160] Adding ssh config for containerlab nodes     
INFO[0160] 🎉 New containerlab version 0.56.0 is available! Release notes: https://containerlab.dev/rn/0.56/
Run 'containerlab version upgrade' to upgrade or go check other installation options at https://containerlab.dev/install/ 
+----+------------+--------------+---------------------------------------+---------------+---------+-----------------+--------------+
| #  |    Name    | Container ID |                 Image                 |     Kind      |  State  |  IPv4 Address   | IPv6 Address |
+----+------------+--------------+---------------------------------------+---------------+---------+-----------------+--------------+
|  1 | client1    | 58bced7bdbb8 | ghcr.io/srl-labs/network-multitool    | linux         | running | 172.80.80.31/24 | N/A          |
|  2 | client2    | c9299a135755 | ghcr.io/srl-labs/network-multitool    | linux         | running | 172.80.80.32/24 | N/A          |
|  3 | client3    | 76b13ab02d22 | ghcr.io/srl-labs/network-multitool    | linux         | running | 172.80.80.33/24 | N/A          |
|  4 | gnmic      | 9c8bcdf06771 | ghcr.io/openconfig/gnmic:0.38.1       | linux         | running | 172.80.80.41/24 | N/A          |
|  5 | grafana    | 5311e1599a98 | grafana/grafana:10.2.1                | linux         | running | 172.80.80.43/24 | N/A          |
|  6 | leaf1      | ab713b6e1e0c | ghcr.io/nokia/srlinux:24.7.1          | nokia_srlinux | running | 172.80.80.11/24 | N/A          |
|  7 | leaf2      | 393baa2fa810 | ghcr.io/nokia/srlinux:24.7.1          | nokia_srlinux | running | 172.80.80.12/24 | N/A          |
|  8 | leaf3      | 3b07538996bf | ghcr.io/nokia/srlinux:24.7.1          | nokia_srlinux | running | 172.80.80.13/24 | N/A          |
|  9 | loki       | e1e4ca0cf1cd | grafana/loki:2.9.2                    | linux         | running | 172.80.80.46/24 | N/A          |
| 10 | prometheus | 6cc9ffa56077 | quay.io/prometheus/prometheus:v2.47.2 | linux         | running | 172.80.80.42/24 | N/A          |
| 11 | promtail   | cb172f1cd35a | grafana/promtail:2.9.2                | linux         | running | 172.80.80.45/24 | N/A          |
| 12 | spine1     | 0683df675a94 | ghcr.io/nokia/srlinux:24.7.1          | nokia_srlinux | running | 172.80.80.21/24 | N/A          |
| 13 | spine2     | 2e6f8714068c | ghcr.io/nokia/srlinux:24.7.1          | nokia_srlinux | running | 172.80.80.22/24 | N/A          |
+----+------------+--------------+---------------------------------------+---------------+---------+-----------------+--------------+
@pradeepgadde ➜ /workspaces/srl-telemetry-lab (main) $ 
```

## Accessing the network elements

Once the lab has been deployed, the different SR Linux nodes can be accessed via SSH through their management IP address, given in the summary displayed after the execution of the deploy command. It is also possible to reach those nodes directly via their hostname, defined in the topology file. Linux clients cannot be reached via SSH, as it is not enabled, but it is possible to connect to them with a docker exec command.

```sh
@pradeepgadde ➜ /workspaces/srl-telemetry-lab (main) $ ssh admin@leaf1
Warning: Permanently added 'leaf1' (ED25519) to the list of known hosts.
................................................................
:                  Welcome to Nokia SR Linux!                  :
:              Open Network OS for the NetOps era.             :
:                                                              :
:    This is a freely distributed official container image.    :
:                      Use it - Share it                       :
:                                                              :
: Get started: https://learn.srlinux.dev                       :
: Container:   https://go.srlinux.dev/container-image          :
: Docs:        https://doc.srlinux.dev/24-7                    :
: Rel. notes:  https://doc.srlinux.dev/rn24-7-1                :
: YANG:        https://yang.srlinux.dev/release/v24.7.1        :
: Discord:     https://go.srlinux.dev/discord                  :
: Contact:     https://go.srlinux.dev/contact-sales            :
................................................................

Using configuration file(s): ['/home/admin/.srlinuxrc']
Welcome to the srlinux CLI.
Type 'help' (and press <ENTER>) if you need any help using this.

--{ running }--[  ]--
A:leaf1#
```

### Verifying the underlay and overlay status

The underlay network runs eBGP, while iBGP is used for the overlay network. The Layer 2 EVPN service is configured as explained in this comprehensive tutorial: [L2EVPN on Nokia SR Linux](https://learn.srlinux.dev/tutorials/l2evpn/intro/).

By connecting via SSH to one of the leaves, we can verify the status of those BGP sessions.

```sh
--{ running }--[  ]--
A:leaf1# show network-instance default protocols bgp neighbor
-------------------------------------------------------------------------------------------------------------------------
BGP neighbor summary for network-instance "default"
Flags: S static, D dynamic, L discovered by LLDP, B BFD enabled, - disabled, * slow
-------------------------------------------------------------------------------------------------------------------------
-------------------------------------------------------------------------------------------------------------------------
+------------+------------+------------+------------+------------+------------+------------+------------+------------+
|  Net-Inst  |    Peer    |   Group    |   Flags    |  Peer-AS   |   State    |   Uptime   |  AFI/SAFI  | [Rx/Active |
|            |            |            |            |            |            |            |            |    /Tx]    |
+============+============+============+============+============+============+============+============+============+
| default    | 10.0.2.1   | iBGP-      | S          | 100        | establishe | 0d:0h:3m:3 | evpn       | [4/4/2]    |
|            |            | overlay    |            |            | d          | 5s         |            |            |
| default    | 10.0.2.2   | iBGP-      | S          | 100        | establishe | 0d:0h:3m:4 | evpn       | [4/0/2]    |
|            |            | overlay    |            |            | d          | 4s         |            |            |
| default    | 192.168.11 | eBGP       | S          | 201        | establishe | 0d:0h:3m:4 | ipv4-      | [3/3/2]    |
|            | .1         |            |            |            | d          | 2s         | unicast    |            |
| default    | 192.168.12 | eBGP       | S          | 202        | establishe | 0d:0h:3m:5 | ipv4-      | [3/3/4]    |
|            | .1         |            |            |            | d          | 0s         | unicast    |            |
+------------+------------+------------+------------+------------+------------+------------+------------+------------+
-------------------------------------------------------------------------------------------------------------------------
Summary:
4 configured neighbors, 4 configured sessions are established, 0 disabled peers
0 dynamic peers

--{ running }--[  ]--
A:leaf1#
Current mode: running                                                                             admin (31)  Fri 10:29PM
```

## Telemetry stack

As the lab name suggests, telemetry is at its core. The following telemetry stack is used in this lab:

| Role                | Software                               |
| :------------------ | :------------------------------------- |
| Telemetry collector | [gnmic](https://gnmic.openconfig.net/) |
| Time-Series DB      | [prometheus](https://prometheus.io/)   |
| Visualization       | [grafana](https://grafana.com/)        |

### gnmic

[gnmic](https://gnmic.openconfig.net/) is an Openconfig project that allows to subscribe to streaming telemetry data from network devices and export it to a variety of destinations. In this lab, gnmic is used to subscribe to the telemetry data from the fabric nodes and export it to the prometheus time-series database.

The gnmic configuration file - [gnmic-config.yml](https://vscode-remote+codespaces-002burban-002dhappiness-002dxpjvx6544rpcpp7p.vscode-resource.vscode-cdn.net/workspaces/srl-telemetry-lab/gnmic-config.yml) - is applied to the gnmic container at the startup and instructs it to subscribe to the telemetry data and export it to the prometheus time-series database.

### Prometheus

[Prometheus](https://prometheus.io/) is a popular open-source time-series database. It is used in this lab to store the telemetry data exported by gnmic. The prometheus configuration file - [configs/prometheus/prometheus.yml](https://vscode-remote+codespaces-002burban-002dhappiness-002dxpjvx6544rpcpp7p.vscode-resource.vscode-cdn.net/workspaces/srl-telemetry-lab/configs/prometheus/prometheus.yml) - has a minimal configuration and instructs prometheus to scrape the data from the gnmic collector with a 5s interval.

### Grafana

Grafana is another key component of this lab as it provides the visualisation for the collected telemetry data. Lab's topology file includes grafana node and configuration parameters such as dashboards, datasources and required plugins.

Grafana dashboard provided by this repository provides multiple views on the collected real-time data. Powered by [flowchart plugin](https://grafana.com/grafana/plugins/agenty-flowcharting-panel/) it overlays telemetry sourced data over graphics such as topology and front panel views:

Using the flowchart plugin and real telemetry data users can create interactive topology maps (aka weathermap) with a visual indication of link rate/utilization.



### Access details

Using containerlab's ability to expose ports of the containers to the host, the following services are available on the host machine:

- Grafana: [http://localhost:3000](http://localhost:3000/). Anonymous access is enabled; no credentials are required. If you want to act as an admin, use `admin/admin` credentials.
- Prometheus: http://localhost:9090/graph

## Traffic generation

When the lab is started, there is not traffic running between the nodes as the clients are sending any data. To run traffic between the nodes, leverage `traffic.sh` control script.

To start the traffic:

- `bash traffic.sh start all` - start traffic between all nodes
- `bash traffic.sh start 1-2` - start traffic between client1 and client2
- `bash traffic.sh start 1-3` - start traffic between client1 and client3

To stop the traffic:

- `bash traffic.sh stop` - stop traffic generation between all nodes
- `bash traffic.sh stop 1-2` - stop traffic generation between client1 and client2
- `bash traffic.sh stop 1-3` - stop traffic generation between client1 and client3

As a result, the traffic will be generated between the clients and the traffic rate will be reflected on the grafana dashboard.

https://github.com/srl-labs/srl-telemetry-lab/assets/5679861/158914fc-9100-416b-8b0f-cde932895cec

```sh
@pradeepgadde ➜ /workspaces/srl-telemetry-lab (main) $ bash traffic.sh start all
starting traffic on all clients
Connecting to host 2002::172:17:0:1, port 5201
[  5] local 2002::172:17:0:2 port 36005 connected to 2002::172:17:0:1 port 5201
[  7] local 2002::172:17:0:2 port 48763 connected to 2002::172:17:0:1 port 5201
[  9] local 2002::172:17:0:2 port 52073 connected to 2002::172:17:0:1 port 5201
[ 11] local 2002::172:17:0:2 port 50467 connected to 2002::172:17:0:1 port 5201
[ 13] local 2002::172:17:0:2 port 34289 connected to 2002::172:17:0:1 port 5201
[ 15] local 2002::172:17:0:2 port 59131 connected to 2002::172:17:0:1 port 5201
[ 17] local 2002::172:17:0:2 port 59145 connected to 2002::172:17:0:1 port 5201
[ 19] local 2002::172:17:0:2 port 49093 connected to 2002::172:17:0:1 port 5201
[ ID] Interval           Transfer     Bitrate         Retr  Cwnd
[  5]   0.00-1.00   sec   107 KBytes   880 Kbits/sec    2   1.41 KBytes       
[  7]   0.00-1.00   sec   107 KBytes   880 Kbits/sec    3   1.41 KBytes       
[  9]   0.00-1.00   sec   107 KBytes   880 Kbits/sec    3   1.41 KBytes       
[ 11]   0.00-1.00   sec   107 KBytes   880 Kbits/sec    2   1.41 KBytes       
[ 13Connecting to host 2002::172:17:0:1, port 5202
[  5] local 2002::172:17:0:3 port 49989 connected to 2002::172:17:0:1 port 5202
[  7] local 2002::172:17:0:3 port 45813 connected to 2002::172:17:0:1 port 5202
[  9] local 2002::172:17:0:3 port 52445 connected to 2002::172:17:0:1 port 5202
[ 11] local 2002::172:17:0:3 port 54499 connected to 2002::172:17:0:1 port 5202
[ 13] local 2002::172:17:0:3 port 38357 connected to 2002::172:17:0:1 port 5202
[ 15] local 2002::172:17:0:3 port 59651 connected to 2002::172:17:0:1 port 5202
[ 17] local 2002::172:17:0:3 port 40319 connected to 2002::172:17:0:1 port 5202
[ 19] local 2002::172:17:0:3 port 55133 connected to 2002::172:17:0:1 port 5202
[ ID] Interval           Transfer     Bitrate         Retr  Cwnd
[  5]   0.00-1.00   sec   107 KBytes   880 Kbits/sec    2   1.41 KBytes       
[  7]   0.00-1.00   sec   107 KBytes   880 Kbits/sec    2   1.41 KBytes       
[  9]   0.00-1.00   sec   107 KBytes   880 Kbits/sec    2   1.41 KBytes       
[ 11]   0.00-1.00   sec   107 KBytes   880 Kbits/sec    2   1.41 KBytes       
[ 13]   0.00-1.00   sec   107 KBytes   880 Kbits/sec    2   1.41 KBytes       
[ 15]   0.00-1.00   sec   107 KBytes   880 Kbits/sec    2   1.41 KBytes       
[ 17]   0.00-1.00   sec   107 KBytes   880 Kbits/sec    2   1.41 KBytes       
[ 19]   0.00-1.00   sec   107 KBytes   880 Kbits/sec    2   1.41 KBytes       
[SUM]   0.00-1.00   sec   860 KBytes  7.04 Mbits/sec   16             
- - - - - - - - - - - - - - - - - - - - - - - - -
[  5]   1.00-2.00   sec  0.00 Bytes  0.00 bits/sec   17   24.0 KBytes       
[  7]   1.00-2.00   sec  0.00 Bytes  0.00 bits/sec   11   22.6 KBytes       
[  9]   1.00-2.00   sec  0.00 Bytes  0.00 bits/sec   15   22.6 KBytes       
[ 11]   1.00-2.00   sec  0.00 Bytes  0.00 bits/sec    1   35.4 KBytes       
[ 13]   1.00-2.00   sec  0.00 Bytes  0.00 bits/sec   10   22.6 KBytes       
[ 15]   1.00-2.00   sec  0.00 Bytes  0.00 bits/sec   21   21.2 KBytes       
[ 17]   1.00-2.00   sec  0.00 Bytes  0.00 bits/sec   25   24.0 KBytes       
[ 19]   1.00-2.00   sec  0.00 Bytes  0.00 bits/sec    @pradeepgadde ➜ /workspaces/srl-telemetry-lab (main) $ 
```



## Logging stack

The logging stack leverages the promtail->Loki pipeline, where promtail is a log agent that extracts, transforms and ships logs to Loki, a log aggregation system.

The logging infrastructure logs every message from SR Linux that is above Info level. This includes all the BGP messages, all the system messages, all the interface state changes, etc. The dashboard provides a view on the collected logs and allows filtering on a per-application level.

```sh
A:leaf1# quit
Connection to leaf1 closed.
@pradeepgadde ➜ /workspaces/srl-telemetry-lab (main) $ sudo containerlab destroy
INFO[0000] Parsing & checking topology file: st.clab.yml 
INFO[0000] Destroying lab: st                           
INFO[0001] Removed container: client1                   
INFO[0001] Removed container: gnmic                     
INFO[0001] Removed container: client2                   
INFO[0002] Removed container: grafana                   
INFO[0002] Removed container: client3                   
INFO[0002] Removed container: leaf1                     
INFO[0002] Removed container: prometheus                
INFO[0002] Removed container: loki                      
INFO[0002] Removed container: promtail                  
INFO[0002] Removed container: leaf2                     
INFO[0002] Removed container: leaf3                     
INFO[0002] Removed container: spine1                    
INFO[0003] Removed container: spine2                    
INFO[0003] Removing containerlab host entries from /etc/hosts file 
INFO[0003] Removing ssh config for containerlab nodes   
@pradeepgadde ➜ /workspaces/srl-telemetry-lab (main) $ 
```

