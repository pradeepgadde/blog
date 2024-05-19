---

layout: single
title:  "CLI-HTTP Load Balancer with Cloud Armor"
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
# CLI-HTTP Load Balancer with Cloud Armor
## Configure HTTP and health check firewall rules

Configure firewall rules to allow HTTP traffic to the backends and TCP traffic from the Google Cloud health checker.

```sh
Welcome to Cloud Shell! Type "help" to get started.
Your Cloud Platform project in this session is set to qwiklabs-gcp-04-7195e922f04d.
Use “gcloud config set project [PROJECT_ID]” to change to a different project.
gcloud compute --project=qwiklabs-gcp-04-7195e922f04d firewall-rules create default-allow-http --direction=INGRESS --priority=1000 --network=default --action=ALLOW --rules=tcp:80 -student_01_aa8f3a74b714@cloudshell:~ (qwiklabs-gcp-04-7195e922f04d)$ gcloud compute --project=qwiklabs-gcp-04-7195e922f04d firewall-rules create default-allow-http --direction=INGRESS --priority=1000 --network=default --action=ALLOW --rules=tcp:80 --source-ranges=0.0.0.0/0 --target-tags=http-server
Creating firewall...working..Created [https://www.googleapis.com/compute/v1/projects/qwiklabs-gcp-04-7195e922f04d/global/firewalls/default-allow-http].                            
Creating firewall...done.                                                                                                                                                          
NAME: default-allow-http
NETWORK: default
DIRECTION: INGRESS
PRIORITY: 1000
ALLOW: tcp:80
DENY: 
DISABLED: False
student_01_aa8f3a74b714@cloudshell:~ (qwiklabs-gcp-04-7195e922f04d)$ gcloud compute --project=qwiklabs-gcp-04-7195e922f04d firewall-rules create default-allow-health-check --direction=INGRESS --priority=1000 --network=default --action=ALLOW --rules=tcp --source-ranges=130.211.0.0/22,35.191.0.0/16 --target-tags=http-server
Creating firewall...working..Created [https://www.googleapis.com/compute/v1/projects/qwiklabs-gcp-04-7195e922f04d/global/firewalls/default-allow-health-check].                    
Creating firewall...done.                                                                                                                                                          
NAME: default-allow-health-check
NETWORK: default
DIRECTION: INGRESS
PRIORITY: 1000
ALLOW: tcp
DENY: 
DISABLED: False
student_01_aa8f3a74b714@cloudshell:~ (qwiklabs-gcp-04-7195e922f04d)$ 
```

## Configure instance templates and create instance groups

A managed instance group uses an instance template to create a group  of identical instances. Use these to create the backends of the HTTP  Load Balancer.



### Configure the instance templates

An instance template is an API resource that you use to create VM instances and managed instance groups. Instance templates define the machine type, boot disk image, subnet, labels, and other instance properties.

```sh
student_01_aa8f3a74b714@cloudshell:~ (qwiklabs-gcp-04-7195e922f04d)$ gcloud compute instance-templates create us-central1-template --project=qwiklabs-gcp-04-7195e922f04d --machine-type=e2-micro --network-interface=network-tier=PREMIUM,subnet=default --metadata=startup-script-url=gs://cloud-training/gcpnet/httplb/startup.sh,enable-oslogin=true --maintenance-policy=MIGRATE --provisioning-model=STANDARD --service-account=271285976134-compute@developer.gserviceaccount.com --scopes=https://www.googleapis.com/auth/devstorage.read_only,https://www.googleapis.com/auth/logging.write,https://www.googleapis.com/auth/monitoring.write,https://www.googleapis.com/auth/servicecontrol,https://www.googleapis.com/auth/service.management.readonly,https://www.googleapis.com/auth/trace.append --region=us-central1 --tags=http-server --create-disk=auto-delete=yes,boot=yes,device-name=us-central1-template,image=projects/debian-cloud/global/images/debian-12-bookworm-v20240415,mode=rw,size=10,type=pd-balanced --no-shielded-secure-boot --shielded-vtpm --shielded-integrity-monitoring --reservation-affinity=any
Created [https://www.googleapis.com/compute/v1/projects/qwiklabs-gcp-04-7195e922f04d/global/instanceTemplates/us-central1-template].
NAME: us-central1-template
MACHINE_TYPE: e2-micro
PREEMPTIBLE: 
CREATION_TIMESTAMP: 2024-05-18T21:01:08.749-07:00
student_01_aa8f3a74b714@cloudshell:~ (qwiklabs-gcp-04-7195e922f04d)$ 
```

