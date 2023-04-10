---


layout: single
title:  "Working with Virtual Machines"
date:   2023-04-09 07:59:04 +0530
categories: Cloud
tags: GCP
classes: wide
show_date: true
header:
  overlay_image: /assets/images/gcp-banner-2.png
  og_image: /assets/images/gcp-banner-2.png
  teaser: /assets/images/gpcne.png
author:
  name     : "Cloud Network Engineer"
  avatar   : "/assets/images/gpcne.png"

sidebar:
  - title: "Topics"
    nav: my-sidebar

---

# Working with Virtual Machines

In this lab, you set up a game application—a Minecraft server.

The Minecraft server software will run on a Compute Engine instance.

You use an e2-medium machine type that includes a 10-GB boot disk, 2  virtual CPU (vCPU), and 4 GB of RAM. This machine type runs Debian Linux by default.

To make sure there is plenty of room for the  Minecraft server's  world data, you also attach a high-performance 50-GB persistent  solid-state drive (SSD) to the instance. This dedicated Minecraft server can support up to 50 players.

```sh
gcloud compute instances create mc-server --project=qwiklabs-gcp-03-2b0cb03f1121 --zone=us-central1-a --machine-type=e2-medium --network-interface=address=34.134.15.247,network-tier=PREMIUM,subnet=default --metadata=enable-oslogin=true --maintenance-policy=MIGRATE --provisioning-model=STANDARD --service-account=657674383499-compute@developer.gserviceaccount.com --scopes=https://www.googleapis.com/auth/servicecontrol,https://www.googleapis.com/auth/service.management.readonly,https://www.googleapis.com/auth/logging.write,https://www.googleapis.com/auth/monitoring.write,https://www.googleapis.com/auth/trace.append,https://www.googleapis.com/auth/devstorage.read_write --create-disk=auto-delete=yes,boot=yes,device-name=mc-server,image=projects/debian-cloud/global/images/debian-11-bullseye-v20230306,mode=rw,size=10,type=projects/qwiklabs-gcp-03-2b0cb03f1121/zones/us-central1-a/diskTypes/pd-balanced --create-disk=device-name=minecraft-disk,mode=rw,name=minecraft-disk,size=50,type=projects/qwiklabs-gcp-03-2b0cb03f1121/zones/us-central1-a/diskTypes/pd-ssd --no-shielded-secure-boot --shielded-vtpm --shielded-integrity-monitoring --labels=ec-src=vm_add-gcloud --reservation-affinity=any
```

```sh
student-01-cd9de465be5c@mc-server:~$ sudo mkdir -p /home/minecraft
student-01-cd9de465be5c@mc-server:~$ sudo mkfs.ext4 -F -E lazy_itable_init=0,\
lazy_journal_init=0,discard \
/dev/disk/by-id/google-minecraft-disk
mke2fs 1.46.2 (28-Feb-2021)
Discarding device blocks: done                            
Creating filesystem with 13107200 4k blocks and 3276800 inodes
Filesystem UUID: 458ee512-23e3-460c-9e8a-e787b9e57c2f
Superblock backups stored on blocks: 
        32768, 98304, 163840, 229376, 294912, 819200, 884736, 1605632, 2654208, 
        4096000, 7962624, 11239424

Allocating group tables: done                            
Writing inode tables: done                            
Creating journal (65536 blocks): done
Writing superblocks and filesystem accounting information: done   

student-01-cd9de465be5c@mc-server:~$ sudo mount -o discard,defaults /dev/disk/by-id/google-minecraft-disk /home/minecraft
student-01-cd9de465be5c@mc-server:~$ 

```

