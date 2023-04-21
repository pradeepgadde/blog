---
layout: single
title:  "Examining Billing data with BigQuery"
date:   2023-04-21 09:59:04 +0530
categories: Cloud
tags: GCP
classes: wide
show_date: true
header:
  overlay_image: /assets/images/gcp-banner-1.png
  og_image: /assets/images/gcp-banner-1.png
  teaser: /assets/images/gpcne.png
author:
  name     : "Cloud Network Engineer"
  avatar   : "/assets/images/gpcne.png"

sidebar:
  - title: "Topics"
    nav: my-sidebar

---

# Examining Billing data with BigQuery

Perform the following tasks:

- Sign in to BigQuery from the Cloud Console
  
- Create a dataset
  
- Create a table
  
- Import data from a billing CSV file stored in a bucket
  
- Run complex queries on a larger dataset



## 1. Use BigQuery to import data

![]({{ site.url }}{{ site.baseurl }}/assets/images/gcp-billing-1.png)

![]({{ site.url }}{{ site.baseurl }}/assets/images/gcp-billing-2.png)

![]({{ site.url }}{{ site.baseurl }}/assets/images/gcp-billing-3.png)



## 2. Examine the table

![]({{ site.url }}{{ site.baseurl }}/assets/images/gcp-billing-4.png)

![]({{ site.url }}{{ site.baseurl }}/assets/images/gcp-billing-5.png)

![]({{ site.url }}{{ site.baseurl }}/assets/images/gcp-billing-6.png)

![]({{ site.url }}{{ site.baseurl }}/assets/images/gcp-billing-7.png)



## 3. Compose a simple query

When you reference a table in a query, both the dataset ID and table ID must be specified; the project ID is optional.

```sql
SELECT * FROM `imported_billing_data.sampleinfotable`
WHERE Cost > 0
```

![]({{ site.url }}{{ site.baseurl }}/assets/images/gcp-billing-8.png)

![]({{ site.url }}{{ site.baseurl }}/assets/images/gcp-billing-9.png)

![]({{ site.url }}{{ site.baseurl }}/assets/images/gcp-billing-10.png)



## 4. Analyze a large billing dataset with SQL

In the next activity, you use BigQuery to analyze a sample dataset with 22,537 lines of billing data.

![]({{ site.url }}{{ site.baseurl }}/assets/images/gcp-billing-11.png)

![]({{ site.url }}{{ site.baseurl }}/assets/images/gcp-billing-12.png)

![]({{ site.url }}{{ site.baseurl }}/assets/images/gcp-billing-13.png)

![]({{ site.url }}{{ site.baseurl }}/assets/images/gcp-billing-14.png)

![]({{ site.url }}{{ site.baseurl }}/assets/images/gcp-billing-15.png)

![]({{ site.url }}{{ site.baseurl }}/assets/images/gcp-billing-16.png)

![]({{ site.url }}{{ site.baseurl }}/assets/images/gcp-billing-17.png)





## 5. Review

In this lab, you imported billing data into BigQuery that had been  generated as a CSV file. You ran a simple query on the file. Then you  accessed a shared dataset containing more than 22,000 records of billing information. You ran a variety of queries on that data to explore how  you can use BigQuery to ask and answer questions by running queries.
