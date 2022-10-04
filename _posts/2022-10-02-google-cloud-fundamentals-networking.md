---

layout: single
title:  "Google Cloud Fundamentals—Networking"
date:   2022-10-02 08:59:04 +0530
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

## Virtual Machines and Networks in the Cloud

- Virtual Private Cloud (VPC) Networking
- Compute Engine
- Scaling Virtual Machines
- Important VPC Capabilities
- Cloud Load Balancing
- Cloud DNS 
- Cloud CDN
- Connecting networks to Google VPC

Google Cloud Virtual Private Cloud (VPC) provides networking functionality to Compute Engine virtual machine (VM) instances, Kubernetes Engine containers, and App Engine flexible environment. In other words, without a VPC network you cannot create VM instances, containers, or App Engine applications. Therefore, each Google Cloud project has a **default** network to get you started.

You can think of a VPC network as similar to a physical network, except that it is virtualized within Google Cloud. A VPC network is a global resource that consists of a list of regional virtual subnetworks (subnets) in data centers, all connected by a global wide area network (WAN). VPC networks are logically isolated from each other in Google Cloud.



![]({{ site.url }}{{ site.baseurl }}/assets/images/gcp-4.png)

Routes tell VM instances and the VPC network how to send traffic from an instance to a destination, either inside the network or outside Google Cloud. Each VPC network comes with some default routes to route traffic among its subnets and send traffic from eligible instances to the internet.

![]({{ site.url }}{{ site.baseurl }}/assets/images/gcp-5.png)

![]({{ site.url }}{{ site.baseurl }}/assets/images/gcp-6.png)

Each VPC network implements a distributed virtual firewall that you can configure. Firewall rules allow you to control which packets are allowed to travel to which destinations. Every VPC network has two implied firewall rules that block all incoming connections and allow all outgoing connections.

![]({{ site.url }}{{ site.baseurl }}/assets/images/gcp-7.png)

If we delete the virtual network, we cant create any VMs

![]({{ site.url }}{{ site.baseurl }}/assets/images/gcp-8.png)



Create a new network

![]({{ site.url }}{{ site.baseurl }}/assets/images/gcp-9.png)

Create a new VM and Verify that the **Internal IP** for the new instance was assigned from the IP address range for the subnet in **europe-west4** (10.164.0.0/20).



The **Internal IP** should be 10.164.0.2 because 10.164.0.1 is reserved for the gateway and you have not configured any other instances in that subnet.

![]({{ site.url }}{{ site.baseurl }}/assets/images/gcp-10.png)

The **External IP addresses** for VM instances are ephemeral. If an instance is stopped, any ephemeral external IP addresses assigned to the instance are released back into the general Compute Engine pool and become available for use by other projects. When a stopped instance is started again, a new ephemeral external IP address is assigned to the instance.



Alternatively, you can reserve a static external IP address, which assigns the address to your project indefinitely until you explicitly release it.

If we try to delete the virtual network while it is in use, we will get an error.

![]({{ site.url }}{{ site.baseurl }}/assets/images/gcp-11.png)



Deleted the VPC and VM and created again this time with a new name.

![]({{ site.url }}{{ site.baseurl }}/assets/images/gcp-12.png)

Total of two VMs in each region

![]({{ site.url }}{{ site.baseurl }}/assets/images/gcp-13.png)

Explore the connectivity for the VM instances. Specifically, try to SSH to your VM instances using tcp:22, and ping both the internal and external IP addresses of your VM instances using ICMP. Then explore the effects of the firewall rules on connectivity by removing the firewall rules individually.

```sh
student-01-d8fa2820dfbd@mynet-us-vm:~$ ping -c 3 10.164.0.2
PING 10.164.0.2 (10.164.0.2) 56(84) bytes of data.
64 bytes from 10.164.0.2: icmp_seq=1 ttl=64 time=103 ms
64 bytes from 10.164.0.2: icmp_seq=2 ttl=64 time=101 ms
64 bytes from 10.164.0.2: icmp_seq=3 ttl=64 time=101 ms

--- 10.164.0.2 ping statistics ---
3 packets transmitted, 3 received, 0% packet loss, time 2004ms
rtt min/avg/max/mdev = 101.444/101.876/102.690/0.575 ms
student-01-d8fa2820dfbd@mynet-us-vm:~$ 
```

