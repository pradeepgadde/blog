---

layout: single
title:  "Getting Started with Cloud Shell and gcloud"
date:   2023-03-29 09:59:04 +0530
categories: Cloud
tags: GCP
classes: wide
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

# Getting Started with Cloud Shell and gcloud

Cloud Shell is a Debian-based virtual machine with a persistent 5-GB  home directory, which makes it easy for you to manage your Google Cloud  projects and resources.

- gcloud auth list



- gcloud config list project



- gcloud config set compute/region 



- gcloud config get-value compute/region



- gcloud config set compute/zone 



- gcloud config get-value compute/zone



- gcloud config get-value project



- gcloud compute project-info describe --project $(gcloud config get-value project)



- export PROJECT_ID=$(gcloud config get-value project)



- export ZONE=$(gcloud config get-value compute/zone)



- echo -e "PROJECT ID: $PROJECT_ID\nZONE: $ZONE"



- gcloud compute instances create gcelab2 --machine-type e2-medium --zone $ZONE



- gcloud compute instances create --help



- gcloud -h



- gcloud config --help



- gcloud help config



- gcloud config list



- gcloud config list --all



- gcloud components list



- gcloud compute instances list



- gcloud compute instances list --filter="name=('gcelab2')"



- gcloud compute firewall-rules list



- gcloud compute firewall-rules list --filter="network='default'"



- gcloud compute firewall-rules list --filter="NETWORK:'default' AND ALLOW:'icmp'"



- gcloud compute ssh gcelab2 --zone $ZONE



- gcloud compute instances add-tags gcelab2 --tags http-server,https-server



- gcloud compute firewall-rules create default-allow-http --direction=INGRESS --priority=1000 --network=default --action=ALLOW --rules=tcp:80 --source-ranges=0.0.0.0/0 --target-tags=http-server



- gcloud compute firewall-rules list --filter=ALLOW:'80'



- curl http://$(gcloud compute instances list --filter=name:gcelab2 --format='value(EXTERNAL_IP)')



- gcloud logging logs list



- gcloud logging logs list --filter="compute"



- gcloud logging read "resource.type=gce_instance" --limit 5



- gcloud logging read "resource.type=gce_instance AND labels.instance_name='gcelab2'" --limit 5



```sh

Welcome to Cloud Shell! Type "help" to get started.
Your Cloud Platform project in this session is set to qwiklabs-gcp-01-823ec07351c5.
Use “gcloud config set project [PROJECT_ID]” to change to a different project.
student_01_c4faa281a7ed@cloudshell:~ (qwiklabs-gcp-01-823ec07351c5)$ gcloud auth list
Credentialed Accounts

ACTIVE: *
ACCOUNT: student-01-c4faa281a7ed@qwiklabs.net

To set the active account, run:
    $ gcloud config set account `ACCOUNT`

student_01_c4faa281a7ed@cloudshell:~ (qwiklabs-gcp-01-823ec07351c5)$ gcloud config list project
[core]
project = qwiklabs-gcp-01-823ec07351c5

Your active configuration is: [cloudshell-20105]
student_01_c4faa281a7ed@cloudshell:~ (qwiklabs-gcp-01-823ec07351c5)$ gcloud config set compute/region us-central1
Updated property [compute/region].
student_01_c4faa281a7ed@cloudshell:~ (qwiklabs-gcp-01-823ec07351c5)$ gcloud config get-value compute/region
Your active configuration is: [cloudshell-20105]
us-central1
student_01_c4faa281a7ed@cloudshell:~ (qwiklabs-gcp-01-823ec07351c5)$ gcloud config set compute/zone us-central1-b
Updated property [compute/zone].
student_01_c4faa281a7ed@cloudshell:~ (qwiklabs-gcp-01-823ec07351c5)$ gcloud config get-value compute/zone
Your active configuration is: [cloudshell-20105]
us-central1-b
student_01_c4faa281a7ed@cloudshell:~ (qwiklabs-gcp-01-823ec07351c5)$ gcloud config get-value project
Your active configuration is: [cloudshell-20105]
qwiklabs-gcp-01-823ec07351c5
student_01_c4faa281a7ed@cloudshell:~ (qwiklabs-gcp-01-823ec07351c5)$ gcloud compute project-info describe --project $(gcloud config get-value project)
Your active configuration is: [cloudshell-20105]
commonInstanceMetadata:
  fingerprint: izYsTiqWDBk=
  items:
  - key: google-compute-default-zone
    value: us-central1-b
  - key: google-compute-default-region
    value: us-central1
  - key: ssh-keys
    value: student-01-c4faa281a7ed:ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAABAQChGzae2clruxeLpMQa9nYtUC6tOtj8BKoUWh93VJ+wfrCkwyxRMcDgRLpql4gbDVgiJRRohUDzsqIxgZFVbUs0wESO27WqpQ55g4+mpP0egoM1mZSftmhbkqfO4EIAkNCSB6rkeUOWgFJPN8KsW4Ude+iVPErMNdo8V+qK+ikwlyx4MsyzXWKHGooLaQPxeSbt8xjiHPxOnr32BQ9CbViXsXgpv6n6K0t8THZb7mJ7OuahnTfaLjJsP2dlqhWDMopo+e9sn3mPcZJIEMi0w8euQz/PYPaUzop1pMgpM4xcZfNzEi62Kt8JjGDGO2vwC1yH5Z2mjZXg0KFE9PNUVm5T
      student-01-c4faa281a7ed@qwiklabs.net
  - key: enable-oslogin
    value: 'true'
  kind: compute#metadata
creationTimestamp: '2023-03-28T02:24:45.710-07:00'
defaultNetworkTier: PREMIUM
defaultServiceAccount: 213639941430-compute@developer.gserviceaccount.com
id: '3357936118112652466'
kind: compute#project
name: qwiklabs-gcp-01-823ec07351c5
quotas:
- limit: 1000.0
  metric: SNAPSHOTS
  usage: 0.0
- limit: 5.0
  metric: NETWORKS
  usage: 1.0
- limit: 100.0
  metric: FIREWALLS
  usage: 4.0
- limit: 100.0
  metric: IMAGES
  usage: 0.0
- limit: 8.0
  metric: STATIC_ADDRESSES
  usage: 0.0
- limit: 200.0
  metric: ROUTES
  usage: 1.0
- limit: 15.0
  metric: FORWARDING_RULES
  usage: 0.0
- limit: 50.0
  metric: TARGET_POOLS
  usage: 0.0
- limit: 50.0
  metric: HEALTH_CHECKS
  usage: 0.0
- limit: 4.0
  metric: IN_USE_ADDRESSES
  usage: 0.0
- limit: 50.0
  metric: TARGET_INSTANCES
  usage: 0.0
- limit: 10.0
  metric: TARGET_HTTP_PROXIES
  usage: 0.0
- limit: 10.0
  metric: URL_MAPS
  usage: 0.0
- limit: 50.0
  metric: BACKEND_SERVICES
  usage: 0.0
- limit: 100.0
  metric: INSTANCE_TEMPLATES
  usage: 0.0
- limit: 5.0
  metric: TARGET_VPN_GATEWAYS
  usage: 0.0
- limit: 10.0
  metric: VPN_TUNNELS
  usage: 0.0
- limit: 3.0
  metric: BACKEND_BUCKETS
  usage: 0.0
- limit: 10.0
  metric: ROUTERS
  usage: 0.0
- limit: 10.0
  metric: TARGET_SSL_PROXIES
  usage: 0.0
- limit: 10.0
  metric: TARGET_HTTPS_PROXIES
  usage: 0.0
- limit: 10.0
  metric: SSL_CERTIFICATES
  usage: 0.0
- limit: 100.0
  metric: SUBNETWORKS
  usage: 0.0
- limit: 10.0
  metric: TARGET_TCP_PROXIES
  usage: 0.0
- limit: 12.0
  metric: CPUS_ALL_REGIONS
  usage: 0.0
- limit: 0.0
  metric: SECURITY_POLICIES
  usage: 0.0
- limit: 0.0
  metric: SECURITY_POLICY_RULES
  usage: 0.0
- limit: 1000.0
  metric: XPN_SERVICE_PROJECTS
  usage: 0.0
- limit: 20.0
  metric: PACKET_MIRRORINGS
  usage: 0.0
- limit: 100.0
  metric: NETWORK_ENDPOINT_GROUPS
  usage: 0.0
- limit: 6.0
  metric: INTERCONNECTS
  usage: 0.0
- limit: 5000.0
  metric: GLOBAL_INTERNAL_ADDRESSES
  usage: 0.0
- limit: 5.0
  metric: VPN_GATEWAYS
  usage: 0.0
- limit: 100.0
  metric: MACHINE_IMAGES
  usage: 0.0
- limit: 0.0
  metric: SECURITY_POLICY_CEVAL_RULES
  usage: 0.0
- limit: 0.0
  metric: GPUS_ALL_REGIONS
  usage: 0.0
- limit: 5.0
  metric: EXTERNAL_VPN_GATEWAYS
  usage: 0.0
- limit: 1.0
  metric: PUBLIC_ADVERTISED_PREFIXES
  usage: 0.0
- limit: 10.0
  metric: PUBLIC_DELEGATED_PREFIXES
  usage: 0.0
- limit: 128.0
  metric: STATIC_BYOIP_ADDRESSES
  usage: 0.0
- limit: 10.0
  metric: NETWORK_FIREWALL_POLICIES
  usage: 0.0
- limit: 15.0
  metric: INTERNAL_TRAFFIC_DIRECTOR_FORWARDING_RULES
  usage: 0.0
- limit: 15.0
  metric: GLOBAL_EXTERNAL_MANAGED_FORWARDING_RULES
  usage: 0.0
- limit: 50.0
  metric: GLOBAL_EXTERNAL_MANAGED_BACKEND_SERVICES
  usage: 0.0
- limit: 50.0
  metric: GLOBAL_EXTERNAL_PROXY_LB_BACKEND_SERVICES
  usage: 0.0
- limit: 100.0
  metric: GLOBAL_INTERNAL_TRAFFIC_DIRECTOR_BACKEND_SERVICES
  usage: 0.0
selfLink: https://www.googleapis.com/compute/v1/projects/qwiklabs-gcp-01-823ec07351c5
vmDnsSetting: ZONAL_ONLY
xpnProjectStatus: UNSPECIFIED_XPN_PROJECT_STATUS
student_01_c4faa281a7ed@cloudshell:~ (qwiklabs-gcp-01-823ec07351c5)$ export PROJECT_ID=$(gcloud config get-value project)
Your active configuration is: [cloudshell-20105]
student_01_c4faa281a7ed@cloudshell:~ (qwiklabs-gcp-01-823ec07351c5)$ export ZONE=$(gcloud config get-value compute/zone)
Your active configuration is: [cloudshell-20105]
student_01_c4faa281a7ed@cloudshell:~ (qwiklabs-gcp-01-823ec07351c5)$ echo -e "PROJECT ID: $PROJECT_ID\nZONE: $ZONE"
PROJECT ID: qwiklabs-gcp-01-823ec07351c5
ZONE: us-central1-b
```



