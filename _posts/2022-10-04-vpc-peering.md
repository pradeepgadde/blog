---

layout: single
title:  "VPC Peering"
date:   2022-10-04 13:59:04 +0530
categories: Cloud
tags: GCP
show_date: true
classes: wide
header:
  teaser: /assets/images/gcp.png
author:
  name     : "Google Cloud Platform"
  avatar   : "/assets/images/gcp.png"

sidebar:
  - title: "Blog"
   
    text: "Checkout other topics"
    nav: my-sidebar

---

# Configure VPC Network Peering Between Two Networks



VPC network peering allows you to build SaaS (Software as a service) ecosystems in Google Cloud, which makes services available privately across different VPC networks within and across organizations. This allows workloads to communicate in private RFC 1918 space.

VPC network peering gives you several advantages over using external IP addresses or VPNs to connect networks, including:

- **Network Latency:** Public IP networking results in higher latency than private networking.
- **Network Security:** Service owners do not need to have their services exposed to the public internet and deal with its associated risks.
- **Network Cost:** Google Cloud charges [egress bandwidth pricing](https://cloud.google.com/compute/pricing#internet_egress) for networks using external IPs to communicate, even if the traffic is within the same zone. If, however, the networks are peered, they can use internal IPs to communicate and save on those egress costs. [Regular network pricing](https://cloud.google.com/compute/pricing#network) still applies to all traffic.



In this post,

- Explore connectivity between non-peered VPC networks
- Configure VPC network peering
- Verify private communication between peered VPC networks
- Delete VPC network peering

In a peered VPC network, no subnet IP range can overlap with another subnet IP range.

![]({{ site.url }}{{ site.baseurl }}/assets/images/gcp-57.png)

> mynet-us-vm

```sh
Linux mynet-us-vm 5.10.0-18-cloud-amd64 #1 SMP Debian 5.10.140-1 (2022-09-02) x86_64

The programs included with the Debian GNU/Linux system are free software;
the exact distribution terms for each program are described in the
individual files in /usr/share/doc/*/copyright.

Debian GNU/Linux comes with ABSOLUTELY NO WARRANTY, to the extent
permitted by applicable law.
Creating directory '/home/student-02-fe5bae609b7b'.
student-02-fe5bae609b7b@mynet-us-vm:~$ ping -c 3 172.16.0.2
PING 172.16.0.2 (172.16.0.2) 56(84) bytes of data.
^C
--- 172.16.0.2 ping statistics ---
3 packets transmitted, 0 received, 100% packet loss, time 2049ms

student-02-fe5bae609b7b@mynet-us-vm:~$ ping -c 3 34.69.155.153
PING 34.69.155.153 (34.69.155.153) 56(84) bytes of data.
64 bytes from 34.69.155.153: icmp_seq=1 ttl=61 time=2.51 ms
64 bytes from 34.69.155.153: icmp_seq=2 ttl=61 time=0.673 ms
^C
--- 34.69.155.153 ping statistics ---
2 packets transmitted, 2 received, 0% packet loss, time 1002ms
rtt min/avg/max/mdev = 0.673/1.589/2.506/0.916 ms
student-02-fe5bae609b7b@mynet-us-vm:~$ 

```



The **mynet-us-vm** and **privatenet-us-vm** instances are in the same zone (us-central1-a) but in different VPC networks (**mynetwork** and **privatenet**). Because VPC network peering has not been configured between those networks, private communication fails between the instances of those networks



## Task 2. Configure VPC network peering

VPC network peering can be configured for different VPC networks within and across organizations. Configure the following peering connections in this project:

- **peering-1-2**: Peer **mynetwork** with **privatenet**
- **peering-2-1**: Peer **privatenet** with **mynetwork**

Each side of a peering association is set up independently. Peering is active only when the configuration from both sides matches.

We will need the following info. [Learn more](https://cloud.google.com/compute/docs/peering-networks)

1. The project ID (if you are connecting to a VPC network in another project)
2. The name of the VPC network you want to peer with

Note: The subnet IP ranges in peered VPC networks cannot overlap.

![]({{ site.url }}{{ site.baseurl }}/assets/images/gcp-58.png)

**Note:** At this point, the peering status remains **INACTIVE** because the other side has not been configured, the networks are not yet peered.

![]({{ site.url }}{{ site.baseurl }}/assets/images/gcp-59.png)



As expected, the peering status changes to **ACTIVE** when the configuration from both sides matches.

![]({{ site.url }}{{ site.baseurl }}/assets/images/gcp-60.png)



## Task 3. Verify private communication between peered VPC networks

Verify private RFC 1918 connectivity across **mynetwork** and **privatenet**



erify that routes have been established between **mynetwork** and **privatenet**.

- In the Cloud Console, on the **Navigation menu** (![Navigation menu](https://cdn.qwiklabs.com/tkgw1TDgj4Q%2BYKQUW4jUFd0O5OEKlUMBRYbhlCrF0WY%3D)), click **VPC network** > **Routes**. Notice that there is a route for each subnet in **mynetwork** with **peering-1-2** as the **Next hop**. Similarly, notice that there is a route for each subnet in **privatenet** with **peering-2-1** as the **Next hop**.

These routes were automatically created with the VPC peering connection.

**Note** User-configured routes are not propagated across peered networks. If you configure a route in a network to a destination in a VPC network, that destination will not be reachable from a peered network.

![]({{ site.url }}{{ site.baseurl }}/assets/images/gcp-61.png)



```sh
Linux mynet-us-vm 5.10.0-18-cloud-amd64 #1 SMP Debian 5.10.140-1 (2022-09-02) x86_64

The programs included with the Debian GNU/Linux system are free software;
the exact distribution terms for each program are described in the
individual files in /usr/share/doc/*/copyright.

Debian GNU/Linux comes with ABSOLUTELY NO WARRANTY, to the extent
permitted by applicable law.
Last login: Wed Oct  5 07:08:59 2022 from 35.235.241.19
student-02-fe5bae609b7b@mynet-us-vm:~$ ping -c 3 172.16.0.2
PING 172.16.0.2 (172.16.0.2) 56(84) bytes of data.
64 bytes from 172.16.0.2: icmp_seq=1 ttl=64 time=1.30 ms
64 bytes from 172.16.0.2: icmp_seq=2 ttl=64 time=0.346 ms
64 bytes from 172.16.0.2: icmp_seq=3 ttl=64 time=0.325 ms

--- 172.16.0.2 ping statistics ---
3 packets transmitted, 3 received, 0% packet loss, time 2027ms
rtt min/avg/max/mdev = 0.325/0.655/1.295/0.452 ms
student-02-fe5bae609b7b@mynet-us-vm:~$ 

```

This is working because of the route that was established by the peering connection.

```sh
Linux privatenet-us-vm 5.10.0-18-cloud-amd64 #1 SMP Debian 5.10.140-1 (2022-09-02) x86_64

The programs included with the Debian GNU/Linux system are free software;
the exact distribution terms for each program are described in the
individual files in /usr/share/doc/*/copyright.

Debian GNU/Linux comes with ABSOLUTELY NO WARRANTY, to the extent
permitted by applicable law.
Creating directory '/home/student-02-fe5bae609b7b'.
student-02-fe5bae609b7b@privatenet-us-vm:~$ ping -c 3 10.128.0.2
PING 10.128.0.2 (10.128.0.2) 56(84) bytes of data.
64 bytes from 10.128.0.2: icmp_seq=1 ttl=64 time=1.37 ms
64 bytes from 10.128.0.2: icmp_seq=2 ttl=64 time=0.341 ms
64 bytes from 10.128.0.2: icmp_seq=3 ttl=64 time=0.293 ms

--- 10.128.0.2 ping statistics ---
3 packets transmitted, 3 received, 0% packet loss, time 2008ms
rtt min/avg/max/mdev = 0.293/0.667/1.367/0.495 ms
student-02-fe5bae609b7b@privatenet-us-vm:~$
```

This is also working because of the route that was established by the peering connection.

To test Compute Engine DNS across peered networks, run the following command.

```sh
student-02-fe5bae609b7b@privatenet-us-vm:~$ ping -c 3 mynet-us-vm
ping: mynet-us-vm: Name or service not known
student-02-fe5bae609b7b@privatenet-us-vm:~$ 
```

Compute Engine internal DNS names created in a network are not accessible to peered networks. The IP address of the VM should be used to reach the VM instances in peered network.



Now that we've verified the VPC Peering connection, we could stop both instances to remove their external IP addresses. This helps secure our instances and reduces our networking costs. We can still SSH to an instance without a public IP address using [Cloud IAP tunnels.](https://cloud.google.com/iap/docs/using-tcp-forwarding#tunneling_with_ssh)

## Task 4. Delete the VPC Peering Connection

Delete the VPC Peering connection and verify the deletion.

Deleting one side of the peering connection terminates the overall peering connection.

Verify that **routes** no longer exist for the peering connection and that there is no private RFC 1918 connectivity across **mynetwork** and **privatenet**.

![]({{ site.url }}{{ site.baseurl }}/assets/images/gcp-62.png)

Deleting one side of the VPC Peering connection removes all peering routes.



In this lab, we configured VPC network peering between two networks (**privatenet** and **mynetwork**). Then we verified private RFC 1918 connectivity across **mynetwork** and **privatenet** by pinging VMs on their internal IP addresses within those networks. Finally, we deleted one side of the VPC network peering connection to demonstrate that this removes private RFC 1918 connectivity across those networks.
