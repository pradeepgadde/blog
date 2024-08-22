---
layout: single
title:  "Python Collections Module"
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
# Python Collections Module

```py
# Collections module
from collections import Counter

mylist = [10,10,10,4,4,5,6,4,7,10,5,2,3,2,3,1]
count = Counter(mylist)
print(count)
most_common = count.most_common(3)
print(most_common)
print(f"{most_common[0][0]} is repeated the maximum times, that is {most_common[0][1]}")
print("Next Most common is ")
print(f"{most_common[1][0]} is repeated the maximum times, that is {most_common[1][1]}")
print("Next Most common is ")
print(f"{most_common[2][0]} is repeated the maximum times, that is {most_common[2][1]}")

# f-strings

i=3
print (f"{i} squared is {i*i} and half of {i} is {i/2}")


```

```sh
(base) pradeep:~$/usr/local/bin/python3 /Users/pradeep/LearnPython/collections_counter.py
Counter({10: 4, 4: 3, 5: 2, 2: 2, 3: 2, 6: 1, 7: 1, 1: 1})
[(10, 4), (4, 3), (5, 2)]
10 is repeated the maximum times, that is 4
Next Most common is 
4 is repeated the maximum times, that is 3
Next Most common is 
5 is repeated the maximum times, that is 2
3 squared is 9 and half of 3 is 1.5
(base) pradeep:~$
```

