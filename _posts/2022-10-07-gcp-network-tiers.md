---

layout: single
title:  "GCP—Optimizing network spend with Network Tiers"
date:   2022-10-07 04:59:04 +0530
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
  - title: "Topics"
    nav: my-sidebar

---

# Network Tiers

In this lab, you create one VM in the Premium network service tier (default) and one VM in the Standard network service tier. Then you compare the latency and network paths for each VM instance.

With Network Service Tiers, Google Cloud enables you to optimize your cloud network for performance by choosing the Premium Tier, or for cost with the new Standard Tier.

- Premium Tier
Premium Tier delivers traffic over Google’s well-provisioned, low-latency, highly reliable global network. This network consists of an extensive global private fiber network with over 100 points of presence (POPs) across the globe.

- Standard Tier
Standard Tier is a new, lower-cost offering. The network quality of this tier is comparable to the quality of other public cloud providers and regional network services, such as regional load balancing with one VIP per region, but lower than the quality of Premium Tier.

Standard Tier is priced lower than Premium because the traffic between Google Cloud and your end user (internet) is delivered over transit (ISP) networks instead of Google’s network.

## Task 1. Create the VM instances
In this task, you create two VM instances and define their network service tier during the instance creation. You can configure the network tier for your VM instances at the project level or at the resource level.

Create the Premium Tier VM
Create a VM instance using the Premium service tier, which is the default.

![]({{ site.url }}{{ site.baseurl }}/assets/images/gcp-100.png)

```sh
gcloud compute instances create vm-premium --project=qwiklabs-gcp-02-fcdded3b3476 --zone=us-central1-c --machine-type=n1-standard-1 --network-interface=network-tier=PREMIUM,subnet=default --metadata=enable-oslogin=true --maintenance-policy=MIGRATE --provisioning-model=STANDARD --service-account=457883812419-compute@developer.gserviceaccount.com --scopes=https://www.googleapis.com/auth/devstorage.read_only,https://www.googleapis.com/auth/logging.write,https://www.googleapis.com/auth/monitoring.write,https://www.googleapis.com/auth/servicecontrol,https://www.googleapis.com/auth/service.management.readonly,https://www.googleapis.com/auth/trace.append --create-disk=auto-delete=yes,boot=yes,device-name=vm-premium,image=projects/debian-cloud/global/images/debian-11-bullseye-v20220920,mode=rw,size=10,type=projects/qwiklabs-gcp-02-fcdded3b3476/zones/us-central1-c/diskTypes/pd-balanced --no-shielded-secure-boot --shielded-vtpm --shielded-integrity-monitoring --reservation-affinity=any
```

Create the Standard Tier VM
Create a VM instance of the same machine type and in the same zone but use the Standard service tier.

![]({{ site.url }}{{ site.baseurl }}/assets/images/gcp-101.png)

```sh
gcloud compute instances create vm-standard --project=qwiklabs-gcp-02-fcdded3b3476 --zone=us-central1-c --machine-type=n1-standard-1 --network-interface=network-tier=STANDARD,subnet=default --metadata=enable-oslogin=true --maintenance-policy=MIGRATE --provisioning-model=STANDARD --service-account=457883812419-compute@developer.gserviceaccount.com --scopes=https://www.googleapis.com/auth/devstorage.read_only,https://www.googleapis.com/auth/logging.write,https://www.googleapis.com/auth/monitoring.write,https://www.googleapis.com/auth/servicecontrol,https://www.googleapis.com/auth/service.management.readonly,https://www.googleapis.com/auth/trace.append --create-disk=auto-delete=yes,boot=yes,device-name=vm-standard,image=projects/debian-cloud/global/images/debian-11-bullseye-v20220920,mode=rw,size=10,type=projects/qwiklabs-gcp-02-fcdded3b3476/zones/us-central1-c/diskTypes/pd-balanced --no-shielded-secure-boot --shielded-vtpm --shielded-integrity-monitoring --reservation-affinity=any
```

Both VM instances have the same machine type, zone, and VPC network. The only differences are the Network Service Tier and the instance names.



![]({{ site.url }}{{ site.baseurl }}/assets/images/gcp-102.png)



## Task 2. Explore the latency and network paths
In this task you explore some of the network performance differences between the Premium Tier and Standard Tier.

Explore the latency for each VM instance
In this step you explore the latency from a third-party service in Europe to your VM instances in us-central1-c. Latency is defined as the round-trip time (RTT), or the time network packets take to get from one host to the other and back. Lower latency improves user experience and also improves transfer speeds.

In this step, you use the website `ping.eu` to demonstrate the latency a user in Europe might experience when accessing your server in us-central1-c.

```sh
--- PING 104.154.31.108 (104.154.31.108) 56(84) bytes of data. ---
64 bytes from 104.154.31.108: icmp_seq=1 ttl=60 time=115 ms
64 bytes from 104.154.31.108: icmp_seq=2 ttl=60 time=114 ms
64 bytes from 104.154.31.108: icmp_seq=3 ttl=60 time=114 ms
64 bytes from 104.154.31.108: icmp_seq=4 ttl=60 time=114 ms


--- 104.154.31.108 ping statistics ---

					
packets transmitted 	4
received 	4
packet loss 	0 %
time 	3011 ms


					
--- Round Trip Time (rtt) ---

					
min 	114.128 ms
avg 	114.411 ms
max 	115.103 ms
mdev 	0.400 ms
```