```sh
student-01-cd9de465be5c@mc-server:~$ sudo apt-get update
Get:1 http://packages.cloud.google.com/apt google-compute-engine-bullseye-stable InRelease [5146 B]
Get:2 http://packages.cloud.google.com/apt cloud-sdk-bullseye InRelease [6400 B]                              
Hit:3 http://deb.debian.org/debian bullseye InRelease                                                         
Get:4 http://security.debian.org/debian-security bullseye-security InRelease [48.4 kB]                        
Get:5 http://deb.debian.org/debian bullseye-updates InRelease [44.1 kB]
Get:6 http://deb.debian.org/debian bullseye-backports InRelease [49.0 kB]
Get:7 http://packages.cloud.google.com/apt google-compute-engine-bullseye-stable/main amd64 Packages [1928 B]
Get:8 http://packages.cloud.google.com/apt cloud-sdk-bullseye/main amd64 Packages [269 kB]
Get:9 http://deb.debian.org/debian bullseye-updates/main Sources.diff/Index [17.3 kB]
Get:10 http://deb.debian.org/debian bullseye-updates/main amd64 Packages.diff/Index [17.3 kB]
Get:11 http://deb.debian.org/debian bullseye-updates/main Sources T-2023-03-25-2025.40-F-2023-03-25-2025.40.pdiff [391 B]
Get:11 http://deb.debian.org/debian bullseye-updates/main Sources T-2023-03-25-2025.40-F-2023-03-25-2025.40.pdiff [391 B]
Get:12 http://deb.debian.org/debian bullseye-updates/main amd64 Packages T-2023-03-25-2025.40-F-2023-03-25-2025.40.pdiff [288 B]
Get:12 http://deb.debian.org/debian bullseye-updates/main amd64 Packages T-2023-03-25-2025.40-F-2023-03-25-2025.40.pdiff [288 B]
Get:13 http://security.debian.org/debian-security bullseye-security/main Sources [189 kB]
Get:14 http://security.debian.org/debian-security bullseye-security/main amd64 Packages [236 kB]
Get:15 http://deb.debian.org/debian bullseye-backports/main Sources.diff/Index [63.3 kB]
Get:16 http://security.debian.org/debian-security bullseye-security/main Translation-en [155 kB]
Ign:15 http://deb.debian.org/debian bullseye-backports/main Sources.diff/Index                  
Get:17 http://deb.debian.org/debian bullseye-backports/main amd64 Packages.diff/Index [63.3 kB] 
Get:18 http://deb.debian.org/debian bullseye-backports/main Translation-en.diff/Index [63.3 kB]
Get:19 http://deb.debian.org/debian bullseye-backports/main amd64 Packages T-2023-04-06-0203.00-F-2023-03-06-2006.07.pdiff [61.8 kB]
Get:19 http://deb.debian.org/debian bullseye-backports/main amd64 Packages T-2023-04-06-0203.00-F-2023-03-06-2006.07.pdiff [61.8 kB]
Get:20 http://deb.debian.org/debian bullseye-backports/main Translation-en T-2023-04-03-2007.49-F-2023-03-07-2009.30.pdiff [30.8 kB]
Get:20 http://deb.debian.org/debian bullseye-backports/main Translation-en T-2023-04-03-2007.49-F-2023-03-07-2009.30.pdiff [30.8 kB]
Get:21 http://deb.debian.org/debian bullseye-backports/main Sources [420 kB]     
Fetched 1742 kB in 1s (1404 kB/s)                                                            
Reading package lists... Done
student-01-cd9de465be5c@mc-server:~$ 
```

