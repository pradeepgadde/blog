---

layout: single
title:  "Speech-to-Text API: Qwik Start"
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
# Speech-to-Text API: Qwik Start

The Speech-to-Text API enables easy integration of Google speech  recognition technologies into developer applications. The Speech-to-Text API allows you to send audio and receive a text transcription from the  service.

- Create an API key
- Create a Speech-to-Text API request
- Call the Speech-to-Text API

## Create an API key

Since you'll be using `curl` to send a request to the Speech-to-Text API, you need to generate an API key to pass in our request URL.

1. To create an API key, click **Navigation menu** > **APIs & services** > **Credentials**.
2. Then click **Create credentials**.
3. In the drop down menu, select **API key**.
4. Copy the key you just generated and click **Close**.

Now that you have an API key, save it as an environment variable to  avoid having to insert the value of your API key in each request.

To perform the next steps, connect using SSH to the instance provisioned for you.

1. In the **Navigation menu**,  select **Compute Engine**. You should see a `linux-instance` listed in the **VM instances** window.
2. Click on the **SSH** button in line with the `linux-instance`. You will be brought to an interactive shell.
3. In the command line, enter in the following, replacing `<YOUR_API_KEY>` with the API key you copied from previously generated:
4. You remain in this SSH session for the rest of the lab.

```sh
Welcome to Cloud Shell! Type "help" to get started.
Your Cloud Platform project in this session is set to qwiklabs-gcp-02-db8137c3a897.
Use “gcloud config set project [PROJECT_ID]” to change to a different project.
student_03_90b6e8064b80@cloudshell:~ (qwiklabs-gcp-02-db8137c3a897)$ export key=AIzaSyCcc3Zs_rvncxiWJ0FKPAXCaBhtDyE1mpE
student_03_90b6e8064b80@cloudshell:~ (qwiklabs-gcp-02-db8137c3a897)$ gcloud compute ssh --zone "us-east4-a" "linux-instance" --project "qwiklabs-gcp-02-db8137c3a897"
WARNING: The private SSH key file for gcloud does not exist.
WARNING: The public SSH key file for gcloud does not exist.
WARNING: You do not have an SSH key for gcloud.
WARNING: SSH keygen will be executed to generate a key.
This tool needs to create the directory [/home/student_03_90b6e8064b80/.ssh] before being able to generate SSH keys.

Do you want to continue (Y/n)?  y

Generating public/private rsa key pair.
Enter passphrase (empty for no passphrase): 
Enter same passphrase again: 
Your identification has been saved in /home/student_03_90b6e8064b80/.ssh/google_compute_engine
Your public key has been saved in /home/student_03_90b6e8064b80/.ssh/google_compute_engine.pub
The key fingerprint is:
SHA256:VaepKZIa/V0Kk5xb7TBS+fdl6xw0swr6ITL4ujrYwrg student_03_90b6e8064b80@cs-765520566984-default
The key's randomart image is:
+---[RSA 3072]----+
|            . .  |
|           o +   |
|          + o    |
|     . o = =     |
|    . + S * + .+o|
|     o.o O * ..o*|
|o o .. oo.oo.  +.|
|.+ o  . o o o + .|
|E...ooo. ... . o |
+----[SHA256]-----+
Warning: Permanently added 'compute.564891784240009304' (ED25519) to the list of known hosts.
Linux linux-instance 5.10.0-30-cloud-amd64 #1 SMP Debian 5.10.218-1 (2024-06-01) x86_64

The programs included with the Debian GNU/Linux system are free software;
the exact distribution terms for each program are described in the
individual files in /usr/share/doc/*/copyright.

Debian GNU/Linux comes with ABSOLUTELY NO WARRANTY, to the extent
permitted by applicable law.
Creating directory '/home/student-03-90b6e8064b80'.
student-03-90b6e8064b80@linux-instance:~$ export API_KEY=AIzaSyCcc3Zs_rvncxiWJ0FKPAXCaBhtDyE1mpE
student-03-90b6e8064b80@linux-instance:~$ 
```



## Create your Speech-to-Text API request

Create `request.json` in the SSH command line. You'll use this to build your request to the Speech-to-Text API:

You will use a pre-recorded file that's available on Cloud Storage: `gs://cloud-samples-tests/speech/brooklyn.flac`

The request body has a `config` and `audio` object.

```sh
student-03-90b6e8064b80@linux-instance:~$ touch request.json
student-03-90b6e8064b80@linux-instance:~$ nano request.json 
student-03-90b6e8064b80@linux-instance:~$ cat request.json 
{
  "config": {
      "encoding":"FLAC",
      "languageCode": "en-US"
  },
  "audio": {
      "uri":"gs://cloud-samples-tests/speech/brooklyn.flac"
  }
}
student-03-90b6e8064b80@linux-instance:~$ 
```



In `config`, you tell the Speech-to-Text API how to process the request. The `encoding` parameter tells the API which type of audio encoding you're using while the file is being sent to the API. `FLAC` is the encoding type for .raw files. Learn more about encoding types in the [RecognitionConfig Guide](https://cloud.google.com/speech/reference/rest/v1/RecognitionConfig).

There are other parameters you can add to your `config` object, but `encoding` is the only required one.

In the `audio` object, you pass the API the uri of the audio file in Cloud Storage.



## Call the Speech-to-Text API

1. Pass your request body, along with the API key environment variable, to the Speech-to-Text API with the following `curl` command (all in one single command line):

```sh
student-03-90b6e8064b80@linux-instance:~$ curl -s -X POST -H "Content-Type: application/json" --data-binary @request.json \
"https://speech.googleapis.com/v1/speech:recognize?key=${API_KEY}"
{
  "results": [
    {
      "alternatives": [
        {
          "transcript": "how old is the Brooklyn Bridge",
          "confidence": 0.93075204
        }
      ],
      "resultEndTime": "1.770s",
      "languageCode": "en-us"
    }
  ],
  "totalBilledTime": "2s",
  "requestId": "8998333739176426442"
}
student-03-90b6e8064b80@linux-instance:~$ 
```



The `transcript` value will return the Speech-to-Text API's text transcription of your audio file, and the `confidence` value indicates how sure the API is that it has accurately transcribed your audio.

You'll notice that you called the `syncrecognize` method  in the request above. The Speech-to-Text API supports both synchronous  and asynchronous speech to text transcription. In this example you sent  it a complete audio file, but you can also use the `syncrecognize` method to perform streaming speech to text transcription while the user is still speaking.

You created a Speech-to-Text API request then called the Speech-to-Text API.

1. Run the following command to save the response in a `result.json` file:

```sh
student-03-90b6e8064b80@linux-instance:~$ curl -s -X POST -H "Content-Type: application/json" --data-binary @request.json \
"https://speech.googleapis.com/v1/speech:recognize?key=${API_KEY}" > result.json
student-03-90b6e8064b80@linux-instance:~$ cat result.json 
{
  "results": [
    {
      "alternatives": [
        {
          "transcript": "how old is the Brooklyn Bridge",
          "confidence": 0.9307521
        }
      ],
      "resultEndTime": "1.770s",
      "languageCode": "en-us"
    }
  ],
  "totalBilledTime": "2s",
  "requestId": "9041167218622948864"
}
student-03-90b6e8064b80@linux-instance:~$ 
```



You used the Speech-to-Text API to retrieve a transcript of an input audio file.
