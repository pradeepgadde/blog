---

layout: single
title:  "Vertex AI PaLM API: Qwik Start"
categories: Cloud
tags: GCP
classes: wide
show_date: true
header:
  overlay_image: /assets/images/Google_Gemini_logo.png
  og_image: /assets/images/Google_Gemini_logo.png
  teaser: /assets/images/gemini.webp
author:
  name     : "Gemini"
  avatar   : "/assets/images/gemini.webp"

sidebar:
  - title: "Topics"
    nav: my-sidebar

---

# Vertex AI PaLM API: Qwik Start

The Vertex AI PaLM API enables you to test, customize, and deploy instances of Google's PaLM large language models (LLM) so that you can leverage the capabilities of PaLM in your applications. The PaLM family of models supports text completion and multi-turn chat.

 how to use the PaLM family of models to suport:

- Text completion use cases
- Multi-turn chat integration

### Enable the Vertex AI API

1. In the Google Cloud Console, enter **Vertex AI API** in the top search bar.
2. Click on the result for **Vertex AI API** under Marketplace.
3. Click **Enable**.

## Sample text prompts

You will perform the tasks in this lab using curl within Cloud Shell.

## Sample chat prompts

For chat API calls, the `context`, `examples`, and `messages` combine to form the prompt. The following table shows the parameters  that you need to configure for the Vertex AI PaLM API for text:

