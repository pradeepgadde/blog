---

layout: single
title:  "Dataprep: Qwik Start"
categories: Cloud
tags: GCP
classes: wide
show_date: true
header:
  overlay_image: /assets/images/gcp-banner-2.png
  og_image: /assets/images/gcp-banner-2.png
  teaser: /assets/images/pca-gcp.png
author:
  name     : "Professional Cloud Architect"
  avatar   : "/assets/images/pca-gcp.png"

sidebar:
  - title: "Topics"
    nav: my-sidebar

---
# Dataprep: Qwik Start

[Cloud Dataprep by Trifacta](https://cloud.google.com/dataprep/) is an intelligent data service for visually exploring, cleaning, and  preparing data for analysis. Cloud Dataprep is serverless and works at  any scale. There is no infrastructure to deploy or manage. Easy data  preparation with clicks and no code!

In this lab, you use Dataprep to manipulate a dataset. You import  datasets, correct mismatched data, transform data, and join data. If  this is new to you, you'll know what it all is by the end of this lab.

- Import data
- Correct mismatched data
- Transform data
- Join data

## Create a Cloud Storage bucket in your project

1. In the Cloud Console, select **Navigation menu**(![Navigation menu icon](https://cdn.qwiklabs.com/tkgw1TDgj4Q%2BYKQUW4jUFd0O5OEKlUMBRYbhlCrF0WY%3D)) > **Cloud Storage** > **Buckets**.
2. Click **Create bucket**.
3. In the **Create a bucket** dialog, **Name** the bucket a unique name. Leave other settings at their default value.
4. Uncheck **Enforce public access prevention on this bucket** for `Choose how to control access to objects`.
5. Click **Create**.

You created your bucket. Remember the bucket name for later steps.



## Initialize Cloud Dataprep

1. Open **Cloud Shell** and run the following command:

```sh
Welcome to Cloud Shell! Type "help" to get started.
To set your Cloud Platform project in this session use “gcloud config set project [PROJECT_ID]”
student_01_932d053b64d1@cloudshell:~$ gcloud config set project         qwiklabs-gcp-00-8ff6ea4c38ae 
Updated property [core/project].
student_01_932d053b64d1@cloudshell:~ (qwiklabs-gcp-00-8ff6ea4c38ae)$ gcloud beta services identity create --service=dataprep.googleapis.com
Service identity created: service-239293164562@trifacta-gcloud-prod.iam.gserviceaccount.com
student_01_932d053b64d1@cloudshell:~ (qwiklabs-gcp-00-8ff6ea4c38ae)$ 
```



1. You should see a message saying the service identity was created.

   1. Select **Navigation menu** > **Dataprep**.
   2. Check to accept the Google Dataprep Terms of Service, then click **Accept**.
   3. Check to authorize sharing your account information with Trifacta, then click **Agree and Continue**.
   4. Click **Allow** to allow Trifacta to access project data.
   5. Click your student username to sign in to Cloud Dataprep by Trifacta. Your username is the  **Username** in the left panel in your lab.
   6. Click **Allow** to grant Cloud Dataprep access to your Google Cloud lab account.
   7. Check to agree to Trifacta Terms of Service, and then click **Accept**.
   8. Click **Continue** on the **First time setup** screen to create the default storage location.

Dataprep opens.

## Create a flow

   Cloud Dataprep uses a `flow` workspace to access and manipulate datasets.

   1. Click **Flows** icon, then the **Create** button, then select **Blank Flow** :
   2. Click on **Untitled Flow**, then name and describe the flow. Since this lab uses 2016 data from the  [United States Federal Elections Commission 2016](https://www.fec.gov/data/browse-data/?tab=bulk-data), name the flow "FEC-2016", and then describe the flow as "United States Federal Elections Commission 2016".
   3. Click **OK**.

   The FEC-2016 flow page opens.

## Import datasets

In this section you import and add data to the FEC-2016 flow.

1. Click **Add Datasets**, then select the **Import Datasets** link.
2. In the left menu pane, select **Cloud Storage** to import datasets from Cloud Storage, then click on the pencil to edit the file path.

1. Type `gs://spls/gsp105` in the **Choose a file or folder** text box, then click **Go**.

You may have to widen the browser window to see the **Go** and **Cancel** buttons.

1. Click **us-fec/**.
2. Click the **+** icon next to `cn-2016.txt` to create a dataset shown in the right pane. Click on the title in the  dataset in the right pane and rename it "Candidate Master 2016".
3. In the same way add the `itcont-2016-orig.txt` dataset, and rename it "Campaign Contributions 2016".
4. Both datasets are listed in the right pane; click **Import & Add to Flow**.

## Prep the candidate file

1. By default, the Candidate Master 2016 dataset is selected. In the right pane, click **Edit Recipe**.

2. The Transformer page is where you build your transformation recipe  and see the results applied to the sample. When you are satisfied with  what you see, execute the job against your dataset.

1. Each of the column heads have a Name and value that specify the data type. To see data types, click the column icon:
2. Notice also that when you click the name of the column, a **Details** panel opens on the right.
3. Click **X** in the top right of the Details panel to close the **Details** panel.

In the following steps you explore data in the grid view and apply transformation steps to your recipe.

1. Column5 provides data from 1990-2064. Widen column5 (like you would  on a spreadsheet) to separate each year. Click to select the tallest  bin, which represents the year 2016.

2. This creates a step where these values are selected.

1. In the **Suggestions** panel on the right, in the **Keep rows** section, click **Add** to add this step to your recipe.

2. The Recipe panel on the right now has the following step:

```
            Keep rows where(DATE(2016, 1, 1) <= column5) && (column5 < DATE(2018, 1, 1))
```

1. In Column6 (State), hover over and click on the mismatched (red) portion of the header to select the mismatched rows.

2. Scroll down to the bottom (highlighted in red) find the mismatched  values and notice how most of these records have the value "P" in  column7, and "US" in column6. The mismatch occurs because column6 is  marked as a "State" column (indicated by the flag icon), but there are  non-state (such as "US") values.

1. To correct the mismatch, click **X** in the top of the  Suggestions panel to cancel the transformation, then click on the flag  icon in Column6 and change it to a "String" column.

2. There is no longer a mismatch and the column marker is now green.

1. Filter on just the presidential candidates, which are those records  that have the value "P" in column7. In the histogram for column7, hover  over the two bins to see which is "H" and which is "P". Click the "P"  bin.

2. In the right Suggestions panel, click **Add** to accept the step to the recipe.

## Wrangle the Contributions file and join it to the Candidates file

On the Join page, you can add your current dataset to another dataset or recipe based on information that is common to both datasets.

Before you join the Contributions file to the Candidates file, clean up the Contributions file.

1. Click on  **FEC-2016** (the dataset selector) at the top of the grid view page.

2. Click to select the grayed out **Campaign Contributions 2016**.

3. In the right pane, click **Add** > **Recipe**,  then click **Edit Recipe**.

4. Click the **recipe** icon at the top right of the page, then click **Add New Step**.

5. Remove extra delimiters in the dataset.

1. Insert the following Wrangle language command in the Search box:

2. The Transformation Builder parses the Wrangle command and populates the Find and Replace transformation fields.

3. Click **Add** to add the transform to the recipe.

4. Add another new step to the recipe. Click **New Step**, then type "Join" in the Search box.

5. Click **Join datasets** to open the Joins page.

6. Click on "Candidate Master 2016" to join with Campaign Contributions 2016, then **Accept** in the bottom right.

7. On the right side, hover in the Join keys section, then click on the pencil (Edit icon).

8. Dataprep infers common keys. There are many common values that Dataprep suggests as Join Keys.

1. In the Add Key panel, in the Suggested join keys section, click **column2 = column11**.
2. Click **Save and Continue**.

Columns 2 and 11 open for your review.

1. Click **Next**, then check the checkbox to the left of the "Column" label to add all columns of both datasets to the joined dataset.

1. Click **Review**, and then **Add to Recipe** to return to the grid view.

## Summary of data

Generate a useful summary by aggregating, averaging, and counting the contributions in Column 16 and grouping the candidates by IDs, names,  and party affiliation in Columns 2, 24, 8 respectively.

At the top of the Recipe panel on the right, click on New Step and enter the following formula in the Transformation search box to preview the aggregated data.

An initial sample of the joined and aggregated data is displayed, representing a summary table of US presidential candidates and their 2016 campaign contribution metrics.

Click Add to open a summary table of major US presidential candidates and their 2016 campaign contribution metrics.


## Rename columns

You can make the data easier to interpret by renaming the columns.

Add each of the renaming and rounding steps individually to the recipe by clicking New Step, then enter:

1. Then click **Add**.
2. Add in this last **New Step** to round the Average Contribution amount:
3. Then click **Add**.

You used Dataprep to add a dataset and created recipes to wrangle the data into meaningful results.
