---

layout: single
title:  "Video Intelligence: Qwik Start"
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
# Video Intelligence: Qwik Start

Google Cloud Video Intelligence makes videos searchable and discoverable by extracting metadata with an easy to use REST API. You can now search every moment of every video file in your catalog. It quickly annotates  videos stored in [Cloud Storage](https://cloud.google.com/storage/), and helps you identify key entities (nouns) within your video; and when they occur within the video. Separate signal from noise by retrieving  relevant information within the entire video, shot-by-shot, -or per  frame.

- Set up authorization for a custom service account
- Send an annotate video request to the Video Intelligence API

## Set up authorization

For this lab, you create and use a service account that is tied to your Google Cloud project for authorization.

In Cloud Shell, run the following command to create a new service account named `quickstart`:

Create a service account key file, replacing `<your-project-123>` with your Project ID:

Now authenticate your service account, passing the location of your service account key file:

Obtain an authorization token using your service account:

```sh
Welcome to Cloud Shell! Type "help" to get started.
To set your Cloud Platform project in this session use “gcloud config set project [PROJECT_ID]”
student_01_932d053b64d1@cloudshell:~$ gcloud config set project qwiklabs-gcp-03-d5fcafe3101b 
Updated property [core/project].
student_01_932d053b64d1@cloudshell:~ (qwiklabs-gcp-03-d5fcafe3101b)$ gcloud iam service-accounts create quickstart
Created service account [quickstart].
student_01_932d053b64d1@cloudshell:~ (qwiklabs-gcp-03-d5fcafe3101b)$ gcloud iam service-accounts keys create key.json --iam-account quickstart@qwiklabs-gcp-03-d5fcafe3101b.iam.gserviceaccount.com
created key [5f8fe4300ac72e65cfd86b378d6837945b52a052] of type [json] as [key.json] for [quickstart@qwiklabs-gcp-03-d5fcafe3101b.iam.gserviceaccount.com]
student_01_932d053b64d1@cloudshell:~ (qwiklabs-gcp-03-d5fcafe3101b)$ gcloud auth activate-service-account --key-file key.json
Activated service account credentials for: [quickstart@qwiklabs-gcp-03-d5fcafe3101b.iam.gserviceaccount.com]
student_01_932d053b64d1@cloudshell:~ (qwiklabs-gcp-03-d5fcafe3101b)$ gcloud auth print-access-token
ya29.c.c0AY_VpZgnY6WHSQBYEZgeV2WX-n3DuyOrBQ75FVAJsfbzB8maQKESunDLKLrp8LjOLhOXOGH5jygmNQudE4a6O9PX7ZFiCq3ruC0R0OHlGtIFqsrdj2mX_EzYhfOQxhOYK26lXpmAI51esvu52tMG_OfUlVIJ0zrNVWTQS8wAAEkYIg55UOZSQwU1Cy0rauURiCprnQYdEjhXyAYamzoEYtUBHC16jxM8Y8UVx2OSMLWhjZV4SqQrtPv1J64HWYELUU3n4SL8heKMZip8_sxIQ--v8LP8LZv14Pki-oS6eqRQJRvpT2iJcryIh38FtbucWqra0mk0Z6WA3Wu53krDRZNgB2lFUFNNtaStItWqAD3ujHxqHxwiqM-V3y-k-hRM1oRVntQG400KVa9ZsRYOSes_lUlIV0vifkVutqoWpiiUiJ4lga8Ozw-0VzXFt-QU98J3wQosgckdSJ8bFiQbspnhluBf7vjry8zXxVfvjI94p-mk3oJw-kZJg6zWkJdsgUB5Scub8sMtyhZYn3kM-1lQ5I2fS4_4t2gfBuzgo7Ze63Y7kdupRp1p--UxB-pnZ0_j2U4Bmkfr-bpYp42-d3OtUaq0y3FkaJgf6r0af0MyF8_Mb11RZf-S_B4dS4doIBW82o8SuZmXyhFmrQXeaqd9ROx8acsBxjRW9ZjvjgVhr4eiOlBeqs5imgVyuWU2FJnSvOflIQjdrf9yaF6B8OwYmtrmckh-UowpRFIUtSgsBpS6RV-ewbbqoJh22U9-v4Ruj3dp1St6SodVfJIrbcVJk3a8qvZd2Xa9XdmdrrFfQMdQxQWS7J6bI-18Xk7i5o2siMV1au-I63Yje72g7wXukS0V24glxyMWQxy9thtiJqgVsuquhgtRt72ql76w6mv1Zc1aYtRj8bzl4wm4Uu169hJiFJ6U78UZ0to5qJvckItdzaQOh6rvomhXvpke1bn260q25zpU2XjquMh5mfrpWwS5-Ol3l5jjeYWr-6lQWpZjWSVXsIX
student_01_932d053b64d1@cloudshell:~ (qwiklabs-gcp-03-d5fcafe3101b)$ 
```

The token will print in the output, and you'll be using it in a future step.

## Make an annotate video request

**Note:** In this lab, the Cloud Video Intelligence API is already enabled for you.

1. Run this command to create a JSON request file with the following text, and save it as `request.json` :

```json
student_01_932d053b64d1@cloudshell:~ (qwiklabs-gcp-03-d5fcafe3101b)$ cat > request.json <<EOF
{
   "inputUri":"gs://spls/gsp154/video/train.mp4",
   "features": [
       "LABEL_DETECTION"
   ]
}
EOF
student_01_932d053b64d1@cloudshell:~ (qwiklabs-gcp-03-d5fcafe3101b)$ 
```

To make the process simpler, a public video of a train available to your project is used as the value for your `inputUri`. If preferred or running in a personal project, any video can be used in place by uploading it to Cloud Storage and providing its Cloud Storage  URI (format: `gs://bucket/object`) for the value of `inputUri`.

Use `curl` to make a `videos:annotate` request passing the filename of the entity request:

```sh
student_01_932d053b64d1@cloudshell:~ (qwiklabs-gcp-03-d5fcafe3101b)$ curl -s -H 'Content-Type: application/json' \
    -H 'Authorization: Bearer '$(gcloud auth print-access-token)'' \
    'https://videointelligence.googleapis.com/v1/videos:annotate' \
    -d @request.json
{
  "name": "projects/980826156684/locations/asia-east1/operations/9410431344799372719"
}
student_01_932d053b64d1@cloudshell:~ (qwiklabs-gcp-03-d5fcafe3101b)$ 
```



The Video Intelligence API creates an operation to process your request. You should now see a response that includes your operation name, which  should look similar to this one:

You will use this operation name, locations and projects in the future step.

1. Use this script to request information on the operation by calling the `v1.operations` endpoint. Replace the `PROJECTS`, `LOCATIONS` and `OPERATION_NAME` with the value you just received in the previous command:

You'll now see information related to your operation. If the operation has completed, a `done` field is included and set to `true`:

```sh
student_01_932d053b64d1@cloudshell:~ (qwiklabs-gcp-03-d5fcafe3101b)$ curl -s -H 'Content-Type: application/json'     -H 'Authorization: Bearer '$(gcloud auth print-access-token)''     'https://videointelligence.googleapis.com/v1/projects/980826156684/locations/asia-east1/operations/9410431344799372719'
{
  "name": "projects/980826156684/locations/asia-east1/operations/9410431344799372719",
  "metadata": {
    "@type": "type.googleapis.com/google.cloud.videointelligence.v1.AnnotateVideoProgress",
    "annotationProgress": [
      {
        "inputUri": "/spls/gsp154/video/train.mp4",
        "progressPercent": 100,
        "startTime": "2024-07-01T12:45:40.646895Z",
        "updateTime": "2024-07-01T12:45:48.084813Z"
      }
    ]
  },
  "done": true,
  "response": {
    "@type": "type.googleapis.com/google.cloud.videointelligence.v1.AnnotateVideoResponse",
    "annotationResults": [
      {
        "inputUri": "/spls/gsp154/video/train.mp4",
        "segmentLabelAnnotations": [
          {
            "entity": {
              "entityId": "/m/0467y7",
              "description": "passenger car",
              "languageCode": "en-US"
            },
            "categoryEntities": [
              {
                "entityId": "/m/07yv9",
                "description": "vehicle",
                "languageCode": "en-US"
              }
            ],
            "segments": [
              {
                "segment": {
                  "startTimeOffset": "0s",
                  "endTimeOffset": "12.640s"
                },
                "confidence": 0.782828
              }
            ]
          },
          {
            "entity": {
              "entityId": "/m/025s53m",
              "description": "rolling stock",
              "languageCode": "en-US"
            },
            "categoryEntities": [
              {
                "entityId": "/m/07yv9",
                "description": "vehicle",
                "languageCode": "en-US"
              }
            ],
            "segments": [
              {
                "segment": {
                  "startTimeOffset": "0s",
                  "endTimeOffset": "12.640s"
                },
                "confidence": 0.3284738
              }
            ]
          },
          {
            "entity": {
              "entityId": "/m/01vk9q",
              "description": "track",
              "languageCode": "en-US"
            },
            "segments": [
              {
                "segment": {
                  "startTimeOffset": "0s",
                  "endTimeOffset": "12.640s"
                },
                "confidence": 0.9854606
              }
            ]
          },
          {
            "entity": {
              "entityId": "/m/0db2f",
              "description": "high speed rail",
              "languageCode": "en-US"
            },
            "categoryEntities": [
              {
                "entityId": "/m/06d_3",
                "description": "rail transport",
                "languageCode": "en-US"
              }
            ],
            "segments": [
              {
                "segment": {
                  "startTimeOffset": "0s",
                  "endTimeOffset": "12.640s"
                },
                "confidence": 0.32582685
              }
            ]
          },
          {
            "entity": {
              "entityId": "/m/01g50p",
              "description": "railroad car",
              "languageCode": "en-US"
            },
            "categoryEntities": [
              {
                "entityId": "/m/07yv9",
                "description": "vehicle",
                "languageCode": "en-US"
              }
            ],
            "segments": [
              {
                "segment": {
                  "startTimeOffset": "0s",
                  "endTimeOffset": "12.640s"
                },
                "confidence": 0.9871346
              }
            ]
          },
          {
            "entity": {
              "entityId": "/m/04h5c",
              "description": "locomotive",
              "languageCode": "en-US"
            },
            "categoryEntities": [
              {
                "entityId": "/m/07yv9",
                "description": "vehicle",
                "languageCode": "en-US"
              }
            ],
            "segments": [
              {
                "segment": {
                  "startTimeOffset": "0s",
                  "endTimeOffset": "12.640s"
                },
                "confidence": 0.98347765
              }
            ]
          },
          {
            "entity": {
              "entityId": "/m/01prls",
              "description": "land vehicle",
              "languageCode": "en-US"
            },
            "categoryEntities": [
              {
                "entityId": "/m/07yv9",
                "description": "vehicle",
                "languageCode": "en-US"
              }
            ],
            "segments": [
              {
                "segment": {
                  "startTimeOffset": "0s",
                  "endTimeOffset": "12.640s"
                },
                "confidence": 0.9941471
              }
            ]
          },
          {
            "entity": {
              "entityId": "/m/0195fx",
              "description": "rapid transit",
              "languageCode": "en-US"
            },
            "categoryEntities": [
              {
                "entityId": "/m/07bsy",
                "description": "transport",
                "languageCode": "en-US"
              }
            ],
            "segments": [
              {
                "segment": {
                  "startTimeOffset": "0s",
                  "endTimeOffset": "12.640s"
                },
                "confidence": 0.8040178
              }
            ]
          },
          {
            "entity": {
              "entityId": "/m/07yv9",
              "description": "vehicle",
              "languageCode": "en-US"
            },
            "segments": [
              {
                "segment": {
                  "startTimeOffset": "0s",
                  "endTimeOffset": "12.640s"
                },
                "confidence": 0.9194523
              }
            ]
          },
          {
            "entity": {
              "entityId": "/m/0py27",
              "description": "train station",
              "languageCode": "en-US"
            },
            "categoryEntities": [
              {
                "entityId": "/m/0cgh4",
                "description": "building",
                "languageCode": "en-US"
              }
            ],
            "segments": [
              {
                "segment": {
                  "startTimeOffset": "0s",
                  "endTimeOffset": "12.640s"
                },
                "confidence": 0.7776639
              }
            ]
          },
          {
            "entity": {
              "entityId": "/m/07jdr",
              "description": "train",
              "languageCode": "en-US"
            },
            "categoryEntities": [
              {
                "entityId": "/m/07yv9",
                "description": "vehicle",
                "languageCode": "en-US"
              }
            ],
            "segments": [
              {
                "segment": {
                  "startTimeOffset": "0s",
                  "endTimeOffset": "12.640s"
                },
                "confidence": 0.99541986
              }
            ]
          },
          {
            "entity": {
              "entityId": "/m/07bsy",
              "description": "transport",
              "languageCode": "en-US"
            },
            "segments": [
              {
                "segment": {
                  "startTimeOffset": "0s",
                  "endTimeOffset": "12.640s"
                },
                "confidence": 0.98366255
              }
            ]
          },
          {
            "entity": {
              "entityId": "/m/05zdp",
              "description": "public transport",
              "languageCode": "en-US"
            },
            "categoryEntities": [
              {
                "entityId": "/m/07bsy",
                "description": "transport",
                "languageCode": "en-US"
              }
            ],
            "segments": [
              {
                "segment": {
                  "startTimeOffset": "0s",
                  "endTimeOffset": "12.640s"
                },
                "confidence": 0.9222028
              }
            ]
          },
          {
            "entity": {
              "entityId": "/m/06d_3",
              "description": "rail transport",
              "languageCode": "en-US"
            },
            "categoryEntities": [
              {
                "entityId": "/m/07bsy",
                "description": "transport",
                "languageCode": "en-US"
              }
            ],
            "segments": [
              {
                "segment": {
                  "startTimeOffset": "0s",
                  "endTimeOffset": "12.640s"
                },
                "confidence": 0.9922013
              }
            ]
          }
        ],
        "shotLabelAnnotations": [
          {
            "entity": {
              "entityId": "/m/0467y7",
              "description": "passenger car",
              "languageCode": "en-US"
            },
            "categoryEntities": [
              {
                "entityId": "/m/07yv9",
                "description": "vehicle",
                "languageCode": "en-US"
              }
            ],
            "segments": [
              {
                "segment": {
                  "startTimeOffset": "0s",
                  "endTimeOffset": "12.640s"
                },
                "confidence": 0.782828
              }
            ]
          },
          {
            "entity": {
              "entityId": "/m/025s53m",
              "description": "rolling stock",
              "languageCode": "en-US"
            },
            "categoryEntities": [
              {
                "entityId": "/m/07yv9",
                "description": "vehicle",
                "languageCode": "en-US"
              }
            ],
            "segments": [
              {
                "segment": {
                  "startTimeOffset": "0s",
                  "endTimeOffset": "12.640s"
                },
                "confidence": 0.31272778
              }
            ]
          },
          {
            "entity": {
              "entityId": "/m/01vk9q",
              "description": "track",
              "languageCode": "en-US"
            },
            "segments": [
              {
                "segment": {
                  "startTimeOffset": "0s",
                  "endTimeOffset": "12.640s"
                },
                "confidence": 0.9854606
              }
            ]
          },
          {
            "entity": {
              "entityId": "/m/0db2f",
              "description": "high speed rail",
              "languageCode": "en-US"
            },
            "categoryEntities": [
              {
                "entityId": "/m/06d_3",
                "description": "rail transport",
                "languageCode": "en-US"
              }
            ],
            "segments": [
              {
                "segment": {
                  "startTimeOffset": "0s",
                  "endTimeOffset": "12.640s"
                },
                "confidence": 0.32582685
              }
            ]
          },
          {
            "entity": {
              "entityId": "/m/01g50p",
              "description": "railroad car",
              "languageCode": "en-US"
            },
            "categoryEntities": [
              {
                "entityId": "/m/07yv9",
                "description": "vehicle",
                "languageCode": "en-US"
              }
            ],
            "segments": [
              {
                "segment": {
                  "startTimeOffset": "0s",
                  "endTimeOffset": "12.640s"
                },
                "confidence": 0.98800284
              }
            ]
          },
          {
            "entity": {
              "entityId": "/m/04h5c",
              "description": "locomotive",
              "languageCode": "en-US"
            },
            "categoryEntities": [
              {
                "entityId": "/m/07yv9",
                "description": "vehicle",
                "languageCode": "en-US"
              }
            ],
            "segments": [
              {
                "segment": {
                  "startTimeOffset": "0s",
                  "endTimeOffset": "12.640s"
                },
                "confidence": 0.98347765
              }
            ]
          },
          {
            "entity": {
              "entityId": "/m/01prls",
              "description": "land vehicle",
              "languageCode": "en-US"
            },
            "categoryEntities": [
              {
                "entityId": "/m/07yv9",
                "description": "vehicle",
                "languageCode": "en-US"
              }
            ],
            "segments": [
              {
                "segment": {
                  "startTimeOffset": "0s",
                  "endTimeOffset": "12.640s"
                },
                "confidence": 0.9941471
              }
            ]
          },
          {
            "entity": {
              "entityId": "/m/0195fx",
              "description": "rapid transit",
              "languageCode": "en-US"
            },
            "categoryEntities": [
              {
                "entityId": "/m/07bsy",
                "description": "transport",
                "languageCode": "en-US"
              }
            ],
            "segments": [
              {
                "segment": {
                  "startTimeOffset": "0s",
                  "endTimeOffset": "12.640s"
                },
                "confidence": 0.811784
              }
            ]
          },
          {
            "entity": {
              "entityId": "/m/07yv9",
              "description": "vehicle",
              "languageCode": "en-US"
            },
            "segments": [
              {
                "segment": {
                  "startTimeOffset": "0s",
                  "endTimeOffset": "12.640s"
                },
                "confidence": 0.92183924
              }
            ]
          },
          {
            "entity": {
              "entityId": "/m/0py27",
              "description": "train station",
              "languageCode": "en-US"
            },
            "categoryEntities": [
              {
                "entityId": "/m/0cgh4",
                "description": "building",
                "languageCode": "en-US"
              }
            ],
            "segments": [
              {
                "segment": {
                  "startTimeOffset": "0s",
                  "endTimeOffset": "12.640s"
                },
                "confidence": 0.7776639
              }
            ]
          },
          {
            "entity": {
              "entityId": "/m/07jdr",
              "description": "train",
              "languageCode": "en-US"
            },
            "categoryEntities": [
              {
                "entityId": "/m/07yv9",
                "description": "vehicle",
                "languageCode": "en-US"
              }
            ],
            "segments": [
              {
                "segment": {
                  "startTimeOffset": "0s",
                  "endTimeOffset": "12.640s"
                },
                "confidence": 0.99541986
              }
            ]
          },
          {
            "entity": {
              "entityId": "/m/07bsy",
              "description": "transport",
              "languageCode": "en-US"
            },
            "segments": [
              {
                "segment": {
                  "startTimeOffset": "0s",
                  "endTimeOffset": "12.640s"
                },
                "confidence": 0.98366255
              }
            ]
          },
          {
            "entity": {
              "entityId": "/m/05zdp",
              "description": "public transport",
              "languageCode": "en-US"
            },
            "categoryEntities": [
              {
                "entityId": "/m/07bsy",
                "description": "transport",
                "languageCode": "en-US"
              }
            ],
            "segments": [
              {
                "segment": {
                  "startTimeOffset": "0s",
                  "endTimeOffset": "12.640s"
                },
                "confidence": 0.92998904
              }
            ]
          },
          {
            "entity": {
              "entityId": "/m/06d_3",
              "description": "rail transport",
              "languageCode": "en-US"
            },
            "categoryEntities": [
              {
                "entityId": "/m/07bsy",
                "description": "transport",
                "languageCode": "en-US"
              }
            ],
            "segments": [
              {
                "segment": {
                  "startTimeOffset": "0s",
                  "endTimeOffset": "12.640s"
                },
                "confidence": 0.9922013
              }
            ]
          }
        ],
        "segment": {
          "startTimeOffset": "0s",
          "endTimeOffset": "12.640s"
        }
      }
    ]
  }
}
student_01_932d053b64d1@cloudshell:~ (qwiklabs-gcp-03-d5fcafe3101b)$ 
```

After giving the request some time (about a minute, typically), re-run  the command and the same request returns annotated results:

You've sent your first request to Cloud Video Intelligence API.



You sent an annotate video request to the Video Intelligence API and received results.

## History

```sh
student_01_932d053b64d1@cloudshell:~ (qwiklabs-gcp-03-d5fcafe3101b)$ history 
    1  gcloud config set project qwiklabs-gcp-03-d5fcafe3101b 
    2  gcloud iam service-accounts create quickstart
    3  gcloud iam service-accounts keys create key.json --iam-account quickstart@qwiklabs-gcp-03-d5fcafe3101b.iam.gserviceaccount.com
    4  gcloud auth activate-service-account --key-file key.json
    5  gcloud auth print-access-token
    6  cat > request.json <<EOF
{
   "inputUri":"gs://spls/gsp154/video/train.mp4",
   "features": [
       "LABEL_DETECTION"
   ]
}
EOF

    7  curl -s -H 'Content-Type: application/json'     -H 'Authorization: Bearer '$(gcloud auth print-access-token)''     'https://videointelligence.googleapis.com/v1/videos:annotate'     -d @request.json
    
    8  curl -s -H 'Content-Type: application/json'     -H 'Authorization: Bearer '$(gcloud auth print-access-token)''     'https://videointelligence.googleapis.com/v1/projects/980826156684/locations/asia-east1/operations/9410431344799372719'
   9  history 
student_01_932d053b64d1@cloudshell:~ (qwiklabs-gcp-03-d5fcafe3101b)$ 
```