```sh
student_01_c4faa281a7ed@cloudshell:~ (qwiklabs-gcp-01-823ec07351c5)$ gcloud compute instances create gcelab2 --machine-type e2-medium --zone $ZONE
Created [https://www.googleapis.com/compute/v1/projects/qwiklabs-gcp-01-823ec07351c5/zones/us-central1-b/instances/gcelab2].
NAME: gcelab2
ZONE: us-central1-b
MACHINE_TYPE: e2-medium
PREEMPTIBLE:
INTERNAL_IP: 10.128.0.2
EXTERNAL_IP: 34.136.108.219
STATUS: RUNNING
student_01_c4faa281a7ed@cloudshell:~ (qwiklabs-gcp-01-823ec07351c5)$
```



```sh
student_01_c4faa281a7ed@cloudshell:~ (qwiklabs-gcp-01-823ec07351c5)$ gcloud -h
Usage: gcloud [optional flags] <group | command>
  group may be           access-approval | access-context-manager |
                         active-directory | ai | ai-platform | alloydb | alpha |
                         anthos | api-gateway | apigee | app | artifacts |
                         asset | assured | auth | batch | beta | bigtable |
                         billing | bms | builds | certificate-manager |
                         cloud-shell | components | composer | compute |
                         config | container | data-catalog |
                         database-migration | dataflow | dataplex | dataproc |
                         datastore | datastream | debug | deploy |
                         deployment-manager | dns | domains | edge-cache |
                         edge-cloud | emulators | endpoints |
                         essential-contacts | eventarc | filestore | firebase |
                         firestore | functions | game | healthcare | iam | iap |
                         identity | ids | immersive-stream | iot | kms |
                         logging | memcache | metastore | ml | ml-engine |
                         monitoring | network-connectivity |
                         network-management | network-security |
                         network-services | notebooks | org-policies |
                         organizations | policy-intelligence |
                         policy-troubleshoot | privateca | projects | pubsub |
                         recaptcha | recommender | redis | resource-manager |
                         resource-settings | run | scc | scheduler | secrets |
                         service-directory | services | source | spanner | sql |
                         storage | tasks | topic | transcoder | transfer |
                         vmware | workflows | workspace-add-ons
  command may be         cheat-sheet | docker | feedback | help | info | init |
                         survey | version

For detailed information on this command and its flags, run:
  gcloud --help
student_01_c4faa281a7ed@cloudshell:~ (qwiklabs-gcp-01-823ec07351c5)$
```



```sh
student_01_c4faa281a7ed@cloudshell:~ (qwiklabs-gcp-01-823ec07351c5)$ gcloud config list
[accessibility]
screen_reader = True
[component_manager]
disable_update_check = True
[compute]
gce_metadata_read_timeout_sec = 30
region = us-central1
zone = us-central1-b
[core]
account = student-01-c4faa281a7ed@qwiklabs.net
disable_usage_reporting = True
project = qwiklabs-gcp-01-823ec07351c5
[metrics]
environment = devshell

Your active configuration is: [cloudshell-20105]
student_01_c4faa281a7ed@cloudshell:~ (qwiklabs-gcp-01-823ec07351c5)$
```