```sh
student-01-cd9de465be5c@mc-server:~$ sudo apt-get install -y default-jre-headless
Reading package lists... Done
Building dependency tree... Done
Reading state information... Done
The following additional packages will be installed:
  alsa-topology-conf alsa-ucm-conf ca-certificates-java fontconfig-config fonts-dejavu-core java-common
  libasound2 libasound2-data libavahi-client3 libavahi-common-data libavahi-common3 libcups2 libfontconfig1
  libgraphite2-3 libharfbuzz0b libjpeg62-turbo liblcms2-2 libnspr4 libnss3 libpcsclite1
  openjdk-11-jre-headless
Suggested packages:
  default-jre libasound2-plugins alsa-utils cups-common liblcms2-utils pcscd libnss-mdns fonts-dejavu-extra
  fonts-ipafont-gothic fonts-ipafont-mincho fonts-wqy-microhei | fonts-wqy-zenhei fonts-indic
The following NEW packages will be installed:
  alsa-topology-conf alsa-ucm-conf ca-certificates-java default-jre-headless fontconfig-config
  fonts-dejavu-core java-common libasound2 libasound2-data libavahi-client3 libavahi-common-data
  libavahi-common3 libcups2 libfontconfig1 libgraphite2-3 libharfbuzz0b libjpeg62-turbo liblcms2-2 libnspr4
  libnss3 libpcsclite1 openjdk-11-jre-headless
0 upgraded, 22 newly installed, 0 to remove and 6 not upgraded.
Need to get 43.5 MB of archives.
After this operation, 187 MB of additional disk space will be used.
Get:1 http://deb.debian.org/debian bullseye/main amd64 alsa-topology-conf all 1.2.4-1 [12.8 kB]
Get:2 http://security.debian.org/debian-security bullseye-security/main amd64 libnss3 amd64 2:3.61-1+deb11u3 [1305 kB]
Get:3 http://deb.debian.org/debian bullseye/main amd64 libasound2-data all 1.2.4-1.1 [38.2 kB]
Get:4 http://deb.debian.org/debian bullseye/main amd64 libasound2 amd64 1.2.4-1.1 [356 kB]
Get:5 http://deb.debian.org/debian bullseye/main amd64 alsa-ucm-conf all 1.2.4-2 [28.1 kB]
Get:6 http://deb.debian.org/debian bullseye/main amd64 java-common all 0.72 [14.5 kB]
Get:7 http://deb.debian.org/debian bullseye/main amd64 libavahi-common-data amd64 0.8-5+deb11u1 [124 kB]
Get:8 http://deb.debian.org/debian bullseye/main amd64 libavahi-common3 amd64 0.8-5+deb11u1 [58.7 kB]
Get:9 http://deb.debian.org/debian bullseye/main amd64 libavahi-client3 amd64 0.8-5+deb11u1 [62.4 kB]
Get:10 http://deb.debian.org/debian bullseye/main amd64 libcups2 amd64 2.3.3op2-3+deb11u2 [350 kB]
Get:11 http://deb.debian.org/debian bullseye/main amd64 liblcms2-2 amd64 2.12~rc1-2 [150 kB]
Get:12 http://deb.debian.org/debian bullseye/main amd64 libjpeg62-turbo amd64 1:2.0.6-4 [151 kB]
Get:13 http://deb.debian.org/debian bullseye/main amd64 fonts-dejavu-core all 2.37-2 [1069 kB]
Get:14 http://security.debian.org/debian-security bullseye-security/main amd64 openjdk-11-jre-headless amd64 11.0.18+10-1~deb11u1 [37.4 MB]
Get:15 http://deb.debian.org/debian bullseye/main amd64 fontconfig-config all 2.13.1-4.2 [281 kB]
Get:16 http://deb.debian.org/debian bullseye/main amd64 libfontconfig1 amd64 2.13.1-4.2 [347 kB]
Get:17 http://deb.debian.org/debian bullseye/main amd64 libnspr4 amd64 2:4.29-1 [112 kB]
Get:18 http://deb.debian.org/debian bullseye/main amd64 libgraphite2-3 amd64 1.3.14-1 [81.2 kB]
Get:19 http://deb.debian.org/debian bullseye/main amd64 libharfbuzz0b amd64 2.7.4-1 [1471 kB]
Get:20 http://deb.debian.org/debian bullseye/main amd64 libpcsclite1 amd64 1.9.1-1 [60.2 kB]
Get:21 http://deb.debian.org/debian bullseye/main amd64 default-jre-headless amd64 2:1.11-72 [10.9 kB]
Get:22 http://deb.debian.org/debian bullseye/main amd64 ca-certificates-java all 20190909 [15.7 kB]
Fetched 43.5 MB in 1s (42.4 MB/s)                                                            
Preconfiguring packages ...
Selecting previously unselected package alsa-topology-conf.
(Reading database ... 55808 files and directories currently installed.)
Preparing to unpack .../00-alsa-topology-conf_1.2.4-1_all.deb ...
Unpacking alsa-topology-conf (1.2.4-1) ...
Selecting previously unselected package libasound2-data.
Preparing to unpack .../01-libasound2-data_1.2.4-1.1_all.deb ...
Unpacking libasound2-data (1.2.4-1.1) ...
Selecting previously unselected package libasound2:amd64.
Preparing to unpack .../02-libasound2_1.2.4-1.1_amd64.deb ...
Unpacking libasound2:amd64 (1.2.4-1.1) ...
Selecting previously unselected package alsa-ucm-conf.
Preparing to unpack .../03-alsa-ucm-conf_1.2.4-2_all.deb ...
Unpacking alsa-ucm-conf (1.2.4-2) ...
Selecting previously unselected package java-common.
Preparing to unpack .../04-java-common_0.72_all.deb ...
Unpacking java-common (0.72) ...
Selecting previously unselected package libavahi-common-data:amd64.
Preparing to unpack .../05-libavahi-common-data_0.8-5+deb11u1_amd64.deb ...
Unpacking libavahi-common-data:amd64 (0.8-5+deb11u1) ...
Selecting previously unselected package libavahi-common3:amd64.
Preparing to unpack .../06-libavahi-common3_0.8-5+deb11u1_amd64.deb ...
Unpacking libavahi-common3:amd64 (0.8-5+deb11u1) ...
Selecting previously unselected package libavahi-client3:amd64.
Preparing to unpack .../07-libavahi-client3_0.8-5+deb11u1_amd64.deb ...
Unpacking libavahi-client3:amd64 (0.8-5+deb11u1) ...
Selecting previously unselected package libcups2:amd64.
Preparing to unpack .../08-libcups2_2.3.3op2-3+deb11u2_amd64.deb ...
Unpacking libcups2:amd64 (2.3.3op2-3+deb11u2) ...
Selecting previously unselected package liblcms2-2:amd64.
Preparing to unpack .../09-liblcms2-2_2.12~rc1-2_amd64.deb ...
Unpacking liblcms2-2:amd64 (2.12~rc1-2) ...
Selecting previously unselected package libjpeg62-turbo:amd64.
Preparing to unpack .../10-libjpeg62-turbo_1%3a2.0.6-4_amd64.deb ...
Unpacking libjpeg62-turbo:amd64 (1:2.0.6-4) ...
Selecting previously unselected package fonts-dejavu-core.
Preparing to unpack .../11-fonts-dejavu-core_2.37-2_all.deb ...
Unpacking fonts-dejavu-core (2.37-2) ...
Selecting previously unselected package fontconfig-config.
Preparing to unpack .../12-fontconfig-config_2.13.1-4.2_all.deb ...
Unpacking fontconfig-config (2.13.1-4.2) ...
Selecting previously unselected package libfontconfig1:amd64.
Preparing to unpack .../13-libfontconfig1_2.13.1-4.2_amd64.deb ...
Unpacking libfontconfig1:amd64 (2.13.1-4.2) ...
Selecting previously unselected package libnspr4:amd64.
Preparing to unpack .../14-libnspr4_2%3a4.29-1_amd64.deb ...
Unpacking libnspr4:amd64 (2:4.29-1) ...
Selecting previously unselected package libnss3:amd64.
Preparing to unpack .../15-libnss3_2%3a3.61-1+deb11u3_amd64.deb ...
Unpacking libnss3:amd64 (2:3.61-1+deb11u3) ...
Selecting previously unselected package libgraphite2-3:amd64.
Preparing to unpack .../16-libgraphite2-3_1.3.14-1_amd64.deb ...
Unpacking libgraphite2-3:amd64 (1.3.14-1) ...
Selecting previously unselected package libharfbuzz0b:amd64.
Preparing to unpack .../17-libharfbuzz0b_2.7.4-1_amd64.deb ...
Unpacking libharfbuzz0b:amd64 (2.7.4-1) ...
Selecting previously unselected package libpcsclite1:amd64.
Preparing to unpack .../18-libpcsclite1_1.9.1-1_amd64.deb ...
Unpacking libpcsclite1:amd64 (1.9.1-1) ...
Selecting previously unselected package openjdk-11-jre-headless:amd64.
Preparing to unpack .../19-openjdk-11-jre-headless_11.0.18+10-1~deb11u1_amd64.deb ...
Unpacking openjdk-11-jre-headless:amd64 (11.0.18+10-1~deb11u1) ...
Selecting previously unselected package default-jre-headless.
Preparing to unpack .../20-default-jre-headless_2%3a1.11-72_amd64.deb ...
Unpacking default-jre-headless (2:1.11-72) ...
Selecting previously unselected package ca-certificates-java.
Preparing to unpack .../21-ca-certificates-java_20190909_all.deb ...
Unpacking ca-certificates-java (20190909) ...
Setting up libgraphite2-3:amd64 (1.3.14-1) ...
Setting up liblcms2-2:amd64 (2.12~rc1-2) ...
Setting up java-common (0.72) ...
Setting up libasound2-data (1.2.4-1.1) ...
Setting up libjpeg62-turbo:amd64 (1:2.0.6-4) ...
Setting up libnspr4:amd64 (2:4.29-1) ...
Setting up libavahi-common-data:amd64 (0.8-5+deb11u1) ...
Setting up fonts-dejavu-core (2.37-2) ...
Setting up libpcsclite1:amd64 (1.9.1-1) ...
Setting up alsa-topology-conf (1.2.4-1) ...
Setting up libasound2:amd64 (1.2.4-1.1) ...
Setting up libharfbuzz0b:amd64 (2.7.4-1) ...
Setting up alsa-ucm-conf (1.2.4-2) ...
Setting up fontconfig-config (2.13.1-4.2) ...
Setting up libavahi-common3:amd64 (0.8-5+deb11u1) ...
Setting up libnss3:amd64 (2:3.61-1+deb11u3) ...
Setting up libfontconfig1:amd64 (2.13.1-4.2) ...
Setting up libavahi-client3:amd64 (0.8-5+deb11u1) ...
Setting up libcups2:amd64 (2.3.3op2-3+deb11u2) ...
Setting up ca-certificates-java (20190909) ...
head: cannot open '/etc/ssl/certs/java/cacerts' for reading: No such file or directory
Adding debian:Entrust_Root_Certification_Authority_-_G2.pem
Adding debian:GlobalSign_Root_CA_-_R6.pem
Adding debian:SecureTrust_CA.pem
Adding debian:Starfield_Root_Certificate_Authority_-_G2.pem
Adding debian:Trustis_FPS_Root_CA.pem
Adding debian:QuoVadis_Root_CA_3.pem
Adding debian:e-Szigno_Root_CA_2017.pem
Adding debian:Microsec_e-Szigno_Root_CA_2009.pem
Adding debian:Hellenic_Academic_and_Research_Institutions_ECC_RootCA_2015.pem
Adding debian:GTS_Root_R2.pem
Adding debian:CFCA_EV_ROOT.pem
Adding debian:QuoVadis_Root_CA_1_G3.pem
Adding debian:SSL.com_EV_Root_Certification_Authority_RSA_R2.pem
Adding debian:OISTE_WISeKey_Global_Root_GB_CA.pem
Adding debian:TUBITAK_Kamu_SM_SSL_Kok_Sertifikasi_-_Surum_1.pem
Adding debian:OISTE_WISeKey_Global_Root_GC_CA.pem
Adding debian:Baltimore_CyberTrust_Root.pem
Adding debian:Amazon_Root_CA_1.pem
Adding debian:Secure_Global_CA.pem
Adding debian:Certum_Trusted_Network_CA.pem
Adding debian:Security_Communication_Root_CA.pem
Adding debian:UCA_Global_G2_Root.pem
Adding debian:Sonera_Class_2_Root_CA.pem
Adding debian:Comodo_AAA_Services_root.pem
Adding debian:Hellenic_Academic_and_Research_Institutions_RootCA_2011.pem
Adding debian:Security_Communication_RootCA2.pem
Adding debian:Starfield_Class_2_CA.pem
Adding debian:Cybertrust_Global_Root.pem
Adding debian:Certum_Trusted_Network_CA_2.pem
Adding debian:Go_Daddy_Class_2_CA.pem
Adding debian:Atos_TrustedRoot_2011.pem
Adding debian:Amazon_Root_CA_3.pem
Adding debian:Actalis_Authentication_Root_CA.pem
Adding debian:AffirmTrust_Premium_ECC.pem
Adding debian:DigiCert_Global_Root_G3.pem
Adding debian:DigiCert_Assured_ID_Root_CA.pem
Adding debian:QuoVadis_Root_CA_2_G3.pem
Adding debian:TWCA_Global_Root_CA.pem
Adding debian:GlobalSign_ECC_Root_CA_-_R4.pem
Adding debian:USERTrust_ECC_Certification_Authority.pem
Adding debian:ISRG_Root_X1.pem
Adding debian:UCA_Extended_Validation_Root.pem
Adding debian:SSL.com_EV_Root_Certification_Authority_ECC.pem
Adding debian:Entrust_Root_Certification_Authority_-_G4.pem
Adding debian:SwissSign_Silver_CA_-_G2.pem
Adding debian:emSign_Root_CA_-_C1.pem
Adding debian:COMODO_Certification_Authority.pem
Adding debian:DigiCert_Global_Root_CA.pem
Adding debian:DigiCert_Assured_ID_Root_G2.pem
Adding debian:Hellenic_Academic_and_Research_Institutions_RootCA_2015.pem
Adding debian:VeriSign_Universal_Root_Certification_Authority.pem
Adding debian:IdenTrust_Commercial_Root_CA_1.pem
Adding debian:GDCA_TrustAUTH_R5_ROOT.pem
Adding debian:TrustCor_RootCert_CA-1.pem
Adding debian:Buypass_Class_3_Root_CA.pem
Adding debian:Global_Chambersign_Root_-_2008.pem
Adding debian:emSign_ECC_Root_CA_-_C3.pem
Adding debian:Staat_der_Nederlanden_EV_Root_CA.pem
Adding debian:AffirmTrust_Commercial.pem
Adding debian:GlobalSign_Root_CA_-_R2.pem
Adding debian:GlobalSign_Root_CA_-_R3.pem
Adding debian:SwissSign_Gold_CA_-_G2.pem
Adding debian:Entrust_Root_Certification_Authority_-_EC1.pem
Adding debian:Entrust.net_Premium_2048_Secure_Server_CA.pem
Adding debian:SSL.com_Root_Certification_Authority_ECC.pem
Adding debian:Hongkong_Post_Root_CA_1.pem
Adding debian:emSign_ECC_Root_CA_-_G3.pem
Adding debian:Amazon_Root_CA_4.pem
Adding debian:QuoVadis_Root_CA_3_G3.pem
Adding debian:GlobalSign_ECC_Root_CA_-_R5.pem
Adding debian:AffirmTrust_Networking.pem
Adding debian:certSIGN_Root_CA_G2.pem
Adding debian:SSL.com_Root_Certification_Authority_RSA.pem
Adding debian:Network_Solutions_Certificate_Authority.pem
Adding debian:TrustCor_ECA-1.pem
Adding debian:NAVER_Global_Root_Certification_Authority.pem
Adding debian:CA_Disig_Root_R2.pem
Adding debian:GTS_Root_R1.pem
Adding debian:Autoridad_de_Certificacion_Firmaprofesional_CIF_A62634068.pem
Adding debian:QuoVadis_Root_CA_2.pem
Adding debian:GTS_Root_R4.pem
Adding debian:Certigna_Root_CA.pem
Adding debian:DigiCert_Assured_ID_Root_G3.pem
Adding debian:USERTrust_RSA_Certification_Authority.pem
Adding debian:DigiCert_Trusted_Root_G4.pem
Adding debian:E-Tugra_Certification_Authority.pem
Adding debian:TeliaSonera_Root_CA_v1.pem
Adding debian:Izenpe.com.pem
Adding debian:DST_Root_CA_X3.pem
Adding debian:Buypass_Class_2_Root_CA.pem
Adding debian:SecureSign_RootCA11.pem
Adding debian:Certigna.pem
Adding debian:Starfield_Services_Root_Certificate_Authority_-_G2.pem
Adding debian:Trustwave_Global_ECC_P256_Certification_Authority.pem
Adding debian:Entrust_Root_Certification_Authority.pem
Adding debian:D-TRUST_Root_Class_3_CA_2_EV_2009.pem
Adding debian:emSign_Root_CA_-_G1.pem
Adding debian:COMODO_RSA_Certification_Authority.pem
Adding debian:T-TeleSec_GlobalRoot_Class_3.pem
Adding debian:Amazon_Root_CA_2.pem
Adding debian:GTS_Root_R3.pem
Adding debian:GlobalSign_Root_CA.pem
Adding debian:Chambers_of_Commerce_Root_-_2008.pem
Adding debian:SZAFIR_ROOT_CA2.pem
Adding debian:Microsoft_ECC_Root_Certificate_Authority_2017.pem
Adding debian:AC_RAIZ_FNMT-RCM.pem
Adding debian:certSIGN_ROOT_CA.pem
Adding debian:Trustwave_Global_Certification_Authority.pem
Adding debian:EC-ACC.pem
Adding debian:Microsoft_RSA_Root_Certificate_Authority_2017.pem
Adding debian:XRamp_Global_CA_Root.pem
Adding debian:Trustwave_Global_ECC_P384_Certification_Authority.pem
Adding debian:QuoVadis_Root_CA.pem
Adding debian:GeoTrust_Primary_Certification_Authority_-_G2.pem
Adding debian:DigiCert_Global_Root_G2.pem
Adding debian:Hongkong_Post_Root_CA_3.pem
Adding debian:AffirmTrust_Premium.pem
Adding debian:IdenTrust_Public_Sector_Root_CA_1.pem
Adding debian:COMODO_ECC_Certification_Authority.pem
Adding debian:DigiCert_High_Assurance_EV_Root_CA.pem
Adding debian:TWCA_Root_Certification_Authority.pem
Adding debian:Go_Daddy_Root_Certificate_Authority_-_G2.pem
Adding debian:D-TRUST_Root_Class_3_CA_2_2009.pem
Adding debian:T-TeleSec_GlobalRoot_Class_2.pem
Adding debian:NetLock_Arany_=Class_Gold=_Főtanúsítvány.pem
Adding debian:ACCVRAIZ1.pem
Adding debian:ePKI_Root_Certification_Authority.pem
Adding debian:TrustCor_RootCert_CA-2.pem
Adding debian:Staat_der_Nederlanden_Root_CA_-_G3.pem
done.
Setting up default-jre-headless (2:1.11-72) ...
Processing triggers for libc-bin (2.31-13+deb11u5) ...
Processing triggers for man-db (2.9.4-2) ...
Processing triggers for ca-certificates (20210119) ...
Updating certificates in /etc/ssl/certs...
0 added, 0 removed; done.
Running hooks in /etc/ca-certificates/update.d...

done.
done.
Setting up openjdk-11-jre-headless:amd64 (11.0.18+10-1~deb11u1) ...
update-alternatives: using /usr/lib/jvm/java-11-openjdk-amd64/bin/java to provide /usr/bin/java (java) in auto mode
update-alternatives: using /usr/lib/jvm/java-11-openjdk-amd64/bin/jjs to provide /usr/bin/jjs (jjs) in auto mode
update-alternatives: using /usr/lib/jvm/java-11-openjdk-amd64/bin/keytool to provide /usr/bin/keytool (keytool) in auto mode
update-alternatives: using /usr/lib/jvm/java-11-openjdk-amd64/bin/rmid to provide /usr/bin/rmid (rmid) in auto mode
update-alternatives: using /usr/lib/jvm/java-11-openjdk-amd64/bin/rmiregistry to provide /usr/bin/rmiregistry (rmiregistry) in auto mode
update-alternatives: using /usr/lib/jvm/java-11-openjdk-amd64/bin/pack200 to provide /usr/bin/pack200 (pack200) in auto mode
update-alternatives: using /usr/lib/jvm/java-11-openjdk-amd64/bin/unpack200 to provide /usr/bin/unpack200 (unpack200) in auto mode
update-alternatives: using /usr/lib/jvm/java-11-openjdk-amd64/lib/jexec to provide /usr/bin/jexec (jexec) in auto mode
student-01-cd9de465be5c@mc-server:~$ 
```

