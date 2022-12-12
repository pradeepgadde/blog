---
layout: single
title:  "Getting Started with OpenAI"
date:   2022-12-12 10:55:04 +0530
categories: OpenAI
tags: OpenAI
author: "Pradeep Gadde"
classes: wide
show_date: true
header:
 
  teaser: /assets/images/openai.png

sidebar:
  - title: "Blog"
    text: "Topics"
    nav: my-sidebar
---



# OpenAI

https://beta.openai.com/docs/quickstart

```sh
(base) pradeep:~$pwd
/Users/pradeep
(base) pradeep:~$git clone https://github.com/openai/openai-quickstart-python.git
Cloning into 'openai-quickstart-python'...
remote: Enumerating objects: 22, done.
remote: Counting objects: 100% (15/15), done.
remote: Compressing objects: 100% (13/13), done.
Receiving objects: 100% (22/22), 5.32 KiB | 1.33 MiB/s, done.
Resolving deltas: 100% (7/7), done.
remote: Total 22 (delta 7), reused 2 (delta 2), pack-reused 7
(base) pradeep:~$cd openai-quickstart-python
cp .env.example .env
(base) pradeep:~$ls
README.md		app.py			requirements.txt	static			templates
(base) pradeep:~$python -m venv venv
. venv/bin/activate
pip install -r requirements.txt
flask run

Collecting autopep8==1.6.0
  Downloading autopep8-1.6.0-py2.py3-none-any.whl (45 kB)
     ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━ 45.3/45.3 KB 639.0 kB/s eta 0:00:00
Collecting certifi==2021.10.8
  Using cached certifi-2021.10.8-py2.py3-none-any.whl (149 kB)
Collecting charset-normalizer==2.0.7
  Downloading charset_normalizer-2.0.7-py3-none-any.whl (38 kB)
Collecting click==8.0.3
  Using cached click-8.0.3-py3-none-any.whl (97 kB)
Collecting et-xmlfile==1.1.0
  Downloading et_xmlfile-1.1.0-py3-none-any.whl (4.7 kB)
Collecting Flask==2.0.2
  Downloading Flask-2.0.2-py3-none-any.whl (95 kB)
     ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━ 95.2/95.2 KB 607.1 kB/s eta 0:00:00
Collecting idna==3.3
  Using cached idna-3.3-py3-none-any.whl (61 kB)
Collecting itsdangerous==2.0.1
  Using cached itsdangerous-2.0.1-py3-none-any.whl (18 kB)
Collecting Jinja2==3.0.2
  Using cached Jinja2-3.0.2-py3-none-any.whl (133 kB)
Collecting MarkupSafe==2.0.1
  Using cached MarkupSafe-2.0.1-cp39-cp39-macosx_10_9_x86_64.whl (13 kB)
Collecting numpy==1.21.3
  Downloading numpy-1.21.3-cp39-cp39-macosx_10_9_x86_64.whl (17.0 MB)
     ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━ 17.0/17.0 MB 1.3 MB/s eta 0:00:00
Collecting openai==0.19.0
  Downloading openai-0.19.0.tar.gz (42 kB)
     ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━ 42.4/42.4 KB 984.1 kB/s eta 0:00:00
  Installing build dependencies ... done
  Getting requirements to build wheel ... done
  Preparing metadata (pyproject.toml) ... done
Collecting openpyxl==3.0.9
  Downloading openpyxl-3.0.9-py2.py3-none-any.whl (242 kB)
     ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━ 242.2/242.2 KB 2.0 MB/s eta 0:00:00
Collecting pandas==1.3.4
  Downloading pandas-1.3.4-cp39-cp39-macosx_10_9_x86_64.whl (11.6 MB)
     ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━ 11.6/11.6 MB 2.2 MB/s eta 0:00:00
Collecting pandas-stubs==1.2.0.35
  Downloading pandas_stubs-1.2.0.35-py3-none-any.whl (159 kB)
     ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━ 159.3/159.3 KB 1.6 MB/s eta 0:00:00
Collecting pycodestyle==2.8.0
  Downloading pycodestyle-2.8.0-py2.py3-none-any.whl (42 kB)
     ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━ 42.1/42.1 KB 950.2 kB/s eta 0:00:00
Collecting python-dateutil==2.8.2
  Using cached python_dateutil-2.8.2-py2.py3-none-any.whl (247 kB)
Collecting python-dotenv==0.19.2
  Downloading python_dotenv-0.19.2-py2.py3-none-any.whl (17 kB)
Collecting pytz==2021.3
  Downloading pytz-2021.3-py2.py3-none-any.whl (503 kB)
     ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━ 503.5/503.5 KB 2.6 MB/s eta 0:00:00
Collecting requests==2.26.0
  Downloading requests-2.26.0-py2.py3-none-any.whl (62 kB)
     ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━ 62.3/62.3 KB 917.8 kB/s eta 0:00:00
Collecting six==1.16.0
  Using cached six-1.16.0-py2.py3-none-any.whl (11 kB)
Collecting toml==0.10.2
  Downloading toml-0.10.2-py2.py3-none-any.whl (16 kB)
Collecting tqdm==4.62.3
  Downloading tqdm-4.62.3-py2.py3-none-any.whl (76 kB)
     ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━ 76.2/76.2 KB 1.2 MB/s eta 0:00:00
Collecting urllib3==1.26.7
  Downloading urllib3-1.26.7-py2.py3-none-any.whl (138 kB)
     ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━ 138.8/138.8 KB 1.4 MB/s eta 0:00:00
Collecting Werkzeug==2.0.2
  Using cached Werkzeug-2.0.2-py3-none-any.whl (288 kB)
Building wheels for collected packages: openai
  Building wheel for openai (pyproject.toml) ... done
  Created wheel for openai: filename=openai-0.19.0-py3-none-any.whl size=53512 sha256=f0de4bf700953f8b3bccb81b7f42f797e11c13bc18eac854f7e9a12f34347f7a
  Stored in directory: /Users/pradeep/Library/Caches/pip/wheels/02/b0/11/038d87bdba9a4a9dfd20b1e24a643bd3bc9de8b24621a8c063
Successfully built openai
Installing collected packages: pytz, pandas-stubs, certifi, Werkzeug, urllib3, tqdm, toml, six, python-dotenv, pycodestyle, numpy, MarkupSafe, itsdangerous, idna, et-xmlfile, click, charset-normalizer, requests, python-dateutil, openpyxl, Jinja2, autopep8, pandas, Flask, openai
Successfully installed Flask-2.0.2 Jinja2-3.0.2 MarkupSafe-2.0.1 Werkzeug-2.0.2 autopep8-1.6.0 certifi-2021.10.8 charset-normalizer-2.0.7 click-8.0.3 et-xmlfile-1.1.0 idna-3.3 itsdangerous-2.0.1 numpy-1.21.3 openai-0.19.0 openpyxl-3.0.9 pandas-1.3.4 pandas-stubs-1.2.0.35 pycodestyle-2.8.0 python-dateutil-2.8.2 python-dotenv-0.19.2 pytz-2021.3 requests-2.26.0 six-1.16.0 toml-0.10.2 tqdm-4.62.3 urllib3-1.26.7
WARNING: You are using pip version 22.0.4; however, version 22.3.1 is available.
You should consider upgrading via the '/Users/pradeep/openai-quickstart-python/venv/bin/python -m pip install --upgrade pip' command.
 * Tip: There are .env or .flaskenv files present. Do "pip install python-dotenv" to use them.
 * Environment: production
   WARNING: This is a development server. Do not use it in a production deployment.
   Use a production WSGI server instead.
 * Debug mode: off
Usage: flask run [OPTIONS]
Try 'flask run --help' for help.

Error: While importing "app", an ImportError was raised:

Traceback (most recent call last):
  File "/Users/pradeep/opt/anaconda3/lib/python3.9/site-packages/flask/cli.py", line 240, in locate_app
    __import__(module_name)
  File "/Users/pradeep/openai-quickstart-python/app.py", line 3, in <module>
    import openai
ModuleNotFoundError: No module named 'openai'

(venv) (base) pradeep:~$

```

