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

Slices

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

Abstract Data types also possible in Python. We will see them later.



