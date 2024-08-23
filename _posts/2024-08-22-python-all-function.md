---
layout: single
title:  "Python All Function"
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
# Python All Function

:evergreen_tree:

## all()

Return True if bool(x) is True for all values x in the iterable.

If the iterable is empty, return True.

In this example, let's make use of `all()` function to find if a given string is a palindrome or not.

```py
s="pradeep"
print(s[0],s[~0])
print(s[1],s[~1])
print(s[2],s[~2])
print(s[3],s[~3])
print(s[4],s[~4])
print(s[5],s[~5])
print(s[6],s[~6])

l1=[1,0,0]
l2=[1,1]

x=all(l1)
y=all(l2)

print(x,y)
print(type(x))
all()

def is_palindrome(s):
    result = [s[i] == s[~i] for i in range(len(s)//2)]
    print(result)
    return all(s[i] == s[~i] for i in range(len(s)//2))
        

print(is_palindrome("pradeep"))
print(is_palindrome("gracecarg"))
print(is_palindrome("ogracecargo"))
print(is_palindrome("o"))
print(is_palindrome("oo"))
print(is_palindrome("ooo"))
```

:star: As seen here, we can confirm that palindrome test is working fine.

```sh
(base) pradeep:~$/usr/local/bin/python3 /Users/pradeep/LearnPython/palindrome_string.py
p p
r e
a e
d d
e a
e r
p p
False True
<class 'bool'>
[True, False, False]
False
[True, True, True, True]
True
[True, True, True, True, True]
True
[]
True
[True]
True
[True]
True
(base) pradeep:~$
```

:bulb:  When used on a dictionary, the all() function checks if all the *keys* are true, not the *values*.

```py
d1 = {0 : "Blue", 1 : "Green"}
d2 = {0 : "Black", 0 : "White"}
d3 = {1 : "Hello", 1 : "World"}
x = all(d1)
y = all(d2)
z = all(d3)
print(x,y,z)

```

```sh
(base) pradeep:~$/usr/local/bin/python3 /Users/pradeep/LearnPython/palindrome_string.py
False False True
(base) pradeep:~$
```