```python
(venv) (base) pradeep:~$ cat app.py 
import os

import openai
from flask import Flask, redirect, render_template, request, url_for

app = Flask(__name__)
openai.api_key = os.getenv("OPENAI_API_KEY")


@app.route("/", methods=("GET", "POST"))
def index():
    if request.method == "POST":
        animal = request.form["animal"]
        response = openai.Completion.create(
            model="text-davinci-002",
            prompt=generate_prompt(animal),
            temperature=0.6,
        )
        return redirect(url_for("index", result=response.choices[0].text))

    result = request.args.get("result")
    return render_template("index.html", result=result)


def generate_prompt(animal):
    return """Suggest three names for an animal that is a superhero.

Animal: Cat
Names: Captain Sharpclaw, Agent Fluffball, The Incredible Feline
Animal: Dog
Names: Ruff the Protector, Wonder Canine, Sir Barks-a-Lot
Animal: {}
Names:""".format(
        animal.capitalize()
    )
(venv) (base) pradeep:~$
```

```sh
(venv) (base) pradeep:~$cat requirements.txt 
autopep8==1.6.0
certifi==2021.10.8
charset-normalizer==2.0.7
click==8.0.3
et-xmlfile==1.1.0
Flask==2.0.2
idna==3.3
itsdangerous==2.0.1
Jinja2==3.0.2
MarkupSafe==2.0.1
numpy==1.21.3
openai==0.19.0
openpyxl==3.0.9
pandas==1.3.4
pandas-stubs==1.2.0.35
pycodestyle==2.8.0
python-dateutil==2.8.2
python-dotenv==0.19.2
pytz==2021.3
requests==2.26.0
six==1.16.0
toml==0.10.2
tqdm==4.62.3
urllib3==1.26.7
Werkzeug==2.0.2
(venv) (base) pradeep:~$
```