You can ping **mynet-eu-vm**'s internal IP because of the **allow-custom** firewall rule.

```sh
Linux mynet-us-vm 5.10.0-18-cloud-amd64 #1 SMP Debian 5.10.140-1 (2022-09-02) x86_64

The programs included with the Debian GNU/Linux system are free software;
the exact distribution terms for each program are described in the
individual files in /usr/share/doc/*/copyright.

Debian GNU/Linux comes with ABSOLUTELY NO WARRANTY, to the extent
permitted by applicable law.
Creating directory '/home/student-01-d8fa2820dfbd'.
student-01-d8fa2820dfbd@mynet-us-vm:~$ ping -c 3 34.90.55.19
PING 34.90.55.19 (34.90.55.19) 56(84) bytes of data.
64 bytes from 34.90.55.19: icmp_seq=1 ttl=51 time=104 ms
64 bytes from 34.90.55.19: icmp_seq=2 ttl=51 time=103 ms
64 bytes from 34.90.55.19: icmp_seq=3 ttl=51 time=103 ms

--- 34.90.55.19 ping statistics ---
3 packets transmitted, 3 received, 0% packet loss, time 2002ms
rtt min/avg/max/mdev = 102.774/103.242/104.166/0.653 ms
student-01-d8fa2820dfbd@mynet-us-vm:~$
```

**mynetwork-allow-icmp** is allowing the external connectivity.



![]({{ site.url }}{{ site.baseurl }}/assets/images/gcp-14.png)

Delete the Firewall rule and verify connectivity

```sh
student-01-d8fa2820dfbd@mynet-us-vm:~$ ping -c 3 10.164.0.2
PING 10.164.0.2 (10.164.0.2) 56(84) bytes of data.
64 bytes from 10.164.0.2: icmp_seq=1 ttl=64 time=111 ms
64 bytes from 10.164.0.2: icmp_seq=2 ttl=64 time=109 ms
64 bytes from 10.164.0.2: icmp_seq=3 ttl=64 time=109 ms

--- 10.164.0.2 ping statistics ---
3 packets transmitted, 3 received, 0% packet loss, time 2003ms
rtt min/avg/max/mdev = 109.194/109.671/110.562/0.630 ms
student-01-d8fa2820dfbd@mynet-us-vm:~$ ping -c 3 34.90.55.19
PING 34.90.55.19 (34.90.55.19) 56(84) bytes of data.

--- 34.90.55.19 ping statistics ---
3 packets transmitted, 0 received, 100% packet loss, time 2042ms

student-01-d8fa2820dfbd@mynet-us-vm:~$ 
```

![]({{ site.url }}{{ site.baseurl }}/assets/images/gcp-15.png)

After deleting one more rule

```sh
student-01-d8fa2820dfbd@mynet-us-vm:~$ ping -c 3 10.164.0.2
PING 10.164.0.2 (10.164.0.2) 56(84) bytes of data.

--- 10.164.0.2 ping statistics ---
3 packets transmitted, 0 received, 100% packet loss, time 2037ms

student-01-d8fa2820dfbd@mynet-us-vm:~$ ping -c 3 34.90.55.19
PING 34.90.55.19 (34.90.55.19) 56(84) bytes of data.

--- 34.90.55.19 ping statistics ---
3 packets transmitted, 0 received, 100% packet loss, time 2054ms

student-01-d8fa2820dfbd@mynet-us-vm:~$ 
```

The **100% packet loss** indicates that you cannot ping **mynet-eu-vm**'s internal IP. This is expected because you deleted the **allow-custom** firewall rule!



Delete SSH rule also

![]({{ site.url }}{{ site.baseurl }}/assets/images/gcp-16.png)

![]({{ site.url }}{{ site.baseurl }}/assets/images/gcp-17.png)

In this lab, we explored the default network along with its subnets, routes, and firewall rules. We deleted the default network and determined that we cannot create any VM instances without a VPC network. Thus, we created a new auto mode VPC network with subnets, routes, firewall rules, and two VM instances. Then we tested the connectivity for the VM instances and explored the effects of the firewall rules on connectivity.