```sh
student_01_aa8f3a74b714@cloudshell:~ (qwiklabs-gcp-04-7195e922f04d)$ gcloud compute instance-templates create us-west1-template --project=qwiklabs-gcp-04-7195e922f04d --machine-type=e2-micro --network-interface=network-tier=PREMIUM,subnet=default --metadata=startup-script-url=gs://cloud-training/gcpnet/httplb/startup.sh,enable-oslogin=true --maintenance-policy=MIGRATE --provisioning-model=STANDARD --service-account=271285976134-compute@developer.gserviceaccount.com --scopes=https://www.googleapis.com/auth/devstorage.read_only,https://www.googleapis.com/auth/logging.write,https://www.googleapis.com/auth/monitoring.write,https://www.googleapis.com/auth/servicecontrol,https://www.googleapis.com/auth/service.management.readonly,https://www.googleapis.com/auth/trace.append --region=us-west1 --tags=http-server --create-disk=auto-delete=yes,boot=yes,device-name=us-west1-template,image=projects/debian-cloud/global/images/debian-12-bookworm-v20240415,mode=rw,size=10,type=pd-balanced --no-shielded-secure-boot --shielded-vtpm --shielded-integrity-monitoring --reservation-affinity=any
Created [https://www.googleapis.com/compute/v1/projects/qwiklabs-gcp-04-7195e922f04d/global/instanceTemplates/us-west1-template].
NAME: us-west1-template
MACHINE_TYPE: e2-micro
PREEMPTIBLE: 
CREATION_TIMESTAMP: 2024-05-18T21:03:10.502-07:00
student_01_aa8f3a74b714@cloudshell:~ (qwiklabs-gcp-04-7195e922f04d)$ 
```

### Create the managed instance groups

Create a managed instance group in  and one in .

1. Still in **Compute Engine**, click **Instance groups** in the left menu.
2. Click **Create instance group**.

```sh
student_01_aa8f3a74b714@cloudshell:~ (qwiklabs-gcp-04-7195e922f04d)$ gcloud beta compute instance-groups managed create us-central1-mig --project=qwiklabs-gcp-04-7195e922f04d --base-instance-name=us-central1-mig --template=projects/qwiklabs-gcp-04-7195e922f04d/global/instanceTemplates/us-central1-template --size=1 --zones=us-central1-c,us-central1-f,us-central1-b --target-distribution-shape=EVEN --instance-redistribution-type=PROACTIVE --default-action-on-vm-failure=repair --no-force-update-on-repair --standby-policy-mode=manual --list-managed-instances-results=PAGELESS && gcloud beta compute instance-groups managed set-autoscaling us-central1-mig --project=qwiklabs-gcp-04-7195e922f04d --region=us-central1 --mode=on --min-num-replicas=1 --max-num-replicas=2 --target-cpu-utilization=0.8 --cool-down-period=45
Created [https://www.googleapis.com/compute/beta/projects/qwiklabs-gcp-04-7195e922f04d/regions/us-central1/instanceGroupManagers/us-central1-mig].
NAME: us-central1-mig
LOCATION: us-central1
SCOPE: region
BASE_INSTANCE_NAME: us-central1-mig
SIZE: 0
TARGET_SIZE: 1
INSTANCE_TEMPLATE: us-central1-template
AUTOSCALED: no
Created [https://www.googleapis.com/compute/beta/projects/qwiklabs-gcp-04-7195e922f04d/regions/us-central1/autoscalers/us-central1-mig-kv7f].
---
autoscalingPolicy:
  coolDownPeriodSec: 45
  cpuUtilization:
    utilizationTarget: 0.8
  maxNumReplicas: 2
  minNumReplicas: 1
  mode: ON
creationTimestamp: '2024-05-18T21:06:41.765-07:00'
id: '7816346165445968958'
kind: compute#autoscaler
name: us-central1-mig-kv7f
region: https://www.googleapis.com/compute/beta/projects/qwiklabs-gcp-04-7195e922f04d/regions/us-central1
selfLink: https://www.googleapis.com/compute/beta/projects/qwiklabs-gcp-04-7195e922f04d/regions/us-central1/autoscalers/us-central1-mig-kv7f
status: ACTIVE
target: https://www.googleapis.com/compute/beta/projects/qwiklabs-gcp-04-7195e922f04d/regions/us-central1/instanceGroupManagers/us-central1-mig
student_01_aa8f3a74b714@cloudshell:~ (qwiklabs-gcp-04-7195e922f04d)$ 
```

