---

layout: single
title:  "Develop an app with Gemini"
categories: Cloud
tags: GCP
classes: wide
show_date: true
header:
  overlay_image: /assets/images/Google_Gemini_logo.png
  og_image: /assets/images/Google_Gemini_logo.png
  teaser: /assets/images/gemini.webp
author:
  name     : "Gemini"
  avatar   : "/assets/images/gemini.webp"

sidebar:
  - title: "Topics"
    nav: my-sidebar

---

# Develop an app with Gemini

- Create a cloud-based application development environment by using Cloud Workstations.
- Explore various Google services that you can use to deploy an app by asking Gemini context-based questions.
- Prompt Gemini to provide templates that you can use to develop a basic app in Cloud Run.
- Create, explore, and modify the app by using Gemini to explain and generate the code.
- Run and test the app locally, and then deploy it to Google Cloud by using Gemini to generate the steps.

## Configure your environment and account

```sh
Welcome to Cloud Shell! Type "help" to get started.
Your Cloud Platform project in this session is set to qwiklabs-gcp-03-286f27ed8fa5.
Use “gcloud config set project [PROJECT_ID]” to change to a different project.
student_03_f481970cb176@cloudshell:~ (qwiklabs-gcp-03-286f27ed8fa5)$ PROJECT_ID=$(gcloud config get-value project)
REGION=us-west1
echo "PROJECT_ID=${PROJECT_ID}"
echo "REGION=${REGION}"
Your active configuration is: [cloudshell-1322]
PROJECT_ID=qwiklabs-gcp-03-286f27ed8fa5
REGION=us-west1
student_03_f481970cb176@cloudshell:~ (qwiklabs-gcp-03-286f27ed8fa5)$ USER=$(gcloud config get-value account 2> /dev/null)
echo "USER=${USER}"
USER=student-03-f481970cb176@qwiklabs.net
student_03_f481970cb176@cloudshell:~ (qwiklabs-gcp-03-286f27ed8fa5)$ gcloud services enable cloudaicompanion.googleapis.com --project ${PROJECT_ID}
Operation "operations/acat.p2-153673707911-b724c71c-12df-4ebf-965b-c70dd74ab42f" finished successfully.
student_03_f481970cb176@cloudshell:~ (qwiklabs-gcp-03-286f27ed8fa5)$ gcloud projects add-iam-policy-binding ${PROJECT_ID} --member user:${USER} --role=roles/cloudaicompanion.user
gcloud projects add-iam-policy-binding ${PROJECT_ID} --member user:${USER} --role=roles/serviceusage.serviceUsageViewer
Updated IAM policy for project [qwiklabs-gcp-03-286f27ed8fa5].
bindings:
- members:
  - serviceAccount:qwiklabs-gcp-03-286f27ed8fa5@qwiklabs-gcp-03-286f27ed8fa5.iam.gserviceaccount.com
  role: roles/bigquery.admin
- members:
  - user:student-03-f481970cb176@qwiklabs.net
  role: roles/cloudaicompanion.user
- members:
  - serviceAccount:153673707911@cloudbuild.gserviceaccount.com
  role: roles/cloudbuild.builds.builder
- members:
  - serviceAccount:service-153673707911@gcp-sa-cloudbuild.iam.gserviceaccount.com
  role: roles/cloudbuild.serviceAgent
- members:
  - serviceAccount:service-153673707911@compute-system.iam.gserviceaccount.com
  role: roles/compute.serviceAgent
- members:
  - serviceAccount:service-153673707911@container-engine-robot.iam.gserviceaccount.com
  role: roles/container.serviceAgent
- members:
  - serviceAccount:153673707911-compute@developer.gserviceaccount.com
  - serviceAccount:153673707911@cloudservices.gserviceaccount.com
  role: roles/editor
- members:
  - serviceAccount:service-153673707911@gcp-sa-notebooks.iam.gserviceaccount.com
  role: roles/notebooks.serviceAgent
- members:
  - serviceAccount:admiral@qwiklabs-services-prod.iam.gserviceaccount.com
  - serviceAccount:qwiklabs-gcp-03-286f27ed8fa5@qwiklabs-gcp-03-286f27ed8fa5.iam.gserviceaccount.com
  - user:student-03-f481970cb176@qwiklabs.net
  role: roles/owner
- members:
  - serviceAccount:qwiklabs-gcp-03-286f27ed8fa5@qwiklabs-gcp-03-286f27ed8fa5.iam.gserviceaccount.com
  role: roles/storage.admin
- members:
  - user:student-03-f481970cb176@qwiklabs.net
  role: roles/viewer
- members:
  - serviceAccount:service-153673707911@gcp-sa-websecurityscanner.iam.gserviceaccount.com
  role: roles/websecurityscanner.serviceAgent
- members:
  - serviceAccount:service-153673707911@gcp-sa-workstations.iam.gserviceaccount.com
  role: roles/workstations.serviceAgent
etag: BwYg2qeUbpQ=
version: 1
Updated IAM policy for project [qwiklabs-gcp-03-286f27ed8fa5].
bindings:
- members:
  - serviceAccount:qwiklabs-gcp-03-286f27ed8fa5@qwiklabs-gcp-03-286f27ed8fa5.iam.gserviceaccount.com
  role: roles/bigquery.admin
- members:
  - user:student-03-f481970cb176@qwiklabs.net
  role: roles/cloudaicompanion.user
- members:
  - serviceAccount:153673707911@cloudbuild.gserviceaccount.com
  role: roles/cloudbuild.builds.builder
- members:
  - serviceAccount:service-153673707911@gcp-sa-cloudbuild.iam.gserviceaccount.com
  role: roles/cloudbuild.serviceAgent
- members:
  - serviceAccount:service-153673707911@compute-system.iam.gserviceaccount.com
  role: roles/compute.serviceAgent
- members:
  - serviceAccount:service-153673707911@container-engine-robot.iam.gserviceaccount.com
  role: roles/container.serviceAgent
- members:
  - serviceAccount:153673707911-compute@developer.gserviceaccount.com
  - serviceAccount:153673707911@cloudservices.gserviceaccount.com
  role: roles/editor
- members:
  - serviceAccount:service-153673707911@gcp-sa-notebooks.iam.gserviceaccount.com
  role: roles/notebooks.serviceAgent
- members:
  - serviceAccount:admiral@qwiklabs-services-prod.iam.gserviceaccount.com
  - serviceAccount:qwiklabs-gcp-03-286f27ed8fa5@qwiklabs-gcp-03-286f27ed8fa5.iam.gserviceaccount.com
  - user:student-03-f481970cb176@qwiklabs.net
  role: roles/owner
- members:
  - user:student-03-f481970cb176@qwiklabs.net
  role: roles/serviceusage.serviceUsageViewer
- members:
  - serviceAccount:qwiklabs-gcp-03-286f27ed8fa5@qwiklabs-gcp-03-286f27ed8fa5.iam.gserviceaccount.com
  role: roles/storage.admin
- members:
  - user:student-03-f481970cb176@qwiklabs.net
  role: roles/viewer
- members:
  - serviceAccount:service-153673707911@gcp-sa-websecurityscanner.iam.gserviceaccount.com
  role: roles/websecurityscanner.serviceAgent
- members:
  - serviceAccount:service-153673707911@gcp-sa-workstations.iam.gserviceaccount.com
  role: roles/workstations.serviceAgent
etag: BwYg2qe2-UM=
version: 1
student_03_f481970cb176@cloudshell:~ (qwiklabs-gcp-03-286f27ed8fa5)$ 
```