```sh
student_01_c4faa281a7ed@cloudshell:~ (qwiklabs-gcp-01-823ec07351c5)$ gcloud config list --all
[accessibility]
screen_reader = True
[ai]
region (unset)
[ai_platform]
region (unset)
[api_endpoint_overrides]
accessapproval (unset)
accesscontextmanager (unset)
ai (unset)
aiplatform (unset)
anthosevents (unset)
anthospolicycontrollerstatus_pa (unset)
apigateway (unset)
apigee (unset)
appengine (unset)
artifactregistry (unset)
assuredworkloads (unset)
baremetalsolution (unset)
bigtableadmin (unset)
certificatemanager (unset)
cloudasset (unset)
cloudbilling (unset)
cloudbuild (unset)
cloudcommerceconsumerprocurement (unset)
clouddebugger (unset)
clouddeploy (unset)
clouderrorreporting (unset)
cloudfunctions (unset)
cloudidentity (unset)
cloudiot (unset)
cloudkms (unset)
cloudresourcemanager (unset)
cloudscheduler (unset)
cloudtasks (unset)
cloudtrace (unset)
composer (unset)
compute (unset)
config (unset)
container (unset)
datacatalog (unset)
dataflow (unset)
datafusion (unset)
datamigration (unset)
datapipelines (unset)
dataplex (unset)
dataproc (unset)
datastore (unset)
datastream (unset)
deploymentmanager (unset)
dns (unset)
domains (unset)
edgecontainer (unset)
eventarc (unset)
eventarcpublishing (unset)
events (unset)
file (unset)
firestore (unset)
gameservices (unset)
genomics (unset)
gkemulticloud (unset)
healthcare (unset)
iam (unset)
iap (unset)
ids (unset)
krmapihosting (unset)
language (unset)
lifesciences (unset)
logging (unset)
looker (unset)
managedidentities (unset)
marketplacesolutions (unset)
mediaasset (unset)
memcache (unset)
metastore (unset)
monitoring (unset)
netapp (unset)
networkconnectivity (unset)
networkmanagement (unset)
networksecurity (unset)
networkservices (unset)
notebooks (unset)
orgpolicy (unset)
policyanalyzer (unset)
privateca (unset)
publicca (unset)
pubsub (unset)
recaptchaenterprise (unset)
recommender (unset)
redis (unset)
resourcesettings (unset)
run (unset)
runtimeconfig (unset)
sddc (unset)
secretmanager (unset)
securitycenter (unset)
servicedirectory (unset)
servicemanagement (unset)
sourcerepo (unset)
spanner (unset)
speech (unset)
sql (unset)
storage (unset)
testing (unset)
vision (unset)
vmwareengine (unset)
workflowexecutions (unset)
workflows (unset)
[app]
cloud_build_timeout (unset)
promote_by_default (unset)
stop_previous_version (unset)
use_runtime_builders (unset)
[artifacts]
location (unset)
repository (unset)
[auth]
access_token_file (unset)
disable_credentials (unset)
impersonate_service_account (unset)
login_config_file (unset)
service_account_disable_id_token_refresh (unset)
service_account_use_self_signed_jwt (unset)
token_host (unset)
[batch]
location (unset)
[billing]
quota_project (unset)
[builds]
kaniko_cache_ttl (unset)
region (unset)
timeout (unset)
use_kaniko (unset)
[component_manager]
additional_repositories (unset)
disable_update_check = True
[composer]
location (unset)
[compute]
gce_metadata_read_timeout_sec = 30
image_family_scope (unset)
region = us-central1
use_new_list_usable_subnets_api (unset)
zone = us-central1-b
[container]
build_timeout (unset)
cluster (unset)
use_application_default_credentials (unset)
use_client_certificate (unset)
[container_attached]
location (unset)
[container_aws]
location (unset)
[container_azure]
location (unset)
[container_bare_metal]
location (unset)
[container_vmware]
location (unset)
[context_aware]
use_client_certificate (unset)
[core]
account = student-01-c4faa281a7ed@qwiklabs.net
console_log_format (unset)
custom_ca_certs_file (unset)
default_format (unset)
default_regional_backend_service (unset)
disable_color (unset)
disable_file_logging (unset)
disable_prompts (unset)
disable_usage_reporting = True
enable_feature_flags (unset)
format (unset)
log_http (unset)
max_log_days (unset)
parse_error_details (unset)
pass_credentials_to_gsutil (unset)
project = qwiklabs-gcp-01-823ec07351c5
show_structured_logs (unset)
trace_token (unset)
user_output_enabled (unset)
verbosity (unset)
[dataflow]
disable_public_ips (unset)
enable_streaming_engine (unset)
print_only (unset)
[datafusion]
location (unset)
[datapipelines]
disable_public_ips (unset)
enable_streaming_engine (unset)
[dataplex]
asset (unset)
lake (unset)
location (unset)
zone (unset)
[dataproc]
location (unset)
region (unset)
[deploy]
delivery_pipeline (unset)
region (unset)
[deployment_manager]
glob_imports (unset)
[eventarc]
location (unset)
[filestore]
location (unset)
region (unset)
zone (unset)
[functions]
gen2 (unset)
region (unset)
[game_services]
default_deployment (unset)
default_realm (unset)
location (unset)
[gcloudignore]
enabled (unset)
[gkebackup]
backup (unset)
backup_plan (unset)
location (unset)
restore (unset)
restore_plan (unset)
[healthcare]
dataset (unset)
location (unset)
[interactive]
bottom_bindings_line (unset)
bottom_status_line (unset)
completion_menu_lines (unset)
context (unset)
fixed_prompt_position (unset)
help_lines (unset)
hidden (unset)
justify_bottom_lines (unset)
manpage_generator (unset)
multi_column_completion_menu (unset)
prompt (unset)
show_help (unset)
suggest (unset)
[lifesciences]
location (unset)
[looker]
region (unset)
[media_asset]
location (unset)
[memcache]
region (unset)
[metastore]
location (unset)
tier (unset)
[metrics]
environment = devshell
[ml_engine]
local_python (unset)
polling_interval (unset)
[mps]
vendor (unset)
[netapp]
location (unset)
region (unset)
[notebooks]
location (unset)
[privateca]
location (unset)
[proxy]
address (unset)
password (unset)
port (unset)
rdns (unset)
type (unset)
username (unset)
[redis]
region (unset)
[run]
cluster (unset)
cluster_location (unset)
platform (unset)
region (unset)
[scc]
organization (unset)
parent (unset)
[secrets]
locations (unset)
replication-policy (unset)
[spanner]
instance (unset)
[ssh]
putty_force_connect (unset)
verify_internal_ip (unset)
[storage]
additional_headers (unset)
base_retry_delay (unset)
check_hashes (unset)
check_mv_early_deletion_fee (unset)
convert_incompatible_windows_path_characters (unset)
copy_chunk_size (unset)
download_chunk_size (unset)
exponential_sleep_multiplier (unset)
key_store_path (unset)
max_retries (unset)
max_retry_delay (unset)
parallel_composite_upload_compatibility_check (unset)
parallel_composite_upload_component_size (unset)
parallel_composite_upload_enabled (unset)
parallel_composite_upload_threshold (unset)
process_count (unset)
resumable_threshold (unset)
rsync_files_directory (unset)
s3_endpoint_url (unset)
sliced_object_download_component_size (unset)
sliced_object_download_max_components (unset)
sliced_object_download_threshold (unset)
suggest_transfer (unset)
thread_count (unset)
tracker_files_directory (unset)
upload_chunk_size (unset)
use_gcloud_crc32c (unset)
use_gsutil (unset)
use_magicfile (unset)
use_threading_local (unset)
[survey]
disable_prompts (unset)
[vmware]
region (unset)

Your active configuration is: [cloudshell-20105]
student_01_c4faa281a7ed@cloudshell:~ (qwiklabs-gcp-01-823ec07351c5)$
```



