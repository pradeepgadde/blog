---

layout: single
title:  "Configuring Network Services"
date:   2023-03-25 06:59:04 +0530
categories: Cloud
tags: GCP
toc: true
toc_sticky: true
show_date: true
classes: wide
header:
  overlay_image: /assets/images/gpcne.png
  og_image: /assets/images/gpcne.png
  teaser: /assets/images/gpcne.png
author:
  name     : "Cloud Network Engineer"
  avatar   : "/assets/images/gpcne.png"

sidebar:
  - title: "Topics"
    nav: my-sidebar

---

# Configuring Network Services

## Configuring load balancing
There are many types of load balancer available in Google Cloud. Some operate at layer 4 (L4) and others at layer 7 (L7). For HTTP and HTTPS traffic the L7 internal or external regional or global HTTP(S) load balancer is typically the best choice.
When supporting both HTTP and HTTPS for the same domains, the recommended approach is to use 2 separate target proxies. Each target proxy requires a forwarding rule. One of the target proxies is used to redirect HTTP requests to HTTPS and requires no backend bucket or service. Each target proxy would also have a URL map that it uses to either redirect traffic, or direct traffic to backend services or backend buckets based on the domain or path of the request. This is referred to as content-based load balancing.
For global load balancers, each backend bucket or backend service can operate over many regions and will intelligently route to the nearest region to where the user traffic originates. Backend buckets are connected to Cloud Storage buckets. Backend buckets are the most effective way to serve static resources because those resources can be served directly from the GCS bucket without requiring any compute resources. Static resources can also easily be populated into Cloud CDN to be served with low latency worldwide.

Managed instance groups and network endpoint groups can both autoscale. They create or delete replicas to serve traffic automatically, based on resource target values that are measured by metrics collected by Google Cloud Monitoring. Managed instance groups are always groups of identical VMs. Network endpoint groups can be more general, such as FQDN/port or IP/port combinations, or managed services. In GKE you can use either managed instance groups or network endpoint groups, but container-native load balancing only uses network endpoint groups.

In GKE workloads, autoscaling is typically accomplished using a HorizontalPodAutoscaler, though VerticalPodAutoscaler and MultidimPodAutoscaler are possible alternatives. GKE cluster autoscaling occurs based on the resource demands and scheduling of pods across all workloads using a given node pool.

## Configuring Google Cloud Armor policies

Cloud Armor can provide WAF capabilities via an expression language that allows logical combinations of various clauses in allow or deny rules. Allow rules allow matching traffic and block all other traffic, while deny rules block matching traffic and allow all other traffic. Allow and deny rules can be combined in a priority order.

Clauses can incorporate attack patterns or attributes of the request, such as source IP address, headers being present, or their values.

## Configuring Cloud CDN

Cloud CDN has a simple configuration model that utilizes a cache mode parameter.

The parameter can be one of three values.

1. CACHE_ALL_STATIC will cache static content based on Content-Type/MIME matching standard static types such as Javascript, CSS, photos, video, and audio, unless Cache-Control metadata for the associated object in Cloud Storage has a private or no-store directive. When the mode is CACHE_ALL_STATIC, a configuration parameter called Default TTL, which defaults to 1h, sets the lifetime for static file types that don’t have Cache-Control specified expiry time. If they do have Cache-Control specified expiry time, they use the specified value. Other file types that are not in the set identified as static would be cached or not based on Cache-Control metadata normally.

2. FORCE_CACHE_ALL will enforce caching of all objects regardless of Cache-Control metadata and will also use the Default TTL value for expiry time.

3. USE_ORIGIN_HEADERS strictly uses the Cache-Control headers for controlling the caching. In addition to the Default TTL parameter there are also Max TTL and Client TTL configuration parameters that can adjust the behavior in the CACHE_ALL_STATIC and FORCE_CACHE_ALL modes.

Signed URLs and Cookies provide a mechanism to give controlled access to objects that are stored in Cloud Storage and cached in Cloud CDN.



## Configuring and maintaining Cloud DNS

Cloud DNS allows for creation of managed zones that model DNS zones. These zones can contain DNS record sets for authoritative DNS serving for a specified public or private domain.

Cloud DNS also supports many other capabilities including DNS forwarding and peering, but requires valid configuration to support these DNS scenarios. Separate managed zones must be created for different domains, public vs private DNS, DNS peering, or DNS forwarding.

Cloud DNS also allows for DNS server policies to be created for VPCs that allow inbound or outbound forwarding of DNS requests. For outbound DNS forwarding, both outbound DNS server policies and forwarding zones can be used. When forwarding DNS requests to on-premises DNS servers, forwarding zones are the recommended approach.



DNS Peering is recommended to avoid outbound DNS forwarding from multiple VPCs which can cause problems with return traffic. DNS peering allows a single forwarding zone to be associated with a single VPC and then other VPCs to have their requests forwarded by DNS peering with the forwarding zone.



## Configuring Cloud NAT

Cloud NAT allows outbound internet requests for VMs without external IP addresses and will allow responses to those outbound requests to be forwarded back to the initiating VM. Cloud NAT can either automatically allocate the set of external IP addresses it uses, or they can be manually specified. Each NAT IP address can support 64512 TCP and 64512 UDP connections. The connections can be distributed among all the VMs using that NAT gateway based on the minimum ports per VM configuration. This minimum ports per VM value can be either statically or dynamically configured, but will create a limit on how many VMs can be served by each NAT IP address.



## Configuring network packet inspection

VMs with multiple NICs can be used to connect multiple VPCs. Each NIC in a VM with multiple NICs must be in a separate VPC and each VPC can have its own independent set of custom routes (in addition to the default routes) and custom routes can have next hops on a VM with multiple NICs that act as a gateway between the VPCs. This pattern can be useful in scenarios where VPCs with overlapping subnet ranges need to be connected for private IP communication, or in cases where traffic needs to be scanned between trusted and untrusted networks without using public IP addresses.



Each packet mirroring policy sends traffic from a collection of source VMs, which must all be in the same project, VPC, and region, to a destination internal TCP/UDP load balancer configured for packet mirroring. The source VMs can be specified in the packet mirroring policy by name, tag, or subnet, with different limitations depending on the approach taken. The destination load balancer can be connected to one or more

VMs in an unmanaged or managed instance group. It must also be in the same region, but can be in either the same VPC or a peered VPC. Packet mirroring policies can be configured to forward only ingress or only egress traffic, or to limit the traffic to only include certain source or destination IP ranges and/or protocols
