---
layout: single
title:  "Configuring Network and HTTP Load Balancers in GCP"
categories: Cloud
tags: GCP
classes: wide
show_date: true
header:
  overlay_image: /assets/images/gcp-banner-1.png
  og_image: /assets/images/gcp-banner-1.png
  teaser: /assets/images/gpcne.png
author:
  name     : "Professional Cloud Architect"
  avatar   : "/assets/images/pca-gcp.png"

sidebar:
  - title: "Topics"
    nav: my-sidebar

---

# Configuring a Network and  HTTP Load Balancer in GCP

## Setup Network Load Balancer

```sh

Welcome to Cloud Shell! Type "help" to get started.
Your Cloud Platform project in this session is set to qwiklabs-gcp-00-925831822ff8.
Use “gcloud config set project [PROJECT_ID]” to change to a different project.
student_01_6c95c772cfb8@cloudshell:~ (qwiklabs-gcp-00-925831822ff8)$ gcloud config set compute/region us-central1
Updated property [compute/region].
student_01_6c95c772cfb8@cloudshell:~ (qwiklabs-gcp-00-925831822ff8)$ gcloud config set compute/zone us-central1-a
Updated property [compute/zone].
student_01_6c95c772cfb8@cloudshell:~ (qwiklabs-gcp-00-925831822ff8)$   gcloud compute instances create www1 \
    --zone=us-central1-a \
    --tags=network-lb-tag \
    --machine-type=e2-small \
    --image-family=debian-11 \
    --image-project=debian-cloud \
    --metadata=startup-script='#!/bin/bash
      apt-get update
      apt-get install apache2 -y
      service apache2 restart
      echo "
<h3>Web Server: www1</h3>" | tee /var/www/html/index.html'
Created [https://www.googleapis.com/compute/v1/projects/qwiklabs-gcp-00-925831822ff8/zones/us-central1-a/instances/www1].
NAME: www1
ZONE: us-central1-a
MACHINE_TYPE: e2-small
PREEMPTIBLE: 
INTERNAL_IP: 10.128.0.2
EXTERNAL_IP: 34.132.60.231
STATUS: RUNNING
student_01_6c95c772cfb8@cloudshell:~ (qwiklabs-gcp-00-925831822ff8)$   gcloud compute instances create www2 \
    --zone=us-central1-a \
    --tags=network-lb-tag \
    --machine-type=e2-small \
    --image-family=debian-11 \
    --image-project=debian-cloud \
    --metadata=startup-script='#!/bin/bash
      apt-get update
      apt-get install apache2 -y
      service apache2 restart
      echo "
<h3>Web Server: www2</h3>" | tee /var/www/html/index.html'
Created [https://www.googleapis.com/compute/v1/projects/qwiklabs-gcp-00-925831822ff8/zones/us-central1-a/instances/www2].
NAME: www2
ZONE: us-central1-a
MACHINE_TYPE: e2-small
PREEMPTIBLE: 
INTERNAL_IP: 10.128.0.3
EXTERNAL_IP: 34.133.34.101
STATUS: RUNNING
student_01_6c95c772cfb8@cloudshell:~ (qwiklabs-gcp-00-925831822ff8)$   gcloud compute instances create www3 \
    --zone=us-central1-a  \
    --tags=network-lb-tag \
    --machine-type=e2-small \
    --image-family=debian-11 \
    --image-project=debian-cloud \
    --metadata=startup-script='#!/bin/bash
      apt-get update
      apt-get install apache2 -y
      service apache2 restart
      echo "
<h3>Web Server: www3</h3>" | tee /var/www/html/index.html'
Created [https://www.googleapis.com/compute/v1/projects/qwiklabs-gcp-00-925831822ff8/zones/us-central1-a/instances/www3].
NAME: www3
ZONE: us-central1-a
MACHINE_TYPE: e2-small
PREEMPTIBLE: 
INTERNAL_IP: 10.128.0.4
EXTERNAL_IP: 34.172.62.255
STATUS: RUNNING
student_01_6c95c772cfb8@cloudshell:~ (qwiklabs-gcp-00-925831822ff8)$ gcloud compute firewall-rules create www-firewall-network-lb \
    --target-tags network-lb-tag --allow tcp:80
Creating firewall...working..Created [https://www.googleapis.com/compute/v1/projects/qwiklabs-gcp-00-925831822ff8/global/firewalls/www-firewall-network-lb].
Creating firewall...done.                                                                                                                     
NAME: www-firewall-network-lb
NETWORK: default
DIRECTION: INGRESS
PRIORITY: 1000
ALLOW: tcp:80
DENY: 
DISABLED: False
student_01_6c95c772cfb8@cloudshell:~ (qwiklabs-gcp-00-925831822ff8)$ gcloud compute instances list
NAME: www1
ZONE: us-central1-a
MACHINE_TYPE: e2-small
PREEMPTIBLE: 
INTERNAL_IP: 10.128.0.2
EXTERNAL_IP: 34.132.60.231
STATUS: RUNNING

NAME: www2
ZONE: us-central1-a
MACHINE_TYPE: e2-small
PREEMPTIBLE: 
INTERNAL_IP: 10.128.0.3
EXTERNAL_IP: 34.133.34.101
STATUS: RUNNING

NAME: www3
ZONE: us-central1-a
MACHINE_TYPE: e2-small
PREEMPTIBLE: 
INTERNAL_IP: 10.128.0.4
EXTERNAL_IP: 34.172.62.255
STATUS: RUNNING
student_01_6c95c772cfb8@cloudshell:~ (qwiklabs-gcp-00-925831822ff8)$ curl 34.132.60.231

<h3>Web Server: www1</h3>
student_01_6c95c772cfb8@cloudshell:~ (qwiklabs-gcp-00-925831822ff8)$ curl 34.133.34.101

<h3>Web Server: www2</h3>
student_01_6c95c772cfb8@cloudshell:~ (qwiklabs-gcp-00-925831822ff8)$ curl 34.172.62.255

<h3>Web Server: www3</h3>
student_01_6c95c772cfb8@cloudshell:~ (qwiklabs-gcp-00-925831822ff8)$ gcloud compute addresses create network-lb-ip-1 \
  --region us-central1
Created [https://www.googleapis.com/compute/v1/projects/qwiklabs-gcp-00-925831822ff8/regions/us-central1/addresses/network-lb-ip-1].
student_01_6c95c772cfb8@cloudshell:~ (qwiklabs-gcp-00-925831822ff8)$ gcloud compute http-health-checks create basic-check
Created [https://www.googleapis.com/compute/v1/projects/qwiklabs-gcp-00-925831822ff8/global/httpHealthChecks/basic-check].
NAME: basic-check
HOST: 
PORT: 80
REQUEST_PATH: /
student_01_6c95c772cfb8@cloudshell:~ (qwiklabs-gcp-00-925831822ff8)$ gcloud compute target-pools create www-pool \
  --region us-central1 --http-health-check basic-check
Created [https://www.googleapis.com/compute/v1/projects/qwiklabs-gcp-00-925831822ff8/regions/us-central1/targetPools/www-pool].
NAME: www-pool
REGION: us-central1
SESSION_AFFINITY: NONE
BACKUP: 
HEALTH_CHECKS: basic-check
student_01_6c95c772cfb8@cloudshell:~ (qwiklabs-gcp-00-925831822ff8)$ gcloud compute target-pools add-instances www-pool \
    --instances www1,www2,www3
Updated [https://www.googleapis.com/compute/v1/projects/qwiklabs-gcp-00-925831822ff8/regions/us-central1/targetPools/www-pool].
student_01_6c95c772cfb8@cloudshell:~ (qwiklabs-gcp-00-925831822ff8)$ gcloud compute forwarding-rules create www-rule \
    --region  us-central1 \
    --ports 80 \
    --address network-lb-ip-1 \
    --target-pool www-pool
Created [https://www.googleapis.com/compute/v1/projects/qwiklabs-gcp-00-925831822ff8/regions/us-central1/forwardingRules/www-rule].
student_01_6c95c772cfb8@cloudshell:~ (qwiklabs-gcp-00-925831822ff8)$ gcloud compute forwarding-rules describe www-rule --region us-central1
IPAddress: 34.172.10.49
IPProtocol: TCP
creationTimestamp: '2024-04-01T09:56:21.359-07:00'
description: ''
fingerprint: u3UsHb_XK0Y=
id: '647094288607828442'
kind: compute#forwardingRule
labelFingerprint: 42WmSpB8rSM=
loadBalancingScheme: EXTERNAL
name: www-rule
networkTier: PREMIUM
portRange: 80-80
region: https://www.googleapis.com/compute/v1/projects/qwiklabs-gcp-00-925831822ff8/regions/us-central1
selfLink: https://www.googleapis.com/compute/v1/projects/qwiklabs-gcp-00-925831822ff8/regions/us-central1/forwardingRules/www-rule
target: https://www.googleapis.com/compute/v1/projects/qwiklabs-gcp-00-925831822ff8/regions/us-central1/targetPools/www-pool
student_01_6c95c772cfb8@cloudshell:~ (qwiklabs-gcp-00-925831822ff8)$ IPADDRESS=$(gcloud compute forwarding-rules describe www-rule --region us-central1 --format="json" | jq -r .IPAddress)
student_01_6c95c772cfb8@cloudshell:~ (qwiklabs-gcp-00-925831822ff8)$ echo $IPADDRESS
34.172.10.49
student_01_6c95c772cfb8@cloudshell:~ (qwiklabs-gcp-00-925831822ff8)$ while true; do curl -m1 $IPADDRESS; done

<h3>Web Server: www1</h3>

<h3>Web Server: www1</h3>

<h3>Web Server: www3</h3>

<h3>Web Server: www3</h3>

<h3>Web Server: www2</h3>

<h3>Web Server: www1</h3>

<h3>Web Server: www3</h3>

<h3>Web Server: www2</h3>

<h3>Web Server: www1</h3>

<h3>Web Server: www3</h3>
^C
student_01_6c95c772cfb8@cloudshell:~ (qwiklabs-gcp-00-925831822ff8)$ 
```
## Setup HTTP Load Balancer