```sh
tudent_01_c4faa281a7ed@cloudshell:~ (qwiklabs-gcp-01-823ec07351c5)$ gcloud components list

Your current Google Cloud CLI version is: 423.0.0
The latest available version is: 424.0.0

Components

Status: Update Available
Name: BigQuery Command Line Tool
ID: bq
Size: 1.6 MiB

Status: Update Available
Name: Google Cloud CLI Core Libraries
ID: core
Size: 20.1 MiB

Status: Update Available
Name: gcloud Alpha Commands
ID: alpha
Size: < 1 MiB

Status: Update Available
Name: gcloud Beta Commands
ID: beta
Size: < 1 MiB

Status: Not Installed
Name: Appctl
ID: appctl
Size: 21.0 MiB

Status: Not Installed
Name: Artifact Registry Go Module Package Helper
ID: package-go-module
Size: < 1 MiB

Status: Not Installed
Name: Cloud Firestore Emulator
ID: cloud-firestore-emulator
Size: 41.6 MiB

Status: Not Installed
Name: Cloud SQL Proxy
ID: cloud_sql_proxy
Size: 7.8 MiB

Status: Not Installed
Name: Cloud Spanner Emulator
ID: cloud-spanner-emulator
Size: 28.7 MiB

Status: Not Installed
Name: Cloud Spanner Migration Tool
ID: harbourbridge
Size: 22.3 MiB

Status: Not Installed
Name: Google Container Registry's Docker credential helper
ID: docker-credential-gcr
Size: 1.8 MiB

Status: Not Installed
Name: Kustomize
ID: kustomize
Size: 4.3 MiB

Status: Not Installed
Name: Log Streaming
ID: log-streaming
Size: 13.9 MiB

Status: Not Installed
Name: Nomos CLI
ID: nomos
Size: 25.2 MiB

Status: Not Installed
Name: Terraform Tools
ID: terraform-tools
Size: 61.7 MiB

Status: Not Installed
Name: anthos-auth
ID: anthos-auth
Size: 20.4 MiB

Status: Not Installed
Name: config-connector
ID: config-connector
Size: 56.7 MiB

Status: Not Installed
Name: enterprise-certificate-proxy
ID: enterprise-certificate-proxy
Size: 6.6 MiB

Status: Not Installed
Name: kubectl
ID: kubectl
Size: < 1 MiB

Status: Not Installed
Name: kubectl-oidc
ID: kubectl-oidc
Size: 20.4 MiB

Status: Not Installed
Name: pkg
ID: pkg
Size:

Status: Installed
Name: App Engine Go Extensions
ID: app-engine-go
Size: 4.5 MiB

Status: Installed
Name: Bundled Python 3.9
ID: bundled-python3-unix
Size: 63.4 MiB

Status: Installed
Name: Cloud Bigtable Command Line Tool
ID: cbt
Size: 10.4 MiB

Status: Installed
Name: Cloud Bigtable Emulator
ID: bigtable
Size: 6.7 MiB

Status: Installed
Name: Cloud Datastore Emulator
ID: cloud-datastore-emulator
Size: 35.1 MiB

Status: Installed
Name: Cloud Pub/Sub Emulator
ID: pubsub-emulator
Size: 66.4 MiB

Status: Installed
Name: Cloud Run Proxy
ID: cloud-run-proxy
Size: 9.0 MiB

Status: Installed
Name: Cloud Storage Command Line Tool
ID: gsutil
Size: 15.5 MiB

Status: Installed
Name: Google Cloud CRC32C Hash Tool
ID: gcloud-crc32c
Size: 1.2 MiB

Status: Installed
Name: Minikube
ID: minikube
Size: 33.1 MiB

Status: Installed
Name: On-Demand Scanning API extraction helper
ID: local-extract
Size: 13.9 MiB

Status: Installed
Name: Skaffold
ID: skaffold
Size: 22.0 MiB

Status: Installed
Name: gcloud app Java Extensions
ID: app-engine-java
Size: 64.6 MiB

Status: Installed
Name: gcloud app Python Extensions
ID: app-engine-python
Size: 8.4 MiB

Status: Installed
Name: gcloud app Python Extensions (Extra Libraries)
ID: app-engine-python-extras
Size: 26.4 MiB

Status: Installed
Name: gke-gcloud-auth-plugin
ID: gke-gcloud-auth-plugin
Size: 7.7 MiB

Status: Installed
Name: kpt
ID: kpt
Size: 19.4 MiB
To install or remove components at your current SDK version [423.0.0], run:
  $ gcloud components install COMPONENT_ID
  $ gcloud components remove COMPONENT_ID

To update your SDK installation to the latest version [424.0.0], run:
  $ gcloud components update

student_01_c4faa281a7ed@cloudshell:~ (qwiklabs-gcp-01-823ec07351c5)$
```



```sh
student_01_c4faa281a7ed@cloudshell:~ (qwiklabs-gcp-01-823ec07351c5)$ gcloud compute instances list
NAME: gcelab2
ZONE: us-central1-b
MACHINE_TYPE: e2-medium
PREEMPTIBLE:
INTERNAL_IP: 10.128.0.2
EXTERNAL_IP: 34.136.108.219
STATUS: RUNNING
student_01_c4faa281a7ed@cloudshell:~ (qwiklabs-gcp-01-823ec07351c5)$
```



```sh
student_01_c4faa281a7ed@cloudshell:~ (qwiklabs-gcp-01-823ec07351c5)$ gcloud compute instances list --filter="name=('gcelab2')"
NAME: gcelab2
ZONE: us-central1-b
MACHINE_TYPE: e2-medium
PREEMPTIBLE:
INTERNAL_IP: 10.128.0.2
EXTERNAL_IP: 34.136.108.219
STATUS: RUNNING
student_01_c4faa281a7ed@cloudshell:~ (qwiklabs-gcp-01-823ec07351c5)$
```



```sh
student_01_c4faa281a7ed@cloudshell:~ (qwiklabs-gcp-01-823ec07351c5)$ gcloud compute firewall-rules list
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

To show all fields of the firewall, please show in JSON format: --format=json
To show all fields in table format, please see the examples in --help.

student_01_c4faa281a7ed@cloudshell:~ (qwiklabs-gcp-01-823ec07351c5)$
```

```sh
student_01_c4faa281a7ed@cloudshell:~ (qwiklabs-gcp-01-823ec07351c5)$ gcloud compute firewall-rules list --filter="network='default'"
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

To show all fields of the firewall, please show in JSON format: --format=json
To show all fields in table format, please see the examples in --help.

student_01_c4faa281a7ed@cloudshell:~ (qwiklabs-gcp-01-823ec07351c5)$
```

```sh
student_01_c4faa281a7ed@cloudshell:~ (qwiklabs-gcp-01-823ec07351c5)$ gcloud compute firewall-rules list --filter="NETWORK:'default' AND ALLOW:'icmp'"
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

To show all fields of the firewall, please show in JSON format: --format=json
To show all fields in table format, please see the examples in --help.

student_01_c4faa281a7ed@cloudshell:~ (qwiklabs-gcp-01-823ec07351c5)$
```