Managed instance groups offer **autoscaling** capabilities that  allow you to automatically add or remove instances from a managed  instance group based on increases or decreases in load. Autoscaling  helps your applications gracefully handle increases in traffic and  reduces cost when the need for resources is lower. You just define the  autoscaling policy and the autoscaler performs automatic scaling based  on the measured load.

1. Click **Create**.

Now repeat the same procedure to create a second instance group for **us-west1-mig** in :

```sh
student_01_aa8f3a74b714@cloudshell:~ (qwiklabs-gcp-04-7195e922f04d)$ gcloud beta compute instance-groups managed create us-west1-mig --project=qwiklabs-gcp-04-7195e922f04d --base-instance-name=us-west1-mig --template=projects/qwiklabs-gcp-04-7195e922f04d/global/instanceTemplates/us-west1-template --size=1 --zones=us-west1-b,us-west1-c,us-west1-a --target-distribution-shape=EVEN --instance-redistribution-type=PROACTIVE --default-action-on-vm-failure=repair --no-force-update-on-repair --standby-policy-mode=manual --list-managed-instances-results=PAGELESS && gcloud beta compute instance-groups managed set-autoscaling us-west1-mig --project=qwiklabs-gcp-04-7195e922f04d --region=us-west1 --mode=on --min-num-replicas=1 --max-num-replicas=2 --target-cpu-utilization=0.8 --cool-down-period=45
Created [https://www.googleapis.com/compute/beta/projects/qwiklabs-gcp-04-7195e922f04d/regions/us-west1/instanceGroupManagers/us-west1-mig].
NAME: us-west1-mig
LOCATION: us-west1
SCOPE: region
BASE_INSTANCE_NAME: us-west1-mig
SIZE: 0
TARGET_SIZE: 1
INSTANCE_TEMPLATE: us-west1-template
AUTOSCALED: no
Created [https://www.googleapis.com/compute/beta/projects/qwiklabs-gcp-04-7195e922f04d/regions/us-west1/autoscalers/us-west1-mig-codu].
---
autoscalingPolicy:
  coolDownPeriodSec: 45
  cpuUtilization:
    utilizationTarget: 0.8
  maxNumReplicas: 2
  minNumReplicas: 1
  mode: ON
creationTimestamp: '2024-05-18T21:09:26.798-07:00'
id: '2275574041318573465'
kind: compute#autoscaler
name: us-west1-mig-codu
region: https://www.googleapis.com/compute/beta/projects/qwiklabs-gcp-04-7195e922f04d/regions/us-west1
selfLink: https://www.googleapis.com/compute/beta/projects/qwiklabs-gcp-04-7195e922f04d/regions/us-west1/autoscalers/us-west1-mig-codu
status: ACTIVE
target: https://www.googleapis.com/compute/beta/projects/qwiklabs-gcp-04-7195e922f04d/regions/us-west1/instanceGroupManagers/us-west1-mig
student_01_aa8f3a74b714@cloudshell:~ (qwiklabs-gcp-04-7195e922f04d)$ 
```

### Verify the backends

Verify that VM instances are being created in both regions and access their HTTP sites.

```sh
student_01_aa8f3a74b714@cloudshell:~ (qwiklabs-gcp-04-7195e922f04d)$ gcloud compute instances list
NAME: us-central1-mig-vb0q
ZONE: us-central1-c
MACHINE_TYPE: e2-micro
PREEMPTIBLE: 
INTERNAL_IP: 10.128.0.2
EXTERNAL_IP: 34.41.16.58
STATUS: RUNNING

NAME: us-west1-mig-r4bh
ZONE: us-west1-a
MACHINE_TYPE: e2-micro
PREEMPTIBLE: 
INTERNAL_IP: 10.138.0.2
EXTERNAL_IP: 34.105.61.159
STATUS: RUNNING
student_01_aa8f3a74b714@cloudshell:~ (qwiklabs-gcp-04-7195e922f04d)$ 
```