```sh
student-01-cd9de465be5c@mc-server:~$ cd /home/minecraft
student-01-cd9de465be5c@mc-server:/home/minecraft$ sudo apt-get install wget
Reading package lists... Done
Building dependency tree... Done
Reading state information... Done
The following NEW packages will be installed:
  wget
0 upgraded, 1 newly installed, 0 to remove and 6 not upgraded.
Need to get 964 kB of archives.
After this operation, 3559 kB of additional disk space will be used.
Get:1 http://deb.debian.org/debian bullseye/main amd64 wget amd64 1.21-1+deb11u1 [964 kB]
Fetched 964 kB in 0s (3705 kB/s)
Selecting previously unselected package wget.
(Reading database ... 56788 files and directories currently installed.)
Preparing to unpack .../wget_1.21-1+deb11u1_amd64.deb ...
Unpacking wget (1.21-1+deb11u1) ...
Setting up wget (1.21-1+deb11u1) ...
Processing triggers for man-db (2.9.4-2) ...
student-01-cd9de465be5c@mc-server:/home/minecraft$ 
```

```sh
student-01-cd9de465be5c@mc-server:/home/minecraft$ sudo wget https://launcher.mojang.com/v1/objects/d0d0fe2b1dc6ab4c65554cb734270872b72dadd6/server.jar
--2023-04-10 02:26:00--  https://launcher.mojang.com/v1/objects/d0d0fe2b1dc6ab4c65554cb734270872b72dadd6/server.jar
Resolving launcher.mojang.com (launcher.mojang.com)... 13.107.237.38, 13.107.238.38, 2620:1ec:4e:1::38, ...
Connecting to launcher.mojang.com (launcher.mojang.com)|13.107.237.38|:443... connected.
HTTP request sent, awaiting response... 200 OK
Length: 35952401 (34M) [application/zip]
Saving to: ‘server.jar’

server.jar                  100%[==========================================>]  34.29M  37.2MB/s    in 0.9s    

2023-04-10 02:26:01 (37.2 MB/s) - ‘server.jar’ saved [35952401/35952401]

student-01-cd9de465be5c@mc-server:/home/minecraft$ 
```

