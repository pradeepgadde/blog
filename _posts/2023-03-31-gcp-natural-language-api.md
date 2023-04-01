---

layout: single
title:  "Entity and Sentiment Analysis with the Natural Language API "
date:   2023-03-31 03:59:04 +0530
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

# Entity and Sentiment Analysis with the Natural Language API

The Cloud Natural Language API lets you extract entities from text,  perform sentiment and syntactic analysis, and classify text into  categories.



- Creating a Natural Language API request and calling the API with curl

- Extracting entities and running sentiment analysis on text with the Natural Language API

- Performing linguistic analysis on text with the Natural Language API

- Creating a Natural Language API request in a different language

## 1. Create an API key

  ![]({{ site.url }}{{ site.baseurl }}/assets/images/gcp-434.png)

## 2. Make an entity analysis request

  ```sh
  student-04-5e2e787da110@linux-instance:~$ cat request.json 
  {
    "document":{
      "type":"PLAIN_TEXT",
      "content":"Joanne Rowling, who writes under the pen names J. K. Rowling and Robert Galbraith, is a British novelist and screenwriter who wrote the Harry Potter fantasy series."
    },
    "encodingType":"UTF8"
  }
  student-04-5e2e787da110@linux-instance:~$ 
  ```



## 3. Call the Natural Language API

```sh
student-04-5e2e787da110@linux-instance:~$ curl "https://language.googleapis.com/v1/documents:analyzeEntities?key=${API_KEY}" \
  -s -X POST -H "Content-Type: application/json" --data-binary @request.json > result.json
student-04-5e2e787da110@linux-instance:~$ cat result.json 
{
  "entities": [
    {
      "name": "Joanne Rowling",
      "type": "PERSON",
      "metadata": {
        "wikipedia_url": "https://en.wikipedia.org/wiki/J._K._Rowling",
        "mid": "/m/042xh"
      },
      "salience": 0.79828626,
      "mentions": [
        {
          "text": {
            "content": "Joanne Rowling",
            "beginOffset": 0
          },
          "type": "PROPER"
        },
        {
          "text": {
            "content": "Rowling",
            "beginOffset": 53
          },
          "type": "PROPER"
        },
        {
          "text": {
            "content": "novelist",
            "beginOffset": 96
          },
          "type": "COMMON"
        },
        {
          "text": {
            "content": "Robert Galbraith",
            "beginOffset": 65
          },
          "type": "PROPER"
        }
      ]
    },
    {
      "name": "pen names",
      "type": "OTHER",
      "metadata": {},
      "salience": 0.07300248,
      "mentions": [
        {
          "text": {
            "content": "pen names",
            "beginOffset": 37
          },
          "type": "COMMON"
        }
      ]
    },
    {
      "name": "J.K.",
      "type": "PERSON",
      "metadata": {},
      "salience": 0.043804582,
      "mentions": [
        {
          "text": {
            "content": "J. K.",
            "beginOffset": 47
          },
          "type": "PROPER"
        }
      ]
    },
    {
      "name": "British",
      "type": "LOCATION",
      "metadata": {
        "mid": "/m/07ssc",
        "wikipedia_url": "https://en.wikipedia.org/wiki/United_Kingdom"
      },
      "salience": 0.019752095,
      "mentions": [
        {
          "text": {
            "content": "British",
            "beginOffset": 88
          },
          "type": "PROPER"
        }
      ]
    },
    {
      "name": "fantasy series",
      "type": "WORK_OF_ART",
      "metadata": {},
      "salience": 0.01764168,
      "mentions": [
        {
          "text": {
            "content": "fantasy series",
            "beginOffset": 149
          },
          "type": "COMMON"
        }
      ]
    },
    {
      "name": "Harry Potter",
      "type": "WORK_OF_ART",
      "metadata": {
        "mid": "/m/078ffw",
        "wikipedia_url": "https://en.wikipedia.org/wiki/Harry_Potter"
      },
      "salience": 0.014916742,
      "mentions": [
        {
          "text": {
            "content": "Harry Potter",
            "beginOffset": 136
          },
          "type": "PROPER"
        }
      ]
    },
    {
      "name": "screenwriter",
      "type": "PERSON",
      "metadata": {},
      "salience": 0.011085264,
      "mentions": [
        {
          "text": {
            "content": "screenwriter",
            "beginOffset": 109
          },
          "type": "COMMON"
        }
      ]
    }
  ],
  "language": "en"
}
student-04-5e2e787da110@linux-instance:~$ 
```



