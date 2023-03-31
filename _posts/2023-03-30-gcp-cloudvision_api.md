---

layout: single
title:  "Detect Labels, Faces, and Landmarks in Images with the Cloud Vision API"
date:   2023-03-30 13:59:04 +0530
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

# Detect Labels, Faces, and Landmarks in Images with the Cloud Vision API


The Cloud Vision API lets you understand the content of an image by encapsulating powerful machine learning models in a simple REST API.



   -  Creating a Vision API request and calling the API with curl

   -  Using the label, face, and landmark detection methods of the vision API

Since you'll be using curl to send a request to the Vision API, you'll need to generate an API key to pass in your request URL.

To create an API key, from the Navigation menu go to APIs & Services > Credentials in your Cloud Console.

Click Create Credentials and select API key.



Save it to an environment variable to avoid having to insert the value of your API key in each request.

### Creating a Cloud Storage bucket

There are two ways to send an image to the Vision API for image  detection: by sending the API a base64 encoded image string, or passing  it the URL of a file stored in Cloud Storage. We'll be using a Cloud  Storage URL. The first step is to create a Cloud Storage bucket to store our images.

1. From the **Navigation menu**, select **Cloud Storage** > **Buckets**. Next to **Buckets**, click **Create**.
2. Give your bucket a unique name, i.e your Project ID.
3. After naming your bucket, click **Choose how to control access to objects**.
4. Uncheck **Enforce public access prevention on this bucket** and select the **Fine-grained** circle.

All other settings for your bucket can remain as the default setting.

1. Click **Create**.

Upload files and modify access settings

Click **Add entry** then enter the following:

- **Entity:** Public
- **Name:** allUsers
- **Access:** Reader

create a `request.json` file that specifies the feature type



Label Detection

Web Detection

Face Detection

Landmark annotation

Object localization

```sh
curl -s -X POST -H "Content-Type: application/json" --data-binary @request.json  https://vision.googleapis.com/v1/images:annotate?key=${API_KEY}
```



```sh
student_01_c41e1c322859@cloudshell:~ (qwiklabs-gcp-04-a3c77a21135e)$ history
    1  export API_KEY=export API_KEY=AIzaSyDl0knmDlMzTq2RjiMMBmB_3SIXwvQxVz4
    2  curl -s -X POST -H "Content-Type: application/json" --data-binary @request.json  https://vision.googleapis.com/v1/images:annotate?key=${API_KEY}
    3  ls
    4  ls /
    5  pwd
    6  ls home
    7  ls /home
    8  vi request.json
    9  ls
   10  curl -s -X POST -H "Content-Type: application/json" --data-binary @request.json  https://vision.googleapis.com/v1/images:annotate?key=${API_KEY}
   11  vi request.json
   12  curl -s -X POST -H "Content-Type: application/json" --data-binary @request.json  https://vision.googleapis.com/v1/images:annotate?key=${API_KEY}
   13  vi request.json
   14  mv request.json request.json.bkp
   15  vi request.json
   16  curl -s -X POST -H "Content-Type: application/json" --data-binary @request.json  https://vision.googleapis.com/v1/images:annotate?key=${API_KEY}
   17  mv request.json request.json.bkp2
   18  vi request.json
   19  curl -s -X POST -H "Content-Type: application/json" --data-binary @request.json  https://vision.googleapis.com/v1/images:annotate?key=${API_KEY}
   20  mv request.json request.json.bkp3
   21  vi request.json
   22  curl -s -X POST -H "Content-Type: application/json" --data-binary @request.json  https://vision.googleapis.com/v1/images:annotate?key=${API_KEY}
   23  history
student_01_c41e1c322859@cloudshell:~ (qwiklabs-gcp-04-a3c77a21135e)$
```






## Web Detection
```sh
student_01_c41e1c322859@cloudshell:~ (qwiklabs-gcp-04-a3c77a21135e)$ cat request.json.bkp
{
  "requests": [
      {
        "image": {
          "source": {
              "gcsImageUri": "gs://qwiklabs-gcp-04-a3c77a21135e/donuts.png"
          }
        },
        "features": [
          {
            "type": "WEB_DETECTION",
            "maxResults": 10
          }
        ]
      }
  ]
}
```
## Face Detection 
```sh
student_01_c41e1c322859@cloudshell:~ (qwiklabs-gcp-04-a3c77a21135e)$ cat request.json
{
  "requests": [
      {
        "image": {
          "source": {
              "gcsImageUri": "gs://qwiklabs-gcp-04-a3c77a21135e/selfie.png"
          }
        },
        "features": [
          {
            "type": "FACE_DETECTION"
          },
          {
            "type": "LANDMARK_DETECTION"
          }
        ]
      }
  ]
}
```
## Landmark Detection
```sh
student_01_c41e1c322859@cloudshell:~ (qwiklabs-gcp-04-a3c77a21135e)$ cat request.json
{
  "requests": [
      {
        "image": {
          "source": {
              "gcsImageUri": "gs://qwiklabs-gcp-04-a3c77a21135e/city.png"
          }
        },
        "features": [
          {
            "type": "LANDMARK_DETECTION",
            "maxResults": 10
          }
        ]
      }
  ]
}

```
## Object Localization
```sh

student_01_c41e1c322859@cloudshell:~ (qwiklabs-gcp-04-a3c77a21135e)$ cat request.json
{
  "requests": [
    {
      "image": {
        "source": {
          "imageUri": "https://cloud.google.com/vision/docs/images/bicycle_example.png"
        }
      },
      "features": [
        {
          "maxResults": 10,
          "type": "OBJECT_LOCALIZATION"
        }
      ]
    }
  ]
}
student_01_c41e1c322859@cloudshell:~ (qwiklabs-gcp-04-a3c77a21135e)$
```

We've learned how to analyze images with the Vision API. In this lab we passed the API the Cloud Storage URL of different images, and it returned the labels, faces, landmarks, and objects it found in the image. We can also pass the API a base64 encoded string of an image, which is useful if we want to analyze an image that's stored in a database or in memory.

