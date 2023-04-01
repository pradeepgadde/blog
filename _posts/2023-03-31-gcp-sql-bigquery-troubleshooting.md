---

layout: single
title:  "Troubleshooting Common SQL Errors with BigQuery "
date:   2023-03-31 06:59:04 +0530
categories: Cloud
tags: GCP
classes: wide
show_date: true
header:
  overlay_image: /assets/images/gcp-banner.png
  og_image: /assets/images/gcp-banner.png
  teaser: /assets/images/gpcne.png
author:
  name     : "Cloud Network Engineer"
  avatar   : "/assets/images/gpcne.png"

sidebar:
  - title: "Topics"
    nav: my-sidebar

---

# Troubleshooting Common SQL Errors with BigQuery

In this lab, learn how to perform the following tasks:

- Query the data-to-insights public dataset
- Use the BigQuery Query editor to troubleshoot common SQL errors
- Use the Query Validator
- Troubleshoot syntax and logical SQL errors

## BigQuery Query editor

Copy and  paste the query into the BigQuery Query editor. If there are errors, you see a red exclamation point at the line containing the error and in the query validator (bottom corner). If you run the query with the errors, the query fails and the error is specified in the Job information. When the query is error free, you see a green checkmark in the query validator. When you see the green checkmark, click **Run** to run the query to view what you get for results.

Your goal in this section is to construct a query that gives you the number of unique visitors who successfully went through the checkout process for your website. The data is in the `rev_transactions` table which your data analyst team has provided. 

### Troubleshoot queries that contain query validator, alias, and comma errors

```sql
#standardSQL
SELECT  FROM `data-to-inghts.ecommerce.rev_transactions` LIMIT 1000
```
![]({{ site.url }}{{ site.baseurl }}/assets/images/gcp-457.png)

```sql
#standardSQL
SELECT * FROM [data-to-insights:ecommerce.rev_transactions] LIMIT 1000
```
![]({{ site.url }}{{ site.baseurl }}/assets/images/gcp-458.png)

```sql
#standardSQL
SELECT FROM `data-to-insights.ecommerce.rev_transactions`
```
![]({{ site.url }}{{ site.baseurl }}/assets/images/gcp-459.png)
```sql
#standardSQL
SELECT
fullVisitorId
FROM `data-to-insights.ecommerce.rev_transactions`
```
![]({{ site.url }}{{ site.baseurl }}/assets/images/gcp-460.png)
```sql
#standardSQL
SELECT fullVisitorId hits_page_pageTitle
FROM `data-to-insights.ecommerce.rev_transactions` LIMIT 1000
```
![]({{ site.url }}{{ site.baseurl }}/assets/images/gcp-461.png)
```sql
#standardSQL
SELECT
  fullVisitorId
  , hits_page_pageTitle
FROM `data-to-insights.ecommerce.rev_transactions` LIMIT 1000
```

![]({{ site.url }}{{ site.baseurl }}/assets/images/gcp-462.png)

```sql
#standardSQL
SELECT
COUNT(fullVisitorId) AS visitor_count
, hits_page_pageTitle
FROM `data-to-insights.ecommerce.rev_transactions`
```

![]({{ site.url }}{{ site.baseurl }}/assets/images/gcp-463.png)

```sql
#standardSQL
SELECT
COUNT(DISTINCT fullVisitorId) AS visitor_count
, hits_page_pageTitle
FROM `data-to-insights.ecommerce.rev_transactions`
GROUP BY hits_page_pageTitle
```

![]({{ site.url }}{{ site.baseurl }}/assets/images/gcp-464.png)

```sql
SELECT
geoNetwork_city,
totals_transactions,
COUNT( DISTINCT fullVisitorId) AS distinct_visitors
FROM
`data-to-insights.ecommerce.rev_transactions`
GROUP BY
```

![]({{ site.url }}{{ site.baseurl }}/assets/images/gcp-465.png)