```sh
student-01-cd9de465be5c@mc-server:/home/minecraft$ sudo java -Xmx1024M -Xms1024M -jar server.jar nogui
[02:26:31] [main/ERROR]: Failed to load properties from file: server.properties
[02:26:32] [main/WARN]: Failed to load eula.txt
[02:26:32] [main/INFO]: You need to agree to the EULA in order to run the server. Go to eula.txt for more info.
student-01-cd9de465be5c@mc-server:/home/minecraft$ 
```

```sh
student-01-cd9de465be5c@mc-server:/home/minecraft$ sudo ls -l
total 35140
-rw-r--r-- 1 root root      181 Apr 10 02:26 eula.txt
drwxr-xr-x 2 root root     4096 Apr 10 02:26 logs
drwx------ 2 root root    16384 Apr 10 02:22 lost+found
-rw-r--r-- 1 root root 35952401 Apr 12  2022 server.jar
-rw-r--r-- 1 root root      912 Apr 10 02:26 server.properties
student-01-cd9de465be5c@mc-server:/home/minecraft$ sudo nano eula.txt
student-01-cd9de465be5c@mc-server:/home/minecraft$ cat eula.txt 
#By changing the setting below to TRUE you are indicating your agreement to our EULA (https://account.mojang.com/documents/minecraft_eula).
#Mon Apr 10 02:26:32 UTC 2023
eula=true
student-01-cd9de465be5c@mc-server:/home/minecraft$ 
```