```sh

student_01_c4faa281a7ed@cloudshell:~ (qwiklabs-gcp-01-823ec07351c5)$ gcloud compute ssh gcelab2 --zone $ZONE
WARNING: The private SSH key file for gcloud does not exist.
WARNING: The public SSH key file for gcloud does not exist.
WARNING: You do not have an SSH key for gcloud.
WARNING: SSH keygen will be executed to generate a key.
This tool needs to create the directory [/home/student_01_c4faa281a7ed/.ssh] before being able to generate SSH keys.

Do you want to continue (Y/n)?  Y

Generating public/private rsa key pair.
Enter passphrase (empty for no passphrase):
Enter same passphrase again:
Your identification has been saved in /home/student_01_c4faa281a7ed/.ssh/google_compute_engine
Your public key has been saved in /home/student_01_c4faa281a7ed/.ssh/google_compute_engine.pub
The key fingerprint is:
SHA256:syYUySRt2GcdFlKVqQ0fzsenm/uf0sSk7VAw8rbCcpI student_01_c4faa281a7ed@cs-765069876913-default
The key's randomart image is:
+---[RSA 3072]----+
|    .+. .o=+.o   |
|    .++.ooo.+o   |
|     .+o   Booo  |
|       .  . =ooo.|
|      . S o ..Bo |
|     .   E + +.+ |
|      . o + . =o |
|       o     .oo.|
|              o++|
+----[SHA256]-----+
Warning: Permanently added 'compute.2507327757924029184' (ECDSA) to the list of known hosts.
Linux gcelab2 5.10.0-21-cloud-amd64 #1 SMP Debian 5.10.162-1 (2023-01-21) x86_64

The programs included with the Debian GNU/Linux system are free software;
the exact distribution terms for each program are described in the
individual files in /usr/share/doc/*/copyright.

Debian GNU/Linux comes with ABSOLUTELY NO WARRANTY, to the extent
permitted by applicable law.
Creating directory '/home/student-01-c4faa281a7ed'.
student-01-c4faa281a7ed@gcelab2:~$ sudo apt install -y nginx
Reading package lists... Done
Building dependency tree... Done
Reading state information... Done
The following additional packages will be installed:
  fontconfig-config fonts-dejavu-core geoip-database libdeflate0 libfontconfig1 libgd3 libgeoip1 libicu67 libjbig0 libjpeg62-turbo libnginx-mod-http-geoip
  libnginx-mod-http-image-filter libnginx-mod-http-xslt-filter libnginx-mod-mail libnginx-mod-stream libnginx-mod-stream-geoip libtiff5 libwebp6 libx11-6 libx11-data libxau6
  libxcb1 libxdmcp6 libxml2 libxpm4 libxslt1.1 nginx-common nginx-core
Suggested packages:
  libgd-tools geoip-bin fcgiwrap nginx-doc ssl-cert
The following NEW packages will be installed:
  fontconfig-config fonts-dejavu-core geoip-database libdeflate0 libfontconfig1 libgd3 libgeoip1 libicu67 libjbig0 libjpeg62-turbo libnginx-mod-http-geoip
  libnginx-mod-http-image-filter libnginx-mod-http-xslt-filter libnginx-mod-mail libnginx-mod-stream libnginx-mod-stream-geoip libtiff5 libwebp6 libx11-6 libx11-data libxau6
  libxcb1 libxdmcp6 libxml2 libxpm4 libxslt1.1 nginx nginx-common nginx-core
0 upgraded, 29 newly installed, 0 to remove and 0 not upgraded.
Need to get 18.0 MB of archives.
After this operation, 60.1 MB of additional disk space will be used.
Get:1 http://deb.debian.org/debian bullseye/main amd64 fonts-dejavu-core all 2.37-2 [1069 kB]
Get:2 http://security.debian.org/debian-security bullseye-security/main amd64 libtiff5 amd64 4.2.0-1+deb11u4 [290 kB]
Get:3 http://deb.debian.org/debian bullseye/main amd64 fontconfig-config all 2.13.1-4.2 [281 kB]
Get:4 http://deb.debian.org/debian bullseye/main amd64 geoip-database all 20191224-3 [3032 kB]
Get:5 http://deb.debian.org/debian bullseye/main amd64 libdeflate0 amd64 1.7-1 [53.1 kB]
Get:6 http://deb.debian.org/debian bullseye/main amd64 libfontconfig1 amd64 2.13.1-4.2 [347 kB]
Get:7 http://deb.debian.org/debian bullseye/main amd64 libjpeg62-turbo amd64 1:2.0.6-4 [151 kB]
Get:8 http://deb.debian.org/debian bullseye/main amd64 libjbig0 amd64 2.1-3.1+b2 [31.0 kB]
Get:9 http://deb.debian.org/debian bullseye/main amd64 libwebp6 amd64 0.6.1-2.1 [258 kB]
Get:10 http://deb.debian.org/debian bullseye/main amd64 libxau6 amd64 1:1.0.9-1 [19.7 kB]
Get:11 http://deb.debian.org/debian bullseye/main amd64 libxdmcp6 amd64 1:1.1.2-3 [26.3 kB]
Get:12 http://deb.debian.org/debian bullseye/main amd64 libxcb1 amd64 1.14-3 [140 kB]
Get:13 http://deb.debian.org/debian bullseye/main amd64 libx11-data all 2:1.7.2-1 [311 kB]
Get:14 http://deb.debian.org/debian bullseye/main amd64 libx11-6 amd64 2:1.7.2-1 [772 kB]
Get:15 http://deb.debian.org/debian bullseye/main amd64 libxpm4 amd64 1:3.5.12-1 [49.1 kB]
Get:16 http://deb.debian.org/debian bullseye/main amd64 libgd3 amd64 2.3.0-2 [137 kB]
Get:17 http://deb.debian.org/debian bullseye/main amd64 libgeoip1 amd64 1.6.12-7 [92.5 kB]
Get:18 http://deb.debian.org/debian bullseye/main amd64 libicu67 amd64 67.1-7 [8622 kB]
Get:19 http://deb.debian.org/debian bullseye/main amd64 nginx-common all 1.18.0-6.1+deb11u3 [126 kB]
Get:20 http://deb.debian.org/debian bullseye/main amd64 libnginx-mod-http-geoip amd64 1.18.0-6.1+deb11u3 [98.4 kB]
Get:21 http://deb.debian.org/debian bullseye/main amd64 libnginx-mod-http-image-filter amd64 1.18.0-6.1+deb11u3 [102 kB]
Get:22 http://deb.debian.org/debian bullseye/main amd64 libxml2 amd64 2.9.10+dfsg-6.7+deb11u3 [693 kB]
Get:23 http://deb.debian.org/debian bullseye/main amd64 libxslt1.1 amd64 1.1.34-4+deb11u1 [240 kB]
Get:24 http://deb.debian.org/debian bullseye/main amd64 libnginx-mod-http-xslt-filter amd64 1.18.0-6.1+deb11u3 [100 kB]
Get:25 http://deb.debian.org/debian bullseye/main amd64 libnginx-mod-mail amd64 1.18.0-6.1+deb11u3 [129 kB]
Get:26 http://deb.debian.org/debian bullseye/main amd64 libnginx-mod-stream amd64 1.18.0-6.1+deb11u3 [154 kB]
Get:27 http://deb.debian.org/debian bullseye/main amd64 libnginx-mod-stream-geoip amd64 1.18.0-6.1+deb11u3 [97.7 kB]
Get:28 http://deb.debian.org/debian bullseye/main amd64 nginx-core amd64 1.18.0-6.1+deb11u3 [515 kB]
Get:29 http://deb.debian.org/debian bullseye/main amd64 nginx all 1.18.0-6.1+deb11u3 [92.9 kB]
Fetched 18.0 MB in 1s (33.6 MB/s)
Preconfiguring packages ...
Selecting previously unselected package fonts-dejavu-core.
(Reading database ... 55808 files and directories currently installed.)
Preparing to unpack .../00-fonts-dejavu-core_2.37-2_all.deb ...
Unpacking fonts-dejavu-core (2.37-2) ...
Selecting previously unselected package fontconfig-config.
Preparing to unpack .../01-fontconfig-config_2.13.1-4.2_all.deb ...
Unpacking fontconfig-config (2.13.1-4.2) ...
Selecting previously unselected package geoip-database.
Preparing to unpack .../02-geoip-database_20191224-3_all.deb ...
Unpacking geoip-database (20191224-3) ...
Selecting previously unselected package libdeflate0:amd64.
Preparing to unpack .../03-libdeflate0_1.7-1_amd64.deb ...
Unpacking libdeflate0:amd64 (1.7-1) ...
Selecting previously unselected package libfontconfig1:amd64.
Preparing to unpack .../04-libfontconfig1_2.13.1-4.2_amd64.deb ...
Unpacking libfontconfig1:amd64 (2.13.1-4.2) ...
Selecting previously unselected package libjpeg62-turbo:amd64.
Preparing to unpack .../05-libjpeg62-turbo_1%3a2.0.6-4_amd64.deb ...
Unpacking libjpeg62-turbo:amd64 (1:2.0.6-4) ...
Selecting previously unselected package libjbig0:amd64.
Preparing to unpack .../06-libjbig0_2.1-3.1+b2_amd64.deb ...
Unpacking libjbig0:amd64 (2.1-3.1+b2) ...
Selecting previously unselected package libwebp6:amd64.
Preparing to unpack .../07-libwebp6_0.6.1-2.1_amd64.deb ...
Unpacking libwebp6:amd64 (0.6.1-2.1) ...
Selecting previously unselected package libtiff5:amd64.
Preparing to unpack .../08-libtiff5_4.2.0-1+deb11u4_amd64.deb ...
Unpacking libtiff5:amd64 (4.2.0-1+deb11u4) ...
Selecting previously unselected package libxau6:amd64.
Preparing to unpack .../09-libxau6_1%3a1.0.9-1_amd64.deb ...
Unpacking libxau6:amd64 (1:1.0.9-1) ...
Selecting previously unselected package libxdmcp6:amd64.
Preparing to unpack .../10-libxdmcp6_1%3a1.1.2-3_amd64.deb ...
Unpacking libxdmcp6:amd64 (1:1.1.2-3) ...
Selecting previously unselected package libxcb1:amd64.
Preparing to unpack .../11-libxcb1_1.14-3_amd64.deb ...
Unpacking libxcb1:amd64 (1.14-3) ...
Selecting previously unselected package libx11-data.
Preparing to unpack .../12-libx11-data_2%3a1.7.2-1_all.deb ...
Unpacking libx11-data (2:1.7.2-1) ...
Selecting previously unselected package libx11-6:amd64.
Preparing to unpack .../13-libx11-6_2%3a1.7.2-1_amd64.deb ...
Unpacking libx11-6:amd64 (2:1.7.2-1) ...
Selecting previously unselected package libxpm4:amd64.
Preparing to unpack .../14-libxpm4_1%3a3.5.12-1_amd64.deb ...
Unpacking libxpm4:amd64 (1:3.5.12-1) ...
Selecting previously unselected package libgd3:amd64.
Preparing to unpack .../15-libgd3_2.3.0-2_amd64.deb ...
Unpacking libgd3:amd64 (2.3.0-2) ...
Selecting previously unselected package libgeoip1:amd64.
Preparing to unpack .../16-libgeoip1_1.6.12-7_amd64.deb ...
Unpacking libgeoip1:amd64 (1.6.12-7) ...
Selecting previously unselected package libicu67:amd64.
Preparing to unpack .../17-libicu67_67.1-7_amd64.deb ...
Unpacking libicu67:amd64 (67.1-7) ...
Selecting previously unselected package nginx-common.
Preparing to unpack .../18-nginx-common_1.18.0-6.1+deb11u3_all.deb ...
Unpacking nginx-common (1.18.0-6.1+deb11u3) ...
Selecting previously unselected package libnginx-mod-http-geoip.
Preparing to unpack .../19-libnginx-mod-http-geoip_1.18.0-6.1+deb11u3_amd64.deb ...
Unpacking libnginx-mod-http-geoip (1.18.0-6.1+deb11u3) ...
Selecting previously unselected package libnginx-mod-http-image-filter.
Preparing to unpack .../20-libnginx-mod-http-image-filter_1.18.0-6.1+deb11u3_amd64.deb ...
Unpacking libnginx-mod-http-image-filter (1.18.0-6.1+deb11u3) ...
Selecting previously unselected package libxml2:amd64.
Preparing to unpack .../21-libxml2_2.9.10+dfsg-6.7+deb11u3_amd64.deb ...
Unpacking libxml2:amd64 (2.9.10+dfsg-6.7+deb11u3) ...
Selecting previously unselected package libxslt1.1:amd64.
Preparing to unpack .../22-libxslt1.1_1.1.34-4+deb11u1_amd64.deb ...
Unpacking libxslt1.1:amd64 (1.1.34-4+deb11u1) ...
Selecting previously unselected package libnginx-mod-http-xslt-filter.
Preparing to unpack .../23-libnginx-mod-http-xslt-filter_1.18.0-6.1+deb11u3_amd64.deb ...
Unpacking libnginx-mod-http-xslt-filter (1.18.0-6.1+deb11u3) ...
Selecting previously unselected package libnginx-mod-mail.
Preparing to unpack .../24-libnginx-mod-mail_1.18.0-6.1+deb11u3_amd64.deb ...
Unpacking libnginx-mod-mail (1.18.0-6.1+deb11u3) ...
Selecting previously unselected package libnginx-mod-stream.
Preparing to unpack .../25-libnginx-mod-stream_1.18.0-6.1+deb11u3_amd64.deb ...
Unpacking libnginx-mod-stream (1.18.0-6.1+deb11u3) ...
Selecting previously unselected package libnginx-mod-stream-geoip.
Preparing to unpack .../26-libnginx-mod-stream-geoip_1.18.0-6.1+deb11u3_amd64.deb ...
Unpacking libnginx-mod-stream-geoip (1.18.0-6.1+deb11u3) ...
Selecting previously unselected package nginx-core.
Preparing to unpack .../27-nginx-core_1.18.0-6.1+deb11u3_amd64.deb ...
Unpacking nginx-core (1.18.0-6.1+deb11u3) ...
Selecting previously unselected package nginx.
Preparing to unpack .../28-nginx_1.18.0-6.1+deb11u3_all.deb ...
Unpacking nginx (1.18.0-6.1+deb11u3) ...
Setting up libxau6:amd64 (1:1.0.9-1) ...
Setting up libxdmcp6:amd64 (1:1.1.2-3) ...
Setting up libxcb1:amd64 (1.14-3) ...
Setting up libicu67:amd64 (67.1-7) ...
Setting up libdeflate0:amd64 (1.7-1) ...
Setting up nginx-common (1.18.0-6.1+deb11u3) ...
Created symlink /etc/systemd/system/multi-user.target.wants/nginx.service → /lib/systemd/system/nginx.service.
Setting up libjbig0:amd64 (2.1-3.1+b2) ...
Setting up libjpeg62-turbo:amd64 (1:2.0.6-4) ...
Setting up libx11-data (2:1.7.2-1) ...
Setting up libwebp6:amd64 (0.6.1-2.1) ...
Setting up fonts-dejavu-core (2.37-2) ...
Setting up libgeoip1:amd64 (1.6.12-7) ...
Setting up libx11-6:amd64 (2:1.7.2-1) ...
Setting up libtiff5:amd64 (4.2.0-1+deb11u4) ...
Setting up geoip-database (20191224-3) ...
Setting up libxml2:amd64 (2.9.10+dfsg-6.7+deb11u3) ...
Setting up libnginx-mod-mail (1.18.0-6.1+deb11u3) ...
Setting up libxpm4:amd64 (1:3.5.12-1) ...
Setting up fontconfig-config (2.13.1-4.2) ...
Setting up libnginx-mod-stream (1.18.0-6.1+deb11u3) ...
Setting up libnginx-mod-stream-geoip (1.18.0-6.1+deb11u3) ...
Setting up libnginx-mod-http-geoip (1.18.0-6.1+deb11u3) ...
Setting up libxslt1.1:amd64 (1.1.34-4+deb11u1) ...
Setting up libfontconfig1:amd64 (2.13.1-4.2) ...
Setting up libnginx-mod-http-xslt-filter (1.18.0-6.1+deb11u3) ...
Setting up libgd3:amd64 (2.3.0-2) ...
Setting up libnginx-mod-http-image-filter (1.18.0-6.1+deb11u3) ...
Setting up nginx-core (1.18.0-6.1+deb11u3) ...
Upgrading binary: nginx.
Setting up nginx (1.18.0-6.1+deb11u3) ...
Processing triggers for man-db (2.9.4-2) ...
Processing triggers for libc-bin (2.31-13+deb11u5) ...
student-01-c4faa281a7ed@gcelab2:~$ exit
logout
Connection to 34.136.108.219 closed.
student_01_c4faa281a7ed@cloudshell:~ (qwiklabs-gcp-01-823ec07351c5)$
```



