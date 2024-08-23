---
layout: single
title:  "Python Arguments and Lambda"
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
# Python Arguments and Lambda

There are two types of arguments: positional and keyword arguments.

```py
def arg_demo(*arg):
    print(*arg)

arg_demo(1,2,3)

def kwarg_demo(**kwarg):
    print(kwarg['y'])

kwarg_demo(x= 10, y=2)

def args_kwargs(*arg, **kwarg):
    print(kwarg["x"],kwarg["y"],*arg)

args_kwargs(6,7,8,x="hello",y="world")

x=[i for i in range(10)]
print(x)

x=[[j for j in range(5)] for  i in range(10)]
print(x)

x= [(i, i+1) for i in range(5)]
print(x)
import dis
data = [1,2,-4,2,-5]
for index, value in enumerate(data):
    # print(index, value)
    if value < 0:
        data[index]=0
    
print(data)
print(sorted(data,reverse=True))

add_two = lambda x:x+2
print(add_two(234))
sum = lambda x,y:x+y

print(sum(4,5))

dis.dis(sum)
```

```sh
(base) pradeep:~$/usr/local/bin/python3 /Users/pradeep/LearnPython/demo_arg.py
1 2 3
2
hello world 6 7 8
[0, 1, 2, 3, 4, 5, 6, 7, 8, 9]
[[0, 1, 2, 3, 4], [0, 1, 2, 3, 4], [0, 1, 2, 3, 4], [0, 1, 2, 3, 4], [0, 1, 2, 3, 4], [0, 1, 2, 3, 4], [0, 1, 2, 3, 4], [0, 1, 2, 3, 4], [0, 1, 2, 3, 4], [0, 1, 2, 3, 4]]
[(0, 1), (1, 2), (2, 3), (3, 4), (4, 5)]
[1, 2, 0, 2, 0]
[2, 2, 1, 0, 0]
236
9
 36           0 RESUME                   0
              2 LOAD_FAST                0 (x)
              4 LOAD_FAST                1 (y)
              6 BINARY_OP                0 (+)
             10 RETURN_VALUE
(base) pradeep:~$
```

> Positional arguments should be listed first, otherwise a SyntaxError will be shown

```py
(base) pradeep:~$/usr/local/bin/python3 /Users/pradeep/LearnPython/demo_arg.py
  File "/Users/pradeep/LearnPython/demo_arg.py", line 14
    args_kwargs(x="hello",y="world",6,7,8)
                                         ^
SyntaxError: positional argument follows keyword argument
(base) pradeep:~$
```

The `dis.dis()` Disassemble classes, methods, functions, and other compiled objects.

With no argument, disassemble the last traceback.

Compiled objects currently include generator objects, async generator objects, and coroutine objects, all of which store their code object in a special attribute.

## Lambda
A lambda function is a small anonymous function.
A lambda function can take any number of arguments, but can only have one expression.

```py
import dis
add_two = lambda x:x+2
print(add_two(234))
sum = lambda x,y:x+y

print(sum(4,5))

dis.dis(sum)

dis.show_code(sum)
```

```sh
(base) pradeep:~$/usr/local/bin/python3 /Users/pradeep/LearnPython/demo_arg.py
236
9
 36           0 RESUME                   0
              2 LOAD_FAST                0 (x)
              4 LOAD_FAST                1 (y)
              6 BINARY_OP                0 (+)
             10 RETURN_VALUE
Name:              <lambda>
Filename:          /Users/pradeep/LearnPython/demo_arg.py
Argument count:    2
Positional-only arguments: 0
Kw-only arguments: 0
Number of locals:  2
Stack size:        2
Flags:             OPTIMIZED, NEWLOCALS
Constants:
   0: None
Variable names:
   0: x
   1: y
(base) pradeep:~$
```

```py
x = lambda a, b, c : a * b + c
print(x(5, 6, 2))
```
```sh
(base) pradeep:~$/usr/local/bin/python3 /Users/pradeep/LearnPython/demo_arg.py
32
(base) pradeep:~$
```