```sh
student-01-cd9de465be5c@mc-server:/home/minecraft$ sudo apt-get install -y screen
Reading package lists... Done
Building dependency tree... Done
Reading state information... Done
The following additional packages will be installed:
  libutempter0
Suggested packages:
  byobu | screenie | iselect ncurses-term
The following NEW packages will be installed:
  libutempter0 screen
0 upgraded, 2 newly installed, 0 to remove and 6 not upgraded.
Need to get 618 kB of archives.
After this operation, 1089 kB of additional disk space will be used.
Get:1 http://deb.debian.org/debian bullseye/main amd64 libutempter0 amd64 1.2.1-2 [8960 B]
Get:2 http://deb.debian.org/debian bullseye/main amd64 screen amd64 4.8.0-6 [609 kB]
Fetched 618 kB in 0s (2193 kB/s)
Selecting previously unselected package libutempter0:amd64.
(Reading database ... 56876 files and directories currently installed.)
Preparing to unpack .../libutempter0_1.2.1-2_amd64.deb ...
Unpacking libutempter0:amd64 (1.2.1-2) ...
Selecting previously unselected package screen.
Preparing to unpack .../screen_4.8.0-6_amd64.deb ...
Unpacking screen (4.8.0-6) ...
Setting up libutempter0:amd64 (1.2.1-2) ...
Setting up screen (4.8.0-6) ...
Processing triggers for libc-bin (2.31-13+deb11u5) ...
Processing triggers for man-db (2.9.4-2) ...
student-01-cd9de465be5c@mc-server:/home/minecraft$ 
```

