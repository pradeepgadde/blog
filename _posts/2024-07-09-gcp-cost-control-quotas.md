---

layout: single
title:  "Setting Up Cost Control with Quota"
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
# Setting Up Cost Control with Quota

Explore controlling your BigQuery costs by modifying quota:

- Query a public dataset and explore associated costs.
- Modify quota.
- Try to rerun the query after quota has been modified.

### BigQuery pricing

BigQuery offers scalable, flexible pricing options to meet your technical needs and your budget.

With BigQuery, you can incur storage and query costs. In this lab, you explore query costs.

here are two pricing models for query costs in BigQuery:

- On-demand: On-demand pricing is based on the amount of data processed by each query you run. This is the most flexible option.
- Flat-rate: Flat-rate customers purchase dedicated resources for query processing and are not charged for individual queries. This option is  predictable and is best for customers with fixed budgets.

### Open the BigQuery console

In the Google Cloud Console, select **Navigation menu** > **BigQuery**.

The **Welcome to BigQuery in the Cloud Console** message box opens. This message box provides a link to the quickstart guide and the release notes.

Click **Done**.

The BigQuery console opens.

## Query a public dataset in BigQuery

In this lab, you query the `bigquery-public-data:wise_all_sky_data_release` public dataset. Learn more about this dataset from the blog post [Querying the Stars with BigQuery GIS](https://cloud.google.com/blog/products/data-analytics/querying-the-stars-with-bigquery-gis).

In the **Query editor** paste the following query:

```sql
SELECT
    w1mpro_ep,
    mjd,
    load_id,
    frame_id
FROM
    `bigquery-public-data.wise_all_sky_data_release.mep_wise`
ORDER BY
    mjd ASC
LIMIT 500
```

Use the query validator to determine how many bytes of data this will process when you run.

`This query will process 1.36 TB when run. `

Processing large amounts of data without proper cost controls, even  with simple queries like the above, can lead to unanticipated charges on your bill. To manage this, examine how BigQuery pricing works and how  you can setup custom quotas for your teams.

Now **run the query** and see how quickly BigQuery processes that size of data.

## Explore query cost

The first 1 TB of query data processed per month is free.

- Learn more about cost from the [BigQuery pricing guide](https://cloud.google.com/bigquery/pricing).
- For more information on calculating and estimating costs in Google Cloud, use the [Google Cloud Pricing Calculator](https://cloud.google.com/products/calculator/).

## Update BigQuery quota

In this task, you update the BigQuery API quota to restrict the data processed in queries in your project.

In your **Cloud Shell**, run this command to view your current usage quotas with the **BigQuery API**:

```sh
Welcome to Cloud Shell! Type "help" to get started.
Your Cloud Platform project in this session is set to qwiklabs-gcp-03-85cb4606093c.
Use “gcloud config set project [PROJECT_ID]” to change to a different project.
student_01_e488ef31cb8a@cloudshell:~ (qwiklabs-gcp-03-85cb4606093c)$ gcloud alpha services quota list --service=bigquery.googleapis.com --consumer=projects/${DEVSHELL_PROJECT_ID} --filter="usage"
---
consumerQuotaLimits:
- metric: bigquery.googleapis.com/quota/query/usage
  quotaBuckets:
  - defaultLimit: '9223372036854775807'
    effectiveLimit: '9223372036854775807'
  unit: 1/d/{project}
- metric: bigquery.googleapis.com/quota/query/usage
  quotaBuckets:
  - defaultLimit: '9223372036854775807'
    effectiveLimit: '9223372036854775807'
  unit: 1/d/{project}/{user}
displayName: Query usage
metric: bigquery.googleapis.com/quota/query/usage
unit: MiBy
student_01_e488ef31cb8a@cloudshell:~ (qwiklabs-gcp-03-85cb4606093c)$ 

```

The **consumerQuotaLimits** display your current query per day limits. There is a separate quota for **usage per project** and **usage per user**.

Run this command in **Cloud Shell** to update your **per user** quota to **.25 TiB** per day:

```sh
student_01_e488ef31cb8a@cloudshell:~ (qwiklabs-gcp-03-85cb4606093c)$ gcloud alpha services quota update --consumer=projects/${DEVSHELL_PROJECT_ID} --service bigquery.googleapis.com --metric bigquery.googleapis.com/quota/query/usage --value 262144 --unit 1/d/{project}/{user} --force
Operation "operations/quf.p33-410213670204-55b0dda3-dc6e-484d-843a-d0d1b4c299b2" finished successfully.
student_01_e488ef31cb8a@cloudshell:~ (qwiklabs-gcp-03-85cb4606093c)$ 
```

After the quota is updated, examine your **consumerQuotaLimits** again:

```sh
student_01_e488ef31cb8a@cloudshell:~ (qwiklabs-gcp-03-85cb4606093c)$ gcloud alpha services quota list --service=bigquery.googleapis.com --consumer=projects/${DEVSHELL_PROJECT_ID} --filter="usage"
---
consumerQuotaLimits:
- metric: bigquery.googleapis.com/quota/query/usage
  quotaBuckets:
  - defaultLimit: '9223372036854775807'
    effectiveLimit: '9223372036854775807'
  unit: 1/d/{project}
- metric: bigquery.googleapis.com/quota/query/usage
  quotaBuckets:
  - consumerOverride:
      name: projects/410213670204/services/bigquery.googleapis.com/consumerQuotaMetrics/bigquery.googleapis.com%2Fquota%2Fquery%2Fusage/limits/%2Fd%2Fproject%2Fuser/consumerOverrides/Cg1RdW90YU92ZXJyaWRl
      overrideValue: '262144'
    defaultLimit: '9223372036854775807'
    effectiveLimit: '262144'
  unit: 1/d/{project}/{user}
displayName: Query usage
metric: bigquery.googleapis.com/quota/query/usage
unit: MiBy
student_01_e488ef31cb8a@cloudshell:~ (qwiklabs-gcp-03-85cb4606093c)$ 
```

You should see the same limits from before but also a **consumerOverride** with the value used in the previous step:

Next, you will re-run your query with the updated quota.

## Rerun your query

In the **Cloud Console**, click **BigQuery**.

The query you previously ran should still be in the query editor, but if it isn't, paste the following query in the Query editor and click **Run**:

Note the validator still mentions `This query will process 1.36 TB when run`. However, the query has run successfully and hasn't processed any data. Why do you think that is?

Running the same query again may not process any data because of the automatic query ___caching___ feature in BigQuery.                  

If your query is already blocked by your custom quota, don't worry. It's likely that you set the custom quota and re-run the query before the  first query had time to cache the results.

Queries that use cached query results are at no additional charge and do not count against your quota. For more information on using cached  query results, see [Using cached query results](https://cloud.google.com/bigquery/docs/cached-results).

In order for us to test the newly set quota, you must to disable query cache to process data using the previous query.

To test that the quota has changed, disable the cached query results. In the **Query results** pane, click **More** > **Query settings**:

Uncheck **Use cached results** and click **Save**.

Run the query again so that it counts against your daily quota.

Once the query has run successfully and processed the 1.36 TB, run the query once more.

What happened? Were you able to run the query? You should have received an error like the following:

```sh
Custom quota exceeded: Your usage exceeded the custom quota for QueryUsagePerUserPerDay, which is set by your administrator. For more information, see https://cloud.google.com/bigquery/cost-controls 
```


## Explore BigQuery best practices

Quotas can be used for cost controls but it's up to your business to  determine which quotas make sense for your team. This is one example of  how to set quotas to protect from unexpected costs. One way to reduce  the amount of data queried is to optimize your queries.

Learn more about optimizing BigQuery queries from the [Control costs in BigQuery guide](https://cloud.google.com/bigquery/docs/best-practices-costs).

In this lab you completed the following tasks:

- Queried a public dataset and explore associated costs.
- Modified BigQuery API quota.
- Tried to rerun the query after quota had been modified.

## History

```sh
student_01_e488ef31cb8a@cloudshell:~ (qwiklabs-gcp-03-85cb4606093c)$ history 
    1  gcloud alpha services quota list --service=bigquery.googleapis.com --consumer=projects/${DEVSHELL_PROJECT_ID} --filter="usage"
    2  gcloud alpha services quota update --consumer=projects/${DEVSHELL_PROJECT_ID} --service bigquery.googleapis.com --metric bigquery.googleapis.com/quota/query/usage --value 262144 --unit 1/d/{project}/{user} --force
    3  gcloud alpha services quota list --service=bigquery.googleapis.com --consumer=projects/${DEVSHELL_PROJECT_ID} --filter="usage"
    4  history 
student_01_e488ef31cb8a@cloudshell:~ (qwiklabs-gcp-03-85cb4606093c)$ 
```