```sh

Welcome to Cloud Shell! Type "help" to get started.
Your Cloud Platform project in this session is set to qwiklabs-gcp-03-f1d99a2bd4d4.
Use “gcloud config set project [PROJECT_ID]” to change to a different project.
student_01_e721493cd4a9@cloudshell:~ (qwiklabs-gcp-03-f1d99a2bd4d4)$ MODEL_ID="text-bison"
PROJECT_ID=$DEVSHELL_PROJECT_ID

curl \
-X POST \
-H "Authorization: Bearer $(gcloud auth print-access-token)" \
-H "Content-Type: application/json" \
https://us-central1-aiplatform.googleapis.com/v1/projects/${PROJECT_ID}/locations/us-central1/publishers/google/models/${MODEL_ID}:predict -d \
$'{
  "instances": [
    { "prompt": "Provide a summary with about two sentences for the following article:
The efficient-market hypothesis (EMH) is a hypothesis in financial \
economics that states that asset prices reflect all available \
information. A direct implication is that it is impossible to \
}'} "topP": 0.95kens": 256,nyam, and Tong 2014; McLean and Pontiff \\\et \
{
  "predictions": [
    {
      "citationMetadata": {
        "citations": []
      },
      "safetyAttributes": {
        "scores": [
          1,
          0.1,
          0.1,
          0.1
        ],
        "safetyRatings": [
          {
            "probabilityScore": 0.2,
            "severityScore": 0,
            "category": "Dangerous Content",
            "severity": "NEGLIGIBLE"
          },
          {
            "severityScore": 0.1,
            "probabilityScore": 0.1,
            "severity": "NEGLIGIBLE",
            "category": "Harassment"
          },
          {
            "probabilityScore": 0,
            "severity": "NEGLIGIBLE",
            "severityScore": 0,
            "category": "Hate Speech"
          },
          {
            "category": "Sexually Explicit",
            "severityScore": 0.1,
            "probabilityScore": 0.1,
            "severity": "NEGLIGIBLE"
          }
        ],
        "blocked": false,
        "categories": [
          "Finance",
          "Insult",
          "Legal",
          "Sexual"
        ]
      },
      "content": " The efficient-market hypothesis (EMH) states that asset prices reflect all available information, making it impossible to consistently outperform the market on a risk-adjusted basis. While some studies have found evidence of market anomalies or deviations from specific risk models, recent research suggests that return predictability has become more elusive due to factors such as out-of-sample failure and advancements in trading technology."
    }
  ],
  "metadata": {
    "tokenMetadata": {
      "inputTokenCount": {
        "totalBillableCharacters": 1579,
        "totalTokens": 500
      },
      "outputTokenCount": {
        "totalBillableCharacters": 383,
        "totalTokens": 76
      }
    }
  }
}
student_01_e721493cd4a9@cloudshell:~ (qwiklabs-gcp-03-f1d99a2bd4d4)$ MODEL_ID="text-bison"
PROJECT_ID=$DEVSHELL_PROJECT_ID

curl \
-X POST \
-H "Authorization: Bearer $(gcloud auth print-access-token)" \
-H "Content-Type: application/json" \
https://us-central1-aiplatform.googleapis.com/v1/projects/${PROJECT_ID}/locations/us-central1/publishers/google/models/${MODEL_ID}:predict -d \
$'{
  "instances": [
    { "prompt": "Give me ten interview questions for the role of program manager."}
  ],
  "parameters": {
    "temperature": 0.2,
}'} "topP": 0.8okens": 1024,
{
  "predictions": [
    {
      "safetyAttributes": {
        "blocked": false,
        "scores": [
          0.1,
          0.3,
          0.1,
          0.2
        ],
        "categories": [
          "Derogatory",
          "Finance",
          "Insult",
          "Profanity"
        ],
        "safetyRatings": [
          {
            "probabilityScore": 0,
            "severityScore": 0.1,
            "category": "Dangerous Content",
            "severity": "NEGLIGIBLE"
          },
          {
            "severityScore": 0,
            "severity": "NEGLIGIBLE",
            "probabilityScore": 0.1,
            "category": "Harassment"
          },
          {
            "severity": "NEGLIGIBLE",
            "severityScore": 0.1,
            "probabilityScore": 0.1,
            "category": "Hate Speech"
          },
          {
            "category": "Sexually Explicit",
            "severityScore": 0,
            "probabilityScore": 0,
            "severity": "NEGLIGIBLE"
          }
        ]
      },
      "citationMetadata": {
        "citations": []
      },
      "content": " 1. **Tell me about your experience managing programs.**\n2. **What are your strengths and weaknesses as a program manager?**\n3. **What do you think are the most important qualities for a successful program manager?**\n4. **How do you manage stakeholder expectations?**\n5. **How do you handle conflict and difficult situations?**\n6. **What are your strategies for managing risk and uncertainty?**\n7. **How do you measure and evaluate the success of a program?**\n8. **What are your thoughts on Agile project management?**\n9. **What are your salary expectations?**\n10. **Why are you interested in this role?**"
    }
  ],
  "metadata": {
    "tokenMetadata": {
      "outputTokenCount": {
        "totalBillableCharacters": 510,
        "totalTokens": 145
      },
      "inputTokenCount": {
        "totalBillableCharacters": 54,
        "totalTokens": 12
      }
    }
  }
}
student_01_e721493cd4a9@cloudshell:~ (qwiklabs-gcp-03-f1d99a2bd4d4)$ MODEL_ID="text-bison"
PROJECT_ID=$DEVSHELL_PROJECT_ID
curl \
-X POST \
-H "Authorization: Bearer $(gcloud auth print-access-token)" \
-H "Content-Type: application/json" \
https://us-central1-aiplatform.googleapis.com/v1/projects/${PROJECT_ID}/locations/us-central1/publishers/google/models/${MODEL_ID}:predict -d \
$'{
  "instances": [
    { "prompt": "Provide a summary with about two sentences for the following article:
The efficient-market hypothesis (EMH) is a hypothesis in financial \
economics that states that asset prices reflect all available \
information. A direct implication is that it is impossible to \
\\"beat the market\\" consistently on a risk-adjusted basis since market \
}' > summarization_prompt_example.txtTong 2014; McLean and Pontiff \\\
  % Total    % Received % Xferd  Average Speed   Time    Time     Time  Current
                                 Dload  Upload   Total   Spent    Left  Speed
100  3889    0  1814  100  2075   1718   1965  0:00:01  0:00:01 --:--:--  3686
student_01_e721493cd4a9@cloudshell:~ (qwiklabs-gcp-03-f1d99a2bd4d4)$ export PROJECT_ID=$(gcloud config get-value project)
gsutil cp *.txt gs://$PROJECT_ID
Your active configuration is: [cloudshell-28891]
Copying file://README-cloudshell.txt [Content-Type=text/plain]...
Copying file://summarization_prompt_example.txt [Content-Type=text/plain]...    
- [2 files][  2.7 KiB/  2.7 KiB]                                                
Operation completed over 2 objects/2.7 KiB.                                      
student_01_e721493cd4a9@cloudshell:~ (qwiklabs-gcp-03-f1d99a2bd4d4)$ history 
    1  MODEL_ID="text-bison"
    2  PROJECT_ID=$DEVSHELL_PROJECT_ID
    3  curl -X POST -H "Authorization: Bearer $(gcloud auth print-access-token)" -H "Content-Type: application/json" https://us-central1-aiplatform.googleapis.com/v1/projects/${PROJECT_ID}/locations/us-central1/publishers/google/models/${MODEL_ID}:predict -d $'{
  "instances": [
    { "prompt": "Provide a summary with about two sentences for the following article:
The efficient-market hypothesis (EMH) is a hypothesis in financial \
economics that states that asset prices reflect all available \
information. A direct implication is that it is impossible to \
\\"beat the market\\" consistently on a risk-adjusted basis since market \
prices should only react to new information. Because the EMH is \
formulated in terms of risk adjustment, it only makes testable \
predictions when coupled with a particular model of risk. As a \
result, research in financial economics since at least the 1990s has \
focused on market anomalies, that is, deviations from specific \
models of risk. The idea that financial market returns are difficult \
to predict goes back to Bachelier, Mandelbrot, and Samuelson, but \
is closely associated with Eugene Fama, in part due to his \
influential 1970 review of the theoretical and empirical research. \
The EMH provides the basic logic for modern risk-based theories of \
asset prices, and frameworks such as consumption-based asset pricing \
and intermediary asset pricing can be thought of as the combination \
of a model of risk with the EMH. Many decades of empirical research \
on return predictability has found mixed evidence. Research in the \
1950s and 1960s often found a lack of predictability (e.g. Ball and \
Brown 1968; Fama, Fisher, Jensen, and Roll 1969), yet the \
1980s-2000s saw an explosion of discovered return predictors (e.g. \
Rosenberg, Reid, and Lanstein 1985; Campbell and Shiller 1988; \
Jegadeesh and Titman 1993). Since the 2010s, studies have often \
found that return predictability has become more elusive, as \
predictability fails to work out-of-sample (Goyal and Welch 2008), \
or has been weakened by advances in trading technology and investor \
learning (Chordia, Subrahmanyam, and Tong 2014; McLean and Pontiff \
2016; Martineau 2021).
Summary:
"}
  ],
  "parameters": {
    "temperature": 0.2,
    "maxOutputTokens": 256,
    "topK": 40,
    "topP": 0.95
  }
}'
    4  MODEL_ID="text-bison"
    5  PROJECT_ID=$DEVSHELL_PROJECT_ID
    6  curl -X POST -H "Authorization: Bearer $(gcloud auth print-access-token)" -H "Content-Type: application/json" https://us-central1-aiplatform.googleapis.com/v1/projects/${PROJECT_ID}/locations/us-central1/publishers/google/models/${MODEL_ID}:predict -d $'{
  "instances": [
    { "prompt": "Give me ten interview questions for the role of program manager."}
  ],
  "parameters": {
    "temperature": 0.2,
    "maxOutputTokens": 1024,
    "topK": 40,
    "topP": 0.8
  }
}'
    7  MODEL_ID="text-bison"
    8  PROJECT_ID=$DEVSHELL_PROJECT_ID
    9  curl -X POST -H "Authorization: Bearer $(gcloud auth print-access-token)" -H "Content-Type: application/json" https://us-central1-aiplatform.googleapis.com/v1/projects/${PROJECT_ID}/locations/us-central1/publishers/google/models/${MODEL_ID}:predict -d $'{
  "instances": [
    { "prompt": "Provide a summary with about two sentences for the following article:
The efficient-market hypothesis (EMH) is a hypothesis in financial \
economics that states that asset prices reflect all available \
information. A direct implication is that it is impossible to \
\\"beat the market\\" consistently on a risk-adjusted basis since market \
prices should only react to new information. Because the EMH is \
formulated in terms of risk adjustment, it only makes testable \
predictions when coupled with a particular model of risk. As a \
result, research in financial economics since at least the 1990s has \
focused on market anomalies, that is, deviations from specific \
models of risk. The idea that financial market returns are difficult \
to predict goes back to Bachelier, Mandelbrot, and Samuelson, but \
is closely associated with Eugene Fama, in part due to his \
influential 1970 review of the theoretical and empirical research. \
The EMH provides the basic logic for modern risk-based theories of \
asset prices, and frameworks such as consumption-based asset pricing \
and intermediary asset pricing can be thought of as the combination \
of a model of risk with the EMH. Many decades of empirical research \
on return predictability has found mixed evidence. Research in the \
1950s and 1960s often found a lack of predictability (e.g. Ball and \
Brown 1968; Fama, Fisher, Jensen, and Roll 1969), yet the \
1980s-2000s saw an explosion of discovered return predictors (e.g. \
Rosenberg, Reid, and Lanstein 1985; Campbell and Shiller 1988; \
Jegadeesh and Titman 1993). Since the 2010s, studies have often \
found that return predictability has become more elusive, as \
predictability fails to work out-of-sample (Goyal and Welch 2008), \
or has been weakened by advances in trading technology and investor \
learning (Chordia, Subrahmanyam, and Tong 2014; McLean and Pontiff \
2016; Martineau 2021).
Summary:
"}
  ],
  "parameters": {
    "temperature": 0.2,
    "maxOutputTokens": 256,
    "topK": 40,
    "topP": 0.95
  }
}' > summarization_prompt_example.txt
   10  export PROJECT_ID=$(gcloud config get-value project)
   11  gsutil cp *.txt gs://$PROJECT_ID
   12  history 
student_01_e721493cd4a9@cloudshell:~ (qwiklabs-gcp-03-f1d99a2bd4d4)$ MODEL_ID="chat-bison"
PROJECT_ID=$DEVSHELL_PROJECT_ID

curl \
-X POST \
-H "Authorization: Bearer $(gcloud auth print-access-token)" \
-H "Content-Type: application/json" \
https://us-central1-aiplatform.googleapis.com/v1/projects/${PROJECT_ID}/locations/us-central1/publishers/google/models/${MODEL_ID}:predict -d \
'{
  "instances": [{
      "context":  "My name is Ned. You are my personal assistant. My favorite movies are Lord of the Rings and Hobbit.",
      "examples": [ { 
          "input": {"content": "Who do you work for?"},
          "output": {"content": "I work for Ned."}
}'} "topK": 408,eps": 200,my favorite movies based on a book series?",
{
  "predictions": [
    {
      "citationMetadata": [
        {
          "citations": []
        }
      ],
      "candidates": [
        {
          "content": " Yes, both Lord of the Rings and Hobbit are based on the book series written by J.R.R. Tolkien.",
          "author": "1"
        }
      ],
      "groundingMetadata": [
        {}
      ],
      "safetyAttributes": [
        {
          "safetyRatings": [
            {
              "category": "Dangerous Content",
              "severityScore": 0,
              "severity": "NEGLIGIBLE",
              "probabilityScore": 0
            },
            {
              "severity": "NEGLIGIBLE",
              "severityScore": 0,
              "probabilityScore": 0.1,
              "category": "Harassment"
            },
            {
              "severityScore": 0.1,
              "severity": "NEGLIGIBLE",
              "probabilityScore": 0.1,
              "category": "Hate Speech"
            },
            {
              "probabilityScore": 0.1,
              "severity": "NEGLIGIBLE",
              "category": "Sexually Explicit",
              "severityScore": 0
            }
          ],
          "scores": [
            0.1,
            0.1,
            0.2,
            0.1
          ],
          "blocked": false,
          "categories": [
            "Derogatory",
            "Insult",
            "Religion & Belief",
            "Sexual"
          ]
        }
      ]
    }
  ],
  "metadata": {
    "tokenMetadata": {
      "outputTokenCount": {
        "totalBillableCharacters": 77,
        "totalTokens": 25
      },
      "inputTokenCount": {
        "totalBillableCharacters": 182,
        "totalTokens": 53
      }
    }
  }
}
student_01_e721493cd4a9@cloudshell:~ (qwiklabs-gcp-03-f1d99a2bd4d4)$ MODEL_ID="chat-bison"
PROJECT_ID=$DEVSHELL_PROJECT_ID
curl \
-X POST \
-H "Authorization: Bearer $(gcloud auth print-access-token)" \
-H "Content-Type: application/json" \
https://us-central1-aiplatform.googleapis.com/v1/projects/${PROJECT_ID}/locations/us-central1/publishers/google/models/${MODEL_ID}:predict -d \
'{
  "instances": [{
      "context":  "My name is Ned. You are my personal assistant. My favorite movies are Lord of the Rings and Hobbit.",
      "examples": [ { 
          "input": {"content": "Who do you work for?"},
          "output": {"content": "I work for Ned."}
      },
}' > sample_chat_prompts.txt favorite movies based on a book series?",
  % Total    % Received % Xferd  Average Speed   Time    Time     Time  Current
                                 Dload  Upload   Total   Spent    Left  Speed
100  2387    0  1725  100   662   2616   1003 --:--:-- --:--:-- --:--:--  3622
student_01_e721493cd4a9@cloudshell:~ (qwiklabs-gcp-03-f1d99a2bd4d4)$ export PROJECT_ID=$(gcloud config get-value project)
gsutil cp sample_chat_prompts.txt gs://$PROJECT_ID
Your active configuration is: [cloudshell-28891]
Copying file://sample_chat_prompts.txt [Content-Type=text/plain]...
/ [1 files][  1.7 KiB/  1.7 KiB]                                                
Operation completed over 1 objects/1.7 KiB.                                      
student_01_e721493cd4a9@cloudshell:~ (qwiklabs-gcp-03-f1d99a2bd4d4)$ MODEL_ID="text-bison"
PROJECT_ID=$DEVSHELL_PROJECT_ID
curl \
-X POST \
-H "Authorization: Bearer $(gcloud auth print-access-token)" \
-H "Content-Type: application/json" \
https://us-central1-aiplatform.googleapis.com/v1/projects/${PROJECT_ID}/locations/us-central1/publishers/google/models/${MODEL_ID}:predict -d \
$'{
  "instances": [
    { "prompt": "Give me ten interview questions for the role of program manager."}
  ],
  "parameters": {
    "temperature": 0.2,
    "maxOutputTokens": 1024,
}' > ideation_prompt_example.txt
  % Total    % Received % Xferd  Average Speed   Time    Time     Time  Current
                                 Dload  Upload   Total   Spent    Left  Speed
100  2206    0  1990  100   216   1236    134  0:00:01  0:00:01 --:--:--  1371
student_01_e721493cd4a9@cloudshell:~ (qwiklabs-gcp-03-f1d99a2bd4d4)$ export PROJECT_ID=$(gcloud config get-value project)
gsutil cp *.txt gs://$PROJECT_ID
Your active configuration is: [cloudshell-28891]
Copying file://ideation_prompt_example.txt [Content-Type=text/plain]...
Copying file://README-cloudshell.txt [Content-Type=text/plain]...               
Copying file://sample_chat_prompts.txt [Content-Type=text/plain]...             
Copying file://summarization_prompt_example.txt [Content-Type=text/plain]...    
- [4 files][  6.3 KiB/  6.3 KiB]                                                
Operation completed over 4 objects/6.3 KiB.                                      
student_01_e721493cd4a9@cloudshell:~ (qwiklabs-gcp-03-f1d99a2bd4d4)$ history 
    1  MODEL_ID="text-bison"
    2  PROJECT_ID=$DEVSHELL_PROJECT_ID
    3  curl -X POST -H "Authorization: Bearer $(gcloud auth print-access-token)" -H "Content-Type: application/json" https://us-central1-aiplatform.googleapis.com/v1/projects/${PROJECT_ID}/locations/us-central1/publishers/google/models/${MODEL_ID}:predict -d $'{
  "instances": [
    { "prompt": "Provide a summary with about two sentences for the following article:
The efficient-market hypothesis (EMH) is a hypothesis in financial \
economics that states that asset prices reflect all available \
information. A direct implication is that it is impossible to \
\\"beat the market\\" consistently on a risk-adjusted basis since market \
prices should only react to new information. Because the EMH is \
formulated in terms of risk adjustment, it only makes testable \
predictions when coupled with a particular model of risk. As a \
result, research in financial economics since at least the 1990s has \
focused on market anomalies, that is, deviations from specific \
models of risk. The idea that financial market returns are difficult \
to predict goes back to Bachelier, Mandelbrot, and Samuelson, but \
is closely associated with Eugene Fama, in part due to his \
influential 1970 review of the theoretical and empirical research. \
The EMH provides the basic logic for modern risk-based theories of \
asset prices, and frameworks such as consumption-based asset pricing \
and intermediary asset pricing can be thought of as the combination \
of a model of risk with the EMH. Many decades of empirical research \
on return predictability has found mixed evidence. Research in the \
1950s and 1960s often found a lack of predictability (e.g. Ball and \
Brown 1968; Fama, Fisher, Jensen, and Roll 1969), yet the \
1980s-2000s saw an explosion of discovered return predictors (e.g. \
Rosenberg, Reid, and Lanstein 1985; Campbell and Shiller 1988; \
Jegadeesh and Titman 1993). Since the 2010s, studies have often \
found that return predictability has become more elusive, as \
predictability fails to work out-of-sample (Goyal and Welch 2008), \
or has been weakened by advances in trading technology and investor \
learning (Chordia, Subrahmanyam, and Tong 2014; McLean and Pontiff \
2016; Martineau 2021).
Summary:
"}
  ],
  "parameters": {
    "temperature": 0.2,
    "maxOutputTokens": 256,
    "topK": 40,
    "topP": 0.95
  }
}'
    4  MODEL_ID="text-bison"
    5  PROJECT_ID=$DEVSHELL_PROJECT_ID
    6  curl -X POST -H "Authorization: Bearer $(gcloud auth print-access-token)" -H "Content-Type: application/json" https://us-central1-aiplatform.googleapis.com/v1/projects/${PROJECT_ID}/locations/us-central1/publishers/google/models/${MODEL_ID}:predict -d $'{
  "instances": [
    { "prompt": "Give me ten interview questions for the role of program manager."}
  ],
  "parameters": {
    "temperature": 0.2,
    "maxOutputTokens": 1024,
    "topK": 40,
    "topP": 0.8
  }
}'
    7  MODEL_ID="text-bison"
    8  PROJECT_ID=$DEVSHELL_PROJECT_ID
    9  curl -X POST -H "Authorization: Bearer $(gcloud auth print-access-token)" -H "Content-Type: application/json" https://us-central1-aiplatform.googleapis.com/v1/projects/${PROJECT_ID}/locations/us-central1/publishers/google/models/${MODEL_ID}:predict -d $'{
  "instances": [
    { "prompt": "Provide a summary with about two sentences for the following article:
The efficient-market hypothesis (EMH) is a hypothesis in financial \
economics that states that asset prices reflect all available \
information. A direct implication is that it is impossible to \
\\"beat the market\\" consistently on a risk-adjusted basis since market \
prices should only react to new information. Because the EMH is \
formulated in terms of risk adjustment, it only makes testable \
predictions when coupled with a particular model of risk. As a \
result, research in financial economics since at least the 1990s has \
focused on market anomalies, that is, deviations from specific \
models of risk. The idea that financial market returns are difficult \
to predict goes back to Bachelier, Mandelbrot, and Samuelson, but \
is closely associated with Eugene Fama, in part due to his \
influential 1970 review of the theoretical and empirical research. \
The EMH provides the basic logic for modern risk-based theories of \
asset prices, and frameworks such as consumption-based asset pricing \
and intermediary asset pricing can be thought of as the combination \
of a model of risk with the EMH. Many decades of empirical research \
on return predictability has found mixed evidence. Research in the \
1950s and 1960s often found a lack of predictability (e.g. Ball and \
Brown 1968; Fama, Fisher, Jensen, and Roll 1969), yet the \
1980s-2000s saw an explosion of discovered return predictors (e.g. \
Rosenberg, Reid, and Lanstein 1985; Campbell and Shiller 1988; \
Jegadeesh and Titman 1993). Since the 2010s, studies have often \
found that return predictability has become more elusive, as \
predictability fails to work out-of-sample (Goyal and Welch 2008), \
or has been weakened by advances in trading technology and investor \
learning (Chordia, Subrahmanyam, and Tong 2014; McLean and Pontiff \
2016; Martineau 2021).
Summary:
"}
  ],
  "parameters": {
    "temperature": 0.2,
    "maxOutputTokens": 256,
    "topK": 40,
    "topP": 0.95
  }
}' > summarization_prompt_example.txt
   10  export PROJECT_ID=$(gcloud config get-value project)
   11  gsutil cp *.txt gs://$PROJECT_ID
   12  history 
   13  MODEL_ID="chat-bison"
   14  PROJECT_ID=$DEVSHELL_PROJECT_ID
   15  curl -X POST -H "Authorization: Bearer $(gcloud auth print-access-token)" -H "Content-Type: application/json" https://us-central1-aiplatform.googleapis.com/v1/projects/${PROJECT_ID}/locations/us-central1/publishers/google/models/${MODEL_ID}:predict -d '{
  "instances": [{
      "context":  "My name is Ned. You are my personal assistant. My favorite movies are Lord of the Rings and Hobbit.",
      "examples": [ { 
          "input": {"content": "Who do you work for?"},
          "output": {"content": "I work for Ned."}
      },
      { 
          "input": {"content": "What do I like?"},
          "output": {"content": "Ned likes watching movies."}
      }],
      "messages": [
      { 
          "author": "user",
          "content": "Are my favorite movies based on a book series?",
      }],
  }],
  "parameters": {
    "temperature": 0.3,
    "maxDecodeSteps": 200,
    "topP": 0.8,
    "topK": 40
  }
}'
   16  MODEL_ID="chat-bison"
   17  PROJECT_ID=$DEVSHELL_PROJECT_ID
   18  curl -X POST -H "Authorization: Bearer $(gcloud auth print-access-token)" -H "Content-Type: application/json" https://us-central1-aiplatform.googleapis.com/v1/projects/${PROJECT_ID}/locations/us-central1/publishers/google/models/${MODEL_ID}:predict -d '{
  "instances": [{
      "context":  "My name is Ned. You are my personal assistant. My favorite movies are Lord of the Rings and Hobbit.",
      "examples": [ { 
          "input": {"content": "Who do you work for?"},
          "output": {"content": "I work for Ned."}
      },
      { 
          "input": {"content": "What do I like?"},
          "output": {"content": "Ned likes watching movies."}
      }],
      "messages": [
      { 
          "author": "user",
          "content": "Are my favorite movies based on a book series?",
      }],
  }],
  "parameters": {
    "temperature": 0.3,
    "maxDecodeSteps": 200,
    "topP": 0.8,
    "topK": 40
  }
}' > sample_chat_prompts.txt
   19  export PROJECT_ID=$(gcloud config get-value project)
   20  gsutil cp sample_chat_prompts.txt gs://$PROJECT_ID
   21  MODEL_ID="text-bison"
   22  PROJECT_ID=$DEVSHELL_PROJECT_ID
   23  curl -X POST -H "Authorization: Bearer $(gcloud auth print-access-token)" -H "Content-Type: application/json" https://us-central1-aiplatform.googleapis.com/v1/projects/${PROJECT_ID}/locations/us-central1/publishers/google/models/${MODEL_ID}:predict -d $'{
  "instances": [
    { "prompt": "Give me ten interview questions for the role of program manager."}
  ],
  "parameters": {
    "temperature": 0.2,
    "maxOutputTokens": 1024,
    "topK": 40,
    "topP": 0.8
  }
}' > ideation_prompt_example.txt
   24  export PROJECT_ID=$(gcloud config get-value project)
   25  gsutil cp *.txt gs://$PROJECT_ID
   26  history 
student_01_e721493cd4a9@cloudshell:~ (qwiklabs-gcp-03-f1d99a2bd4d4)$ 
```



In this lab you learned how to use the Vertex AI PaLM API to create text and chat responses. You have taken the first step to start your journey using Vertex AI's PaLM API and other Generative AI development tools!
