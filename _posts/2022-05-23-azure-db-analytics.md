---

layout: single
title:  "Azure Database and Analytics Services"
date:   2022-05-23 04:59:04 +0530
categories: Cloud
tags: Azure
show_date: true
classes: wide
header:
  teaser: /assets/images/microsoft-certified-fundamentals-badge.svg
author:
  name     : "Microsoft"
  avatar   : "/assets/images/azure.png"

sidebar:
  - title: "Blog"
   
    text: "Checkout other topics"
    nav: my-sidebar

---
In this post let us look at several of the database services that are available on Microsoft Azure, such as Azure Cosmos DB, Azure SQL Database, Azure SQL Managed Instance, Azure Database for MySQL, and Azure Database for PostgreSQL.

Azure database services are globally distributed, and Azure supports many of the industry standard databases and APIs.

## Azure Cosmos DB
Azure Cosmos DB is a globally distributed, multi-model database service. 
Azure Cosmos DB supports schemaless data, which lets you build highly responsive and "Always On" applications to support constantly changing data. You can use this feature to store data that's updated and maintained by users around the world.

Azure Cosmos DB is flexible. 

At the lowest level, Azure Cosmos DB stores data in atom-record-sequence (ARS) format. The data is then abstracted and projected as an API, which you specify when you're creating your database. 

Your choices include SQL, MongoDB, Cassandra, Tables, and Gremlin. This level of flexibility means that as you migrate your company's databases to Azure Cosmos DB, your developers can stick with the API with which they're the most comfortable.

## Azure SQL Database
Azure SQL Database is a relational database based on the latest stable version of the Microsoft SQL Server database engine. 

Azure SQL Database is a platform as a service (PaaS) database engine.

SQL Database is a fully managed service that has built-in high availability, backups, and other common maintenance operations. Microsoft handles all updates to the SQL and operating system code. You don't have to manage the underlying infrastructure.

You can migrate your existing SQL Server databases with minimal downtime by using the Azure Database Migration Service.

## Azure Database for MySQL
Azure Database for MySQL is a relational database service in the cloud, and it's based on the MySQL Community Edition database engine, versions 5.6, 5.7, and 8.0. 

With every Azure Database for MySQL server, you take advantage of built-in security, fault tolerance, and data protection that you would otherwise have to buy or design, build, and manage. 

With Azure Database for MySQL, you can use point-in-time restore to recover a server to an earlier state, as far back as 35 days.

## Azure Database for PostgreSQL
Azure Database for PostgreSQL is a relational database service in the cloud. The server software is based on the community version of the open-source PostgreSQL database engine.

Azure Database for PostgreSQL is available in two deployment options: Single Server and Hyperscale (Citus).

## Azure SQL Managed Instance
Azure SQL Managed Instance is a scalable cloud data service that provides the broadest SQL Server database engine compatibility with all the benefits of a fully managed platform as a service. 

Azure SQL Database and Azure SQL Managed Instance offer many of the same features; however, Azure SQL Managed Instance provides several options that might not be available to Azure SQL Database.

## Big Data and Analytics
Microsoft Azure supports a broad range of technologies and services to provide big data and analytic solutions, including Azure Synapse Analytics, Azure HDInsight, Azure Databricks, and Azure Data Lake Analytics.

## Azure Synapse Analytics
Azure Synapse Analytics (formerly Azure SQL Data Warehouse) is a limitless analytics service that brings together enterprise-data warehousing and big-data analytics. 

You have a unified experience to ingest, prepare, manage, and serve data for immediate business intelligence and machine learning needs.

## Azure HDInsight
Azure HDInsight is a fully managed, open-source analytics service for enterprises. 

It's a cloud service that makes it easier, faster, and more cost-effective to process massive amounts of data. 
 
## Azure Databricks
Azure Databricks helps you unlock insights from all your data and build artificial intelligence solutions. You can set up your Apache Spark environment in minutes, then autoscale and collaborate on shared projects in an interactive workspace.

Azure Databricks supports Python, Scala, R, Java, and SQL, as well as data science frameworks and libraries including TensorFlow, PyTorch, and scikit-learn.


## Azure Data Lake Analytics
Azure Data Lake Analytics is an on-demand analytics job service that simplifies big data. Instead of deploying, configuring, and tuning hardware, you can write queries to transform your data and extract valuable insights.

The analytics service can handle jobs of any scale instantly by setting the dial for how much power you need. You only pay for your job when it's running, making it more cost-effective.
