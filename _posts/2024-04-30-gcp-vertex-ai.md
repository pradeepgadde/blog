---

layout: single
title:  "Get Started with Vertex AI Studio"
categories: Cloud
tags: GCP
classes: wide
show_date: true
header:
  teaser: /assets/images/vertex.png
author:
  name     : "Vertex AI"
  avatar   : "/assets/images/vertex.png"

sidebar:
  - title: "Topics"
    nav: my-sidebar

---

# Get Started with Vertex AI Studio

**Vertex AI** is a comprehensive machine learning development platform that provides both **predictive** and **generative AI** capabilities. It allows you to train, evaluate, and deploy predictive  machine learning models for forecasting purposes. Additionally, you can  utilize the platform to discover, tune, and serve generative AI models  to produce content.

**[Vertex AI Studio](https://cloud.google.com/generative-ai-studio)** lets you quickly test and customize **generative AI models** so you can leverage their capabilities in your applications. It  provides a variety of tools and resources including both UI (user  interface) and coding examples that make it easy to start with  generative AI, even if you don't have a background in machine learning.



- Analyze images with Gemini multimodal.
- Explore multimodal capabilities.
- Design prompts with free-form and structured mode.
- Generate conversations.

### Enable the Vertex AI API

1. In the Google Cloud Console, enter **Vertex AI API** in the top search bar.
2. Click on the result for **Vertex AI API** under Marketplace & APIs.
3. Click **Enable**.

In the Google Cloud console, navigate to **Navigation menu** (![Navigation menu](https://cdn.qwiklabs.com/tkgw1TDgj4Q%2BYKQUW4jUFd0O5OEKlUMBRYbhlCrF0WY%3D))>**Artificial Intelligence** > **Vertex AI**> **Vertex AI Studio**> **Overview**.

You find four features: **Multimodal**, **Language**, **Vision**, and **Speech**. You focus on the first two in this lab.

Under **Multimodal powered by Gemini**, click **Try Gemini**.



The UI contains three main sections:

- Prompt (located at the top): Here, you can create a task that utilizes multimodal capabilities.

- Configuration (located on the right): This section allows you to  select models, configure parameters, and obtain the corresponding code.

- Response (located at the bottom): This section displays the results of your task.

Temperature controls the degree of randomness in token selection. Lower  temperatures are good for prompts that expect a true or correct  response, while higher temperatures can lead to more diverse or  unexpected results. With a temperature of 0 the highest probability  token is always selected.

Tune the parameter. Adjust the temperature by scrolling from left (0) to right (1). Resubmit the prompt to observe any changes in the outcome  compared to the previous result.



Save the prompt. Once you finish the prompt design, save the prompt by clicking **Save** and if prompted to select the region select  from the dropdown and then confirm **Save**. To find your saved prompts, navigate to **Multimodal**>**My prompts**.



In addition to **images** and **text**, Gemini multimodal is capable of accepting **videos** as inputs and generating text as an output.

Multimodal powered by Gemini offers many capabilities such as writing  stories from images, analyzing videos, and generating multimedia ads.  Explore more multimodal use cases by clicking **Multimodal**>**Sample Prompts**.

## Design prompts with free-form and structured mode

1. In the Vertex AI menu, for **Vertex AI Studio > overview** page , click Open for **Language Powered by Gemini**.

### Create prompt

Create Prompt lets you design prompts for tasks relevant to your business use case including code generation.

Click on the **Text Prompt** 

### Prompt design

You can feed your desired input text, e.g. a question, to the model.  The model will then provide a response based on how you structured your  prompt. The process of figuring out and designing the best input text  (prompt) to get the desired response back from the model is called **Prompt Design**.

There is no best way to design the prompts yet. There are 3 methods you can use to shape the model's response:

- **Zero-shot prompting** - This is a method where the LLM is given only a prompt that describes the task and no additional data. For example, if you want the LLM to answer a question, you just prompt  "what is prompt design?".
- **One-shot prompting** - This is a method where the LLM is  given a single example of the task that it is being asked to perform.   For example, if you want the LLM to write a poem, you might give it a  single example poem.
- **Few-shot prompting** - This is a method where the LLM is  given a small number of examples of the task that it is being asked to  perform. For example, if you want the LLM to write a news article, you  might give it a few news articles to read.

You may also notice the **FREE-FORM** and **STRUCTURED** tabs.

hose are the two modes that you can use when designing your prompt.

- **FREE-FORM** - This mode provides a free and easy approach to design your prompt. It is suitable for small and experimental  prompts with no additional examples. You will be using this to explore  zero-shot prompting.
- **STRUCTURED** - This mode provides an easy-to-use template approach to prompt design. Context and multiple examples can be added  to the prompt in this mode. This is especially useful for one-shot and  few-shot prompting methods which you will be exploring later.

- adjust the `Token limit` parameter to `1` and click the **SUBMIT** button
- adjust the `Token limit` parameter to `1024` and click the **SUBMIT** button
- adjust the `Temperature` parameter to `0.5` and click the **SUBMIT** button
- adjust the `Temperature` parameter to `1.0` and click the **SUBMIT** button

### FREE-FORM mode

Try zero-shot prompting in **FREE-FORM** mode.

1. Copy the following over to the prompt input field. Keep the current default model setting, which is **Gemini Pro**.

### STRUCTURED mode

With **STRUCTURED** mode, you can design prompts in more organized ways. You can provide **Context** and **Examples** in their respective input fields. This is a good opportunity to learn one-shot and few-shot prompting.

In this section, you will ask the model to complete a sentence.

1. Return to the **Text Prompt** window.
2. At the top of the page, click on the **STRUCTURED** tab.
3. Remove any text from the **Context**
4. Under **Test** field, copy the following in **INPUT** field.

## Generate conversations

Create Chat Prompt lets you have a freeform chat with the model,  which tracks what was previously said and responds based on context.

1. Return to the **Language** page.
2. Click on the **TEXT CHAT** button to create a new chat prompt.
3. Under **Model**, select **chat-bison (latest)**. You will see the new chat prompt page.

For this section, you will add context to the chat and let the model respond based on the context provided.

1. Then the following context to **Context** field.
