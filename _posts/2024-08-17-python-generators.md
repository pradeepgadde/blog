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

A generator function in Python is defined like a  normal function, but whenever it needs to generate a value, it does so  with the  yield keyword rather than return. If the body of a def contains yield, the function automatically becomes a Python generator function. 

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

## Generators to save memory

```py
import sys
mylist = [i for i in range(100000)]
print(sum(mylist))
print(sys.getsizeof(mylist), "bytes")
print(type(mylist))
mygen = (i for i in range(100000))
print(sum(mygen))
print(sys.getsizeof(mygen), "bytes")
print(type(mygen))
```

As seen in the output, instead of usings Lists, if we use generators, we can save a lot of space.

```sh
base) pradeep:~$/usr/local/bin/python3 /Users/pradeep/LearnPython/sizeof.py
4999950000
800984 bytes
<class 'list'>
4999950000
200 bytes
<class 'generator'>
(base) pradeep:~$
```

`sys.getsizeof()` will help us find the amount of memory.
