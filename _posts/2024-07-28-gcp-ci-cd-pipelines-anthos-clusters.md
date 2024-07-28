---

layout: single
title:  "Creating CI/CD pipelines for Anthos clusters "
categories: Cloud
tags: GCP
classes: wide
show_date: true
header:
  overlay_image: /assets/images/anthos.png
  og_image: /assets/images/anthos.png
  teaser: /assets/images/pca-gcp.png
author:
  name     : "Professional Cloud Architect"
  avatar   : "/assets/images/pca-gcp.png"

sidebar:
  - title: "Topics"
    nav: my-sidebar

---
# Creating CI/CD pipelines for Anthos clusters

In this exercise, you create a modern serverless continuous integration/continuous delivery (CI/CD) pipeline to automate application delivery to GKE and Anthos clusters.

The pipeline starts with a "hello world" Node.js code stored in Cloud Source Repositories in a private Git repository. Commits that are made to the master branch trigger a Cloud Build workflow, which builds a new container image, pushes it to Container Registry, and triggers Google Cloud Deploy as the CD part.

Google Cloud Deploy delivers the application to the dev cluster, which, after testing, allows you to promote it to a staging and  production cluster. Additionally, you learn to implement approvals to ensure that code only reaches production when the right stakeholders approve the release.

![Lab architecture diagram](https://cdn.qwiklabs.com/7mWXsBQVFSuCxnbU%2BDDoQGFV%2FDzdDAtCpBqpTglle38%3D)

- Create and use a repository in Cloud Source Repositories.
- Set up continous integration with Cloud Build workflows.
- Configure continous delivery pipelines with Google Cloud Deploy.

## Create a basic Node.js application from the Google Cloud Console

1. Activate Cloud Shell.

2. If the tab under the Cloud Shell header does not list your project ID *qwiklabs-gcp-[hex]*, press the down arrow to create a new Cloud Shell tab associated with the correct project.

3. Enable the required Google Cloud APIs:

4. Export the current Qwiklabs project ID into a PROJECT_ID environmental variable:

5. Set the Zone environment variable:

6. Authenticate to the Container Registry:

7. Clone a copy of a hello world example into Cloud Shell, and change into the HelloWorldNodeJs folder that the clone operation creates

8. You can explore the code in the Cloud Shell Editor.

   Use the Docker CLI to build the code into a container and push it to Container Registry:

9. The *docker build* command looks in the specified path (.) for a Dockerfile. If it finds one, it follows the instructions to build a Docker image (in this case adding the specified tag `-t`). *docker push* then uploads a copy of the image to Container Registry.

   In the Google Cloud Console, navigate to Container Registry and confirm that the image was created and uploaded as expected.