```sh
student_01_c4faa281a7ed@cloudshell:~ (qwiklabs-gcp-01-823ec07351c5)$ gcloud compute firewall-rules list
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

To show all fields of the firewall, please show in JSON format: --format=json
To show all fields in table format, please see the examples in --help.

student_01_c4faa281a7ed@cloudshell:~ (qwiklabs-gcp-01-823ec07351c5)$
```



```sh
student_01_c4faa281a7ed@cloudshell:~ (qwiklabs-gcp-01-823ec07351c5)$ gcloud compute instances add-tags gcelab2 --tags http-server,https-server
Updated [https://www.googleapis.com/compute/v1/projects/qwiklabs-gcp-01-823ec07351c5/zones/us-central1-b/instances/gcelab2].
student_01_c4faa281a7ed@cloudshell:~ (qwiklabs-gcp-01-823ec07351c5)$
```



```sh
student_01_c4faa281a7ed@cloudshell:~ (qwiklabs-gcp-01-823ec07351c5)$ gcloud compute firewall-rules create default-allow-http --direction=INGRESS --priority=1000 --network=default --action=ALLOW --rules=tcp:80 --source-ranges=0.0.0.0/0 --target-tags=http-server
Creating firewall...working..Created [https://www.googleapis.com/compute/v1/projects/qwiklabs-gcp-01-823ec07351c5/global/firewalls/default-allow-http].
Creating firewall...done.
NAME: default-allow-http
NETWORK: default
DIRECTION: INGRESS
PRIORITY: 1000
ALLOW: tcp:80
DENY:
DISABLED: False
student_01_c4faa281a7ed@cloudshell:~ (qwiklabs-gcp-01-823ec07351c5)$
```