## Create a Cloud Workstation

Cloud Workstations is a fully  managed integrated development environment that includes native  integration with Gemini.

In this task, you configure and provision your Cloud Workstation environment, and you enable the Cloud Code plugin for Gemini.

In the Google Cloud console, select the  **Navigation menu** (![Navigation menu icon](https://cdn.qwiklabs.com/tkgw1TDgj4Q%2BYKQUW4jUFd0O5OEKlUMBRYbhlCrF0WY%3D)), and then select **View All Products > Tools > Cloud Workstations**.

Workstation clusters manage workstations during their lifecycle and  provide a configurable gateway for network connectivity. Workstation  cluster creation can take up to 20 minutes, but is only needed once  within a region.

##  Workstation configurations 

Workstation configurations act as templates for workstations. Create a workstation  configuration first, and then create workstations. Add creators to a  configuration to allow them to create their own workstation. 

Before creating a workstation, you must create a workstation configuration in Cloud Workstations.

In the **Navigation pane**, click **Workstation configurations**, and then click **Create Workstation Configuration**.

Check the **Status** of the configuration being created. If the status of the configuration is `Reconciling` or `Updating`, periodically refresh and wait until the status becomes `Ready` before moving to the next step.

### Create a workstation

In the **Navigation pane**, click **Workstations**, and then click **Create Workstation**.

Click **Create**.

After the workstation is created, it is listed under **My workstations** with a status of `Stopped`.

To start the workstation, click **Start**.

As the workstation starts up, the status changes to `Starting`. Wait for the status to change to `Running` which indicates that it is ready to be used. It might take several minutes for the workstation to fully start up.

### Launch the IDE

To function properly, some extensions need third-party cookies to be enabled in your browser.

1. To enable third-party cookies in Chrome, in the **Chrome** menu, click **Settings**.
2. In the search bar, type **Third-party cookies**.
3. Click the **Third-party cookies** setting, and select **Allow third-party cookies**.

## Update the Cloud Code extension to enable Gemini

In this task, you enable Gemini in Cloud Code for your Workstation IDE.

### Connect to Google Cloud

To connect to Google Cloud in your workstation, perform these steps:

1. At the bottom of the window, on the status bar, click **Cloud Code - Sign In**.

2. When you're prompted to sign in, click **Proceed to sign in**.

   A link is displayed in the terminal.

3. To launch the Cloud Cloud sign-in flow, press Control (for Windows  and Linux) or Command (for MacOS) and click the link in the terminal.

4. If you are asked to confirm the opening of the external website, click **Open**.

5. Click the student email address.

6. When you're prompted to continue, click **Continue**.

7. To let the Google Cloud SDK access your Google Account and agree to the terms, click **Allow**.

   Your verification code is displayed in the browser tab.

   **Note:** You may see a warning that you ran a gcloud auth login command. This process is normal. The IDE ran this command on your behalf. 

8. Click **Copy**.

9. Back in the IDE, in the terminal, where it says **Enter authorization code**, paste the code.

10. If asked to approve copying from the clipboard, click **Allow**.

11. Click Enter, and then wait for the status bar to show **Cloud Code - No Project**.

    You're now connected to Google Cloud.

### Enable Gemini in Cloud Code

To enable Gemini in Cloud Code for your workstation IDE, perform these steps:

1. In your workstation IDE, click the menu (![Code OSS main menu](https://cdn.qwiklabs.com/H8AIg8zPKj8LfgE5rJjMSqNVieIjiopITlMQfBWkz0I%3D)), and then navigate to **File > Preferences > Settings**.

2. On the **User** tab of the Settings dialog, select **Extensions > Gemini Code Assist + Google Cloud Code**.

3. In **Search settings**, enter `Gemini`.

4. On the Qwiklabs lab credentials panel, to copy the Project ID, click **Copy**.

5. On the Cloud Code settings page, for **Cloudcode > Duet AI: Project**, paste the Google Cloud project ID.

   **Note:** This setting may be specified as **Cloudcode > Gemini: Project**. 

   Confirm that **Cloud Code > Duet AI: Enable** is enabled.

   **Note:** This setting may be specified as **Cloudcode > Gemini: Enable**. 

   In the IDE status bar, click **Cloud Code - No Project**.

   Click **Select a Google Cloud Project**, and then click your project ID.

   The project ID is now shown in the status bar. Gemini is now ready to use.

## Chat with Gemini

Gemini can help you choose the Google Cloud services that meet the  requirements of your application architecture. If you want to develop  and test your app in your local IDE, and then deploy it to Google Cloud, you can chat with Gemini to get help.

In this task, you use the **Gemini Code Assist pane** to enter prompts and view the responses from Gemini.

Prompts are questions or statements that describe the help that you  need. Prompts can include context from existing code that Google Cloud  analyzes to provide more useful or complete responses. For more  information on writing prompts to generate good responses, see [Write better prompts for Gemini in Google Cloud](https://cloud.google.com/gemini/docs/discover/write-prompts).



### Prompt Gemini

To prompt Gemini about Google Cloud services, perform these steps:

1. To open the Gemini chat pane, in the IDE activity bar, click **Gemini Code Assist** (![Code OSS Gemini menu](https://cdn.qwiklabs.com/8enFLY%2FtuyohEbT5f1NbBaC%2Fa%2Be7cuSzbbq1l19r%2B3Q%3D)).
2. If an error occurs when trying to open the Gemini chat pane, refresh the browser window.
3. In the **Gemini Code Assist** pane, type the following prompt, and then click **Send** (![Gemini send](https://cdn.qwiklabs.com/%2FmzrTT7U8dkGsp7pSuOUCbVh%2B9qgdh64vfB7FtF8io4%3D)):

```sh
I am new to Google Cloud and I want to use the Cloud Code extension. Give me some examples of Google services that I can use to build and deploy a sample app.
```

```sh
Here are some examples of Google Cloud services you can use with Cloud Code to build and deploy a sample app, along with their relevance to a beginner:

Beginner-Friendly:

    Cloud Run: A great starting point. It's a fully managed serverless platform, so you don't need to worry about infrastructure. Deploy containerized apps quickly.
        Example: Deploy a simple "Hello World" web app written in Python (Flask), Node.js (Express), or Go.
    App Engine: Another good option for beginners. It's a platform-as-a-service (PaaS) that simplifies deployment and scaling.
        Example: Deploy a basic web app with a form that stores data in Cloud Firestore (a NoSQL database).

Slightly More Advanced (but still manageable):

    Cloud Functions: Run small pieces of code (functions) in response to events without managing servers.
        Example: Create a function triggered by new file uploads to Cloud Storage, resizing the image automatically.
    Cloud SQL: If you prefer a relational database, Cloud SQL offers managed MySQL, PostgreSQL, and SQL Server instances.
        Example: Build a simple blog app using Cloud SQL as the database and a framework like Django (Python) or Spring Boot (Java).

How Cloud Code Helps:

    Simplified Development: Cloud Code provides tools to create, run, debug, and deploy your apps directly from your IDE (like VS Code or IntelliJ).
    Streamlined Deployment: It automates the process of building container images (if needed) and deploying them to your chosen service.
    Local Emulation: Cloud Code lets you run your app locally in a simulated Cloud Run or Cloud Functions environment for testing.

Getting Started Resources:

    Cloud Code Documentation: https://cloud.google.com/code
    Cloud Run Quickstarts: https://cloud.google.com/run/docs/quickstarts
    App Engine Quickstarts: https://cloud.google.com/appengine/docs/standard/quickstart

Remember to start with a simple example and gradually explore more services as you gain experience.
```

Chat history state is kept in memory only, and doesn't  persist when you switch to another workspace or when you close your IDE. Gemini doesn't use your prompts or its responses as data to train its  model. For more information, see [How Gemini for Google Cloud uses your data](https://cloud.google.com/gemini/docs/discover/data-governance). 

## Develop a Python app

Let's now use Cloud Run to create and deploy a basic Python app.  Because you're new to Cloud Run and Cloud Code, you need help with the  steps for creating the app.

In this task, you prompt Gemini for help to build a Hello World Python app in Cloud Run.

### Get help from Gemini

Let's now use Cloud Run to create and deploy a basic Python app.  Because you're new to Cloud Run and Cloud Code, you need help with the  steps for creating the app.

In this task, you prompt Gemini for help to build a Hello World Python app in Cloud Run.

```sh
How do I create a new Cloud Run app in Cloud Code using the command palette? What languages are supported?
```

```sh
To create a new Cloud Run app in Cloud Code using the command palette:

    Open the command palette: Go to View > Command Palette or use the shortcut Ctrl+Shift+P (or Cmd+Shift+P on macOS).
    Start typing "Cloud Code: New Application": This should narrow down the options in the command palette.
    Select "Cloud Code: New Application": This will open a wizard to guide you through the creation process.
    Choose "Cloud Run" as the deployment platform: You'll be presented with a list of Google Cloud services. Select "Cloud Run".
    Select your preferred language: Cloud Code supports several languages for Cloud Run, including:
        Go
        Node.js
        Python
        Java
        .NET (C#)
    Follow the prompts: The wizard will guide you through the rest of the process, including naming your application, choosing a location, and setting up any necessary dependencies.

Once the process is complete, Cloud Code will create a new project with the necessary files and configurations for your chosen language and deploy it to Cloud Run.
Rate this answer:
      
```

### Create a Python app by using the steps from Gemini

1. Click the menu (![Code OSS main menu](https://cdn.qwiklabs.com/H8AIg8zPKj8LfgE5rJjMSqNVieIjiopITlMQfBWkz0I%3D)), and then navigate to **View > Command Palette**.

2. Type `Cloud Code New`, and then select **Cloud Code: New Application**.

3. Select **Cloud Run application**.

4. Select **Python (Flask): Cloud Run**.

5. Update the name of the app and top-level folder to `/home/user/hello-world`, and then click **Ok**.

   Cloud Code downloads the template and creates the application files in the folder in your IDE.

### Explore the app with Gemini

Now that you've created your `Hello World` app in Cloud Run, you can use Gemini to explain the files and code snippets that are deployed in your IDE.

1. If the files are not visible, in the IDE activity bar, click **Explorer** (![Code OSS Explorer menu](https://cdn.qwiklabs.com/O0MFBj4lYUnKniPTeF5CRvcp8WYEfriQ1TKU2JDw%2FgQ%3D)).

2. In the Explorer pane, select **Dockerfile**.

3. Select the entire contents of the Dockerfile, click the bulb (![Code OSS Gemini bulb](https://cdn.qwiklabs.com/tLYH7qWU%2BLK5Z3aeYQf6nH7omJ0LQhao5WaGbYeXKLk%3D)), and from the **More Actions** menu,  click **Gemini: Explain this**.

   Gemini generates a natural-language explanation about the contents and function of the `Dockerfile`. You can also select portions of the file contents, click the bulb (![Code OSS Gemini bulb](https://cdn.qwiklabs.com/tLYH7qWU%2BLK5Z3aeYQf6nH7omJ0LQhao5WaGbYeXKLk%3D)), and then click **Gemini: Explain this**.

4. Select the line that begins with **ENTRYPOINT**, click the bulb (![Code OSS Gemini bulb](https://cdn.qwiklabs.com/tLYH7qWU%2BLK5Z3aeYQf6nH7omJ0LQhao5WaGbYeXKLk%3D)), and then click **Gemini: Explain this**.

   Gemini responds with details about the ENTRYPOINT instruction. You  learn that, with this instruction, Docker will run the app.py file when  the container launches.

5. To view the contents of the `app.py` file, in the activity bar, click **Explorer** (![Code OSS Explorer menu](https://cdn.qwiklabs.com/O0MFBj4lYUnKniPTeF5CRvcp8WYEfriQ1TKU2JDw%2FgQ%3D)), and then click `app.py`.

6. In the `hello()` function definition, select the lines that contain the `K_SERVICE`, and `K_REVISION` environment variables. Click the bulb (![Code OSS Gemini bulb](https://cdn.qwiklabs.com/tLYH7qWU%2BLK5Z3aeYQf6nH7omJ0LQhao5WaGbYeXKLk%3D)), then click **Gemini: Explain this**.

   Gemini responds with a detailed explanation of these two Cloud Run  environment variables and how they are used in the application code.

   ```sh
   This Dockerfile defines how to build a Docker image for a Python application. Here's a breakdown:
   
       FROM python:3.12-alpine: This line specifies the base image for the Docker image. It uses the official Python 3.12 image from Docker Hub, specifically the "alpine" variant which is known for its small size.
   
       WORKDIR /app: This sets the working directory inside the container to /app. Subsequent commands like COPY and RUN will be executed in this directory.
   
       COPY requirements.txt .: This copies the requirements.txt file from your local machine (the directory where you run the docker build command) into the container's /app directory.
   
       RUN pip install --trusted-host pypi.python.org -r requirements.txt: This line installs the Python dependencies listed in the requirements.txt file. The --trusted-host pypi.python.org flag tells pip to trust the official Python Package Index (PyPI) as a source for packages.
   
       COPY . .: This copies all other files and directories from your local machine's current directory into the container's /app directory. This ensures your application code is available within the container.
   
       ENTRYPOINT ["python", "app.py"]: This defines the command that will be executed when a container is started from this image. In this case, it runs the Python script app.py located in the /app directory.
   
   In summary: This Dockerfile sets up a lightweight container with Python 3.12, installs the necessary dependencies, copies your application code, and specifies the command to run your application when the container starts.
   
   ```

   

### Run the app locally

You can run your app locally from your IDE by using the Cloud Run emulator. In this case, *locally* means on the workstation machine.

In the activity bar of your IDE, click **Cloud Code** (![Code OSS Cloud Code menu](https://cdn.qwiklabs.com/h7a1cGNlfuYGKBrCndl%2Fn9cDcLtr2yacmETamGQFgAs%3D)), and then click **Cloud Run**.

In the Cloud Run activity bar, click **Run App on Local Cloud Run Emulator** (![Cloud Run - run on local emulator](https://cdn.qwiklabs.com/z1o4I2oLA6FB2DV9wgz8KLptStQuC0l4PxaS%2Bq%2F3PVs%3D)), and then click **Run**.

The **Output** tab in the IDE displays the progress of the build.

When prompted at the top of the screen to **Enable minikube gcp-auth addon to access Google APIs**, select **Yes**.

```sh
Starting to run the app using configuration 'Cloud Run: Run/Debug Locally' from .vscode/launch.json...
To view more detailed logs, go to Output channel : "Cloud Run: Run/Debug Locally - Detailed"
Dependency check started
Dependency check succeeded
Starting minikube, this may take a while.....................................
minikube successfully started
The minikube profile 'cloud-run-dev-internal' has been scheduled to stop automatically after exiting Cloud Code. To disable this on future deployments, set autoStop to false in your launch configuration /home/user/hello-world/.vscode/launch.json
To access Google APIs inside your minikube cluster, you must enable the minikube gcp-auth addon. More info can be found here: https://minikube.sigs.k8s.io/docs/handbook/addons/gcp-auth/. Enable gcp-auth now?
Input required. Please respond to the prompt at the top of the screen.
Configuring minikube gcp-auth addon
Using GCP project 'qwiklabs-gcp-03-286f27ed8fa5' with minikube gcp-auth


Update initiated
Build started for artifact hello-world
Build completed for artifact hello-world

Deploy started
Status check started
Resource pod/hello-world-6dcb9b468d-5jdh7 status updated to In Progress
Resource deployment/hello-world status updated to In Progress
Resource deployment/hello-world status completed successfully
Status check succeeded

**************URLs*****************
Deploy completed

Forwarded URL from service hello-world: http://localhost:8080
Update succeeded
***********************************
Watching for changes...
To disable watch mode for subsequent runs, set watch to false in your launch configuration /home/user/hello-world/.vscode/launch.json and relaunch the application.

```

## Enhance the Python app

Let's now add data and functionality to the app so that it can be used for management of inventory data.

In this task, you first add inventory data for the app.

### Generate sample data using Gemini

In the activity bar of your IDE, click **Explorer** (![Code OSS Explorer menu](https://cdn.qwiklabs.com/O0MFBj4lYUnKniPTeF5CRvcp8WYEfriQ1TKU2JDw%2FgQ%3D)).

Click **New file** (![Explorer - new file](https://cdn.qwiklabs.com/FeWHYTzVb8DXHcyWlssgCQ5rwEMk%2FOlcYYeDZC2Lnak%3D)), and create a file named `inventory.py`.

To let Gemini generate the sample data, open the **Gemini Code Assist** pane, type the following prompt, and then click **Send**:

```sh
Create a variable called inventory which is a list of 3 JSON objects. Each JSON object has 2 attributes: productid and onhandqty. Both attributes are strings.
```

Gemini generates the `inventory` JSON array that contains 3 JSON objects.

To insert the sample JSON data in the inventory.py file, in the Gemini response, click **Insert in current file** (![Gemini - insert in current file](https://cdn.qwiklabs.com/rqqAHJzWVn36fpBDgUmKEKBIsWqFpgHDHES0QFKLFX4%3D)). The contents of the file is similar to:

```json
inventory = [
    {"productid": "12345", "onhandqty": "10"},
    {"productid": "67890", "onhandqty": "5"},
    {"productid": "13579", "onhandqty": "20"}
]

```

### Add the GET /inventory list API method to the app

You now introduce API methods in the `app.py` file that can operate on the inventory data. To complete this subtask, you use the code generation feature in Gemini.

1. In **Explorer**, open the file `app.py`.
2. Modify the flask import statement to include the `inventory` file and the `jsonify` library:
3. In the `app.py` file, position your cursor below the *app* assignment statement:



To let **Gemini Code Assist** generate the code for the first API method, in the `app.py` file, enter the following comment:

```sh
# Generate an app route to display a list of inventory items in the JSON format from the inventory.py file.
# Use the GET method.
```

Select the comment lines, including the blank line below the comment.

Click the bulb (![Code OSS Gemini bulb](https://cdn.qwiklabs.com/tLYH7qWU%2BLK5Z3aeYQf6nH7omJ0LQhao5WaGbYeXKLk%3D)), and then select **Gemini: Generate code**.

Gemini generates a function for the GET operation that returns a list of items from the `inventory.py` file. The function generally looks similar to this:

```py
@app.route('/inventory', methods=['GET'])
def inventory_list():
    """Return a list of inventory items in JSON format."""
    return jsonify(inventory)
```

To learn more about the `jsonify(inventory)` function, highlight the term and prompt Gemini to explain the code to you. 

his line of code takes the `inventory` variable (which,  based on your provided code, is a list of dictionaries representing  inventory items) and converts it into a JSON (JavaScript Object  Notation) formatted string. This is a common format for sending data  over the internet, particularly in web applications.

Here's a breakdown:

- **`jsonify(inventory)`**: This calls the `jsonify` function from the `flask` library. This function takes Python data structures like lists and dictionaries and converts them into a JSON string.
- **`return`**: This sends the resulting JSON string back as a response to the client that made the request to the `/inventory` route.

In essence, this line of code is responsible for fetching the  inventory data and packaging it in a format suitable for transmission to a web browser or other client application.

To accept the generated code, hold the pointer over any part of the generate code response, then click **Accept**.

### Add the GET /inventory/{productID} method to the app

Let's add another API method to return data about a specific  inventory item, given its product ID. If the product ID is not found,  the API returns the standard HTTP status code of 404.

1. Add a few blank lines following the `/inventory' route.

2. To let Gemini generate the code for this second API method, in the `app.py` file, enter the following comment:

3. Select the 3 comment lines and the blank line that follows the comment, click the bulb (![Code OSS Gemini bulb](https://cdn.qwiklabs.com/tLYH7qWU%2BLK5Z3aeYQf6nH7omJ0LQhao5WaGbYeXKLk%3D)), and then select **Gemini: Generate code**.

   Gemini generates a function for the GET operation that returns the  item from the inventory file whose productID is provided in the request, or the 404 status code if the product does not exist.

```sh
# Generate an App route to get a product from the list of inventory items given the productID.
# Use the GET method.
# If there is an invalid productID, return a 404 error with an error message in the JSON.
```

```py
@app.route('/inventory/<productid>', methods=['GET'])
def get_product(productid):
    for item in inventory:
        if item['productid'] == productid:
            return jsonify(item)
    return jsonify({'error': 'Product not found'}), 404
```

Hold the pointer over any part of the generate code response. To accept the generated code, in the toolbar, click **Accept**.

If the generated code is different from the code shown above, replace the generated code snippet with the snippet displayed above.

Your `app.py` file should now look similar to this:

### Rebuild and redeploy the app locally

You can run your app locally from your IDE using the Cloud Run emulator. In this case, *locally* means on the workstation machine.

1. In the activity bar of your IDE, click **Cloud Code** (![Code OSS Cloud Code menu](https://cdn.qwiklabs.com/h7a1cGNlfuYGKBrCndl%2Fn9cDcLtr2yacmETamGQFgAs%3D)).
2. In the Cloud Run activity bar, click **Run App on Local Cloud Run Emulator** (![Cloud Run - run on local emulator](https://cdn.qwiklabs.com/z1o4I2oLA6FB2DV9wgz8KLptStQuC0l4PxaS%2Bq%2F3PVs%3D)).
3. When prompted at the top of the screen to **Enable minikube gcp-auth addon to access Google APIs**, select **Yes**.

```sh
Starting to run the app using configuration 'Cloud Run: Run/Debug Locally' from .vscode/launch.json...
To view more detailed logs, go to Output channel : "Cloud Run: Run/Debug Locally - Detailed"
Dependency check started
Dependency check succeeded
The minikube profile 'cloud-run-dev-internal' has been scheduled to stop automatically after exiting Cloud Code. To disable this on future deployments, set autoStop to false in your launch configuration /home/user/hello-world/.vscode/launch.json
To access Google APIs inside your minikube cluster, you must enable the minikube gcp-auth addon. More info can be found here: https://minikube.sigs.k8s.io/docs/handbook/addons/gcp-auth/. Enable gcp-auth now?
Input required. Please respond to the prompt at the top of the screen.
Configuring minikube gcp-auth addon
Using GCP project 'qwiklabs-gcp-03-286f27ed8fa5' with minikube gcp-auth



Update initiated
Deploy started
Status check started
Resource pod/hello-world-6fbb87dd77-pr82l status updated to In Progress
Resource deployment/hello-world status completed successfully
Status check succeeded

**************URLs*****************
Deploy completed

Forwarded URL from service hello-world: http://localhost:8081
Update succeeded
***********************************
Watching for changes...
To disable watch mode for subsequent runs, set watch to false in your launch configuration /home/user/hello-world/.vscode/launch.json and relaunch the application.

```

### Test the API methods

1. Follow the steps in the earlier task to run the app locally.

2. After following the link to view the running app in a separate browser tab, append `/inventory` to the URL in this tab, and type Enter.

   The API returns a JSON response that contains the list of products from the `inventory.py` file. The JSON response should resemble this:

   Append `/{productID}` to the URL that ends with `/inventory`, where `{productID}` is a product ID in your inventory.

   For the example above, the end of a valid URL would be `/inventory/12345`.

   Type Enter.

   The API returns a JSON response that contains data about the specific product.

   In the URL, replace the product ID with `XXXXX` and type Enter.

   The URL should now end with `/inventory/XXXXX`.

   XXXXX is not a valid product ID, so the API returns a JSON error response indicating that the product is not found.

## Deploy the app to Cloud Run

You can now deploy the app to Cloud Run on Google Cloud.

1. In the activity bar main menu (![Code OSS main menu](https://cdn.qwiklabs.com/H8AIg8zPKj8LfgE5rJjMSqNVieIjiopITlMQfBWkz0I%3D)), click **View > Command Palette**.

2. In the command palette field, type **Cloud Code Deploy**, and then select **Cloud Code: Deploy to Cloud Run** from the list.

3. To enable the Cloud Run API for your project, click **Enable API**.

4. On the **Service Settings** page, for **Region**, select .

5. Leave the remaining settings as their defaults, and then click **Deploy**.

   Cloud Code builds your image, pushes it to the registry, and deploys your service to Cloud Run. This may take a few minutes.

To view your running service, open the URL that is displayed in the Deploy to Cloud Run dialog.

Test your service by appending the `/inventory`, and `/inventory/{productID}` paths to the URL, and verify the response.

To get the URL for the Cloud Run service inventory page, in Cloud Shell, run the following command:

```sh
student_03_f481970cb176@cloudshell:~ (qwiklabs-gcp-03-286f27ed8fa5)$ export SVC_URL=$(gcloud run services describe hello-world \
  --project qwiklabs-gcp-03-286f27ed8fa5 \
  --region us-west1 \
  --platform managed \
  --format='value(status.url)')
echo ${SVC_URL}/inventory
https://hello-world-ygfra2tc3a-uw.a.run.app/inventory
student_03_f481970cb176@cloudshell:~ (qwiklabs-gcp-03-286f27ed8fa5)$ 
```

https://hello-world-ygfra2tc3a-uw.a.run.app/inventory/12346

```json
{"error":"Product not found"}

```

https://hello-world-ygfra2tc3a-uw.a.run.app/inventory/12345

```json
{"onhandqty":"10","productid":"12345"}
```

