---
layout: single
title:  "Python for Network Engineers Week#1"
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
# Python for Network Engineers Week 1

In this post, I am recording my learnings from the Python for Network Engineers course taught by Kirk Byers.

https://pynet.twb-tech.com/free-python-course.html

It's a free ten-week, email-course for network  engineers wanting to learn Python.

## Create a Virtual Environment

Create a virtual environment in the current directory with the `python3 -m venv <name>`. In this case, I am creating a venv named `Python4Networking`.
Once it is created, you need to activate the venv with the `source <name>/bin/activate`.

Then the prompt changes showing the venv name. In this case `(Python4Networking) (base)`.

```py
Last login: Mon Jul 22 16:31:14 on console

(base) pradeep:~$
(base) pradeep:~$cd LearnPython 
(base) pradeep:~$which python
/Users/pradeep/opt/anaconda3/bin/python
(base) pradeep:~$which python3
/Users/pradeep/opt/anaconda3/bin/python3
(base) pradeep:~$python3 -m venv Python4Networking
(base) pradeep:~$source Python4Networking/bin/activate
(Python4Networking) (base) pradeep:~$pwd
/Users/pradeep/LearnPython
(Python4Networking) (base) pradeep:~$
```

## Input and Output

Next, lets read from standard input using the `input` method and print to the standard output using the `print`.

For `print`, there are a couple of methods. Three different methods are shown here.
Using `%`, `.format` and `f` string methods.

```py
(Python4Networking) (base) pradeep:~$python3
Python 3.9.12 (main, Apr  5 2022, 01:53:17) 
[Clang 12.0.0 ] :: Anaconda, Inc. on darwin
Type "help", "copyright", "credits" or "license" for more information.
>>> my_ip = input ("Enter an IP address: ")
Enter an IP address: 192.168.100.1
>>> my_ip
'192.168.100.1'
>>> print(my_ip)
192.168.100.1
>>> print ("My IP address is : %s" % my_ip)
My IP address is : 192.168.100.1
>>> print ("My IP address is {}".format(my_ip))
My IP address is 192.168.100.1
>>> print (f"My IP address is {my_ip}")
My IP address is 192.168.100.1
>>> 
```

When there are multiple variables,

```py
>>> ip1 = "192.168.1.1"
>>> ip2 = "192.168.1.2"
>>> print (f"My two IPs are {ip1}, {ip2}")
My two IPs are 192.168.1.1, 192.168.1.2
>>> 
```

Using the `.format` method

```py
>>> print ("My two IPs are {}, {}".format(ip1,ip2))
My two IPs are 192.168.1.1, 192.168.1.2
>>> 
```

Using the `%` method

```py
>>> print ("My two IPs are %s %s" % (ip1,ip2))
My two IPs are 192.168.1.1 192.168.1.2
>>> 
```

Of all three methods, format strings `f` is recommended.

## Dir and Help

```py
>>> type(ip1)
<class 'str'>
>>> dir(ip1)
['__add__', '__class__', '__contains__', '__delattr__', '__dir__', '__doc__', '__eq__', '__format__', '__ge__', '__getattribute__', '__getitem__', '__getnewargs__', '__gt__', '__hash__', '__init__', '__init_subclass__', '__iter__', '__le__', '__len__', '__lt__', '__mod__', '__mul__', '__ne__', '__new__', '__reduce__', '__reduce_ex__', '__repr__', '__rmod__', '__rmul__', '__setattr__', '__sizeof__', '__str__', '__subclasshook__', 'capitalize', 'casefold', 'center', 'count', 'encode', 'endswith', 'expandtabs', 'find', 'format', 'format_map', 'index', 'isalnum', 'isalpha', 'isascii', 'isdecimal', 'isdigit', 'isidentifier', 'islower', 'isnumeric', 'isprintable', 'isspace', 'istitle', 'isupper', 'join', 'ljust', 'lower', 'lstrip', 'maketrans', 'partition', 'removeprefix', 'removesuffix', 'replace', 'rfind', 'rindex', 'rjust', 'rpartition', 'rsplit', 'rstrip', 'split', 'splitlines', 'startswith', 'strip', 'swapcase', 'title', 'translate', 'upper', 'zfill']
>>> help(ip1.split)

>>> 
```

```sh
Help on built-in function split:

split(sep=None, maxsplit=-1) method of builtins.str instance
    Return a list of the words in the string, using sep as the delimiter string.
    
    sep
      The delimiter according which to split the string.
      None (the default value) means split according to any whitespace,
      and discard empty strings from the result.
    maxsplit
      Maximum number of splits to do.
      -1 (the default value) means no limit.
```

## Strings and Formatting

Let's see `split` in action

```py
>>> ip1.split
<built-in method split of str object at 0x7fd4035398b0>
>>> ip1.split()
['192.168.1.1']
>>> ip1.split(".")
['192', '168', '1', '1']
>>> 
```

```py
>>> ip1 = "192.168.1.1"
>>> type(ip1)
<class 'str'>
>>> ip1=ip1.split(".")
>>> type(ip1)
<class 'list'>
>>> ip1
['192', '168', '1', '1']
>>> 
```

Original value is unmodified. You can use the same name to reassign. In this case we have changed from `str` to `list` with the split method.

Let's see the opposite of `split` that is `join`.

```py
>>> mac1 = ["01","02","03","04","05","06"]
>>> type(mac1)
<class 'list'>
>>> (":").join(mac1)
'01:02:03:04:05:06'
>>> ("-").join(mac1)
'01-02-03-04-05-06'
>>> 
>>> mac1=("-").join(mac1)
>>> type(mac1)
<class 'str'>
>>> mac1
'01-02-03-04-05-06'
>>> 
```

In this case, we converted from `list` to `str`.

Now, let's format the output a bit, left aligned (default), right aligned, and center aligned. You can use it with strings, expressions, and variables.

```py
>>> print(f"{mac1:20}")
01-02-03-04-05-06   
>>> print(f"{mac1:>20}")
   01-02-03-04-05-06
>>> print(f"{mac1:^20}")
 01-02-03-04-05-06  
>>> 
```

```py
>>> print(f"{'MAC Address':^20}")
    MAC Address     
>>> print(f"{'-' * 20:^20}")
--------------------
>>> print(f"{mac1:^20}")
 01-02-03-04-05-06  
>>> 
```

```py
>>> print (f"My IP and MAC are {ip2}, {mac1}")
My IP and MAC are 192.168.1.2, 01-02-03-04-05-06
>>> print (f"My  IP and MAC are {ip2:^30}, {mac1:^30}")
My  IP and MAC are          192.168.1.2          ,       01-02-03-04-05-06       
>>> print (f"My  IP and MAC are {ip2:^20}, {mac1:^20}")
My  IP and MAC are     192.168.1.2     ,  01-02-03-04-05-06  
>>> 
```