## 4. Sentiment analysis with the Natural Language API

```sh
student-04-5e2e787da110@linux-instance:~$ cat request.json 
 {
  "document":{
    "type":"PLAIN_TEXT",
    "content":"Harry Potter is the best book. I think everyone should read it."
  },
  "encodingType": "UTF8"
}
student-04-5e2e787da110@linux-instance:~$ curl "https://language.googleapis.com/v1/documents:analyzeSentiment?key=${API_KEY}" \
  -s -X POST -H "Content-Type: application/json" --data-binary @request.json
{
  "documentSentiment": {
    "magnitude": 1.9,
    "score": 0.9
  },
  "language": "en",
  "sentences": [
    {
      "text": {
        "content": "Harry Potter is the best book.",
        "beginOffset": 0
      },
      "sentiment": {
        "magnitude": 0.9,
        "score": 0.9
      }
    },
    {
      "text": {
        "content": "I think everyone should read it.",
        "beginOffset": 31
      },
      "sentiment": {
        "magnitude": 0.9,
        "score": 0.9
      }
    }
  ]
}
student-04-5e2e787da110@linux-instance:~$ 
```



## 5. Analyzing entity sentiment
```sh
student-04-5e2e787da110@linux-instance:~$ cat request.json 
 {
  "document":{
    "type":"PLAIN_TEXT",
    "content":"I liked the sushi but the service was terrible."
  },
  "encodingType": "UTF8"
}
student-04-5e2e787da110@linux-instance:~$ curl "https://language.googleapis.com/v1/documents:analyzeEntitySentiment?key=${API_KEY}" \
  -s -X POST -H "Content-Type: application/json" --data-binary @request.json
{
  "entities": [
    {
      "name": "sushi",
      "type": "CONSUMER_GOOD",
      "metadata": {},
      "salience": 0.51064336,
      "mentions": [
        {
          "text": {
            "content": "sushi",
            "beginOffset": 12
          },
          "type": "COMMON",
          "sentiment": {
            "magnitude": 0,
            "score": 0
          }
        }
      ],
      "sentiment": {
        "magnitude": 0,
        "score": 0
      }
    },
    {
      "name": "service",
      "type": "OTHER",
      "metadata": {},
      "salience": 0.48935664,
      "mentions": [
        {
          "text": {
            "content": "service",
            "beginOffset": 26
          },
          "type": "COMMON",
          "sentiment": {
            "magnitude": 0.7,
            "score": -0.7
          }
        }
      ],
      "sentiment": {
        "magnitude": 0.7,
        "score": -0.7
      }
    }
  ],
  "language": "en"
}
student-04-5e2e787da110@linux-instance:~$ 
```
## 6. Analyzing syntax and parts of speech
```sh
student-04-5e2e787da110@linux-instance:~$ cat request.json 
{
  "document":{
    "type":"PLAIN_TEXT",
    "content": "Joanne Rowling is a British novelist, screenwriter and film producer."
  },
  "encodingType": "UTF8"
}
student-04-5e2e787da110@linux-instance:~$ curl "https://language.googleapis.com/v1/documents:analyzeSyntax?key=${API_KEY}" \
  -s -X POST -H "Content-Type: application/json" --data-binary @request.json
{
  "sentences": [
    {
      "text": {
        "content": "Joanne Rowling is a British novelist, screenwriter and film producer.",
        "beginOffset": 0
      }
    }
  ],
  "tokens": [
    {
      "text": {
        "content": "Joanne",
        "beginOffset": 0
      },
      "partOfSpeech": {
        "tag": "NOUN",
        "aspect": "ASPECT_UNKNOWN",
        "case": "CASE_UNKNOWN",
        "form": "FORM_UNKNOWN",
        "gender": "GENDER_UNKNOWN",
        "mood": "MOOD_UNKNOWN",
        "number": "SINGULAR",
        "person": "PERSON_UNKNOWN",
        "proper": "PROPER",
        "reciprocity": "RECIPROCITY_UNKNOWN",
        "tense": "TENSE_UNKNOWN",
        "voice": "VOICE_UNKNOWN"
      },
      "dependencyEdge": {
        "headTokenIndex": 1,
        "label": "NN"
      },
      "lemma": "Joanne"
    },
    {
      "text": {
        "content": "Rowling",
        "beginOffset": 7
      },
      "partOfSpeech": {
        "tag": "NOUN",
        "aspect": "ASPECT_UNKNOWN",
        "case": "CASE_UNKNOWN",
        "form": "FORM_UNKNOWN",
        "gender": "GENDER_UNKNOWN",
        "mood": "MOOD_UNKNOWN",
        "number": "SINGULAR",
        "person": "PERSON_UNKNOWN",
        "proper": "PROPER",
        "reciprocity": "RECIPROCITY_UNKNOWN",
        "tense": "TENSE_UNKNOWN",
        "voice": "VOICE_UNKNOWN"
      },
      "dependencyEdge": {
        "headTokenIndex": 2,
        "label": "NSUBJ"
      },
      "lemma": "Rowling"
    },
    {
      "text": {
        "content": "is",
        "beginOffset": 15
      },
      "partOfSpeech": {
        "tag": "VERB",
        "aspect": "ASPECT_UNKNOWN",
        "case": "CASE_UNKNOWN",
        "form": "FORM_UNKNOWN",
        "gender": "GENDER_UNKNOWN",
        "mood": "INDICATIVE",
        "number": "SINGULAR",
        "person": "THIRD",
        "proper": "PROPER_UNKNOWN",
        "reciprocity": "RECIPROCITY_UNKNOWN",
        "tense": "PRESENT",
        "voice": "VOICE_UNKNOWN"
      },
      "dependencyEdge": {
        "headTokenIndex": 2,
        "label": "ROOT"
      },
      "lemma": "be"
    },
    {
      "text": {
        "content": "a",
        "beginOffset": 18
      },
      "partOfSpeech": {
        "tag": "DET",
        "aspect": "ASPECT_UNKNOWN",
        "case": "CASE_UNKNOWN",
        "form": "FORM_UNKNOWN",
        "gender": "GENDER_UNKNOWN",
        "mood": "MOOD_UNKNOWN",
        "number": "NUMBER_UNKNOWN",
        "person": "PERSON_UNKNOWN",
        "proper": "PROPER_UNKNOWN",
        "reciprocity": "RECIPROCITY_UNKNOWN",
        "tense": "TENSE_UNKNOWN",
        "voice": "VOICE_UNKNOWN"
      },
      "dependencyEdge": {
        "headTokenIndex": 5,
        "label": "DET"
      },
      "lemma": "a"
    },
    {
      "text": {
        "content": "British",
        "beginOffset": 20
      },
      "partOfSpeech": {
        "tag": "ADJ",
        "aspect": "ASPECT_UNKNOWN",
        "case": "CASE_UNKNOWN",
        "form": "FORM_UNKNOWN",
        "gender": "GENDER_UNKNOWN",
        "mood": "MOOD_UNKNOWN",
        "number": "NUMBER_UNKNOWN",
        "person": "PERSON_UNKNOWN",
        "proper": "PROPER_UNKNOWN",
        "reciprocity": "RECIPROCITY_UNKNOWN",
        "tense": "TENSE_UNKNOWN",
        "voice": "VOICE_UNKNOWN"
      },
      "dependencyEdge": {
        "headTokenIndex": 5,
        "label": "AMOD"
      },
      "lemma": "British"
    },
    {
      "text": {
        "content": "novelist",
        "beginOffset": 28
      },
      "partOfSpeech": {
        "tag": "NOUN",
        "aspect": "ASPECT_UNKNOWN",
        "case": "CASE_UNKNOWN",
        "form": "FORM_UNKNOWN",
        "gender": "GENDER_UNKNOWN",
        "mood": "MOOD_UNKNOWN",
        "number": "SINGULAR",
        "person": "PERSON_UNKNOWN",
        "proper": "PROPER_UNKNOWN",
        "reciprocity": "RECIPROCITY_UNKNOWN",
        "tense": "TENSE_UNKNOWN",
        "voice": "VOICE_UNKNOWN"
      },
      "dependencyEdge": {
        "headTokenIndex": 2,
        "label": "ATTR"
      },
      "lemma": "novelist"
    },
    {
      "text": {
        "content": ",",
        "beginOffset": 36
      },
      "partOfSpeech": {
        "tag": "PUNCT",
        "aspect": "ASPECT_UNKNOWN",
        "case": "CASE_UNKNOWN",
        "form": "FORM_UNKNOWN",
        "gender": "GENDER_UNKNOWN",
        "mood": "MOOD_UNKNOWN",
        "number": "NUMBER_UNKNOWN",
        "person": "PERSON_UNKNOWN",
        "proper": "PROPER_UNKNOWN",
        "reciprocity": "RECIPROCITY_UNKNOWN",
        "tense": "TENSE_UNKNOWN",
        "voice": "VOICE_UNKNOWN"
      },
      "dependencyEdge": {
        "headTokenIndex": 5,
        "label": "P"
      },
      "lemma": ","
    },
    {
      "text": {
        "content": "screenwriter",
        "beginOffset": 38
      },
      "partOfSpeech": {
        "tag": "NOUN",
        "aspect": "ASPECT_UNKNOWN",
        "case": "CASE_UNKNOWN",
        "form": "FORM_UNKNOWN",
        "gender": "GENDER_UNKNOWN",
        "mood": "MOOD_UNKNOWN",
        "number": "SINGULAR",
        "person": "PERSON_UNKNOWN",
        "proper": "PROPER_UNKNOWN",
        "reciprocity": "RECIPROCITY_UNKNOWN",
        "tense": "TENSE_UNKNOWN",
        "voice": "VOICE_UNKNOWN"
      },
      "dependencyEdge": {
        "headTokenIndex": 5,
        "label": "CONJ"
      },
      "lemma": "screenwriter"
    },
    {
      "text": {
        "content": "and",
        "beginOffset": 51
      },
      "partOfSpeech": {
        "tag": "CONJ",
        "aspect": "ASPECT_UNKNOWN",
        "case": "CASE_UNKNOWN",
        "form": "FORM_UNKNOWN",
        "gender": "GENDER_UNKNOWN",
        "mood": "MOOD_UNKNOWN",
        "number": "NUMBER_UNKNOWN",
        "person": "PERSON_UNKNOWN",
        "proper": "PROPER_UNKNOWN",
        "reciprocity": "RECIPROCITY_UNKNOWN",
        "tense": "TENSE_UNKNOWN",
        "voice": "VOICE_UNKNOWN"
      },
      "dependencyEdge": {
        "headTokenIndex": 5,
        "label": "CC"
      },
      "lemma": "and"
    },
    {
      "text": {
        "content": "film",
        "beginOffset": 55
      },
      "partOfSpeech": {
        "tag": "NOUN",
        "aspect": "ASPECT_UNKNOWN",
        "case": "CASE_UNKNOWN",
        "form": "FORM_UNKNOWN",
        "gender": "GENDER_UNKNOWN",
        "mood": "MOOD_UNKNOWN",
        "number": "SINGULAR",
        "person": "PERSON_UNKNOWN",
        "proper": "PROPER_UNKNOWN",
        "reciprocity": "RECIPROCITY_UNKNOWN",
        "tense": "TENSE_UNKNOWN",
        "voice": "VOICE_UNKNOWN"
      },
      "dependencyEdge": {
        "headTokenIndex": 10,
        "label": "NN"
      },
      "lemma": "film"
    },
    {
      "text": {
        "content": "producer",
        "beginOffset": 60
      },
      "partOfSpeech": {
        "tag": "NOUN",
        "aspect": "ASPECT_UNKNOWN",
        "case": "CASE_UNKNOWN",
        "form": "FORM_UNKNOWN",
        "gender": "GENDER_UNKNOWN",
        "mood": "MOOD_UNKNOWN",
        "number": "SINGULAR",
        "person": "PERSON_UNKNOWN",
        "proper": "PROPER_UNKNOWN",
        "reciprocity": "RECIPROCITY_UNKNOWN",
        "tense": "TENSE_UNKNOWN",
        "voice": "VOICE_UNKNOWN"
      },
      "dependencyEdge": {
        "headTokenIndex": 5,
        "label": "CONJ"
      },
      "lemma": "producer"
    },
    {
      "text": {
        "content": ".",
        "beginOffset": 68
      },
      "partOfSpeech": {
        "tag": "PUNCT",
        "aspect": "ASPECT_UNKNOWN",
        "case": "CASE_UNKNOWN",
        "form": "FORM_UNKNOWN",
        "gender": "GENDER_UNKNOWN",
        "mood": "MOOD_UNKNOWN",
        "number": "NUMBER_UNKNOWN",
        "person": "PERSON_UNKNOWN",
        "proper": "PROPER_UNKNOWN",
        "reciprocity": "RECIPROCITY_UNKNOWN",
        "tense": "TENSE_UNKNOWN",
        "voice": "VOICE_UNKNOWN"
      },
      "dependencyEdge": {
        "headTokenIndex": 2,
        "label": "P"
      },
      "lemma": "."
    }
  ],
  "language": "en"
}
student-04-5e2e787da110@linux-instance:~$ 
```

