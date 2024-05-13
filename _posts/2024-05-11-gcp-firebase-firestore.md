---

layout: single
title:  "Getting Started with Firebase Cloud Firestore"
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

# Getting Started with Firebase Cloud Firestore

[Cloud Firestore](https://firebase.google.com/docs/firestore) is a flexible, scalable database for mobile, web, and server  development from Firebase and Google Cloud. Firebase Cloud Firestore is a NoSQL document database. This means your data is stored in documents  organized in collections. Documents can contain a variety of data types, including strings, numbers, arrays, objects, and geopoints. Like  Firebase Realtime Database, it keeps your data in sync across client  apps through real time listeners and offers offline support for mobile  and web so you can build responsive apps that work regardless of network latency or Internet connectivity. Cloud Firestore also offers seamless  integration with other Firebase and Google Cloud products, including  Cloud Functions.

Firebase Cloud Firestore provides eventual consistency by default.  This means that your data may not be immediately available to all  clients after it is written. However, you can use transactions to ensure your data is always consistent.

Installing Firebase

Creating a Firebase Application

Using the Firebase Emulator

Creating a Cloud Firestore Database

Writing content to the database

Reading content from the database



## Setting Database Security Rules

Before the database can be used, the security rules need to be  configured. In this lab Cloud Firestore will be used in development  mode.

The Firestore database can now be accessed by the application. The  default security rules setting does not permit read/write access to the  database. Ensure the security rules reflect the desired access, to learn more on this subject visit [Get started with Cloud Firestore Security Rules | Firebase](https://firebase.google.com/docs/firestore/security/get-started#auth-required).

With the database security rules in place, the next step is to  configure the environment. Learn more about it in the next section.





## Configuring the Firebase Environment

Before you can add Firebase to your JavaScript app, you need to  create a Firebase project and register your app with that project. When  you register your app with Firebase, you'll get a Firebase configuration object that you'll use to connect your app with your Firebase project  resources.



## Creating a Firebase Application

The following section creates the elements required to perform  Firebase Authentication using an email/password combination on the web.



## Adding a Webpack configuration

Webpack is a common method of bundling web code and assets.



## Writing to a Firestore Document

Update the Cloud Firestore firestore configuration to write information to the database.

To add content to the linked Firebase project, use the **getFirestore** call. Update the **src/index.js** code created earlier to write to the project Firestore database.



## Reading a Firestore Document

Update the Cloud Firestore firestore configuration to read information from the database.

To access document information from the linked Firebase project, add the `getDoc` call. Update the `src/index.js` code created earlier to enable the application to read from the project Firestore database.



In just 30 minutes, you developed a solid understanding of Firebase on  the Web and the key features. You learned about installing Firebase  Firestore, writing information to the database and reading a document.  You are now ready to take more labs.
