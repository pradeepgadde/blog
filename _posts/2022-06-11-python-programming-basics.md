---
layout: single
title:  "Python Programming Basics Day#1"
date:   2022-06-11 18:56:04 +0530
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
This weekend, I have attended a training from Linux World (#13) on Python Programming (Basic to Advanced). Its a free training, check out [https://learning.hash13.com](https://learning.hash13.com) for more details.

This is my notes from that session. 

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

## List Unpacking

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

## For Loop
```python
>>> mylist
[2, 3, 4, 5]
>>> for i in mylist:
...     print(i)
... 
2
3
4
5
>>> 
We can also perform some operation on each item
```python
>>> for i in mylist:
...     print (i*3)
... 
6
9
12
15
>>> 
```
How to send email using Python?  Find out!




```python
>>> print("Hi"*10)
HiHiHiHiHiHiHiHiHiHi
>>> 

```

## Conditional Statements
A block of code is identified with indentation (Spacing). Increases readability.

### IndentationError

```python
>>> for i in mylist:
... print(i)
  File "<stdin>", line 2
    print(i)
    ^
IndentationError: expected an indented block
>>> 
```

## If-Else

```python
>>> temp
50 
>>> if temp>40:
...     print("Hot")
... else:
...     print("Cold")
... 
Hot
>>> if True:
...     print("Hi")
... else:
...     print("Bye")
... 
Hi
>>> if False:
...     print("Hi")
... else:
...     print("Bye")
... 
Bye
>>>

```

If Function condition always returns/looks for a Boolean value.

## Bool Function

```python
>>> bool("Pradeep")
True
>>> bool(67)
True
>>> bool(0) ## Only for zero, `False` returned
False
>>> bool(-45)
True
>>> bool("xyz")
True
>>> 

```

```python
>>> bool(print("Pradeep"))
Pradeep
False
>>> 

```
Print function does not return anything, and hence False.
## Not

```python
>>> not bool(print("Pradeep"))
Pradeep
True
```

```python
>>> not True
False
>>> not False
True
>>> 
```

```python
>>> if not bool(print("Juniper")):
...             print("Juniper")
... else:
...             print("Networks")
... 
Juniper
Juniper
>>> if bool(print("Juniper")):
...     print("Juniper")
... else:
...     print("Networks")
... 
Juniper
Networks
>>> 

```

## Print

`\n` new line character, default at the end of the `print` . There is an argument called `end` in the print function with default value.

`\t` tab

```python
>>> s="Juniper Networks\n is Awesome!"
>>> s
'Juniper Networks\n is Awesome!'
>>> print(s)
Juniper Networks
 is Awesome!
>>> 
```

```python
>>> s="Juniper"
>>> print(s)
Juniper
>>> print(s, end=".....")
Juniper.....>>> 

```

```python
>>> print(s, end="")
Juniper>>> 
```

Go through all emails, and print only gmail IDs
```python
>>> emails
['xyz@abc.com', 'jkl@mno.com', 'ghi@gmail.com', 'def@ghi.com', 'abc@xyz.com']
>>> for e in emails:
...     if e.endswith("gmail.com"):
...             print(e)
... 
ghi@gmail.com
>>> 

```

```python
>>> mobiles
['+919900977110', '+19900977110']
>>> for m in mobiles:
...     if m.startswith("+91"):
...             print(m)
... 
+919900977110
>>> 

```

## Libraries

Let us say, you want to send a whatsapp message, you install a Library `pywhatkit`.

Every library comes with several modules.

Use the `pip install pywhatkit` command.

```python
(base) pradeep:~$pip install pywhatkit
Collecting pywhatkit
  Downloading pywhatkit-5.3-py3-none-any.whl (15 kB)
Requirement already satisfied: requests in /Users/pradeep/opt/anaconda3/lib/python3.9/site-packages (from pywhatkit) (2.27.1)
Requirement already satisfied: Pillow in /Users/pradeep/opt/anaconda3/lib/python3.9/site-packages (from pywhatkit) (9.0.1)
Collecting pyautogui
  Downloading PyAutoGUI-0.9.53.tar.gz (59 kB)
     |████████████████████████████████| 59 kB 2.2 MB/s 
Collecting wikipedia
  Downloading wikipedia-1.4.0.tar.gz (27 kB)
Collecting pymsgbox
  Downloading PyMsgBox-1.0.9.tar.gz (18 kB)
  Installing build dependencies ... done
  Getting requirements to build wheel ... done
    Preparing wheel metadata ... done
Collecting PyTweening>=1.0.1
  Downloading pytweening-1.0.4.tar.gz (14 kB)
Collecting pyscreeze>=0.1.21
  Downloading PyScreeze-0.1.28.tar.gz (25 kB)
  Installing build dependencies ... done
  Getting requirements to build wheel ... done
    Preparing wheel metadata ... done
Collecting pygetwindow>=0.0.5
  Downloading PyGetWindow-0.0.9.tar.gz (9.7 kB)
Collecting mouseinfo
  Downloading MouseInfo-0.1.3.tar.gz (10 kB)
Collecting pyobjc-core
  Downloading pyobjc_core-8.5-cp39-cp39-macosx_10_9_universal2.whl (578 kB)
     |████████████████████████████████| 578 kB 2.5 MB/s 
Collecting pyobjc
  Downloading pyobjc-8.5-py3-none-any.whl (3.8 kB)
Collecting pyrect
  Downloading PyRect-0.2.0.tar.gz (17 kB)
Collecting pyperclip
  Downloading pyperclip-1.8.2.tar.gz (20 kB)
Collecting rubicon-objc
  Downloading rubicon_objc-0.4.2-py3-none-any.whl (58 kB)
     |████████████████████████████████| 58 kB 2.6 MB/s 
Collecting pyobjc-framework-GameplayKit==8.5
  Downloading pyobjc_framework_GameplayKit-8.5-cp36-abi3-macosx_10_9_x86_64.whl (8.4 kB)
Collecting pyobjc-framework-SyncServices==8.5
  Downloading pyobjc_framework_SyncServices-8.5-cp36-abi3-macosx_10_9_x86_64.whl (10 kB)
Collecting pyobjc-framework-DiscRecordingUI==8.5
  Downloading pyobjc_framework_DiscRecordingUI-8.5-py2.py3-none-any.whl (4.2 kB)
Collecting pyobjc-framework-ExecutionPolicy==8.5
  Downloading pyobjc_framework_ExecutionPolicy-8.5-py2.py3-none-any.whl (3.3 kB)
Collecting pyobjc-framework-ImageCaptureCore==8.5
  Downloading pyobjc_framework_ImageCaptureCore-8.5-cp36-abi3-macosx_10_9_x86_64.whl (12 kB)
Collecting pyobjc-framework-UserNotifications==8.5
  Downloading pyobjc_framework_UserNotifications-8.5-cp36-abi3-macosx_10_9_x86_64.whl (7.3 kB)
Collecting pyobjc-framework-MailKit==8.5
  Downloading pyobjc_framework_MailKit-8.5-py2.py3-none-any.whl (4.0 kB)
Collecting pyobjc-framework-ExceptionHandling==8.5
  Downloading pyobjc_framework_ExceptionHandling-8.5-py2.py3-none-any.whl (7.4 kB)
Collecting pyobjc-framework-LocalAuthentication==8.5
  Downloading pyobjc_framework_LocalAuthentication-8.5-py2.py3-none-any.whl (5.0 kB)
Collecting pyobjc-framework-InstallerPlugins==8.5
  Downloading pyobjc_framework_InstallerPlugins-8.5-py2.py3-none-any.whl (4.3 kB)
Collecting pyobjc-framework-CoreData==8.5
  Downloading pyobjc_framework_CoreData-8.5-cp36-abi3-macosx_10_9_x86_64.whl (13 kB)
Collecting pyobjc-framework-SoundAnalysis==8.5
  Downloading pyobjc_framework_SoundAnalysis-8.5-py2.py3-none-any.whl (3.5 kB)
Collecting pyobjc-framework-AppleScriptObjC==8.5
  Downloading pyobjc_framework_AppleScriptObjC-8.5-py2.py3-none-any.whl (3.9 kB)
Collecting pyobjc-framework-AutomaticAssessmentConfiguration==8.5
  Downloading pyobjc_framework_AutomaticAssessmentConfiguration-8.5-cp36-abi3-macosx_10_9_x86_64.whl (6.4 kB)
Collecting pyobjc-framework-NotificationCenter==8.5
  Downloading pyobjc_framework_NotificationCenter-8.5-cp36-abi3-macosx_10_9_x86_64.whl (7.2 kB)
Collecting pyobjc-framework-SystemExtensions==8.5
  Downloading pyobjc_framework_SystemExtensions-8.5-cp36-abi3-macosx_10_9_x86_64.whl (6.4 kB)
Collecting pyobjc-framework-SafariServices==8.5
  Downloading pyobjc_framework_SafariServices-8.5-cp36-abi3-macosx_10_9_x86_64.whl (5.8 kB)
Collecting pyobjc-framework-GameController==8.5
  Downloading pyobjc_framework_GameController-8.5-cp36-abi3-macosx_10_9_x86_64.whl (9.2 kB)
Collecting pyobjc-framework-Intents==8.5
  Downloading pyobjc_framework_Intents-8.5-cp36-abi3-macosx_10_9_x86_64.whl (21 kB)
Collecting pyobjc-framework-Metal==8.5
  Downloading pyobjc_framework_Metal-8.5-cp36-abi3-macosx_10_9_x86_64.whl (32 kB)
Collecting pyobjc-framework-EventKit==8.5
  Downloading pyobjc_framework_EventKit-8.5-py2.py3-none-any.whl (5.9 kB)
Collecting pyobjc-framework-DeviceCheck==8.5
  Downloading pyobjc_framework_DeviceCheck-8.5-py2.py3-none-any.whl (3.2 kB)
Collecting pyobjc-framework-Photos==8.5
  Downloading pyobjc_framework_Photos-8.5-cp36-abi3-macosx_10_9_x86_64.whl (9.2 kB)
Collecting pyobjc-framework-ShazamKit==8.5
  Downloading pyobjc_framework_ShazamKit-8.5-cp39-cp39-macosx_10_9_universal2.whl (9.0 kB)
Collecting pyobjc-framework-CoreAudio==8.5
  Downloading pyobjc_framework_CoreAudio-8.5-cp39-cp39-macosx_10_9_universal2.whl (35 kB)
Collecting pyobjc-framework-MediaAccessibility==8.5
  Downloading pyobjc_framework_MediaAccessibility-8.5-py2.py3-none-any.whl (3.9 kB)
Collecting pyobjc-framework-LatentSemanticMapping==8.5
  Downloading pyobjc_framework_LatentSemanticMapping-8.5-py2.py3-none-any.whl (4.9 kB)
Collecting pyobjc-framework-PencilKit==8.5
  Downloading pyobjc_framework_PencilKit-8.5-py2.py3-none-any.whl (3.2 kB)
Collecting pyobjc-framework-ServiceManagement==8.5
  Downloading pyobjc_framework_ServiceManagement-8.5-py2.py3-none-any.whl (4.5 kB)
Collecting pyobjc-framework-PhotosUI==8.5
  Downloading pyobjc_framework_PhotosUI-8.5-cp36-abi3-macosx_10_9_x86_64.whl (7.8 kB)
Collecting pyobjc-framework-CoreML==8.5
  Downloading pyobjc_framework_CoreML-8.5-cp36-abi3-macosx_10_9_x86_64.whl (8.2 kB)
Collecting pyobjc-framework-AdServices==8.5
  Downloading pyobjc_framework_AdServices-8.5-py2.py3-none-any.whl (3.0 kB)
Collecting pyobjc-framework-Security==8.5
  Downloading pyobjc_framework_Security-8.5-cp39-cp39-macosx_10_9_universal2.whl (40 kB)
     |████████████████████████████████| 40 kB 4.9 MB/s 
Collecting pyobjc-framework-KernelManagement==8.5
  Downloading pyobjc_framework_KernelManagement-8.5-py2.py3-none-any.whl (3.2 kB)
Collecting pyobjc-framework-AVFoundation==8.5
  Downloading pyobjc_framework_AVFoundation-8.5-cp36-abi3-macosx_10_9_x86_64.whl (45 kB)
     |████████████████████████████████| 45 kB 4.3 MB/s 
Collecting pyobjc-framework-CoreLocation==8.5
  Downloading pyobjc_framework_CoreLocation-8.5-cp36-abi3-macosx_10_9_x86_64.whl (9.3 kB)
Collecting pyobjc-framework-IMServicePlugIn==8.5
  Downloading pyobjc_framework_IMServicePlugIn-8.5-cp36-abi3-macosx_10_9_x86_64.whl (9.5 kB)
Collecting pyobjc-framework-CalendarStore==8.5
  Downloading pyobjc_framework_CalendarStore-8.5-py2.py3-none-any.whl (4.6 kB)
Collecting pyobjc-framework-ModelIO==8.5
  Downloading pyobjc_framework_ModelIO-8.5-cp36-abi3-macosx_10_9_x86_64.whl (13 kB)
Collecting pyobjc-framework-PassKit==8.5
  Downloading pyobjc_framework_PassKit-8.5-cp36-abi3-macosx_10_9_x86_64.whl (9.5 kB)
Collecting pyobjc-framework-SpriteKit==8.5
  Downloading pyobjc_framework_SpriteKit-8.5-cp39-cp39-macosx_10_9_universal2.whl (17 kB)
Collecting pyobjc-framework-CoreText==8.5
  Downloading pyobjc_framework_CoreText-8.5-cp39-cp39-macosx_10_9_universal2.whl (30 kB)
Collecting pyobjc-framework-CoreAudioKit==8.5
  Downloading pyobjc_framework_CoreAudioKit-8.5-cp36-abi3-macosx_10_9_x86_64.whl (5.4 kB)
Collecting pyobjc-framework-VideoSubscriberAccount==8.5
  Downloading pyobjc_framework_VideoSubscriberAccount-8.5-py2.py3-none-any.whl (3.8 kB)
Collecting pyobjc-framework-CoreBluetooth==8.5
  Downloading pyobjc_framework_CoreBluetooth-8.5-cp36-abi3-macosx_10_9_x86_64.whl (9.9 kB)
Collecting pyobjc-framework-DataDetection==8.5
  Downloading pyobjc_framework_DataDetection-8.5-py2.py3-none-any.whl (3.0 kB)
Collecting pyobjc-framework-CoreMotion==8.5
  Downloading pyobjc_framework_CoreMotion-8.5-py2.py3-none-any.whl (4.6 kB)
Collecting pyobjc-framework-AppleScriptKit==8.5
  Downloading pyobjc_framework_AppleScriptKit-8.5-py2.py3-none-any.whl (3.8 kB)
Collecting pyobjc-framework-Quartz==8.5
  Downloading pyobjc_framework_Quartz-8.5-cp39-cp39-macosx_10_9_universal2.whl (226 kB)
     |████████████████████████████████| 226 kB 587 kB/s 
Collecting pyobjc-framework-GameCenter==8.5
  Downloading pyobjc_framework_GameCenter-8.5-cp36-abi3-macosx_10_9_x86_64.whl (12 kB)
Collecting pyobjc-framework-SystemConfiguration==8.5
  Downloading pyobjc_framework_SystemConfiguration-8.5-cp36-abi3-macosx_10_9_x86_64.whl (16 kB)
Collecting pyobjc-framework-MediaToolbox==8.5
  Downloading pyobjc_framework_MediaToolbox-8.5-cp36-abi3-macosx_10_9_x86_64.whl (8.3 kB)
Collecting pyobjc-framework-Automator==8.5
  Downloading pyobjc_framework_Automator-8.5-py2.py3-none-any.whl (5.0 kB)
Collecting pyobjc-framework-NetFS==8.5
  Downloading pyobjc_framework_NetFS-8.5-py2.py3-none-any.whl (3.7 kB)
Collecting pyobjc-framework-ColorSync==8.5
  Downloading pyobjc_framework_ColorSync-8.5-py2.py3-none-any.whl (5.4 kB)
Collecting pyobjc-framework-AdSupport==8.5
  Downloading pyobjc_framework_AdSupport-8.5-py2.py3-none-any.whl (2.9 kB)
Collecting pyobjc-framework-CoreServices==8.5
  Downloading pyobjc_framework_CoreServices-8.5-cp36-abi3-macosx_10_9_x86_64.whl (27 kB)
Collecting pyobjc-framework-OpenDirectory==8.5
  Downloading pyobjc_framework_OpenDirectory-8.5-py2.py3-none-any.whl (11 kB)
Collecting pyobjc-framework-SearchKit==8.5
  Downloading pyobjc_framework_SearchKit-8.5-py2.py3-none-any.whl (3.3 kB)
Collecting pyobjc-framework-DiskArbitration==8.5
  Downloading pyobjc_framework_DiskArbitration-8.5-py2.py3-none-any.whl (4.4 kB)
Collecting pyobjc-framework-CoreWLAN==8.5
  Downloading pyobjc_framework_CoreWLAN-8.5-cp36-abi3-macosx_10_9_x86_64.whl (8.1 kB)
Collecting pyobjc-framework-FileProvider==8.5
  Downloading pyobjc_framework_FileProvider-8.5-cp39-cp39-macosx_10_9_universal2.whl (17 kB)
Collecting pyobjc-framework-DiscRecording==8.5
  Downloading pyobjc_framework_DiscRecording-8.5-cp36-abi3-macosx_10_9_x86_64.whl (12 kB)
Collecting pyobjc-framework-DictionaryServices==8.5
  Downloading pyobjc_framework_DictionaryServices-8.5-py2.py3-none-any.whl (3.4 kB)
Collecting pyobjc-framework-WebKit==8.5
  Downloading pyobjc_framework_WebKit-8.5-cp36-abi3-macosx_10_9_x86_64.whl (29 kB)
Collecting pyobjc-framework-ContactsUI==8.5
  Downloading pyobjc_framework_ContactsUI-8.5-cp36-abi3-macosx_10_9_x86_64.whl (5.7 kB)
Collecting pyobjc-framework-AudioVideoBridging==8.5
  Downloading pyobjc_framework_AudioVideoBridging-8.5-py2.py3-none-any.whl (6.3 kB)
Collecting pyobjc-framework-InstantMessage==8.5
  Downloading pyobjc_framework_InstantMessage-8.5-py2.py3-none-any.whl (4.9 kB)
Collecting pyobjc-framework-ClassKit==8.5
  Downloading pyobjc_framework_ClassKit-8.5-cp36-abi3-macosx_10_9_x86_64.whl (6.3 kB)
Collecting pyobjc-framework-Speech==8.5
  Downloading pyobjc_framework_Speech-8.5-cp36-abi3-macosx_10_9_x86_64.whl (6.4 kB)
Collecting pyobjc-framework-AddressBook==8.5
  Downloading pyobjc_framework_AddressBook-8.5-cp36-abi3-macosx_10_9_x86_64.whl (10 kB)
Collecting pyobjc-framework-CloudKit==8.5
  Downloading pyobjc_framework_CloudKit-8.5-py2.py3-none-any.whl (7.8 kB)
Collecting pyobjc-framework-DVDPlayback==8.5
  Downloading pyobjc_framework_DVDPlayback-8.5-py2.py3-none-any.whl (7.8 kB)
Collecting pyobjc-framework-CryptoTokenKit==8.5
  Downloading pyobjc_framework_CryptoTokenKit-8.5-cp36-abi3-macosx_10_9_x86_64.whl (9.3 kB)
Collecting pyobjc-framework-ScreenTime==8.5
  Downloading pyobjc_framework_ScreenTime-8.5-py2.py3-none-any.whl (3.1 kB)
Collecting pyobjc-framework-LaunchServices==8.5
  Downloading pyobjc_framework_LaunchServices-8.5-py2.py3-none-any.whl (3.4 kB)
Collecting pyobjc-framework-OSAKit==8.5
  Downloading pyobjc_framework_OSAKit-8.5-py2.py3-none-any.whl (3.6 kB)
Collecting pyobjc-framework-AppTrackingTransparency==8.5
  Downloading pyobjc_framework_AppTrackingTransparency-8.5-py2.py3-none-any.whl (3.3 kB)
Collecting pyobjc-framework-AVKit==8.5
  Downloading pyobjc_framework_AVKit-8.5-cp36-abi3-macosx_10_9_x86_64.whl (8.0 kB)
Collecting pyobjc-framework-IntentsUI==8.5
  Downloading pyobjc_framework_IntentsUI-8.5-cp39-cp39-macosx_10_9_universal2.whl (9.9 kB)
Collecting pyobjc-framework-MultipeerConnectivity==8.5
  Downloading pyobjc_framework_MultipeerConnectivity-8.5-cp36-abi3-macosx_10_9_x86_64.whl (8.8 kB)
Collecting pyobjc-framework-ScriptingBridge==8.5
  Downloading pyobjc_framework_ScriptingBridge-8.5-cp36-abi3-macosx_10_9_x86_64.whl (6.6 kB)
Collecting pyobjc-framework-LocalAuthenticationEmbeddedUI==8.5
  Downloading pyobjc_framework_LocalAuthenticationEmbeddedUI-8.5-py2.py3-none-any.whl (3.1 kB)
Collecting pyobjc-framework-Contacts==8.5
  Downloading pyobjc_framework_Contacts-8.5-cp36-abi3-macosx_10_9_x86_64.whl (9.1 kB)
Collecting pyobjc-framework-SecurityInterface==8.5
  Downloading pyobjc_framework_SecurityInterface-8.5-cp36-abi3-macosx_10_9_x86_64.whl (7.5 kB)
Collecting pyobjc-framework-CallKit==8.5
  Downloading pyobjc_framework_CallKit-8.5-py2.py3-none-any.whl (4.4 kB)
Collecting pyobjc-framework-MetalPerformanceShaders==8.5
  Downloading pyobjc_framework_MetalPerformanceShaders-8.5-cp36-abi3-macosx_10_9_x86_64.whl (19 kB)
Collecting pyobjc-framework-StoreKit==8.5
  Downloading pyobjc_framework_StoreKit-8.5-cp36-abi3-macosx_10_9_x86_64.whl (8.5 kB)
Collecting pyobjc-framework-FSEvents==8.5
  Downloading pyobjc_framework_FSEvents-8.5-cp36-abi3-macosx_10_9_x86_64.whl (8.9 kB)
Collecting pyobjc-framework-CFNetwork==8.5
  Downloading pyobjc_framework_CFNetwork-8.5-cp36-abi3-macosx_10_9_x86_64.whl (12 kB)
Collecting pyobjc-framework-libdispatch==8.5
  Downloading pyobjc_framework_libdispatch-8.5-cp39-cp39-macosx_10_9_universal2.whl (20 kB)
Collecting pyobjc-framework-NaturalLanguage==8.5
  Downloading pyobjc_framework_NaturalLanguage-8.5-py2.py3-none-any.whl (4.4 kB)
Collecting pyobjc-framework-MediaPlayer==8.5
  Downloading pyobjc_framework_MediaPlayer-8.5-py2.py3-none-any.whl (6.0 kB)
Collecting pyobjc-framework-FinderSync==8.5
  Downloading pyobjc_framework_FinderSync-8.5-py2.py3-none-any.whl (4.4 kB)
Collecting pyobjc-framework-MediaLibrary==8.5
  Downloading pyobjc_framework_MediaLibrary-8.5-py2.py3-none-any.whl (3.8 kB)
Collecting pyobjc-framework-BusinessChat==8.5
  Downloading pyobjc_framework_BusinessChat-8.5-py2.py3-none-any.whl (3.0 kB)
Collecting pyobjc-framework-IOSurface==8.5
  Downloading pyobjc_framework_IOSurface-8.5-py2.py3-none-any.whl (4.4 kB)
Collecting pyobjc-framework-ScreenSaver==8.5
  Downloading pyobjc_framework_ScreenSaver-8.5-cp36-abi3-macosx_10_9_x86_64.whl (6.3 kB)
Collecting pyobjc-framework-MetalPerformanceShadersGraph==8.5
  Downloading pyobjc_framework_MetalPerformanceShadersGraph-8.5-py2.py3-none-any.whl (4.9 kB)
Collecting pyobjc-framework-QuickLookThumbnailing==8.5
  Downloading pyobjc_framework_QuickLookThumbnailing-8.5-py2.py3-none-any.whl (3.5 kB)
Collecting pyobjc-framework-ReplayKit==8.5
  Downloading pyobjc_framework_ReplayKit-8.5-cp36-abi3-macosx_10_9_x86_64.whl (7.2 kB)
Collecting pyobjc-framework-SecurityFoundation==8.5
  Downloading pyobjc_framework_SecurityFoundation-8.5-py2.py3-none-any.whl (3.3 kB)
Collecting pyobjc-framework-VideoToolbox==8.5
  Downloading pyobjc_framework_VideoToolbox-8.5-cp36-abi3-macosx_10_9_x86_64.whl (9.5 kB)
Collecting pyobjc-framework-PreferencePanes==8.5
  Downloading pyobjc_framework_PreferencePanes-8.5-py2.py3-none-any.whl (4.3 kB)
Collecting pyobjc-framework-MapKit==8.5
  Downloading pyobjc_framework_MapKit-8.5-cp36-abi3-macosx_10_9_x86_64.whl (14 kB)
Collecting pyobjc-framework-GameKit==8.5
  Downloading pyobjc_framework_GameKit-8.5-cp36-abi3-macosx_10_9_x86_64.whl (15 kB)
Collecting pyobjc-framework-NetworkExtension==8.5
  Downloading pyobjc_framework_NetworkExtension-8.5-cp36-abi3-macosx_10_9_x86_64.whl (10 kB)
Collecting pyobjc-framework-UserNotificationsUI==8.5
  Downloading pyobjc_framework_UserNotificationsUI-8.5-py2.py3-none-any.whl (3.4 kB)
Collecting pyobjc-framework-MLCompute==8.5
  Downloading pyobjc_framework_MLCompute-8.5-py2.py3-none-any.whl (6.1 kB)
Collecting pyobjc-framework-CoreHaptics==8.5
  Downloading pyobjc_framework_CoreHaptics-8.5-py2.py3-none-any.whl (4.6 kB)
Collecting pyobjc-framework-Vision==8.5
  Downloading pyobjc_framework_Vision-8.5-cp36-abi3-macosx_10_9_x86_64.whl (9.9 kB)
Collecting pyobjc-framework-ApplicationServices==8.5
  Downloading pyobjc_framework_ApplicationServices-8.5-cp39-cp39-macosx_10_9_universal2.whl (25 kB)
Collecting pyobjc-framework-PushKit==8.5
  Downloading pyobjc_framework_PushKit-8.5-cp36-abi3-macosx_10_9_x86_64.whl (5.9 kB)
Collecting pyobjc-framework-Virtualization==8.5
  Downloading pyobjc_framework_Virtualization-8.5-cp36-abi3-macosx_10_9_x86_64.whl (6.8 kB)
Collecting pyobjc-framework-Cocoa==8.5
  Downloading pyobjc_framework_Cocoa-8.5-cp39-cp39-macosx_10_9_universal2.whl (388 kB)
     |████████████████████████████████| 388 kB 2.4 MB/s 
Collecting pyobjc-framework-UniformTypeIdentifiers==8.5
  Downloading pyobjc_framework_UniformTypeIdentifiers-8.5-py2.py3-none-any.whl (4.0 kB)
Collecting pyobjc-framework-FileProviderUI==8.5
  Downloading pyobjc_framework_FileProviderUI-8.5-py2.py3-none-any.whl (3.1 kB)
Collecting pyobjc-framework-SceneKit==8.5
  Downloading pyobjc_framework_SceneKit-8.5-cp36-abi3-macosx_10_9_x86_64.whl (21 kB)
Collecting pyobjc-framework-MetalKit==8.5
  Downloading pyobjc_framework_MetalKit-8.5-cp36-abi3-macosx_10_9_x86_64.whl (6.4 kB)
Collecting pyobjc-framework-LinkPresentation==8.5
  Downloading pyobjc_framework_LinkPresentation-8.5-py2.py3-none-any.whl (3.3 kB)
Collecting pyobjc-framework-AuthenticationServices==8.5
  Downloading pyobjc_framework_AuthenticationServices-8.5-cp36-abi3-macosx_10_9_x86_64.whl (9.1 kB)
Collecting pyobjc-framework-ExternalAccessory==8.5
  Downloading pyobjc_framework_ExternalAccessory-8.5-cp36-abi3-macosx_10_9_x86_64.whl (6.5 kB)
Collecting pyobjc-framework-CoreSpotlight==8.5
  Downloading pyobjc_framework_CoreSpotlight-8.5-cp36-abi3-macosx_10_9_x86_64.whl (6.9 kB)
Collecting pyobjc-framework-iTunesLibrary==8.5
  Downloading pyobjc_framework_iTunesLibrary-8.5-py2.py3-none-any.whl (4.7 kB)
Collecting pyobjc-framework-Accessibility==8.5
  Downloading pyobjc_framework_Accessibility-8.5-cp36-abi3-macosx_10_9_x86_64.whl (7.1 kB)
Collecting pyobjc-framework-Collaboration==8.5
  Downloading pyobjc_framework_Collaboration-8.5-py2.py3-none-any.whl (4.4 kB)
Collecting pyobjc-framework-Accounts==8.5
  Downloading pyobjc_framework_Accounts-8.5-py2.py3-none-any.whl (4.6 kB)
Collecting pyobjc-framework-OSLog==8.5
  Downloading pyobjc_framework_OSLog-8.5-cp36-abi3-macosx_10_9_x86_64.whl (5.8 kB)
Collecting pyobjc-framework-MetricKit==8.5
  Downloading pyobjc_framework_MetricKit-8.5-cp39-cp39-macosx_10_9_universal2.whl (8.6 kB)
Collecting pyobjc-framework-Social==8.5
  Downloading pyobjc_framework_Social-8.5-py2.py3-none-any.whl (4.0 kB)
Collecting pyobjc-framework-CoreMedia==8.5
  Downloading pyobjc_framework_CoreMedia-8.5-cp39-cp39-macosx_10_9_universal2.whl (23 kB)
Collecting pyobjc-framework-Network==8.5
  Downloading pyobjc_framework_Network-8.5-cp36-abi3-macosx_10_9_x86_64.whl (14 kB)
Collecting pyobjc-framework-CoreMediaIO==8.5
  Downloading pyobjc_framework_CoreMediaIO-8.5-cp36-abi3-macosx_10_9_x86_64.whl (13 kB)
Collecting pyobjc-framework-CoreMIDI==8.5
  Downloading pyobjc_framework_CoreMIDI-8.5-cp36-abi3-macosx_10_9_x86_64.whl (12 kB)
Collecting pyobjc-framework-InputMethodKit==8.5
  Downloading pyobjc_framework_InputMethodKit-8.5-cp36-abi3-macosx_10_9_x86_64.whl (7.4 kB)
Requirement already satisfied: charset-normalizer~=2.0.0 in /Users/pradeep/opt/anaconda3/lib/python3.9/site-packages (from requests->pywhatkit) (2.0.4)
Requirement already satisfied: urllib3<1.27,>=1.21.1 in /Users/pradeep/opt/anaconda3/lib/python3.9/site-packages (from requests->pywhatkit) (1.26.9)
Requirement already satisfied: idna<4,>=2.5 in /Users/pradeep/opt/anaconda3/lib/python3.9/site-packages (from requests->pywhatkit) (3.3)
Requirement already satisfied: certifi>=2017.4.17 in /Users/pradeep/opt/anaconda3/lib/python3.9/site-packages (from requests->pywhatkit) (2021.10.8)
Requirement already satisfied: beautifulsoup4 in /Users/pradeep/opt/anaconda3/lib/python3.9/site-packages (from wikipedia->pywhatkit) (4.11.1)
Requirement already satisfied: soupsieve>1.2 in /Users/pradeep/opt/anaconda3/lib/python3.9/site-packages (from beautifulsoup4->wikipedia->pywhatkit) (2.3.1)
Building wheels for collected packages: pyautogui, pygetwindow, pyscreeze, PyTweening, mouseinfo, pymsgbox, pyperclip, pyrect, wikipedia
  Building wheel for pyautogui (setup.py) ... done
  Created wheel for pyautogui: filename=PyAutoGUI-0.9.53-py3-none-any.whl size=36614 sha256=94af4d00d262d26fe339cbe520688bb0e6ff6b70d7da88af7adc13d877aa9f5c
  Stored in directory: /Users/pradeep/Library/Caches/pip/wheels/d8/97/e4/d2edca92a87d3b5fbfb527264750a17b4ba297b9a7cab6e67f
  Building wheel for pygetwindow (setup.py) ... done
  Created wheel for pygetwindow: filename=PyGetWindow-0.0.9-py3-none-any.whl size=11081 sha256=1dcebfa6e96557062300630b3da7401f735196f48c54bbd030ea74eab6734a9b
  Stored in directory: /Users/pradeep/Library/Caches/pip/wheels/44/ab/20/423c3a444793767e4e41f8377bc902f77bee212e68dcce85a5
  Building wheel for pyscreeze (PEP 517) ... done
  Created wheel for pyscreeze: filename=PyScreeze-0.1.28-py3-none-any.whl size=13009 sha256=fc9f9c53c25fe45eb2733f5f55bc7001e8feb4dbbcae05a4ae464c53fd5f5722
  Stored in directory: /Users/pradeep/Library/Caches/pip/wheels/a2/5b/86/99f1d8fac5d92de0ccb3f0d4ad15e3f4278baf75a9b0f20b93
  Building wheel for PyTweening (setup.py) ... done
  Created wheel for PyTweening: filename=pytweening-1.0.4-py3-none-any.whl size=5854 sha256=87cafac8e143401ee005da8f1733ed4ff84c09c000edc62c53cc27ffd64cda71
  Stored in directory: /Users/pradeep/Library/Caches/pip/wheels/a4/5d/d2/ba4c8f82163233ffaadcf383c1e34d7d92635d357d13e7b78d
  Building wheel for mouseinfo (setup.py) ... done
  Created wheel for mouseinfo: filename=MouseInfo-0.1.3-py3-none-any.whl size=10906 sha256=5747f4f83425ee14cc02de90f85612a8ae449021b618eaebd6f778e740aa1ddb
  Stored in directory: /Users/pradeep/Library/Caches/pip/wheels/61/73/b9/6fb1131ab36e650206e3aa0ad7a68907b41b32ac2d4f75f543
  Building wheel for pymsgbox (PEP 517) ... done
  Created wheel for pymsgbox: filename=PyMsgBox-1.0.9-py3-none-any.whl size=7406 sha256=fd411b6367bdcf6287c4cb6f43570e40a7561bd567c91b04bafb93347bc00b7b
  Stored in directory: /Users/pradeep/Library/Caches/pip/wheels/7f/13/8c/584c519464297d9637f9cd29fd1dcdf55e2a2cab225c76a2db
  Building wheel for pyperclip (setup.py) ... done
  Created wheel for pyperclip: filename=pyperclip-1.8.2-py3-none-any.whl size=11137 sha256=5cac388a6e05b16b92acacadeb72f856a473d10bb523fa43c4b340ef4b9dc39b
  Stored in directory: /Users/pradeep/Library/Caches/pip/wheels/0c/09/9e/49e21a6840ef7955b06d47394afef0058f0378c0914e48b8b8
  Building wheel for pyrect (setup.py) ... done
  Created wheel for pyrect: filename=PyRect-0.2.0-py2.py3-none-any.whl size=11196 sha256=a87f671bcef627c1c998738b49eea73f60843763eb0a42139b0a07150cfa922e
  Stored in directory: /Users/pradeep/Library/Caches/pip/wheels/25/80/fa/27bb4a1c2e21f64ec71390489d52e57b7cc8afbe79bd595c5e
  Building wheel for wikipedia (setup.py) ... done
  Created wheel for wikipedia: filename=wikipedia-1.4.0-py3-none-any.whl size=11695 sha256=b899940a933b78c54dee29dc16e07c1d9b229a209f9a8d56345d39240ae0d7b2
  Stored in directory: /Users/pradeep/Library/Caches/pip/wheels/c2/46/f4/caa1bee71096d7b0cdca2f2a2af45cacf35c5760bee8f00948
Successfully built pyautogui pygetwindow pyscreeze PyTweening mouseinfo pymsgbox pyperclip pyrect wikipedia
Installing collected packages: pyobjc-core, pyobjc-framework-Cocoa, pyobjc-framework-Security, pyobjc-framework-Quartz, pyobjc-framework-Metal, pyobjc-framework-FSEvents, pyobjc-framework-CoreMedia, pyobjc-framework-CoreAudio, pyobjc-framework-UserNotifications, pyobjc-framework-SpriteKit, pyobjc-framework-MetalPerformanceShaders, pyobjc-framework-LocalAuthentication, pyobjc-framework-Intents, pyobjc-framework-FileProvider, pyobjc-framework-DiscRecording, pyobjc-framework-CoreServices, pyobjc-framework-CoreML, pyobjc-framework-CoreLocation, pyobjc-framework-CoreData, pyobjc-framework-Contacts, pyobjc-framework-AVFoundation, pyobjc-framework-Accounts, rubicon-objc, pyrect, pyperclip, pyobjc-framework-WebKit, pyobjc-framework-Vision, pyobjc-framework-Virtualization, pyobjc-framework-VideoToolbox, pyobjc-framework-VideoSubscriberAccount, pyobjc-framework-UserNotificationsUI, pyobjc-framework-UniformTypeIdentifiers, pyobjc-framework-SystemExtensions, pyobjc-framework-SystemConfiguration, pyobjc-framework-SyncServices, pyobjc-framework-StoreKit, pyobjc-framework-Speech, pyobjc-framework-SoundAnalysis, pyobjc-framework-Social, pyobjc-framework-ShazamKit, pyobjc-framework-ServiceManagement, pyobjc-framework-SecurityInterface, pyobjc-framework-SecurityFoundation, pyobjc-framework-SearchKit, pyobjc-framework-ScriptingBridge, pyobjc-framework-ScreenTime, pyobjc-framework-ScreenSaver, pyobjc-framework-SceneKit, pyobjc-framework-SafariServices, pyobjc-framework-ReplayKit, pyobjc-framework-QuickLookThumbnailing, pyobjc-framework-PushKit, pyobjc-framework-PreferencePanes, pyobjc-framework-PhotosUI, pyobjc-framework-Photos, pyobjc-framework-PencilKit, pyobjc-framework-PassKit, pyobjc-framework-OSLog, pyobjc-framework-OSAKit, pyobjc-framework-OpenDirectory, pyobjc-framework-NotificationCenter, pyobjc-framework-NetworkExtension, pyobjc-framework-Network, pyobjc-framework-NetFS, pyobjc-framework-NaturalLanguage, pyobjc-framework-MultipeerConnectivity, pyobjc-framework-ModelIO, pyobjc-framework-MLCompute, pyobjc-framework-MetricKit, pyobjc-framework-MetalPerformanceShadersGraph, pyobjc-framework-MetalKit, pyobjc-framework-MediaToolbox, pyobjc-framework-MediaPlayer, pyobjc-framework-MediaLibrary, pyobjc-framework-MediaAccessibility, pyobjc-framework-MapKit, pyobjc-framework-MailKit, pyobjc-framework-LocalAuthenticationEmbeddedUI, pyobjc-framework-LinkPresentation, pyobjc-framework-libdispatch, pyobjc-framework-LaunchServices, pyobjc-framework-LatentSemanticMapping, pyobjc-framework-KernelManagement, pyobjc-framework-iTunesLibrary, pyobjc-framework-IOSurface, pyobjc-framework-IntentsUI, pyobjc-framework-InstantMessage, pyobjc-framework-InstallerPlugins, pyobjc-framework-InputMethodKit, pyobjc-framework-IMServicePlugIn, pyobjc-framework-ImageCaptureCore, pyobjc-framework-GameplayKit, pyobjc-framework-GameKit, pyobjc-framework-GameController, pyobjc-framework-GameCenter, pyobjc-framework-FinderSync, pyobjc-framework-FileProviderUI, pyobjc-framework-ExternalAccessory, pyobjc-framework-ExecutionPolicy, pyobjc-framework-ExceptionHandling, pyobjc-framework-EventKit, pyobjc-framework-DVDPlayback, pyobjc-framework-DiskArbitration, pyobjc-framework-DiscRecordingUI, pyobjc-framework-DictionaryServices, pyobjc-framework-DeviceCheck, pyobjc-framework-DataDetection, pyobjc-framework-CryptoTokenKit, pyobjc-framework-CoreWLAN, pyobjc-framework-CoreText, pyobjc-framework-CoreSpotlight, pyobjc-framework-CoreMotion, pyobjc-framework-CoreMIDI, pyobjc-framework-CoreMediaIO, pyobjc-framework-CoreHaptics, pyobjc-framework-CoreBluetooth, pyobjc-framework-CoreAudioKit, pyobjc-framework-ContactsUI, pyobjc-framework-ColorSync, pyobjc-framework-Collaboration, pyobjc-framework-CloudKit, pyobjc-framework-ClassKit, pyobjc-framework-CFNetwork, pyobjc-framework-CallKit, pyobjc-framework-CalendarStore, pyobjc-framework-BusinessChat, pyobjc-framework-AVKit, pyobjc-framework-Automator, pyobjc-framework-AutomaticAssessmentConfiguration, pyobjc-framework-AuthenticationServices, pyobjc-framework-AudioVideoBridging, pyobjc-framework-AppTrackingTransparency, pyobjc-framework-ApplicationServices, pyobjc-framework-AppleScriptObjC, pyobjc-framework-AppleScriptKit, pyobjc-framework-AdSupport, pyobjc-framework-AdServices, pyobjc-framework-AddressBook, pyobjc-framework-Accessibility, PyTweening, pyscreeze, pyobjc, pymsgbox, pygetwindow, mouseinfo, wikipedia, pyautogui, pywhatkit
Successfully installed PyTweening-1.0.4 mouseinfo-0.1.3 pyautogui-0.9.53 pygetwindow-0.0.9 pymsgbox-1.0.9 pyobjc-8.5 pyobjc-core-8.5 pyobjc-framework-AVFoundation-8.5 pyobjc-framework-AVKit-8.5 pyobjc-framework-Accessibility-8.5 pyobjc-framework-Accounts-8.5 pyobjc-framework-AdServices-8.5 pyobjc-framework-AdSupport-8.5 pyobjc-framework-AddressBook-8.5 pyobjc-framework-AppTrackingTransparency-8.5 pyobjc-framework-AppleScriptKit-8.5 pyobjc-framework-AppleScriptObjC-8.5 pyobjc-framework-ApplicationServices-8.5 pyobjc-framework-AudioVideoBridging-8.5 pyobjc-framework-AuthenticationServices-8.5 pyobjc-framework-AutomaticAssessmentConfiguration-8.5 pyobjc-framework-Automator-8.5 pyobjc-framework-BusinessChat-8.5 pyobjc-framework-CFNetwork-8.5 pyobjc-framework-CalendarStore-8.5 pyobjc-framework-CallKit-8.5 pyobjc-framework-ClassKit-8.5 pyobjc-framework-CloudKit-8.5 pyobjc-framework-Cocoa-8.5 pyobjc-framework-Collaboration-8.5 pyobjc-framework-ColorSync-8.5 pyobjc-framework-Contacts-8.5 pyobjc-framework-ContactsUI-8.5 pyobjc-framework-CoreAudio-8.5 pyobjc-framework-CoreAudioKit-8.5 pyobjc-framework-CoreBluetooth-8.5 pyobjc-framework-CoreData-8.5 pyobjc-framework-CoreHaptics-8.5 pyobjc-framework-CoreLocation-8.5 pyobjc-framework-CoreMIDI-8.5 pyobjc-framework-CoreML-8.5 pyobjc-framework-CoreMedia-8.5 pyobjc-framework-CoreMediaIO-8.5 pyobjc-framework-CoreMotion-8.5 pyobjc-framework-CoreServices-8.5 pyobjc-framework-CoreSpotlight-8.5 pyobjc-framework-CoreText-8.5 pyobjc-framework-CoreWLAN-8.5 pyobjc-framework-CryptoTokenKit-8.5 pyobjc-framework-DVDPlayback-8.5 pyobjc-framework-DataDetection-8.5 pyobjc-framework-DeviceCheck-8.5 pyobjc-framework-DictionaryServices-8.5 pyobjc-framework-DiscRecording-8.5 pyobjc-framework-DiscRecordingUI-8.5 pyobjc-framework-DiskArbitration-8.5 pyobjc-framework-EventKit-8.5 pyobjc-framework-ExceptionHandling-8.5 pyobjc-framework-ExecutionPolicy-8.5 pyobjc-framework-ExternalAccessory-8.5 pyobjc-framework-FSEvents-8.5 pyobjc-framework-FileProvider-8.5 pyobjc-framework-FileProviderUI-8.5 pyobjc-framework-FinderSync-8.5 pyobjc-framework-GameCenter-8.5 pyobjc-framework-GameController-8.5 pyobjc-framework-GameKit-8.5 pyobjc-framework-GameplayKit-8.5 pyobjc-framework-IMServicePlugIn-8.5 pyobjc-framework-IOSurface-8.5 pyobjc-framework-ImageCaptureCore-8.5 pyobjc-framework-InputMethodKit-8.5 pyobjc-framework-InstallerPlugins-8.5 pyobjc-framework-InstantMessage-8.5 pyobjc-framework-Intents-8.5 pyobjc-framework-IntentsUI-8.5 pyobjc-framework-KernelManagement-8.5 pyobjc-framework-LatentSemanticMapping-8.5 pyobjc-framework-LaunchServices-8.5 pyobjc-framework-LinkPresentation-8.5 pyobjc-framework-LocalAuthentication-8.5 pyobjc-framework-LocalAuthenticationEmbeddedUI-8.5 pyobjc-framework-MLCompute-8.5 pyobjc-framework-MailKit-8.5 pyobjc-framework-MapKit-8.5 pyobjc-framework-MediaAccessibility-8.5 pyobjc-framework-MediaLibrary-8.5 pyobjc-framework-MediaPlayer-8.5 pyobjc-framework-MediaToolbox-8.5 pyobjc-framework-Metal-8.5 pyobjc-framework-MetalKit-8.5 pyobjc-framework-MetalPerformanceShaders-8.5 pyobjc-framework-MetalPerformanceShadersGraph-8.5 pyobjc-framework-MetricKit-8.5 pyobjc-framework-ModelIO-8.5 pyobjc-framework-MultipeerConnectivity-8.5 pyobjc-framework-NaturalLanguage-8.5 pyobjc-framework-NetFS-8.5 pyobjc-framework-Network-8.5 pyobjc-framework-NetworkExtension-8.5 pyobjc-framework-NotificationCenter-8.5 pyobjc-framework-OSAKit-8.5 pyobjc-framework-OSLog-8.5 pyobjc-framework-OpenDirectory-8.5 pyobjc-framework-PassKit-8.5 pyobjc-framework-PencilKit-8.5 pyobjc-framework-Photos-8.5 pyobjc-framework-PhotosUI-8.5 pyobjc-framework-PreferencePanes-8.5 pyobjc-framework-PushKit-8.5 pyobjc-framework-Quartz-8.5 pyobjc-framework-QuickLookThumbnailing-8.5 pyobjc-framework-ReplayKit-8.5 pyobjc-framework-SafariServices-8.5 pyobjc-framework-SceneKit-8.5 pyobjc-framework-ScreenSaver-8.5 pyobjc-framework-ScreenTime-8.5 pyobjc-framework-ScriptingBridge-8.5 pyobjc-framework-SearchKit-8.5 pyobjc-framework-Security-8.5 pyobjc-framework-SecurityFoundation-8.5 pyobjc-framework-SecurityInterface-8.5 pyobjc-framework-ServiceManagement-8.5 pyobjc-framework-ShazamKit-8.5 pyobjc-framework-Social-8.5 pyobjc-framework-SoundAnalysis-8.5 pyobjc-framework-Speech-8.5 pyobjc-framework-SpriteKit-8.5 pyobjc-framework-StoreKit-8.5 pyobjc-framework-SyncServices-8.5 pyobjc-framework-SystemConfiguration-8.5 pyobjc-framework-SystemExtensions-8.5 pyobjc-framework-UniformTypeIdentifiers-8.5 pyobjc-framework-UserNotifications-8.5 pyobjc-framework-UserNotificationsUI-8.5 pyobjc-framework-VideoSubscriberAccount-8.5 pyobjc-framework-VideoToolbox-8.5 pyobjc-framework-Virtualization-8.5 pyobjc-framework-Vision-8.5 pyobjc-framework-WebKit-8.5 pyobjc-framework-iTunesLibrary-8.5 pyobjc-framework-libdispatch-8.5 pyperclip-1.8.2 pyrect-0.2.0 pyscreeze-0.1.28 pywhatkit-5.3 rubicon-objc-0.4.2 wikipedia-1.4.0
(base) pradeep:~$

```



```python
>>> import pywhatkit
>>> pywhatkit.sendwhatmsg("+919876543210","Hello from Pradeep",16,40)
In 2 Seconds WhatsApp will open and after 15 Seconds Message will be Delivered!
>>> 

```