## 7. Multilingual natural language processing
```sh
student-04-5e2e787da110@linux-instance:~$ cat request.json 
{
  "document":{
    "type":"PLAIN_TEXT",
    "content":"日本のグーグルのオフィスは、東京の六本木ヒルズにあります"
  }
}
student-04-5e2e787da110@linux-instance:~$ curl "https://language.googleapis.com/v1/documents:analyzeEntities?key=${API_KEY}" \
  -s -X POST -H "Content-Type: application/json" --data-binary @request.json
{
  "entities": [
    {
      "name": "日本",
      "type": "LOCATION",
      "metadata": {
        "wikipedia_url": "https://en.wikipedia.org/wiki/Japan",
        "mid": "/m/03_3d"
      },
      "salience": 0.23804513,
      "mentions": [
        {
          "text": {
            "content": "日本",
            "beginOffset": -1
          },
          "type": "PROPER"
        }
      ]
    },
    {
      "name": "グーグル",
      "type": "ORGANIZATION",
      "metadata": {
        "wikipedia_url": "https://en.wikipedia.org/wiki/Google",
        "mid": "/m/045c7b"
      },
      "salience": 0.21214141,
      "mentions": [
        {
          "text": {
            "content": "グーグル",
            "beginOffset": -1
          },
          "type": "PROPER"
        }
      ]
    },
    {
      "name": "六本木ヒルズ",
      "type": "PERSON",
      "metadata": {
        "wikipedia_url": "https://en.wikipedia.org/wiki/Roppongi_Hills",
        "mid": "/m/01r2_k"
      },
      "salience": 0.19418614,
      "mentions": [
        {
          "text": {
            "content": "六本木ヒルズ",
            "beginOffset": -1
          },
          "type": "PROPER"
        }
      ]
    },
    {
      "name": "東京",
      "type": "LOCATION",
      "metadata": {
        "wikipedia_url": "https://de.wikipedia.org/wiki/Tokio",
        "mid": "/g/12lnhn10f"
      },
      "salience": 0.18159479,
      "mentions": [
        {
          "text": {
            "content": "東京",
            "beginOffset": -1
          },
          "type": "PROPER"
        }
      ]
    },
    {
      "name": "オフィス",
      "type": "OTHER",
      "metadata": {},
      "salience": 0.17403255,
      "mentions": [
        {
          "text": {
            "content": "オフィス",
            "beginOffset": -1
          },
          "type": "COMMON"
        }
      ]
    }
  ],
  "language": "ja"
}
student-04-5e2e787da110@linux-instance:~$ 
```



We've learned how to perform text analysis with the Cloud Natural  Language API by extracting entities, analyzing sentiment, and doing  syntax annotation. 
