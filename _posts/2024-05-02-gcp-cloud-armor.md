---

layout: single
title:  "Cloud Armor Preconfigured WAF Rules"
categories: Cloud
tags: GCP
toc: true
toc_sticky: true
show_date: true
header:
  overlay_image: /assets/images/gcp-banner.png
  og_image: /assets/images/gcp-banner.png
  teaser: /assets/images/gpcne.png
author:
  name     : "Cloud Network Engineer"
  avatar   : "/assets/images/gpcne.png"

sidebar:
  - title: "Topics"
    nav: my-sidebar

---

# Cloud Armor Preconfigured WAF Rules

Google Cloud Armor is Google's enterprise edge network security  solution providing DDOS protection, WAF rule enforcement, and adaptive  manageability at scale.

Cloud Armor has extended the preconfigured WAF rule sets to mitigate against the [OWASP Top 10](https://owasp.org/www-project-top-ten/) web application security vulnerabilities. The rule sets are based on the [OWASP Modsecurity core rule set](https://github.com/coreruleset/coreruleset/) version 3.0.2 to protect against some of the most common web  application security risks including local file inclusion (lfi), remote  file inclusion (rfi), remote code execution (rce), and many more.

Set up an Instance Group and a Global Load Balancer to support a service

Configure Cloud Armor security policies with preconfigured WAF rules to protect against lfi, rce, scanners, protocol attacks, and session  fixation

Validate that Cloud Armor mitigated an attack by observing logs

The [OWASP Juice Shop application](https://owasp.org/www-project-juice-shop/) is useful for security training and awareness, because it contains  instances of each of the OWASP Top 10 security vulnerabilities—by  design. An attacker can exploit it for testing purposes. In this lab,  you use it to demonstrate some application attacks followed by  protecting the application with Cloud Armor WAF rules. The application  is fronted by a Google Cloud Load Balancer, onto which the Cloud Armor  security policy and rules are be applied. It is served on the public  internet thus reachable from almost anywhere and protected using Cloud  Armor and VPC firewall rules.

```sh
Welcome to Cloud Shell! Type "help" to get started.
Your Cloud Platform project in this session is set to qwiklabs-gcp-04-08c687827d12.
Use “gcloud config set project [PROJECT_ID]” to change to a different project.
student_04_c33562da3df8@cloudshell:~ (qwiklabs-gcp-04-08c687827d12)$ gcloud config list project
export PROJECT_ID=$(gcloud config get-value project)
echo $PROJECT_ID
gcloud config set project $PROJECT_ID
[core]
project = qwiklabs-gcp-04-08c687827d12

Your active configuration is: [cloudshell-11076]
Your active configuration is: [cloudshell-11076]
qwiklabs-gcp-04-08c687827d12
Updated property [core/project].
student_04_c33562da3df8@cloudshell:~ (qwiklabs-gcp-04-08c687827d12)$ 
```



## Create the VPC network

```sh
student_04_c33562da3df8@cloudshell:~ (qwiklabs-gcp-04-08c687827d12)$ gcloud compute networks create ca-lab-vpc --subnet-mode custom
Created [https://www.googleapis.com/compute/v1/projects/qwiklabs-gcp-04-08c687827d12/global/networks/ca-lab-vpc].
NAME: ca-lab-vpc
SUBNET_MODE: CUSTOM
BGP_ROUTING_MODE: REGIONAL
IPV4_RANGE: 
GATEWAY_IPV4: 

Instances on this network will not be reachable until firewall rules
are created. As an example, you can allow all internal traffic between
instances as well as SSH, RDP, and ICMP by running:

$ gcloud compute firewall-rules create <FIREWALL_NAME> --network ca-lab-vpc --allow tcp,udp,icmp --source-ranges <IP_RANGE>
$ gcloud compute firewall-rules create <FIREWALL_NAME> --network ca-lab-vpc --allow tcp:22,tcp:3389,icmp

student_04_c33562da3df8@cloudshell:~ (qwiklabs-gcp-04-08c687827d12)$ 
```



### Create a subnet

```sh
student_04_c33562da3df8@cloudshell:~ (qwiklabs-gcp-04-08c687827d12)$ gcloud compute networks subnets create ca-lab-subnet \
        --network ca-lab-vpc --range 10.0.0.0/24 --region us-west1
Created [https://www.googleapis.com/compute/v1/projects/qwiklabs-gcp-04-08c687827d12/regions/us-west1/subnetworks/ca-lab-subnet].
NAME: ca-lab-subnet
REGION: us-west1
NETWORK: ca-lab-vpc
RANGE: 10.0.0.0/24
STACK_TYPE: IPV4_ONLY
IPV6_ACCESS_TYPE: 
INTERNAL_IPV6_PREFIX: 
EXTERNAL_IPV6_PREFIX: 
student_04_c33562da3df8@cloudshell:~ (qwiklabs-gcp-04-08c687827d12)$ 
```



### Create VPC firewall rules

After creating the VPC and subnet, set up a few firewall rules.

- The first firewall rule named `allow-js-site` allows all IPs to access the external IP of the test application's website on port `3000`.
- The second firewall rule named `allow-health-check` allows health-checks from source IP of the load balancers.

```sh
student_04_c33562da3df8@cloudshell:~ (qwiklabs-gcp-04-08c687827d12)$ gcloud compute firewall-rules create allow-js-site --allow tcp:3000 --network ca-lab-vpc
Creating firewall...working..Created [https://www.googleapis.com/compute/v1/projects/qwiklabs-gcp-04-08c687827d12/global/firewalls/allow-js-site].                                 
Creating firewall...done.                                                                                                                                                          
NAME: allow-js-site
NETWORK: ca-lab-vpc
DIRECTION: INGRESS
PRIORITY: 1000
ALLOW: tcp:3000
DENY: 
DISABLED: False
student_04_c33562da3df8@cloudshell:~ (qwiklabs-gcp-04-08c687827d12)$ 
```

```sh
student_04_c33562da3df8@cloudshell:~ (qwiklabs-gcp-04-08c687827d12)$ gcloud compute firewall-rules create allow-health-check \
    --network=ca-lab-vpc \
    --action=allow \
    --direction=ingress \
    --source-ranges=130.211.0.0/22,35.191.0.0/16 \
    --target-tags=allow-healthcheck \
    --rules=tcp
Creating firewall...working..Created [https://www.googleapis.com/compute/v1/projects/qwiklabs-gcp-04-08c687827d12/global/firewalls/allow-health-check].                            
Creating firewall...done.                                                                                                                                                          
NAME: allow-health-check
NETWORK: ca-lab-vpc
DIRECTION: INGRESS
PRIORITY: 1000
ALLOW: tcp
DENY: 
DISABLED: False
student_04_c33562da3df8@cloudshell:~ (qwiklabs-gcp-04-08c687827d12)$ 
```



## Set up the test application

Create the test application, in this case, the OWASP Juice Shop web  server. When you create the compute instance, you use a container image to  ensure the server has the appropriate services. You deploy this server  in the  and has a network tag that allows health checks.

### Create the OWASP Juice Shop application

- Use the open source well-known OWASP Juice Shop application to serve as the vulnerable application. You can also use this application to do  OWASP security challenges through the [OWASP website](https://owasp.org/www-project-juice-shop/).

```sh
student_04_c33562da3df8@cloudshell:~ (qwiklabs-gcp-04-08c687827d12)$ gcloud compute instances create-with-container owasp-juice-shop-app --container-image bkimminich/juice-shop \
     --network ca-lab-vpc \
     --subnet ca-lab-subnet \
     --private-network-ip=10.0.0.3 \
     --machine-type n1-standard-2 \
     --zone us-west1-c \
     --tags allow-healthcheck
Created [https://www.googleapis.com/compute/v1/projects/qwiklabs-gcp-04-08c687827d12/zones/us-west1-c/instances/owasp-juice-shop-app].
NAME: owasp-juice-shop-app
ZONE: us-west1-c
MACHINE_TYPE: n1-standard-2
PREEMPTIBLE: 
INTERNAL_IP: 10.0.0.3
EXTERNAL_IP: 34.82.141.64
STATUS: RUNNING
student_04_c33562da3df8@cloudshell:~ (qwiklabs-gcp-04-08c687827d12)$ 
```



### Set up the Cloud load balancer component: instance group

```sh
student_04_c33562da3df8@cloudshell:~ (qwiklabs-gcp-04-08c687827d12)$ gcloud compute instance-groups unmanaged create juice-shop-group \
    --zone=us-west1-c
Created [https://www.googleapis.com/compute/v1/projects/qwiklabs-gcp-04-08c687827d12/zones/us-west1-c/instanceGroups/juice-shop-group].
NAME: juice-shop-group
LOCATION: us-west1-c
SCOPE: zone
NETWORK: 
MANAGED: 
INSTANCES: 0
student_04_c33562da3df8@cloudshell:~ (qwiklabs-gcp-04-08c687827d12)$
```

Add the Juice Shop Google Compute Engine (GCE) instance to the unmanaged instance group:

```sh
student_04_c33562da3df8@cloudshell:~ (qwiklabs-gcp-04-08c687827d12)$ gcloud compute instance-groups unmanaged add-instances juice-shop-group \
    --zone=us-west1-c \
    --instances=owasp-juice-shop-app
Updated [https://www.googleapis.com/compute/v1/projects/qwiklabs-gcp-04-08c687827d12/zones/us-west1-c/instanceGroups/juice-shop-group].
student_04_c33562da3df8@cloudshell:~ (qwiklabs-gcp-04-08c687827d12)$ 
```

Set the named port to that of the Juice Shop application:

```sh
student_04_c33562da3df8@cloudshell:~ (qwiklabs-gcp-04-08c687827d12)$ gcloud compute instance-groups unmanaged set-named-ports \
juice-shop-group \
   --named-ports=http:3000 \
   --zone=us-west1-c
Updated [https://www.googleapis.com/compute/v1/projects/qwiklabs-gcp-04-08c687827d12/zones/us-west1-c/instanceGroups/juice-shop-group].
student_04_c33562da3df8@cloudshell:~ (qwiklabs-gcp-04-08c687827d12)$ 
```



### Set up the Cloud load balancer component: health check

Now that you've created the unmanaged instance group, create a health  check, backend service, URL map, target proxy, and forwarding rule.

```sh
student_04_c33562da3df8@cloudshell:~ (qwiklabs-gcp-04-08c687827d12)$ gcloud compute health-checks create tcp tcp-port-3000 \
        --port 3000
Created [https://www.googleapis.com/compute/v1/projects/qwiklabs-gcp-04-08c687827d12/global/healthChecks/tcp-port-3000].
NAME: tcp-port-3000
PROTOCOL: TCP
student_04_c33562da3df8@cloudshell:~ (qwiklabs-gcp-04-08c687827d12)$ 
```

### Set up the Cloud load balancer component: backend service

```sh
student_04_c33562da3df8@cloudshell:~ (qwiklabs-gcp-04-08c687827d12)$ gcloud compute backend-services create juice-shop-backend \
        --protocol HTTP \
        --port-name http \
        --health-checks tcp-port-3000 \
        --enable-logging \
        --global
Created [https://www.googleapis.com/compute/v1/projects/qwiklabs-gcp-04-08c687827d12/global/backendServices/juice-shop-backend].
NAME: juice-shop-backend
BACKENDS: 
PROTOCOL: HTTP
student_04_c33562da3df8@cloudshell:~ (qwiklabs-gcp-04-08c687827d12)$ 
```

Add the Juice Shop instance group to the backend service:

```sh
student_04_c33562da3df8@cloudshell:~ (qwiklabs-gcp-04-08c687827d12)$  gcloud compute backend-services add-backend juice-shop-backend \
        --instance-group=juice-shop-group \
        --instance-group-zone=us-west1-c \
        --global
Updated [https://www.googleapis.com/compute/v1/projects/qwiklabs-gcp-04-08c687827d12/global/backendServices/juice-shop-backend].
student_04_c33562da3df8@cloudshell:~ (qwiklabs-gcp-04-08c687827d12)$ 
```



### Set up the Cloud load balancer component: URL map

```sh
student_04_c33562da3df8@cloudshell:~ (qwiklabs-gcp-04-08c687827d12)$ gcloud compute url-maps create juice-shop-loadbalancer \
        --default-service juice-shop-backend
Created [https://www.googleapis.com/compute/v1/projects/qwiklabs-gcp-04-08c687827d12/global/urlMaps/juice-shop-loadbalancer].
NAME: juice-shop-loadbalancer
DEFAULT_SERVICE: backendServices/juice-shop-backend
student_04_c33562da3df8@cloudshell:~ (qwiklabs-gcp-04-08c687827d12)$ 
```



### Set up the Cloud load balancer component: target proxy

```sh
student_04_c33562da3df8@cloudshell:~ (qwiklabs-gcp-04-08c687827d12)$ gcloud compute target-http-proxies create juice-shop-proxy \
        --url-map juice-shop-loadbalancer
Created [https://www.googleapis.com/compute/v1/projects/qwiklabs-gcp-04-08c687827d12/global/targetHttpProxies/juice-shop-proxy].
NAME: juice-shop-proxy
URL_MAP: juice-shop-loadbalancer
student_04_c33562da3df8@cloudshell:~ (qwiklabs-gcp-04-08c687827d12)$ 
```



### Set up the Cloud load balancer component: forwarding rule

```sh
student_04_c33562da3df8@cloudshell:~ (qwiklabs-gcp-04-08c687827d12)$ gcloud compute forwarding-rules create juice-shop-rule \
        --global \
        --target-http-proxy=juice-shop-proxy \
        --ports=80
Created [https://www.googleapis.com/compute/v1/projects/qwiklabs-gcp-04-08c687827d12/global/forwardingRules/juice-shop-rule].
student_04_c33562da3df8@cloudshell:~ (qwiklabs-gcp-04-08c687827d12)$ 
```



### Verify the Juice Shop service is online

```sh
student_04_c33562da3df8@cloudshell:~ (qwiklabs-gcp-04-08c687827d12)$ PUBLIC_SVC_IP="$(gcloud compute forwarding-rules describe juice-shop-rule  --global --format="value(IPAddress)")"
echo $PUBLIC_SVC_IP
34.144.205.142
student_04_c33562da3df8@cloudshell:~ (qwiklabs-gcp-04-08c687827d12)$
```

```sh
student_04_c33562da3df8@cloudshell:~ (qwiklabs-gcp-04-08c687827d12)$ curl -Ii http://$PUBLIC_SVC_IP
HTTP/1.1 200 OK
Access-Control-Allow-Origin: *
X-Content-Type-Options: nosniff
X-Frame-Options: SAMEORIGIN
Feature-Policy: payment 'self'
X-Recruiting: /#/jobs
Accept-Ranges: bytes
Cache-Control: public, max-age=0
Last-Modified: Fri, 03 May 2024 02:16:21 GMT
ETag: W/"ea4-18f3c3cff0d"
Content-Type: text/html; charset=UTF-8
Content-Length: 3748
Vary: Accept-Encoding
Date: Fri, 03 May 2024 02:23:37 GMT
Via: 1.1 google

student_04_c33562da3df8@cloudshell:~ (qwiklabs-gcp-04-08c687827d12)$ 
```

You're now ready to explore the Juice Shop vulnerabilities and protect against them with Cloud Armor WAF rule sets.

## Demonstrate known vulnerabilities

In this lab, you demonstrate the states before and after Cloud Armor WAF rules are propagated in condensed steps.

### Observe an LFI vulnerability: path traversal

Local File Inclusion is the process of observing files present on the  server by exploiting lack of input validation in the request to  potentially expose sensitive data. The following shows a path traversal  is possible. In your browser or with curl, observe an existing path  served by the application.

```sh
student_04_c33562da3df8@cloudshell:~ (qwiklabs-gcp-04-08c687827d12)$ curl -Ii http://$PUBLIC_SVC_IP/ftp
HTTP/1.1 200 OK
Access-Control-Allow-Origin: *
X-Content-Type-Options: nosniff
X-Frame-Options: SAMEORIGIN
Feature-Policy: payment 'self'
X-Recruiting: /#/jobs
Content-Type: text/html; charset=utf-8
Content-Length: 11081
Vary: Accept-Encoding
Date: Fri, 03 May 2024 02:25:24 GMT
Via: 1.1 google

student_04_c33562da3df8@cloudshell:~ (qwiklabs-gcp-04-08c687827d12)$ 
```

Observe that path traversal works too.

```sh
student_04_c33562da3df8@cloudshell:~ (qwiklabs-gcp-04-08c687827d12)$ curl -Ii http://$PUBLIC_SVC_IP/ftp/../
HTTP/1.1 200 OK
Access-Control-Allow-Origin: *
X-Content-Type-Options: nosniff
X-Frame-Options: SAMEORIGIN
Feature-Policy: payment 'self'
X-Recruiting: /#/jobs
Accept-Ranges: bytes
Cache-Control: public, max-age=0
Last-Modified: Fri, 03 May 2024 02:16:21 GMT
ETag: W/"ea4-18f3c3cff0d"
Content-Type: text/html; charset=UTF-8
Content-Length: 3748
Vary: Accept-Encoding
Date: Fri, 03 May 2024 02:25:53 GMT
Via: 1.1 google

student_04_c33562da3df8@cloudshell:~ (qwiklabs-gcp-04-08c687827d12)$ 
```



### Observe an RCE vulnerability

Remote Code Execution includes various UNIX and Windows command  injection scenarios allowing attackers to execute OS commands usually  restricted to privileged users. The following shows a simple `ls` command execution passed in.

```sh
student_04_c33562da3df8@cloudshell:~ (qwiklabs-gcp-04-08c687827d12)$ curl -Ii http://$PUBLIC_SVC_IP/ftp?doc=/bin/ls
HTTP/1.1 200 OK
Access-Control-Allow-Origin: *
X-Content-Type-Options: nosniff
X-Frame-Options: SAMEORIGIN
Feature-Policy: payment 'self'
X-Recruiting: /#/jobs
Content-Type: text/html; charset=utf-8
Content-Length: 11081
Vary: Accept-Encoding
Date: Fri, 03 May 2024 02:26:31 GMT
Via: 1.1 google

student_04_c33562da3df8@cloudshell:~ (qwiklabs-gcp-04-08c687827d12)$ 
```

Remove the curl flags to observe the full output.



### Observe a well-known scanner's access

Both commercial and open source scan applications for various purposes,  including to find vulnerabilities. These tools use well-known User-Agent and other Headers. Observe curl works with a well-known User-Agent  Header.

```sh
student_04_c33562da3df8@cloudshell:~ (qwiklabs-gcp-04-08c687827d12)$ curl -Ii http://$PUBLIC_SVC_IP -H "User-Agent: blackwidow"
HTTP/1.1 200 OK
Access-Control-Allow-Origin: *
X-Content-Type-Options: nosniff
X-Frame-Options: SAMEORIGIN
Feature-Policy: payment 'self'
X-Recruiting: /#/jobs
Accept-Ranges: bytes
Cache-Control: public, max-age=0
Last-Modified: Fri, 03 May 2024 02:16:21 GMT
ETag: W/"ea4-18f3c3cff0d"
Content-Type: text/html; charset=UTF-8
Content-Length: 3748
Vary: Accept-Encoding
Date: Fri, 03 May 2024 02:28:29 GMT
Via: 1.1 google

student_04_c33562da3df8@cloudshell:~ (qwiklabs-gcp-04-08c687827d12)$ 
```



### Observe a protocol attack: HTTP splitting

Some web applications use input from the user to generate the headers in the responses. If the application doesn't properly filter the input, an attacker can potentially poison the input parameter with the  sequence `%0d%0a` (the CRLF sequence that is used to separate different lines).

The response could then be interpreted as two responses by anything  that happens to parse it, like an intermediary proxy server, potentially serving false content in subsequent requests. Insert the sequence `%0d%0a` into the input parameter, which can lead to serving a misleading page.

```sh
tudent_04_c33562da3df8@cloudshell:~ (qwiklabs-gcp-04-08c687827d12)$ curl -Ii "http://$PUBLIC_SVC_IP/index.html?foo=advanced%0d%0aContent-Length:%200%0d%0a%0d%0aHTTP/1.1%20200%20OK%0d%0aContent-Type:%20text/html%0d%0aContent-Length:%2035%0d%0a%0d%0a<html>Sorry,%20System%20Down</html>"
HTTP/1.1 200 OK
Access-Control-Allow-Origin: *
X-Content-Type-Options: nosniff
X-Frame-Options: SAMEORIGIN
Feature-Policy: payment 'self'
X-Recruiting: /#/jobs
Accept-Ranges: bytes
Cache-Control: public, max-age=0
Last-Modified: Fri, 03 May 2024 02:16:21 GMT
ETag: W/"ea4-18f3c3cff0d"
Content-Type: text/html; charset=UTF-8
Content-Length: 3748
Vary: Accept-Encoding
Date: Fri, 03 May 2024 02:29:44 GMT
Via: 1.1 google

student_04_c33562da3df8@cloudshell:~ (qwiklabs-gcp-04-08c687827d12)$ 
```



### Observe session fixation

```sh
student_04_c33562da3df8@cloudshell:~ (qwiklabs-gcp-04-08c687827d12)$ curl -Ii http://$PUBLIC_SVC_IP -H session_id=X
HTTP/1.1 200 OK
Access-Control-Allow-Origin: *
X-Content-Type-Options: nosniff
X-Frame-Options: SAMEORIGIN
Feature-Policy: payment 'self'
X-Recruiting: /#/jobs
Accept-Ranges: bytes
Cache-Control: public, max-age=0
Last-Modified: Fri, 03 May 2024 02:16:21 GMT
ETag: W/"ea4-18f3c3cff0d"
Content-Type: text/html; charset=UTF-8
Content-Length: 3748
Vary: Accept-Encoding
Date: Fri, 03 May 2024 02:30:22 GMT
Via: 1.1 google

student_04_c33562da3df8@cloudshell:~ (qwiklabs-gcp-04-08c687827d12)$ 
```



## Define Cloud Armor WAF rules

List the preconfigured WAF rules, using the following command in Cloud Shell:

```sh


student_04_c33562da3df8@cloudshell:~ (qwiklabs-gcp-04-08c687827d12)$ gcloud compute security-policies list-preconfigured-expression-sets 

{snip}
RULE_ID: owasp-crs-v030301-id944300-java
SENSITIVITY: 3


EXPRESSION_SET: nodejs-v33-stable


RULE_ID: owasp-crs-v030301-id934100-nodejs
SENSITIVITY: 1
student_04_c33562da3df8@cloudshell:~ (qwiklabs-gcp-04-08c687827d12)$ 
```

```sh
student_04_c33562da3df8@cloudshell:~ (qwiklabs-gcp-04-08c687827d12)$ gcloud compute security-policies create block-with-modsec-crs \
    --description "Block with OWASP ModSecurity CRS"
Created [https://www.googleapis.com/compute/v1/projects/qwiklabs-gcp-04-08c687827d12/global/securityPolicies/block-with-modsec-crs].
student_04_c33562da3df8@cloudshell:~ (qwiklabs-gcp-04-08c687827d12)$ 
```

Update the security policy default rule. The default rule priority has a numerical value of 2147483647.

```sh
student_04_c33562da3df8@cloudshell:~ (qwiklabs-gcp-04-08c687827d12)$ gcloud compute security-policies rules update 2147483647 \
    --security-policy block-with-modsec-crs \
    --action "deny-403"
Updated [https://www.googleapis.com/compute/v1/projects/qwiklabs-gcp-04-08c687827d12/global/securityPolicies/block-with-modsec-crs].
student_04_c33562da3df8@cloudshell:~ (qwiklabs-gcp-04-08c687827d12)$ 
```

Since the default rule is configured with action deny, you must allow  access from your IP. Please find your public IP (curl, ipmonkey,  whatismyip, etc):

```sh
student_04_c33562da3df8@cloudshell:~ (qwiklabs-gcp-04-08c687827d12)$ MY_IP=$(curl ifconfig.me)
  % Total    % Received % Xferd  Average Speed   Time    Time     Time  Current
                                 Dload  Upload   Total   Spent    Left  Speed
100    14  100    14    0     0     60      0 --:--:-- --:--:-- --:--:--    60
student_04_c33562da3df8@cloudshell:~ (qwiklabs-gcp-04-08c687827d12)$ 
```

Add the first rule to allow access from your IP (INSERT YOUR IP BELOW):

```sh
student_04_c33562da3df8@cloudshell:~ (qwiklabs-gcp-04-08c687827d12)$ gcloud compute security-policies rules create 10000 \
    --security-policy block-with-modsec-crs  \
    --description "allow traffic from my IP" \
    --src-ip-ranges "$MY_IP/32" \
    --action "allow"
Updated [https://www.googleapis.com/compute/v1/projects/qwiklabs-gcp-04-08c687827d12/global/securityPolicies/block-with-modsec-crs].
student_04_c33562da3df8@cloudshell:~ (qwiklabs-gcp-04-08c687827d12)$ 
```

Update the security policy to block LFI attacks.

Apply the OWASP ModSecurity Core Rule Set that prevents path traversal for local file inclusions.

```sh
student_04_c33562da3df8@cloudshell:~ (qwiklabs-gcp-04-08c687827d12)$ gcloud compute security-policies rules create 9000 \
    --security-policy block-with-modsec-crs  \
    --description "block local file inclusion" \
     --expression "evaluatePreconfiguredExpr('lfi-stable')" \
    --action deny-403
Updated [https://www.googleapis.com/compute/v1/projects/qwiklabs-gcp-04-08c687827d12/global/securityPolicies/block-with-modsec-crs].
student_04_c33562da3df8@cloudshell:~ (qwiklabs-gcp-04-08c687827d12)$ 
```

Update the security policy to block Remote Code Execution (rce).

Per the OWASP ModSecurity Core Rule Set, apply rules that look for  rce, including command injection. Typical OS commands are detected and  blocked.

```sh
student_04_c33562da3df8@cloudshell:~ (qwiklabs-gcp-04-08c687827d12)$ gcloud compute security-policies rules create 9001 \
    --security-policy block-with-modsec-crs  \
    --description "block rce attacks" \
     --expression "evaluatePreconfiguredExpr('rce-stable')" \
    --action deny-403
Updated [https://www.googleapis.com/compute/v1/projects/qwiklabs-gcp-04-08c687827d12/global/securityPolicies/block-with-modsec-crs].
student_04_c33562da3df8@cloudshell:~ (qwiklabs-gcp-04-08c687827d12)$ 
```

Update the security policy to block security scanners.

Apply the OWASP ModSecurity Core Rule Set to block well-known security scanners, scripting HTTP clients, and web crawlers.

```sh
student_04_c33562da3df8@cloudshell:~ (qwiklabs-gcp-04-08c687827d12)$ gcloud compute security-policies rules create 9002 \
    --security-policy block-with-modsec-crs  \
    --description "block scanners" \
     --expression "evaluatePreconfiguredExpr('scannerdetection-stable')" \
    --action deny-403
Updated [https://www.googleapis.com/compute/v1/projects/qwiklabs-gcp-04-08c687827d12/global/securityPolicies/block-with-modsec-crs].
student_04_c33562da3df8@cloudshell:~ (qwiklabs-gcp-04-08c687827d12)$ 
```

Update the security policy to block protocol attacks.

Per the OWASP ModSecurity Core Rule Set, apply rules that look for Carriage Return (CR) `%0d` and Linefeed (LF)`%0a` characters and other types of protocol attacks like HTTP Request Smuggling.

```sh
student_04_c33562da3df8@cloudshell:~ (qwiklabs-gcp-04-08c687827d12)$ 
student_04_c33562da3df8@cloudshell:~ (qwiklabs-gcp-04-08c687827d12)$ gcloud compute security-policies rules create 9003 \
    --security-policy block-with-modsec-crs  \
    --description "block protocol attacks" \
     --expression "evaluatePreconfiguredExpr('protocolattack-stable')" \
    --action deny-403
Updated [https://www.googleapis.com/compute/v1/projects/qwiklabs-gcp-04-08c687827d12/global/securityPolicies/block-with-modsec-crs].
student_04_c33562da3df8@cloudshell:~ (qwiklabs-gcp-04-08c687827d12)$ 
```

Update the security policy to block session fixation.

Per the OWASP ModSecurity Core Rule Set, apply the following rules using Cloud Shell:

```sh
student_04_c33562da3df8@cloudshell:~ (qwiklabs-gcp-04-08c687827d12)$ gcloud compute security-policies rules create 9004 \
    --security-policy block-with-modsec-crs \
    --description "block session fixation attacks" \
     --expression "evaluatePreconfiguredExpr('sessionfixation-stable')" \
    --action deny-403
Updated [https://www.googleapis.com/compute/v1/projects/qwiklabs-gcp-04-08c687827d12/global/securityPolicies/block-with-modsec-crs].
student_04_c33562da3df8@cloudshell:~ (qwiklabs-gcp-04-08c687827d12)$ 
```

Attach the security policy to the backend service:

```sh
student_04_c33562da3df8@cloudshell:~ (qwiklabs-gcp-04-08c687827d12)$ gcloud compute backend-services update juice-shop-backend \
    --security-policy block-with-modsec-crs \
    --global
Updated [https://www.googleapis.com/compute/v1/projects/qwiklabs-gcp-04-08c687827d12/global/backendServices/juice-shop-backend].
student_04_c33562da3df8@cloudshell:~ (qwiklabs-gcp-04-08c687827d12)$ 
```

Rules may take some time to propagate (but not more than 10 mins).

Once sufficient time has passed, test the vulnerabilities previously demonstrated to confirm Cloud Armor WAF rule enforcement in the next  step.

### Observe Cloud Armor protection with OWASP ModSecurity Core Rule Set

```sh
student_04_c33562da3df8@cloudshell:~ (qwiklabs-gcp-04-08c687827d12)$ curl -Ii http://$PUBLIC_SVC_IP/?a=../
HTTP/1.1 403 Forbidden
Content-Length: 134
Content-Type: text/html; charset=UTF-8
Date: Fri, 03 May 2024 02:44:32 GMT

student_04_c33562da3df8@cloudshell:~ (qwiklabs-gcp-04-08c687827d12)$ 
```

```sh
student_04_c33562da3df8@cloudshell:~ (qwiklabs-gcp-04-08c687827d12)$ curl -Ii http://$PUBLIC_SVC_IP/ftp?doc=/bin/ls
HTTP/1.1 403 Forbidden
Content-Length: 134
Content-Type: text/html; charset=UTF-8
Date: Fri, 03 May 2024 02:44:59 GMT

student_04_c33562da3df8@cloudshell:~ (qwiklabs-gcp-04-08c687827d12)$ 
```

```sh
student_04_c33562da3df8@cloudshell:~ (qwiklabs-gcp-04-08c687827d12)$ curl -Ii http://$PUBLIC_SVC_IP -H "User-Agent: blackwidow"
HTTP/1.1 403 Forbidden
Content-Length: 134
Content-Type: text/html; charset=UTF-8
Date: Fri, 03 May 2024 02:45:19 GMT

student_04_c33562da3df8@cloudshell:~ (qwiklabs-gcp-04-08c687827d12)$ 
```

```sh
student_04_c33562da3df8@cloudshell:~ (qwiklabs-gcp-04-08c687827d12)$ curl -Ii "http://$PUBLIC_SVC_IP/index.html?foo=advanced%0d%0aContent-Length:%200%0d%0a%0d%0aHTTP/1.1%20200%20OK%0d%0aContent-Type:%20text/html%0d%0aContent-Length:%2035%0d%0a%0d%0a<html>Sorry,%20System%20Down</html>"
HTTP/1.1 403 Forbidden
Content-Length: 134
Content-Type: text/html; charset=UTF-8
Date: Fri, 03 May 2024 02:45:41 GMT

student_04_c33562da3df8@cloudshell:~ (qwiklabs-gcp-04-08c687827d12)$ 
```

```sh
student_04_c33562da3df8@cloudshell:~ (qwiklabs-gcp-04-08c687827d12)$ curl -Ii http://$PUBLIC_SVC_IP/?session_id=a
HTTP/1.1 403 Forbidden
Content-Length: 134
Content-Type: text/html; charset=UTF-8
Date: Fri, 03 May 2024 02:45:58 GMT

student_04_c33562da3df8@cloudshell:~ (qwiklabs-gcp-04-08c687827d12)$ 
```



## Review Cloud Armor Security rules

Now that you've created the security policy, look at what rules have been configured.

Rules are evaluated by priority: lower numbers are evaluated first  and once triggered, processing does not continue for rules with higher  priority values.

- Priority `9000` - Block LFI (local file inclusion)
- Priority `9001` - Block RCE (remote code execution/command injection)
- Priority `9002` - Block Scanners Detected
- Priority `9003` - Block Protocol Attacks like HTTP splitting and HTTP smuggling
- Priority `9004` - Block Session Fixation Attacks
- Priority `10000` - Allow your IP to access the Website
- Priority `Default` - Deny.

Notice the "allow your IP" rule is configured with the highest priority  number to allow access to the site, however blocks any attack.

## Observe Cloud Armor security policy logs

From the Cloud Armor console page, view details of the security  policy and click the Logs tab followed by the View policy logs link to  be directed to the Cloud Logging page. It automatically filters based on the security policy of interest, for example,  resource.type:(http_load_balancer) AND  jsonPayload.enforcedSecurityPolicy.name:. Observe the 403 error response codes and expand the log details to  observe the enforced security policy's name, matched field value, and  further down the preconfigured expression IDs (or the signature id).

It automatically filters based on the security policy of interest,  for example, resource.type:(http_load_balancer) AND  jsonPayload.enforcedSecurityPolicy.name:().

Observe the 403 error response codes and expand the log details to  observe the enforced security policy's name, matched field value, and  further down the preconfigured expression IDs (or the signature id).

```json
{
  "insertId": "13yrc2aevoos5",
  "jsonPayload": {
    "@type": "type.googleapis.com/google.cloud.loadbalancing.type.LoadBalancerLogEntry",
    "backendTargetProjectNumber": "projects/15934676962",
    "statusDetails": "denied_by_security_policy",
    "enforcedSecurityPolicy": {
      "name": "block-with-modsec-crs",
      "outcome": "DENY",
      "preconfiguredExprIds": [
        "owasp-crs-v030001-id930110-lfi"
      ],
      "priority": 9000,
      "configuredAction": "DENY"
    },
    "remoteIp": "34.142.184.163",
    "cacheDecision": [
      "RESPONSE_HAS_CONTENT_TYPE",
      "CACHE_MODE_USE_ORIGIN_HEADERS"
    ],
    "securityPolicyRequestData": {
      "remoteIpInfo": {
        "regionCode": "SG"
      }
    }
  },
  "httpRequest": {
    "requestMethod": "HEAD",
    "requestUrl": "http://34.144.205.142/?a=../",
    "requestSize": "85",
    "status": 403,
    "responseSize": "124",
    "userAgent": "curl/7.81.0",
    "remoteIp": "34.142.184.163",
    "latency": "0.256205s"
  },
  "resource": {
    "type": "http_load_balancer",
    "labels": {
      "zone": "global",
      "target_proxy_name": "juice-shop-proxy",
      "backend_service_name": "juice-shop-backend",
      "forwarding_rule_name": "juice-shop-rule",
      "project_id": "qwiklabs-gcp-04-08c687827d12",
      "url_map_name": "juice-shop-loadbalancer"
    }
  },
  "timestamp": "2024-05-03T02:44:32.097944Z",
  "severity": "WARNING",
  "logName": "projects/qwiklabs-gcp-04-08c687827d12/logs/requests",
  "trace": "projects/qwiklabs-gcp-04-08c687827d12/traces/dc20f5f2ccdda3b2e9e76ed388cf250c",
  "receiveTimestamp": "2024-05-03T02:44:33.228750741Z",
  "spanId": "c7ea50b7b7f4fdab"
}
```



```json
{
  "insertId": "9pz2diexq9ek",
  "jsonPayload": {
    "enforcedSecurityPolicy": {
      "configuredAction": "DENY",
      "priority": 9001,
      "outcome": "DENY",
      "name": "block-with-modsec-crs",
      "preconfiguredExprIds": [
        "owasp-crs-v030001-id932160-rce"
      ]
    },
    "remoteIp": "34.142.184.163",
    "securityPolicyRequestData": {
      "remoteIpInfo": {
        "regionCode": "SG"
      }
    },
    "backendTargetProjectNumber": "projects/15934676962",
    "statusDetails": "denied_by_security_policy",
    "@type": "type.googleapis.com/google.cloud.loadbalancing.type.LoadBalancerLogEntry",
    "cacheDecision": [
      "RESPONSE_HAS_CONTENT_TYPE",
      "CACHE_MODE_USE_ORIGIN_HEADERS"
    ]
  },
  "httpRequest": {
    "requestMethod": "HEAD",
    "requestUrl": "http://34.144.205.142/ftp?doc=/bin/ls",
    "requestSize": "94",
    "status": 403,
    "responseSize": "124",
    "userAgent": "curl/7.81.0",
    "remoteIp": "34.142.184.163",
    "latency": "0.257092s"
  },
  "resource": {
    "type": "http_load_balancer",
    "labels": {
      "forwarding_rule_name": "juice-shop-rule",
      "backend_service_name": "juice-shop-backend",
      "url_map_name": "juice-shop-loadbalancer",
      "target_proxy_name": "juice-shop-proxy",
      "zone": "global",
      "project_id": "qwiklabs-gcp-04-08c687827d12"
    }
  },
  "timestamp": "2024-05-03T02:44:59.294433Z",
  "severity": "WARNING",
  "logName": "projects/qwiklabs-gcp-04-08c687827d12/logs/requests",
  "trace": "projects/qwiklabs-gcp-04-08c687827d12/traces/f1b7ff50de33b0139ca2ffd6877c535b",
  "receiveTimestamp": "2024-05-03T02:45:00.034915124Z",
  "spanId": "93ab2bf259e98c4f"
}
```



```json
{
  "insertId": "6xsitvekytxg",
  "jsonPayload": {
    "enforcedSecurityPolicy": {
      "priority": 9002,
      "outcome": "DENY",
      "preconfiguredExprIds": [
        "owasp-crs-v030001-id913102-scannerdetection"
      ],
      "name": "block-with-modsec-crs",
      "configuredAction": "DENY"
    },
    "securityPolicyRequestData": {
      "remoteIpInfo": {
        "regionCode": "SG"
      }
    },
    "cacheDecision": [
      "RESPONSE_HAS_CONTENT_TYPE",
      "CACHE_MODE_USE_ORIGIN_HEADERS"
    ],
    "@type": "type.googleapis.com/google.cloud.loadbalancing.type.LoadBalancerLogEntry",
    "remoteIp": "34.142.184.163",
    "backendTargetProjectNumber": "projects/15934676962",
    "statusDetails": "denied_by_security_policy"
  },
  "httpRequest": {
    "requestMethod": "HEAD",
    "requestUrl": "http://34.144.205.142/",
    "requestSize": "78",
    "status": 403,
    "responseSize": "124",
    "userAgent": "blackwidow",
    "remoteIp": "34.142.184.163",
    "latency": "0.258696s"
  },
  "resource": {
    "type": "http_load_balancer",
    "labels": {
      "url_map_name": "juice-shop-loadbalancer",
      "backend_service_name": "juice-shop-backend",
      "zone": "global",
      "target_proxy_name": "juice-shop-proxy",
      "project_id": "qwiklabs-gcp-04-08c687827d12",
      "forwarding_rule_name": "juice-shop-rule"
    }
  },
  "timestamp": "2024-05-03T02:45:19.057792Z",
  "severity": "WARNING",
  "logName": "projects/qwiklabs-gcp-04-08c687827d12/logs/requests",
  "trace": "projects/qwiklabs-gcp-04-08c687827d12/traces/596a35d887b36dfb6957f030b88bc99a",
  "receiveTimestamp": "2024-05-03T02:45:20.230037137Z",
  "spanId": "1b3065dcf460e0e8"
}
```

```json
{
  "insertId": "12aqxsdemouik",
  "jsonPayload": {
    "@type": "type.googleapis.com/google.cloud.loadbalancing.type.LoadBalancerLogEntry",
    "statusDetails": "denied_by_security_policy",
    "securityPolicyRequestData": {
      "remoteIpInfo": {
        "regionCode": "SG"
      }
    },
    "remoteIp": "34.142.184.163",
    "cacheDecision": [
      "RESPONSE_HAS_CONTENT_TYPE",
      "CACHE_MODE_USE_ORIGIN_HEADERS"
    ],
    "enforcedSecurityPolicy": {
      "configuredAction": "DENY",
      "priority": 9004,
      "preconfiguredExprIds": [
        "owasp-crs-v030001-id943120-sessionfixation"
      ],
      "name": "block-with-modsec-crs",
      "outcome": "DENY"
    },
    "backendTargetProjectNumber": "projects/15934676962"
  },
  "httpRequest": {
    "requestMethod": "HEAD",
    "requestUrl": "http://34.144.205.142/?session_id=a",
    "requestSize": "92",
    "status": 403,
    "responseSize": "124",
    "userAgent": "curl/7.81.0",
    "remoteIp": "34.142.184.163",
    "latency": "0.257400s"
  },
  "resource": {
    "type": "http_load_balancer",
    "labels": {
      "forwarding_rule_name": "juice-shop-rule",
      "zone": "global",
      "backend_service_name": "juice-shop-backend",
      "url_map_name": "juice-shop-loadbalancer",
      "target_proxy_name": "juice-shop-proxy",
      "project_id": "qwiklabs-gcp-04-08c687827d12"
    }
  },
  "timestamp": "2024-05-03T02:45:58.497351Z",
  "severity": "WARNING",
  "logName": "projects/qwiklabs-gcp-04-08c687827d12/logs/requests",
  "trace": "projects/qwiklabs-gcp-04-08c687827d12/traces/bb9a8db7369067fb0018512759f70275",
  "receiveTimestamp": "2024-05-03T02:45:59.434504639Z",
  "spanId": "d568166c5baaede8"
}
```



You've successfully mitigated some of the common vulnerabilities by using Google Cloud Armor WAF rules.