```sh
student_01_6c95c772cfb8@cloudshell:~ (qwiklabs-gcp-00-925831822ff8)$ 
student_01_6c95c772cfb8@cloudshell:~ (qwiklabs-gcp-00-925831822ff8)$ gcloud compute instance-templates create lb-backend-template \
   --region=us-central1 \
   --network=default \
   --subnet=default \
   --tags=allow-health-check \
   --machine-type=e2-medium \
   --image-family=debian-11 \
   --image-project=debian-cloud \
   --metadata=startup-script='#!/bin/bash
     apt-get update
     apt-get install apache2 -y
     a2ensite default-ssl
     a2enmod ssl
     vm_hostname="$(curl -H "Metadata-Flavor:Google" \
     http://169.254.169.254/computeMetadata/v1/instance/name)"
     echo "Page served from: $vm_hostname" | \
     tee /var/www/html/index.html
     systemctl restart apache2'
Created [https://www.googleapis.com/compute/v1/projects/qwiklabs-gcp-00-925831822ff8/global/instanceTemplates/lb-backend-template].
NAME: lb-backend-template
MACHINE_TYPE: e2-medium
PREEMPTIBLE: 
CREATION_TIMESTAMP: 2024-04-01T10:00:02.608-07:00
student_01_6c95c772cfb8@cloudshell:~ (qwiklabs-gcp-00-925831822ff8)$ gcloud compute instance-groups managed create lb-backend-group \
   --template=lb-backend-template --size=2 --zone=us-central1-a
Created [https://www.googleapis.com/compute/v1/projects/qwiklabs-gcp-00-925831822ff8/zones/us-central1-a/instanceGroupManagers/lb-backend-group].
NAME: lb-backend-group
LOCATION: us-central1-a
SCOPE: zone
BASE_INSTANCE_NAME: lb-backend-group
SIZE: 0
TARGET_SIZE: 2
INSTANCE_TEMPLATE: lb-backend-template
AUTOSCALED: no
student_01_6c95c772cfb8@cloudshell:~ (qwiklabs-gcp-00-925831822ff8)$ gcloud compute firewall-rules create fw-allow-health-check \
  --network=default \
  --action=allow \
  --direction=ingress \
  --source-ranges=130.211.0.0/22,35.191.0.0/16 \
  --target-tags=allow-health-check \
  --rules=tcp:80
Creating firewall...working..Created [https://www.googleapis.com/compute/v1/projects/qwiklabs-gcp-00-925831822ff8/global/firewalls/fw-allow-health-check].
Creating firewall...done.                                                                                                                   
NAME: fw-allow-health-check
NETWORK: default
DIRECTION: INGRESS
PRIORITY: 1000
ALLOW: tcp:80
DENY: 
DISABLED: False
student_01_6c95c772cfb8@cloudshell:~ (qwiklabs-gcp-00-925831822ff8)$ gcloud compute addresses create lb-ipv4-1 \
  --ip-version=IPV4 \
  --global
Created [https://www.googleapis.com/compute/v1/projects/qwiklabs-gcp-00-925831822ff8/global/addresses/lb-ipv4-1].
student_01_6c95c772cfb8@cloudshell:~ (qwiklabs-gcp-00-925831822ff8)$ gcloud compute addresses describe lb-ipv4-1 \
  --format="get(address)" \
  --global
34.107.158.122
student_01_6c95c772cfb8@cloudshell:~ (qwiklabs-gcp-00-925831822ff8)$ gcloud compute health-checks create http http-basic-check \
  --port 80
Created [https://www.googleapis.com/compute/v1/projects/qwiklabs-gcp-00-925831822ff8/global/healthChecks/http-basic-check].
NAME: http-basic-check
PROTOCOL: HTTP
student_01_6c95c772cfb8@cloudshell:~ (qwiklabs-gcp-00-925831822ff8)$ gcloud compute backend-services create web-backend-service \
  --protocol=HTTP \
  --port-name=http \
  --health-checks=http-basic-check \
  --global
Created [https://www.googleapis.com/compute/v1/projects/qwiklabs-gcp-00-925831822ff8/global/backendServices/web-backend-service].
NAME: web-backend-service
BACKENDS: 
PROTOCOL: HTTP
student_01_6c95c772cfb8@cloudshell:~ (qwiklabs-gcp-00-925831822ff8)$ gcloud compute backend-services add-backend web-backend-service \
  --instance-group=lb-backend-group \
  --instance-group-zone=us-central1-a \
  --global
Updated [https://www.googleapis.com/compute/v1/projects/qwiklabs-gcp-00-925831822ff8/global/backendServices/web-backend-service].
student_01_6c95c772cfb8@cloudshell:~ (qwiklabs-gcp-00-925831822ff8)$ gcloud compute url-maps create web-map-http \
    --default-service web-backend-service
Created [https://www.googleapis.com/compute/v1/projects/qwiklabs-gcp-00-925831822ff8/global/urlMaps/web-map-http].
NAME: web-map-http
DEFAULT_SERVICE: backendServices/web-backend-service
student_01_6c95c772cfb8@cloudshell:~ (qwiklabs-gcp-00-925831822ff8)$ gcloud compute target-http-proxies create http-lb-proxy \
    --url-map web-map-http
Created [https://www.googleapis.com/compute/v1/projects/qwiklabs-gcp-00-925831822ff8/global/targetHttpProxies/http-lb-proxy].
NAME: http-lb-proxy
URL_MAP: web-map-http
student_01_6c95c772cfb8@cloudshell:~ (qwiklabs-gcp-00-925831822ff8)$ gcloud compute forwarding-rules create http-content-rule \
   --address=lb-ipv4-1\
   --global \
   --target-http-proxy=http-lb-proxy \
   --ports=80

Created [https://www.googleapis.com/compute/v1/projects/qwiklabs-gcp-00-925831822ff8/global/forwardingRules/http-content-rule].
student_01_6c95c772cfb8@cloudshell:~ (qwiklabs-gcp-00-925831822ff8)$ 
```
## Summary
```sh
student_01_6c95c772cfb8@cloudshell:~ (qwiklabs-gcp-00-925831822ff8)$ history 
    1  gcloud config set compute/region us-central1
    2  gcloud config set compute/zone us-central1-a
    3  gcloud compute firewall-rules create www-firewall-network-lb     --target-tags network-lb-tag --allow tcp:80
    4  gcloud compute instances list
    5  curl 34.132.60.231
    6  curl 34.133.34.101
    7  curl 34.172.62.255
    8  gcloud compute addresses create network-lb-ip-1   --region us-central1
    9  gcloud compute http-health-checks create basic-check
   10  gcloud compute target-pools create www-pool   --region us-central1 --http-health-check basic-check
   11  gcloud compute target-pools add-instances www-pool     --instances www1,www2,www3
   12  gcloud compute forwarding-rules create www-rule     --region  us-central1     --ports 80     --address network-lb-ip-1     --target-pool www-pool
   13  gcloud compute forwarding-rules describe www-rule --region us-central1
   14  IPADDRESS=$(gcloud compute forwarding-rules describe www-rule --region us-central1 --format="json" | jq -r .IPAddress)
   15  echo $IPADDRESS
   16  while true; do curl -m1 $IPADDRESS; done
   17  gcloud compute instance-templates create lb-backend-template    --region=us-central1    --network=default    --subnet=default    --tags=allow-health-check    --machine-type=e2-medium    --image-family=debian-11    --image-project=debian-cloud    --metadata=startup-script='#!/bin/bash
     apt-get update
     apt-get install apache2 -y
     a2ensite default-ssl
     a2enmod ssl
     vm_hostname="$(curl -H "Metadata-Flavor:Google" \
     http://169.254.169.254/computeMetadata/v1/instance/name)"
     echo "Page served from: $vm_hostname" | \
     tee /var/www/html/index.html
     systemctl restart apache2'
   18  gcloud compute instance-groups managed create lb-backend-group    --template=lb-backend-template --size=2 --zone=us-central1-a
   19  gcloud compute firewall-rules create fw-allow-health-check   --network=default   --action=allow   --direction=ingress   --source-ranges=130.211.0.0/22,35.191.0.0/16   --target-tags=allow-health-check   --rules=tcp:80
   20  gcloud compute addresses create lb-ipv4-1   --ip-version=IPV4   --global
   21  gcloud compute addresses describe lb-ipv4-1   --format="get(address)"   --global
   22  gcloud compute health-checks create http http-basic-check   --port 80
   23  gcloud compute backend-services create web-backend-service   --protocol=HTTP   --port-name=http   --health-checks=http-basic-check   --global
   24  gcloud compute backend-services add-backend web-backend-service   --instance-group=lb-backend-group   --instance-group-zone=us-central1-a   --global
   25  gcloud compute url-maps create web-map-http     --default-service web-backend-service
   26  gcloud compute target-http-proxies create http-lb-proxy     --url-map web-map-http
   27  gcloud compute forwarding-rules create http-content-rule    --address=lb-ipv4-1   --global    --target-http-proxy=http-lb-proxy    --ports=80
   28  history 
student_01_6c95c772cfb8@cloudshell:~ (qwiklabs-gcp-00-925831822ff8)$ 
```