```sh
^[[A^[[A^[[A^[[A^[[A^[[A^[[A^[[A^[[A^[[A^[[A^[[A^[[A^[[A^[[A^[[A^[[A^[[A^[[A^[[A^[[A[02:29:17] [main/WARN]: Ambiguity between arguments [teleport, destination] and [teleport, targets] with inputs: [Player, 0123, @e, dd12be42-52a9-4a91-a8a1-11c01849e498]
[02:29:17] [main/WARN]: Ambiguity between arguments [teleport, location] and [teleport, destination] with inputs: [0.1 -0.5 .9, 0 0 0]
[02:29:17] [main/WARN]: Ambiguity between arguments [teleport, location] and [teleport, targets] with inputs: [0.1 -0.5 .9, 0 0 0]
[02:29:17] [main/WARN]: Ambiguity between arguments [teleport, targets] and [teleport, destination] with inputs: [Player, 0123, dd12be42-52a9-4a91-a8a1-11c01849e498]
[02:29:17] [main/WARN]: Ambiguity between arguments [teleport, targets, location] and [teleport, targets, destination] with inputs: [0.1 -0.5 .9, 0 0 0]
[02:29:17] [Server thread/INFO]: Starting minecraft server version 1.14.3
[02:29:17] [Server thread/INFO]: Loading properties
[02:29:17] [Server thread/INFO]: Default game type: SURVIVAL
[02:29:17] [Server thread/INFO]: Generating keypair
[02:29:17] [Server thread/INFO]: Starting Minecraft server on *:25565
[02:29:18] [Server thread/INFO]: Using epoll channel type
[02:29:18] [Server thread/INFO]: Preparing level "world"
[02:29:18] [Server thread/INFO]: Found new data pack vanilla, loading it automatically
[02:29:18] [Server thread/INFO]: Reloading ResourceManager: Default
```

**Congratulations!** You set up and customized a VM and installed and configured application software—a Minecraft server!



```sh
gcloud compute --project=qwiklabs-gcp-03-2b0cb03f1121 firewall-rules create minecraft-rule --direction=INGRESS --priority=1000 --network=default --action=ALLOW --rules=tcp:25565 --source-ranges=0.0.0.0/0 --target-tags=minecraft-server
```

```sh
student-01-cd9de465be5c@mc-server:~$ export YOUR_BUCKET_NAME=pradeepgadde
student-01-cd9de465be5c@mc-server:~$ echo $YOUR_BUCKET_NAME
pradeepgadde
student-01-cd9de465be5c@mc-server:~$ 

student-01-cd9de465be5c@mc-server:~$ gsutil mb gs://$YOUR_BUCKET_NAME-minecraft-backup
Creating gs://pradeepgadde-minecraft-backup/...
student-01-cd9de465be5c@mc-server:~$ 
```

