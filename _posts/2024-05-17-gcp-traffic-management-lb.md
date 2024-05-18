---

layout: single
title:  "Configuring Traffic Management with a Load Balancer"
categories: Cloud
tags: GCP
classes: wide
show_date: true
header:
  overlay_image: /assets/images/gcp-banner.png
  og_image: /assets/images/gcp-banner.png
  teaser: /assets/images/ace-gcp.png
author:
  name     : "Associate Cloud Engineer"
  avatar   : "/assets/images/ace-gcp.png"

sidebar:
  - title: "Topics"
    nav: my-sidebar

---
# Configuring Traffic Management with a Load Balancer 

Google Cloud load balancers offer traffic management capabilities that vary by load balancer.

In this lab, you create a regional internal HTTP(S) load balancer  with two backends. Each backend will be an instance group. You will  configure the load balancer to create a blue-green deployment.

The blue deployment refers to the current version of your  application, and the green deployment refers to a new application  version. You configure the load balancer to send 70% of the traffic to  the blue deployment and 30% to the green deployment. When you’re  finished, the environment will look like this:

![The image shows a VPC network with two subnets, each with a managed instance group. One subnet is used for the blue deployment, and the other is used for the green deploynment. Client traffic to the subnets is handled by the load balancer.](https://cdn.qwiklabs.com/FgrZkcSEqghVxKV14KPVgNzTUMo0lQfmqTgpG45%2BTYA%3D) 

- View the Google Cloud infrastructure that the load balancer will use.
- Create a regional internal HTTP(S) load balancer with two backends.
- Implement traffic management on the load balancer.
- Test the load balancer.

## View the Google Cloud infrastructure that the load balancer will use

### **View the network, firewall rules, and Cloud Router**

The network *my-internal-app*, with *subnet-a* and *subnet-b* and firewall rules for RDP, SSH, and ICMP traffic, has been configured  for you. Additional firewall rules have been configured to allow  communication between the load balancer and the backends. Later, you  create a regional internal HTTP(S) load balancer in the my-internal-app  network.

```sh
Welcome to Cloud Shell! Type "help" to get started.
Your Cloud Platform project in this session is set to qwiklabs-gcp-02-4a5e20083f9e.
Use “gcloud config set project [PROJECT_ID]” to change to a different project.
student_03_28bc690c3174@cloudshell:~ (qwiklabs-gcp-02-4a5e20083f9e)$ gcloud compute networks list
NAME: default
SUBNET_MODE: AUTO
BGP_ROUTING_MODE: REGIONAL
IPV4_RANGE: 
GATEWAY_IPV4: 

NAME: my-internal-app
SUBNET_MODE: CUSTOM
BGP_ROUTING_MODE: REGIONAL
IPV4_RANGE: 
GATEWAY_IPV4: 
student_03_28bc690c3174@cloudshell:~ (qwiklabs-gcp-02-4a5e20083f9e)$ gcloud compute networks describe my-internal-app
autoCreateSubnetworks: false
creationTimestamp: '2024-05-17T16:10:49.705-07:00'
id: '452544016838974742'
kind: compute#network
name: my-internal-app
networkFirewallPolicyEnforcementOrder: AFTER_CLASSIC_FIREWALL
routingConfig:
  routingMode: REGIONAL
selfLink: https://www.googleapis.com/compute/v1/projects/qwiklabs-gcp-02-4a5e20083f9e/global/networks/my-internal-app
selfLinkWithId: https://www.googleapis.com/compute/v1/projects/qwiklabs-gcp-02-4a5e20083f9e/global/networks/452544016838974742
subnetworks:
- https://www.googleapis.com/compute/v1/projects/qwiklabs-gcp-02-4a5e20083f9e/regions/us-west1/subnetworks/subnet-a
- https://www.googleapis.com/compute/v1/projects/qwiklabs-gcp-02-4a5e20083f9e/regions/us-west1/subnetworks/subnet-b
x_gcloud_bgp_routing_mode: REGIONAL
x_gcloud_subnet_mode: CUSTOM
student_03_28bc690c3174@cloudshell:~ (qwiklabs-gcp-02-4a5e20083f9e)$ 
student_03_28bc690c3174@cloudshell:~ (qwiklabs-gcp-02-4a5e20083f9e)$ gcloud compute networks subnets list --network my-internal-app
NAME: subnet-a
REGION: us-west1
NETWORK: my-internal-app
RANGE: 10.10.20.0/24
STACK_TYPE: IPV4_ONLY
IPV6_ACCESS_TYPE: 
INTERNAL_IPV6_PREFIX: 
EXTERNAL_IPV6_PREFIX: 

NAME: subnet-b
REGION: us-west1
NETWORK: my-internal-app
RANGE: 10.10.30.0/24
STACK_TYPE: IPV4_ONLY
IPV6_ACCESS_TYPE: 
INTERNAL_IPV6_PREFIX: 
EXTERNAL_IPV6_PREFIX: 
student_03_28bc690c3174@cloudshell:~ (qwiklabs-gcp-02-4a5e20083f9e)$ 
```



```sh
student_03_28bc690c3174@cloudshell:~ (qwiklabs-gcp-02-4a5e20083f9e)$ gcloud compute firewall-rules list 
NAME: app-allow-icmp
NETWORK: my-internal-app
DIRECTION: INGRESS
PRIORITY: 1000
ALLOW: icmp
DENY: 
DISABLED: False

NAME: app-allow-ssh-rdp
NETWORK: my-internal-app
DIRECTION: INGRESS
PRIORITY: 1000
ALLOW: tcp:22,tcp:3389
DENY: 
DISABLED: False

NAME: default-allow-icmp
NETWORK: default
DIRECTION: INGRESS
PRIORITY: 65534
ALLOW: icmp
DENY: 
DISABLED: False

NAME: default-allow-internal
NETWORK: default
DIRECTION: INGRESS
PRIORITY: 65534
ALLOW: tcp:0-65535,udp:0-65535,icmp
DENY: 
DISABLED: False

NAME: default-allow-rdp
NETWORK: default
DIRECTION: INGRESS
PRIORITY: 65534
ALLOW: tcp:3389
DENY: 
DISABLED: False

NAME: default-allow-ssh
NETWORK: default
DIRECTION: INGRESS
PRIORITY: 65534
ALLOW: tcp:22
DENY: 
DISABLED: False

NAME: fw-allow-health-checks
NETWORK: my-internal-app
DIRECTION: INGRESS
PRIORITY: 1000
ALLOW: tcp:80
DENY: 
DISABLED: False

NAME: fw-allow-lb-access
NETWORK: my-internal-app
DIRECTION: INGRESS
PRIORITY: 1000
ALLOW: all
DENY: 
DISABLED: False

To show all fields of the firewall, please show in JSON format: --format=json
To show all fields in table format, please see the examples in --help.

student_03_28bc690c3174@cloudshell:~ (qwiklabs-gcp-02-4a5e20083f9e)$ 
```



In the Google Cloud console, on the **Navigation menu** (![Navigation menu](https://cdn.qwiklabs.com/tkgw1TDgj4Q%2BYKQUW4jUFd0O5OEKlUMBRYbhlCrF0WY%3D)), click **View All Products**. In the left hand pane, select __Networking > **Network Connectivity > Cloud Routers**.
 Note the *nat-router* Cloud Router that was created for the *my-internal-app* network. The load balancer will use this Cloud Router.

```json
{
  "creationTimestamp": "2024-05-17T16:11:02.313-07:00",
  "encryptedInterconnectRouter": false,
  "id": "1581615026885253865",
  "kind": "compute#router",
  "name": "nat-router",
  "nats": [
    {
      "natIpAllocateOption": "AUTO_ONLY",
      "name": "nat-config",
      "udpIdleTimeoutSec": 30,
      "type": "PUBLIC",
      "icmpIdleTimeoutSec": 30,
      "tcpTransitoryIdleTimeoutSec": 30,
      "endpointTypes": [
        "ENDPOINT_TYPE_VM"
      ],
      "tcpEstablishedIdleTimeoutSec": 1200,
      "enableEndpointIndependentMapping": true,
      "autoNetworkTier": "PREMIUM",
      "sourceSubnetworkIpRangesToNat": "ALL_SUBNETWORKS_ALL_IP_RANGES"
    }
  ],
  "network": "projects/qwiklabs-gcp-02-4a5e20083f9e/global/networks/my-internal-app",
  "region": "projects/qwiklabs-gcp-02-4a5e20083f9e/regions/us-west1",
  "selfLink": "projects/qwiklabs-gcp-02-4a5e20083f9e/regions/us-west1/routers/nat-router"
}
```

```sh
student_03_28bc690c3174@cloudshell:~ (qwiklabs-gcp-02-4a5e20083f9e)$ gcloud compute routers list
NAME: nat-router
REGION: us-west1
NETWORK: my-internal-app
student_03_28bc690c3174@cloudshell:~ (qwiklabs-gcp-02-4a5e20083f9e)$
```

```sh
student_03_28bc690c3174@cloudshell:~ (qwiklabs-gcp-02-4a5e20083f9e)$ gcloud compute instances list
NAME: instance-group-2-m1h9
ZONE: us-west1-a
MACHINE_TYPE: e2-medium
PREEMPTIBLE: 
INTERNAL_IP: 10.10.30.2
EXTERNAL_IP: 
STATUS: RUNNING

NAME: instance-group-1-j6fw
ZONE: us-west1-c
MACHINE_TYPE: e2-medium
PREEMPTIBLE: 
INTERNAL_IP: 10.10.20.2
EXTERNAL_IP: 
STATUS: RUNNING
student_03_28bc690c3174@cloudshell:~ (qwiklabs-gcp-02-4a5e20083f9e)$ 
```

### **Create a VM for testing**

You create a VM called *utility-vm* in *subnet-a* of the *my-internal-app* network and use it to test the load balancer.

```sh
gcloud compute instances create utility-vm --project=qwiklabs-gcp-02-4a5e20083f9e --zone=us-west1-c --machine-type=e2-medium --network-interface=private-network-ip=10.10.20.50,stack-type=IPV4_ONLY,subnet=subnet-a,no-address --metadata=enable-oslogin=true --maintenance-policy=MIGRATE --provisioning-model=STANDARD --service-account=608305237403-compute@developer.gserviceaccount.com --scopes=https://www.googleapis.com/auth/devstorage.read_only,https://www.googleapis.com/auth/logging.write,https://www.googleapis.com/auth/monitoring.write,https://www.googleapis.com/auth/servicecontrol,https://www.googleapis.com/auth/service.management.readonly,https://www.googleapis.com/auth/trace.append --create-disk=auto-delete=yes,boot=yes,device-name=utility-vm,image=projects/debian-cloud/global/images/debian-11-bullseye-v20240415,mode=rw,size=10,type=projects/qwiklabs-gcp-02-4a5e20083f9e/zones/us-west1-c/diskTypes/pd-balanced --no-shielded-secure-boot --shielded-vtpm --shielded-integrity-monitoring --labels=goog-ec-src=vm_add-gcloud --reservation-affinity=any
```

```sh
student_03_28bc690c3174@cloudshell:~ (qwiklabs-gcp-02-4a5e20083f9e)$ gcloud compute ssh --zone "us-west1-c" "utility-vm" --tunnel-through-iap --project "qwiklabs-gcp-02-4a5e20083f9e"
WARNING: The private SSH key file for gcloud does not exist.
WARNING: The public SSH key file for gcloud does not exist.
WARNING: You do not have an SSH key for gcloud.
WARNING: SSH keygen will be executed to generate a key.
This tool needs to create the directory [/home/student_03_28bc690c3174/.ssh] before being able to generate SSH keys.

Do you want to continue (Y/n)?  y

Generating public/private rsa key pair.
Enter passphrase (empty for no passphrase): 
Enter same passphrase again: 
Your identification has been saved in /home/student_03_28bc690c3174/.ssh/google_compute_engine
Your public key has been saved in /home/student_03_28bc690c3174/.ssh/google_compute_engine.pub
The key fingerprint is:
SHA256:j4pn4MO3sNibItK/1mxVSOPNMD6VOmMUWJwKgK0ks1I student_03_28bc690c3174@cs-851217029112-default
The key's randomart image is:
+---[RSA 3072]----+
| o..   +oo .     |
|+.E . . O o      |
|o=   . * X       |
|+     . X +      |
|.      .S=       |
|    .   .o       |
| . o.+ .. .      |
|o oo==B.         |
|...+OOo.         |
+----[SHA256]-----+
WARNING: 

To increase the performance of the tunnel, consider installing NumPy. For instructions,
please see https://cloud.google.com/iap/docs/using-tcp-forwarding#increasing_the_tcp_upload_bandwidth

Warning: Permanently added 'compute.7189292875944141560' (ED25519) to the list of known hosts.
Linux utility-vm 5.10.0-28-cloud-amd64 #1 SMP Debian 5.10.209-2 (2024-01-31) x86_64

The programs included with the Debian GNU/Linux system are free software;
the exact distribution terms for each program are described in the
individual files in /usr/share/doc/*/copyright.

Debian GNU/Linux comes with ABSOLUTELY NO WARRANTY, to the extent
permitted by applicable law.
Creating directory '/home/student-03-28bc690c3174'.
student-03-28bc690c3174@utility-vm:~$ curl 10.10.20.2
<h1>Internal Load Balancing Lab</h1><h2>Client IP</h2>Your IP address : 10.10.20.50<h2>Hostname</h2>Server Hostname: instance-group-1-j6fw<h2>Server Location</h2>Region and Zone: us-west1-cstudent-03-28bc690c3174@utility-vm:~$ curl 10.10.30.2
<h1>Internal Load Balancing Lab</h1><h2>Client IP</h2>Your IP address : 10.10.20.50<h2>Hostname</h2>Server Hostname: instance-group-2-m1h9<h2>Server Location</h2>Region and Zone: us-west1-astudent-03-28bc690c3174@utility-vm:~$ 
```



## Configure the load balancer

Configure a regional internal HTTP(S) load balancer to balance traffic between the two backends (*instance-group-1* in  and *instance-group-2* in ), as shown (the region and zones may vary as per the lab requirement):

In the Google Cloud console, in the **Navigation menu** (![Navigation menu](https://cdn.qwiklabs.com/tkgw1TDgj4Q%2BYKQUW4jUFd0O5OEKlUMBRYbhlCrF0WY%3D)), click **View All Products**. In the left hand pane, select  **Networking > Network Services > Load balancing**.

Click **Create load balancer**.

Under **Application Load Balancer (HTTP/HTTPS)**, click **next**.

For **Public facing or internal**, select **internal** and click **next**. This selection creates a regional internal HTTP(S) load balancer. This choice requires the backends to be in a single region .

For **Cross-region or single region deployment**, select **Best for regional workloads** and click **next**.

Click **Configure**.

1. or **Name**, type **my-ilb**
2. For **Region**, select 
3. For **Network**, select **my-internal-app**.

The proxy servers that implement the regional internal HTTP(S) load  balancer require IP addresses. These IP addresses are allocated  automatically from a subnet that you specify.

1. Under **Proxy-only subnet required**, click **Reserve subnet**.
2. For **Name**, type **my-proxy-subnet**
3. For **IP address range**, type **10.10.40.0/24**
4. Click **Add**.
5. Wait for the proxy-only subnet to be created. When that is successful,  the console displays the name of the proxy-only subnet followed by the  IP address range that you specified.



### **Configure the blue-service backend**

This backend service refers to the present ("blue") version of your application.

1. Click **Backend configuration**.

2. For **Backend configuration**, for **Create or select backend service**, select **Create a backend service**.

3. For **Name**, type **blue-service**.

4. In **Backends**, specify the following, and leave the remaining settings as their defaults:

5. Click **Done**.

   For **Health check**, select **Create a health check**.

   Specify the following, and leave the remaining settings as their defaults:

### **Configure the green-service backend**

This backend service refers to the new ("green") version of your application.

1. For **Backend configuration**, for **Create or select backend service**, select **Create a backend service**.
2. For **Name**, type **green-service**.
3. In **Backends**, specify the following, and leave the remaining settings as their defaults:

### **Configure the "blue-green" routing rule**

Create a routing rule that routes 70% of traffic to the blue-service and 30% of traffic to the green service.

1. Click **Routing rules**.
2. In the **Routing rules** panel, for **Mode**, select **Advanded host and path rule**.
3. Click **Add host and path rule**.
4. For **Hosts**, type *****. The * (asterisk) matches all hosts.
5. Traffic management is configured using YAML format. Examine the  following YAML code, and then copy and paste it into line 1 of the  multi-line field **Path matcher (matches, actions, and services)**.

```yaml
defaultService: regions/us-west1/backendServices/blue-service
name: matcher1
routeRules:
 - matchRules:
     - prefixMatch: /
   priority: 0
   routeAction:
     weightedBackendServices:
       - backendService: regions/us-west1/backendServices/blue-service
         weight: 70
       - backendService: regions/us-west1/backendServices/green-service
         weight: 30
```

### **Configure the default routing rule**

When traffic does not match any of the other routing rules, the load  balancer uses the default routing rule. Even though the rule you  configured is designed to match all traffic, the default routing rule is required. You will configure the default routing rule to use the  blue-service backend.

1. Click **(Default) Route traffic to backend "" for any unmatched hosts**.
2. In the **Edit host and path rule** panel, for **Service**, select **blue-service**.

### **Configure the frontend**

The frontend forwards traffic to the backends.

1. Click **Frontend configuration**.

2. Specify the following, and leave the remaining settings as their defaults:

   | Property                    | Value (type value or select option as specified) |
   | --------------------------- | ------------------------------------------------ |
   | Subnetwork                  | subnet-b                                         |
   | IP address                  | Ephemeral (Custom)                               |
   | Custom ephemeral IP address | 10.10.30.5                                       |

3. Click **Done**.

```json
POST https://compute.googleapis.com/compute/beta/projects/qwiklabs-gcp-02-4a5e20083f9e/regions/us-west1/healthChecks
{
  "checkIntervalSec": 10,
  "description": "",
  "healthyThreshold": 2,
  "logConfig": {
    "enable": false
  },
  "name": "blue-health-check",
  "region": "projects/qwiklabs-gcp-02-4a5e20083f9e/regions/us-west1",
  "tcpHealthCheck": {
    "port": 80,
    "proxyHeader": "NONE"
  },
  "timeoutSec": 5,
  "type": "TCP",
  "unhealthyThreshold": 3
}

POST https://compute.googleapis.com/compute/v1/projects/qwiklabs-gcp-02-4a5e20083f9e/zones/us-west1-c/instanceGroups/instance-group-1/setNamedPorts
{
  "namedPorts": [
    {
      "name": "http",
      "port": 80
    }
  ]
}

POST https://compute.googleapis.com/compute/beta/projects/qwiklabs-gcp-02-4a5e20083f9e/regions/us-west1/backendServices
{
  "backends": [
    {
      "balancingMode": "UTILIZATION",
      "capacityScaler": 1,
      "group": "projects/qwiklabs-gcp-02-4a5e20083f9e/zones/us-west1-c/instanceGroups/instance-group-1",
      "maxUtilization": 0.8
    }
  ],
  "connectionDraining": {
    "drainingTimeoutSec": 300
  },
  "description": "",
  "enableCDN": false,
  "healthChecks": [
    "projects/qwiklabs-gcp-02-4a5e20083f9e/regions/us-west1/healthChecks/blue-health-check"
  ],
  "loadBalancingScheme": "INTERNAL_MANAGED",
  "localityLbPolicy": "ROUND_ROBIN",
  "name": "blue-service",
  "portName": "http",
  "protocol": "HTTP",
  "region": "projects/qwiklabs-gcp-02-4a5e20083f9e/regions/us-west1",
  "securityPolicy": "projects/qwiklabs-gcp-02-4a5e20083f9e/regions/us-west1/securityPolicies/default-security-policy-for-backend-service-blue-service",
  "sessionAffinity": "NONE",
  "timeoutSec": 30
}

POST https://compute.googleapis.com/compute/v1/projects/qwiklabs-gcp-02-4a5e20083f9e/regions/us-west1/backendServices/blue-service/setSecurityPolicy
{
  "securityPolicy": "projects/qwiklabs-gcp-02-4a5e20083f9e/regions/us-west1/securityPolicies/default-security-policy-for-backend-service-blue-service"
}

POST https://compute.googleapis.com/compute/beta/projects/qwiklabs-gcp-02-4a5e20083f9e/regions/us-west1/healthChecks
{
  "checkIntervalSec": 10,
  "description": "",
  "healthyThreshold": 2,
  "logConfig": {
    "enable": false
  },
  "name": "green-health-check",
  "region": "projects/qwiklabs-gcp-02-4a5e20083f9e/regions/us-west1",
  "tcpHealthCheck": {
    "port": 80,
    "proxyHeader": "NONE"
  },
  "timeoutSec": 5,
  "type": "TCP",
  "unhealthyThreshold": 3
}

POST https://compute.googleapis.com/compute/v1/projects/qwiklabs-gcp-02-4a5e20083f9e/zones/us-west1-a/instanceGroups/instance-group-2/setNamedPorts
{
  "namedPorts": [
    {
      "name": "http",
      "port": 80
    }
  ]
}

POST https://compute.googleapis.com/compute/beta/projects/qwiklabs-gcp-02-4a5e20083f9e/regions/us-west1/backendServices
{
  "backends": [
    {
      "balancingMode": "UTILIZATION",
      "capacityScaler": 1,
      "group": "projects/qwiklabs-gcp-02-4a5e20083f9e/zones/us-west1-a/instanceGroups/instance-group-2",
      "maxUtilization": 0.8
    }
  ],
  "connectionDraining": {
    "drainingTimeoutSec": 300
  },
  "description": "",
  "enableCDN": false,
  "healthChecks": [
    "projects/qwiklabs-gcp-02-4a5e20083f9e/regions/us-west1/healthChecks/green-health-check"
  ],
  "loadBalancingScheme": "INTERNAL_MANAGED",
  "localityLbPolicy": "ROUND_ROBIN",
  "name": "green-service",
  "portName": "http",
  "protocol": "HTTP",
  "region": "projects/qwiklabs-gcp-02-4a5e20083f9e/regions/us-west1",
  "securityPolicy": "projects/qwiklabs-gcp-02-4a5e20083f9e/regions/us-west1/securityPolicies/default-security-policy-for-backend-service-green-service",
  "sessionAffinity": "NONE",
  "timeoutSec": 30
}

POST https://compute.googleapis.com/compute/v1/projects/qwiklabs-gcp-02-4a5e20083f9e/regions/us-west1/backendServices/green-service/setSecurityPolicy
{
  "securityPolicy": "projects/qwiklabs-gcp-02-4a5e20083f9e/regions/us-west1/securityPolicies/default-security-policy-for-backend-service-green-service"
}

POST https://compute.googleapis.com/compute/v1/projects/qwiklabs-gcp-02-4a5e20083f9e/regions/us-west1/urlMaps
{
  "defaultService": "projects/qwiklabs-gcp-02-4a5e20083f9e/regions/us-west1/backendServices/blue-service",
  "hostRules": [
    {
      "hosts": [
        "*"
      ],
      "pathMatcher": "matcher1"
    }
  ],
  "name": "my-ilb",
  "pathMatchers": [
    {
      "defaultService": "regions/us-west1/backendServices/blue-service",
      "name": "matcher1",
      "routeRules": [
        {
          "matchRules": [
            {
              "prefixMatch": "/"
            }
          ],
          "priority": 0,
          "routeAction": {
            "weightedBackendServices": [
              {
                "backendService": "regions/us-west1/backendServices/blue-service",
                "weight": 70
              },
              {
                "backendService": "regions/us-west1/backendServices/green-service",
                "weight": 30
              }
            ]
          }
        }
      ]
    }
  ],
  "region": "projects/qwiklabs-gcp-02-4a5e20083f9e/regions/us-west1"
}

POST https://compute.googleapis.com/compute/v1/projects/qwiklabs-gcp-02-4a5e20083f9e/regions/us-west1/targetHttpProxies
{
  "name": "my-ilb-target-proxy",
  "region": "projects/qwiklabs-gcp-02-4a5e20083f9e/regions/us-west1",
  "urlMap": "projects/qwiklabs-gcp-02-4a5e20083f9e/regions/us-west1/urlMaps/my-ilb"
}

POST https://compute.googleapis.com/compute/v1/projects/qwiklabs-gcp-02-4a5e20083f9e/regions/us-west1/forwardingRules
{
  "IPAddress": "10.10.30.5",
  "IPProtocol": "TCP",
  "allowGlobalAccess": false,
  "loadBalancingScheme": "INTERNAL_MANAGED",
  "name": "my-ilb-forwarding-rule",
  "networkTier": "PREMIUM",
  "portRange": "80",
  "region": "projects/qwiklabs-gcp-02-4a5e20083f9e/regions/us-west1",
  "subnetwork": "projects/qwiklabs-gcp-02-4a5e20083f9e/regions/us-west1/subnetworks/subnet-b",
  "target": "projects/qwiklabs-gcp-02-4a5e20083f9e/regions/us-west1/targetHttpProxies/my-ilb-target-proxy"
}

POST https://compute.googleapis.com/compute/beta/projects/qwiklabs-gcp-02-4a5e20083f9e/regions/us-west1/securityPolicies
{
  "description": "Default security policy for: blue-service",
  "name": "default-security-policy-for-backend-service-blue-service",
  "region": "us-west1",
  "rules": [
    {
      "action": "allow",
      "match": {
        "config": {
          "srcIpRanges": [
            "*"
          ]
        },
        "versionedExpr": "SRC_IPS_V1"
      },
      "priority": 2147483647
    },
    {
      "action": "throttle",
      "description": "Default rate limiting rule",
      "match": {
        "config": {
          "srcIpRanges": [
            "*"
          ]
        },
        "versionedExpr": "SRC_IPS_V1"
      },
      "priority": 2147483646,
      "rateLimitOptions": {
        "conformAction": "allow",
        "enforceOnKey": "IP",
        "exceedAction": "deny(403)",
        "rateLimitThreshold": {
          "count": 500,
          "intervalSec": 60
        }
      }
    }
  ],
  "selfLink": "projects/qwiklabs-gcp-02-4a5e20083f9e/regions/us-west1/securityPolicies/default-security-policy-for-backend-service-blue-service",
  "type": ""
}

POST https://compute.googleapis.com/compute/beta/projects/qwiklabs-gcp-02-4a5e20083f9e/regions/us-west1/securityPolicies
{
  "description": "Default security policy for: green-service",
  "name": "default-security-policy-for-backend-service-green-service",
  "region": "us-west1",
  "rules": [
    {
      "action": "allow",
      "match": {
        "config": {
          "srcIpRanges": [
            "*"
          ]
        },
        "versionedExpr": "SRC_IPS_V1"
      },
      "priority": 2147483647
    },
    {
      "action": "throttle",
      "description": "Default rate limiting rule",
      "match": {
        "config": {
          "srcIpRanges": [
            "*"
          ]
        },
        "versionedExpr": "SRC_IPS_V1"
      },
      "priority": 2147483646,
      "rateLimitOptions": {
        "conformAction": "allow",
        "enforceOnKey": "IP",
        "exceedAction": "deny(403)",
        "rateLimitThreshold": {
          "count": 500,
          "intervalSec": 60
        }
      }
    }
  ],
  "selfLink": "projects/qwiklabs-gcp-02-4a5e20083f9e/regions/us-west1/securityPolicies/default-security-policy-for-backend-service-green-service",
  "type": ""
}
```

### **Review and create the load balancer**

1. (Optional) Click **Review and finalize**. Review the **Backend** and **Frontend**.
2. Click **Create**.
    Wait for the load balancer to be created before starting the next task.

## Test the load balancer

Verify that the *my-ilb* IP address forwards most of the traffic to the *blue-service* running on *instance-group-1* in .

### **Access the load balancer**

```sh
student-03-28bc690c3174@utility-vm:~$ curl 10.10.30.5
<h1>Internal Load Balancing Lab</h1><h2>Client IP</h2>Your IP address : 10.10.40.3<h2>Hostname</h2>Server Hostname: instance-group-2-m1h9<h2>Server Location</h2>Region and Zone: us-west1-astudent-03-28bc690c3174@utility-vm:~$ curl 10.10.30.5
<h1>Internal Load Balancing Lab</h1><h2>Client IP</h2>Your IP address : 10.10.40.3<h2>Hostname</h2>Server Hostname: instance-group-1-j6fw<h2>Server Location</h2>Region and Zone: us-west1-cstudent-03-28bc690c3174@utility-vm:~$ curl 10.10.30.5
<h1>Internal Load Balancing Lab</h1><h2>Client IP</h2>Your IP address : 10.10.40.4<h2>Hostname</h2>Server Hostname: instance-group-1-j6fw<h2>Server Location</h2>Region and Zone: us-west1-cstudent-03-28bc690c3174@utility-vm:~$ curl 10.10.30.5
<h1>Internal Load Balancing Lab</h1><h2>Client IP</h2>Your IP address : 10.10.40.3<h2>Hostname</h2>Server Hostname: instance-group-2-m1h9<h2>Server Location</h2>Region and Zone: us-west1-astudent-03-28bc690c3174@utility-vm:~$ curl 10.10.30.5
<h1>Internal Load Balancing Lab</h1><h2>Client IP</h2>Your IP address : 10.10.40.2<h2>Hostname</h2>Server Hostname: instance-group-2-m1h9<h2>Server Location</h2>Region and Zone: us-west1-astudent-03-28bc690c3174@utility-vm:~$ curl 10.10.30.5
<h1>Internal Load Balancing Lab</h1><h2>Client IP</h2>Your IP address : 10.10.40.4<h2>Hostname</h2>Server Hostname: instance-group-2-m1h9<h2>Server Location</h2>Region and Zone: us-west1-astudent-03-28bc690c3174@utility-vm:~$ curl 10.10.30.5
<h1>Internal Load Balancing Lab</h1><h2>Client IP</h2>Your IP address : 10.10.40.4<h2>Hostname</h2>Server Hostname: instance-group-1-j6fw<h2>Server Location</h2>Region and Zone: us-west1-cstudent-03-28bc690c3174@utility-vm:~$ curl 10.10.30.5
<h1>Internal Load Balancing Lab</h1><h2>Client IP</h2>Your IP address : 10.10.40.3<h2>Hostname</h2>Server Hostname: instance-group-2-m1h9<h2>Server Location</h2>Region and Zone: us-west1-astudent-03-28bc690c3174@utility-vm:~$ curl 10.10.30.5
<h1>Internal Load Balancing Lab</h1><h2>Client IP</h2>Your IP address : 10.10.40.2<h2>Hostname</h2>Server Hostname: instance-group-1-j6fw<h2>Server Location</h2>Region and Zone: us-west1-cstudent-03-28bc690c3174@utility-vm:~$ curl 10.10.30.5
<h1>Internal Load Balancing Lab</h1><h2>Client IP</h2>Your IP address : 10.10.40.3<h2>Hostname</h2>Server Hostname: instance-group-1-j6fw<h2>Server Location</h2>Region and Zone: us-west1-cstudent-03-28bc690c3174@utility-vm:~$ curl 10.10.30.5
<h1>Internal Load Balancing Lab</h1><h2>Client IP</h2>Your IP address : 10.10.40.2<h2>Hostname</h2>Server Hostname: instance-group-1-j6fw<h2>Server Location</h2>Region and Zone: us-west1-cstudent-03-28bc690c3174@utility-vm:~$ curl 10.10.30.5
<h1>Internal Load Balancing Lab</h1><h2>Client IP</h2>Your IP address : 10.10.40.2<h2>Hostname</h2>Server Hostname: instance-group-1-j6fw<h2>Server Location</h2>Region and Zone: us-west1-cstudent-03-28bc690c3174@utility-vm:~$ curl 10.10.30.5
<h1>Internal Load Balancing Lab</h1><h2>Client IP</h2>Your IP address : 10.10.40.3<h2>Hostname</h2>Server Hostname: instance-group-1-j6fw<h2>Server Location</h2>Region and Zone: us-west1-cstudent-03-28bc690c3174@utility-vm:~$ curl 10.10.30.5
<h1>Internal Load Balancing Lab</h1><h2>Client IP</h2>Your IP address : 10.10.40.4<h2>Hostname</h2>Server Hostname: instance-group-2-m1h9<h2>Server Location</h2>Region and Zone: us-west1-astudent-03-28bc690c3174@utility-vm:~$ curl 10.10.30.5
<h1>Internal Load Balancing Lab</h1><h2>Client IP</h2>Your IP address : 10.10.40.3<h2>Hostname</h2>Server Hostname: instance-group-2-m1h9<h2>Server Location</h2>Region and Zone: us-west1-astudent-03-28bc690c3174@utility-vm:~$ curl 10.10.30.5
<h1>Internal Load Balancing Lab</h1><h2>Client IP</h2>Your IP address : 10.10.40.2<h2>Hostname</h2>Server Hostname: instance-group-2-m1h9<h2>Server Location</h2>Region and Zone: us-west1-astudent-03-28bc690c3174@utility-vm:~$ curl 10.10.30.5
<h1>Internal Load Balancing Lab</h1><h2>Client IP</h2>Your IP address : 10.10.40.3<h2>Hostname</h2>Server Hostname: instance-group-1-j6fw<h2>Server Location</h2>Region and Zone: us-west1-cstudent-03-28bc690c3174@utility-vm:~$ 
```



## Review

In this lab, you created two managed instance groups in the  region. You also created some firewall rules. The firewall rules allow  traffic from clients and the health checkers to the managed instance  groups. You configured a regional internal HTTP(S) load balancer, using  the managed instance groups as backends. Finally, you tested the load  balancer to ensure that it works as expected.