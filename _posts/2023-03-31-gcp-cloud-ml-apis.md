---

layout: single
title:  "Extract, Analyze, and Translate Text from Images with the Cloud ML APIs"
date:   2023-03-31 02:59:04 +0530
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

# Extract, Analyze, and Translate Text from Images with the Cloud ML APIs

- Creating a Vision API request and calling the API with curl
- Using the text detection (OCR) method of the Vision API
- Using the Translation API to translate text from your image
- Using the Natural Language API to analyze the text

## 1. Create an API key

Since you'll be using `curl` to send a request to the Vision API, you'll need to generate an API key to pass in your request URL.

1. To create an API key, navigate to: **Navigation Menu** > **APIs & services** > **Credentials**:

## 2. Upload an image to a Cloud Storage bucket

### Creating a Cloud Storage bucket

There are two ways to send an image to the Vision API for image  detection: by sending the API a base64 encoded image string, or passing  it the URL of a file stored in Cloud Storage. For this lab you'll create a Cloud Storage bucket to store your images.

1. Navigate to the **Navigation menu** > **Cloud Storage** browser in the Console.
2. Then click **Create bucket**.
3. Give your bucket a globally unique name and click on **Choose how to control access to objects**.
4. Uncheck the box for **Enforce public access prevention on this bucket**.
5. Choose **Fine-grained** under Access Control and click **Create**.

## 3. Create your Vision API request

```json
student_01_c41e1c322859@cloudshell:~ (qwiklabs-gcp-04-ac2146a4cfe4)$ cat ocr-request.json
{
  "requests": [
      {
        "image": {
          "source": {
              "gcsImageUri": "gs://pradeep-gcp-cs/sign.jpg"
          }
        },
        "features": [
          {
            "type": "TEXT_DETECTION",
            "maxResults": 10
          }
        ]
      }
  ]
}
student_01_c41e1c322859@cloudshell:~ (qwiklabs-gcp-04-ac2146a4cfe4)$
```



## 4. Call the Vision API's text detection method

```sh
curl -s -X POST -H "Content-Type: application/json" --data-binary @ocr-request.json  https://vision.googleapis.com/v1/images:annotate?key=${API_KEY}
```



```sh
student_01_c41e1c322859@cloudshell:~ (qwiklabs-gcp-04-ac2146a4cfe4)$ curl -s -X POST -H "Content-Type: application/json" --data-binary @ocr-request.json  https://vision.googleapis.com/v1/images:annotate?key=${API_KEY} -o ocr-response.json
student_01_c41e1c322859@cloudshell:~ (qwiklabs-gcp-04-ac2146a4cfe4)$
```



## 5. Sending text from the image to the Translation API

```sh
1_c41e1c322859@cloudshell:~ (qwiklabs-gcp-04-ac2146a4cfe4)$
student_01_c41e1c322859@cloudshell:~ (qwiklabs-gcp-04-ac2146a4cfe4)$ curl -s -X POST -H "Content-Type: application/json" --data-binary @ocr-request.json  https://vision.googleapis.com/v1/images:annotate?key=${API_KEY} -o ocr-response.json
student_01_c41e1c322859@cloudshell:~ (qwiklabs-gcp-04-ac2146a4cfe4)$ vi translation-request.json
student_01_c41e1c322859@cloudshell:~ (qwiklabs-gcp-04-ac2146a4cfe4)$ STR=$(jq .responses[0].textAnnotations[0].description ocr-response.json) && STR="${STR//\"}" && sed -i "s|your_text_here|$STR|g" translation-request.json
student_01_c41e1c322859@cloudshell:~ (qwiklabs-gcp-04-ac2146a4cfe4)$ curl -s -X POST -H "Content-Type: application/json" --data-binary @translation-request.json https://translation.googleapis.com/language/translate/v2?key=${API_KEY} -o translation-response.json
student_01_c41e1c322859@cloudshell:~ (qwiklabs-gcp-04-ac2146a4cfe4)$ cat translation-response.json
{
  "data": {
    "translations": [
      {
        "translatedText": "THE PUBLIC GOOD the dispatches For Obama, the mustard is from Dijon ATELINE",
        "detectedSourceLanguage": "fr"
      }
    ]
  }
}
student_01_c41e1c322859@cloudshell:~ (qwiklabs-gcp-04-ac2146a4cfe4)$
```



## 6. Analyzing the image's text with the Natural Language API

```sh
student_01_c41e1c322859@cloudshell:~ (qwiklabs-gcp-04-ac2146a4cfe4)$ vi nl-request.json
student_01_c41e1c322859@cloudshell:~ (qwiklabs-gcp-04-ac2146a4cfe4)$ STR=$(jq .data.translations[0].translatedText  translation-response.json) && STR="${STR//\"}" && sed -i "s|your_text_here|$STR|g" nl-request.json
student_01_c41e1c322859@cloudshell:~ (qwiklabs-gcp-04-ac2146a4cfe4)$ curl "https://language.googleapis.com/v1/documents:analyzeEntities?key=${API_KEY}" \
  -s -X POST -H "Content-Type: application/json" --data-binary @nl-request.json
{
  "entities": [
    {
      "name": "dispatches",
      "type": "OTHER",
      "metadata": {},
      "salience": 0.31229365,
      "mentions": [
        {
          "text": {
            "content": "dispatches",
            "beginOffset": 20
          },
          "type": "COMMON"
        }
      ]
    },
    {
      "name": "PUBLIC GOOD",
      "type": "OTHER",
      "metadata": {
        "mid": "/m/017bkk",
        "wikipedia_url": "https://en.wikipedia.org/wiki/Public_good_(economics)"
      },
      "salience": 0.27652362,
      "mentions": [
        {
          "text": {
            "content": "PUBLIC GOOD",
            "beginOffset": 4
          },
          "type": "PROPER"
        }
      ]
    },
    {
      "name": "Obama",
      "type": "PERSON",
      "metadata": {
        "mid": "/m/02mjmr",
        "wikipedia_url": "https://en.wikipedia.org/wiki/Barack_Obama"
      },
      "salience": 0.19338216,
      "mentions": [
        {
          "text": {
            "content": "Obama",
            "beginOffset": 35
          },
          "type": "PROPER"
        }
      ]
    },
    {
      "name": "mustard",
      "type": "OTHER",
      "metadata": {},
      "salience": 0.12168745,
      "mentions": [
        {
          "text": {
            "content": "mustard",
            "beginOffset": 46
          },
          "type": "COMMON"
        }
      ]
    },
    {
      "name": "Dijon ATELINE",
      "type": "OTHER",
      "metadata": {},
      "salience": 0.09611311,
      "mentions": [
        {
          "text": {
            "content": "Dijon ATELINE",
            "beginOffset": 62
          },
          "type": "PROPER"
        }
      ]
    }
  ],
  "language": "en"
}
student_01_c41e1c322859@cloudshell:~ (qwiklabs-gcp-04-ac2146a4cfe4)$
```



![]({{ site.url }}{{ site.baseurl }}/assets/images/gcp-431.png)

![]({{ site.url }}{{ site.baseurl }}/assets/images/gcp-432.png)

![]({{ site.url }}{{ site.baseurl }}/assets/images/gcp-433.png)

![]({{ site.url }}{{ site.baseurl }}/assets/images/gcp-sign.png)