```sh
student-01-cd9de465be5c@mc-server:~$ cd /home/minecraft
student-01-cd9de465be5c@mc-server:/home/minecraft$ sudo nano /home/minecraft/backup.sh
student-01-cd9de465be5c@mc-server:/home/minecraft$ cat /home/minecraft/backup.sh 
#!/bin/bash
screen -r mcs -X stuff '/save-all\n/save-off\n'
/usr/bin/gsutil cp -R ${BASH_SOURCE%/*}/world gs://${YOUR_BUCKET_NAME}-minecraft-backup/$(date "+%Y%m%d-%H%M%S")-world
screen -r mcs -X stuff '/save-on\n'
student-01-cd9de465be5c@mc-server:/home/minecraft$ 
student-01-cd9de465be5c@mc-server:/home/minecraft$ sudo chmod 755 /home/minecraft/backup.sh
student-01-cd9de465be5c@mc-server:/home/minecraft$ 
```

```sh
student-01-cd9de465be5c@mc-server:/home/minecraft$ . /home/minecraft/backup.sh
No screen session found.
Copying file:///home/minecraft/world/level.dat [Content-Type=application/octet-stream]...
Copying file:///home/minecraft/world/session.lock [Content-Type=application/octet-stream]...
Copying file:///home/minecraft/world/data/raids.dat [Content-Type=application/octet-stream]...
Copying file:///home/minecraft/world/region/r.-1.-1.mca [Content-Type=application/octet-stream]...
- [4 files][  2.0 MiB/  2.0 MiB]                                                
==> NOTE: You are performing a sequence of gsutil operations that may
run significantly faster if you instead use gsutil -m cp ... Please
see the -m section under "gsutil help options" for further information
about when gsutil -m can be advantageous.

Copying file:///home/minecraft/world/region/r.-1.0.mca [Content-Type=application/octet-stream]...
Copying file:///home/minecraft/world/region/r.0.-1.mca [Content-Type=application/octet-stream]...
Copying file:///home/minecraft/world/region/r.0.0.mca [Content-Type=application/octet-stream]...
Copying file:///home/minecraft/world/DIM1/data/raids_end.dat [Content-Type=application/octet-stream]...
Copying file:///home/minecraft/world/poi/r.0.0.mca [Content-Type=application/octet-stream]...
Copying file:///home/minecraft/world/DIM-1/data/raids_nether.dat [Content-Type=application/octet-stream]...
\ [10 files][  3.9 MiB/  3.9 MiB]                                               
Operation completed over 10 objects/3.9 MiB.                                     
No screen session found.
student-01-cd9de465be5c@mc-server:/home/minecraft$ 
```

Now that you've verified that the backups are working, you can schedule a cron job to automate the task.

```sh
student-01-cd9de465be5c@mc-server:/home/minecraft$ sudo crontab -e
no crontab for root - using an empty one

Select an editor.  To change later, run 'select-editor'.
  1. /bin/nano        <---- easiest
  2. /usr/bin/vim.basic
  3. /usr/bin/vim.tiny

Choose 1-3 [1]: 1
crontab: installing new crontab
student-01-cd9de465be5c@mc-server:/home/minecraft$
```

```sh
student-01-cd9de465be5c@mc-server:/home/minecraft$ sudo screen -r -X stuff '/stop\n'
There is a screen on:
        3281.mcs        (04/10/23 02:29:09)     (Attached)
No screen session found.
student-01-cd9de465be5c@mc-server:/home/minecraft$ 
```

### Automate server maintenance with startup and shutdown scripts

Instead of following the manual process to mount the persistent disk  and launch the server application in a screen, you can use metadata  scripts to create a startup script and a shutdown script to do this for  you.

In this lab, you created a customized virtual machine instance by  installing base software (a headless JRE) and application software (a  Minecraft game server). You customized the VM by attaching and preparing a high-speed SSD data disk, and you reserved a static external IP so  the address would remain consistent. Then you verified availability of  the gaming server online.

You set up a backup system to back up the server's data to a Cloud  Storage bucket, and you tested the backup system. Then you automated  backups using cron. Finally, you set up maintenance scripts using  metadata for graceful startup and shutdown of the server.

![]({{ site.url }}{{ site.baseurl }}/assets/images/minecraft-1.png)

![]({{ site.url }}{{ site.baseurl }}/assets/images/minecraft-2.png)

![]({{ site.url }}{{ site.baseurl }}/assets/images/minecraft-3.png)

![]({{ site.url }}{{ site.baseurl }}/assets/images/minecraft-4.png)

![]({{ site.url }}{{ site.baseurl }}/assets/images/minecraft-5.png)

![]({{ site.url }}{{ site.baseurl }}/assets/images/minecraft-6.png)

![]({{ site.url }}{{ site.baseurl }}/assets/images/minecraft-7.png)

![]({{ site.url }}{{ site.baseurl }}/assets/images/minecraft-8.png)

![]({{ site.url }}{{ site.baseurl }}/assets/images/minecraft-9.png)





