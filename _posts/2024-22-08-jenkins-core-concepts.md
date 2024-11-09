---
layout: single
title:  "Jenkins Core Concepts"
categories: Jenkins  Automation
tags: Jenkins
author: "Pradeep Gadde"
classes: wide
show_date: true

header:
  overlay_image: /assets/images/jenkins.svg
  og_image: /assets/images/jenkins.svg
  caption: "Photo credit: [**Jenkins**](https://jenkins.io)"
  teaser: /assets/images/jenkins.svg
  actions:
    - label: "Learn more"
      url: "https://jenkins.io"
author:
  name     : "Jenkins"
  avatar   : "/assets/images/jenkins.svg"

sidebar:
  - title: "Blog"
    
    text: "Checkout other topics"
    nav: my-sidebar
---



## Jenkins

Getting started with Essentials of Jenkins.

- Jobs
- Builds
- Freestyle Project
- Pipelines
- Stages
- Nodes
- Plugins



## Continuous Integration

Main > Feature Branch A > Commit > Pull Request > Review Approve > Merge > Manual Deploy > Production

Main > Feature Branch B > Commit > Pull Request > Review Approve > Merge > Manual Deploy > Production



Without CI, testing typically occurs later in the development cycle, mostly after multiple merges

Deploying code to staging and production relies on manual processes

With Continuous Integration, 

- Unit Testing

- Dependecny Scan

- Build Artifact

- Code Scanning (Vulenrability scanning)

  

Main > Feature Branch A > Commit > Pull Request  > Review Approve > CI > Merge > CI > Manual Deploy > Production

Main > Feature Branch B > Commit > Pull Request > Review Approve > CI > Merge > CI > Manual Deploy > Production

## Continuous Deployment/ Continous Delivery

Main > Feature Branch A > Commit > Pull Request  > Review Approve > CI >  CD > Merge > CI > CD (Automatic Deployment) > Production

Main > Feature Branch B > Commit > Pull Request > Review Approve > CI  > CD > Merge > CI > CD (Automatic Deployment) > Production

In Continuous Delivery, manual approval required as a safety net before deploying (automatically) to Production.



## Jenkins Architecture

Controller Node

Worker Node

In basic deployments single node acting as both controller and worker nodes

In advanced deployments, separate controller and worker nodes (Linux/Windows)

Nodes communicate using SSH/JNLP (Java Network Launch Protocol)

Executors

Agents

## Jenkins Installation

Requires JRE / JDK

Otherwise, installation fails with error : failed to find a valid java installation

JENKINS_HOME : /var/lib/jenkins

`java -version`

`systemctl status jenkins`

`/var/lib/jenkins/secrets/initialAdminPassword`

# macOS Installers for Jenkins LTS

