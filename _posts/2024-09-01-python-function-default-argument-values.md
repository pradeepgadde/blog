---
layout: single
title:  "Python Functions Default Arguments"
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
# Python Functions Default Arguments
Default arguments are evaluated once, specifically when the module is loaded, and are shared across all callers.
As a rule, all mandatory arguments should have None as their default values.
The function can test the arguments in its body, if it is None, the function can assign it to the desired default value.

```py
def foo(x=[]):
    x.append(1)
    return x
result = foo()
print(result)
result = foo()
print(result)
```

```sh
(base) pradeep:~$/usr/local/bin/python3 /Users/pradeep/LearnPython/read_file_array.py
[1]
[1, 1]
(base) pradeep:~$
```

```py
def foo(x=None):
    if x is None:
        x=[]
    x.append(1)
    return x
result = foo()
print(result)
result = foo()
print(result)
```

```sh
(base) pradeep:~$/usr/local/bin/python3 /Users/pradeep/LearnPython/read_file_array.py
[1]
[1]
(base) pradeep:~$
```

Another example


```py
def read_file_array(filename, default=[]):
    try:
        filehandle = open(filename)
        return filename.read().split()
    except Exception:
        return default

a=read_file_array('abc.txt')
a.append('first')
print(a)
b=read_file_array('abc.txt')
b.append('second')
print(b)
```



```sh
(base) pradeep:~$/usr/local/bin/python3 /Users/pradeep/LearnPython/read_file_array.py
['first']
['first', 'second']
(base) pradeep:~$

```



```py
def read_file_array(filename, default=None):
    if default is None:
        default = []
    try:
        filehandle = open(filename)
        return filename.read().split()
    except Exception:
        return default

a=read_file_array('abc.txt')
a.append('first')
print(a)
b=read_file_array('abc.txt')
b.append('second')
print(b)
```

```sh
(base) pradeep:~$/usr/local/bin/python3 /Users/pradeep/LearnPython/read_file_array.py
['first']
['second']
(base) pradeep:~$
```



```py
def foo(x,y,z):
    try:
        result = x*x + y*y + z*z

    except Exception as err:
        print(f"There is an error {err}")
    finally:
        print(f"Result is {result}")

foo(1,2,3)
foo(z=1, y=2, x=1)
foo(1,z=3,y=2)
#foo(x=1,2,z=3)
#foo(1,x=2,z=3,y=4)
```

```sh
(base) pradeep:~$/usr/local/bin/python3 /Users/pradeep/LearnPython/read_file_array.py
Result is 14
Result is 6
Result is 14
(base) pradeep:~$
```

```sh
 File "/Users/pradeep/LearnPython/read_file_array.py", line 61
    foo(x=1,2,z=3)
                 ^
SyntaxError: positional argument follows keyword argument
```

```sh
Traceback (most recent call last):
  File "/Users/pradeep/LearnPython/read_file_array.py", line 62, in <module>
    foo(1,x=2,z=3,y=4)
TypeError: foo() got multiple values for argument 'x'
(base) pradeep:~$
```