```sh
student_01_c4faa281a7ed@cloudshell:~ (qwiklabs-gcp-01-823ec07351c5)$ gcloud compute firewall-rules list --filter=ALLOW:'80'
NAME: default-allow-http
NETWORK: default
DIRECTION: INGRESS
PRIORITY: 1000
ALLOW: tcp:80
DENY:
DISABLED: False

To show all fields of the firewall, please show in JSON format: --format=json
To show all fields in table format, please see the examples in --help.

student_01_c4faa281a7ed@cloudshell:~ (qwiklabs-gcp-01-823ec07351c5)$
```

```sh
student_01_c4faa281a7ed@cloudshell:~ (qwiklabs-gcp-01-823ec07351c5)$ curl http://$(gcloud compute instances list --filter=name:gcelab2 --format='value(EXTERNAL_IP)')
<!DOCTYPE html>
<html>
<head>
<title>Welcome to nginx!</title>
<style>
    body {
        width: 35em;
        margin: 0 auto;
        font-family: Tahoma, Verdana, Arial, sans-serif;
    }
</style>
</head>
<body>
<h1>Welcome to nginx!</h1>
<p>If you see this page, the nginx web server is successfully installed and
working. Further configuration is required.</p>

<p>For online documentation and support please refer to
<a href="http://nginx.org/">nginx.org</a>.<br/>
Commercial support is available at
<a href="http://nginx.com/">nginx.com</a>.</p>

<p><em>Thank you for using nginx.</em></p>
</body>
</html>
student_01_c4faa281a7ed@cloudshell:~ (qwiklabs-gcp-01-823ec07351c5)$
```



```sh
student_01_c4faa281a7ed@cloudshell:~ (qwiklabs-gcp-01-823ec07351c5)$ gcloud logging logs list
NAME: projects/qwiklabs-gcp-01-823ec07351c5/logs/GCEGuestAgent

NAME: projects/qwiklabs-gcp-01-823ec07351c5/logs/OSConfigAgent

NAME: projects/qwiklabs-gcp-01-823ec07351c5/logs/cloudaudit.googleapis.com%2Factivity

NAME: projects/qwiklabs-gcp-01-823ec07351c5/logs/cloudaudit.googleapis.com%2Fdata_access

NAME: projects/qwiklabs-gcp-01-823ec07351c5/logs/compute.googleapis.com%2Fshielded_vm_integrity

NAME: projects/qwiklabs-gcp-01-823ec07351c5/logs/networkanalyzer.googleapis.com%2Fanalyzer_reports
student_01_c4faa281a7ed@cloudshell:~ (qwiklabs-gcp-01-823ec07351c5)$
```



```sh
student_01_c4faa281a7ed@cloudshell:~ (qwiklabs-gcp-01-823ec07351c5)$ gcloud logging logs list --filter="compute"
NAME: projects/qwiklabs-gcp-01-823ec07351c5/logs/compute.googleapis.com%2Fshielded_vm_integrity
student_01_c4faa281a7ed@cloudshell:~ (qwiklabs-gcp-01-823ec07351c5)$
```