```sh
student_01_aa8f3a74b714@cloudshell:~ (qwiklabs-gcp-04-7195e922f04d)$ curl 34.41.16.58
<h1>HTTP Load Balancing Lab</h1><h2>Client IP</h2>Your IP address : 34.87.100.91<h2>Hostname</h2>Server Hostname: us-central1-mig-vb0q<h2>Server Location</h2>Region and Zone: us-central1-cstudent_01_aa8f3a74b714@cloudshell:~ (qwiklabs-gcp-04-7195e922f04d)$ curl 34.105.61.159
<h1>HTTP Load Balancing Lab</h1><h2>Client IP</h2>Your IP address : 34.87.100.91<h2>Hostname</h2>Server Hostname: us-west1-mig-r4bh<h2>Server Location</h2>Region and Zone: us-west1-astudent_01_aa8f3a74b714@cloudshell:~ (qwiklabs-gcp-04-7195e922f04d)$ 
```

## Configure the HTTP Load Balancer

Configure the HTTP Load Balancer to balance traffic between the two backends

HTTP(S) load balancing supports both IPv4 and IPv6 addresses for client  traffic. Client IPv6 requests are terminated at the global load  balancing layer, then proxied over IPv4 to your backends.

Backend services direct incoming traffic to one or more attached  backends. Each backend is composed of an instance group and additional  serving capacity metadata.

