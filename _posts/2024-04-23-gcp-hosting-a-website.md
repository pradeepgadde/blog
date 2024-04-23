---
layout: single
title:  "Hosting a Web App on Google Cloud Using Compute Engine"
categories: Cloud
tags: GCP
classes: wide
show_date: true
header:
  teaser: /assets/images/pca-gcp.png
author:
  name     : "Professional Cloud Architect"
  avatar   : "/assets/images/pca-gcp.png"

sidebar:
  - title: "Topics"
    nav: my-sidebar

---

# Hosting a Web App on Google Cloud Using Compute Engine

- Create [Compute Engine instances](https://cloud.google.com/compute/docs/instances/)

- Create [instance templates](https://cloud.google.com/compute/docs/instance-templates/) from source instances

- Create [managed instance groups](https://cloud.google.com/compute/docs/instance-groups/)

- Create and test [managed instance group health checks](https://cloud.google.com/compute/docs/instance-groups/autohealing-instances-in-migs)

- Create HTTP(S) [Load Balancers](https://cloud.google.com/load-balancing/)

- Create [load balancer health checks](https://cloud.google.com/load-balancing/docs/health-checks)

- Use a [Content Delivery Network (CDN)](https://cloud.google.com/cdn/) for Caching


At the end, you will have instances inside managed instance  groups to provide autohealing, load balancing, autoscaling, and rolling  updates for your website.

Run the following gcloud commands in Cloud Console to set the default region and zone for your lab:

```sh
Welcome to Cloud Shell! Type "help" to get started.
Your Cloud Platform project in this session is set to qwiklabs-gcp-04-83eb8e6fd864.
Use “gcloud config set project [PROJECT_ID]” to change to a different project.
student_01_cad4a58911a5@cloudshell:~ (qwiklabs-gcp-04-83eb8e6fd864)$ gcloud config set compute/zone "us-west1-a"
export ZONE=$(gcloud config get compute/zone)

gcloud config set compute/region "us-west1"
export REGION=$(gcloud config get compute/region)
WARNING: Property validation for compute/zone was skipped.
Updated property [compute/zone].
Your active configuration is: [cloudshell-28083]
WARNING: Property validation for compute/region was skipped.
Updated property [compute/region].
Your active configuration is: [cloudshell-28083]
student_01_cad4a58911a5@cloudshell:~ (qwiklabs-gcp-04-83eb8e6fd864)$ 
```

## Enable Compute Engine API

```sh
student_01_cad4a58911a5@cloudshell:~ (qwiklabs-gcp-04-83eb8e6fd864)$ gcloud services enable compute.googleapis.com
student_01_cad4a58911a5@cloudshell:~ (qwiklabs-gcp-04-83eb8e6fd864)$ 
```

## Create Cloud Storage bucket

You will use a Cloud Storage bucket to house your built code as well as your startup scripts.

- From Cloud Shell, execute the following to create a new Cloud Storage bucket:

  ```sh
  student_01_cad4a58911a5@cloudshell:~ (qwiklabs-gcp-04-83eb8e6fd864)$ gsutil mb gs://fancy-store-$DEVSHELL_PROJECT_ID
  Creating gs://fancy-store-qwiklabs-gcp-04-83eb8e6fd864/...
  student_01_cad4a58911a5@cloudshell:~ (qwiklabs-gcp-04-83eb8e6fd864)$ 
  ```

  > Use of the `$DEVSHELL_PROJECT_ID`  environment variable within Cloud Shell is to help ensure the names of  objects are unique. Since all Project IDs within Google Cloud must be  unique, appending the Project ID should make other names unique as well.

## Clone source repository

Use the existing Fancy Store ecommerce website based on the `monolith-to-microservices` repository as the basis for your website.

Clone the source code so you can focus on the aspects of deploying to Compute Engine. Later on in this lab, you will perform a small update  to the code to demonstrate the simplicity of updating on Compute Engine.

Clone the source code and then navigate to the `monolith-to-microservices` directory:

```sh
student_01_cad4a58911a5@cloudshell:~ (qwiklabs-gcp-04-83eb8e6fd864)$ git clone https://github.com/googlecodelabs/monolith-to-microservices.git
Cloning into 'monolith-to-microservices'...
remote: Enumerating objects: 1136, done.
remote: Counting objects: 100% (241/241), done.
remote: Compressing objects: 100% (119/119), done.
remote: Total 1136 (delta 184), reused 142 (delta 122), pack-reused 895
Receiving objects: 100% (1136/1136), 3.74 MiB | 19.24 MiB/s, done.
Resolving deltas: 100% (533/533), done.
student_01_cad4a58911a5@cloudshell:~ (qwiklabs-gcp-04-83eb8e6fd864)$ cd ~/monolith-to-microservices
student_01_cad4a58911a5@cloudshell:~/monolith-to-microservices (qwiklabs-gcp-04-83eb8e6fd864)$ 
```

Run the initial build of the code to allow the application to run locally:

```sh
student_01_cad4a58911a5@cloudshell:~/monolith-to-microservices (qwiklabs-gcp-04-83eb8e6fd864)$ ./setup.sh
Installing monolith dependencies...

added 59 packages, and audited 60 packages in 2s

7 packages are looking for funding
  run `npm fund` for details

1 moderate severity vulnerability

To address all issues, run:
  npm audit fix

Run `npm audit` for details.
Completed.

Installing microservices dependencies...

added 87 packages, and audited 88 packages in 10s

14 packages are looking for funding
  run `npm fund` for details

1 moderate severity vulnerability

To address all issues, run:
  npm audit fix

Run `npm audit` for details.
Completed.

Installing React app dependencies...
npm WARN deprecated source-map-url@0.4.1: See https://github.com/lydell/source-map-url#deprecated
npm WARN deprecated svgo@1.3.2: This SVGO version is no longer supported. Upgrade to v2.x.x.

added 1476 packages, and audited 1477 packages in 30s

202 packages are looking for funding
  run `npm fund` for details

16 vulnerabilities (8 moderate, 7 high, 1 critical)

To address issues that do not require attention, run:
  npm audit fix

To address all issues (including breaking changes), run:
  npm audit fix --force

Run `npm audit` for details.
Completed.

Building React app and placing into sub projects...

> frontend@0.1.0 prebuild
> npm run build:monolith


> frontend@0.1.0 build:monolith
> env-cmd -f .env.monolith react-scripts build

Creating an optimized production build...
Browserslist: caniuse-lite is outdated. Please run:
  npx update-browserslist-db@latest
  Why you should do it regularly: https://github.com/browserslist/update-db#readme
Compiled successfully.

File sizes after gzip:

  93.04 kB  build/static/js/main.f58ec764.js

The project was built assuming it is hosted at /.
You can control this with the homepage field in your package.json.

The build folder is ready to be deployed.
You may serve it with a static server:

  npm install -g serve
  serve -s build

Find out more about deployment here:

  https://cra.link/deployment


> frontend@0.1.0 postbuild:monolith
> node scripts/post-build.js ./build ../monolith/public

Deleting stale folder: ../monolith/public
Deleted stale destination folder: ../monolith/public
Copying files from ./build to ../monolith/public
Copied ./build to ../monolith/public successfully!

> frontend@0.1.0 build
> react-scripts build

Creating an optimized production build...
Browserslist: caniuse-lite is outdated. Please run:
  npx update-browserslist-db@latest
  Why you should do it regularly: https://github.com/browserslist/update-db#readme
Compiled successfully.

File sizes after gzip:

  93.05 kB (+18 B)  build/static/js/main.3f3d4bec.js

The project was built assuming it is hosted at /.
You can control this with the homepage field in your package.json.

The build folder is ready to be deployed.
You may serve it with a static server:

  npm install -g serve
  serve -s build

Find out more about deployment here:

  https://cra.link/deployment


> frontend@0.1.0 postbuild
> node scripts/post-build.js ./build ../microservices/src/frontend/public

Deleting stale folder: ../microservices/src/frontend/public
Deleted stale destination folder: ../microservices/src/frontend/public
Copying files from ./build to ../microservices/src/frontend/public
Copied ./build to ../microservices/src/frontend/public successfully!
Completed.

Setup completed successfully!
student_01_cad4a58911a5@cloudshell:~/monolith-to-microservices (qwiklabs-gcp-04-83eb8e6fd864)$ 
```

Once completed, ensure Cloud Shell is running a compatible nodeJS version with the following command:

```sh
tudent_01_cad4a58911a5@cloudshell:~/monolith-to-microservices (qwiklabs-gcp-04-83eb8e6fd864)$ nvm install --lts
Installing latest LTS version.
v20.12.2 is already installed.
Now using node v20.12.2 (npm v10.5.2)
student_01_cad4a58911a5@cloudshell:~/monolith-to-microservices (qwiklabs-gcp-04-83eb8e6fd864)$ 
```

Next, run the following to test the application, switch to the `microservices` directory, and start the web server:

```sh
student_01_cad4a58911a5@cloudshell:~/monolith-to-microservices (qwiklabs-gcp-04-83eb8e6fd864)$ cd microservices
npm start

> microservices@1.0.0 start
> concurrently "npm run frontend" "npm run products" "npm run orders"

[1] 
[1] > microservices@1.0.0 products
[1] > node ./src/products/server.js
[1] 
[2] 
[2] > microservices@1.0.0 orders
[2] > node ./src/orders/server.js
[2] 
[0] 
[0] > microservices@1.0.0 frontend
[0] > node ./src/frontend/server.js
[0] 
[1] Products microservice listening on port 8082!
[2] Orders microservice listening on port 8081!
[0] Frontend microservice listening on port 8080!

```

> **Note:** Within the Preview option, you should be able to  see the Frontend; however, the Products and Orders functions will not  work, as those services are not yet exposed.



```sh
^C[0] npm run frontend exited with code SIGINT
[1] npm run products exited with code SIGINT
[2] npm run orders exited with code SIGINT
student_01_cad4a58911a5@cloudshell:~/monolith-to-microservices/microservices (qwiklabs-gcp-04-83eb8e6fd864)$ 
```

## Create Compute Engine instances

Now it's time to start deploying some Compute Engine instances!

In the following steps you will:

1. Create a startup script to configure instances.
2. Clone source code and upload to Cloud Storage.
3. Deploy a Compute Engine instance to host the backend microservices.
4. Reconfigure the frontend code to utilize the backend microservices instance.
5. Deploy a Compute Engine instance to host the frontend microservice.
6. Configure the network to allow communication.

### Create the startup script

A startup script will be used to instruct the instance what to do  each time it is started. This way the instances are automatically  configured.

1. In Cloud Shell, run the following command to create a file called `startup-script.sh`:

   ```sh
   student_01_cad4a58911a5@cloudshell:~/monolith-to-microservices/microservices (qwiklabs-gcp-04-83eb8e6fd864)$ cat ~/monolith-to-microservices/startup-script.sh
   #!/bin/bash
   
   # Install logging monitor. The monitor will automatically pick up logs sent to
   # syslog.
   curl -s "https://storage.googleapis.com/signals-agents/logging/google-fluentd-install.sh" | bash
   service google-fluentd restart &
   
   # Install dependencies from apt
   apt-get update
   apt-get install -yq ca-certificates git build-essential supervisor psmisc
   
   # Install nodejs
   mkdir /opt/nodejs
   curl https://nodejs.org/dist/v16.14.0/node-v16.14.0-linux-x64.tar.gz | tar xvzf - -C /opt/nodejs --strip-components=1
   ln -s /opt/nodejs/bin/node /usr/bin/node
   ln -s /opt/nodejs/bin/npm /usr/bin/npm
   
   # Get the application source code from the Google Cloud Storage bucket.
   mkdir /fancy-store
   gsutil -m cp -r gs://fancy-store-qwiklabs-gcp-04-83eb8e6fd864/monolith-to-microservices/microservices/* /fancy-store/
   
   # Install app dependencies.
   cd /fancy-store/
   npm install
   
   # Create a nodeapp user. The application will run as this user.
   useradd -m -d /home/nodeapp nodeapp
   chown -R nodeapp:nodeapp /opt/app
   
   # Configure supervisor to run the node app.
   cat >/etc/supervisor/conf.d/node-app.conf << EOF
   [program:nodeapp]
   directory=/fancy-store
   command=npm start
   autostart=true
   autorestart=true
   user=nodeapp
   environment=HOME="/home/nodeapp",USER="nodeapp",NODE_ENV="production"
   stdout_logfile=syslog
   stderr_logfile=syslog
   EOF
   
   supervisorctl reread
   supervisorctl updatestudent_01_cad4a58911a5@cloudshell:~/monolith-to-microservices/microservices (qwiklabs-gcp-04-83eb8e6fd864)$ 
   ```

   

Return to Cloud Shell Terminal and run the following to copy the `startup-script.sh` file into your bucket:

```sh
student_01_cad4a58911a5@cloudshell:~/monolith-to-microservices/microservices (qwiklabs-gcp-04-83eb8e6fd864)$ gsutil cp ~/monolith-to-microservices/startup-script.sh gs://fancy-store-$DEVSHELL_PROJECT_ID
Copying file:///home/student_01_cad4a58911a5/monolith-to-microservices/startup-script.sh [Content-Type=text/x-sh]...
/ [1 files][  1.3 KiB/  1.3 KiB]                                                
Operation completed over 1 objects/1.3 KiB.                                      
student_01_cad4a58911a5@cloudshell:~/monolith-to-microservices/microservices (qwiklabs-gcp-04-83eb8e6fd864)$ 
```

It will now be accessible at: `https://storage.googleapis.com/[BUCKET_NAME]/startup-script.sh`.

[BUCKET_NAME] represents the name of the Cloud Storage bucket. This  will only be viewable by authorized users and service accounts by  default, therefor inaccessible through a web browser. Compute Engine  instances will automatically be able to access this through their  service account.

The startup script performs the following tasks:

- Installs the Logging agent. The agent automatically collects logs from syslog.
- Installs Node.js and Supervisor. Supervisor runs the app as a daemon.
- Clones the app's source code from Cloud Storage Bucket and installs dependencies.
- Configures Supervisor to run the app. Supervisor makes sure the app  is restarted if it exits unexpectedly or is stopped by an admin or  process. It also sends the app's stdout and stderr to syslog for the  Logging agent to collect.

### Copy code into the Cloud Storage bucket

When instances launch, they pull code from the Cloud Storage bucket, so you can store some configuration variables within the `.env` file of the code.

Copy the cloned code into your bucket:

```sh
student_01_cad4a58911a5@cloudshell:~/monolith-to-microservices/microservices (qwiklabs-gcp-04-83eb8e6fd864)$ cd ~
rm -rf monolith-to-microservices/*/node_modules
gsutil -m cp -r monolith-to-microservices gs://fancy-store-$DEVSHELL_PROJECT_ID/
Copying file://monolith-to-microservices/.gitignore [Content-Type=application/octet-stream]...
Copying file://monolith-to-microservices/setup.sh [Content-Type=text/x-sh]...
Copying file://monolith-to-microservices/deploy-monolith.sh [Content-Type=text/x-sh]...
Copying file://monolith-to-microservices/CONTRIBUTING.md [Content-Type=text/markdown]...
Copying file://monolith-to-microservices/package-lock.json [Content-Type=application/json]...
Copying file://monolith-to-microservices/startup-script.sh [Content-Type=text/x-sh]...
Copying file://monolith-to-microservices/README.md [Content-Type=text/markdown]...
Copying file://monolith-to-microservices/LICENSE [Content-Type=application/octet-stream]...
Copying file://monolith-to-microservices/monolith/package.json [Content-Type=application/json]...
Copying file://monolith-to-microservices/monolith/.gitignore [Content-Type=application/octet-stream]...
Copying file://monolith-to-microservices/monolith/package-lock.json [Content-Type=application/json]...
Copying file://monolith-to-microservices/monolith/.gcloudignore [Content-Type=application/octet-stream]...
Copying file://monolith-to-microservices/monolith/Dockerfile [Content-Type=application/octet-stream]...
Copying file://monolith-to-microservices/monolith/.dockerignore [Content-Type=application/octet-stream]...
Copying file://monolith-to-microservices/monolith/data/orders.json [Content-Type=application/json]...
Copying file://monolith-to-microservices/monolith/data/products.json [Content-Type=application/json]...
Copying file://monolith-to-microservices/monolith/k8s/service.yml [Content-Type=application/octet-stream]...
Copying file://monolith-to-microservices/monolith/k8s/deployment.yml [Content-Type=application/octet-stream]...
Copying file://monolith-to-microservices/monolith/public/index.html [Content-Type=text/html]...
Copying file://monolith-to-microservices/monolith/public/asset-manifest.json [Content-Type=application/json]...
Copying file://monolith-to-microservices/monolith/public/manifest.json [Content-Type=application/json]...
Copying file://monolith-to-microservices/monolith/public/robots.txt [Content-Type=text/plain]...
Copying file://monolith-to-microservices/monolith/public/static/img/products/record-player.jpg [Content-Type=image/jpeg]...
Copying file://monolith-to-microservices/monolith/public/static/img/products/film-camera.jpg [Content-Type=image/jpeg]...
Copying file://monolith-to-microservices/monolith/public/static/img/products/air-plant.jpg [Content-Type=image/jpeg]...
Copying file://monolith-to-microservices/monolith/public/static/img/products/credits.txt [Content-Type=text/plain]...
Copying file://monolith-to-microservices/monolith/public/static/img/products/barista-kit.jpg [Content-Type=image/jpeg]...
Copying file://monolith-to-microservices/monolith/public/static/img/products/typewriter.jpg [Content-Type=image/jpeg]...
Copying file://monolith-to-microservices/monolith/public/static/img/products/terrarium.jpg [Content-Type=image/jpeg]...
Copying file://monolith-to-microservices/monolith/public/static/img/products/camp-mug.jpg [Content-Type=image/jpeg]...
Copying file://monolith-to-microservices/monolith/public/static/img/products/camera-lens.jpg [Content-Type=image/jpeg]...
Copying file://monolith-to-microservices/monolith/public/static/img/products/city-bike.jpg [Content-Type=image/jpeg]...
Copying file://monolith-to-microservices/monolith/public/static/js/main.f58ec764.js [Content-Type=text/javascript]...
Copying file://monolith-to-microservices/monolith/public/static/js/main.f58ec764.js.LICENSE.txt [Content-Type=text/plain]...
Copying file://monolith-to-microservices/monolith/public/static/js/main.f58ec764.js.map [Content-Type=application/octet-stream]...
Copying file://monolith-to-microservices/monolith/src/server.js [Content-Type=text/javascript]...
Copying file://monolith-to-microservices/microservices/package.json [Content-Type=application/json]...
Copying file://monolith-to-microservices/microservices/.gitignore [Content-Type=application/octet-stream]...
Copying file://monolith-to-microservices/microservices/package-lock.json [Content-Type=application/json]...
Copying file://monolith-to-microservices/microservices/src/products/package.json [Content-Type=application/json]...
Copying file://monolith-to-microservices/microservices/src/products/.gitignore [Content-Type=application/octet-stream]...
Copying file://monolith-to-microservices/microservices/src/products/package-lock.json [Content-Type=application/json]...
Copying file://monolith-to-microservices/microservices/src/products/Dockerfile [Content-Type=application/octet-stream]...
Copying file://monolith-to-microservices/microservices/src/products/server.js [Content-Type=text/javascript]...
Copying file://monolith-to-microservices/microservices/src/products/.dockerignore [Content-Type=application/octet-stream]...
Copying file://monolith-to-microservices/microservices/src/products/data/products.json [Content-Type=application/json]...
Copying file://monolith-to-microservices/microservices/src/products/k8s/service.yml [Content-Type=application/octet-stream]...
Copying file://monolith-to-microservices/microservices/src/products/k8s/deployment.yml [Content-Type=application/octet-stream]...
Copying file://monolith-to-microservices/microservices/src/orders/package.json [Content-Type=application/json]...
Copying file://monolith-to-microservices/microservices/src/orders/.gitignore [Content-Type=application/octet-stream]...
Copying file://monolith-to-microservices/microservices/src/orders/package-lock.json [Content-Type=application/json]...
Copying file://monolith-to-microservices/microservices/src/orders/Dockerfile [Content-Type=application/octet-stream]...
Copying file://monolith-to-microservices/microservices/src/orders/server.js [Content-Type=text/javascript]...
Copying file://monolith-to-microservices/microservices/src/orders/.dockerignore [Content-Type=application/octet-stream]...
Copying file://monolith-to-microservices/microservices/src/orders/data/orders.json [Content-Type=application/json]...
Copying file://monolith-to-microservices/microservices/src/orders/k8s/service.yml [Content-Type=application/octet-stream]...
Copying file://monolith-to-microservices/microservices/src/orders/k8s/deployment.yml [Content-Type=application/octet-stream]...
Copying file://monolith-to-microservices/microservices/src/frontend/package.json [Content-Type=application/json]...
Copying file://monolith-to-microservices/microservices/src/frontend/.gitignore [Content-Type=application/octet-stream]...
Copying file://monolith-to-microservices/microservices/src/frontend/package-lock.json [Content-Type=application/json]...
Copying file://monolith-to-microservices/microservices/src/frontend/.gcloudignore [Content-Type=application/octet-stream]...
Copying file://monolith-to-microservices/microservices/src/frontend/Dockerfile [Content-Type=application/octet-stream]...
Copying file://monolith-to-microservices/microservices/src/frontend/server.js [Content-Type=text/javascript]...
Copying file://monolith-to-microservices/microservices/src/frontend/.dockerignore [Content-Type=application/octet-stream]...
Copying file://monolith-to-microservices/microservices/src/frontend/k8s/service.yml [Content-Type=application/octet-stream]...
Copying file://monolith-to-microservices/microservices/src/frontend/k8s/deployment.yml [Content-Type=application/octet-stream]...
Copying file://monolith-to-microservices/microservices/src/frontend/public/asset-manifest.json [Content-Type=application/json]...
Copying file://monolith-to-microservices/microservices/src/frontend/public/index.html [Content-Type=text/html]...
Copying file://monolith-to-microservices/microservices/src/frontend/public/manifest.json [Content-Type=application/json]...
Copying file://monolith-to-microservices/microservices/src/frontend/public/robots.txt [Content-Type=text/plain]...
Copying file://monolith-to-microservices/microservices/src/frontend/public/static/img/products/record-player.jpg [Content-Type=image/jpeg]...
Copying file://monolith-to-microservices/microservices/src/frontend/public/static/img/products/film-camera.jpg [Content-Type=image/jpeg]...
Copying file://monolith-to-microservices/microservices/src/frontend/public/static/img/products/air-plant.jpg [Content-Type=image/jpeg]...
Copying file://monolith-to-microservices/microservices/src/frontend/public/static/img/products/credits.txt [Content-Type=text/plain]...
Copying file://monolith-to-microservices/microservices/src/frontend/public/static/img/products/barista-kit.jpg [Content-Type=image/jpeg]...
Copying file://monolith-to-microservices/microservices/src/frontend/public/static/img/products/typewriter.jpg [Content-Type=image/jpeg]...
Copying file://monolith-to-microservices/microservices/src/frontend/public/static/img/products/terrarium.jpg [Content-Type=image/jpeg]...
Copying file://monolith-to-microservices/microservices/src/frontend/public/static/img/products/camp-mug.jpg [Content-Type=image/jpeg]...
Copying file://monolith-to-microservices/microservices/src/frontend/public/static/img/products/camera-lens.jpg [Content-Type=image/jpeg]...
Copying file://monolith-to-microservices/microservices/src/frontend/public/static/img/products/city-bike.jpg [Content-Type=image/jpeg]...
Copying file://monolith-to-microservices/microservices/src/frontend/public/static/js/main.3f3d4bec.js [Content-Type=text/javascript]...
Copying file://monolith-to-microservices/microservices/src/frontend/public/static/js/main.3f3d4bec.js.LICENSE.txt [Content-Type=text/plain]...
Copying file://monolith-to-microservices/microservices/src/frontend/public/static/js/main.3f3d4bec.js.map [Content-Type=application/octet-stream]...
Copying file://monolith-to-microservices/react-app/package.json [Content-Type=application/json]...
Copying file://monolith-to-microservices/react-app/.env [Content-Type=application/octet-stream]...
Copying file://monolith-to-microservices/react-app/.gitignore [Content-Type=application/octet-stream]...
Copying file://monolith-to-microservices/react-app/package-lock.json [Content-Type=application/json]...
Copying file://monolith-to-microservices/react-app/README.md [Content-Type=text/markdown]...
Copying file://monolith-to-microservices/react-app/.env.monolith [Content-Type=application/octet-stream]...
Copying file://monolith-to-microservices/react-app/build/index.html [Content-Type=text/html]...
Copying file://monolith-to-microservices/react-app/build/asset-manifest.json [Content-Type=application/json]...
Copying file://monolith-to-microservices/react-app/build/manifest.json [Content-Type=application/json]...
Copying file://monolith-to-microservices/react-app/build/robots.txt [Content-Type=text/plain]...
Copying file://monolith-to-microservices/react-app/build/static/img/products/record-player.jpg [Content-Type=image/jpeg]...
Copying file://monolith-to-microservices/react-app/build/static/img/products/film-camera.jpg [Content-Type=image/jpeg]...
Copying file://monolith-to-microservices/react-app/build/static/img/products/air-plant.jpg [Content-Type=image/jpeg]...
Copying file://monolith-to-microservices/react-app/build/static/img/products/credits.txt [Content-Type=text/plain]...
Copying file://monolith-to-microservices/react-app/build/static/img/products/barista-kit.jpg [Content-Type=image/jpeg]...
Copying file://monolith-to-microservices/react-app/build/static/img/products/typewriter.jpg [Content-Type=image/jpeg]...
Copying file://monolith-to-microservices/react-app/build/static/img/products/terrarium.jpg [Content-Type=image/jpeg]...
Copying file://monolith-to-microservices/react-app/build/static/img/products/camp-mug.jpg [Content-Type=image/jpeg]...
Copying file://monolith-to-microservices/react-app/build/static/img/products/camera-lens.jpg [Content-Type=image/jpeg]...
Copying file://monolith-to-microservices/react-app/build/static/img/products/city-bike.jpg [Content-Type=image/jpeg]...
Copying file://monolith-to-microservices/react-app/build/static/js/main.3f3d4bec.js [Content-Type=text/javascript]...
Copying file://monolith-to-microservices/react-app/build/static/js/main.3f3d4bec.js.LICENSE.txt [Content-Type=text/plain]...
Copying file://monolith-to-microservices/react-app/build/static/js/main.3f3d4bec.js.map [Content-Type=application/octet-stream]...
Copying file://monolith-to-microservices/react-app/public/index.html [Content-Type=text/html]...
Copying file://monolith-to-microservices/react-app/public/manifest.json [Content-Type=application/json]...
Copying file://monolith-to-microservices/react-app/public/robots.txt [Content-Type=text/plain]...
Copying file://monolith-to-microservices/react-app/public/static/img/products/record-player.jpg [Content-Type=image/jpeg]...
Copying file://monolith-to-microservices/react-app/public/static/img/products/film-camera.jpg [Content-Type=image/jpeg]...
Copying file://monolith-to-microservices/react-app/public/static/img/products/air-plant.jpg [Content-Type=image/jpeg]...
Copying file://monolith-to-microservices/react-app/public/static/img/products/credits.txt [Content-Type=text/plain]...
Copying file://monolith-to-microservices/react-app/public/static/img/products/barista-kit.jpg [Content-Type=image/jpeg]...
Copying file://monolith-to-microservices/react-app/public/static/img/products/typewriter.jpg [Content-Type=image/jpeg]...
Copying file://monolith-to-microservices/react-app/public/static/img/products/terrarium.jpg [Content-Type=image/jpeg]...
Copying file://monolith-to-microservices/react-app/public/static/img/products/camp-mug.jpg [Content-Type=image/jpeg]...
Copying file://monolith-to-microservices/react-app/public/static/img/products/camera-lens.jpg [Content-Type=image/jpeg]...
Copying file://monolith-to-microservices/react-app/public/static/img/products/city-bike.jpg [Content-Type=image/jpeg]...
Copying file://monolith-to-microservices/react-app/src/App.js [Content-Type=text/javascript]...
Copying file://monolith-to-microservices/react-app/src/index.js [Content-Type=text/javascript]...
Copying file://monolith-to-microservices/react-app/src/components/ClippedDrawer/index.js [Content-Type=text/javascript]...
Copying file://monolith-to-microservices/react-app/src/pages/OrderDetails/index.js [Content-Type=text/javascript]...
Copying file://monolith-to-microservices/react-app/src/pages/Orders/index.js [Content-Type=text/javascript]...
Copying file://monolith-to-microservices/react-app/src/pages/NotFound/index.js [Content-Type=text/javascript]...
Copying file://monolith-to-microservices/react-app/src/pages/Home/index.js [Content-Type=text/javascript]...
Copying file://monolith-to-microservices/react-app/src/pages/Home/index.js.new [Content-Type=application/octet-stream]...
Copying file://monolith-to-microservices/react-app/src/pages/Products/index.js [Content-Type=text/javascript]...
Copying file://monolith-to-microservices/react-app/scripts/post-build.js [Content-Type=text/javascript]...
Copying file://monolith-to-microservices/.git/description [Content-Type=application/octet-stream]...
Copying file://monolith-to-microservices/.git/packed-refs [Content-Type=application/octet-stream]...
Copying file://monolith-to-microservices/.git/config [Content-Type=application/octet-stream]...
Copying file://monolith-to-microservices/.git/HEAD [Content-Type=application/octet-stream]...
Copying file://monolith-to-microservices/.git/index [Content-Type=application/octet-stream]...
Copying file://monolith-to-microservices/.git/logs/HEAD [Content-Type=application/octet-stream]...
Copying file://monolith-to-microservices/.git/logs/refs/heads/master [Content-Type=application/octet-stream]...
Copying file://monolith-to-microservices/.git/logs/refs/remotes/origin/HEAD [Content-Type=application/octet-stream]...
Copying file://monolith-to-microservices/.git/objects/pack/pack-9101dc88562686bc44dd54dbac2b6a740ed36c42.idx [Content-Type=application/octet-stream]...
Copying file://monolith-to-microservices/.git/objects/pack/pack-9101dc88562686bc44dd54dbac2b6a740ed36c42.pack [Content-Type=application/octet-stream]...
Copying file://monolith-to-microservices/.git/refs/heads/master [Content-Type=application/octet-stream]...
Copying file://monolith-to-microservices/.git/refs/remotes/origin/HEAD [Content-Type=application/octet-stream]...
Copying file://monolith-to-microservices/.git/info/exclude [Content-Type=application/octet-stream]...
Copying file://monolith-to-microservices/.git/hooks/update.sample [Content-Type=application/octet-stream]...
Copying file://monolith-to-microservices/.git/hooks/pre-merge-commit.sample [Content-Type=application/octet-stream]...
Copying file://monolith-to-microservices/.git/hooks/pre-applypatch.sample [Content-Type=application/octet-stream]...
Copying file://monolith-to-microservices/.git/hooks/pre-push.sample [Content-Type=application/octet-stream]...
Copying file://monolith-to-microservices/.git/hooks/pre-commit.sample [Content-Type=application/octet-stream]...
Copying file://monolith-to-microservices/.git/hooks/commit-msg.sample [Content-Type=application/octet-stream]...
Copying file://monolith-to-microservices/.git/hooks/prepare-commit-msg.sample [Content-Type=application/octet-stream]...
Copying file://monolith-to-microservices/.git/hooks/push-to-checkout.sample [Content-Type=application/octet-stream]...
Copying file://monolith-to-microservices/.git/hooks/applypatch-msg.sample [Content-Type=application/octet-stream]...
Copying file://monolith-to-microservices/.git/hooks/post-update.sample [Content-Type=application/octet-stream]...
Copying file://monolith-to-microservices/.git/hooks/fsmonitor-watchman.sample [Content-Type=application/octet-stream]...
Copying file://monolith-to-microservices/.git/hooks/pre-rebase.sample [Content-Type=application/octet-stream]...
Copying file://monolith-to-microservices/.git/hooks/pre-receive.sample [Content-Type=application/octet-stream]...
/ [155/155 files][ 13.6 MiB/ 13.6 MiB] 100% Done                                
Operation completed over 155 objects/13.6 MiB.                                   
student_01_cad4a58911a5@cloudshell:~ (qwiklabs-gcp-04-83eb8e6fd864)$ 
```

> The `node_modules` dependencies directories are deleted to  ensure the copy is as fast and efficient as possible. These are  recreated on the instances when they start up.

### Deploy the backend instance

The first instance to be deployed will be the backend instance which will house the Orders and Products microservices. In a production environment, you may want to separate each microservice  into their own instance and instance group to allow them to scale  independently. For demonstration purposes, both backend microservices  (Orders & Products) will reside on the same instance and instance  group.

Execute the following command to create an `e2-standard-2` instance that is configured to use the startup script. It is tagged as a `backend` instance so you can apply specific firewall rules to it later:

```sh
student_01_cad4a58911a5@cloudshell:~ (qwiklabs-gcp-04-83eb8e6fd864)$ gcloud compute instances create backend \
    --zone=$ZONE \
    --machine-type=e2-standard-2 \
    --tags=backend \
   --metadata=startup-script-url=https://storage.googleapis.com/fancy-store-$DEVSHELL_PROJECT_ID/startup-script.sh
Created [https://www.googleapis.com/compute/v1/projects/qwiklabs-gcp-04-83eb8e6fd864/zones/us-west1-a/instances/backend].
NAME: backend
ZONE: us-west1-a
MACHINE_TYPE: e2-standard-2
PREEMPTIBLE: 
INTERNAL_IP: 10.138.0.2
EXTERNAL_IP: 35.247.18.211
STATUS: RUNNING
student_01_cad4a58911a5@cloudshell:~ (qwiklabs-gcp-04-83eb8e6fd864)$ 
```

### Configure a connection to the backend

Before you deploy the frontend of the application, you need to update the configuration to point to the backend you just deployed.

1. Retrieve the external IP address of the backend with the following command, look under the `EXTERNAL_IP` tab for the backend instance:

   ```sh
   student_01_cad4a58911a5@cloudshell:~ (qwiklabs-gcp-04-83eb8e6fd864)$ gcloud compute instances list
   NAME: backend
   ZONE: us-west1-a
   MACHINE_TYPE: e2-standard-2
   PREEMPTIBLE: 
   INTERNAL_IP: 10.138.0.2
   EXTERNAL_IP: 35.247.18.211
   STATUS: RUNNING
   student_01_cad4a58911a5@cloudshell:~ (qwiklabs-gcp-04-83eb8e6fd864)$ 
   ```

1. In the Cloud Shell Explorer, navigate to `monolith-to-microservices` > `react-app`.
2. In the Code Editor, select **View** > **Toggle Hidden Files** in order to see the `.env` file.

In the next step, you edit the `.env` file to point to the External IP of the backend. **[BACKEND_ADDRESS]** represents the External IP address of the backend instance determined from the above `gcloud` command.

1. In the `.env` file, replace `localhost` with your `[BACKEND_ADDRESS]`:

```sh
REACT_APP_ORDERS_URL=http://35.247.18.211:8081/api/orders
REACT_APP_PRODUCTS_URL=http://35.247.18.211:8082/api/products
```

In Cloud Shell, run the following to rebuild `react-app`, which will update the frontend code:

```sh
student_01_cad4a58911a5@cloudshell:~ (qwiklabs-gcp-04-83eb8e6fd864)$ cd ~/monolith-to-microservices/react-app
npm install && npm run-script build
npm WARN deprecated source-map-url@0.4.1: See https://github.com/lydell/source-map-url#deprecated
npm WARN deprecated svgo@1.3.2: This SVGO version is no longer supported. Upgrade to v2.x.x.

added 1476 packages, and audited 1477 packages in 19s

202 packages are looking for funding
  run `npm fund` for details

16 vulnerabilities (8 moderate, 7 high, 1 critical)

To address issues that do not require attention, run:
  npm audit fix

To address all issues (including breaking changes), run:
  npm audit fix --force

Run `npm audit` for details.

> frontend@0.1.0 prebuild
> npm run build:monolith


> frontend@0.1.0 build:monolith
> env-cmd -f .env.monolith react-scripts build

Creating an optimized production build...
Browserslist: caniuse-lite is outdated. Please run:
  npx update-browserslist-db@latest
  Why you should do it regularly: https://github.com/browserslist/update-db#readme
Compiled successfully.

File sizes after gzip:

  93.04 kB (-18 B)  build/static/js/main.f58ec764.js

The project was built assuming it is hosted at /.
You can control this with the homepage field in your package.json.

The build folder is ready to be deployed.
You may serve it with a static server:

  npm install -g serve
  serve -s build

Find out more about deployment here:

  https://cra.link/deployment


> frontend@0.1.0 postbuild:monolith
> node scripts/post-build.js ./build ../monolith/public

Deleting stale folder: ../monolith/public
Deleted stale destination folder: ../monolith/public
Copying files from ./build to ../monolith/public
Copied ./build to ../monolith/public successfully!

> frontend@0.1.0 build
> react-scripts build

Creating an optimized production build...
Browserslist: caniuse-lite is outdated. Please run:
  npx update-browserslist-db@latest
  Why you should do it regularly: https://github.com/browserslist/update-db#readme
Compiled successfully.

File sizes after gzip:

  93.06 kB (+25 B)  build/static/js/main.3cfdbc3c.js

The project was built assuming it is hosted at /.
You can control this with the homepage field in your package.json.

The build folder is ready to be deployed.
You may serve it with a static server:

  npm install -g serve
  serve -s build

Find out more about deployment here:

  https://cra.link/deployment


> frontend@0.1.0 postbuild
> node scripts/post-build.js ./build ../microservices/src/frontend/public

Deleting stale folder: ../microservices/src/frontend/public
Deleted stale destination folder: ../microservices/src/frontend/public
Copying files from ./build to ../microservices/src/frontend/public
Copied ./build to ../microservices/src/frontend/public successfully!
student_01_cad4a58911a5@cloudshell:~/monolith-to-microservices/react-app (qwiklabs-gcp-04-83eb8e6fd864)$ 
```

Then copy the application code into the Cloud Storage bucket:

```sh
student_01_cad4a58911a5@cloudshell:~/monolith-to-microservices/react-app (qwiklabs-gcp-04-83eb8e6fd864)$ cd ~
rm -rf monolith-to-microservices/*/node_modules
gsutil -m cp -r monolith-to-microservices gs://fancy-store-$DEVSHELL_PROJECT_ID/
Copying file://monolith-to-microservices/deploy-monolith.sh [Content-Type=text/x-sh]...
Copying file://monolith-to-microservices/setup.sh [Content-Type=text/x-sh]...
Copying file://monolith-to-microservices/.gitignore [Content-Type=application/octet-stream]...
Copying file://monolith-to-microservices/README.md [Content-Type=text/markdown]...
Copying file://monolith-to-microservices/package-lock.json [Content-Type=application/json]...
Copying file://monolith-to-microservices/LICENSE [Content-Type=application/octet-stream]...
Copying file://monolith-to-microservices/startup-script.sh [Content-Type=text/x-sh]...
Copying file://monolith-to-microservices/CONTRIBUTING.md [Content-Type=text/markdown]...
Copying file://monolith-to-microservices/monolith/package.json [Content-Type=application/json]...
Copying file://monolith-to-microservices/monolith/.gitignore [Content-Type=application/octet-stream]...
Copying file://monolith-to-microservices/monolith/package-lock.json [Content-Type=application/json]...
Copying file://monolith-to-microservices/monolith/.gcloudignore [Content-Type=application/octet-stream]...
Copying file://monolith-to-microservices/monolith/Dockerfile [Content-Type=application/octet-stream]...
Copying file://monolith-to-microservices/monolith/.dockerignore [Content-Type=application/octet-stream]...
Copying file://monolith-to-microservices/monolith/data/orders.json [Content-Type=application/json]...
Copying file://monolith-to-microservices/monolith/data/products.json [Content-Type=application/json]...
Copying file://monolith-to-microservices/monolith/k8s/service.yml [Content-Type=application/octet-stream]...
Copying file://monolith-to-microservices/monolith/k8s/deployment.yml [Content-Type=application/octet-stream]...
Copying file://monolith-to-microservices/monolith/public/index.html [Content-Type=text/html]...
Copying file://monolith-to-microservices/monolith/public/asset-manifest.json [Content-Type=application/json]...
Copying file://monolith-to-microservices/monolith/public/manifest.json [Content-Type=application/json]...
Copying file://monolith-to-microservices/monolith/public/robots.txt [Content-Type=text/plain]...
Copying file://monolith-to-microservices/monolith/public/static/img/products/record-player.jpg [Content-Type=image/jpeg]...
Copying file://monolith-to-microservices/monolith/public/static/img/products/film-camera.jpg [Content-Type=image/jpeg]...
Copying file://monolith-to-microservices/monolith/public/static/img/products/air-plant.jpg [Content-Type=image/jpeg]...
Copying file://monolith-to-microservices/monolith/public/static/img/products/credits.txt [Content-Type=text/plain]...
Copying file://monolith-to-microservices/monolith/public/static/img/products/barista-kit.jpg [Content-Type=image/jpeg]...
Copying file://monolith-to-microservices/monolith/public/static/img/products/typewriter.jpg [Content-Type=image/jpeg]...
Copying file://monolith-to-microservices/monolith/public/static/img/products/terrarium.jpg [Content-Type=image/jpeg]...
Copying file://monolith-to-microservices/monolith/public/static/img/products/camp-mug.jpg [Content-Type=image/jpeg]...
Copying file://monolith-to-microservices/monolith/public/static/img/products/camera-lens.jpg [Content-Type=image/jpeg]...
Copying file://monolith-to-microservices/monolith/public/static/img/products/city-bike.jpg [Content-Type=image/jpeg]...
Copying file://monolith-to-microservices/monolith/public/static/js/main.f58ec764.js [Content-Type=text/javascript]...
Copying file://monolith-to-microservices/monolith/public/static/js/main.f58ec764.js.LICENSE.txt [Content-Type=text/plain]...
Copying file://monolith-to-microservices/monolith/src/server.js [Content-Type=text/javascript]...
Copying file://monolith-to-microservices/monolith/public/static/js/main.f58ec764.js.map [Content-Type=application/octet-stream]...
Copying file://monolith-to-microservices/microservices/package.json [Content-Type=application/json]...
Copying file://monolith-to-microservices/microservices/.gitignore [Content-Type=application/octet-stream]...
Copying file://monolith-to-microservices/microservices/package-lock.json [Content-Type=application/json]...
Copying file://monolith-to-microservices/microservices/src/products/package.json [Content-Type=application/json]...
Copying file://monolith-to-microservices/microservices/src/products/.gitignore [Content-Type=application/octet-stream]...
Copying file://monolith-to-microservices/microservices/src/products/package-lock.json [Content-Type=application/json]...
Copying file://monolith-to-microservices/microservices/src/products/Dockerfile [Content-Type=application/octet-stream]...
Copying file://monolith-to-microservices/microservices/src/products/server.js [Content-Type=text/javascript]...
Copying file://monolith-to-microservices/microservices/src/products/.dockerignore [Content-Type=application/octet-stream]...
Copying file://monolith-to-microservices/microservices/src/products/data/products.json [Content-Type=application/json]...
Copying file://monolith-to-microservices/microservices/src/products/k8s/service.yml [Content-Type=application/octet-stream]...
Copying file://monolith-to-microservices/microservices/src/products/k8s/deployment.yml [Content-Type=application/octet-stream]...
Copying file://monolith-to-microservices/microservices/src/orders/package.json [Content-Type=application/json]...
Copying file://monolith-to-microservices/microservices/src/orders/.gitignore [Content-Type=application/octet-stream]...
Copying file://monolith-to-microservices/microservices/src/orders/package-lock.json [Content-Type=application/json]...
Copying file://monolith-to-microservices/microservices/src/orders/Dockerfile [Content-Type=application/octet-stream]...
Copying file://monolith-to-microservices/microservices/src/orders/server.js [Content-Type=text/javascript]...
Copying file://monolith-to-microservices/microservices/src/orders/.dockerignore [Content-Type=application/octet-stream]...
Copying file://monolith-to-microservices/microservices/src/orders/data/orders.json [Content-Type=application/json]...
Copying file://monolith-to-microservices/microservices/src/orders/k8s/service.yml [Content-Type=application/octet-stream]...
Copying file://monolith-to-microservices/microservices/src/orders/k8s/deployment.yml [Content-Type=application/octet-stream]...
Copying file://monolith-to-microservices/microservices/src/frontend/package.json [Content-Type=application/json]...
Copying file://monolith-to-microservices/microservices/src/frontend/.gitignore [Content-Type=application/octet-stream]...
Copying file://monolith-to-microservices/microservices/src/frontend/package-lock.json [Content-Type=application/json]...
Copying file://monolith-to-microservices/microservices/src/frontend/.gcloudignore [Content-Type=application/octet-stream]...
Copying file://monolith-to-microservices/microservices/src/frontend/Dockerfile [Content-Type=application/octet-stream]...
Copying file://monolith-to-microservices/microservices/src/frontend/server.js [Content-Type=text/javascript]...
Copying file://monolith-to-microservices/microservices/src/frontend/.dockerignore [Content-Type=application/octet-stream]...
Copying file://monolith-to-microservices/microservices/src/frontend/k8s/service.yml [Content-Type=application/octet-stream]...
Copying file://monolith-to-microservices/microservices/src/frontend/k8s/deployment.yml [Content-Type=application/octet-stream]...
Copying file://monolith-to-microservices/microservices/src/frontend/public/index.html [Content-Type=text/html]...
Copying file://monolith-to-microservices/microservices/src/frontend/public/asset-manifest.json [Content-Type=application/json]...
Copying file://monolith-to-microservices/microservices/src/frontend/public/manifest.json [Content-Type=application/json]...
Copying file://monolith-to-microservices/microservices/src/frontend/public/robots.txt [Content-Type=text/plain]...
Copying file://monolith-to-microservices/microservices/src/frontend/public/static/img/products/record-player.jpg [Content-Type=image/jpeg]...
Copying file://monolith-to-microservices/microservices/src/frontend/public/static/img/products/film-camera.jpg [Content-Type=image/jpeg]...
Copying file://monolith-to-microservices/microservices/src/frontend/public/static/img/products/air-plant.jpg [Content-Type=image/jpeg]...
Copying file://monolith-to-microservices/microservices/src/frontend/public/static/img/products/credits.txt [Content-Type=text/plain]...
Copying file://monolith-to-microservices/microservices/src/frontend/public/static/img/products/barista-kit.jpg [Content-Type=image/jpeg]...
Copying file://monolith-to-microservices/microservices/src/frontend/public/static/img/products/typewriter.jpg [Content-Type=image/jpeg]...
Copying file://monolith-to-microservices/microservices/src/frontend/public/static/img/products/terrarium.jpg [Content-Type=image/jpeg]...
Copying file://monolith-to-microservices/microservices/src/frontend/public/static/img/products/camp-mug.jpg [Content-Type=image/jpeg]...
Copying file://monolith-to-microservices/microservices/src/frontend/public/static/img/products/camera-lens.jpg [Content-Type=image/jpeg]...
Copying file://monolith-to-microservices/microservices/src/frontend/public/static/img/products/city-bike.jpg [Content-Type=image/jpeg]...
Copying file://monolith-to-microservices/microservices/src/frontend/public/static/js/main.3cfdbc3c.js.LICENSE.txt [Content-Type=text/plain]...
Copying file://monolith-to-microservices/microservices/src/frontend/public/static/js/main.3cfdbc3c.js.map [Content-Type=application/octet-stream]...
Copying file://monolith-to-microservices/microservices/src/frontend/public/static/js/main.3cfdbc3c.js [Content-Type=text/javascript]...
Copying file://monolith-to-microservices/react-app/package.json [Content-Type=application/json]...
Copying file://monolith-to-microservices/react-app/.env [Content-Type=application/octet-stream]...
Copying file://monolith-to-microservices/react-app/.gitignore [Content-Type=application/octet-stream]...
Copying file://monolith-to-microservices/react-app/package-lock.json [Content-Type=application/json]...
Copying file://monolith-to-microservices/react-app/README.md [Content-Type=text/markdown]...
Copying file://monolith-to-microservices/react-app/.env.monolith [Content-Type=application/octet-stream]...
Copying file://monolith-to-microservices/react-app/build/index.html [Content-Type=text/html]...
Copying file://monolith-to-microservices/react-app/build/asset-manifest.json [Content-Type=application/json]...
Copying file://monolith-to-microservices/react-app/build/manifest.json [Content-Type=application/json]...
Copying file://monolith-to-microservices/react-app/build/robots.txt [Content-Type=text/plain]...
Copying file://monolith-to-microservices/react-app/build/static/img/products/record-player.jpg [Content-Type=image/jpeg]...
Copying file://monolith-to-microservices/react-app/build/static/img/products/film-camera.jpg [Content-Type=image/jpeg]...
Copying file://monolith-to-microservices/react-app/build/static/img/products/air-plant.jpg [Content-Type=image/jpeg]...
Copying file://monolith-to-microservices/react-app/build/static/img/products/credits.txt [Content-Type=text/plain]...
Copying file://monolith-to-microservices/react-app/build/static/img/products/barista-kit.jpg [Content-Type=image/jpeg]...
Copying file://monolith-to-microservices/react-app/build/static/img/products/typewriter.jpg [Content-Type=image/jpeg]...
Copying file://monolith-to-microservices/react-app/build/static/img/products/terrarium.jpg [Content-Type=image/jpeg]...
Copying file://monolith-to-microservices/react-app/build/static/img/products/camp-mug.jpg [Content-Type=image/jpeg]...
Copying file://monolith-to-microservices/react-app/build/static/img/products/camera-lens.jpg [Content-Type=image/jpeg]...
Copying file://monolith-to-microservices/react-app/build/static/img/products/city-bike.jpg [Content-Type=image/jpeg]...
Copying file://monolith-to-microservices/react-app/build/static/js/main.3cfdbc3c.js.LICENSE.txt [Content-Type=text/plain]...
Copying file://monolith-to-microservices/react-app/build/static/js/main.3cfdbc3c.js.map [Content-Type=application/octet-stream]...
Copying file://monolith-to-microservices/react-app/build/static/js/main.3cfdbc3c.js [Content-Type=text/javascript]...
Copying file://monolith-to-microservices/react-app/public/index.html [Content-Type=text/html]...
Copying file://monolith-to-microservices/react-app/public/manifest.json [Content-Type=application/json]...
Copying file://monolith-to-microservices/react-app/public/robots.txt [Content-Type=text/plain]...
Copying file://monolith-to-microservices/react-app/public/static/img/products/record-player.jpg [Content-Type=image/jpeg]...
Copying file://monolith-to-microservices/react-app/public/static/img/products/film-camera.jpg [Content-Type=image/jpeg]...
Copying file://monolith-to-microservices/react-app/public/static/img/products/air-plant.jpg [Content-Type=image/jpeg]...
Copying file://monolith-to-microservices/react-app/public/static/img/products/barista-kit.jpg [Content-Type=image/jpeg]...
Copying file://monolith-to-microservices/react-app/public/static/img/products/credits.txt [Content-Type=text/plain]...
Copying file://monolith-to-microservices/react-app/public/static/img/products/typewriter.jpg [Content-Type=image/jpeg]...
Copying file://monolith-to-microservices/react-app/public/static/img/products/terrarium.jpg [Content-Type=image/jpeg]...
Copying file://monolith-to-microservices/react-app/public/static/img/products/camp-mug.jpg [Content-Type=image/jpeg]...
Copying file://monolith-to-microservices/react-app/public/static/img/products/camera-lens.jpg [Content-Type=image/jpeg]...
Copying file://monolith-to-microservices/react-app/public/static/img/products/city-bike.jpg [Content-Type=image/jpeg]...
Copying file://monolith-to-microservices/react-app/src/App.js [Content-Type=text/javascript]...
Copying file://monolith-to-microservices/react-app/src/index.js [Content-Type=text/javascript]...
Copying file://monolith-to-microservices/react-app/src/components/ClippedDrawer/index.js [Content-Type=text/javascript]...
Copying file://monolith-to-microservices/react-app/src/pages/OrderDetails/index.js [Content-Type=text/javascript]...
Copying file://monolith-to-microservices/react-app/src/pages/Orders/index.js [Content-Type=text/javascript]...
Copying file://monolith-to-microservices/react-app/src/pages/NotFound/index.js [Content-Type=text/javascript]...
Copying file://monolith-to-microservices/react-app/src/pages/Home/index.js [Content-Type=text/javascript]...
Copying file://monolith-to-microservices/react-app/src/pages/Home/index.js.new [Content-Type=application/octet-stream]...
Copying file://monolith-to-microservices/react-app/src/pages/Products/index.js [Content-Type=text/javascript]...
Copying file://monolith-to-microservices/react-app/scripts/post-build.js [Content-Type=text/javascript]...
Copying file://monolith-to-microservices/.git/description [Content-Type=application/octet-stream]...
Copying file://monolith-to-microservices/.git/packed-refs [Content-Type=application/octet-stream]...
Copying file://monolith-to-microservices/.git/config [Content-Type=application/octet-stream]...
Copying file://monolith-to-microservices/.git/HEAD [Content-Type=application/octet-stream]...
Copying file://monolith-to-microservices/.git/index [Content-Type=application/octet-stream]...
Copying file://monolith-to-microservices/.git/logs/HEAD [Content-Type=application/octet-stream]...
Copying file://monolith-to-microservices/.git/logs/refs/heads/master [Content-Type=application/octet-stream]...
Copying file://monolith-to-microservices/.git/logs/refs/remotes/origin/HEAD [Content-Type=application/octet-stream]...
Copying file://monolith-to-microservices/.git/objects/pack/pack-9101dc88562686bc44dd54dbac2b6a740ed36c42.idx [Content-Type=application/octet-stream]...
Copying file://monolith-to-microservices/.git/objects/pack/pack-9101dc88562686bc44dd54dbac2b6a740ed36c42.pack [Content-Type=application/octet-stream]...
Copying file://monolith-to-microservices/.git/refs/heads/master [Content-Type=application/octet-stream]...
Copying file://monolith-to-microservices/.git/refs/remotes/origin/HEAD [Content-Type=application/octet-stream]...
Copying file://monolith-to-microservices/.git/info/exclude [Content-Type=application/octet-stream]...
Copying file://monolith-to-microservices/.git/hooks/update.sample [Content-Type=application/octet-stream]...
Copying file://monolith-to-microservices/.git/hooks/pre-merge-commit.sample [Content-Type=application/octet-stream]...
Copying file://monolith-to-microservices/.git/hooks/pre-applypatch.sample [Content-Type=application/octet-stream]...
Copying file://monolith-to-microservices/.git/hooks/pre-push.sample [Content-Type=application/octet-stream]...
Copying file://monolith-to-microservices/.git/hooks/pre-commit.sample [Content-Type=application/octet-stream]...
Copying file://monolith-to-microservices/.git/hooks/commit-msg.sample [Content-Type=application/octet-stream]...
Copying file://monolith-to-microservices/.git/hooks/prepare-commit-msg.sample [Content-Type=application/octet-stream]...
Copying file://monolith-to-microservices/.git/hooks/push-to-checkout.sample [Content-Type=application/octet-stream]...
Copying file://monolith-to-microservices/.git/hooks/applypatch-msg.sample [Content-Type=application/octet-stream]...
Copying file://monolith-to-microservices/.git/hooks/post-update.sample [Content-Type=application/octet-stream]...
Copying file://monolith-to-microservices/.git/hooks/fsmonitor-watchman.sample [Content-Type=application/octet-stream]...
Copying file://monolith-to-microservices/.git/hooks/pre-rebase.sample [Content-Type=application/octet-stream]...
Copying file://monolith-to-microservices/.git/hooks/pre-receive.sample [Content-Type=application/octet-stream]...
- [155/155 files][ 13.6 MiB/ 13.6 MiB] 100% Done  12.6 KiB/s ETA 00:00:00       
Operation completed over 155 objects/13.6 MiB.                                   
student_01_cad4a58911a5@cloudshell:~ (qwiklabs-gcp-04-83eb8e6fd864)$ 
```

### Deploy the frontend instance

Now that the code is configured, deploy the frontend instance.

- Execute the following to deploy the `frontend` instance with a similar command as before. This instance is tagged as `frontend` for firewall purposes:

  ```sh
  student_01_cad4a58911a5@cloudshell:~ (qwiklabs-gcp-04-83eb8e6fd864)$ gcloud compute instances create frontend \
      --zone=$ZONE \
      --machine-type=e2-standard-2 \
      --tags=frontend \
      --metadata=startup-script-url=https://storage.googleapis.com/fancy-store-$DEVSHELL_PROJECT_ID/startup-script.sh
  Created [https://www.googleapis.com/compute/v1/projects/qwiklabs-gcp-04-83eb8e6fd864/zones/us-west1-a/instances/frontend].
  NAME: frontend
  ZONE: us-west1-a
  MACHINE_TYPE: e2-standard-2
  PREEMPTIBLE: 
  INTERNAL_IP: 10.138.0.3
  EXTERNAL_IP: 35.230.8.224
  STATUS: RUNNING
  student_01_cad4a58911a5@cloudshell:~ (qwiklabs-gcp-04-83eb8e6fd864)$ 
  ```

  > The deployment command and startup script is used with both the frontend and backend instances for simplicity, and because the code is  configured to launch all microservices by default. As a result, all  microservices run on both the frontend and backend in this sample. In a  production environment you'd only run the microservices you need on each component.

### Configure the network

1. Create firewall rules to allow access to port 8080 for the frontend, and ports 8081-8082 for the backend. These firewall commands use the  tags assigned during instance creation for application:

   ```sh
   student_01_cad4a58911a5@cloudshell:~ (qwiklabs-gcp-04-83eb8e6fd864)$ gcloud compute firewall-rules create fw-fe \
       --allow tcp:8080 \
       --target-tags=frontend
   Creating firewall...working..Created [https://www.googleapis.com/compute/v1/projects/qwiklabs-gcp-04-83eb8e6fd864/global/firewalls/fw-fe].    
   Creating firewall...done.                                                                                                                     
   NAME: fw-fe
   NETWORK: default
   DIRECTION: INGRESS
   PRIORITY: 1000
   ALLOW: tcp:8080
   DENY: 
   DISABLED: False
   student_01_cad4a58911a5@cloudshell:~ (qwiklabs-gcp-04-83eb8e6fd864)$ 
   ```

```sh
student_01_cad4a58911a5@cloudshell:~ (qwiklabs-gcp-04-83eb8e6fd864)$ gcloud compute firewall-rules create fw-be \
    --allow tcp:8081-8082 \
    --target-tags=backend
Creating firewall...working..Created [https://www.googleapis.com/compute/v1/projects/qwiklabs-gcp-04-83eb8e6fd864/global/firewalls/fw-be].    
Creating firewall...done.                                                                                                                     
NAME: fw-be
NETWORK: default
DIRECTION: INGRESS
PRIORITY: 1000
ALLOW: tcp:8081-8082
DENY: 
DISABLED: False
student_01_cad4a58911a5@cloudshell:~ (qwiklabs-gcp-04-83eb8e6fd864)$ 
```

The website should now be fully functional.

1. In order to navigate to the external IP of the `frontend`, you need to know the address. Run the following and look for the EXTERNAL_IP of the `frontend` instance:

   ```sh
   student_01_cad4a58911a5@cloudshell:~ (qwiklabs-gcp-04-83eb8e6fd864)$ gcloud compute instances list
   NAME: backend
   ZONE: us-west1-a
   MACHINE_TYPE: e2-standard-2
   PREEMPTIBLE: 
   INTERNAL_IP: 10.138.0.2
   EXTERNAL_IP: 35.247.18.211
   STATUS: RUNNING
   
   NAME: frontend
   ZONE: us-west1-a
   MACHINE_TYPE: e2-standard-2
   PREEMPTIBLE: 
   INTERNAL_IP: 10.138.0.3
   EXTERNAL_IP: 35.230.8.224
   STATUS: RUNNING
   student_01_cad4a58911a5@cloudshell:~ (qwiklabs-gcp-04-83eb8e6fd864)$ 
   ```

   It may take a couple minutes for the instance to start and be configured.

   1. Wait 3 minutes and then open a new browser tab and browse to `http://[FRONTEND_ADDRESS]:8080` to access the website, where [FRONTEND_ADDRESS] is the frontend EXTERNAL_IP determined above.
   2. Try navigating to the **Products** and **Orders** pages; these should now work.

   ## Create managed instance groups

   To allow the application to scale, managed instance groups will be created and will use the `frontend` and `backend` instances as Instance Templates.

   A managed instance group (MIG) contains identical instances that you  can manage as a single entity in a single zone. Managed instance groups  maintain high availability of your apps by proactively keeping your  instances available, that is, in the RUNNING state. You will be using  managed instance groups for your frontend and backend instances to  provide autohealing, load balancing, autoscaling, and rolling updates.

   ### Create instance template from source instance

   Before you can create a managed instance group, you have to first  create an instance template that will be the foundation for the group.  Instance templates allow you to define the machine type, boot disk image or container image, network, and other instance properties to use when  creating new VM instances. You can use instance templates to create  instances in a managed instance group or even to create individual  instances.

   To create the instance template, use the existing instances you created previously.

   1. First, stop both instances:

```sh
student_01_cad4a58911a5@cloudshell:~ (qwiklabs-gcp-04-83eb8e6fd864)$ gcloud compute instances stop frontend --zone=$ZONE
Stopping instance(s) frontend...done.                                                                                                         
Updated [https://compute.googleapis.com/compute/v1/projects/qwiklabs-gcp-04-83eb8e6fd864/zones/us-west1-a/instances/frontend].
student_01_cad4a58911a5@cloudshell:~ (qwiklabs-gcp-04-83eb8e6fd864)$
```

```sh
student_01_cad4a58911a5@cloudshell:~ (qwiklabs-gcp-04-83eb8e6fd864)$ gcloud compute instances stop backend --zone=$ZONE
Stopping instance(s) backend...done.                                                                                                          
Updated [https://compute.googleapis.com/compute/v1/projects/qwiklabs-gcp-04-83eb8e6fd864/zones/us-west1-a/instances/backend].
student_01_cad4a58911a5@cloudshell:~ (qwiklabs-gcp-04-83eb8e6fd864)$ 
```

Then, create the instance template from each of the source instances:

```sh
student_01_cad4a58911a5@cloudshell:~ (qwiklabs-gcp-04-83eb8e6fd864)$ gcloud compute instance-templates create fancy-fe \
    --source-instance-zone=$ZONE \
    --source-instance=frontend
Created [https://www.googleapis.com/compute/v1/projects/qwiklabs-gcp-04-83eb8e6fd864/global/instanceTemplates/fancy-fe].
NAME: fancy-fe
MACHINE_TYPE: e2-standard-2
PREEMPTIBLE: 
CREATION_TIMESTAMP: 2024-04-23T02:50:40.904-07:00
student_01_cad4a58911a5@cloudshell:~ (qwiklabs-gcp-04-83eb8e6fd864)$ 
```

```sh
student_01_cad4a58911a5@cloudshell:~ (qwiklabs-gcp-04-83eb8e6fd864)$ gcloud compute instance-templates create fancy-be \
    --source-instance-zone=$ZONE \
    --source-instance=backend
Created [https://www.googleapis.com/compute/v1/projects/qwiklabs-gcp-04-83eb8e6fd864/global/instanceTemplates/fancy-be].
NAME: fancy-be
MACHINE_TYPE: e2-standard-2
PREEMPTIBLE: 
CREATION_TIMESTAMP: 2024-04-23T02:51:06.084-07:00
student_01_cad4a58911a5@cloudshell:~ (qwiklabs-gcp-04-83eb8e6fd864)$ 
```

Confirm the instance templates were created:

```sh
student_01_cad4a58911a5@cloudshell:~ (qwiklabs-gcp-04-83eb8e6fd864)$ gcloud compute instance-templates list
NAME: fancy-be
MACHINE_TYPE: e2-standard-2
PREEMPTIBLE: 
CREATION_TIMESTAMP: 2024-04-23T02:51:06.084-07:00

NAME: fancy-fe
MACHINE_TYPE: e2-standard-2
PREEMPTIBLE: 
CREATION_TIMESTAMP: 2024-04-23T02:50:40.904-07:00
student_01_cad4a58911a5@cloudshell:~ (qwiklabs-gcp-04-83eb8e6fd864)$ 
```

With the instance templates created, delete the `backend` vm to save resource space:  Type and enter **y** when prompted.

Normally, you could delete the `frontend` vm as well, but you will use it to update the instance template later in the lab.

```sh
student_01_cad4a58911a5@cloudshell:~ (qwiklabs-gcp-04-83eb8e6fd864)$ gcloud compute instances delete backend --zone=$ZONE
The following instances will be deleted. Any attached disks configured to be auto-deleted will be deleted unless they are attached to any other
 instances or the `--keep-disks` flag is given and specifies them for keeping. Deleting a disk is irreversible and any data on the disk will be
 lost.
 - [backend] in [us-west1-a]

Do you want to continue (Y/n)?  y

Deleted [https://www.googleapis.com/compute/v1/projects/qwiklabs-gcp-04-83eb8e6fd864/zones/us-west1-a/instances/backend].
student_01_cad4a58911a5@cloudshell:~ (qwiklabs-gcp-04-83eb8e6fd864)$ 
```

### Create managed instance group

1. Next, create two managed instance groups, one for the frontend and one for the backend:

   ```sh
   student_01_cad4a58911a5@cloudshell:~ (qwiklabs-gcp-04-83eb8e6fd864)$ gcloud compute instance-groups managed create fancy-fe-mig \
       --zone=$ZONE \
       --base-instance-name fancy-fe \
       --size 2 \
       --template fancy-fe
   Created [https://www.googleapis.com/compute/v1/projects/qwiklabs-gcp-04-83eb8e6fd864/zones/us-west1-a/instanceGroupManagers/fancy-fe-mig].
   NAME: fancy-fe-mig
   LOCATION: us-west1-a
   SCOPE: zone
   BASE_INSTANCE_NAME: fancy-fe
   SIZE: 0
   TARGET_SIZE: 2
   INSTANCE_TEMPLATE: fancy-fe
   AUTOSCALED: no
   student_01_cad4a58911a5@cloudshell:~ (qwiklabs-gcp-04-83eb8e6fd864)$ 
   ```

   ```sh
   student_01_cad4a58911a5@cloudshell:~ (qwiklabs-gcp-04-83eb8e6fd864)$ gcloud compute instance-groups managed create fancy-be-mig \
       --zone=$ZONE \
       --base-instance-name fancy-be \
       --size 2 \
       --template fancy-be
   Created [https://www.googleapis.com/compute/v1/projects/qwiklabs-gcp-04-83eb8e6fd864/zones/us-west1-a/instanceGroupManagers/fancy-be-mig].
   NAME: fancy-be-mig
   LOCATION: us-west1-a
   SCOPE: zone
   BASE_INSTANCE_NAME: fancy-be
   SIZE: 0
   TARGET_SIZE: 2
   INSTANCE_TEMPLATE: fancy-be
   AUTOSCALED: no
   student_01_cad4a58911a5@cloudshell:~ (qwiklabs-gcp-04-83eb8e6fd864)$ 
   ```

   These managed instance groups will use the instance templates and are  configured for two instances each within each group to start. The  instances are automatically named based on the `base-instance-name` specified with random characters appended.

   For your application, the `frontend` microservice runs on port 8080, and the `backend` microservice runs on port 8081 for `orders` and port 8082 for products:

   Since these are non-standard ports, you specify named ports to identify  these. Named ports are key:value pair metadata representing the service  name and the port that it's running on. Named ports can be assigned to  an instance group, which indicates that the service is available on all  instances in the group. This information is used by the HTTP Load  Balancing service that will be configured later.

   ```sh
   student_01_cad4a58911a5@cloudshell:~ (qwiklabs-gcp-04-83eb8e6fd864)$ gcloud compute instance-groups set-named-ports fancy-fe-mig \
       --zone=$ZONE \
       --named-ports frontend:8080
   Updated [https://www.googleapis.com/compute/v1/projects/qwiklabs-gcp-04-83eb8e6fd864/zones/us-west1-a/instanceGroups/fancy-fe-mig].
   student_01_cad4a58911a5@cloudshell:~ (qwiklabs-gcp-04-83eb8e6fd864)$ 
   ```

   ```sh
   student_01_cad4a58911a5@cloudshell:~ (qwiklabs-gcp-04-83eb8e6fd864)$ gcloud compute instance-groups set-named-ports fancy-be-mig \
       --zone=$ZONE \
       --named-ports orders:8081,products:8082
   Updated [https://www.googleapis.com/compute/v1/projects/qwiklabs-gcp-04-83eb8e6fd864/zones/us-west1-a/instanceGroups/fancy-be-mig].
   student_01_cad4a58911a5@cloudshell:~ (qwiklabs-gcp-04-83eb8e6fd864)$ 
   ```

   

### Configure autohealing

To improve the availability of the application itself and to verify  it is responding, configure an autohealing policy for the managed  instance groups.

An autohealing policy relies on an application-based health check to  verify that an app is responding as expected. Checking that an app  responds is more precise than simply verifying that an instance is in a  RUNNING state, which is the default behavior.

> Separate health checks for load balancing and for autohealing will be  used. Health checks for load balancing can and should be more aggressive because these health checks determine whether an instance receives user traffic. You want to catch non-responsive instances quickly so you can  redirect traffic if necessary. In contrast, health checking for autohealing causes Compute Engine to  proactively replace failing instances, so this health check should be  more conservative than a load balancing health check.



Create a health check that repairs the instance if it returns "unhealthy" 3 consecutive times for the `frontend` and `backend`:

```sh
student_01_cad4a58911a5@cloudshell:~ (qwiklabs-gcp-04-83eb8e6fd864)$ gcloud compute health-checks create http fancy-fe-hc \
    --port 8080 \
    --check-interval 30s \
    --healthy-threshold 1 \
    --timeout 10s \
    --unhealthy-threshold 3
Created [https://www.googleapis.com/compute/v1/projects/qwiklabs-gcp-04-83eb8e6fd864/global/healthChecks/fancy-fe-hc].
NAME: fancy-fe-hc
PROTOCOL: HTTP
student_01_cad4a58911a5@cloudshell:~ (qwiklabs-gcp-04-83eb8e6fd864)$ 
```

```sh
student_01_cad4a58911a5@cloudshell:~ (qwiklabs-gcp-04-83eb8e6fd864)$ gcloud compute health-checks create http fancy-be-hc \
    --port 8081 \
    --request-path=/api/orders \
    --check-interval 30s \
    --healthy-threshold 1 \
    --timeout 10s \
    --unhealthy-threshold 3
Created [https://www.googleapis.com/compute/v1/projects/qwiklabs-gcp-04-83eb8e6fd864/global/healthChecks/fancy-be-hc].
NAME: fancy-be-hc
PROTOCOL: HTTP
student_01_cad4a58911a5@cloudshell:~ (qwiklabs-gcp-04-83eb8e6fd864)$ 
```

Create a firewall rule to allow the health check probes to connect to the microservices on ports 8080-8081:

```sh
student_01_cad4a58911a5@cloudshell:~ (qwiklabs-gcp-04-83eb8e6fd864)$ gcloud compute firewall-rules create allow-health-check \
    --allow tcp:8080-8081 \
    --source-ranges 130.211.0.0/22,35.191.0.0/16 \
    --network default
Creating firewall...working..Created [https://www.googleapis.com/compute/v1/projects/qwiklabs-gcp-04-83eb8e6fd864/global/firewalls/allow-health-check].
Creating firewall...done.                                                                                                                     
NAME: allow-health-check
NETWORK: default
DIRECTION: INGRESS
PRIORITY: 1000
ALLOW: tcp:8080-8081
DENY: 
DISABLED: False
student_01_cad4a58911a5@cloudshell:~ (qwiklabs-gcp-04-83eb8e6fd864)$ 
```

Apply the health checks to their respective services:

```sh
student_01_cad4a58911a5@cloudshell:~ (qwiklabs-gcp-04-83eb8e6fd864)$ gcloud compute instance-groups managed update fancy-fe-mig \
    --zone=$ZONE \
    --health-check fancy-fe-hc \
    --initial-delay 300
Updated [https://www.googleapis.com/compute/v1/projects/qwiklabs-gcp-04-83eb8e6fd864/zones/us-west1-a/instanceGroupManagers/fancy-fe-mig].
student_01_cad4a58911a5@cloudshell:~ (qwiklabs-gcp-04-83eb8e6fd864)$ 
```

```sh
student_01_cad4a58911a5@cloudshell:~ (qwiklabs-gcp-04-83eb8e6fd864)$ gcloud compute instance-groups managed update fancy-be-mig \
    --zone=$ZONE \
    --health-check fancy-be-hc \
    --initial-delay 300
Updated [https://www.googleapis.com/compute/v1/projects/qwiklabs-gcp-04-83eb8e6fd864/zones/us-west1-a/instanceGroupManagers/fancy-be-mig].
student_01_cad4a58911a5@cloudshell:~ (qwiklabs-gcp-04-83eb8e6fd864)$ 
```

**Note:** It can take 15 minutes before autohealing begins monitoring instances in the group.

## Create load balancers

To complement your managed instance groups, use HTTP(S) Load  Balancers to serve traffic to the frontend and backend microservices,  and use mappings to send traffic to the proper backend services based on pathing rules. This exposes a single load balanced IP for all services.

### Create HTTP(S) load balancer

Google Cloud offers many different types of load balancers. For this  lab you use an HTTP(S) Load Balancer for your traffic. An HTTP load  balancer is structured as follows:

1. A forwarding rule directs incoming requests to a target HTTP proxy.
2. The target HTTP proxy checks each request against a URL map to determine the appropriate backend service for the request.
3. The backend service directs each request to an appropriate backend  based on serving capacity, zone, and instance health of its attached  backends. The health of each backend instance is verified using an HTTP  health check. If the backend service is configured to use an HTTPS or  HTTP/2 health check, the request will be encrypted on its way to the  backend instance.
4. Sessions between the load balancer and the instance can use the  HTTP, HTTPS, or HTTP/2 protocol. If you use HTTPS or HTTP/2, each  instance in the backend services must have an SSL certificate.

Create health checks that will be used to determine which instances are capable of serving traffic for each service:

```sh
student_01_cad4a58911a5@cloudshell:~ (qwiklabs-gcp-04-83eb8e6fd864)$ gcloud compute http-health-checks create fancy-fe-frontend-hc \
  --request-path / \
  --port 8080
Created [https://www.googleapis.com/compute/v1/projects/qwiklabs-gcp-04-83eb8e6fd864/global/httpHealthChecks/fancy-fe-frontend-hc].
NAME: fancy-fe-frontend-hc
HOST: 
PORT: 8080
REQUEST_PATH: /
student_01_cad4a58911a5@cloudshell:~ (qwiklabs-gcp-04-83eb8e6fd864)$ 
```

```sh
student_01_cad4a58911a5@cloudshell:~ (qwiklabs-gcp-04-83eb8e6fd864)$ gcloud compute http-health-checks create fancy-be-orders-hc \
  --request-path /api/orders \
  --port 8081
Created [https://www.googleapis.com/compute/v1/projects/qwiklabs-gcp-04-83eb8e6fd864/global/httpHealthChecks/fancy-be-orders-hc].
NAME: fancy-be-orders-hc
HOST: 
PORT: 8081
REQUEST_PATH: /api/orders
student_01_cad4a58911a5@cloudshell:~ (qwiklabs-gcp-04-83eb8e6fd864)$ 
```

```sh
student_01_cad4a58911a5@cloudshell:~ (qwiklabs-gcp-04-83eb8e6fd864)$ gcloud compute http-health-checks create fancy-be-products-hc \
>   --request-path /api/products \
>   --port 8082
Created [https://www.googleapis.com/compute/v1/projects/qwiklabs-gcp-04-83eb8e6fd864/global/httpHealthChecks/fancy-be-products-hc].
NAME: fancy-be-products-hc
HOST: 
PORT: 8082
REQUEST_PATH: /api/products
student_01_cad4a58911a5@cloudshell:~ (qwiklabs-gcp-04-83eb8e6fd864)$ 
```

> **Note:** These health checks are for the load balancer,  and only handle directing traffic from the load balancer; they do not  cause the managed instance groups to recreate instances.

Create backend services that are the target for load-balanced traffic.  The backend services will use the health checks and named ports you  created:

```sh
student_01_cad4a58911a5@cloudshell:~ (qwiklabs-gcp-04-83eb8e6fd864)$ gcloud compute backend-services create fancy-fe-frontend \
  --http-health-checks fancy-fe-frontend-hc \
  --port-name frontend \
  --global
Created [https://www.googleapis.com/compute/v1/projects/qwiklabs-gcp-04-83eb8e6fd864/global/backendServices/fancy-fe-frontend].
NAME: fancy-fe-frontend
BACKENDS: 
PROTOCOL: HTTP
student_01_cad4a58911a5@cloudshell:~ (qwiklabs-gcp-04-83eb8e6fd864)$ 
```

```sh
student_01_cad4a58911a5@cloudshell:~ (qwiklabs-gcp-04-83eb8e6fd864)$ 
student_01_cad4a58911a5@cloudshell:~ (qwiklabs-gcp-04-83eb8e6fd864)$ gcloud compute backend-services create fancy-be-orders \
  --http-health-checks fancy-be-orders-hc \
  --port-name orders \
  --global
Created [https://www.googleapis.com/compute/v1/projects/qwiklabs-gcp-04-83eb8e6fd864/global/backendServices/fancy-be-orders].
NAME: fancy-be-orders
BACKENDS: 
PROTOCOL: HTTP
student_01_cad4a58911a5@cloudshell:~ (qwiklabs-gcp-04-83eb8e6fd864)$ 
```

```sh
student_01_cad4a58911a5@cloudshell:~ (qwiklabs-gcp-04-83eb8e6fd864)$ gcloud compute backend-services create fancy-be-products \
  --http-health-checks fancy-be-products-hc \
  --port-name products \
  --global
Created [https://www.googleapis.com/compute/v1/projects/qwiklabs-gcp-04-83eb8e6fd864/global/backendServices/fancy-be-products].
NAME: fancy-be-products
BACKENDS: 
PROTOCOL: HTTP
student_01_cad4a58911a5@cloudshell:~ (qwiklabs-gcp-04-83eb8e6fd864)$ 
```

Add the Load Balancer's [backend services](https://cloud.google.com/load-balancing/docs/backend-service):

```sh
student_01_cad4a58911a5@cloudshell:~ (qwiklabs-gcp-04-83eb8e6fd864)$ gcloud compute backend-services add-backend fancy-fe-frontend \
  --instance-group-zone=$ZONE \
  --instance-group fancy-fe-mig \
  --global

Updated [https://www.googleapis.com/compute/v1/projects/qwiklabs-gcp-04-83eb8e6fd864/global/backendServices/fancy-fe-frontend].
student_01_cad4a58911a5@cloudshell:~ (qwiklabs-gcp-04-83eb8e6fd864)$ 
```

```sh
student_01_cad4a58911a5@cloudshell:~ (qwiklabs-gcp-04-83eb8e6fd864)$ gcloud compute backend-services add-backend fancy-be-orders \
  --instance-group-zone=$ZONE \
  --instance-group fancy-be-mig \
  --global
Updated [https://www.googleapis.com/compute/v1/projects/qwiklabs-gcp-04-83eb8e6fd864/global/backendServices/fancy-be-orders].
student_01_cad4a58911a5@cloudshell:~ (qwiklabs-gcp-04-83eb8e6fd864)$ 
```

```sh
student_01_cad4a58911a5@cloudshell:~ (qwiklabs-gcp-04-83eb8e6fd864)$ gcloud compute backend-services add-backend fancy-be-products \
  --instance-group-zone=$ZONE \
  --instance-group fancy-be-mig \
  --global
Updated [https://www.googleapis.com/compute/v1/projects/qwiklabs-gcp-04-83eb8e6fd864/global/backendServices/fancy-be-products].
student_01_cad4a58911a5@cloudshell:~ (qwiklabs-gcp-04-83eb8e6fd864)$ 
```

Create a URL map. The URL map defines which URLs are directed to which backend services:

```sh
student_01_cad4a58911a5@cloudshell:~ (qwiklabs-gcp-04-83eb8e6fd864)$ gcloud compute url-maps create fancy-map \
  --default-service fancy-fe-frontend
Created [https://www.googleapis.com/compute/v1/projects/qwiklabs-gcp-04-83eb8e6fd864/global/urlMaps/fancy-map].
NAME: fancy-map
DEFAULT_SERVICE: backendServices/fancy-fe-frontend
student_01_cad4a58911a5@cloudshell:~ (qwiklabs-gcp-04-83eb8e6fd864)$ 
```

Create a path matcher to allow the `/api/orders` and `/api/products` paths to route to their respective services:

```sh
student_01_cad4a58911a5@cloudshell:~ (qwiklabs-gcp-04-83eb8e6fd864)$ gcloud compute url-maps add-path-matcher fancy-map \
   --default-service fancy-fe-frontend \
   --path-matcher-name orders \
   --path-rules "/api/orders=fancy-be-orders,/api/products=fancy-be-products"
Updated [https://www.googleapis.com/compute/v1/projects/qwiklabs-gcp-04-83eb8e6fd864/global/urlMaps/fancy-map].
student_01_cad4a58911a5@cloudshell:~ (qwiklabs-gcp-04-83eb8e6fd864)$ 
```

Create the proxy which ties to the URL map:

```sh
student_01_cad4a58911a5@cloudshell:~ (qwiklabs-gcp-04-83eb8e6fd864)$ gcloud compute target-http-proxies create fancy-proxy \
  --url-map fancy-map
Created [https://www.googleapis.com/compute/v1/projects/qwiklabs-gcp-04-83eb8e6fd864/global/targetHttpProxies/fancy-proxy].
NAME: fancy-proxy
URL_MAP: fancy-map
student_01_cad4a58911a5@cloudshell:~ (qwiklabs-gcp-04-83eb8e6fd864)$ 
```

Create a global forwarding rule that ties a public IP address and port to the proxy:

```sh
student_01_cad4a58911a5@cloudshell:~ (qwiklabs-gcp-04-83eb8e6fd864)$ gcloud compute forwarding-rules create fancy-http-rule \
  --global \
  --target-http-proxy fancy-proxy \
  --ports 80
Created [https://www.googleapis.com/compute/v1/projects/qwiklabs-gcp-04-83eb8e6fd864/global/forwardingRules/fancy-http-rule].
student_01_cad4a58911a5@cloudshell:~ (qwiklabs-gcp-04-83eb8e6fd864)$ 
```

### Update the configuration

Now that you have a new static IP address, update the code on the `frontend` to point to this new address instead of the ephemeral address used earlier that pointed to the `backend` instance.

1. In Cloud Shell, change to the `react-app` folder which houses the `.env` file that holds the configuration:

Return to the Cloud Shell Editor and edit the `.env` file  again to point to Public IP of Load Balancer. [LB_IP] represents the  External IP address of the backend instance determined above.

```sh
student_01_cad4a58911a5@cloudshell:~ (qwiklabs-gcp-04-83eb8e6fd864)$ cd ~/monolith-to-microservices/react-app/
student_01_cad4a58911a5@cloudshell:~/monolith-to-microservices/react-app (qwiklabs-gcp-04-83eb8e6fd864)$ 
student_01_cad4a58911a5@cloudshell:~/monolith-to-microservices/react-app (qwiklabs-gcp-04-83eb8e6fd864)$ gcloud compute forwarding-rules list --global
NAME: fancy-http-rule
REGION: 
IP_ADDRESS: 34.36.49.82
IP_PROTOCOL: TCP
TARGET: fancy-proxy
student_01_cad4a58911a5@cloudshell:~/monolith-to-microservices/react-app (qwiklabs-gcp-04-83eb8e6fd864)$ 
```

```sh
REACT_APP_ORDERS_URL=http://34.36.49.82/api/orders
REACT_APP_PRODUCTS_URL=http://34.36.49.82/api/products
```

**Note:** The ports are removed in the new address because the load balancer is configured to handle this forwarding for you.

Rebuild `react-app`, which will update the frontend code:

```sh
student_01_cad4a58911a5@cloudshell:~/monolith-to-microservices/react-app (qwiklabs-gcp-04-83eb8e6fd864)$ cd ~/monolith-to-microservices/react-app
npm install && npm run-script build
npm WARN deprecated source-map-url@0.4.1: See https://github.com/lydell/source-map-url#deprecated
npm WARN deprecated svgo@1.3.2: This SVGO version is no longer supported. Upgrade to v2.x.x.

added 1476 packages, and audited 1477 packages in 20s

202 packages are looking for funding
  run `npm fund` for details

16 vulnerabilities (8 moderate, 7 high, 1 critical)

To address issues that do not require attention, run:
  npm audit fix

To address all issues (including breaking changes), run:
  npm audit fix --force

Run `npm audit` for details.

> frontend@0.1.0 prebuild
> npm run build:monolith


> frontend@0.1.0 build:monolith
> env-cmd -f .env.monolith react-scripts build

Creating an optimized production build...
Browserslist: caniuse-lite is outdated. Please run:
  npx update-browserslist-db@latest
  Why you should do it regularly: https://github.com/browserslist/update-db#readme
Compiled successfully.

File sizes after gzip:

  93.04 kB (-25 B)  build/static/js/main.f58ec764.js

The project was built assuming it is hosted at /.
You can control this with the homepage field in your package.json.

The build folder is ready to be deployed.
You may serve it with a static server:

  npm install -g serve
  serve -s build

Find out more about deployment here:

  https://cra.link/deployment


> frontend@0.1.0 postbuild:monolith
> node scripts/post-build.js ./build ../monolith/public

Deleting stale folder: ../monolith/public
Deleted stale destination folder: ../monolith/public
Copying files from ./build to ../monolith/public
Copied ./build to ../monolith/public successfully!

> frontend@0.1.0 build
> react-scripts build

Creating an optimized production build...
Browserslist: caniuse-lite is outdated. Please run:
  npx update-browserslist-db@latest
  Why you should do it regularly: https://github.com/browserslist/update-db#readme
Compiled successfully.

File sizes after gzip:

  93.05 kB (+16 B)  build/static/js/main.d934a9e5.js

The project was built assuming it is hosted at /.
You can control this with the homepage field in your package.json.

The build folder is ready to be deployed.
You may serve it with a static server:

  npm install -g serve
  serve -s build

Find out more about deployment here:

  https://cra.link/deployment


> frontend@0.1.0 postbuild
> node scripts/post-build.js ./build ../microservices/src/frontend/public

Deleting stale folder: ../microservices/src/frontend/public
Deleted stale destination folder: ../microservices/src/frontend/public
Copying files from ./build to ../microservices/src/frontend/public
Copied ./build to ../microservices/src/frontend/public successfully!
student_01_cad4a58911a5@cloudshell:~/monolith-to-microservices/react-app (qwiklabs-gcp-04-83eb8e6fd864)$ 
```

Copy the application code into your bucket:

```sh
student_01_cad4a58911a5@cloudshell:~/monolith-to-microservices/react-app (qwiklabs-gcp-04-83eb8e6fd864)$ cd ~
rm -rf monolith-to-microservices/*/node_modules
gsutil -m cp -r monolith-to-microservices gs://fancy-store-$DEVSHELL_PROJECT_ID/
Copying file://monolith-to-microservices/setup.sh [Content-Type=text/x-sh]...
Copying file://monolith-to-microservices/deploy-monolith.sh [Content-Type=text/x-sh]...
Copying file://monolith-to-microservices/README.md [Content-Type=text/markdown]...
Copying file://monolith-to-microservices/.gitignore [Content-Type=application/octet-stream]...
Copying file://monolith-to-microservices/startup-script.sh [Content-Type=text/x-sh]...
Copying file://monolith-to-microservices/package-lock.json [Content-Type=application/json]...
Copying file://monolith-to-microservices/LICENSE [Content-Type=application/octet-stream]...
Copying file://monolith-to-microservices/monolith/package.json [Content-Type=application/json]...
Copying file://monolith-to-microservices/monolith/.gitignore [Content-Type=application/octet-stream]...
Copying file://monolith-to-microservices/CONTRIBUTING.md [Content-Type=text/markdown]...
Copying file://monolith-to-microservices/monolith/package-lock.json [Content-Type=application/json]...
Copying file://monolith-to-microservices/monolith/.gcloudignore [Content-Type=application/octet-stream]...
Copying file://monolith-to-microservices/monolith/Dockerfile [Content-Type=application/octet-stream]...
Copying file://monolith-to-microservices/monolith/.dockerignore [Content-Type=application/octet-stream]...
Copying file://monolith-to-microservices/monolith/data/orders.json [Content-Type=application/json]...
Copying file://monolith-to-microservices/monolith/data/products.json [Content-Type=application/json]...
Copying file://monolith-to-microservices/monolith/k8s/service.yml [Content-Type=application/octet-stream]...
Copying file://monolith-to-microservices/monolith/k8s/deployment.yml [Content-Type=application/octet-stream]...
Copying file://monolith-to-microservices/monolith/public/index.html [Content-Type=text/html]...
Copying file://monolith-to-microservices/monolith/public/asset-manifest.json [Content-Type=application/json]...
Copying file://monolith-to-microservices/monolith/public/manifest.json [Content-Type=application/json]...
Copying file://monolith-to-microservices/monolith/public/robots.txt [Content-Type=text/plain]...
Copying file://monolith-to-microservices/monolith/public/static/img/products/record-player.jpg [Content-Type=image/jpeg]...
Copying file://monolith-to-microservices/monolith/public/static/img/products/film-camera.jpg [Content-Type=image/jpeg]...
Copying file://monolith-to-microservices/monolith/public/static/img/products/air-plant.jpg [Content-Type=image/jpeg]...
Copying file://monolith-to-microservices/monolith/public/static/img/products/credits.txt [Content-Type=text/plain]...
Copying file://monolith-to-microservices/monolith/public/static/img/products/barista-kit.jpg [Content-Type=image/jpeg]...
Copying file://monolith-to-microservices/monolith/public/static/img/products/typewriter.jpg [Content-Type=image/jpeg]...
Copying file://monolith-to-microservices/monolith/public/static/img/products/terrarium.jpg [Content-Type=image/jpeg]...
Copying file://monolith-to-microservices/monolith/public/static/img/products/camp-mug.jpg [Content-Type=image/jpeg]...
Copying file://monolith-to-microservices/monolith/public/static/img/products/camera-lens.jpg [Content-Type=image/jpeg]...
Copying file://monolith-to-microservices/monolith/public/static/img/products/city-bike.jpg [Content-Type=image/jpeg]...
Copying file://monolith-to-microservices/monolith/public/static/js/main.f58ec764.js [Content-Type=text/javascript]...
Copying file://monolith-to-microservices/monolith/public/static/js/main.f58ec764.js.LICENSE.txt [Content-Type=text/plain]...
Copying file://monolith-to-microservices/monolith/public/static/js/main.f58ec764.js.map [Content-Type=application/octet-stream]...
Copying file://monolith-to-microservices/monolith/src/server.js [Content-Type=text/javascript]...
Copying file://monolith-to-microservices/microservices/package.json [Content-Type=application/json]...
Copying file://monolith-to-microservices/microservices/.gitignore [Content-Type=application/octet-stream]...
Copying file://monolith-to-microservices/microservices/package-lock.json [Content-Type=application/json]...
Copying file://monolith-to-microservices/microservices/src/products/package.json [Content-Type=application/json]...
Copying file://monolith-to-microservices/microservices/src/products/.gitignore [Content-Type=application/octet-stream]...
Copying file://monolith-to-microservices/microservices/src/products/package-lock.json [Content-Type=application/json]...
Copying file://monolith-to-microservices/microservices/src/products/Dockerfile [Content-Type=application/octet-stream]...
Copying file://monolith-to-microservices/microservices/src/products/server.js [Content-Type=text/javascript]...
Copying file://monolith-to-microservices/microservices/src/products/.dockerignore [Content-Type=application/octet-stream]...
Copying file://monolith-to-microservices/microservices/src/products/data/products.json [Content-Type=application/json]...
Copying file://monolith-to-microservices/microservices/src/products/k8s/service.yml [Content-Type=application/octet-stream]...
Copying file://monolith-to-microservices/microservices/src/products/k8s/deployment.yml [Content-Type=application/octet-stream]...
Copying file://monolith-to-microservices/microservices/src/orders/package.json [Content-Type=application/json]...
Copying file://monolith-to-microservices/microservices/src/orders/.gitignore [Content-Type=application/octet-stream]...
Copying file://monolith-to-microservices/microservices/src/orders/package-lock.json [Content-Type=application/json]...
Copying file://monolith-to-microservices/microservices/src/orders/Dockerfile [Content-Type=application/octet-stream]...
Copying file://monolith-to-microservices/microservices/src/orders/server.js [Content-Type=text/javascript]...
Copying file://monolith-to-microservices/microservices/src/orders/.dockerignore [Content-Type=application/octet-stream]...
Copying file://monolith-to-microservices/microservices/src/orders/data/orders.json [Content-Type=application/json]...
Copying file://monolith-to-microservices/microservices/src/orders/k8s/service.yml [Content-Type=application/octet-stream]...
Copying file://monolith-to-microservices/microservices/src/orders/k8s/deployment.yml [Content-Type=application/octet-stream]...
Copying file://monolith-to-microservices/microservices/src/frontend/package.json [Content-Type=application/json]...
Copying file://monolith-to-microservices/microservices/src/frontend/.gitignore [Content-Type=application/octet-stream]...
Copying file://monolith-to-microservices/microservices/src/frontend/package-lock.json [Content-Type=application/json]...
Copying file://monolith-to-microservices/microservices/src/frontend/.gcloudignore [Content-Type=application/octet-stream]...
Copying file://monolith-to-microservices/microservices/src/frontend/Dockerfile [Content-Type=application/octet-stream]...
Copying file://monolith-to-microservices/microservices/src/frontend/server.js [Content-Type=text/javascript]...
Copying file://monolith-to-microservices/microservices/src/frontend/.dockerignore [Content-Type=application/octet-stream]...
Copying file://monolith-to-microservices/microservices/src/frontend/k8s/service.yml [Content-Type=application/octet-stream]...
Copying file://monolith-to-microservices/microservices/src/frontend/k8s/deployment.yml [Content-Type=application/octet-stream]...
Copying file://monolith-to-microservices/microservices/src/frontend/public/index.html [Content-Type=text/html]...
Copying file://monolith-to-microservices/microservices/src/frontend/public/asset-manifest.json [Content-Type=application/json]...
Copying file://monolith-to-microservices/microservices/src/frontend/public/manifest.json [Content-Type=application/json]...
Copying file://monolith-to-microservices/microservices/src/frontend/public/robots.txt [Content-Type=text/plain]...
Copying file://monolith-to-microservices/microservices/src/frontend/public/static/img/products/record-player.jpg [Content-Type=image/jpeg]...
Copying file://monolith-to-microservices/microservices/src/frontend/public/static/img/products/film-camera.jpg [Content-Type=image/jpeg]...
Copying file://monolith-to-microservices/microservices/src/frontend/public/static/img/products/air-plant.jpg [Content-Type=image/jpeg]...
Copying file://monolith-to-microservices/microservices/src/frontend/public/static/img/products/credits.txt [Content-Type=text/plain]...
Copying file://monolith-to-microservices/microservices/src/frontend/public/static/img/products/typewriter.jpg [Content-Type=image/jpeg]...
Copying file://monolith-to-microservices/microservices/src/frontend/public/static/img/products/barista-kit.jpg [Content-Type=image/jpeg]...
Copying file://monolith-to-microservices/microservices/src/frontend/public/static/img/products/camp-mug.jpg [Content-Type=image/jpeg]...
Copying file://monolith-to-microservices/microservices/src/frontend/public/static/img/products/terrarium.jpg [Content-Type=image/jpeg]...
Copying file://monolith-to-microservices/microservices/src/frontend/public/static/img/products/camera-lens.jpg [Content-Type=image/jpeg]...
Copying file://monolith-to-microservices/microservices/src/frontend/public/static/img/products/city-bike.jpg [Content-Type=image/jpeg]...
Copying file://monolith-to-microservices/microservices/src/frontend/public/static/js/main.d934a9e5.js [Content-Type=text/javascript]...
Copying file://monolith-to-microservices/microservices/src/frontend/public/static/js/main.d934a9e5.js.map [Content-Type=application/octet-stream]...
Copying file://monolith-to-microservices/microservices/src/frontend/public/static/js/main.d934a9e5.js.LICENSE.txt [Content-Type=text/plain]...
Copying file://monolith-to-microservices/react-app/package.json [Content-Type=application/json]...
Copying file://monolith-to-microservices/react-app/.env [Content-Type=application/octet-stream]...
Copying file://monolith-to-microservices/react-app/.gitignore [Content-Type=application/octet-stream]...
Copying file://monolith-to-microservices/react-app/package-lock.json [Content-Type=application/json]...
Copying file://monolith-to-microservices/react-app/README.md [Content-Type=text/markdown]...
Copying file://monolith-to-microservices/react-app/.env.monolith [Content-Type=application/octet-stream]...
Copying file://monolith-to-microservices/react-app/build/index.html [Content-Type=text/html]...
Copying file://monolith-to-microservices/react-app/build/asset-manifest.json [Content-Type=application/json]...
Copying file://monolith-to-microservices/react-app/build/manifest.json [Content-Type=application/json]...
Copying file://monolith-to-microservices/react-app/build/robots.txt [Content-Type=text/plain]...
Copying file://monolith-to-microservices/react-app/build/static/img/products/record-player.jpg [Content-Type=image/jpeg]...
Copying file://monolith-to-microservices/react-app/build/static/img/products/film-camera.jpg [Content-Type=image/jpeg]...
Copying file://monolith-to-microservices/react-app/build/static/img/products/air-plant.jpg [Content-Type=image/jpeg]...
Copying file://monolith-to-microservices/react-app/build/static/img/products/credits.txt [Content-Type=text/plain]...
Copying file://monolith-to-microservices/react-app/build/static/img/products/barista-kit.jpg [Content-Type=image/jpeg]...
Copying file://monolith-to-microservices/react-app/build/static/img/products/typewriter.jpg [Content-Type=image/jpeg]...
Copying file://monolith-to-microservices/react-app/build/static/img/products/terrarium.jpg [Content-Type=image/jpeg]...
Copying file://monolith-to-microservices/react-app/build/static/img/products/camp-mug.jpg [Content-Type=image/jpeg]...
Copying file://monolith-to-microservices/react-app/build/static/img/products/camera-lens.jpg [Content-Type=image/jpeg]...
Copying file://monolith-to-microservices/react-app/build/static/img/products/city-bike.jpg [Content-Type=image/jpeg]...
Copying file://monolith-to-microservices/react-app/build/static/js/main.d934a9e5.js [Content-Type=text/javascript]...
Copying file://monolith-to-microservices/react-app/build/static/js/main.d934a9e5.js.map [Content-Type=application/octet-stream]...
Copying file://monolith-to-microservices/react-app/build/static/js/main.d934a9e5.js.LICENSE.txt [Content-Type=text/plain]...
Copying file://monolith-to-microservices/react-app/public/index.html [Content-Type=text/html]...
Copying file://monolith-to-microservices/react-app/public/manifest.json [Content-Type=application/json]...
Copying file://monolith-to-microservices/react-app/public/robots.txt [Content-Type=text/plain]...
Copying file://monolith-to-microservices/react-app/public/static/img/products/record-player.jpg [Content-Type=image/jpeg]...
Copying file://monolith-to-microservices/react-app/public/static/img/products/film-camera.jpg [Content-Type=image/jpeg]...
Copying file://monolith-to-microservices/react-app/public/static/img/products/air-plant.jpg [Content-Type=image/jpeg]...
Copying file://monolith-to-microservices/react-app/public/static/img/products/credits.txt [Content-Type=text/plain]...
Copying file://monolith-to-microservices/react-app/public/static/img/products/barista-kit.jpg [Content-Type=image/jpeg]...
Copying file://monolith-to-microservices/react-app/public/static/img/products/typewriter.jpg [Content-Type=image/jpeg]...
Copying file://monolith-to-microservices/react-app/public/static/img/products/terrarium.jpg [Content-Type=image/jpeg]...
Copying file://monolith-to-microservices/react-app/public/static/img/products/camp-mug.jpg [Content-Type=image/jpeg]...
Copying file://monolith-to-microservices/react-app/public/static/img/products/camera-lens.jpg [Content-Type=image/jpeg]...
Copying file://monolith-to-microservices/react-app/public/static/img/products/city-bike.jpg [Content-Type=image/jpeg]...
Copying file://monolith-to-microservices/react-app/src/App.js [Content-Type=text/javascript]...
Copying file://monolith-to-microservices/react-app/src/index.js [Content-Type=text/javascript]...
Copying file://monolith-to-microservices/react-app/src/pages/OrderDetails/index.js [Content-Type=text/javascript]...
Copying file://monolith-to-microservices/react-app/src/pages/Orders/index.js [Content-Type=text/javascript]...
Copying file://monolith-to-microservices/react-app/src/pages/NotFound/index.js [Content-Type=text/javascript]...
Copying file://monolith-to-microservices/react-app/src/components/ClippedDrawer/index.js [Content-Type=text/javascript]...
Copying file://monolith-to-microservices/react-app/src/pages/Home/index.js [Content-Type=text/javascript]...
Copying file://monolith-to-microservices/react-app/src/pages/Home/index.js.new [Content-Type=application/octet-stream]...
Copying file://monolith-to-microservices/react-app/src/pages/Products/index.js [Content-Type=text/javascript]...
Copying file://monolith-to-microservices/react-app/scripts/post-build.js [Content-Type=text/javascript]...
Copying file://monolith-to-microservices/.git/description [Content-Type=application/octet-stream]...
Copying file://monolith-to-microservices/.git/packed-refs [Content-Type=application/octet-stream]...
Copying file://monolith-to-microservices/.git/config [Content-Type=application/octet-stream]...
Copying file://monolith-to-microservices/.git/HEAD [Content-Type=application/octet-stream]...
Copying file://monolith-to-microservices/.git/index [Content-Type=application/octet-stream]...
Copying file://monolith-to-microservices/.git/logs/HEAD [Content-Type=application/octet-stream]...
Copying file://monolith-to-microservices/.git/logs/refs/heads/master [Content-Type=application/octet-stream]...
Copying file://monolith-to-microservices/.git/logs/refs/remotes/origin/HEAD [Content-Type=application/octet-stream]...
Copying file://monolith-to-microservices/.git/objects/pack/pack-9101dc88562686bc44dd54dbac2b6a740ed36c42.idx [Content-Type=application/octet-stream]...
Copying file://monolith-to-microservices/.git/objects/pack/pack-9101dc88562686bc44dd54dbac2b6a740ed36c42.pack [Content-Type=application/octet-stream]...
Copying file://monolith-to-microservices/.git/refs/heads/master [Content-Type=application/octet-stream]...
Copying file://monolith-to-microservices/.git/refs/remotes/origin/HEAD [Content-Type=application/octet-stream]...
Copying file://monolith-to-microservices/.git/info/exclude [Content-Type=application/octet-stream]...
Copying file://monolith-to-microservices/.git/hooks/update.sample [Content-Type=application/octet-stream]...
Copying file://monolith-to-microservices/.git/hooks/pre-merge-commit.sample [Content-Type=application/octet-stream]...
Copying file://monolith-to-microservices/.git/hooks/pre-applypatch.sample [Content-Type=application/octet-stream]...
Copying file://monolith-to-microservices/.git/hooks/pre-push.sample [Content-Type=application/octet-stream]...
Copying file://monolith-to-microservices/.git/hooks/pre-commit.sample [Content-Type=application/octet-stream]...
Copying file://monolith-to-microservices/.git/hooks/commit-msg.sample [Content-Type=application/octet-stream]...
Copying file://monolith-to-microservices/.git/hooks/prepare-commit-msg.sample [Content-Type=application/octet-stream]...
Copying file://monolith-to-microservices/.git/hooks/push-to-checkout.sample [Content-Type=application/octet-stream]...
Copying file://monolith-to-microservices/.git/hooks/applypatch-msg.sample [Content-Type=application/octet-stream]...
Copying file://monolith-to-microservices/.git/hooks/fsmonitor-watchman.sample [Content-Type=application/octet-stream]...
Copying file://monolith-to-microservices/.git/hooks/post-update.sample [Content-Type=application/octet-stream]...
Copying file://monolith-to-microservices/.git/hooks/pre-rebase.sample [Content-Type=application/octet-stream]...
Copying file://monolith-to-microservices/.git/hooks/pre-receive.sample [Content-Type=application/octet-stream]...
| [155/155 files][ 13.6 MiB/ 13.6 MiB] 100% Done   1.3 MiB/s ETA 00:00:00       
Operation completed over 155 objects/13.6 MiB.                                   
student_01_cad4a58911a5@cloudshell:~ (qwiklabs-gcp-04-83eb8e6fd864)$ 
```

### Update the frontend instances

Now that there is new code and configuration, you want the frontend  instances within the managed instance group to pull the new code.

- Since your instances pull the code at startup, you can issue a rolling restart command:

  ```sh
  student_01_cad4a58911a5@cloudshell:~ (qwiklabs-gcp-04-83eb8e6fd864)$ gcloud compute instance-groups managed rolling-action replace fancy-fe-mig \
      --zone=$ZONE \
      --max-unavailable 100%
  Updated [https://www.googleapis.com/compute/v1/projects/qwiklabs-gcp-04-83eb8e6fd864/zones/us-west1-a/instanceGroupManagers/fancy-fe-mig].
  ---
  autoHealingPolicies:
  - healthCheck: https://www.googleapis.com/compute/v1/projects/qwiklabs-gcp-04-83eb8e6fd864/global/healthChecks/fancy-fe-hc
    initialDelaySec: 300
  baseInstanceName: fancy-fe
  creationTimestamp: '2024-04-23T02:54:03.335-07:00'
  currentActions:
    abandoning: 0
    creating: 1
    creatingWithoutRetries: 0
    deleting: 2
    none: 0
    recreating: 0
    refreshing: 0
    restarting: 0
    resuming: 0
    starting: 0
    stopping: 0
    suspending: 0
    verifying: 0
  fingerprint: 3ioPM6Mp-qE=
  id: '3177438549801319380'
  instanceGroup: https://www.googleapis.com/compute/v1/projects/qwiklabs-gcp-04-83eb8e6fd864/zones/us-west1-a/instanceGroups/fancy-fe-mig
  instanceLifecyclePolicy:
    defaultActionOnFailure: REPAIR
    forceUpdateOnRepair: NO
  instanceTemplate: https://www.googleapis.com/compute/v1/projects/qwiklabs-gcp-04-83eb8e6fd864/global/instanceTemplates/fancy-fe
  kind: compute#instanceGroupManager
  listManagedInstancesResults: PAGELESS
  name: fancy-fe-mig
  namedPorts:
  - name: frontend
    port: 8080
  selfLink: https://www.googleapis.com/compute/v1/projects/qwiklabs-gcp-04-83eb8e6fd864/zones/us-west1-a/instanceGroupManagers/fancy-fe-mig
  status:
    allInstancesConfig:
      effective: true
    isStable: false
    stateful:
      hasStatefulConfig: false
      perInstanceConfigs:
        allEffective: true
    versionTarget:
      isReached: false
  targetSize: 2
  updatePolicy:
    maxSurge:
      calculated: 1
      fixed: 1
    maxUnavailable:
      calculated: 2
      percent: 100
    minimalAction: REPLACE
    replacementMethod: SUBSTITUTE
    type: PROACTIVE
  versions:
  - instanceTemplate: https://www.googleapis.com/compute/v1/projects/qwiklabs-gcp-04-83eb8e6fd864/global/instanceTemplates/fancy-fe
    name: 0/2024-04-23 10:19:30.428990+00:00
    targetSize:
      calculated: 2
  zone: https://www.googleapis.com/compute/v1/projects/qwiklabs-gcp-04-83eb8e6fd864/zones/us-west1-a
  student_01_cad4a58911a5@cloudshell:~ (qwiklabs-gcp-04-83eb8e6fd864)$ 
  ```

  In this example of a rolling replace, you specifically state that all machines can be replaced immediately through the `--max-unavailable` parameter. Without this parameter, the command would keep an instance  alive while restarting others to ensure availability. For testing  purposes, you specify to replace all immediately for speed.

### Test the website

1. Wait 3 minutes after issuing the `rolling-action replace` command in order to give the instances time to be processed, and then  check the status of the managed instance group. Run the following to  confirm the service is listed as **HEALTHY**:

Wait until the 2 services are listed as **HEALTHY**.

Once both items appear as HEALTHY on the list, exit the `watch` command by pressing CTRL+C.

If one instance encounters an issue and is UNHEALTHY it should automatically be repaired. Wait for this to happen. 

If neither instance enters a HEALTHY state after waiting a little while,  something is wrong with the setup of the frontend instances that  accessing them on port 8080 doesn't work. Test this by browsing to the  instances directly on port 8080.

The application will be accessible via http://[LB_IP]  where [LB_IP] is the IP_ADDRESS specified for the Load Balancer, which  can be found with the following command:

```
gcloud compute forwarding-rules list --global
```

You'll be checking the application later in the lab.

```sh
watch -n 2 gcloud compute backend-services get-health fancy-fe-frontend --global
```



```sh
Every 2.0s: gcloud compute backend-services get-health fancy-fe-frontend --global                                                  cs-666271300898-default: Tue Apr 23 10:22:45 2024

---
backend: https://www.googleapis.com/compute/v1/projects/qwiklabs-gcp-04-83eb8e6fd864/zones/us-west1-a/instanceGroups/fancy-fe-mig
status:
  healthStatus:
  - healthState: HEALTHY
    instance: https://www.googleapis.com/compute/v1/projects/qwiklabs-gcp-04-83eb8e6fd864/zones/us-west1-a/instances/fancy-fe-gb8q
    ipAddress: 10.138.0.8
    port: 8080
  - healthState: HEALTHY
    instance: https://www.googleapis.com/compute/v1/projects/qwiklabs-gcp-04-83eb8e6fd864/zones/us-west1-a/instances/fancy-fe-tl6r
    ipAddress: 10.138.0.9
    port: 8080
  kind: compute#backendServiceGroupHealth

```

## Scaling Compute Engine

So far, you have created two managed instance groups with two  instances each. This configuration is fully functional, but a static  configuration regardless of load. Next, you create an autoscaling policy based on utilization to automatically scale each managed instance  group.

### Automatically resize by utilization

- To create the autoscaling policy, execute the following:

  ```sh
  student_01_cad4a58911a5@cloudshell:~ (qwiklabs-gcp-04-83eb8e6fd864)$ gcloud compute instance-groups managed set-autoscaling \
    fancy-fe-mig \
    --zone=$ZONE \
    --max-num-replicas 2 \
    --target-load-balancing-utilization 0.60
  Created [https://www.googleapis.com/compute/v1/projects/qwiklabs-gcp-04-83eb8e6fd864/zones/us-west1-a/autoscalers/fancy-fe-mig-m2nd].
  ---
  autoscalingPolicy:
    coolDownPeriodSec: 60
    loadBalancingUtilization:
      utilizationTarget: 0.6
    maxNumReplicas: 2
    minNumReplicas: 2
    mode: ON
  creationTimestamp: '2024-04-23T03:24:19.692-07:00'
  id: '7774246707026080444'
  kind: compute#autoscaler
  name: fancy-fe-mig-m2nd
  selfLink: https://www.googleapis.com/compute/v1/projects/qwiklabs-gcp-04-83eb8e6fd864/zones/us-west1-a/autoscalers/fancy-fe-mig-m2nd
  status: ACTIVE
  target: https://www.googleapis.com/compute/v1/projects/qwiklabs-gcp-04-83eb8e6fd864/zones/us-west1-a/instanceGroupManagers/fancy-fe-mig
  zone: https://www.googleapis.com/compute/v1/projects/qwiklabs-gcp-04-83eb8e6fd864/zones/us-west1-a
  student_01_cad4a58911a5@cloudshell:~ (qwiklabs-gcp-04-83eb8e6fd864)$ 
  ```

  ```sh
  student_01_cad4a58911a5@cloudshell:~ (qwiklabs-gcp-04-83eb8e6fd864)$ gcloud compute instance-groups managed set-autoscaling \
    fancy-be-mig \
    --zone=$ZONE \
    --max-num-replicas 2 \
    --target-load-balancing-utilization 0.60
  Created [https://www.googleapis.com/compute/v1/projects/qwiklabs-gcp-04-83eb8e6fd864/zones/us-west1-a/autoscalers/fancy-be-mig-pwdi].
  ---
  autoscalingPolicy:
    coolDownPeriodSec: 60
    loadBalancingUtilization:
      utilizationTarget: 0.6
    maxNumReplicas: 2
    minNumReplicas: 2
    mode: ON
  creationTimestamp: '2024-04-23T03:24:48.658-07:00'
  id: '2064664707260363423'
  kind: compute#autoscaler
  name: fancy-be-mig-pwdi
  selfLink: https://www.googleapis.com/compute/v1/projects/qwiklabs-gcp-04-83eb8e6fd864/zones/us-west1-a/autoscalers/fancy-be-mig-pwdi
  status: ACTIVE
  target: https://www.googleapis.com/compute/v1/projects/qwiklabs-gcp-04-83eb8e6fd864/zones/us-west1-a/instanceGroupManagers/fancy-be-mig
  zone: https://www.googleapis.com/compute/v1/projects/qwiklabs-gcp-04-83eb8e6fd864/zones/us-west1-a
  student_01_cad4a58911a5@cloudshell:~ (qwiklabs-gcp-04-83eb8e6fd864)$ 
  ```

  These commands create an autoscaler on the managed instance groups that  automatically adds instances when utilization is above 60% utilization,  and removes instances when the load balancer is below 60% utilization.

  ### Enable content delivery network

  Another feature that can help with scaling is to enable a Content Delivery Network service, to provide caching for the frontend.

  1. Execute the following command on the frontend service:

     ```sh
     student_01_cad4a58911a5@cloudshell:~ (qwiklabs-gcp-04-83eb8e6fd864)$ gcloud compute backend-services update fancy-fe-frontend \
         --enable-cdn --global
     Updated [https://www.googleapis.com/compute/v1/projects/qwiklabs-gcp-04-83eb8e6fd864/global/backendServices/fancy-fe-frontend].
     student_01_cad4a58911a5@cloudshell:~ (qwiklabs-gcp-04-83eb8e6fd864)$ 
     ```

     When a user requests content from the HTTP(S) load balancer, the  request arrives at a Google Front End (GFE) which first looks in the  Cloud CDN cache for a response to the user's request. If the GFE finds a cached response, the GFE sends the cached response to the user. This is called a cache hit.

     If the GFE can't find a cached response for the request, the GFE  makes a request directly to the backend. If the response to this request is cacheable, the GFE stores the response in the Cloud CDN cache so  that the cache can be used for subsequent requests.

## Update the website
### Updating instance template

Existing instance templates are not editable; however, since your  instances are stateless and all configuration is done through the  startup script, you only need to change the instance template if you  want to change the template settings . Now you're going to make a simple change to use a larger machine type and push that out.
    
Complete the following steps to:

- Update the `frontend` instance, which acts as the basis  for the instance template. During the update, put a file on the updated  version of the instance template's image, then update the instance  template, roll out the new template, and then confirm the file exists on the managed instance group instances.
- Modify the machine type of your instance template, by switching from the `e2-standard-2` machine type to `e2-small`.

1. Run the following command to modify the machine type of the frontend instance:

```sh
student_01_cad4a58911a5@cloudshell:~ (qwiklabs-gcp-04-83eb8e6fd864)$ gcloud compute instances set-machine-type frontend \
  --zone=$ZONE \
  --machine-type e2-small
Updated [https://www.googleapis.com/compute/v1/projects/qwiklabs-gcp-04-83eb8e6fd864/zones/us-west1-a/instances/frontend].
student_01_cad4a58911a5@cloudshell:~ (qwiklabs-gcp-04-83eb8e6fd864)$ 
```

Create the new Instance Template:

```sh
student_01_cad4a58911a5@cloudshell:~ (qwiklabs-gcp-04-83eb8e6fd864)$ gcloud compute instance-templates create fancy-fe-new \
    --region=$REGION \
    --source-instance=frontend \
    --source-instance-zone=$ZONE
Created [https://www.googleapis.com/compute/v1/projects/qwiklabs-gcp-04-83eb8e6fd864/global/instanceTemplates/fancy-fe-new].
NAME: fancy-fe-new
MACHINE_TYPE: e2-small
PREEMPTIBLE: 
CREATION_TIMESTAMP: 2024-04-23T03:29:09.584-07:00
student_01_cad4a58911a5@cloudshell:~ (qwiklabs-gcp-04-83eb8e6fd864)$ 
```

Roll out the updated instance template to the Managed Instance Group:

```sh
student_01_cad4a58911a5@cloudshell:~ (qwiklabs-gcp-04-83eb8e6fd864)$ gcloud compute instance-groups managed rolling-action start-update fancy-fe-mig \
  --zone=$ZONE \
  --version template=fancy-fe-new
Updated [https://www.googleapis.com/compute/v1/projects/qwiklabs-gcp-04-83eb8e6fd864/zones/us-west1-a/instanceGroupManagers/fancy-fe-mig].
---
autoHealingPolicies:
- healthCheck: https://www.googleapis.com/compute/v1/projects/qwiklabs-gcp-04-83eb8e6fd864/global/healthChecks/fancy-fe-hc
  initialDelaySec: 300
baseInstanceName: fancy-fe
creationTimestamp: '2024-04-23T02:54:03.335-07:00'
currentActions:
  abandoning: 0
  creating: 1
  creatingWithoutRetries: 0
  deleting: 2
  none: 0
  recreating: 0
  refreshing: 0
  restarting: 0
  resuming: 0
  starting: 0
  stopping: 0
  suspending: 0
  verifying: 0
fingerprint: g3Vf25O8xQ0=
id: '3177438549801319380'
instanceGroup: https://www.googleapis.com/compute/v1/projects/qwiklabs-gcp-04-83eb8e6fd864/zones/us-west1-a/instanceGroups/fancy-fe-mig
instanceLifecyclePolicy:
  defaultActionOnFailure: REPAIR
  forceUpdateOnRepair: NO
instanceTemplate: https://www.googleapis.com/compute/v1/projects/qwiklabs-gcp-04-83eb8e6fd864/global/instanceTemplates/fancy-fe-new
kind: compute#instanceGroupManager
listManagedInstancesResults: PAGELESS
name: fancy-fe-mig
namedPorts:
- name: frontend
  port: 8080
selfLink: https://www.googleapis.com/compute/v1/projects/qwiklabs-gcp-04-83eb8e6fd864/zones/us-west1-a/instanceGroupManagers/fancy-fe-mig
status:
  allInstancesConfig:
    effective: true
  autoscaler: https://www.googleapis.com/compute/v1/projects/qwiklabs-gcp-04-83eb8e6fd864/zones/us-west1-a/autoscalers/fancy-fe-mig-m2nd
  isStable: false
  stateful:
    hasStatefulConfig: false
    perInstanceConfigs:
      allEffective: true
  versionTarget:
    isReached: false
targetSize: 2
updatePolicy:
  maxSurge:
    calculated: 1
    fixed: 1
  maxUnavailable:
    calculated: 2
    percent: 100
  minimalAction: REPLACE
  replacementMethod: SUBSTITUTE
  type: PROACTIVE
versions:
- instanceTemplate: https://www.googleapis.com/compute/v1/projects/qwiklabs-gcp-04-83eb8e6fd864/global/instanceTemplates/fancy-fe-new
  targetSize:
    calculated: 2
zone: https://www.googleapis.com/compute/v1/projects/qwiklabs-gcp-04-83eb8e6fd864/zones/us-west1-a
student_01_cad4a58911a5@cloudshell:~ (qwiklabs-gcp-04-83eb8e6fd864)$ 
```

Wait 3 minutes, and then run the following to monitor the status of the update:

his will take a few moments.

Once you have at least 1 instance in the following condition:

- STATUS: **RUNNING**
- ACTION set to **None**
- INSTANCE_TEMPLATE: the new template name (**fancy-fe-new**)

1. **Copy** the name of one of the machines listed for use in the next command.

   ```sh
   Every 2.0s: gcloud compute instance-groups managed list-instances fancy-fe-mig --zone=us-west1-a                                   cs-666271300898-default: Tue Apr 23 10:31:15 2024
   
   NAME: fancy-fe-g651
   ZONE: us-west1-a
   STATUS: RUNNING
   HEALTH_STATE: TIMEOUT
   ACTION: VERIFYING
   INSTANCE_TEMPLATE: fancy-fe-new
   VERSION_NAME:
   LAST_ERROR:
   
   NAME: fancy-fe-jnfn
   ZONE: us-west1-a
   STATUS: RUNNING
   HEALTH_STATE: TIMEOUT
   ACTION: VERIFYING
   INSTANCE_TEMPLATE: fancy-fe-new
   VERSION_NAME:
   LAST_ERROR:
   ```

Run the following to see if the virtual machine is using the new machine type (e2-small), where [VM_NAME] is the newly created instance:

```sh
student_01_cad4a58911a5@cloudshell:~ (qwiklabs-gcp-04-83eb8e6fd864)$ gcloud compute instances describe fancy-fe-jnfn --zone=$ZONE | grep machineType
machineType: https://www.googleapis.com/compute/v1/projects/qwiklabs-gcp-04-83eb8e6fd864/zones/us-west1-a/machineTypes/e2-small
student_01_cad4a58911a5@cloudshell:~ (qwiklabs-gcp-04-83eb8e6fd864)$ 
```

### Make changes to the website

**Scenario:** Your marketing team has asked you to  change the homepage for your site. They think it should be more  informative of who your company is and what you actually sell.

**Task:** Add some text to the homepage to make the  marketing team happy! It looks like one of the developers has already  created the changes with the file name `index.js.new`. You can just copy this file to `index.js` and the changes should be reflected. Follow the instructions below to make the appropriate changes.

1. Run the following commands to copy the updated file to the correct file name:

```js
student_01_cad4a58911a5@cloudshell:~ (qwiklabs-gcp-04-83eb8e6fd864)$ cd ~/monolith-to-microservices/react-app/src/pages/Home
mv index.js.new index.js
student_01_cad4a58911a5@cloudshell:~/monolith-to-microservices/react-app/src/pages/Home (qwiklabs-gcp-04-83eb8e6fd864)$ cat ~/monolith-to-microservices/react-app/src/pages/Home/index.js
/*
Copyright 2019 Google LLC

Licensed under the Apache License, Version 2.0 (the "License");
you may not use this file except in compliance with the License.
You may obtain a copy of the License at

    https://www.apache.org/licenses/LICENSE-2.0

Unless required by applicable law or agreed to in writing, software
distributed under the License is distributed on an "AS IS" BASIS,
WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
See the License for the specific language governing permissions and
limitations under the License.
*/
import React from "react";
import { Box, Paper, Typography } from "@mui/material";

export default function Home() {
  return (
    <Box sx={{ flexGrow: 1 }}>
      <Paper
        elevation={3}
        sx={{
          width: "800px",
          margin: "0 auto",
          padding: (theme) => theme.spacing(3, 2),
        }}
      >
        <Typography variant="h5">Fancy Fashion &amp; Style Online</Typography>
        <br />
        <Typography variant="body1">
          Tired of mainstream fashion ideas, popular trends and societal norms?
          This line of lifestyle products will help you catch up with the Fancy
          trend and express your personal style. Start shopping Fancy items now!
        </Typography>
      </Paper>
    </Box>
  );
}
student_01_cad4a58911a5@cloudshell:~/monolith-to-microservices/react-app/src/pages/Home (qwiklabs-gcp-04-83eb8e6fd864)$ 
```

You updated the React components, but you need to build the React app to generate the static files.

1. Run the following command to build the React app and copy it into the monolith public directory:

   ```sh
   student_01_cad4a58911a5@cloudshell:~/monolith-to-microservices/react-app/src/pages/Home (qwiklabs-gcp-04-83eb8e6fd864)$ cd ~/monolith-to-microservices/react-app
   npm install && npm run-script build
   npm WARN deprecated source-map-url@0.4.1: See https://github.com/lydell/source-map-url#deprecated
   npm WARN deprecated svgo@1.3.2: This SVGO version is no longer supported. Upgrade to v2.x.x.
   
   added 1476 packages, and audited 1477 packages in 22s
   
   202 packages are looking for funding
     run `npm fund` for details
   
   16 vulnerabilities (8 moderate, 7 high, 1 critical)
   
   To address issues that do not require attention, run:
     npm audit fix
   
   To address all issues (including breaking changes), run:
     npm audit fix --force
   
   Run `npm audit` for details.
   
   > frontend@0.1.0 prebuild
   > npm run build:monolith
   
   
   > frontend@0.1.0 build:monolith
   > env-cmd -f .env.monolith react-scripts build
   
   Creating an optimized production build...
   Browserslist: caniuse-lite is outdated. Please run:
     npx update-browserslist-db@latest
     Why you should do it regularly: https://github.com/browserslist/update-db#readme
   Compiled successfully.
   
   File sizes after gzip:
   
     93.15 kB (+96 B)  build/static/js/main.6e282d58.js
   
   The project was built assuming it is hosted at /.
   You can control this with the homepage field in your package.json.
   
   The build folder is ready to be deployed.
   You may serve it with a static server:
   
     npm install -g serve
     serve -s build
   
   Find out more about deployment here:
   
     https://cra.link/deployment
   
   
   > frontend@0.1.0 postbuild:monolith
   > node scripts/post-build.js ./build ../monolith/public
   
   Deleting stale folder: ../monolith/public
   Deleted stale destination folder: ../monolith/public
   Copying files from ./build to ../monolith/public
   Copied ./build to ../monolith/public successfully!
   
   > frontend@0.1.0 build
   > react-scripts build
   
   Creating an optimized production build...
   Browserslist: caniuse-lite is outdated. Please run:
     npx update-browserslist-db@latest
     Why you should do it regularly: https://github.com/browserslist/update-db#readme
   Compiled successfully.
   
   File sizes after gzip:
   
     93.16 kB (+17 B)  build/static/js/main.731a7d6c.js
   
   The project was built assuming it is hosted at /.
   You can control this with the homepage field in your package.json.
   
   The build folder is ready to be deployed.
   You may serve it with a static server:
   
     npm install -g serve
     serve -s build
   
   Find out more about deployment here:
   
     https://cra.link/deployment
   
   
   > frontend@0.1.0 postbuild
   > node scripts/post-build.js ./build ../microservices/src/frontend/public
   
   Deleting stale folder: ../microservices/src/frontend/public
   Deleted stale destination folder: ../microservices/src/frontend/public
   Copying files from ./build to ../microservices/src/frontend/public
   Copied ./build to ../microservices/src/frontend/public successfully!
   student_01_cad4a58911a5@cloudshell:~/monolith-to-microservices/react-app (qwiklabs-gcp-04-83eb8e6fd864)$ 
   ```

   Then re-push this code to the bucket:

   ```sh
   student_01_cad4a58911a5@cloudshell:~/monolith-to-microservices/react-app (qwiklabs-gcp-04-83eb8e6fd864)$ cd ~
   rm -rf monolith-to-microservices/*/node_modules
   gsutil -m cp -r monolith-to-microservices gs://fancy-store-$DEVSHELL_PROJECT_ID/
   Copying file://monolith-to-microservices/README.md [Content-Type=text/markdown]...
   Copying file://monolith-to-microservices/setup.sh [Content-Type=text/x-sh]...   
   Copying file://monolith-to-microservices/.gitignore [Content-Type=application/octet-stream]...
   Copying file://monolith-to-microservices/deploy-monolith.sh [Content-Type=text/x-sh]...
   Copying file://monolith-to-microservices/startup-script.sh [Content-Type=text/x-sh]...
   Copying file://monolith-to-microservices/package-lock.json [Content-Type=application/json]...
   Copying file://monolith-to-microservices/CONTRIBUTING.md [Content-Type=text/markdown]...
   Copying file://monolith-to-microservices/LICENSE [Content-Type=application/octet-stream]...
   Copying file://monolith-to-microservices/monolith/package.json [Content-Type=application/json]...
   Copying file://monolith-to-microservices/monolith/.gitignore [Content-Type=application/octet-stream]...
   Copying file://monolith-to-microservices/monolith/package-lock.json [Content-Type=application/json]...
   Copying file://monolith-to-microservices/monolith/.gcloudignore [Content-Type=application/octet-stream]...
   Copying file://monolith-to-microservices/monolith/Dockerfile [Content-Type=application/octet-stream]...
   Copying file://monolith-to-microservices/monolith/data/orders.json [Content-Type=application/json]...
   Copying file://monolith-to-microservices/monolith/.dockerignore [Content-Type=application/octet-stream]...
   Copying file://monolith-to-microservices/monolith/data/products.json [Content-Type=application/json]...
   Copying file://monolith-to-microservices/monolith/k8s/deployment.yml [Content-Type=application/octet-stream]...
   Copying file://monolith-to-microservices/monolith/k8s/service.yml [Content-Type=application/octet-stream]...
   Copying file://monolith-to-microservices/monolith/public/index.html [Content-Type=text/html]...
   Copying file://monolith-to-microservices/monolith/public/asset-manifest.json [Content-Type=application/json]...
   Copying file://monolith-to-microservices/monolith/public/manifest.json [Content-Type=application/json]...
   Copying file://monolith-to-microservices/monolith/public/robots.txt [Content-Type=text/plain]...
   Copying file://monolith-to-microservices/monolith/public/static/img/products/record-player.jpg [Content-Type=image/jpeg]...
   Copying file://monolith-to-microservices/monolith/public/static/img/products/film-camera.jpg [Content-Type=image/jpeg]...
   Copying file://monolith-to-microservices/monolith/public/static/img/products/air-plant.jpg [Content-Type=image/jpeg]...
   Copying file://monolith-to-microservices/monolith/public/static/img/products/credits.txt [Content-Type=text/plain]...
   Copying file://monolith-to-microservices/monolith/public/static/img/products/barista-kit.jpg [Content-Type=image/jpeg]...
   Copying file://monolith-to-microservices/monolith/public/static/img/products/terrarium.jpg [Content-Type=image/jpeg]...
   Copying file://monolith-to-microservices/monolith/public/static/img/products/typewriter.jpg [Content-Type=image/jpeg]...
   Copying file://monolith-to-microservices/monolith/public/static/img/products/camp-mug.jpg [Content-Type=image/jpeg]...
   Copying file://monolith-to-microservices/monolith/public/static/img/products/camera-lens.jpg [Content-Type=image/jpeg]...
   Copying file://monolith-to-microservices/monolith/public/static/img/products/city-bike.jpg [Content-Type=image/jpeg]...
   Copying file://monolith-to-microservices/monolith/public/static/js/main.6e282d58.js [Content-Type=text/javascript]...
   Copying file://monolith-to-microservices/monolith/public/static/js/main.6e282d58.js.LICENSE.txt [Content-Type=text/plain]...
   Copying file://monolith-to-microservices/monolith/public/static/js/main.6e282d58.js.map [Content-Type=application/octet-stream]...
   Copying file://monolith-to-microservices/monolith/src/server.js [Content-Type=text/javascript]...
   Copying file://monolith-to-microservices/microservices/package.json [Content-Type=application/json]...
   Copying file://monolith-to-microservices/microservices/.gitignore [Content-Type=application/octet-stream]...
   Copying file://monolith-to-microservices/microservices/package-lock.json [Content-Type=application/json]...
   Copying file://monolith-to-microservices/microservices/src/products/package.json [Content-Type=application/json]...
   Copying file://monolith-to-microservices/microservices/src/products/.gitignore [Content-Type=application/octet-stream]...
   Copying file://monolith-to-microservices/microservices/src/products/package-lock.json [Content-Type=application/json]...
   Copying file://monolith-to-microservices/microservices/src/products/Dockerfile [Content-Type=application/octet-stream]...
   Copying file://monolith-to-microservices/microservices/src/products/server.js [Content-Type=text/javascript]...
   Copying file://monolith-to-microservices/microservices/src/products/.dockerignore [Content-Type=application/octet-stream]...
   Copying file://monolith-to-microservices/microservices/src/products/data/products.json [Content-Type=application/json]...
   Copying file://monolith-to-microservices/microservices/src/products/k8s/service.yml [Content-Type=application/octet-stream]...
   Copying file://monolith-to-microservices/microservices/src/products/k8s/deployment.yml [Content-Type=application/octet-stream]...
   Copying file://monolith-to-microservices/microservices/src/orders/package.json [Content-Type=application/json]...
   Copying file://monolith-to-microservices/microservices/src/orders/.gitignore [Content-Type=application/octet-stream]...
   Copying file://monolith-to-microservices/microservices/src/orders/package-lock.json [Content-Type=application/json]...
   Copying file://monolith-to-microservices/microservices/src/orders/Dockerfile [Content-Type=application/octet-stream]...
   Copying file://monolith-to-microservices/microservices/src/orders/server.js [Content-Type=text/javascript]...
   Copying file://monolith-to-microservices/microservices/src/orders/.dockerignore [Content-Type=application/octet-stream]...
   Copying file://monolith-to-microservices/microservices/src/orders/data/orders.json [Content-Type=application/json]...
   Copying file://monolith-to-microservices/microservices/src/orders/k8s/service.yml [Content-Type=application/octet-stream]...
   Copying file://monolith-to-microservices/microservices/src/orders/k8s/deployment.yml [Content-Type=application/octet-stream]...
   Copying file://monolith-to-microservices/microservices/src/frontend/package.json [Content-Type=application/json]...
   Copying file://monolith-to-microservices/microservices/src/frontend/.gitignore [Content-Type=application/octet-stream]...
   Copying file://monolith-to-microservices/microservices/src/frontend/package-lock.json [Content-Type=application/json]...
   Copying file://monolith-to-microservices/microservices/src/frontend/.gcloudignore [Content-Type=application/octet-stream]...
   Copying file://monolith-to-microservices/microservices/src/frontend/Dockerfile [Content-Type=application/octet-stream]...
   Copying file://monolith-to-microservices/microservices/src/frontend/server.js [Content-Type=text/javascript]...
   Copying file://monolith-to-microservices/microservices/src/frontend/.dockerignore [Content-Type=application/octet-stream]...
   Copying file://monolith-to-microservices/microservices/src/frontend/public/index.html [Content-Type=text/html]...
   Copying file://monolith-to-microservices/microservices/src/frontend/k8s/deployment.yml [Content-Type=application/octet-stream]...
   Copying file://monolith-to-microservices/microservices/src/frontend/public/asset-manifest.json [Content-Type=application/json]...
   Copying file://monolith-to-microservices/microservices/src/frontend/k8s/service.yml [Content-Type=application/octet-stream]...
   Copying file://monolith-to-microservices/microservices/src/frontend/public/manifest.json [Content-Type=application/json]...
   Copying file://monolith-to-microservices/microservices/src/frontend/public/robots.txt [Content-Type=text/plain]...
   Copying file://monolith-to-microservices/microservices/src/frontend/public/static/img/products/record-player.jpg [Content-Type=image/jpeg]...
   Copying file://monolith-to-microservices/microservices/src/frontend/public/static/img/products/film-camera.jpg [Content-Type=image/jpeg]...
   Copying file://monolith-to-microservices/microservices/src/frontend/public/static/img/products/air-plant.jpg [Content-Type=image/jpeg]...
   Copying file://monolith-to-microservices/microservices/src/frontend/public/static/img/products/credits.txt [Content-Type=text/plain]...
   Copying file://monolith-to-microservices/microservices/src/frontend/public/static/img/products/barista-kit.jpg [Content-Type=image/jpeg]...
   Copying file://monolith-to-microservices/microservices/src/frontend/public/static/img/products/typewriter.jpg [Content-Type=image/jpeg]...
   Copying file://monolith-to-microservices/microservices/src/frontend/public/static/img/products/terrarium.jpg [Content-Type=image/jpeg]...
   Copying file://monolith-to-microservices/microservices/src/frontend/public/static/img/products/camp-mug.jpg [Content-Type=image/jpeg]...
   Copying file://monolith-to-microservices/microservices/src/frontend/public/static/img/products/camera-lens.jpg [Content-Type=image/jpeg]...
   Copying file://monolith-to-microservices/microservices/src/frontend/public/static/img/products/city-bike.jpg [Content-Type=image/jpeg]...
   Copying file://monolith-to-microservices/microservices/src/frontend/public/static/js/main.731a7d6c.js.map [Content-Type=application/octet-stream]...
   Copying file://monolith-to-microservices/microservices/src/frontend/public/static/js/main.731a7d6c.js [Content-Type=text/javascript]...
   Copying file://monolith-to-microservices/microservices/src/frontend/public/static/js/main.731a7d6c.js.LICENSE.txt [Content-Type=text/plain]...
   Copying file://monolith-to-microservices/react-app/package.json [Content-Type=application/json]...
   Copying file://monolith-to-microservices/react-app/.env [Content-Type=application/octet-stream]...
   Copying file://monolith-to-microservices/react-app/.gitignore [Content-Type=application/octet-stream]...
   Copying file://monolith-to-microservices/react-app/package-lock.json [Content-Type=application/json]...
   Copying file://monolith-to-microservices/react-app/README.md [Content-Type=text/markdown]...
   Copying file://monolith-to-microservices/react-app/.env.monolith [Content-Type=application/octet-stream]...
   Copying file://monolith-to-microservices/react-app/build/index.html [Content-Type=text/html]...
   Copying file://monolith-to-microservices/react-app/build/asset-manifest.json [Content-Type=application/json]...
   Copying file://monolith-to-microservices/react-app/build/manifest.json [Content-Type=application/json]...
   Copying file://monolith-to-microservices/react-app/build/robots.txt [Content-Type=text/plain]...
   Copying file://monolith-to-microservices/react-app/build/static/img/products/record-player.jpg [Content-Type=image/jpeg]...
   Copying file://monolith-to-microservices/react-app/build/static/img/products/film-camera.jpg [Content-Type=image/jpeg]...
   Copying file://monolith-to-microservices/react-app/build/static/img/products/air-plant.jpg [Content-Type=image/jpeg]...
   Copying file://monolith-to-microservices/react-app/build/static/img/products/credits.txt [Content-Type=text/plain]...
   Copying file://monolith-to-microservices/react-app/build/static/img/products/barista-kit.jpg [Content-Type=image/jpeg]...
   Copying file://monolith-to-microservices/react-app/build/static/img/products/typewriter.jpg [Content-Type=image/jpeg]...
   Copying file://monolith-to-microservices/react-app/build/static/img/products/terrarium.jpg [Content-Type=image/jpeg]...
   Copying file://monolith-to-microservices/react-app/build/static/img/products/camp-mug.jpg [Content-Type=image/jpeg]...
   Copying file://monolith-to-microservices/react-app/build/static/img/products/camera-lens.jpg [Content-Type=image/jpeg]...
   Copying file://monolith-to-microservices/react-app/build/static/img/products/city-bike.jpg [Content-Type=image/jpeg]...
   Copying file://monolith-to-microservices/react-app/build/static/js/main.731a7d6c.js.map [Content-Type=application/octet-stream]...
   Copying file://monolith-to-microservices/react-app/build/static/js/main.731a7d6c.js [Content-Type=text/javascript]...
   Copying file://monolith-to-microservices/react-app/build/static/js/main.731a7d6c.js.LICENSE.txt [Content-Type=text/plain]...
   Copying file://monolith-to-microservices/react-app/public/index.html [Content-Type=text/html]...
   Copying file://monolith-to-microservices/react-app/public/manifest.json [Content-Type=application/json]...
   Copying file://monolith-to-microservices/react-app/public/robots.txt [Content-Type=text/plain]...
   Copying file://monolith-to-microservices/react-app/public/static/img/products/record-player.jpg [Content-Type=image/jpeg]...
   Copying file://monolith-to-microservices/react-app/public/static/img/products/film-camera.jpg [Content-Type=image/jpeg]...
   Copying file://monolith-to-microservices/react-app/public/static/img/products/air-plant.jpg [Content-Type=image/jpeg]...
   Copying file://monolith-to-microservices/react-app/public/static/img/products/credits.txt [Content-Type=text/plain]...
   Copying file://monolith-to-microservices/react-app/public/static/img/products/barista-kit.jpg [Content-Type=image/jpeg]...
   Copying file://monolith-to-microservices/react-app/public/static/img/products/typewriter.jpg [Content-Type=image/jpeg]...
   Copying file://monolith-to-microservices/react-app/public/static/img/products/terrarium.jpg [Content-Type=image/jpeg]...
   Copying file://monolith-to-microservices/react-app/public/static/img/products/camp-mug.jpg [Content-Type=image/jpeg]...
   Copying file://monolith-to-microservices/react-app/public/static/img/products/camera-lens.jpg [Content-Type=image/jpeg]...
   Copying file://monolith-to-microservices/react-app/public/static/img/products/city-bike.jpg [Content-Type=image/jpeg]...
   Copying file://monolith-to-microservices/react-app/src/App.js [Content-Type=text/javascript]...
   Copying file://monolith-to-microservices/react-app/src/index.js [Content-Type=text/javascript]...
   Copying file://monolith-to-microservices/react-app/src/components/ClippedDrawer/index.js [Content-Type=text/javascript]...
   Copying file://monolith-to-microservices/react-app/src/pages/OrderDetails/index.js [Content-Type=text/javascript]...
   Copying file://monolith-to-microservices/react-app/src/pages/Orders/index.js [Content-Type=text/javascript]...
   Copying file://monolith-to-microservices/react-app/src/pages/NotFound/index.js [Content-Type=text/javascript]...
   Copying file://monolith-to-microservices/react-app/src/pages/Home/index.js [Content-Type=text/javascript]...
   Copying file://monolith-to-microservices/react-app/src/pages/Products/index.js [Content-Type=text/javascript]...
   Copying file://monolith-to-microservices/react-app/scripts/post-build.js [Content-Type=text/javascript]...
   Copying file://monolith-to-microservices/.git/description [Content-Type=application/octet-stream]...
   Copying file://monolith-to-microservices/.git/packed-refs [Content-Type=application/octet-stream]...
   Copying file://monolith-to-microservices/.git/config [Content-Type=application/octet-stream]...
   Copying file://monolith-to-microservices/.git/HEAD [Content-Type=application/octet-stream]...
   Copying file://monolith-to-microservices/.git/index [Content-Type=application/octet-stream]...
   Copying file://monolith-to-microservices/.git/logs/HEAD [Content-Type=application/octet-stream]...
   Copying file://monolith-to-microservices/.git/logs/refs/heads/master [Content-Type=application/octet-stream]...
   Copying file://monolith-to-microservices/.git/logs/refs/remotes/origin/HEAD [Content-Type=application/octet-stream]...
   Copying file://monolith-to-microservices/.git/objects/pack/pack-9101dc88562686bc44dd54dbac2b6a740ed36c42.idx [Content-Type=application/octet-stream]...
   Copying file://monolith-to-microservices/.git/objects/pack/pack-9101dc88562686bc44dd54dbac2b6a740ed36c42.pack [Content-Type=application/octet-stream]...
   Copying file://monolith-to-microservices/.git/refs/heads/master [Content-Type=application/octet-stream]...
   Copying file://monolith-to-microservices/.git/refs/remotes/origin/HEAD [Content-Type=application/octet-stream]...
   Copying file://monolith-to-microservices/.git/info/exclude [Content-Type=application/octet-stream]...
   Copying file://monolith-to-microservices/.git/hooks/update.sample [Content-Type=application/octet-stream]...
   Copying file://monolith-to-microservices/.git/hooks/pre-merge-commit.sample [Content-Type=application/octet-stream]...
   Copying file://monolith-to-microservices/.git/hooks/pre-applypatch.sample [Content-Type=application/octet-stream]...
   Copying file://monolith-to-microservices/.git/hooks/pre-push.sample [Content-Type=application/octet-stream]...
   Copying file://monolith-to-microservices/.git/hooks/pre-commit.sample [Content-Type=application/octet-stream]...
   Copying file://monolith-to-microservices/.git/hooks/commit-msg.sample [Content-Type=application/octet-stream]...
   Copying file://monolith-to-microservices/.git/hooks/prepare-commit-msg.sample [Content-Type=application/octet-stream]...
   Copying file://monolith-to-microservices/.git/hooks/push-to-checkout.sample [Content-Type=application/octet-stream]...
   Copying file://monolith-to-microservices/.git/hooks/applypatch-msg.sample [Content-Type=application/octet-stream]...
   Copying file://monolith-to-microservices/.git/hooks/post-update.sample [Content-Type=application/octet-stream]...
   Copying file://monolith-to-microservices/.git/hooks/fsmonitor-watchman.sample [Content-Type=application/octet-stream]...
   Copying file://monolith-to-microservices/.git/hooks/pre-rebase.sample [Content-Type=application/octet-stream]...
   Copying file://monolith-to-microservices/.git/hooks/pre-receive.sample [Content-Type=application/octet-stream]...
   / [154/154 files][ 13.6 MiB/ 13.6 MiB] 100% Done   1.1 MiB/s ETA 00:00:00       
   Operation completed over 154 objects/13.6 MiB.                                   
   student_01_cad4a58911a5@cloudshell:~ (qwiklabs-gcp-04-83eb8e6fd864)$ 
   ```

   

### Push changes with rolling replacements

1. Now force all instances to be replaced to pull the update:

   ```sh
   student_01_cad4a58911a5@cloudshell:~ (qwiklabs-gcp-04-83eb8e6fd864)$ gcloud compute instance-groups managed rolling-action replace fancy-fe-mig \
     --zone=$ZONE \
     --max-unavailable=100%
   Updated [https://www.googleapis.com/compute/v1/projects/qwiklabs-gcp-04-83eb8e6fd864/zones/us-west1-a/instanceGroupManagers/fancy-fe-mig].
   ---
   autoHealingPolicies:
   - healthCheck: https://www.googleapis.com/compute/v1/projects/qwiklabs-gcp-04-83eb8e6fd864/global/healthChecks/fancy-fe-hc
     initialDelaySec: 300
   baseInstanceName: fancy-fe
   creationTimestamp: '2024-04-23T02:54:03.335-07:00'
   currentActions:
     abandoning: 0
     creating: 1
     creatingWithoutRetries: 0
     deleting: 2
     none: 0
     recreating: 0
     refreshing: 0
     restarting: 0
     resuming: 0
     starting: 0
     stopping: 0
     suspending: 0
     verifying: 0
   fingerprint: aZDWVtdi1_Q=
   id: '3177438549801319380'
   instanceGroup: https://www.googleapis.com/compute/v1/projects/qwiklabs-gcp-04-83eb8e6fd864/zones/us-west1-a/instanceGroups/fancy-fe-mig
   instanceLifecyclePolicy:
     defaultActionOnFailure: REPAIR
     forceUpdateOnRepair: NO
   instanceTemplate: https://www.googleapis.com/compute/v1/projects/qwiklabs-gcp-04-83eb8e6fd864/global/instanceTemplates/fancy-fe-new
   kind: compute#instanceGroupManager
   listManagedInstancesResults: PAGELESS
   name: fancy-fe-mig
   namedPorts:
   - name: frontend
     port: 8080
   selfLink: https://www.googleapis.com/compute/v1/projects/qwiklabs-gcp-04-83eb8e6fd864/zones/us-west1-a/instanceGroupManagers/fancy-fe-mig
   status:
     allInstancesConfig:
       effective: true
     autoscaler: https://www.googleapis.com/compute/v1/projects/qwiklabs-gcp-04-83eb8e6fd864/zones/us-west1-a/autoscalers/fancy-fe-mig-m2nd
     isStable: false
     stateful:
       hasStatefulConfig: false
       perInstanceConfigs:
         allEffective: true
     versionTarget:
       isReached: false
   targetSize: 2
   updatePolicy:
     maxSurge:
       calculated: 1
       fixed: 1
     maxUnavailable:
       calculated: 2
       percent: 100
     minimalAction: REPLACE
     replacementMethod: SUBSTITUTE
     type: PROACTIVE
   versions:
   - instanceTemplate: https://www.googleapis.com/compute/v1/projects/qwiklabs-gcp-04-83eb8e6fd864/global/instanceTemplates/fancy-fe-new
     name: 0/2024-04-23 10:36:51.971900+00:00
     targetSize:
       calculated: 2
   zone: https://www.googleapis.com/compute/v1/projects/qwiklabs-gcp-04-83eb8e6fd864/zones/us-west1-a
   student_01_cad4a58911a5@cloudshell:~ (qwiklabs-gcp-04-83eb8e6fd864)$ 
   ```

   

Wait 3 minutes after issuing the `rolling-action replace`  command in order to give the instances time to be processed, and then  check the status of the managed instance group. Run the following to  confirm the service is listed as **HEALTHY**:

1. Once items appear in the list with HEALTHY status, exit the `watch` command by pressing CTRL+C.
2. Browse to the website via `http://[LB_IP]` where [LB_IP] is the IP_ADDRESS specified for the Load Balancer, which can be found with the following command:
3. The new website changes should now be visible.

```sh
Every 2.0s: gcloud compute backend-services get-health fancy-fe-frontend --global                                                  cs-666271300898-default: Tue Apr 23 10:41:15 2024

---
backend: https://www.googleapis.com/compute/v1/projects/qwiklabs-gcp-04-83eb8e6fd864/zones/us-west1-a/instanceGroups/fancy-fe-mig
status:
  healthStatus:
  - healthState: HEALTHY
    instance: https://www.googleapis.com/compute/v1/projects/qwiklabs-gcp-04-83eb8e6fd864/zones/us-west1-a/instances/fancy-fe-kkd6
    ipAddress: 10.138.0.12
    port: 8080
  - healthState: HEALTHY
    instance: https://www.googleapis.com/compute/v1/projects/qwiklabs-gcp-04-83eb8e6fd864/zones/us-west1-a/instances/fancy-fe-klfc
    ipAddress: 10.138.0.13
    port: 8080
  kind: compute#backendServiceGroupHealth

```

```sh
student_01_cad4a58911a5@cloudshell:~ (qwiklabs-gcp-04-83eb8e6fd864)$ gcloud compute forwarding-rules list --global
NAME: fancy-http-rule
REGION: 
IP_ADDRESS: 34.36.49.82
IP_PROTOCOL: TCP
TARGET: fancy-proxy
student_01_cad4a58911a5@cloudshell:~ (qwiklabs-gcp-04-83eb8e6fd864)$ 
```



### Simulate failure

In order to confirm the health check works, log in to an instance and stop the services.

1. To find an instance name, execute the following:

```sh
student_01_cad4a58911a5@cloudshell:~ (qwiklabs-gcp-04-83eb8e6fd864)$ gcloud compute instance-groups list-instances fancy-fe-mig --zone=$ZONE
NAME: fancy-fe-kkd6
ZONE: us-west1-a
STATUS: RUNNING

NAME: fancy-fe-klfc
ZONE: us-west1-a
STATUS: RUNNING
student_01_cad4a58911a5@cloudshell:~ (qwiklabs-gcp-04-83eb8e6fd864)$ 
```

Copy an instance name, then run the following to secure shell into the  instance, where INSTANCE_NAME is one of the instances from the list:

```sh
student_01_cad4a58911a5@cloudshell:~ (qwiklabs-gcp-04-83eb8e6fd864)$ gcloud compute ssh fancy-fe-kkd6 --zone=$ZONE
WARNING: The private SSH key file for gcloud does not exist.
WARNING: The public SSH key file for gcloud does not exist.
WARNING: You do not have an SSH key for gcloud.
WARNING: SSH keygen will be executed to generate a key.
This tool needs to create the directory [/home/student_01_cad4a58911a5/.ssh] before being able to generate SSH keys.

Do you want to continue (Y/n)?  y

Generating public/private rsa key pair.
Enter passphrase (empty for no passphrase): 
Enter same passphrase again: 
Your identification has been saved in /home/student_01_cad4a58911a5/.ssh/google_compute_engine
Your public key has been saved in /home/student_01_cad4a58911a5/.ssh/google_compute_engine.pub
The key fingerprint is:
SHA256:uozxP7bQTWJTLP+p5dECab3VPVzKrbS/w8q1ncsro6s student_01_cad4a58911a5@cs-666271300898-default
The key's randomart image is:
+---[RSA 3072]----+
|                 |
|         .       |
|        . o     .|
|         + o ..+o|
|        S * . =o+|
|       + * o * o.|
|    . o . . B =. |
|     = oo  +.=o++|
|    . +ooEoo+o+BB|
+----[SHA256]-----+
Warning: Permanently added 'compute.3222070084709790112' (ED25519) to the list of known hosts.
Linux fancy-fe-kkd6 5.10.0-28-cloud-amd64 #1 SMP Debian 5.10.209-2 (2024-01-31) x86_64

The programs included with the Debian GNU/Linux system are free software;
the exact distribution terms for each program are described in the
individual files in /usr/share/doc/*/copyright.

Debian GNU/Linux comes with ABSOLUTELY NO WARRANTY, to the extent
permitted by applicable law.
Creating directory '/home/student-01-cad4a58911a5'.
student-01-cad4a58911a5@fancy-fe-kkd6:~$ 
```

Within the instance, use `supervisorctl` to stop the application:

```sh
student-01-cad4a58911a5@fancy-fe-kkd6:~$ sudo supervisorctl stop nodeapp; sudo killall node
nodeapp: stopped
student-01-cad4a58911a5@fancy-fe-kkd6:~$ 
```

Exit the instance and monitor the repair operations

```sh
Every 2.0s: gcloud compute operations list --filter=operationType~compute.instances.repair.*                                       cs-666271300898-default: Tue Apr 23 10:45:32 2024

NAME: repair-1713869118817-616c14004c69a-9cf4dfed-622d6564
TYPE: compute.instances.repair.recreateInstance
TARGET: us-west1-a/instances/fancy-fe-kkd6
HTTP_STATUS: 200
STATUS: DONE
TIMESTAMP: 2024-04-23T03:45:18.818-07:00
```

The managed instance group recreated the instance to repair it.

1. You can also go to **Navigation menu** > **Compute Engine** > **VM instances** to monitor through the Console.

## Congratulations!

You successfully deployed, scaled, and updated your website on Compute Engine.

```sh
student_01_cad4a58911a5@cloudshell:~ (qwiklabs-gcp-04-83eb8e6fd864)$ history 
    1  gcloud config set compute/zone "us-west1-a"
    2  export ZONE=$(gcloud config get compute/zone)
    3  gcloud config set compute/region "us-west1"
    4  export REGION=$(gcloud config get compute/region)
    5  gcloud services enable compute.googleapis.com
    6  gsutil mb gs://fancy-store-$DEVSHELL_PROJECT_ID
    7  git clone https://github.com/googlecodelabs/monolith-to-microservices.git
    8  cd ~/monolith-to-microservices
    9  ./setup.sh
   10  nvm install --lts
   11  cd microservices
   12  npm start
   13  touch ~/monolith-to-microservices/startup-script.sh
   14  cat ~/monolith-to-microservices/startup-script.sh
   15  gsutil cp ~/monolith-to-microservices/startup-script.sh gs://fancy-store-$DEVSHELL_PROJECT_ID
   16  cd ~
   17  rm -rf monolith-to-microservices/*/node_modules
   18  gsutil -m cp -r monolith-to-microservices gs://fancy-store-$DEVSHELL_PROJECT_ID/
   19  gcloud compute instances create backend     --zone=$ZONE     --machine-type=e2-standard-2     --tags=backend    --metadata=startup-script-url=https://storage.googleapis.com/fancy-store-$DEVSHELL_PROJECT_ID/startup-script.sh
   20  gcloud compute instances list
   21  cd ~/monolith-to-microservices/react-app
   22  npm install && npm run-script build
   23  cd ~
   24  rm -rf monolith-to-microservices/*/node_modules
   25  gsutil -m cp -r monolith-to-microservices gs://fancy-store-$DEVSHELL_PROJECT_ID/
   26  gcloud compute instances create frontend     --zone=$ZONE     --machine-type=e2-standard-2     --tags=frontend     --metadata=startup-script-url=https://storage.googleapis.com/fancy-store-$DEVSHELL_PROJECT_ID/startup-script.sh
   27  gcloud compute firewall-rules create fw-fe     --allow tcp:8080     --target-tags=frontend
   28  gcloud compute firewall-rules create fw-be     --allow tcp:8081-8082     --target-tags=backend
   29  gcloud compute instances list
   30  gcloud compute instances stop frontend --zone=$ZONE
   31  gcloud compute instances stop backend --zone=$ZONE
   32  gcloud compute instance-templates create fancy-fe     --source-instance-zone=$ZONE     --source-instance=frontend
   33  gcloud compute instance-templates create fancy-be     --source-instance-zone=$ZONE     --source-instance=backend
   34  gcloud compute instance-templates list
   35  gcloud compute instances delete backend --zone=$ZONE
   36  gcloud compute instance-groups managed create fancy-fe-mig     --zone=$ZONE     --base-instance-name fancy-fe     --size 2     --template fancy-fe
   37  gcloud compute instance-groups managed create fancy-be-mig     --zone=$ZONE     --base-instance-name fancy-be     --size 2     --template fancy-be
   38  gcloud compute instance-groups set-named-ports fancy-fe-mig     --zone=$ZONE     --named-ports frontend:8080
   39  gcloud compute instance-groups set-named-ports fancy-be-mig     --zone=$ZONE     --named-ports orders:8081,products:8082
   40  gcloud compute health-checks create http fancy-fe-hc     --port 8080     --check-interval 30s     --healthy-threshold 1     --timeout 10s     --unhealthy-threshold 3
   41  gcloud compute health-checks create http fancy-be-hc     --port 8081     --request-path=/api/orders     --check-interval 30s     --healthy-threshold 1     --timeout 10s     --unhealthy-threshold 3
   42  gcloud compute firewall-rules create allow-health-check     --allow tcp:8080-8081     --source-ranges 130.211.0.0/22,35.191.0.0/16     --network default
   43  gcloud compute instance-groups managed update fancy-fe-mig     --zone=$ZONE     --health-check fancy-fe-hc     --initial-delay 300
   44  gcloud compute instance-groups managed update fancy-be-mig     --zone=$ZONE     --health-check fancy-be-hc     --initial-delay 300
   45  gcloud compute http-health-checks create fancy-fe-frontend-hc   --request-path /   --port 8080
   46  gcloud compute http-health-checks create fancy-be-orders-hc   --request-path /api/orders   --port 8081
   47  gcloud compute http-health-checks create fancy-be-products-hc   --request-path /api/products   --port 8082
   48  gcloud compute backend-services create fancy-fe-frontend   --http-health-checks fancy-fe-frontend-hc   --port-name frontend   --global
   49  gcloud compute backend-services create fancy-be-orders   --http-health-checks fancy-be-orders-hc   --port-name orders   --global
   50  gcloud compute backend-services create fancy-be-products   --http-health-checks fancy-be-products-hc   --port-name products   --global
   51  gcloud compute backend-services add-backend fancy-fe-frontend   --instance-group-zone=$ZONE   --instance-group fancy-fe-mig   --global
   52  gcloud compute backend-services add-backend fancy-be-orders   --instance-group-zone=$ZONE   --instance-group fancy-be-mig   --global
   53  gcloud compute backend-services add-backend fancy-be-products   --instance-group-zone=$ZONE   --instance-group fancy-be-mig   --global
   54  gcloud compute url-maps create fancy-map   --default-service fancy-fe-frontend
   55  gcloud compute url-maps add-path-matcher fancy-map    --default-service fancy-fe-frontend    --path-matcher-name orders    --path-rules "/api/orders=fancy-be-orders,/api/products=fancy-be-products"
   56  gcloud compute target-http-proxies create fancy-proxy   --url-map fancy-map
   57  gcloud compute forwarding-rules create fancy-http-rule   --global   --target-http-proxy fancy-proxy   --ports 80
   58  cd ~/monolith-to-microservices/react-app/
   59  gcloud compute forwarding-rules list --global
   60  cd ~/monolith-to-microservices/react-app
   61  npm install && npm run-script build
   62  cd ~
   63  rm -rf monolith-to-microservices/*/node_modules
   64  gsutil -m cp -r monolith-to-microservices gs://fancy-store-$DEVSHELL_PROJECT_ID/
   65  gcloud compute instance-groups managed rolling-action replace fancy-fe-mig     --zone=$ZONE     --max-unavailable 100%
   66  watch -n 2 gcloud compute backend-services get-health fancy-fe-frontend --global
   67  gcloud compute instance-groups managed set-autoscaling   fancy-fe-mig   --zone=$ZONE   --max-num-replicas 2   --target-load-balancing-utilization 0.60
   68  gcloud compute instance-groups managed set-autoscaling   fancy-be-mig   --zone=$ZONE   --max-num-replicas 2   --target-load-balancing-utilization 0.60
   69  gcloud compute backend-services update fancy-fe-frontend     --enable-cdn --global
   70  gcloud compute instances set-machine-type frontend   --zone=$ZONE   --machine-type e2-small
   71  gcloud compute instance-templates create fancy-fe-new     --region=$REGION     --source-instance=frontend     --source-instance-zone=$ZONE
   72  gcloud compute instance-groups managed rolling-action start-update fancy-fe-mig   --zone=$ZONE   --version template=fancy-fe-new
   73  watch -n 2 gcloud compute instance-groups managed list-instances fancy-fe-mig   --zone=$ZONE
   74  gcloud compute instances describe fancy-fe-jnfn --zone=$ZONE | grep machineType
   75  cd ~/monolith-to-microservices/react-app/src/pages/Home
   76  mv index.js.new index.js
   77  cat ~/monolith-to-microservices/react-app/src/pages/Home/index.js
   78  cd ~/monolith-to-microservices/react-app
   79  npm install && npm run-script build
   80  cd ~
   81  rm -rf monolith-to-microservices/*/node_modules
   82  gsutil -m cp -r monolith-to-microservices gs://fancy-store-$DEVSHELL_PROJECT_ID/
   83  gcloud compute instance-groups managed rolling-action replace fancy-fe-mig   --zone=$ZONE   --max-unavailable=100%
   84  watch -n 2 gcloud compute backend-services get-health fancy-fe-frontend --global
   85  gcloud compute forwarding-rules list --global
   86  gcloud compute instance-groups list-instances fancy-fe-mig --zone=$ZONE
   87  gcloud compute ssh fancy-fe-kkd6 --zone=$ZONE
   88  watch -n 2 gcloud compute operations list --filter='operationType~compute.instances.repair.*'
   89  history 
student_01_cad4a58911a5@cloudshell:~ (qwiklabs-gcp-04-83eb8e6fd864)$ 
```