```sh
student_01_c4faa281a7ed@cloudshell:~ (qwiklabs-gcp-01-823ec07351c5)$ gcloud logging read "resource.type=gce_instance" --limit 5
---
insertId: -1xv475d7i78
logName: projects/qwiklabs-gcp-01-823ec07351c5/logs/cloudaudit.googleapis.com%2Factivity
operation:
  id: operation-1680109564252-5f80cfdc2cf6b-f7bfd190-f98eb9fc
  last: true
  producer: compute.googleapis.com
protoPayload:
  '@type': type.googleapis.com/google.cloud.audit.AuditLog
  authenticationInfo:
    principalEmail: student-01-c4faa281a7ed@qwiklabs.net
    principalSubject: user:student-01-c4faa281a7ed@qwiklabs.net
  methodName: v1.compute.instances.setTags
  request:
    '@type': type.googleapis.com/compute.instances.setTags
  requestMetadata:
    callerIp: 34.126.87.210
    callerSuppliedUserAgent: google-cloud-sdk gcloud/423.0.0 command/gcloud.compute.instances.add-tags
      invocation-id/dac8d4fa8156403981df403839c36139 environment/devshell environment-version/None
      interactive/True from-script/False python/3.9.2 term/screen (Linux 5.15.89+),gzip(gfe)
    destinationAttributes: {}
    requestAttributes: {}
  resourceName: projects/qwiklabs-gcp-01-823ec07351c5/zones/us-central1-b/instances/gcelab2
  serviceName: compute.googleapis.com
receiveTimestamp: '2023-03-29T17:06:07.122679399Z'
resource:
  labels:
    instance_id: '2507327757924029184'
    project_id: qwiklabs-gcp-01-823ec07351c5
    zone: us-central1-b
  type: gce_instance
severity: NOTICE
timestamp: '2023-03-29T17:06:06.530711Z'
---
insertId: bdk922dwugc
logName: projects/qwiklabs-gcp-01-823ec07351c5/logs/cloudaudit.googleapis.com%2Factivity
operation:
  first: true
  id: operation-1680109564252-5f80cfdc2cf6b-f7bfd190-f98eb9fc
  producer: compute.googleapis.com
protoPayload:
  '@type': type.googleapis.com/google.cloud.audit.AuditLog
  authenticationInfo:
    principalEmail: student-01-c4faa281a7ed@qwiklabs.net
    principalSubject: user:student-01-c4faa281a7ed@qwiklabs.net
  authorizationInfo:
  - granted: true
    permission: compute.instances.setTags
    resourceAttributes:
      name: projects/qwiklabs-gcp-01-823ec07351c5/zones/us-central1-b/instances/gcelab2
      service: compute
      type: compute.instances
  methodName: v1.compute.instances.setTags
  request:
    '@type': type.googleapis.com/compute.instances.setTags
    fingerprint: �e�J�|�#
    tags:
    - http-server
    - https-server
  requestMetadata:
    callerIp: 34.126.87.210
    callerSuppliedUserAgent: google-cloud-sdk gcloud/423.0.0 command/gcloud.compute.instances.add-tags
      invocation-id/dac8d4fa8156403981df403839c36139 environment/devshell environment-version/None
      interactive/True from-script/False python/3.9.2 term/screen (Linux 5.15.89+),gzip(gfe)
    destinationAttributes: {}
    requestAttributes:
      auth: {}
      time: '2023-03-29T17:06:04.727961Z'
  resourceLocation:
    currentLocations:
    - us-central1-b
  resourceName: projects/qwiklabs-gcp-01-823ec07351c5/zones/us-central1-b/instances/gcelab2
  response:
    '@type': type.googleapis.com/operation
    id: '5367456835220132115'
    insertTime: '2023-03-29T10:06:04.681-07:00'
    name: operation-1680109564252-5f80cfdc2cf6b-f7bfd190-f98eb9fc
    operationType: setTags
    progress: '0'
    selfLink: https://www.googleapis.com/compute/v1/projects/qwiklabs-gcp-01-823ec07351c5/zones/us-central1-b/operations/operation-1680109564252-5f80cfdc2cf6b-f7bfd190-f98eb9fc
    selfLinkWithId: https://www.googleapis.com/compute/v1/projects/qwiklabs-gcp-01-823ec07351c5/zones/us-central1-b/operations/5367456835220132115
    startTime: '2023-03-29T10:06:04.697-07:00'
    status: RUNNING
    targetId: '2507327757924029184'
    targetLink: https://www.googleapis.com/compute/v1/projects/qwiklabs-gcp-01-823ec07351c5/zones/us-central1-b/instances/gcelab2
    user: student-01-c4faa281a7ed@qwiklabs.net
    zone: https://www.googleapis.com/compute/v1/projects/qwiklabs-gcp-01-823ec07351c5/zones/us-central1-b
  serviceName: compute.googleapis.com
receiveTimestamp: '2023-03-29T17:06:04.873646818Z'
resource:
  labels:
    instance_id: '2507327757924029184'
    project_id: qwiklabs-gcp-01-823ec07351c5
    zone: us-central1-b
  type: gce_instance
severity: NOTICE
timestamp: '2023-03-29T17:06:04.365658Z'
---
insertId: 1aw6tz8ffl5ggi
jsonPayload:
  localTimestamp: '2023-03-29T16:57:41.9052Z'
  message: Created google sudoers file
  omitempty: null
labels:
  instance_name: gcelab2
logName: projects/qwiklabs-gcp-01-823ec07351c5/logs/GCEGuestAgent
receiveTimestamp: '2023-03-29T16:57:42.394782118Z'
resource:
  labels:
    instance_id: '2507327757924029184'
    project_id: qwiklabs-gcp-01-823ec07351c5
    zone: us-central1-b
  type: gce_instance
severity: INFO
sourceLocation:
  file: non_windows_accounts.go
  function: main.createSudoersGroup
  line: '395'
timestamp: '2023-03-29T16:57:41.905278769Z'
---
insertId: 1aw6tz8ffl5ggh
jsonPayload:
  localTimestamp: '2023-03-29T16:57:41.6318Z'
  message: Enabling OS Login
  omitempty: null
labels:
  instance_name: gcelab2
logName: projects/qwiklabs-gcp-01-823ec07351c5/logs/GCEGuestAgent
receiveTimestamp: '2023-03-29T16:57:42.394782118Z'
resource:
  labels:
    instance_id: '2507327757924029184'
    project_id: qwiklabs-gcp-01-823ec07351c5
    zone: us-central1-b
  type: gce_instance
severity: INFO
sourceLocation:
  file: oslogin.go
  function: main.(*osloginMgr).set
  line: '89'
timestamp: '2023-03-29T16:57:41.631923734Z'
---
insertId: efplabg123jci4
jsonPayload:
  localTimestamp: '2023-03-29T16:57:41.3631Z'
  message: OSConfig Agent (version 20221214.00-g1) started.
  omitempty: null
labels:
  agent_version: 20221214.00-g1
  instance_name: gcelab2
logName: projects/qwiklabs-gcp-01-823ec07351c5/logs/OSConfigAgent
receiveTimestamp: '2023-03-29T16:57:42.642916168Z'
resource:
  labels:
    instance_id: '2507327757924029184'
    project_id: qwiklabs-gcp-01-823ec07351c5
    zone: us-central1-b
  type: gce_instance
severity: INFO
sourceLocation:
  file: main.go
  function: main.run
  line: '151'
timestamp: '2023-03-29T16:57:41.363201778Z'
student_01_c4faa281a7ed@cloudshell:~ (qwiklabs-gcp-01-823ec07351c5)$
```



```sh
student_01_c4faa281a7ed@cloudshell:~ (qwiklabs-gcp-01-823ec07351c5)$ gcloud logging read "resource.type=gce_instance AND labels.instance_name='gcelab2'" --limit 5
student_01_c4faa281a7ed@cloudshell:~ (qwiklabs-gcp-01-823ec07351c5)$
```

