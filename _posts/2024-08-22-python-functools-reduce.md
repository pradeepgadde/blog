---
layout: single
title:  "Python Reduce Function"
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
# Python Reduce Function

## reduce()
reduce(function, iterable[, initial]) -> value

Apply a function of two arguments cumulatively to the items of a sequence or iterable, from left to right, so as to reduce the iterable to a single
value. For example, `reduce(lambda x, y: x+y, [1, 2, 3, 4, 5])` calculates `((((1+2)+3)+4)+5)`. If initial is present, it is placed before the items of the iterable in the calculation, and serves as a default when the iterable is empty.

Here is an example that makes use of `reduce` to find the column value corresponding to the column letters that we see in the spreadsheets.

In this sample, first function does not use the `reduce` method. Second one does use it.

```py
from functools import reduce
def column_id(s):
    return reduce(lambda result,c: result *26 + ord(c) - ord('A')+1,s,0)

print(column_id('CA'))
print(column_id('D'))
print(column_id('ZZ'))


def get_ord(s):
    result = 0
    for c in s:
        result = result *26 + ord(c) - ord('A') + 1
    return result

print(get_ord('CA'))
print(get_ord('D'))
print(get_ord('ZZ'))
```

```sh
(base) pradeep:~$/usr/local/bin/python3 /Users/pradeep/LearnPython/ftools.py
79
4
702
79
4
702
(base) pradeep:~$
```

:hand: We can see combining `lamda` with `reduce` simplifies the code a lot.