```json
POST https://compute.googleapis.com/compute/beta/projects/qwiklabs-gcp-04-7195e922f04d/global/healthChecks
{
  "checkIntervalSec": 5,
  "description": "",
  "healthyThreshold": 2,
  "logConfig": {
    "enable": false
  },
  "name": "http-health-check",
  "tcpHealthCheck": {
    "port": 80,
    "proxyHeader": "NONE"
  },
  "timeoutSec": 5,
  "type": "TCP",
  "unhealthyThreshold": 2
}

POST https://compute.googleapis.com/compute/v1/projects/qwiklabs-gcp-04-7195e922f04d/global/securityPolicies
{
  "description": "Default security policy for: http-backend",
  "name": "default-security-policy-for-backend-service-http-backend",
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
  ]
}

POST https://compute.googleapis.com/compute/beta/projects/qwiklabs-gcp-04-7195e922f04d/global/backendServices
{
  "backends": [
    {
      "balancingMode": "RATE",
      "capacityScaler": 1,
      "group": "projects/qwiklabs-gcp-04-7195e922f04d/regions/us-central1/instanceGroups/us-central1-mig",
      "maxRatePerInstance": 50
    },
    {
      "balancingMode": "UTILIZATION",
      "capacityScaler": 1,
      "group": "projects/qwiklabs-gcp-04-7195e922f04d/regions/us-west1/instanceGroups/us-west1-mig",
      "maxRatePerInstance": 50,
      "maxUtilization": 0.8
    }
  ],
  "cdnPolicy": {
    "cacheKeyPolicy": {
      "includeHost": true,
      "includeProtocol": true,
      "includeQueryString": true
    },
    "cacheMode": "CACHE_ALL_STATIC",
    "clientTtl": 3600,
    "defaultTtl": 3600,
    "maxTtl": 86400,
    "negativeCaching": false,
    "serveWhileStale": 0
  },
  "compressionMode": "DISABLED",
  "connectionDraining": {
    "drainingTimeoutSec": 300
  },
  "description": "",
  "enableCDN": true,
  "healthChecks": [
    "projects/qwiklabs-gcp-04-7195e922f04d/global/healthChecks/http-health-check"
  ],
  "ipAddressSelectionPolicy": "IPV4_ONLY",
  "loadBalancingScheme": "EXTERNAL_MANAGED",
  "localityLbPolicy": "ROUND_ROBIN",
  "logConfig": {
    "enable": true,
    "sampleRate": 1
  },
  "name": "http-backend",
  "portName": "http",
  "protocol": "HTTP",
  "securityPolicy": "projects/qwiklabs-gcp-04-7195e922f04d/global/securityPolicies/default-security-policy-for-backend-service-http-backend",
  "sessionAffinity": "NONE",
  "timeoutSec": 30
}

POST https://compute.googleapis.com/compute/v1/projects/qwiklabs-gcp-04-7195e922f04d/global/backendServices/http-backend/setSecurityPolicy
{
  "securityPolicy": "projects/qwiklabs-gcp-04-7195e922f04d/global/securityPolicies/default-security-policy-for-backend-service-http-backend"
}

POST https://compute.googleapis.com/compute/v1/projects/qwiklabs-gcp-04-7195e922f04d/global/urlMaps
{
  "defaultService": "projects/qwiklabs-gcp-04-7195e922f04d/global/backendServices/http-backend",
  "name": "http-lb"
}

POST https://compute.googleapis.com/compute/v1/projects/qwiklabs-gcp-04-7195e922f04d/global/targetHttpProxies
{
  "name": "http-lb-target-proxy",
  "urlMap": "projects/qwiklabs-gcp-04-7195e922f04d/global/urlMaps/http-lb"
}

POST https://compute.googleapis.com/compute/v1/projects/qwiklabs-gcp-04-7195e922f04d/global/forwardingRules
{
  "IPProtocol": "TCP",
  "ipVersion": "IPV4",
  "loadBalancingScheme": "EXTERNAL_MANAGED",
  "name": "http-lb-forwarding-rule",
  "networkTier": "PREMIUM",
  "portRange": "80",
  "target": "projects/qwiklabs-gcp-04-7195e922f04d/global/targetHttpProxies/http-lb-target-proxy"
}

POST https://compute.googleapis.com/compute/v1/projects/qwiklabs-gcp-04-7195e922f04d/global/targetHttpProxies
{
  "name": "http-lb-target-proxy-2",
  "urlMap": "projects/qwiklabs-gcp-04-7195e922f04d/global/urlMaps/http-lb"
}

POST https://compute.googleapis.com/compute/v1/projects/qwiklabs-gcp-04-7195e922f04d/global/forwardingRules
{
  "IPProtocol": "TCP",
  "ipVersion": "IPV6",
  "loadBalancingScheme": "EXTERNAL_MANAGED",
  "name": "http-lb-forwarding-rule-2",
  "networkTier": "PREMIUM",
  "portRange": "80",
  "target": "projects/qwiklabs-gcp-04-7195e922f04d/global/targetHttpProxies/http-lb-target-proxy-2"
}

POST https://compute.googleapis.com/compute/beta/projects/qwiklabs-gcp-04-7195e922f04d/regions/us-central1/instanceGroups/us-central1-mig/setNamedPorts
{
  "namedPorts": [
    {
      "name": "http",
      "port": 80
    }
  ]
}

POST https://compute.googleapis.com/compute/beta/projects/qwiklabs-gcp-04-7195e922f04d/regions/us-west1/instanceGroups/us-west1-mig/setNamedPorts
{
  "namedPorts": [
    {
      "name": "http",
      "port": 80
    }
  ]
}
```

This configuration means that the load balancer attempts to keep each instance of **-mig** at or below 50 requests per second (RPS).

This configuration means that the load balancer attempts to keep each instance of **-mig** at or below 80% CPU utilization.

Health checks determine which instances receive new connections. This  HTTP health check polls instances every 5 seconds, waits up to 5 seconds for a response and treats 2 successful or 2 failed attempts as healthy  or unhealthy, respectively.

Note the IPv4 and IPv6 addresses of the load balancer for the next task. They will be referred to as `[LB_IP_v4]` and `[LB_IP_v6]`, respectively.

```sh
student_01_aa8f3a74b714@cloudshell:~ (qwiklabs-gcp-04-7195e922f04d)$ gcloud compute forwarding-rules list
NAME: http-lb-forwarding-rule
REGION: 
IP_ADDRESS: 35.227.234.103
IP_PROTOCOL: TCP
TARGET: http-lb-target-proxy

NAME: http-lb-forwarding-rule-2
REGION: 
IP_ADDRESS: 2600:1901:0:5aa8::
IP_PROTOCOL: TCP
TARGET: http-lb-target-proxy-2
student_01_aa8f3a74b714@cloudshell:~ (qwiklabs-gcp-04-7195e922f04d)$ 
```

### Access the HTTP Load Balancer

