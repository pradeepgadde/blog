---
layout: single
title:  "Python Generators"
categories: Programming
tags: Python
show_date: true
classes: wide
header:
  teaser: /assets/images/python.png
author:
  name     : "Python"
  avatar   : "/assets/images/python.png"

sidebar:
  - title: "Topics"
    nav: my-sidebar
---
# Python Generators

Intro to Generators

```py
def generatordemo():
    yield 1
    yield 2
    yield 3

for value in generatordemo():
    print(value)
```

```sh
base) pradeep:~$/usr/local/bin/python3 /Users/pradeep/LearnPython/generator.py
1
2
3
(base) pradeep:~$
```
