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

## Configure your CI/CD pipeline with Cloud Build and Google Cloud Deploy

In the previous task, you took code from a Git repository, used Docker to build an image, and pushed it to Container Registry. Cloud Build allows you to create workflows of build steps similar to the kind you executed manually. Google and the Cloud Build community have created several builders, each usable by name.

In this task, you create a `cloudbuild.yaml` file to  define the build workflow: build the image, push it to Container  Registry, and finally push it to Google Cloud Deploy. From there, Google Cloud Deploy manages continous delivery. You then test for correctness.

![Cloud Build workflow diagram](https://cdn.qwiklabs.com/J1TMRA1wySpsd9wdKzpoWc%2Bw6beVk0o7a9s8hoDq1mM%3D)

Create a `cloudbuild.yaml` file, which executes the same steps you just performed manually:

The file name `cloudbuild.yaml` is the expected default, but you can give it a custom name. At its most basic, the `cloudbuild.yaml` file lays out the steps the Cloud Build worker should complete. The file can then be manually passed to Cloud Build or automated as part of a trigger.

1. Create a `clouddeploy.yaml` file to specify the delivery pipeline containing the target platforms where the `helloworld` file should be delivered; in this case, the three existing GKE clusters: dev-cluster, staging-cluster, and prod-cluster:

2. Create the `scaffold.yaml` file to specify the Kubernetes YAML files to be deployed by Google Cloud Deploy:

3. Create the Kubernetes files to be deployed to the clusters:

4. Deploy the `clouddeploy.yaml` file to Google Cloud Deploy to configure the release pipeline:

5. In the Google Cloud Console, navigate to Google Cloud Deploy and confirm that the pipeline was created.

   Click on **helloworld-pipeline** to open it and view the pipeline.

   Use Cloud Build to execute the workflow:

What's the error?

The problem is that the service account that Cloud Build uses doesn't by default have any access to Google Cloud Deploy.

In the Google Cloud Console, navigate to **IAM & admin > IAM**. Enable the option **Include Google-provided role grants** and locate the service account with the suffix *@cloudbuild.gserviceaccount.com*.

Click **Edit** (image), and add the **Cloud Deploy > Cloud Deploy Releaser** and **Service Accounts > Service Account User** roles.

Return to Cloud Shell and retry the Cloud Build command:

In the Google Cloud Console, navigate to **Cloud Deploy**, open the pipeline you created earlier, and wait for the release to be deployed to dev-cluster.

In the Google Cloud Console, navigate to **Kubernetes Engine > Services & Ingress**, locate the hello-world-demo service, and click on the link with the public IP address to see the deployed application running on dev-cluster.



## Automate continuous integration using a Cloud Build trigger

For this task, you use a very simplified development process. You commit changes to the Git repository local to Cloud Shell and push the changes to the remote repository. When changes are pushed to Cloud Source Reposies master branch, a Cloud Build trigger based on the cloudbuild.yaml file created earlier is excuted, which builds the image, pushes it to Container Registry, and schedules a Google Cloud Deploy release.

1. In Cloud Shell, initialize and configure Git for use:

2. Stage the code for commit in the local Cloud Shell Git repository, and make the initial commit:

3. Create a new *helloworld* Git repository in your Cloud repository:

4. Git allows for the creation of remote team repositories for sharing code between developers.

   In Cloud Shell, create a remote link to the Git repository created in the previous step, and name it `to-google`:

5. Push the code from your local Cloud Shell repository to the Google repository helloworld master branch:

6. In the Google Cloud Console, navigate to Cloud Source Repositories and investigate your new repo.

The code is written locally and stored in a central repository, the build steps are defined in a `cloudbuild.yaml` file, and the deployment steps are specified in the `clouddeploy.yaml` file. The last step is to create a trigger to put it all together.

n the Google Cloud Console, navigate to **Cloud Build > Triggers**.

Click on the **CREATE TRIGGER** button.

Enter the name `HelloWorldJS`.

Under **Source**, enter select the repository to be `helloworld (Cloud Source Repositories)` and the branch to match `^master$`.

Under **Configuration**, select `Cloud Build configuration file`, and enter `/cloudbuild.yaml` as the Cloud Build configuration file location.

Click on **CREATE** to create the trigger.

When the trigger is ready, to test the trigger's logic, click **RUN**. If a window pops up to select a branch, enter `master` and press **RUN TRIGGER**.

Navigate to **Cloud Build > History** and wait for the build workflow to complete.

Navigate to Google Cloud Deploy and confirm that the new pipeline has been triggered.

### Test the process

1. Return to Cloud Shell and edit the services code:

2. Make a change to the services returned message; for example, change `World` to `World 2`.

   Stage and commit the change to your local developers Git repository, and then push the change to the repository:

   Return to the Google Cloud Console and navigate to **Cloud Build > History**.

   When the trigger is finished executing, go to Google Cloud Deploy and wait for the deployment to occur.

   Re-load the application and confirm that the new message is displayed.

## Promote and approve releases

You have been working out of dev-cluster, which is great for developing and testing new features of your application. However, after those features have been developed, you might want to expose them to an internal audience in a staging environment for further testing and eventually deploy them to a production environment.

In this task, you learn how to promote your application to other environments and use approvals for stakeholders to approve a release.

1. In the Google Cloud Console, navigate to **Cloud Deploy** and open your pipeline.
2. In the pipeline visualization area, click on the 3-dot menu in the dev target box. Notice the following actions:
   - Promote release: Launches your application to the next target; in this case, staging-cluster.
   - Re-depoy release: If an error occurs, you can try to redeploy the current version to dev-cluster to fix any transient issues.
   - Roll back release: Discard the latest release and bring the previous version of your application to the target cluster.
3. To deploy your application to staging, click **Promote release**. Investigate the options in the pop-up window, and check the **Manifest Diff** tab to check the differences with the previous version.
4. Click **Promote**.
5. After the application is deployed, to see the deployed application running on staging-cluster, navigate to **Kubernetes Engine > Services & Ingress**.
6. Locate the hello-world-demo service and click on the link with the public IP address.
7. To promote the release from staging to production, return to **Cloud Deploy**, open your pipeline, click **Promote** in the staging box, and then in the dialog, click **Promote** again. Notice that this time the promotion has been blocked by a review action that you specified earlier in the `clouddeploy.yaml`.
8. In the visualization area between staging and production, click **Review**.
9. After reviewing the changes, click **Approve**. Usually, approvals would be done by a different person or team.
10. Return to your Cloud Deploy pipeline visualization, and click on the release that is deployed to prod-cluster.
11. Click again on the rollout to open a details window and open the **Render logs** and **Deployment logs**. Here you can debug if there are any issues in the deployment. Notice that the actual deployment also runs on a Cloud Build runner.
12. After the rollout has finished, to see the deployed application running on prod-cluster, navigate to **Kubernetes Engine > Services & Ingress**, locate the hello-world-demo service, and click on the link with the public IP address.



You have used Cloud Source Repositories, Cloud Build, and Google Cloud  Deploy to automate the application delivery to GKE and Anthos clusters.  You also learned how to promote and approve releases  to staging and  production clusters.