Jenkins can be installed using the [ Homebrew ](https://brew.sh/) package manager. Homebrew formula: [ jenkins-lts ](https://formulae.brew.sh/formula/jenkins-lts) This is a package supported by a third party which may be not as  frequently updated as packages supported by the Jenkins project  directly.

Sample commands:

- Install the latest LTS version: `brew install jenkins-lts`
- Start the Jenkins service: `brew services start jenkins-lts`
- Restart the Jenkins service: `brew services restart jenkins-lts`
- Update the Jenkins version: `brew upgrade jenkins-lts`

After starting the Jenkins service, browse to http://localhost:8080 and follow the instructions to complete the installation.

```sh
(base) pradeep:~$pwd
/Users/pradeep
(base) pradeep:~$brew install jenkins-lts
==> Auto-updating Homebrew...
Adjust how often this is run with HOMEBREW_AUTO_UPDATE_SECS or disable with
HOMEBREW_NO_AUTO_UPDATE. Hide these hints with HOMEBREW_NO_ENV_HINTS (see `man brew`).
==> Auto-updated Homebrew!
Updated 3 taps (hashicorp/tap, homebrew/core and homebrew/cask).
==> New Formulae
azqr                      kanata                    mkdocs-material           toml2json
decasify                  libtatsu                  pytest
==> New Casks
acronis-true-image-cleanup-tool    font-eldur                         lunatask
bobhelper                          font-recursive-desktop             neohtop
default-handler                    hyperconnect                       viz
djuced                             jet-pilot                          whimsical

You have 26 outdated formulae and 2 outdated casks installed.

==> Downloading https://ghcr.io/v2/homebrew/core/jenkins-lts/manifests/2.479.1
################################################################################################# 100.0%
==> Fetching dependencies for jenkins-lts: libpng, freetype, giflib, fontconfig, pcre2, python-packaging, mpdecimal, ca-certificates, openssl@3, readline, sqlite, xz, python@3.13, libunistring, gettext, glib, xorgproto, libxau, libxdmcp, libxcb, libx11, libxext, libxrender, lzo, pixman, cairo, graphite2, icu4c@76, harfbuzz, jpeg-turbo, lz4, zstd, libtiff, little-cms2 and openjdk@21
==> Downloading https://ghcr.io/v2/homebrew/core/libpng/manifests/1.6.44
################################################################################################# 100.0%

{snip}

==> Caveats
==> jenkins-lts
Note: When using launchctl the port will be 8080.

To start jenkins-lts now and restart at login:
  brew services start jenkins-lts
Or, if you don't want/need a background service you can just run:
  /usr/local/opt/openjdk@21/bin/java -Dmail.smtp.starttls.enable\=true -jar /usr/local/opt/jenkins-lts/libexec/jenkins.war --httpListenAddress\=127.0.0.1 --httpPort\=8080
==> azure-cli
zsh completions have been installed to:
  /usr/local/share/zsh/site-functions
(base) pradeep:~$

```



```sh
(base) pradeep:~$ brew services start jenkins-lts
==> Tapping homebrew/services
Cloning into '/usr/local/Homebrew/Library/Taps/homebrew/homebrew-services'...
remote: Enumerating objects: 3559, done.
remote: Counting objects: 100% (706/706), done.
remote: Compressing objects: 100% (280/280), done.
remote: Total 3559 (delta 488), reused 564 (delta 421), pack-reused 2853 (from 1)
Receiving objects: 100% (3559/3559), 1.02 MiB | 4.54 MiB/s, done.
Resolving deltas: 100% (1729/1729), done.
Tapped 2 commands (52 files, 1.2MB).
==> Successfully started `jenkins-lts` (label: homebrew.mxcl.jenkins-lts)
(base) pradeep:~$

```

# Unlock Jenkins

To ensure Jenkins is securely set up by the administrator, a password has been written to the log ([not sure where to find it?](https://www.jenkins.io/redirect/find-jenkins-logs)) and this file on the server: 

```
/Users/pradeep/.jenkins/secrets/initialAdminPassword
```

Please copy the password from either location and paste it below.

```sh
(base) pradeep:~$cat /Users/pradeep/.jenkins/secrets/initialAdminPassword
484beebaf73f43be85306751773db9b6
(base) pradeep:~$
```

# Customize Jenkins

​			Plugins extend Jenkins with additional features to support many different needs. 	

## Build Steps

```sh
curl -s https://api.adviceslip.com/advice > advice.json
cat advice.json

/Users/pradeep/opt/anaconda3/bin/jq -r .slip.advice < advice.json > advice.message
[ $(wc -w < advice.message) -gt 5 ] && echo "Advice has more than 5 words" || (echo "Advice - $(cat advice.message) has 5 words" && exit 1)

cat advice.message | /usr/local/bin/cowsay
```



```sh
12:03:59 Started by user Pradeep Gadde

12:03:59 Running as SYSTEM
12:03:59 Building in workspace /Users/pradeep/.jenkins/workspace/Generate ASCII Artwork
12:03:59 [Generate ASCII Artwork] $ /bin/sh -xe /var/folders/cf/vzmh318x285f0c1sbsnm14m40000gn/T/jenkins8531345099182991739.sh
12:03:59 + curl -s https://api.adviceslip.com/advice
12:03:59 + cat advice.json
12:03:59 {"slip": { "id": 77, "advice": "Mercy is the better part of justice."}}+ /Users/pradeep/opt/anaconda3/bin/jq -r .slip.advice
12:03:59 ++ wc -w
12:03:59 + '[' 7 -gt 5 ']'
12:03:59 + echo 'Advice has more than 5 words'
12:03:59 Advice has more than 5 words
12:03:59 + cat advice.message
12:03:59 + /usr/local/bin/cowsay
12:04:00  ______________________________________ 
12:04:00 < Mercy is the better part of justice. >
12:04:00  -------------------------------------- 
12:04:00         \   ^__^
12:04:00          \  (oo)\_______
12:04:00             (__)\       )\/\
12:04:00                 ||----w |
12:04:00                 ||     ||
12:04:03 Finished: SUCCESS

```