To test IPv4 access to the HTTP Load Balancer, open a new tab in your browser and navigate to `http://[LB_IP_v4]`. Make sure to replace `[LB_IP_v4]` with the IPv4 address of the load balancer.





### Stress test the HTTP Load Balancer

Create a new VM to simulate a load on the HTTP Load Balancer using `siege`. Then, determine if traffic is balanced across both backends when the load is high.

```sh
student_01_aa8f3a74b714@cloudshell:~ (qwiklabs-gcp-04-7195e922f04d)$ gcloud compute instances create siege-vm --project=qwiklabs-gcp-04-7195e922f04d --zone=europe-west1-b --machine-type=e2-medium --network-interface=network-tier=PREMIUM,stack-type=IPV4_ONLY,subnet=default --metadata=enable-oslogin=true --maintenance-policy=MIGRATE --provisioning-model=STANDARD --service-account=271285976134-compute@developer.gserviceaccount.com --scopes=https://www.googleapis.com/auth/devstorage.read_only,https://www.googleapis.com/auth/logging.write,https://www.googleapis.com/auth/monitoring.write,https://www.googleapis.com/auth/servicecontrol,https://www.googleapis.com/auth/service.management.readonly,https://www.googleapis.com/auth/trace.append --create-disk=auto-delete=yes,boot=yes,device-name=siege-vm,image=projects/debian-cloud/global/images/debian-12-bookworm-v20240415,mode=rw,size=10,type=projects/qwiklabs-gcp-04-7195e922f04d/zones/europe-west1-b/diskTypes/pd-balanced --no-shielded-secure-boot --shielded-vtpm --shielded-integrity-monitoring --labels=goog-ec-src=vm_add-gcloud --reservation-affinity=any
Created [https://www.googleapis.com/compute/v1/projects/qwiklabs-gcp-04-7195e922f04d/zones/europe-west1-b/instances/siege-vm].
NAME: siege-vm
ZONE: europe-west1-b
MACHINE_TYPE: e2-medium
PREEMPTIBLE: 
INTERNAL_IP: 10.132.0.2
EXTERNAL_IP: 104.155.82.167
STATUS: RUNNING
student_01_aa8f3a74b714@cloudshell:~ (qwiklabs-gcp-04-7195e922f04d)$ 
```

```sh
student_01_aa8f3a74b714@cloudshell:~ (qwiklabs-gcp-04-7195e922f04d)$ gcloud compute ssh --zone "europe-west1-b" "siege-vm" --project "qwiklabs-gcp-04-7195e922f04d"
WARNING: The private SSH key file for gcloud does not exist.
WARNING: The public SSH key file for gcloud does not exist.
WARNING: You do not have an SSH key for gcloud.
WARNING: SSH keygen will be executed to generate a key.
This tool needs to create the directory [/home/student_01_aa8f3a74b714/.ssh] before being able to generate SSH keys.

Do you want to continue (Y/n)?  y

Generating public/private rsa key pair.
Enter passphrase (empty for no passphrase): 
Enter same passphrase again: 
Your identification has been saved in /home/student_01_aa8f3a74b714/.ssh/google_compute_engine
Your public key has been saved in /home/student_01_aa8f3a74b714/.ssh/google_compute_engine.pub
The key fingerprint is:
SHA256:1G9b8+G6niHZ8pxW6c2sm0Cchqum8flw7j4jsvXk+Cs student_01_aa8f3a74b714@cs-334013342407-default
The key's randomart image is:
+---[RSA 3072]----+
|                 |
|         .       |
|        . .      |
|       .   + .   |
|        S . B o..|
|           =oo.=.|
|      . o ++ooo+o|
|      .+E@o =o=o+|
|      o==BX+oO=o |
+----[SHA256]-----+
Warning: Permanently added 'compute.650609156702912553' (ED25519) to the list of known hosts.
Linux siege-vm.europe-west1-b.c.qwiklabs-gcp-04-7195e922f04d.internal 6.1.0-20-cloud-amd64 #1 SMP PREEMPT_DYNAMIC Debian 6.1.85-1 (2024-04-11) x86_64

The programs included with the Debian GNU/Linux system are free software;
the exact distribution terms for each program are described in the
individual files in /usr/share/doc/*/copyright.

Debian GNU/Linux comes with ABSOLUTELY NO WARRANTY, to the extent
permitted by applicable law.
Creating directory '/home/student-01-aa8f3a74b714'.
student-01-aa8f3a74b714@siege-vm:~$ 
```