```sql
#standardSQL
SELECT
geoNetwork_city,
SUM(totals_transactions) AS totals_transactions,
COUNT( DISTINCT fullVisitorId) AS distinct_visitors
FROM
`data-to-insights.ecommerce.rev_transactions`
GROUP BY geoNetwork_city
```
![]({{ site.url }}{{ site.baseurl }}/assets/images/gcp-466.png)
```sql
#standardSQL
SELECT
geoNetwork_city,
SUM(totals_transactions) AS totals_transactions,
COUNT( DISTINCT fullVisitorId) AS distinct_visitors
FROM
`data-to-insights.ecommerce.rev_transactions`
GROUP BY geoNetwork_city
ORDER BY distinct_visitors DESC
```
![]({{ site.url }}{{ site.baseurl }}/assets/images/gcp-467.png)
```sql
#standardSQL
SELECT
geoNetwork_city,
SUM(totals_transactions) AS total_products_ordered,
COUNT( DISTINCT fullVisitorId) AS distinct_visitors,
SUM(totals_transactions) / COUNT( DISTINCT fullVisitorId) AS avg_products_ordered
FROM
`data-to-insights.ecommerce.rev_transactions`
GROUP BY geoNetwork_city
ORDER BY avg_products_ordered DESC
```
![]({{ site.url }}{{ site.baseurl }}/assets/images/gcp-468.png)
```sql
#standardSQL
SELECT
geoNetwork_city,
SUM(totals_transactions) AS total_products_ordered,
COUNT( DISTINCT fullVisitorId) AS distinct_visitors,
SUM(totals_transactions) / COUNT( DISTINCT fullVisitorId) AS avg_products_ordered
FROM
`data-to-insights.ecommerce.rev_transactions`
WHERE avg_products_ordered > 20
GROUP BY geoNetwork_city
ORDER BY avg_products_ordered DESC
```
![]({{ site.url }}{{ site.baseurl }}/assets/images/gcp-469.png)
```sql
#standardSQL
SELECT
geoNetwork_city,
SUM(totals_transactions) AS total_products_ordered,
COUNT( DISTINCT fullVisitorId) AS distinct_visitors,
SUM(totals_transactions) / COUNT( DISTINCT fullVisitorId) AS avg_products_ordered
FROM
`data-to-insights.ecommerce.rev_transactions`
GROUP BY geoNetwork_city
HAVING avg_products_ordered > 20
ORDER BY avg_products_ordered DESC
```
![]({{ site.url }}{{ site.baseurl }}/assets/images/gcp-470.png)
```sql
#standardSQL
SELECT hits_product_v2ProductName, hits_product_v2ProductCategory
FROM `data-to-insights.ecommerce.rev_transactions`
GROUP BY 1,2
```
![]({{ site.url }}{{ site.baseurl }}/assets/images/gcp-471.png)
```sql
#standardSQL
SELECT
COUNT(hits_product_v2ProductName) as number_of_products,
hits_product_v2ProductCategory
FROM `data-to-insights.ecommerce.rev_transactions`
WHERE hits_product_v2ProductName IS NOT NULL
GROUP BY hits_product_v2ProductCategory
ORDER BY number_of_products DESC
```
![]({{ site.url }}{{ site.baseurl }}/assets/images/gcp-472.png)
```sql
#standardSQL
SELECT
COUNT(DISTINCT hits_product_v2ProductName) as number_of_products,
hits_product_v2ProductCategory
FROM `data-to-insights.ecommerce.rev_transactions`
WHERE hits_product_v2ProductName IS NOT NULL
GROUP BY hits_product_v2ProductCategory
ORDER BY number_of_products DESC
LIMIT 5
```
![]({{ site.url }}{{ site.baseurl }}/assets/images/gcp-473.png)
![]({{ site.url }}{{ site.baseurl }}/assets/images/gcp-474.png)
![]({{ site.url }}{{ site.baseurl }}/assets/images/gcp-475.png)

You troubleshot and fixed broken queries in BigQuery standard SQL.  Remember to use the Query Validator for incorrect query syntax but also  to be critical of your query results even if your query executes  successfully.

  

  