```sh
(venv) (base) pradeep:~$cat .env
FLASK_APP=app
FLASK_ENV=development
OPENAI_API_KEY=
(venv) (base) pradeep:~$
```

```sh
(venv) (base) pradeep:~$flask run
 * Serving Flask app 'app' (lazy loading)
 * Environment: development
 * Debug mode: on
 * Running on http://127.0.0.1:5000/ (Press CTRL+C to quit)
 * Restarting with stat
 * Debugger is active!
 * Debugger PIN: 902-146-329

```

![OpenAI]({{ site.url }}{{ site.baseurl }}/assets/images/openai-1.png)

![OpenAI]({{ site.url }}{{ site.baseurl }}/assets/images/openai-2.png)

![OpenAI]({{ site.url }}{{ site.baseurl }}/assets/images/openai-3.png)

![OpenAI]({{ site.url }}{{ site.baseurl }}/assets/images/openai-4.png)

```sh
(venv) (base) pradeep:~$flask run
 * Serving Flask app 'app' (lazy loading)
 * Environment: development
 * Debug mode: on
 * Running on http://127.0.0.1:5000/ (Press CTRL+C to quit)
 * Restarting with stat
 * Debugger is active!
 * Debugger PIN: 902-146-329
127.0.0.1 - - [12/Dec/2022 18:09:29] "GET / HTTP/1.1" 200 -
127.0.0.1 - - [12/Dec/2022 18:09:30] "GET /static/main.css HTTP/1.1" 304 -
127.0.0.1 - - [12/Dec/2022 18:09:30] "GET /static/dog.png HTTP/1.1" 304 -
127.0.0.1 - - [12/Dec/2022 18:10:23] "POST / HTTP/1.1" 302 -
127.0.0.1 - - [12/Dec/2022 18:10:23] "GET /?result=+Spirit+the+Fearless%2C+Thunder+the+Mighty%2C+Miracle+the+Unstoppable HTTP/1.1" 200 -
127.0.0.1 - - [12/Dec/2022 18:10:23] "GET /static/main.css HTTP/1.1" 304 -
127.0.0.1 - - [12/Dec/2022 18:10:23] "GET /static/dog.png HTTP/1.1" 304 -
127.0.0.1 - - [12/Dec/2022 18:10:48] "POST / HTTP/1.1" 302 -
127.0.0.1 - - [12/Dec/2022 18:10:48] "GET /?result=+Goliath%2C+Maximus%2C+Samson HTTP/1.1" 200 -
127.0.0.1 - - [12/Dec/2022 18:10:49] "GET /static/main.css HTTP/1.1" 304 -
127.0.0.1 - - [12/Dec/2022 18:10:49] "GET /static/dog.png HTTP/1.1" 304 -
127.0.0.1 - - [12/Dec/2022 18:11:10] "POST / HTTP/1.1" 302 -
127.0.0.1 - - [12/Dec/2022 18:11:10] "GET /?result=+The+Amazing+Doggo%2C+The+Unstoppable+Pupper%2C+The+Incredible+Pup HTTP/1.1" 200 -
127.0.0.1 - - [12/Dec/2022 18:11:11] "GET /static/main.css HTTP/1.1" 304 -
127.0.0.1 - - [12/Dec/2022 18:11:11] "GET /static/dog.png HTTP/1.1" 304 -
127.0.0.1 - - [12/Dec/2022 18:11:27] "POST / HTTP/1.1" 302 -
127.0.0.1 - - [12/Dec/2022 18:11:27] "GET /?result=+Pegasus%2C+Black+Beauty%2C+Spirit HTTP/1.1" 200 -
127.0.0.1 - - [12/Dec/2022 18:11:27] "GET /static/main.css HTTP/1.1" 304 -
127.0.0.1 - - [12/Dec/2022 18:11:27] "GET /static/dog.png HTTP/1.1" 304 -
127.0.0.1 - - [12/Dec/2022 18:12:51] "POST / HTTP/1.1" 302 -
127.0.0.1 - - [12/Dec/2022 18:12:51] "GET /?result=+The+Amazing+Polly%2C+Captain+Beak%2C+Superbird HTTP/1.1" 200 -
127.0.0.1 - - [12/Dec/2022 18:12:51] "GET /static/main.css HTTP/1.1" 304 -
127.0.0.1 - - [12/Dec/2022 18:12:51] "GET /static/dog.png HTTP/1.1" 304 -

```