```sh
student-01-aa8f3a74b714@siege-vm:~$ sudo apt-get -y install siege
Reading package lists... Done
Building dependency tree... Done
Reading state information... Done
The following NEW packages will be installed:
  siege
0 upgraded, 1 newly installed, 0 to remove and 0 not upgraded.
Need to get 106 kB of archives.
After this operation, 287 kB of additional disk space will be used.
Get:1 file:/etc/apt/mirrors/debian.list Mirrorlist [30 B]
Get:2 https://deb.debian.org/debian bookworm/main amd64 siege amd64 4.0.7-1+b1 [106 kB]
Fetched 106 kB in 0s (947 kB/s)
Selecting previously unselected package siege.
(Reading database ... 67169 files and directories currently installed.)
Preparing to unpack .../siege_4.0.7-1+b1_amd64.deb ...
Unpacking siege (4.0.7-1+b1) ...
Setting up siege (4.0.7-1+b1) ...
Processing triggers for man-db (2.11.2-2) ...
student-01-aa8f3a74b714@siege-vm:~$ 
```

```sh
student-01-aa8f3a74b714@siege-vm:~$ export LB_IP=35.227.234.103
```
```sh
student-01-aa8f3a74b714@siege-vm:~$ curl http://35.227.234.103/
<h1>HTTP Load Balancing Lab</h1><h2>Client IP</h2>Your IP address : 35.191.17.70<h2>Hostname</h2>Server Hostname: us-central1-mig-vb0q<h2>Server Location</h2>Region and Zone: us-central1-cstudent-01-aa8f3a74b714@siege-vm:~$ curl http://35.227.234.103/
<h1>HTTP Load Balancing Lab</h1><h2>Client IP</h2>Your IP address : 35.191.24.49<h2>Hostname</h2>Server Hostname: us-central1-mig-vb0q<h2>Server Location</h2>Region and Zone: us-central1-cstudent-01-aa8f3a74b714@siege-vm:~$ 
```

```sh
student-01-aa8f3a74b714@siege-vm:~$ siege -c 150 -t120s http://$LB_IP
New configuration template added to /home/student-01-aa8f3a74b714/.siege
Run siege -C to view the current settings in that file
^C
{       "transactions":                         3269,
        "availability":                       100.00,
        "elapsed_time":                       111.33,
        "data_transferred":                     0.50,
        "response_time":                        5.01,
        "transaction_rate":                    29.36,
        "throughput":                           0.00,
        "concurrency":                        147.05,
        "successful_transactions":              3269,
        "failed_transactions":                     0,
        "longest_transaction":                 17.30,
        "shortest_transaction":                 0.10
}
student-01-aa8f3a74b714@siege-vm:~$ 
```

## Denylist the siege-vm

Use Cloud Armor to denylist the **siege-vm** from accessing the HTTP Load Balancer.

### Create the security policy

Create a Cloud Armor security policy with a denylist rule for the **siege-vm**.

```sh
POST https://compute.googleapis.com/compute/v1/projects/qwiklabs-gcp-04-7195e922f04d/global/securityPolicies
{
  "adaptiveProtectionConfig": {
    "layer7DdosDefenseConfig": {
      "enable": false
    }
  },
  "description": "",
  "name": "denylist-seige",
  "rules": [
    {
      "action": "deny(403)",
      "description": "",
      "match": {
        "config": {
          "srcIpRanges": [
            "104.155.82.167"
          ]
        },
        "expr": {
          "description": "",
          "expression": "",
          "location": "",
          "title": ""
        },
        "versionedExpr": "SRC_IPS_V1"
      },
      "preview": false,
      "priority": 1000
    },
    {
      "action": "allow",
      "description": "Default rule, higher priority overrides it",
      "match": {
        "config": {
          "srcIpRanges": [
            "*"
          ]
        },
        "expr": {
          "description": "",
          "expression": "",
          "location": "",
          "title": ""
        },
        "versionedExpr": "SRC_IPS_V1"
      },
      "preview": false,
      "priority": 2147483647
    }
  ],
  "type": "CLOUD_ARMOR"
} && POST https://compute.googleapis.com/compute/v1/projects/qwiklabs-gcp-04-7195e922f04d/global/backendServices/http-backend/setSecurityPolicy
{
  "securityPolicy": "projects/qwiklabs-gcp-04-7195e922f04d/global/securityPolicies/denylist-seige"
}
```

