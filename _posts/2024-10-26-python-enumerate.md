---
layout: single
title:  "Enumerate- Python"
categories: Programming
tags: Python
show_date: true
toc: true
toc_sticky: true
header:
  teaser: /assets/images/python.png
author:
  name     : "Python"
  avatar   : "/assets/images/python.png"

sidebar:
  - title: "Topics"
    nav: my-sidebar
---
# Enumerate using Python

Here is a sample code to implement enumerate using Python.

```py

mylist = ['srx', 'mx', 'qfx','ptx']
for key, value in enumerate(mylist):
    print(key, value)

print ("\nWith a new Index\n")
for key, value in enumerate(mylist,100):
    print(key, value)


print ("\nWith a Dictionary\n")
mydict = {'srx': 100, 'qfx':123, 'mx':99, 'ptx':23}
for (key,value) in mydict.items():
    print(key, value)

print ("\nWith a Dictionary\n")
mydict = {'srx': 100, 'qfx':123, 'mx':99, 'ptx':23}
for index, (key,value) in enumerate(mydict.items()):
    print(index, key, value)

print("\nReturn Type of Enumerate function\n")
c1=enumerate(mylist)
c2=enumerate(mydict)
print(type(c1))
print(type(c2))

print(list(enumerate(mydict)))
print(list(enumerate(mylist)))
```

## Output

```sh

0 srx
1 mx
2 qfx
3 ptx

With a new Index

100 srx
101 mx
102 qfx
103 ptx

With a Dictionary

srx 100
qfx 123
mx 99
ptx 23

With a Dictionary

0 srx 100
1 qfx 123
2 mx 99
3 ptx 23

Return Type of Enumerate function

<class 'enumerate'>
<class 'enumerate'>
[(0, 'srx'), (1, 'qfx'), (2, 'mx'), (3, 'ptx')]
[(0, 'srx'), (1, 'mx'), (2, 'qfx'), (3, 'ptx')]

```

