---

layout: single
title:  "Cloud Spanner: Qwik Start"
categories: Cloud
tags: GCP
toc: true
toc_sticky: true
show_date: true
header:
  overlay_image: /assets/images/gcp-banner.png
  og_image: /assets/images/gcp-banner.png
  teaser: /assets/images/ace-gcp.png
author:
  name     : "Associate Cloud Engineer"
  avatar   : "/assets/images/ace-gcp.png"
sidebar:
  - title: "Topics"
    nav: my-sidebar

---

# Cloud Spanner: Qwik Start

Cloud Spanner is a fully managed, mission-critical, relational  database service that offers transactional consistency at global scale,  schemas, SQL (ANSI 2011 with extensions), and automatic, synchronous  replication for high availability.

Cloud Spanner offers:

- **Strong consistency**, including strongly consistent secondary indexes.
- **SQL support**, with ALTER statements for schema changes.
- **Managed instances with high availability** through transparent, synchronous, built-in data replication.

Cloud Spanner offers regional and multi-region instance configurations.

Cloud Spanner is ideal for relational, structured, and semi-structured  data that requires high availability, strong consistency, and  transactional reads and writes.

## Create an instance

When you first use Cloud Spanner, you must create an instance, which  is an allocation of resources that are used by Cloud Spanner databases  in that instance.

1. From the Console, from the **Navigation menu** select **Spanner**.

2. Then click **CREATE A PROVISIONED INSTANCE**.

3. Fill in the following field:

   Instance Name:  **Test Instance**

4. Leave the **Instance ID** as default and click **Continue**.

5. In **Choose a configuration** page, select **Regional** configuration and from the drop-down menu select region as .

6. Click **Continue**.

7. Leave the **Configure compute capacity** page as default and click **CREATE**.

```sh
student_01_aa8f3a74b714@cloudshell:~ (qwiklabs-gcp-03-fb00428c1359)$ gcloud spanner instances list
NAME: test-instance
DISPLAY_NAME: Test Instance
CONFIG: regional-us-east4
NODE_COUNT: 1
PROCESSING_UNITS: 1000
STATE: READY
INSTANCE_TYPE: PROVISIONED
student_01_aa8f3a74b714@cloudshell:~ (qwiklabs-gcp-03-fb00428c1359)$ gcloud spanner instances describe test-instance
config: projects/qwiklabs-gcp-03-fb00428c1359/instanceConfigs/regional-us-east4
createTime: '2024-05-15T02:23:25.034801Z'
displayName: Test Instance
instanceType: PROVISIONED
name: projects/qwiklabs-gcp-03-fb00428c1359/instances/test-instance
nodeCount: 1
processingUnits: 1000
state: READY
updateTime: '2024-05-15T02:23:25.034801Z'
student_01_aa8f3a74b714@cloudshell:~ (qwiklabs-gcp-03-fb00428c1359)$ 
```

## Create a database

1. In the **Instance Overview** page, click **CREATE DATABASE**.
2. For the Database name, enter **example-db**, and click **CREATE**.

You're now on the **Database Overview** page for the new database you created.

```sh
student_01_aa8f3a74b714@cloudshell:~ (qwiklabs-gcp-03-fb00428c1359)$ gcloud spanner databases list --instance test-instance
NAME: example-db
STATE: READY
VERSION_RETENTION_PERIOD: 1h
EARLIEST_VERSION_TIME: 2024-05-15T02:26:05.577229Z
KMS_KEY_NAME: 
ENABLE_DROP_PROTECTION: 
student_01_aa8f3a74b714@cloudshell:~ (qwiklabs-gcp-03-fb00428c1359)$ 
```

## Create a schema

The Cloud Console provides two ways to create, alter, and delete tables and indexes in a database:

- Use the default Database editor to specify each part of tables, columns, and indexes.
- Enter SQL statements in Cloud Spanner Data Definition Language (DDL) syntax.

This lab uses DDL.

1. Now click **CREATE TABLE**.
2. In the **DDL TEMPLATES** field, replace the existing query with:
3. Click **SUBMIT**.

```sql
CREATE TABLE Singers (
  SingerId   INT64 NOT NULL,
  FirstName  STRING(1024),
  LastName   STRING(1024),
  SingerInfo BYTES(MAX),
  BirthDate  DATE,
) PRIMARY KEY(SingerId);
```


## Insert and modify data

The Cloud Console provides an interface for inserting, editing, and deleting data.

### Insert data

1. Click **Singers**
2. In the left menu click **Data**, then click the **INSERT** button.
3. Replace the placeholder values for the following fields:
4. Then click **RUN**.
5. In the left menu, navigate to **Data** and you'll see that the **Singers** table now has one row.
6. Click **INSERT** to add an additional row and replace the placeholder values with the following:
7. Then click **RUN**.
8. In the left menu, navigate to **Data** and you'll see that the **Singers** table now has two rows.

You can also insert empty string values when you enter data.

1. Click **INSERT**.
2. Then enter in the following:
3. Then click **RUN**.
4. In the left menu, navigate to **Data** and you'll see that the **Singers** table now has three rows, and the row for **SingerID** `3` has a **LastName** that is an empty string.

```sql
  -- Add new values in the VALUES clause in order of the column list.
  -- Each value must be type compatible with its associated column.
INSERT INTO
  Singers (SingerId,
    FirstName,
    LastName,
    SingerInfo,
    BirthDate)
VALUES
  (1, -- type: INT64
    'Marc', -- type: STRING(1024)
    'Richards', -- type: STRING(1024)
    NULL, -- type: BYTES(MAX)
    NULL -- type: DATE
    )
THEN RETURN
  SingerId,
  FirstName,
  LastName,
  SingerInfo,
  BirthDate;
```

### Edit data

You will continue working on the Singers table.

1. Check the box next to the row for **SingerId** `3`.
2. Then click **EDIT**.
3. Enter in the following:

1. hen Click **RUN**.
2. In the left menu navigate to **Data**.

The row for **SingerId** `3` in the **Singers** table now has a **BirthDate** value.

```sql
  -- Change values in the SET clause to update the row where the WHERE condition is true.
UPDATE
  Singers
SET
  FirstName='Kena',
  LastName='',
  SingerInfo=NULL,
  BirthDate='1961-04-01'
WHERE
  SingerId=3
THEN RETURN
  SingerId,
  FirstName,
  LastName,
  SingerInfo,
  BirthDate;
```



### Delete data

Now try deleting some data from the table.

1. Check the box next to the row for **SingerId** `2`.
2. Then click **Delete**.

You can safely ignore the warning that appears in the dialog.

1. In the panel, click **Confirm**.

The **Singers** table now has two rows.

### Run a query

You can execute a SQL statement on the query page of your database.

1. Navigate to your Spanner database by selecting **Google Standard SQL Database** in the path at the top of your current table data page:

1. Click **Spanner Studio** from the left menu.
2. Click **Clease Query** to clear the query you used for your table.
3. For the query enter:

```sql
SELECT
  *
FROM
  Singers
```