### Verify the security policy

Verify that the **siege-vm** cannot access the HTTP Load Balancer.

1. Return to the **SSH** terminal of **siege-vm**.
2. To access the load balancer, run the following:

```sh
student-01-aa8f3a74b714@siege-vm:~$ curl http://35.227.234.103/
<!doctype html><meta charset="utf-8"><meta name=viewport content="width=device-width, initial-scale=1"><title>403</title>403 Forbiddenstudent-01-aa8f3a74b714@siege-vm:~$ curl http://35.227.234.103/
<!doctype html><meta charset="utf-8"><meta name=viewport content="width=device-width, initial-scale=1"><title>403</title>403 Forbiddenstudent-01-aa8f3a74b714@siege-vm:~$ curl http://35.227.234.103/
<!doctype html><meta charset="utf-8"><meta name=viewport content="width=device-width, initial-scale=1"><title>403</title>403 Forbiddenstudent-01-aa8f3a74b714@siege-vm:~$ 
student-01-aa8f3a74b714@siege-vm:~$ 
```

It might take a couple of minutes for the security policy to take  effect. If you are able to access the backends, keep trying until you  get the **403 Forbidden error**.



You can access the HTTP Load Balancer from your browser because of the default rule to **allow** traffic; however, you cannot access it from the **siege-vm** because of the **deny** rule that you implemented.

On the Logging page, make sure to clear all the text in the **Query preview**. Select resource to **Application Load Balancer** > **http-lb-forwarding-rule** > **http-lb** then click **Apply**.



```json
{
  "insertId": "32h8sbf3mwvm1",
  "jsonPayload": {
    "backendTargetProjectNumber": "projects/271285976134",
    "statusDetails": "denied_by_security_policy",
    "cacheId": "BRU-e02d25d8",
    "@type": "type.googleapis.com/google.cloud.loadbalancing.type.LoadBalancerLogEntry",
    "enforcedSecurityPolicy": {
      "name": "denylist-seige",
      "priority": 1000,
      "outcome": "DENY",
      "configuredAction": "DENY"
    },
    "cacheDecision": [
      "RESPONSE_HAS_CONTENT_TYPE",
      "CACHE_MODE_CACHE_ALL_STATIC"
    ],
    "securityPolicyRequestData": {
      "remoteIpInfo": {
        "regionCode": "BE"
      }
    },
    "remoteIp": "104.155.82.167"
  },
  "httpRequest": {
    "requestMethod": "GET",
    "requestUrl": "http://35.227.234.103/",
    "requestSize": "78",
    "status": 403,
    "responseSize": "275",
    "userAgent": "curl/7.88.1",
    "remoteIp": "104.155.82.167",
    "cacheLookup": true,
    "latency": "0.214118s"
  },
  "resource": {
    "type": "http_load_balancer",
    "labels": {
      "url_map_name": "http-lb",
      "project_id": "qwiklabs-gcp-04-7195e922f04d",
      "zone": "global",
      "backend_service_name": "http-backend",
      "target_proxy_name": "http-lb-target-proxy",
      "forwarding_rule_name": "http-lb-forwarding-rule"
    }
  },
  "timestamp": "2024-05-19T04:36:08.674350Z",
  "severity": "WARNING",
  "logName": "projects/qwiklabs-gcp-04-7195e922f04d/logs/requests",
  "trace": "projects/qwiklabs-gcp-04-7195e922f04d/traces/db90b6145b38eca5dc6474f9a87e0fb7",
  "receiveTimestamp": "2024-05-19T04:36:10.033657347Z",
  "spanId": "d34e368dbdc90002"
}
```

Cloud Armor security policies create logs that can be explored to  determine when traffic is denied and when it is allowed, along with the  source of the traffic.

You configured an HTTP Load Balancer with backends in  and . Then, you stress tested the Load Balancer with a VM and denylisted the IP address of that VM with Cloud Armor. You were able to explore the security policy logs to identify why the traffic was blocked.
