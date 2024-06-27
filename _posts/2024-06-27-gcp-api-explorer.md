---

layout: single
title:  "APIs Explorer: Qwik Start"
categories: Cloud
tags: GCP
classes: wide
show_date: true
header:
  overlay_image: /assets/images/gcp-banner-1.png
  og_image: /assets/images/gcp-banner-1.png
  teaser: /assets/images/pca-gcp.png
author:
  name     : "Professional Cloud Architect"
  avatar   : "/assets/images/pca-gcp.png"

sidebar:
  - title: "Topics"
    nav: my-sidebar

---
# APIs Explorer: Qwik Start

The [Google APIs Explorer](http://developers.google.com/apis-explorer/) is a tool that lets you explore various Google API methods without writing code. With the APIs Explorer you can:

- Browse quickly through available **APIs** and versions.
- See **methods** available for each API and what **parameters** they support along with inline documentation.
- Execute requests for any method and see responses in **real time**.
- Make **authenticated and authorized** API calls.
- Search across all **services**, **methods**, and your **recent requests** to quickly find what you are looking for.

The APIs Explorer uses its own [API keys](https://developers.google.com/console/help/using-keys) whenever it makes a request. When you use the APIs Explorer to make a  request, it displays the request syntax, which includes a placeholder  labeled `{YOUR_API_KEY}`. If you want to make the same request in your application, you need to replace this placeholder with your own API key.

- Create a Cloud Storage bucket.
- Upload an image to Cloud Storage and make it public.
- Make a request to the Vision API with that image.

## Create a Cloud Storage bucket

1. In the Cloud console, go to **Cloud Storage** > **Buckets**.
2. Click **Create bucket**.
3. Give your bucket a unique name. Do not include sensitive information in the bucket name - the bucket namespace is global and publicly visible. For example, -bucket.
4. Click **Choose how to control access to objects**.
5. Uncheck **Enforce public access prevention on this bucket**.
6. Select **Fine-grained** Access control.
7. Click **Create**.

## Upload an image

You'll be asking the Cloud Vision API to analyze an image through API Explorer. First, add an image to your bucket for it to analyze. You can use one of your own, or download the image below to your computer and  save it as `demo-image.jpg` .

After the file is uploaded and listed in your bucket, share the image publicly by following these steps:

1. Click the three vertical dot object overflow menu associated with your image.

2. Select **Edit access** from the drop-down menu.

3. In the overlay that appears, click the **+ Add entry** button.

4. Add a permission for 

   allUsers

   .

   - Select **Public** for the *Entity*.
   - Enter **allUsers** for the *Name*.
   - Select **Reader** for the *Access*.

5. Click **Save**.

## Make a request to the Cloud Vision API service

1. Go to **Navigation menu** > **APIs & Services**.
2. Click **+ ENABLE APIS AND SERVICES**, search for **Cloud Vision**, then select the **Cloud Vision API** from the results list and click on it.
3. Make sure that API is enabled, if not click **Enable**.
4. Open the [Cloud Vision - Try this API](https://cloud.google.com/vision/docs/reference/rest/v1/images/annotate?apix=true) link.

This will open a new tab with the APIs Explorer page loaded.

You will now be on the APIs Explorer page.

1. Click inside of the curly braces in the Request body field.
2. You'll be asked to select a property - choose "**requests**". This will generate the next level.
3. Click inside the brackets and click the blue plus sign icon, select `[Add Item]` - for your property select "**features**".
4. Inside "**features**" click inside the curly brace, click the blue plus icon and select `[Add Item]`, select "**type**"; next to it select **LABEL_DETECTION**.
5. You should have the blue plus icon at the end of the "**features**" block where you can choose to add "**image**"; then add "**source**", and "**imageUri**".
6. Next to "**imageUri**" enter the path to the image file in your bucket. The path should look like this: `gs://MY_BUCKET/demo-image.jpg`

When you're done, your Request body field should look like this:

```sh
{
  "requests": [
    {
      "features": [
        {
          "type": "LABEL_DETECTION"
        }
      ],
      "image": {
        "source": {
          "imageUri": "gs://qwiklabs-gcp-02-b9abd10d40f0-bucket/demo-image.jpg"
        }
      }
    }
  ]
}
```



Make sure that **Google OAuth 2.0** and **API key** checkboxes are selected under **Credentials** section.

1. On the right panel of an API Explorer console, you can see Cloud Vision API call with `cURL`, `HTTP` and `JAVASCRIPT`.

```sh
curl --request POST \
  'https://vision.googleapis.com/v1/images:annotate?key=[YOUR_API_KEY]' \
  --header 'Authorization: Bearer [YOUR_ACCESS_TOKEN]' \
  --header 'Accept: application/json' \
  --header 'Content-Type: application/json' \
  --data '{"requests":[{"features":[{"type":"LABEL_DETECTION"}],"image":{"source":{"imageUri":"gs://qwiklabs-gcp-02-b9abd10d40f0-bucket/demo-image.jpg"}}}]}' \
  --compressed
```



1. Now click **Execute**.
2. Select your student account.
3. On the next screen, click **Allow** to give APIs Explorer access.

```sh
{
  "responses": [
    {
      "labelAnnotations": [
        {
          "mid": "/m/0kpmf",
          "description": "Dog breed",
          "score": 0.89873457,
          "topicality": 0.89873457
        },
        {
          "mid": "/m/02p0tk3",
          "description": "Human body",
          "score": 0.88404673,
          "topicality": 0.88404673
        },
        {
          "mid": "/m/0bt9lr",
          "description": "Dog",
          "score": 0.8810844,
          "topicality": 0.8810844
        },
        {
          "mid": "/m/09141t",
          "description": "Collar",
          "score": 0.86219054,
          "topicality": 0.86219054
        },
        {
          "mid": "/m/0244x1",
          "description": "Gesture",
          "score": 0.85260487,
          "topicality": 0.85260487
        },
        {
          "mid": "/m/01lrl",
          "description": "Carnivore",
          "score": 0.85030043,
          "topicality": 0.85030043
        },
        {
          "mid": "/m/07_gml",
          "description": "Working animal",
          "score": 0.84733003,
          "topicality": 0.84733003
        },
        {
          "mid": "/m/08t9c_",
          "description": "Grass",
          "score": 0.8381825,
          "topicality": 0.8381825
        },
        {
          "mid": "/m/0276krm",
          "description": "Fawn",
          "score": 0.81617475,
          "topicality": 0.81617475
        },
        {
          "mid": "/m/03yl64",
          "description": "Companion dog",
          "score": 0.78090554,
          "topicality": 0.78090554
        }
      ]
    }
  ]
}

```

The results of Cloud Vision API analysis of the image will be below  to your right panel. The top part of the results should look like this:

You've made your first `images.annotate` request to the Cloud Vision API service.
