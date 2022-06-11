---
layout: single
title:  "Python Programming Basics"
date:   2022-06-11 01:56:04 +0530
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
  - title: "Blog"
   
    text: "Checkout other topics"
    nav: my-sidebar

---

# Introduction

Python is being used everywhere. While working in AWS, we work with `boto3`, in quantum computing, `qiskit`, and in IoT, we use `micropython` to control Micro Controllers, for web development we use `flask` or `django` and for mobile app development, we use `kivy`. For data sciense and AI/ML we use `Pytorch`, `TensorFlow`and for Big Data, we use `PySpark`. For DevOps, most of the tools such as `Ansible` use Python.

[www.python.org](https://www.python.org) is the official reference.

To interact with the operating system, we write instructions in a programming language of choice.

We store these intstructions in a program file. When we execute the program, it is loaded into RAM and becomes a process.

CPU will take the instructions from the process and run them one at a time.

## Python as a Programming Language

Download and install `Python 3.9` from the [Anaconda Distribution](https://www.anaconda.com/products/distribution).

Before installing the latest version from Anaconda,

```python
pradeep:~$python -V
Python 2.7.18
pradeep:~$
```

After installing Anaconda,

```python
(base) pradeep:~$python -V
Python 3.9.12
(base) pradeep:~$
```

Launch Python, by typing the `python` keyword.

"How are you?", you will get  a `SyntaxError`.

```python
>>> how are you?
  File "<stdin>", line 1
    how are you?
        ^
SyntaxError: invalid syntax
>>> 
```

````python
Last login: Sat Jun 11 14:52:13 on ttys001
(base) pradeep:~$python3
Python 3.9.12 (main, Apr  5 2022, 01:53:17) 
[Clang 12.0.0 ] :: Anaconda, Inc. on darwin
Type "help", "copyright", "credits" or "license" for more information.
>>> 

````

If we try to assign a variable to some other variable, which is not defined, you get `NameError`.

```python
>>> x=5
>>> x=pradeep
Traceback (most recent call last):
  File "<stdin>", line 1, in <module>
NameError: name 'pradeep' is not defined
>>> 
```

```python
>>> print("Hello World!")
Hello World!
>>> 
```

Print on screen, pop-up, printer, or whatsapp message etc.

## Variables

Datatypes for example. Numbers

`bottle = 5`

`print(bottle)`

If we try to assign `bottle = pradeep`

NameError, name `pradeep` is not defined.

> In Python, single quote and double quote are same.

We dont have to define the data type of the variable first. So, Python is not `strongly typed`. we can change the data type of the varaible on the fly  (`type casting` , `type conversion`, `type inference`).
### Integers
```python
>>> x
5
>>> type(x)
<class 'int'>
>>> 
```
### Strings
```python
>>> x="pradeep"
>>> x
'pradeep'
>>> type(x)
<class 'str'>
>>> 
```
### Floating Point
```python
>>> x=3.14
>>> x
3.14
>>> type(x)
<class 'float'>
>>> 

```
### Boolean
```python
>>> x=True
>>> x
True
>>> type(x)
<class 'bool'>
>>> 
```

## How to view the variable data in RAM?

How to see this data assignment on the RAM? when you assigned a value to a variable, like `x=5`, verify the data in the RAM.

When you close the program, is that data lost from the RAM?

```python
>>> x=5
>>> x
5
>>> exit()
(base) pradeep:~$python3
Python 3.9.12 (main, Apr  5 2022, 01:53:17) 
[Clang 12.0.0 ] :: Anaconda, Inc. on darwin
Type "help", "copyright", "credits" or "license" for more information.
>>> x
Traceback (most recent call last):
  File "<stdin>", line 1, in <module>
NameError: name 'x' is not defined
>>> exit()
```

Keep your mind open to use your programming skills to solve real world challenges.



Eclipse, Visual Studio Code, Atom, Notepad, and many more options to write our code.

```pyhton
>>> x
'pradeep'
>>> 
```

The `>>>` symbol stands for `REPL`.

`REPL` stands fro READ, PRINT, EVALUATE/EXECUTE, LOOP



## Data Structures

Data structure is different from data type.

### List

To organize similar kinds of data in a structure, we have `list`, also known as `array` in other programming languages.

It is pre-available data structure. Custom datastructures also possible.

```python
>>> x="pradeep"
>>> x
'pradeep'
>>> x="suma"
>>> x
'suma'
>>> x=["pradeep","suma"]
>>> x
['pradeep', 'suma']
>>> type(x)
<class 'list'>
>>> 
```

It is possible to store different data types in a single list.

```python
>>> x=[1,"linux","pradeep",3.14]
>>> x
[1, 'linux', 'pradeep', 3.14]
>>> 
```



Every item is given an index number, index number starts from `0`

```python
>>> x=["pradeep","suma"]
>>> x[0]
'pradeep'
>>> x[1]
'suma'
>>> 
```



`IndexError`, when we try to call an index which does not exist

```python
>>> x[2]
Traceback (most recent call last):
  File "<stdin>", line 1, in <module>
IndexError: list index out of range
>>> 
```

Slicing 

```python
>>> x=["pradeep","suma","dakshita","sripragnya","gadde"]
>>> x[0:3]
['pradeep', 'suma', 'dakshita']
>>> x[0:4]
['pradeep', 'suma', 'dakshita', 'sripragnya']
>>> x[0:5]
['pradeep', 'suma', 'dakshita', 'sripragnya', 'gadde']
>>> 

```

```python
>>> x[:5]
['pradeep', 'suma', 'dakshita', 'sripragnya', 'gadde']
>>> 
```

```python
>>> x[2:]
['dakshita', 'sripragnya', 'gadde']
>>> 
```

```python
>>> x[1:3]
['suma', 'dakshita']
>>> 
```

Some default functions exist with every variable, to list them use the `dir()` .

In our case, `x` is a variable of type `list`.

```python
>>> type(x)
<class 'list'>
>>> 
```

Methods or Functions available with this data type, `list`; these are pre-created in Python, we dont have to define any functions, these are defaults.

```python
>>> dir(x)
['__add__', '__class__', '__class_getitem__', '__contains__', '__delattr__', '__delitem__', '__dir__', '__doc__', '__eq__', '__format__', '__ge__', '__getattribute__', '__getitem__', '__gt__', '__hash__', '__iadd__', '__imul__', '__init__', '__init_subclass__', '__iter__', '__le__', '__len__', '__lt__', '__mul__', '__ne__', '__new__', '__reduce__', '__reduce_ex__', '__repr__', '__reversed__', '__rmul__', '__setattr__', '__setitem__', '__sizeof__', '__str__', '__subclasshook__', 'append', 'clear', 'copy', 'count', 'extend', 'index', 'insert', 'pop', 'remove', 'reverse', 'sort']
>>> 

```

Abstract Data Types (ADT) also possible in Python. We will see them later.

CRUD

Create

Update

Delete

Read

```python
>>> x.append("spdsp")
>>> x.append("linux")
>>> x
['pradeep', 'suma', 'dakshita', 'sripragnya', 'gadde', 'spdsp', 'linux']
>>> 

```

```python
>>> x[2:]
['dakshita', 'sripragnya', 'gadde', 'spdsp', 'linux']
>>> 
```

To know how to use a function, use the `help`

```python
>>> help(x.index)

Help on built-in function index:

index(value, start=0, stop=9223372036854775807, /) method of builtins.list instance
    Return first index of value.
    
    Raises ValueError if the value is not present.
```

```python
>>> x.index("spdsp")
5
>>> 
```

```python
>>> x[5]
'spdsp'
>>> 

```

It is possible to update entries in the list

```python
>>> x[6]="python"
>>> x
['pradeep', 'suma', 'dakshita', 'sripragnya', 'gadde', 'spdsp', 'python']
>>> 
```

Index works like a Linear Search algorithm.

## Inline Programming Example

Other way to update, in single step ,instead of two steps.  Search for Functional programming! Use shorter code, or inline programming. Inner code is executed first.

```python
>>> x[x.index("python")]="awesome"
>>> x
['pradeep', 'suma', 'dakshita', 'sripragnya', 'gadde', 'spdsp', 'awesome']
>>> 
```

## IDE

Integrated Development Environment

Anaconda IDE: `Jupiter`

In Jupyter notesbooks, we dont have to use `dir` , instead we can use the dot (`.`) after variable name, to show the available methods.

Use `Shift + tab` to show the help options

## Strings

Strings also work like `list`.  A list of characters.

```python
>>> s="Juniper Networks"
>>> s[0]
'J'
>>> dir(s)
['__add__', '__class__', '__contains__', '__delattr__', '__dir__', '__doc__', '__eq__', '__format__', '__ge__', '__getattribute__', '__getitem__', '__getnewargs__', '__gt__', '__hash__', '__init__', '__init_subclass__', '__iter__', '__le__', '__len__', '__lt__', '__mod__', '__mul__', '__ne__', '__new__', '__reduce__', '__reduce_ex__', '__repr__', '__rmod__', '__rmul__', '__setattr__', '__sizeof__', '__str__', '__subclasshook__', 'capitalize', 'casefold', 'center', 'count', 'encode', 'endswith', 'expandtabs', 'find', 'format', 'format_map', 'index', 'isalnum', 'isalpha', 'isascii', 'isdecimal', 'isdigit', 'isidentifier', 'islower', 'isnumeric', 'isprintable', 'isspace', 'istitle', 'isupper', 'join', 'ljust', 'lower', 'lstrip', 'maketrans', 'partition', 'removeprefix', 'removesuffix', 'replace', 'rfind', 'rindex', 'rjust', 'rpartition', 'rsplit', 'rstrip', 'split', 'splitlines', 'startswith', 'strip', 'swapcase', 'title', 'translate', 'upper', 'zfill']
>>> 

```

```python
>>> help(s.capitalize)

Help on built-in function capitalize:

capitalize() method of builtins.str instance
    Return a capitalized version of the string.
    
    More specifically, make the first character have upper case and the rest lower
    case.
(END)

```

```python
>>> s.capitalize()
'Juniper networks'
>>> 
```



```python
>>> s.endswith("d")
False
>>> s.endswith("s")
True
>>> 
```

An example to demonstrate string methods:

```python
>>> emails=["abc@xyz.com", "def@ghi.com", "ghi@gmail.com"]
>>> emails[0]
'abc@xyz.com'
>>> emails[0].endswith("gmail.com") ## Single line method
False
>>> emails[0].endswith("xyz.com")
True
>>> 

```

```python
>>> emails.append("jkl@mno.com")
>>> emails.append("xyz@abc.com")

>>> emails
['abc@xyz.com', 'def@ghi.com', 'ghi@gmail.com', 'jkl@mno.com', 'xyz@abc.com']
>>> 

```

```python
>>> emails[:5]
['abc@xyz.com', 'def@ghi.com', 'ghi@gmail.com', 'jkl@mno.com', 'xyz@abc.com']
>>> emails[2:4]
['ghi@gmail.com', 'jkl@mno.com']
>>> 
```

This usage is same as the name of the list itself, starts from 0, go till the end, in steps of 1 by default.

Start, end, jump step size

```python
>>> emails[:]
['abc@xyz.com', 'def@ghi.com', 'ghi@gmail.com', 'jkl@mno.com', 'xyz@abc.com']
>>> 
```



```python
>>> emails[1:5:2]
['def@ghi.com', 'jkl@mno.com']
>>> 

```

Another example, without specifying the start, end but the step size, to get every alternate item

```python
>>> emails[::2]
['abc@xyz.com', 'ghi@gmail.com', 'xyz@abc.com']
>>> 
```

To reverse the list, start from beginning, go till end, but  step size of `-1` to start from reverse

```python
>>> emails[::-1]
['xyz@abc.com', 'jkl@mno.com', 'ghi@gmail.com', 'def@ghi.com', 'abc@xyz.com']
>>> 
```

```python
>>> x[::-1]
['linux', 'spdsp', 'gadde', 'sripragnya', 'dakshita', 'suma', 'pradeep']
>>> 
```

This is because, last item index number is `-1`.

````sh
0 + -1 = -1
-1 + -1 = -2
-2 + -1 = -3
and so on.
````



```python
>>> emails[-1]
'xyz@abc.com'
>>> 
```

```python
>>> emails[-3:]
['ghi@gmail.com', 'jkl@mno.com', 'xyz@abc.com']
>>> 

```

```python
>>> """example multi line 
... comments 
... in python
... """
'example multi line \ncomments \nin python\n'
>>> 

```

There is a `reverse` method as well, but this actually changes the data. Items are updated in the real list.

```python
>>> emails
['abc@xyz.com', 'def@ghi.com', 'ghi@gmail.com', 'jkl@mno.com', 'xyz@abc.com']
>>> emails.reverse()
>>> emails
['xyz@abc.com', 'jkl@mno.com', 'ghi@gmail.com', 'def@ghi.com', 'abc@xyz.com']
>>> 

```

This reverse function is not available for `string` variables. Its available only for `lists`.

## In (Containment)

```python
>>> s="Juniper Networks is Awesome!"
>>> s
'Juniper Networks is Awesome!'
>>> "Awesome" in s
True
>>> "Downtime" in s
False
>>> 

```

```python
>>> find="Awesome" in s
>>> type(find)
<class 'bool'>
>>> 
```

## Swapping

In other programming languages, we might need a temporary third variable.

```python
>>> x=5
>>> y=10
>>> z=y
>>> y=x
>>> x=z
>>> x
10
>>> y
5
>>> 

```

We can print multiple variables like this 

```python
>>> x,y
(10, 5)
>>> 
```

To swap, in a single line , very intuitive way, in Python. We dont need the third variable

```python
>>> x,y=y,x
>>> x,y
(5, 10)
>>> 

```

Multiple variable assignment in a single line 

```python
>>> i=2
>>> j=3
>>> k=4
>>> i,j,k
(2, 3, 4)
>>> i,j,k=6,7,8 # Single line method
>>> i,j,k
(6, 7, 8)
>>> 

```

```python
>>> print(i,j,k)
6 7 8
>>> 
```

### List Unpacking

Assigning each item of the list to a different variable

```python
>>> mylist=[2,3,4,5]
>>> i,j,k,l=mylist # Unpacking
>>> print(i,j,k,l)
2 3 4 5
>>> 
```

## Underscore (`_`) Variable 

To store the last output

```python
>>> x = 5+7
>>> x
12
>>> _
12
>>> 
```

```python
>>> x=8*9
>>> x
72
>>> _
72
>>> 
```
We can use other operators with the `_` variable name
```python
>>> _
72
>>> _+2
74
>>> 

```

## Multiplication and Power

```python
>>> 9 * 2
18
>>> 9**2
81
>>> 
```

## ValueError while Unpacking
```python
>>> mylist
[2, 3, 4, 5]
>>> i,j,k=mylist
Traceback (most recent call last):
  File "<stdin>", line 1, in <module>
ValueError: too many values to unpack (expected 3)
>>> 

```

## One more use case of Underscore

If you don't want all variables while unpacking, For example, out of the four items, let us say, we dont need to save the first item in a variable, we can use the `_` while unpacking. 

```python
>>> _,i,j,k=mylist
>>> i
3
>>> j
4
>>> k
5
>>> 
>>> _
2
>>> 
```

We can use it multiple times also,

```python
>>> _,i,_,j=mylist
>>> i
3
>>> j
5
>>> _
4
>>> 

```

## Repetition
Two methods:  Iteration or Recursion
Iteration: For, while etc
Recursion: Calling the same function within a function

```python
>>> print("Hi"*10)
HiHiHiHiHiHiHiHiHiHi
>>> 

```