```sh
--- PING 35.208.0.178 (35.208.0.178) 56(84) bytes of data. ---
64 bytes from 35.208.0.178: icmp_seq=1 ttl=56 time=123 ms
64 bytes from 35.208.0.178: icmp_seq=2 ttl=56 time=122 ms
64 bytes from 35.208.0.178: icmp_seq=3 ttl=56 time=122 ms
64 bytes from 35.208.0.178: icmp_seq=4 ttl=56 time=122 ms


--- 35.208.0.178 ping statistics ---

					
packets transmitted 	4
received 	4
packet loss 	0 %
time 	3011 ms


					
--- Round Trip Time (rtt) ---

					
min 	122.313 ms
avg 	122.542 ms
max 	123.077 ms
mdev 	0.310 ms
```

Premium Tier has an `avg 	114.411 ms` while the Standard Tier has  `avg 	122.542 ms`.

Explore the network path for each VM instance
Explore the network path between a third-party service in Europe and your VM instances in us-central1-c using traceroute. Traceroute shows all Layer 3 (routing layer) hops between hosts; therefore, it can illustrate a network path between hosts.

>Premium


```sh
traceroute to 104.154.31.108 (104.154.31.108), 30 hops max, 60 byte packets

			
1	 	 	 	*	*	*

			
2	core23.fsn1.hetzner.com	213.239.245.237	de	0.387 ms	 	 
core24.fsn1.hetzner.com	213.239.245.241	de	0.526 ms	0.310 ms

			
3	core4.fra.hetzner.com	213.239.229.73	de	4.893 ms	 	 
core0.fra.hetzner.com	213.239.252.41	de	7.202 ms	 
core0.fra.hetzner.com	213.239.252.37	de	4.881 ms

			
4	 	 	 	*	*	*

			
5	 	 	 	*	*	*

			
6	 	 	 	*	*	*

			
7	 	 	 	*	*	*

			
8	 	 	 	*	*	*
No reply for 5 hops. Assuming we reached firewall.
```

>Standard



```sh
traceroute to 35.208.0.178 (35.208.0.178), 30 hops max, 60 byte packets

			
1	 	 	 	*	*	*

			
2	core24.fsn1.hetzner.com	213.239.245.241	de	0.299 ms	 	 
core23.fsn1.hetzner.com	213.239.245.237	de	0.234 ms	 
core24.fsn1.hetzner.com	213.239.245.241	de	0.299 ms

			
3	juniper6.dc2.nbg1.hetzner.com	213.239.245.53	de	2.567 ms	 	 
juniper5.nbg1.hetzner.com	213.239.252.249	de	2.596 ms	2.508 ms

			
4	nug-b2-link.ip.twelve99.net	62.115.50.222	se	5.260 ms	 	 
nug-b1-link.ip.twelve99.net	213.248.70.0	se	2.811 ms	2.830 ms

			
5	hbg-bb4-link.ip.twelve99.net	62.115.113.174	se	19.903 ms	19.908 ms	 
prs-bb2-link.ip.twelve99.net	62.115.112.214	se	16.728 ms

			
6	ldn-bb4-link.ip.twelve99.net	62.115.122.161	se	29.903 ms	 	 
ldn-bb4-link.ip.twelve99.net	62.115.133.238	se	29.878 ms	 
ldn-bb4-link.ip.twelve99.net	62.115.122.161	se	29.903 ms

			
7	nyk-bb1-link.ip.twelve99.net	62.115.112.244	se	99.421 ms	99.314 ms	96.611 ms

			
8	chi-b23-link.ip.twelve99.net	80.91.246.162	se	113.039 ms	112.905 ms	115.656 ms

			
9	google-ic326615-chi-b23.ip.twelve99-cust.net	80.239.128.33	de	107.592 ms	107.436 ms	107.606 ms

			
10	 	 	 	*	*	*

			
11	 	 	 	*	*	*

			
12	 	 	 	*	*	*

			
13	 	 	 	*	*	*

			
14	 	 	 	*	*	*
No reply for 5 hops. Assuming we reached firewall.
```

In the example output, the traffic destined for the Premium Tier VM reached Google Cloud's network after the 3rd hop. Therefore, the Premium network tier traffic entered the Google Cloud network very close to the user, as expected.

The traffic destined for the Standard Tier VM reached Google Cloud's network after the 9th hop. 
This demonstrates that Premium network tier traffic enters the Google Cloud network much closer to the user than Standard network tier traffic does.

## Task 3. Review
In this lab, we created one VM in the Premium network service tier (default) and one VM in the Standard network service tier. Then, we used a third-party service to visualize the differences in latency and network paths for each VM instance. The Premium Tier VM had a lower latency and its traffic entered Google Cloud's network sooner than the Standard Tier VM's traffic.

With Network Service Tiers, Google Cloud offers the flexibility to configure our resources for performance or cost by introducing the Standard Tier of networking. 

