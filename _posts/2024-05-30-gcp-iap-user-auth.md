---

layout: single
title:  "User Authentication: Identity-Aware Proxy"
categories: Cloud
tags: GCP
classes: wide
show_date: true
header:
  overlay_image: /assets/images/gcp-banner.png
  og_image: /assets/images/gcp-banner.png
  teaser: /assets/images/pca-gcp.png
author:
  name     : "Professional Cloud Architect"
  avatar   : "/assets/images/pca-gcp.png"

sidebar:
  - title: "Topics"
    nav: my-sidebar

---
# User Authentication: Identity-Aware Proxy

Build a minimal web application with Google App Engine, then explore  various ways to use Identity-Aware Proxy (IAP) to restrict access to the application and provide user identity information to it. Your app will:

- Display a welcome page
- Access user identity information provided by IAP
- Use cryptographic verification to prevent spoofing of user identity information

Authenticating users of your web app is often necessary, and usually  requires special programming in your app. For Google Cloud apps you can  hand those responsibilities off to the  [Identity-Aware Proxy](https://cloud.google.com/iap/) service. If you only need to restrict access to selected users there  are no changes necessary to the application. Should the application need to know the user's identity (such as for keeping user preferences  server-side) Identity-Aware Proxy can provide that with minimal  application code.

Identity-Aware Proxy (IAP) is a Google Cloud service that intercepts web requests sent to your application, authenticates the user making the  request using the Google Identity Service, and only lets the requests  through if they come from a user you authorize. In addition, it can  modify the request headers to include information about the  authenticated user.

```sh
Welcome to Cloud Shell! Type "help" to get started.
Your Cloud Platform project in this session is set to qwiklabs-gcp-02-c134db2035a8.
Use “gcloud config set project [PROJECT_ID]” to change to a different project.
student_02_214654af9074@cloudshell:~ (qwiklabs-gcp-02-c134db2035a8)$ gsutil cp gs://spls/gsp499/user-authentication-with-iap.zip .
Copying gs://spls/gsp499/user-authentication-with-iap.zip...
/ [1 files][ 46.6 KiB/ 46.6 KiB]                                                
Operation completed over 1 objects/46.6 KiB.                                     
student_02_214654af9074@cloudshell:~ (qwiklabs-gcp-02-c134db2035a8)$ unzip user-authentication-with-iap.zip
Archive:  user-authentication-with-iap.zip
   creating: user-authentication-with-iap/
   creating: user-authentication-with-iap/2-HelloUser/
  inflating: user-authentication-with-iap/2-HelloUser/app.yaml  
 extracting: user-authentication-with-iap/2-HelloUser/requirements.txt  
   creating: user-authentication-with-iap/2-HelloUser/templates/
  inflating: user-authentication-with-iap/2-HelloUser/templates/index.html  
  inflating: user-authentication-with-iap/2-HelloUser/templates/privacy.html  
  inflating: user-authentication-with-iap/2-HelloUser/main.py  
  inflating: user-authentication-with-iap/LICENSE  
  inflating: user-authentication-with-iap/README.md  
   creating: user-authentication-with-iap/1-HelloWorld/
  inflating: user-authentication-with-iap/1-HelloWorld/app.yaml  
 extracting: user-authentication-with-iap/1-HelloWorld/requirements.txt  
   creating: user-authentication-with-iap/1-HelloWorld/templates/
  inflating: user-authentication-with-iap/1-HelloWorld/templates/index.html  
  inflating: user-authentication-with-iap/1-HelloWorld/templates/privacy.html  
  inflating: user-authentication-with-iap/1-HelloWorld/main.py  
  inflating: user-authentication-with-iap/CONTRIBUTING.md  
   creating: user-authentication-with-iap/3-HelloVerifiedUser/
  inflating: user-authentication-with-iap/3-HelloVerifiedUser/auth.py  
  inflating: user-authentication-with-iap/3-HelloVerifiedUser/app.yaml  
  inflating: user-authentication-with-iap/3-HelloVerifiedUser/requirements.txt  
   creating: user-authentication-with-iap/3-HelloVerifiedUser/templates/
  inflating: user-authentication-with-iap/3-HelloVerifiedUser/templates/index.html  
  inflating: user-authentication-with-iap/3-HelloVerifiedUser/templates/privacy.html  
  inflating: user-authentication-with-iap/3-HelloVerifiedUser/main.py  
   creating: user-authentication-with-iap/.git/
  inflating: user-authentication-with-iap/.git/config  
   creating: user-authentication-with-iap/.git/objects/
   creating: user-authentication-with-iap/.git/objects/pack/
  inflating: user-authentication-with-iap/.git/objects/pack/pack-2972c1d5574bc3f77dfc4cd84357b512bf5194a1.idx  
  inflating: user-authentication-with-iap/.git/objects/pack/pack-2972c1d5574bc3f77dfc4cd84357b512bf5194a1.pack  
   creating: user-authentication-with-iap/.git/objects/info/
 extracting: user-authentication-with-iap/.git/HEAD  
   creating: user-authentication-with-iap/.git/info/
  inflating: user-authentication-with-iap/.git/info/exclude  
   creating: user-authentication-with-iap/.git/logs/
  inflating: user-authentication-with-iap/.git/logs/HEAD  
   creating: user-authentication-with-iap/.git/logs/refs/
   creating: user-authentication-with-iap/.git/logs/refs/heads/
  inflating: user-authentication-with-iap/.git/logs/refs/heads/master  
   creating: user-authentication-with-iap/.git/logs/refs/remotes/
   creating: user-authentication-with-iap/.git/logs/refs/remotes/origin/
  inflating: user-authentication-with-iap/.git/logs/refs/remotes/origin/HEAD  
  inflating: user-authentication-with-iap/.git/description  
   creating: user-authentication-with-iap/.git/hooks/
  inflating: user-authentication-with-iap/.git/hooks/commit-msg.sample  
  inflating: user-authentication-with-iap/.git/hooks/pre-rebase.sample  
  inflating: user-authentication-with-iap/.git/hooks/pre-commit.sample  
  inflating: user-authentication-with-iap/.git/hooks/applypatch-msg.sample  
  inflating: user-authentication-with-iap/.git/hooks/fsmonitor-watchman.sample  
  inflating: user-authentication-with-iap/.git/hooks/pre-receive.sample  
  inflating: user-authentication-with-iap/.git/hooks/prepare-commit-msg.sample  
  inflating: user-authentication-with-iap/.git/hooks/post-update.sample  
  inflating: user-authentication-with-iap/.git/hooks/pre-merge-commit.sample  
  inflating: user-authentication-with-iap/.git/hooks/pre-applypatch.sample  
  inflating: user-authentication-with-iap/.git/hooks/pre-push.sample  
  inflating: user-authentication-with-iap/.git/hooks/update.sample  
   creating: user-authentication-with-iap/.git/refs/
   creating: user-authentication-with-iap/.git/refs/heads/
 extracting: user-authentication-with-iap/.git/refs/heads/master  
   creating: user-authentication-with-iap/.git/refs/tags/
   creating: user-authentication-with-iap/.git/refs/remotes/
   creating: user-authentication-with-iap/.git/refs/remotes/origin/
 extracting: user-authentication-with-iap/.git/refs/remotes/origin/HEAD  
  inflating: user-authentication-with-iap/.git/index  
   creating: user-authentication-with-iap/.git/branches/
  inflating: user-authentication-with-iap/.git/packed-refs  
student_02_214654af9074@cloudshell:~ (qwiklabs-gcp-02-c134db2035a8)$ cd user-authentication-with-iap
student_02_214654af9074@cloudshell:~/user-authentication-with-iap (qwiklabs-gcp-02-c134db2035a8)$ 
```

## Deploy the application and protect it with IAP

This is an App Engine Standard application written in Python that  simply displays a "Hello, World" welcome page. We will deploy and test  it, then restrict access to it using IAP.

### Review the application code

- Change from the main project folder to the `1-HelloWorld` subfolder that contains code for this step.

```sh
student_02_214654af9074@cloudshell:~/user-authentication-with-iap (qwiklabs-gcp-02-c134db2035a8)$ cd 1-HelloWorld
student_02_214654af9074@cloudshell:~/user-authentication-with-iap/1-HelloWorld (qwiklabs-gcp-02-c134db2035a8)$ ls
app.yaml  main.py  requirements.txt  templates
student_02_214654af9074@cloudshell:~/user-authentication-with-iap/1-HelloWorld (qwiklabs-gcp-02-c134db2035a8)$
```

```yaml
student_02_214654af9074@cloudshell:~/user-authentication-with-iap/1-HelloWorld (qwiklabs-gcp-02-c134db2035a8)$ cat app.yaml 
# Copyright 2019 Google LLC
#
# Licensed under the Apache License, Version 2.0 (the "License");
# you may not use this file except in compliance with the License.
# You may obtain a copy of the License at
#
#     http://www.apache.org/licenses/LICENSE-2.0
#
# Unless required by applicable law or agreed to in writing, software
# distributed under the License is distributed on an "AS IS" BASIS,
# WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
# See the License for the specific language governing permissions and
# limitations under the License.

runtime: python37
automatic_scaling:
  max_instances: 2
student_02_214654af9074@cloudshell:~/user-authentication-with-iap/1-HelloWorld (qwiklabs-gcp-02-c134db2035a8)$
```

```py
student_02_214654af9074@cloudshell:~/user-authentication-with-iap/1-HelloWorld (qwiklabs-gcp-02-c134db2035a8)$ cat main.py 
# Copyright 2019 Google LLC
#
# Licensed under the Apache License, Version 2.0 (the "License");
# you may not use this file except in compliance with the License.
# You may obtain a copy of the License at
#
#     http://www.apache.org/licenses/LICENSE-2.0
#
# Unless required by applicable law or agreed to in writing, software
# distributed under the License is distributed on an "AS IS" BASIS,
# WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
# See the License for the specific language governing permissions and
# limitations under the License.

from flask import Flask, render_template

app = Flask(__name__)


# Disable browser caching so changes in each step are always shown
@app.after_request
def set_response_headers(response):
    response.headers['Cache-Control'] = 'no-cache, no-store, must-revalidate'
    response.headers['Pragma'] = 'no-cache'
    response.headers['Expires'] = '0'
    return response

@app.route('/', methods=['GET'])
def say_hello():
    page = render_template('index.html')
    return page

@app.route('/privacy', methods=['GET'])
def show_policy():
    page = render_template('privacy.html')
    return page


if __name__ == '__main__':
    # This is used when running locally, only to verify it does not have
    # significant errors. It cannot demonstrate restricting access using
    # Identity-Aware Proxy when run locally, only when deployed.
    #
    # When deploying to Google App Engine, a webserver process such as
    # Gunicorn will serve the app. This can be configured by adding an
    # `entrypoint` to app.yaml.
    app.run(host='127.0.0.1', port=8080, debug=True)
student_02_214654af9074@cloudshell:~/user-authentication-with-iap/1-HelloWorld (qwiklabs-gcp-02-c134db2035a8)$ 
```

The application code is in the `main.py` file. It uses the  [Flask](http://flask.pocoo.org/) web framework to respond to web requests with the contents of a template. That template file is in `templates/index.html`, and for this step contains only plain HTML. A second template file contains a skeletal example privacy policy in `templates/privacy.html`.

There are two other files: `requirements.txt` lists all the non-default Python libraries the application uses, and `app.yaml` tells Google Cloud that this is a Python App Engine application.

### Deploy to App Engine

Update python runtime to `python39`. Deploy the app to the App Engine Standard environment for Python.

```sh
student_02_214654af9074@cloudshell:~/user-authentication-with-iap/1-HelloWorld (qwiklabs-gcp-02-c134db2035a8)$ sed -i 's/python37/python39/g' app.yaml
student_02_214654af9074@cloudshell:~/user-authentication-with-iap/1-HelloWorld (qwiklabs-gcp-02-c134db2035a8)$ gcloud app deploy
You are creating an app for project [qwiklabs-gcp-02-c134db2035a8].
WARNING: Creating an App Engine application for a project is irreversible and the region
cannot be changed. More information about regions is at
<https://cloud.google.com/appengine/docs/locations>.

Please choose the region where you want your App Engine application located:

 [1] asia-east1    (supports standard and flexible)
 [2] asia-northeast1 (supports standard and flexible and search_api)
 [3] asia-south1   (supports standard and flexible and search_api)
 [4] asia-southeast1 (supports standard and flexible)
 [5] australia-southeast1 (supports standard and flexible and search_api)
 [6] europe-central2 (supports standard and flexible)
 [7] europe-west   (supports standard and flexible and search_api)
 [8] europe-west2  (supports standard and flexible and search_api)
 [9] europe-west3  (supports standard and flexible and search_api)
 [10] us-central    (supports standard and flexible and search_api)
 [11] us-east1      (supports standard and flexible and search_api)
 [12] us-east4      (supports standard and flexible and search_api)
 [13] us-west1      (supports standard and flexible)
 [14] us-west2      (supports standard and flexible and search_api)
 [15] us-west3      (supports standard and flexible and search_api)
 [16] us-west4      (supports standard and flexible and search_api)
 [17] cancel
Please enter your numeric choice:  12

Creating App Engine application in project [qwiklabs-gcp-02-c134db2035a8] and region [us-east4]....done.                                                                           
Services to deploy:

descriptor:                  [/home/student_02_214654af9074/user-authentication-with-iap/1-HelloWorld/app.yaml]
source:                      [/home/student_02_214654af9074/user-authentication-with-iap/1-HelloWorld]
target project:              [qwiklabs-gcp-02-c134db2035a8]
target service:              [default]
target version:              [20240530t092941]
target url:                  [https://qwiklabs-gcp-02-c134db2035a8.uk.r.appspot.com]
target service account:      [qwiklabs-gcp-02-c134db2035a8@appspot.gserviceaccount.com]


Do you want to continue (Y/n)?  y

Beginning deployment of service [default]...
Created .gcloudignore file. See `gcloud topic gcloudignore` for details.
Uploading 6 files to Google Cloud Storage
17%
33%
50%
67%
83%
100%
100%
File upload done.
Updating service [default]...done.                                                                                                                                                 
Setting traffic split for service [default]...done.                                                                                                                                
Deployed service [default] to [https://qwiklabs-gcp-02-c134db2035a8.uk.r.appspot.com]

You can stream logs from the command line by running:
  $ gcloud app logs tail -s default

To view your application in the web browser run:
  $ gcloud app browse
student_02_214654af9074@cloudshell:~/user-authentication-with-iap/1-HelloWorld (qwiklabs-gcp-02-c134db2035a8)$ 
```

```sh
student_02_214654af9074@cloudshell:~/user-authentication-with-iap/1-HelloWorld (qwiklabs-gcp-02-c134db2035a8)$ gcloud app browse
Did not detect your browser. Go to this link to view your app:
https://qwiklabs-gcp-02-c134db2035a8.uk.r.appspot.com
student_02_214654af9074@cloudshell:~/user-authentication-with-iap/1-HelloWorld (qwiklabs-gcp-02-c134db2035a8)$ 
```

```html
student_02_214654af9074@cloudshell:~/user-authentication-with-iap/1-HelloWorld (qwiklabs-gcp-02-c134db2035a8)$ curl https://qwiklabs-gcp-02-c134db2035a8.uk.r.appspot.com
<!doctype html>
<html>
<head>
  <title>IAP Hello World</title>
</head>
<body>
  <h1>Hello World</h1>

  <p>
    Hello, world! This is step 1 of the <em>User Authentication with IAP</em>
    codelab.
  </p>

</body>
</html>student_02_214654af9074@cloudshell:~/user-authentication-with-iap/1-HelloWorld (qwiklabs-gcp-02-c134db2035a8)$ 
```

You can open that same URL from any computer connected to the Internet to see that web page. Access is not yet restricted.

### Restrict access with IAP

1. In the cloud console window, click the **Navigation menu** ![Navigation menu icon](https://cdn.qwiklabs.com/tkgw1TDgj4Q%2BYKQUW4jUFd0O5OEKlUMBRYbhlCrF0WY%3D) > **Security** > **Identity-Aware Proxy**.
2. Click **ENABLE API**.
3. Click **GO TO IDENTITY-AWARE PROXY**.
4. Click **CONFIGURE CONSENT SCREEN**.
5. Select **Internal** under User Type and click **Create**.

| **Field**                       | **Value**                                                    |
| ------------------------------- | ------------------------------------------------------------ |
| App name                        | IAP Example                                                  |
| User support email              | *Select your lab student email address from the dropdown.*   |
| Application home page           | *The URL you used to view your app. You can find this again by running the gcloud app browse command in cloud shell again.* |
| Application privacy Policy link | *The privacy page link in the app, same as the homepage link with `/privacy` added to the end* |
| Authorized domains              | *Click **+ ADD DOMAIN**The hostname portion of the  application's URL, e.g. iap-example-999999.appspot.com. You can see this in the address bar of the Hello World web page you previously opened.  Do not include the starting* `https://` *or trailing* `/` *from that URL.* |
| Developer Contact Information   |                                                              |

1. Click **Save and Continue**.
2. For **Scopes**, click **Save and Continue**.
3. For **Summary**, click **Back to Dashboard**.

You might be prompted to create credentials. You do not need to  create credentials for this lab, so you can simply close this browser  tab



```sh
student_02_214654af9074@cloudshell:~/user-authentication-with-iap/1-HelloWorld (qwiklabs-gcp-02-c134db2035a8)$ gcloud services disable appengineflex.googleapis.com
Operation "operations/acat.p17-819040408571-d4186b66-2630-42f0-904f-2a0c9c73b79d" finished successfully.
student_02_214654af9074@cloudshell:~/user-authentication-with-iap/1-HelloWorld (qwiklabs-gcp-02-c134db2035a8)$ 
```

App Engine has its standard and flexible environments which are  optimized for different application architectures. Currently, when  enabling IAP for App Engine, if the Flex API is enabled, Google Cloud  will look for a Flex Service Account. Your lab project comes with a  multitude of APIs already enabled for the purpose of convenience.  However, this creates a unique situation where the Flex API is enabled  without a Service Account created.



Return to the Identity-Aware Proxy page and refresh it. You should now see a list of resources you can protect.

Click the toggle button in the IAP column in the **App Engine app** row to turn **IAP** on.

The domain will be protected by IAP. Click **Turn On**.

### Test that  IAP is turned on

1. Open a browser tab and navigate to the URL for your app. A Sign in  with Google screen opens and requires you to log in to access the app.
2. Sign in with the account you used to log into the console. You will see a screen denying you access.



> You have successfully protected your app with IAP, but you have not yet told IAP which accounts to allow through.

1. Return to the Identity-Aware Proxy page of the console, select the checkbox next to **App Engine app**, and see the App Engine sidebar to the right.

Each email address (or Google Group address, or Workspace domain  name) that should be allowed access needs to be added as a Member.

1. Click **Add Principal**.
2. Enter your **Student** email address.
3. Then, pick the **Cloud IAP** > **IAP-Secured Web App User** role to assign to that address.

### Test access

Navigate back to your app and reload the page. You should now see  your web app, since you already logged in with a user you authorized.

If  you still see the "You don't have access" page, IAP did not  recheck your authorization. In that case, do the following steps:

1. Open your web browser to the home page address with `/_gcp_iap/clear_login_cookie` added to the end of the URL, as in `https://iap-example-999999.appspot.com/_gcp_iap/clear_login_cookie`.
2. You will see a new Sign in with Google screen, with your account  already showing. Do not click the account. Instead, click Use another  account, and re-enter your credentials.

These steps cause IAP to recheck your access and you should now see your application's home screen.

If you have access to another browser or can use Incognito Mode in  your browser, and have another valid Gmail or Workspace account, you can use that browser to navigate to your app page and log in with the other account. Since that account has not been authorized, it will see the  "You Don't Have Access" screen instead of your app.



## Access user identity information

Once an app is protected with IAP, it can use the identity  information that IAP provides in the web request headers it passes  through. In this step, the application will get the logged-in user's  email address and a persistent unique user ID assigned by the Google  Identity Service to that user. That data will be displayed to the user  in the welcome page.

### Deploy to App Engine

### Examine the application files

This folder contains the same set of files as seen in the previous app you deployed, `1-HelloWorld`, but two of the files have been changed: `main.py` and `templates/index.html`. The program has been changed to retrieve the user information that IAP  provides in request headers, and the template now displays that data.

There are two lines in `main.py` that get the IAP-provided identity data:

```py
user_email = request.headers.get('X-Goog-Authenticated-User-Email')
user_id = request.headers.get('X-Goog-Authenticated-User-ID')
```



The **X-Goog-Authenticated-User-** headers are provided by  IAP, and the names are case-insensitive, so they could be given in all  lower or all upper case if preferred. The render_template statement now  includes those values so they can be displayed:

The index.html template can display those values by enclosing the names in double curly braces:

`Hello, {{ email }}! Your persistent ID is {{ id }}.`

```py
student_02_214654af9074@cloudshell:~/user-authentication-with-iap/2-HelloUser (qwiklabs-gcp-02-c134db2035a8)$ cat main.py 
# Copyright 2019 Google LLC
#
# Licensed under the Apache License, Version 2.0 (the "License");
# you may not use this file except in compliance with the License.
# You may obtain a copy of the License at
#
#     http://www.apache.org/licenses/LICENSE-2.0
#
# Unless required by applicable law or agreed to in writing, software
# distributed under the License is distributed on an "AS IS" BASIS,
# WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
# See the License for the specific language governing permissions and
# limitations under the License.

from flask import Flask, render_template, request

app = Flask(__name__)


# Disable browser caching so changes in each step are always shown
@app.after_request
def set_response_headers(response):
    response.headers['Cache-Control'] = 'no-cache, no-store, must-revalidate'
    response.headers['Pragma'] = 'no-cache'
    response.headers['Expires'] = '0'
    return response

@app.route('/', methods=['GET'])
def say_hello():
    user_email = request.headers.get('X-Goog-Authenticated-User-Email')
    user_id = request.headers.get('X-Goog-Authenticated-User-ID')

    page = render_template('index.html',
        email=user_email,
        id=user_id)
    return page

@app.route('/privacy', methods=['GET'])
def show_policy():
    page = render_template('privacy.html')
    return page


if __name__ == '__main__':
    # This is used when running locally, only to verify it does not have
    # significant errors. It cannot demonstrate restricting access using
    # Identity-Aware Proxy when run locally, only when deployed.
    #
    # When deploying to Google App Engine, a webserver process such as
    # Gunicorn will serve the app. This can be configured by adding an
    # `entrypoint` to app.yaml.
    app.run(host='127.0.0.1', port=8080, debug=True)
student_02_214654af9074@cloudshell:~/user-authentication-with-iap/2-HelloUser (qwiklabs-gcp-02-c134db2035a8)$ 
```

```html
student_02_214654af9074@cloudshell:~/user-authentication-with-iap/2-HelloUser (qwiklabs-gcp-02-c134db2035a8)$ cat templates/index.html 
<!doctype html>
<html>
<head>
  <title>IAP Hello User</title>
</head>
<body>
  <h1>Hello User</h1>

  <p>
    Hello, {{ email }}! Your persistent ID is {{ id }}.
  </p>

  <p>
    This is step 2 of the <em>User Authentication with IAP</em>
    codelab.
  </p>

</body>
</html>
student_02_214654af9074@cloudshell:~/user-authentication-with-iap/2-HelloUser (qwiklabs-gcp-02-c134db2035a8)$ 
```



```sh
student_02_214654af9074@cloudshell:~/user-authentication-with-iap/1-HelloWorld (qwiklabs-gcp-02-c134db2035a8)$ cd ~/user-authentication-with-iap/2-HelloUser
student_02_214654af9074@cloudshell:~/user-authentication-with-iap/2-HelloUser (qwiklabs-gcp-02-c134db2035a8)$ sed -i 's/python37/python39/g' app.yaml
student_02_214654af9074@cloudshell:~/user-authentication-with-iap/2-HelloUser (qwiklabs-gcp-02-c134db2035a8)$ gcloud app deploy
Services to deploy:

descriptor:                  [/home/student_02_214654af9074/user-authentication-with-iap/2-HelloUser/app.yaml]
source:                      [/home/student_02_214654af9074/user-authentication-with-iap/2-HelloUser]
target project:              [qwiklabs-gcp-02-c134db2035a8]
target service:              [default]
target version:              [20240530t094525]
target url:                  [https://qwiklabs-gcp-02-c134db2035a8.uk.r.appspot.com]
target service account:      [qwiklabs-gcp-02-c134db2035a8@appspot.gserviceaccount.com]


Do you want to continue (Y/n)?  y

Beginning deployment of service [default]...
Created .gcloudignore file. See `gcloud topic gcloudignore` for details.
Uploading 3 files to Google Cloud Storage
33%
67%
100%
100%
File upload done.
Updating service [default]...done.                                                                                                                                                 
Setting traffic split for service [default]...done.                                                                                                                                
Deployed service [default] to [https://qwiklabs-gcp-02-c134db2035a8.uk.r.appspot.com]

You can stream logs from the command line by running:
  $ gcloud app logs tail -s default

To view your application in the web browser run:
  $ gcloud app browse
student_02_214654af9074@cloudshell:~/user-authentication-with-iap/2-HelloUser (qwiklabs-gcp-02-c134db2035a8)$ gcloud app browse
Did not detect your browser. Go to this link to view your app:
https://qwiklabs-gcp-02-c134db2035a8.uk.r.appspot.com
student_02_214654af9074@cloudshell:~/user-authentication-with-iap/2-HelloUser (qwiklabs-gcp-02-c134db2035a8)$ 
```

### Test the updated IAP

Going back to the deployment, when it is ready, you will see a message that you can view your application with `gcloud app browse`.

```html
Hello User

Hello, accounts.google.com:student-02-214654af9074@qwiklabs.net! Your persistent ID is accounts.google.com:116601044709378976788.

This is step 2 of the User Authentication with IAP codelab. 
```

### Turn off IAP

What happens to this app if IAP is disabled, or somehow bypassed  (such as by other applications running in your same cloud project)? Turn off IAP to see.

1. In the cloud console window, click **Navigation menu** > **Security** > **Identity-Aware Proxy**.
2. Click the **IAP** toggle switch next to App Engine app to turn **IAP** off. Click **TURN OFF**.

You will be warned that this will allow all users to access the app.

1. Refresh the application web page. You should see the same page, but without any user information:

```html
Hello User

Hello, None! Your persistent ID is None.

This is step 2 of the User Authentication with IAP codelab. 
```

Since the application is now unprotected, a user could send a web  request that appeared to have passed through IAP. For example, you can  run the following curl command from the Cloud Shell to do that .

```sh
student_02_214654af9074@cloudshell:~/user-authentication-with-iap/2-HelloUser (qwiklabs-gcp-02-c134db2035a8)$ curl -X GET https://qwiklabs-gcp-02-c134db2035a8.uk.r.appspot.com  -H 
"X-Goog-Authenticated-User-Email: totally fake email"
<!doctype html>
<html>
<head>
  <title>IAP Hello User</title>
</head>
<body>
  <h1>Hello User</h1>

  <p>
    Hello, totally fake email! Your persistent ID is None.
  </p>

  <p>
    This is step 2 of the <em>User Authentication with IAP</em>
    codelab.
  </p>

</body>
</html>student_02_214654af9074@cloudshell:~/user-authentication-with-iap/2-HelloUser (qwiklabs-gcp-02-c134db2035a8)$ 
```

There is no way for the application to know that IAP has been disabled  or bypassed. For cases where that is a potential risk, Cryptographic  Verification shows a solution.

## Use Cryptographic Verification

If there is a risk of IAP being turned off or bypassed, your app can  check to make sure the identity information it receives is valid. This  uses a third web request header added by IAP, called `X-Goog-IAP-JWT-Assertion`. The value of the header is a cryptographically signed object that also  contains the user identity data. Your application can verify the digital signature and use the data provided in this object to be certain that  it was provided by IAP without alteration.

Digital signature verification requires several extra steps, such as  retrieving the latest set of Google public keys. You can decide whether  your application needs these extra steps based on the risk that someone  might be able to turn off or bypass IAP, and the sensitivity of the  application.

```sh
student_02_214654af9074@cloudshell:~/user-authentication-with-iap/2-HelloUser (qwiklabs-gcp-02-c134db2035a8)$ cd ~/user-authentication-with-iap/3-HelloVerifiedUser
student_02_214654af9074@cloudshell:~/user-authentication-with-iap/3-HelloVerifiedUser (qwiklabs-gcp-02-c134db2035a8)$ sed -i 's/python37/python39/g' app.yaml
student_02_214654af9074@cloudshell:~/user-authentication-with-iap/3-HelloVerifiedUser (qwiklabs-gcp-02-c134db2035a8)$ cat main.py 
# Copyright 2019 Google LLC
#
# Licensed under the Apache License, Version 2.0 (the "License");
# you may not use this file except in compliance with the License.
# You may obtain a copy of the License at
#
#     http://www.apache.org/licenses/LICENSE-2.0
#
# Unless required by applicable law or agreed to in writing, software
# distributed under the License is distributed on an "AS IS" BASIS,
# WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
# See the License for the specific language governing permissions and
# limitations under the License.

from auth import user
from flask import Flask, render_template, request

app = Flask(__name__)


# Disable browser caching so changes in each step are always shown
@app.after_request
def set_response_headers(response):
    response.headers['Cache-Control'] = 'no-cache, no-store, must-revalidate'
    response.headers['Pragma'] = 'no-cache'
    response.headers['Expires'] = '0'
    return response

@app.route('/', methods=['GET'])
def say_hello():
    user_email = request.headers.get('X-Goog-Authenticated-User-Email')
    user_id = request.headers.get('X-Goog-Authenticated-User-ID')

    verified_email, verified_id = user()

    page = render_template('index.html',
        email=user_email,
        id=user_id,
        verified_email=verified_email,
        verified_id=verified_id)
    return page

@app.route('/privacy', methods=['GET'])
def show_policy():
    page = render_template('privacy.html')
    return page


if __name__ == '__main__':
    # This is used when running locally, only to verify it does not have
    # significant errors. It cannot demonstrate restricting access using
    # Identity-Aware Proxy when run locally, only when deployed.
    #
    # When deploying to Google App Engine, a webserver process such as
    # Gunicorn will serve the app. This can be configured by adding an
    # `entrypoint` to app.yaml.
    app.run(host='127.0.0.1', port=8080, debug=True)
student_02_214654af9074@cloudshell:~/user-authentication-with-iap/3-HelloVerifiedUser (qwiklabs-gcp-02-c134db2035a8)$ cat templates/index.html 
<!doctype html>
<html>
<head>
  <title>IAP Hello Verified User</title>
</head>
<body>
  <h1>Hello Verified User</h1>

  <p>
    Hello, {{ email }}! Your persistent ID appears to be {{ id }}.
  </p>

  <p>
    Your verified email is {{ verified_email }},
    and your verified persistent ID is {{ verified_id }}.
  </p>

  <p>
    This is step 3 of the <em>User Authentication with IAP</em>
    codelab.
  </p>

</body>
</html>
student_02_214654af9074@cloudshell:~/user-authentication-with-iap/3-HelloVerifiedUser (qwiklabs-gcp-02-c134db2035a8)$ 
```

```sh
student_02_214654af9074@cloudshell:~/user-authentication-with-iap/3-HelloVerifiedUser (qwiklabs-gcp-02-c134db2035a8)$ gcloud app deploy
Services to deploy:

descriptor:                  [/home/student_02_214654af9074/user-authentication-with-iap/3-HelloVerifiedUser/app.yaml]
source:                      [/home/student_02_214654af9074/user-authentication-with-iap/3-HelloVerifiedUser]
target project:              [qwiklabs-gcp-02-c134db2035a8]
target service:              [default]
target version:              [20240530t095601]
target url:                  [https://qwiklabs-gcp-02-c134db2035a8.uk.r.appspot.com]
target service account:      [qwiklabs-gcp-02-c134db2035a8@appspot.gserviceaccount.com]


Do you want to continue (Y/n)?  y

Beginning deployment of service [default]...
Created .gcloudignore file. See `gcloud topic gcloudignore` for details.
Uploading 4 files to Google Cloud Storage
25%
50%
75%
100%
100%
File upload done.
Updating service [default]...done.                                                                                                                                                 
Setting traffic split for service [default]...done.                                                                                                                                
Deployed service [default] to [https://qwiklabs-gcp-02-c134db2035a8.uk.r.appspot.com]

You can stream logs from the command line by running:
  $ gcloud app logs tail -s default

To view your application in the web browser run:
  $ gcloud app browse
student_02_214654af9074@cloudshell:~/user-authentication-with-iap/3-HelloVerifiedUser (qwiklabs-gcp-02-c134db2035a8)$ 
```

This folder contains the same set of files as seen in `2-HelloUser`, with two files altered and one new file. The new file is `auth.py`, which provides a `user()` method to retrieve and verify the cryptographically signed identity information. The changed files are `main.py` and `templates/index.html`, which now use the results of that method. The unverified headers as  found in the last deployment are also shown for comparison.

```py
student_02_214654af9074@cloudshell:~/user-authentication-with-iap/3-HelloVerifiedUser (qwiklabs-gcp-02-c134db2035a8)$ cat auth.py 
# Copyright 2019 Google LLC
#
# Licensed under the Apache License, Version 2.0 (the "License");
# you may not use this file except in compliance with the License.
# You may obtain a copy of the License at
#
#     http://www.apache.org/licenses/LICENSE-2.0
#
# Unless required by applicable law or agreed to in writing, software
# distributed under the License is distributed on an "AS IS" BASIS,
# WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
# See the License for the specific language governing permissions and
# limitations under the License.

from flask import request
from jose import jwt
import os
import requests

KEYS = None     # Cached public keys for verification
AUDIENCE = None # Cached value requiring information from metadata server


# Google publishes the public keys needed to verify a JWT. Save them in KEYS.
def keys():
    global KEYS

    if KEYS is None:
        resp = requests.get('https://www.gstatic.com/iap/verify/public_key')
        KEYS = resp.json()

    return KEYS


# Returns the JWT "audience" that should be in the assertion
def audience():
    global AUDIENCE

    if AUDIENCE is None:
        project_id = os.getenv('GOOGLE_CLOUD_PROJECT', None)

        endpoint = 'http://metadata.google.internal'
        path = '/computeMetadata/v1/project/numeric-project-id'
        response = requests.get(
            '{}/{}'.format(endpoint, path),
            headers = {'Metadata-Flavor': 'Google'}
        )
        project_number = response.json()

        AUDIENCE = '/projects/{}/apps/{}'.format(project_number, project_id)

    return AUDIENCE


# Return the authenticated user's email address and persistent user ID if
# available from Cloud Identity Aware Proxy (IAP). If IAP is not active,
# returns None.
#
# Raises an exception if IAP header exists, but JWT token is invalid, which
# would indicates bypass of IAP or inability to fetch KEYS.
def user():
    # Requests coming through IAP have special headers
    assertion = request.headers.get('X-Goog-IAP-JWT-Assertion')
    if assertion is None:   # Request did not come through IAP
        return None, None

    info = jwt.decode(
        assertion,
        keys(),
        algorithms=['ES256'],
        audience=audience()
    )

    return info['email'], info['sub']
student_02_214654af9074@cloudshell:~/user-authentication-with-iap/3-HelloVerifiedUser (qwiklabs-gcp-02-c134db2035a8)$ 
```

The `assertion` is the cryptographically signed data  provided in the specified request header. The code uses a library to  validate and decode that data. Validation uses the public keys that  Google provides for checking data it signs, and knowing the audience  that the data was prepared for (essentially, the Google Cloud project  that is being protected). Helper functions `keys()` and `audience()` gather and return those values.

The signed object has two pieces of data we need: the verified email address, and the unique ID value (provided in the `sub`, for subscriber, standard field).

This completes Step 3.

### Test  the Cryptographic Verification

When the deployment is ready you will see a message that you can view your application with `gcloud app browse`.

```sh
student_02_214654af9074@cloudshell:~/user-authentication-with-iap/3-HelloVerifiedUser (qwiklabs-gcp-02-c134db2035a8)$ gcloud app browse
Did not detect your browser. Go to this link to view your app:
https://qwiklabs-gcp-02-c134db2035a8.uk.r.appspot.com
student_02_214654af9074@cloudshell:~/user-authentication-with-iap/3-HelloVerifiedUser (qwiklabs-gcp-02-c134db2035a8)$ 
```

```html
Hello Verified User

Hello, None! Your persistent ID appears to be None.

Your verified email is None, and your verified persistent ID is None.

This is step 3 of the User Authentication with IAP codelab. 
```

Recall that you previously disabled IAP, so the application provides no  IAP data. You should see a page similar to the above.

Since IAP is disabled, no user information is available. Now turn IAP back on.

1. In the cloud console window, click the **Navigation menu** > **Security** > **Identity-Aware Proxy**.
2. Click the **IAP** toggle switch next to App Engine app to turn IAP on again. Click **TURN ON**.

```html
Hello Verified User

Hello, accounts.google.com:student-02-214654af9074@qwiklabs.net! Your persistent ID appears to be accounts.google.com:116601044709378976788.

Your verified email is student-02-214654af9074@qwiklabs.net, and your verified persistent ID is accounts.google.com:116601044709378976788.

This is step 3 of the User Authentication with IAP codelab. 
```

Notice that the email address provided by the verified method does not have the `accounts.google.com:` prefix.

If IAP is turned off or bypassed, the verified data would either be  missing, or invalid, since it cannot have a valid signature unless it  was created by the holder of Google's private keys.



So far, you deployed an App Engine web application. First, you restricted access to the application to only users you chose. Then you retrieved and  displayed the identity of users that IAP allowed access to your  application, and saw how that information might be spoofed if IAP were  disabled or bypassed. Lastly, you verified cryptographically signed  assertions of the user's identity, which cannot be spoofed.
