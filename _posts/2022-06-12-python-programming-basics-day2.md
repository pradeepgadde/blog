---
layout: single
title:  "Python Programming Basics Day#2"
date:   2022-06-12 16:56:04 +0530
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
## Day2
It is Day#2 of Linux World Python Training.

Loops: Iteration and Recursion
Iteration with `for` and `while` loops

## For loop
```python
>>> mobiles=[1111,2222,3333,4444,5555]
>>> mobiles
[1111, 2222, 3333, 4444, 5555]
>>> for m in mobiles:
...     if m == 6666:
...             print("I found")
...     else:
...             print("Not Found")
... 
Not Found
Not Found
Not Found
Not Found
Not Found
>>> 
```

Instead of using `else` with `if`, use it with the `for` block.

```python
>>> for m in mobiles:
...     if m == 6666:
...             print("Not Found")
... else:
...     print("Not Found")
... 
Not Found
>>> 

```
After iterating through all items, if there is any `else` block, that gets executed.

```python
>>> for m in mobiles:
...     print(m)
... else:
...     print("Done")
... 
1111
2222
3333
4444
5555
Done
>>> 

```

But sometimes, this is not what you may want!

```python
>>> for m in mobiles:
...     if m == 2222:
...             print("I found")
... else:
...     print("Not Found")
... 
I found
Not Found
>>> 

```

You can use `break` in such cases.

```python
>>> for m in mobiles:
...     if m == 2222:
...             print("I found")
...             break
... else:
...     print("Not Found")
... 
I found
>>> 
```

```python
>>> temp = 40
>>> if temp>30:
...     ac=25
... else:
...     ac=29
... 
>>> 
>>> ac
25
>>> 

```

The same code can be achieved with single line in Python with Functional programming.

## Functional Programming

```python
>>> ac = 25 if temp > 30 else 29
>>> ac
25
>>> 
```

Another example,

```python
>>> temp=12
>>> ac = 25 if temp > 30 else 29
>>> ac
29
>>> 
```

```python
>>> temp=16
>>> ac = 25 if temp > 30 else "AC Not Required"
>>> ac
'AC Not Required'
>>> 
```

```python
>>> db=[1,2,3,4,5]
>>> for i in db:
...     print(i*i) # i ** 2 is same as i*i
... 
1
4
9
16
25
>>> 

```



Let's say you want to save this in another list; `Extraction, Transforming, and Load (ETL)`

An example ETL Operation

```python
>>> db_square=[]
>>> for i in db:
...     j=i**2
...     db_square.append(j)
... 
>>> db_square
[1, 4, 9, 16, 25]
>>> 

```

ETL Functional Programming way

```python
>>> db_sq=[i**2 for i in db]
>>> db_sq
[1, 4, 9, 16, 25]
>>> 
```



## List Comprehension

First operation, then `for` loop syntax followed by the condition `if` block.

```python
>>> db_sq=[i**2 for i in db if i !=3]
>>> db_sq
[1, 4, 16, 25]
>>> 
```

## String Concatenation
```python
>>> s="Juniper"

>>> 
>>> s+"Networks"
'JuniperNetworks'
>>> s
'Juniper'
>>> 
```

```python
>>> s+" " + "Networks"
'Juniper Networks'
>>> 
```
Add mail adress to only names 
```python
>>> names=["pradeep","suma","dakshita@gmail.com"]
>>> emails=[ n + "@gmail.com" for n in names if not n.endswith("gmail.com")]
>>> emails
['pradeep@gmail.com', 'suma@gmail.com']
>>> 

```

## While Loop
With shorthand operator `+=`

```python
>>> page=1
>>> while page<=10:
...     print("essay")
...     page+=1 # page = page +1... 
essay
essay
essay
essay
essay
essay
essay
essay
essay
essay
>>> 
```

## Division
Normal Division and Floor Division
```python
>>> 5/2
2.5
>>> 5//2
2
>>> 
>>> 9/2
4.5
>>> 9//2
4
>>> 
```
## While Loop with Else
```python
>>> page=1
>>> while page<=10:
...     print("essay")
...     page+=1
... else:
...     print("bye")
... 
essay
essay
essay
essay
essay
essay
essay
essay
essay
essay
bye
>>> 
```
While loop with a `break`

```python
>>> page=1
>>> while page<=10:
...     if page == 5:
...             print("essay")
...             break
...     page+=1
... else:
...     print("bye")
... 
essay
>>> 
```

## Python Memory Management

Every programming language, depending on the data type, reserves some amount of space in the memory. Memory segements in the RAM.

Amount of space allocated may be different in each language.

Variable name is nothing but a reference to the memory space.

Python has a memory management process.

Use the `id` command to get the location of the memory.

```python
>>> help(id)

Help on built-in function id in module builtins:

id(obj, /)
    Return the identity of an object.
    
    This is guaranteed to be unique among simultaneously existing objects.
    (CPython uses the object's memory address.)
~

```

In Decimal format

```python
>>> x=5
>>> id(x)
140552991545776
>>> 

```

In Hexadecimal format

```python
>>> hex(id(x))
'0x7fd50b2219b0'
>>> 
```
Another variable location

```python
>>> names
['pradeep', 'suma', 'dakshita@gmail.com']
>>> id(names)
140553086075008
>>> 
```

In Python, if multiple variables have the same data(value), they all point to single location, as seen below.

References are created automatically.



```python
>>> x,y,z=5,5,5
>>> x
5
>>> y
5
>>> z
5
>>> id(z)
140552991545776
>>> id(y)
140552991545776
>>> id(x)
140552991545776
>>> 

```

```python
>>> x is y
True
>>> y is z
True
>>> z is z
True
>>> t=10
>>> x is t
False
>>> 
```

List stores all items in contiguous memory locations.

Name of the list is stored in the first item location.

```python
>>> names
['pradeep', 'suma', 'dakshita@gmail.com']
>>> id(names)
140553086075008
>>> id(names[0])
140553086240816
>>> id(names[1])
140553084116016
>>> id(names[2])
140552996080656
```

If another list contains the same items, they refer to the same memory location as seen below.

```python
>>> another_name_list=names
>>> id(another_name_list)
140553086075008
>>> id(another_name_list[2])
140552996080656
>>> 
```



Assignment or reference pointer (denoted by `=`).

```python
>>> names[1] is another_name_list[1]
True
>>> 
```

Modifying one list, modifies the other as well.

```python
>>> another_name_list[2]="dakshita"
>>> names
['pradeep', 'suma', 'dakshita']
>>> another_name_list
['pradeep', 'suma', 'dakshita']
>>> 
```

To make a list read-only, we can make use of `tuples`.

## Tuple

```python
>>> names=('pradeep','suma','dakshita')
>>> type(names)
<class 'tuple'>
>>> id(names)
140553086568960
>>> id(names[0])
140553086240816
>>> id(names[1])
140553084116016
>>> id(names[2])
140553086425008
>>> 

```

```python
>>> another_tuple=names
>>> another_tuple
('pradeep', 'suma', 'dakshita')
>>> id(another_tuple)
140553086568960
>>> id(another_tuple[2])
140553086425008
>>> 

```

```py
>>> names is another_tuple
True
>>> 
```

If we try to change the value of a tuple, you will get a `TypeError`.

```python
>>> another_tuple[2]="sripragnya"
Traceback (most recent call last):
  File "<stdin>", line 1, in <module>
TypeError: 'tuple' object does not support item assignment
>>> 
```

Tuple object does not support item assignment.

Tuple is `immutable`, read-only.

List of methods available with the `tuple`

```python
>>> dir(names)
['__add__', '__class__', '__class_getitem__', '__contains__', '__delattr__', '__dir__', '__doc__', '__eq__', '__format__', '__ge__', '__getattribute__', '__getitem__', '__getnewargs__', '__gt__', '__hash__', '__init__', '__init_subclass__', '__iter__', '__le__', '__len__', '__lt__', '__mul__', '__ne__', '__new__', '__reduce__', '__reduce_ex__', '__repr__', '__rmul__', '__setattr__', '__sizeof__', '__str__', '__subclasshook__', 'count', 'index']
>>> 
```
List of methods available with the `list`
```py
>>> dir(another_name_list)
['__add__', '__class__', '__class_getitem__', '__contains__', '__delattr__', '__delitem__', '__dir__', '__doc__', '__eq__', '__format__', '__ge__', '__getattribute__', '__getitem__', '__gt__', '__hash__', '__iadd__', '__imul__', '__init__', '__init_subclass__', '__iter__', '__le__', '__len__', '__lt__', '__mul__', '__ne__', '__new__', '__reduce__', '__reduce_ex__', '__repr__', '__reversed__', '__rmul__', '__setattr__', '__setitem__', '__sizeof__', '__str__', '__subclasshook__', 'append', 'clear', 'copy', 'count', 'extend', 'index', 'insert', 'pop', 'remove', 'reverse', 'sort']
>>> 
```

Similar to list, we can do slicing

```py
>>> names[::-1]
('dakshita', 'suma', 'pradeep')
>>> names[2:]
('dakshita',)
>>> 

```
We can iterate a tuple using `for` loop
```py
>>> for n in names:
...     print(n)
... 
pradeep
suma
dakshita
>>> 
```

## Functions

A reusable block of code designed to solve a problem.

`def` is the keyword in Python to define a function.

```python
>>> def demo():    ## Defining a function
...     print("Hi")
...     x=5
...     print(x)
... 
>>> demo()  ## Calling a function
Hi
5
>>> 
```



```python
>>> def add():
...     x =5
...     y=10
...     z=x+y
...     print(z)
... 
>>> add()
15
>>> 
```



```python
>>> add(3,4)
Traceback (most recent call last):
  File "<stdin>", line 1, in <module>
TypeError: add() takes 0 positional arguments but 2 were given
>>> 

```

Function with parameters, very flexible.

```python
>>> def add(i,j): ## Function Signature
...     x=i
...     y=j
...     z=x+y
...     print(z)
... 
>>> add(3,4)
7
>>> 
```

We can pass any arguments that we want, the result will be returned.

```python
>>> add(21,45)
66
>>> 
```

If we dont pass any arguments, will get an Error

```python
>>> add()
Traceback (most recent call last):
  File "<stdin>", line 1, in <module>
TypeError: add() missing 2 required positional arguments: 'i' and 'j'
>>> 

```

If we pass more arguments, then also same error

```python
>>> add(1,2,3,4)
Traceback (most recent call last):
  File "<stdin>", line 1, in <module>
TypeError: add() takes 2 positional arguments but 4 were given
>>> 
```

While passing parameters (arguments), we have to follow the function signature.

Instead of printing,if we have to store the value, use the `return`.

```python
>>> def add(i,j):
...     x=i
...     y=j
...     z=x+y
...     return z
... 
>>> add(3,4)
7
>>> r=add(3,4)
>>> r
7
>>> type(r)
<class 'int'>
>>> 

```

If we dont know the number of arguments in advance, what to do?

We can use `*` along with the argument name, then it becomes a tuple. With this we can pass any number of arguments.

```python
>>> def add(*i):
...     return(i)
... 
>>> add(3,4)
(3, 4)
>>> add(4,5,6,7)
(4, 5, 6, 7)
>>> add()
()
>>> 
```

We can confirm the type of the argument with the `type`.

```python
>>> def add(*i):
...     print(type(i))
... 
>>> add(3,4)
<class 'tuple'>
>>> add()
<class 'tuple'>
>>> 
```

```python
>>> def add(*x):
...     j=0
...     for i in x:
...             j=j+i
...     return j
... 
>>> add(3,4)
7
>>> add(3,4,89)
96
>>> 

```

DRY principle: Don't Repeat Yourself!

```python
>>> def mypower(*data):
...     r=[]
...     for i in data:
...             if i!=3:
...                     j=i**2
...                     r.append(j)
...     return(r)
... 
>>> mypower(4,5,3,6)
[16, 25, 36]
>>> mypower(1,3,2)
[1, 4]
>>> 
```

A Function can return multiple values as well.

```python
>>> def test(i,j):
...     return[ i+j, i-j, i*j]
... 
>>> test(2,3)
[5, -1, 6]
>>> test(5,3)
[8, 2, 15]
>>> a=test(5,3)
>>> a
[8, 2, 15]
>>> type(a)
<class 'list'>
>>> 

```

The return statement must be the last statement in a function. Any statements below the return statement are not executed.

```python
>>> def test(i,j):
...     return[i+j,i+j,i*j]
...     print("Testing...") ## will not be executed
... 
>>> test(2,3)
[5, 5, 6]
>>> 

```

A function can also return a tuple, instead of a list.

```python
>>> def test(i,j):
...     return(i+j,i+j,i*j)
... 
>>> test(4,5)
(9, 9, 20)
>>> a=test(4,5)
>>> type(a)
<class 'tuple'>
>>> a
(9, 9, 20)
>>> 

```

What if we forget the signature of a function and pass the arguments in a wrong order?

```python
>>> def mydata(i,j,k):
...     print("name:"+i)
...     print("city:"+j)
...     print("country:"+k)
... 
>>> mydata("pradeep","Bangalore","India")
name:pradeep
city:Bangalore
country:India
>>> mydata("Bangalore","India","Pradeep")
name:Bangalore
city:India
country:Pradeep
>>> 
```

if you forget Positional arguments, use help. Also use proper names to the variables.

```python

>>> help(mydata)

Help on function mydata in module __main__:

mydata(i, j, k)
(END)

```

```python
>>> def mydata(name,city,country):
...     print("name:"+name)
...     print("city:"+city)
...     print("country:"+country)
... 
>>> help(mydata)

Help on function mydata in module __main__:

mydata(name, city, country)
(END)
```

While passing variables, use the name, then the order does not matter.

```python
>>> mydata("pradeep","India","Bangalore") ## Problem with Postional Arguments
name:pradeep
city:India
country:Bangalore
>>> mydata(name="pradeep",country="India",city="Bangalore") ## Solution, named arguments!
name:pradeep
city:Bangalore
country:India
>>> 

```

Default arguments

```python
>>> mydata(name="pradeep",city="Bangalore")
Traceback (most recent call last):
  File "<stdin>", line 1, in <module>
TypeError: mydata() missing 1 required positional argument: 'country'
>>> 
```



```python
>>> def mydata(name,city,country="India"):
...     print("name:"+name)
...     print("city:"+city)
...     print("country:"+country)
... 
>>> mydata(name="pradeep",city="Bangalore")
name:pradeep
city:Bangalore
country:India
>>> 

```
If we pass, it will overwrite the default value.
```python
>>> mydata(name="pradeep",city="Bangalore",country="US")
name:pradeep
city:Bangalore
country:US
>>> 
```

We can see how these are used in the most common `print` function.
```python
>>> help(print)

Help on built-in function print in module builtins:

print(...)
    print(value, ..., sep=' ', end='\n', file=sys.stdout, flush=False)
    
    Prints the values to a stream, or to sys.stdout by default.
    Optional keyword arguments:
    file:  a file-like object (stream); defaults to the current sys.stdout.
    sep:   string inserted between values, default a space.
    end:   string appended after the last value, default a newline.
    flush: whether to forcibly flush the stream.
```

```python
>>> print("Juniper","Networks")
Juniper Networks
>>> print("Juniper","Networks",sep='+')
Juniper+Networks
>>> 
```
A simple demo of this 
```python
>>> db
['pradeep', 'suma', 'dakshita']
>>> for n in db:
...     print(n,"gmail.com",sep='@')
... 
pradeep@gmail.com
suma@gmail.com
dakshita@gmail.com
>>> 

```

Entire block of code is stored somewhere in the memory.

```python
>>> id(mydata)
140351512275552

>>> help(mydata)

Help on function mydata in module __main__:

mydata(name, city, country='India')
(END)
```

```python
>>> another_data=mydata ## assignment
>>> id(another_data)
140351512275552
>>> 

```

```python
>>> another_data()
Traceback (most recent call last):
  File "<stdin>", line 1, in <module>
TypeError: mydata() missing 2 required positional arguments: 'name' and 'city'
>>> 
```



```python
>>> def manager():
...     return("Manager")
... 
>>> def engineer():
...     return("Engineer")
... 
>>> salary=5
>>> title=manager() if salary>4 else engineer()
>>> title
'Manager'
>>> 
```

## Text to Speech

TTS: Text to Speech

Install the `pyttsx3` module

```python
(base) pradeep:~$pip install pyttsx3
Collecting pyttsx3
  Downloading pyttsx3-2.90-py3-none-any.whl (39 kB)
Requirement already satisfied: pyobjc>=2.4 in ./opt/anaconda3/lib/python3.9/site-packages (from pyttsx3) (8.5)
Requirement already satisfied: pyobjc-framework-AudioVideoBridging==8.5 in ./opt/anaconda3/lib/python3.9/site-packages (from pyobjc>=2.4->pyttsx3) (8.5)
Requirement already satisfied: pyobjc-framework-InstallerPlugins==8.5 in ./opt/anaconda3/lib/python3.9/site-packages (from pyobjc>=2.4->pyttsx3) (8.5)
Requirement already satisfied: pyobjc-framework-ShazamKit==8.5 in ./opt/anaconda3/lib/python3.9/site-packages (from pyobjc>=2.4->pyttsx3) (8.5)
Requirement already satisfied: pyobjc-framework-FileProvider==8.5 in ./opt/anaconda3/lib/python3.9/site-packages (from pyobjc>=2.4->pyttsx3) (8.5)
Requirement already satisfied: pyobjc-framework-CoreMedia==8.5 in ./opt/anaconda3/lib/python3.9/site-packages (from pyobjc>=2.4->pyttsx3) (8.5)
Requirement already satisfied: pyobjc-framework-AVFoundation==8.5 in ./opt/anaconda3/lib/python3.9/site-packages (from pyobjc>=2.4->pyttsx3) (8.5)
Requirement already satisfied: pyobjc-framework-NaturalLanguage==8.5 in ./opt/anaconda3/lib/python3.9/site-packages (from pyobjc>=2.4->pyttsx3) (8.5)
Requirement already satisfied: pyobjc-framework-MediaLibrary==8.5 in ./opt/anaconda3/lib/python3.9/site-packages (from pyobjc>=2.4->pyttsx3) (8.5)
Requirement already satisfied: pyobjc-framework-LaunchServices==8.5 in ./opt/anaconda3/lib/python3.9/site-packages (from pyobjc>=2.4->pyttsx3) (8.5)
Requirement already satisfied: pyobjc-framework-UserNotifications==8.5 in ./opt/anaconda3/lib/python3.9/site-packages (from pyobjc>=2.4->pyttsx3) (8.5)
Requirement already satisfied: pyobjc-framework-CoreWLAN==8.5 in ./opt/anaconda3/lib/python3.9/site-packages (from pyobjc>=2.4->pyttsx3) (8.5)
Requirement already satisfied: pyobjc-framework-MetalPerformanceShaders==8.5 in ./opt/anaconda3/lib/python3.9/site-packages (from pyobjc>=2.4->pyttsx3) (8.5)
Requirement already satisfied: pyobjc-framework-CoreData==8.5 in ./opt/anaconda3/lib/python3.9/site-packages (from pyobjc>=2.4->pyttsx3) (8.5)
Requirement already satisfied: pyobjc-framework-ExternalAccessory==8.5 in ./opt/anaconda3/lib/python3.9/site-packages (from pyobjc>=2.4->pyttsx3) (8.5)
Requirement already satisfied: pyobjc-framework-CFNetwork==8.5 in ./opt/anaconda3/lib/python3.9/site-packages (from pyobjc>=2.4->pyttsx3) (8.5)
Requirement already satisfied: pyobjc-framework-LocalAuthentication==8.5 in ./opt/anaconda3/lib/python3.9/site-packages (from pyobjc>=2.4->pyttsx3) (8.5)
Requirement already satisfied: pyobjc-framework-MediaPlayer==8.5 in ./opt/anaconda3/lib/python3.9/site-packages (from pyobjc>=2.4->pyttsx3) (8.5)
Requirement already satisfied: pyobjc-framework-ApplicationServices==8.5 in ./opt/anaconda3/lib/python3.9/site-packages (from pyobjc>=2.4->pyttsx3) (8.5)
Requirement already satisfied: pyobjc-framework-PushKit==8.5 in ./opt/anaconda3/lib/python3.9/site-packages (from pyobjc>=2.4->pyttsx3) (8.5)
Requirement already satisfied: pyobjc-framework-Social==8.5 in ./opt/anaconda3/lib/python3.9/site-packages (from pyobjc>=2.4->pyttsx3) (8.5)
Requirement already satisfied: pyobjc-framework-SecurityFoundation==8.5 in ./opt/anaconda3/lib/python3.9/site-packages (from pyobjc>=2.4->pyttsx3) (8.5)
Requirement already satisfied: pyobjc-framework-CallKit==8.5 in ./opt/anaconda3/lib/python3.9/site-packages (from pyobjc>=2.4->pyttsx3) (8.5)
Requirement already satisfied: pyobjc-framework-MetalKit==8.5 in ./opt/anaconda3/lib/python3.9/site-packages (from pyobjc>=2.4->pyttsx3) (8.5)
Requirement already satisfied: pyobjc-framework-GameController==8.5 in ./opt/anaconda3/lib/python3.9/site-packages (from pyobjc>=2.4->pyttsx3) (8.5)
Requirement already satisfied: pyobjc-framework-Collaboration==8.5 in ./opt/anaconda3/lib/python3.9/site-packages (from pyobjc>=2.4->pyttsx3) (8.5)
Requirement already satisfied: pyobjc-framework-PreferencePanes==8.5 in ./opt/anaconda3/lib/python3.9/site-packages (from pyobjc>=2.4->pyttsx3) (8.5)
Requirement already satisfied: pyobjc-framework-OSAKit==8.5 in ./opt/anaconda3/lib/python3.9/site-packages (from pyobjc>=2.4->pyttsx3) (8.5)
Requirement already satisfied: pyobjc-framework-DictionaryServices==8.5 in ./opt/anaconda3/lib/python3.9/site-packages (from pyobjc>=2.4->pyttsx3) (8.5)
Requirement already satisfied: pyobjc-framework-LocalAuthenticationEmbeddedUI==8.5 in ./opt/anaconda3/lib/python3.9/site-packages (from pyobjc>=2.4->pyttsx3) (8.5)
Requirement already satisfied: pyobjc-framework-CalendarStore==8.5 in ./opt/anaconda3/lib/python3.9/site-packages (from pyobjc>=2.4->pyttsx3) (8.5)
Requirement already satisfied: pyobjc-framework-CoreAudio==8.5 in ./opt/anaconda3/lib/python3.9/site-packages (from pyobjc>=2.4->pyttsx3) (8.5)
Requirement already satisfied: pyobjc-framework-CloudKit==8.5 in ./opt/anaconda3/lib/python3.9/site-packages (from pyobjc>=2.4->pyttsx3) (8.5)
Requirement already satisfied: pyobjc-framework-CoreMotion==8.5 in ./opt/anaconda3/lib/python3.9/site-packages (from pyobjc>=2.4->pyttsx3) (8.5)
Requirement already satisfied: pyobjc-framework-CoreMIDI==8.5 in ./opt/anaconda3/lib/python3.9/site-packages (from pyobjc>=2.4->pyttsx3) (8.5)
Requirement already satisfied: pyobjc-framework-MediaToolbox==8.5 in ./opt/anaconda3/lib/python3.9/site-packages (from pyobjc>=2.4->pyttsx3) (8.5)
Requirement already satisfied: pyobjc-framework-NotificationCenter==8.5 in ./opt/anaconda3/lib/python3.9/site-packages (from pyobjc>=2.4->pyttsx3) (8.5)
Requirement already satisfied: pyobjc-framework-GameCenter==8.5 in ./opt/anaconda3/lib/python3.9/site-packages (from pyobjc>=2.4->pyttsx3) (8.5)
Requirement already satisfied: pyobjc-framework-WebKit==8.5 in ./opt/anaconda3/lib/python3.9/site-packages (from pyobjc>=2.4->pyttsx3) (8.5)
Requirement already satisfied: pyobjc-framework-Cocoa==8.5 in ./opt/anaconda3/lib/python3.9/site-packages (from pyobjc>=2.4->pyttsx3) (8.5)
Requirement already satisfied: pyobjc-framework-LatentSemanticMapping==8.5 in ./opt/anaconda3/lib/python3.9/site-packages (from pyobjc>=2.4->pyttsx3) (8.5)
Requirement already satisfied: pyobjc-framework-MetalPerformanceShadersGraph==8.5 in ./opt/anaconda3/lib/python3.9/site-packages (from pyobjc>=2.4->pyttsx3) (8.5)
Requirement already satisfied: pyobjc-framework-InputMethodKit==8.5 in ./opt/anaconda3/lib/python3.9/site-packages (from pyobjc>=2.4->pyttsx3) (8.5)
Requirement already satisfied: pyobjc-framework-DataDetection==8.5 in ./opt/anaconda3/lib/python3.9/site-packages (from pyobjc>=2.4->pyttsx3) (8.5)
Requirement already satisfied: pyobjc-framework-CoreSpotlight==8.5 in ./opt/anaconda3/lib/python3.9/site-packages (from pyobjc>=2.4->pyttsx3) (8.5)
Requirement already satisfied: pyobjc-framework-EventKit==8.5 in ./opt/anaconda3/lib/python3.9/site-packages (from pyobjc>=2.4->pyttsx3) (8.5)
Requirement already satisfied: pyobjc-framework-AppleScriptObjC==8.5 in ./opt/anaconda3/lib/python3.9/site-packages (from pyobjc>=2.4->pyttsx3) (8.5)
Requirement already satisfied: pyobjc-framework-CoreText==8.5 in ./opt/anaconda3/lib/python3.9/site-packages (from pyobjc>=2.4->pyttsx3) (8.5)
Requirement already satisfied: pyobjc-framework-CoreBluetooth==8.5 in ./opt/anaconda3/lib/python3.9/site-packages (from pyobjc>=2.4->pyttsx3) (8.5)
Requirement already satisfied: pyobjc-framework-Intents==8.5 in ./opt/anaconda3/lib/python3.9/site-packages (from pyobjc>=2.4->pyttsx3) (8.5)
Requirement already satisfied: pyobjc-framework-ExecutionPolicy==8.5 in ./opt/anaconda3/lib/python3.9/site-packages (from pyobjc>=2.4->pyttsx3) (8.5)
Requirement already satisfied: pyobjc-framework-SpriteKit==8.5 in ./opt/anaconda3/lib/python3.9/site-packages (from pyobjc>=2.4->pyttsx3) (8.5)
Requirement already satisfied: pyobjc-framework-DeviceCheck==8.5 in ./opt/anaconda3/lib/python3.9/site-packages (from pyobjc>=2.4->pyttsx3) (8.5)
Requirement already satisfied: pyobjc-framework-MLCompute==8.5 in ./opt/anaconda3/lib/python3.9/site-packages (from pyobjc>=2.4->pyttsx3) (8.5)
Requirement already satisfied: pyobjc-framework-SystemConfiguration==8.5 in ./opt/anaconda3/lib/python3.9/site-packages (from pyobjc>=2.4->pyttsx3) (8.5)
Requirement already satisfied: pyobjc-framework-ClassKit==8.5 in ./opt/anaconda3/lib/python3.9/site-packages (from pyobjc>=2.4->pyttsx3) (8.5)
Requirement already satisfied: pyobjc-core==8.5 in ./opt/anaconda3/lib/python3.9/site-packages (from pyobjc>=2.4->pyttsx3) (8.5)
Requirement already satisfied: pyobjc-framework-FileProviderUI==8.5 in ./opt/anaconda3/lib/python3.9/site-packages (from pyobjc>=2.4->pyttsx3) (8.5)
Requirement already satisfied: pyobjc-framework-Virtualization==8.5 in ./opt/anaconda3/lib/python3.9/site-packages (from pyobjc>=2.4->pyttsx3) (8.5)
Requirement already satisfied: pyobjc-framework-SyncServices==8.5 in ./opt/anaconda3/lib/python3.9/site-packages (from pyobjc>=2.4->pyttsx3) (8.5)
Requirement already satisfied: pyobjc-framework-DiskArbitration==8.5 in ./opt/anaconda3/lib/python3.9/site-packages (from pyobjc>=2.4->pyttsx3) (8.5)
Requirement already satisfied: pyobjc-framework-NetFS==8.5 in ./opt/anaconda3/lib/python3.9/site-packages (from pyobjc>=2.4->pyttsx3) (8.5)
Requirement already satisfied: pyobjc-framework-VideoSubscriberAccount==8.5 in ./opt/anaconda3/lib/python3.9/site-packages (from pyobjc>=2.4->pyttsx3) (8.5)
Requirement already satisfied: pyobjc-framework-DiscRecording==8.5 in ./opt/anaconda3/lib/python3.9/site-packages (from pyobjc>=2.4->pyttsx3) (8.5)
Requirement already satisfied: pyobjc-framework-CoreHaptics==8.5 in ./opt/anaconda3/lib/python3.9/site-packages (from pyobjc>=2.4->pyttsx3) (8.5)
Requirement already satisfied: pyobjc-framework-Metal==8.5 in ./opt/anaconda3/lib/python3.9/site-packages (from pyobjc>=2.4->pyttsx3) (8.5)
Requirement already satisfied: pyobjc-framework-Contacts==8.5 in ./opt/anaconda3/lib/python3.9/site-packages (from pyobjc>=2.4->pyttsx3) (8.5)
Requirement already satisfied: pyobjc-framework-MultipeerConnectivity==8.5 in ./opt/anaconda3/lib/python3.9/site-packages (from pyobjc>=2.4->pyttsx3) (8.5)
Requirement already satisfied: pyobjc-framework-GameplayKit==8.5 in ./opt/anaconda3/lib/python3.9/site-packages (from pyobjc>=2.4->pyttsx3) (8.5)
Requirement already satisfied: pyobjc-framework-Accessibility==8.5 in ./opt/anaconda3/lib/python3.9/site-packages (from pyobjc>=2.4->pyttsx3) (8.5)
Requirement already satisfied: pyobjc-framework-CoreMediaIO==8.5 in ./opt/anaconda3/lib/python3.9/site-packages (from pyobjc>=2.4->pyttsx3) (8.5)
Requirement already satisfied: pyobjc-framework-VideoToolbox==8.5 in ./opt/anaconda3/lib/python3.9/site-packages (from pyobjc>=2.4->pyttsx3) (8.5)
Requirement already satisfied: pyobjc-framework-ExceptionHandling==8.5 in ./opt/anaconda3/lib/python3.9/site-packages (from pyobjc>=2.4->pyttsx3) (8.5)
Requirement already satisfied: pyobjc-framework-SceneKit==8.5 in ./opt/anaconda3/lib/python3.9/site-packages (from pyobjc>=2.4->pyttsx3) (8.5)
Requirement already satisfied: pyobjc-framework-AdServices==8.5 in ./opt/anaconda3/lib/python3.9/site-packages (from pyobjc>=2.4->pyttsx3) (8.5)
Requirement already satisfied: pyobjc-framework-AVKit==8.5 in ./opt/anaconda3/lib/python3.9/site-packages (from pyobjc>=2.4->pyttsx3) (8.5)
Requirement already satisfied: pyobjc-framework-MapKit==8.5 in ./opt/anaconda3/lib/python3.9/site-packages (from pyobjc>=2.4->pyttsx3) (8.5)
Requirement already satisfied: pyobjc-framework-MailKit==8.5 in ./opt/anaconda3/lib/python3.9/site-packages (from pyobjc>=2.4->pyttsx3) (8.5)
Requirement already satisfied: pyobjc-framework-BusinessChat==8.5 in ./opt/anaconda3/lib/python3.9/site-packages (from pyobjc>=2.4->pyttsx3) (8.5)
Requirement already satisfied: pyobjc-framework-Photos==8.5 in ./opt/anaconda3/lib/python3.9/site-packages (from pyobjc>=2.4->pyttsx3) (8.5)
Requirement already satisfied: pyobjc-framework-IntentsUI==8.5 in ./opt/anaconda3/lib/python3.9/site-packages (from pyobjc>=2.4->pyttsx3) (8.5)
Requirement already satisfied: pyobjc-framework-KernelManagement==8.5 in ./opt/anaconda3/lib/python3.9/site-packages (from pyobjc>=2.4->pyttsx3) (8.5)
Requirement already satisfied: pyobjc-framework-PhotosUI==8.5 in ./opt/anaconda3/lib/python3.9/site-packages (from pyobjc>=2.4->pyttsx3) (8.5)
Requirement already satisfied: pyobjc-framework-Speech==8.5 in ./opt/anaconda3/lib/python3.9/site-packages (from pyobjc>=2.4->pyttsx3) (8.5)
Requirement already satisfied: pyobjc-framework-PassKit==8.5 in ./opt/anaconda3/lib/python3.9/site-packages (from pyobjc>=2.4->pyttsx3) (8.5)
Requirement already satisfied: pyobjc-framework-PencilKit==8.5 in ./opt/anaconda3/lib/python3.9/site-packages (from pyobjc>=2.4->pyttsx3) (8.5)
Requirement already satisfied: pyobjc-framework-UniformTypeIdentifiers==8.5 in ./opt/anaconda3/lib/python3.9/site-packages (from pyobjc>=2.4->pyttsx3) (8.5)
Requirement already satisfied: pyobjc-framework-DiscRecordingUI==8.5 in ./opt/anaconda3/lib/python3.9/site-packages (from pyobjc>=2.4->pyttsx3) (8.5)
Requirement already satisfied: pyobjc-framework-iTunesLibrary==8.5 in ./opt/anaconda3/lib/python3.9/site-packages (from pyobjc>=2.4->pyttsx3) (8.5)
Requirement already satisfied: pyobjc-framework-SearchKit==8.5 in ./opt/anaconda3/lib/python3.9/site-packages (from pyobjc>=2.4->pyttsx3) (8.5)
Requirement already satisfied: pyobjc-framework-SystemExtensions==8.5 in ./opt/anaconda3/lib/python3.9/site-packages (from pyobjc>=2.4->pyttsx3) (8.5)
Requirement already satisfied: pyobjc-framework-Network==8.5 in ./opt/anaconda3/lib/python3.9/site-packages (from pyobjc>=2.4->pyttsx3) (8.5)
Requirement already satisfied: pyobjc-framework-MetricKit==8.5 in ./opt/anaconda3/lib/python3.9/site-packages (from pyobjc>=2.4->pyttsx3) (8.5)
Requirement already satisfied: pyobjc-framework-AuthenticationServices==8.5 in ./opt/anaconda3/lib/python3.9/site-packages (from pyobjc>=2.4->pyttsx3) (8.5)
Requirement already satisfied: pyobjc-framework-DVDPlayback==8.5 in ./opt/anaconda3/lib/python3.9/site-packages (from pyobjc>=2.4->pyttsx3) (8.5)
Requirement already satisfied: pyobjc-framework-StoreKit==8.5 in ./opt/anaconda3/lib/python3.9/site-packages (from pyobjc>=2.4->pyttsx3) (8.5)
Requirement already satisfied: pyobjc-framework-AddressBook==8.5 in ./opt/anaconda3/lib/python3.9/site-packages (from pyobjc>=2.4->pyttsx3) (8.5)
Requirement already satisfied: pyobjc-framework-libdispatch==8.5 in ./opt/anaconda3/lib/python3.9/site-packages (from pyobjc>=2.4->pyttsx3) (8.5)
Requirement already satisfied: pyobjc-framework-FinderSync==8.5 in ./opt/anaconda3/lib/python3.9/site-packages (from pyobjc>=2.4->pyttsx3) (8.5)
Requirement already satisfied: pyobjc-framework-CryptoTokenKit==8.5 in ./opt/anaconda3/lib/python3.9/site-packages (from pyobjc>=2.4->pyttsx3) (8.5)
Requirement already satisfied: pyobjc-framework-Vision==8.5 in ./opt/anaconda3/lib/python3.9/site-packages (from pyobjc>=2.4->pyttsx3) (8.5)
Requirement already satisfied: pyobjc-framework-OpenDirectory==8.5 in ./opt/anaconda3/lib/python3.9/site-packages (from pyobjc>=2.4->pyttsx3) (8.5)
Requirement already satisfied: pyobjc-framework-ImageCaptureCore==8.5 in ./opt/anaconda3/lib/python3.9/site-packages (from pyobjc>=2.4->pyttsx3) (8.5)
Requirement already satisfied: pyobjc-framework-AppleScriptKit==8.5 in ./opt/anaconda3/lib/python3.9/site-packages (from pyobjc>=2.4->pyttsx3) (8.5)
Requirement already satisfied: pyobjc-framework-AppTrackingTransparency==8.5 in ./opt/anaconda3/lib/python3.9/site-packages (from pyobjc>=2.4->pyttsx3) (8.5)
Requirement already satisfied: pyobjc-framework-ScreenTime==8.5 in ./opt/anaconda3/lib/python3.9/site-packages (from pyobjc>=2.4->pyttsx3) (8.5)
Requirement already satisfied: pyobjc-framework-IMServicePlugIn==8.5 in ./opt/anaconda3/lib/python3.9/site-packages (from pyobjc>=2.4->pyttsx3) (8.5)
Requirement already satisfied: pyobjc-framework-QuickLookThumbnailing==8.5 in ./opt/anaconda3/lib/python3.9/site-packages (from pyobjc>=2.4->pyttsx3) (8.5)
Requirement already satisfied: pyobjc-framework-CoreML==8.5 in ./opt/anaconda3/lib/python3.9/site-packages (from pyobjc>=2.4->pyttsx3) (8.5)
Requirement already satisfied: pyobjc-framework-GameKit==8.5 in ./opt/anaconda3/lib/python3.9/site-packages (from pyobjc>=2.4->pyttsx3) (8.5)
Requirement already satisfied: pyobjc-framework-ModelIO==8.5 in ./opt/anaconda3/lib/python3.9/site-packages (from pyobjc>=2.4->pyttsx3) (8.5)
Requirement already satisfied: pyobjc-framework-LinkPresentation==8.5 in ./opt/anaconda3/lib/python3.9/site-packages (from pyobjc>=2.4->pyttsx3) (8.5)
Requirement already satisfied: pyobjc-framework-ScriptingBridge==8.5 in ./opt/anaconda3/lib/python3.9/site-packages (from pyobjc>=2.4->pyttsx3) (8.5)
Requirement already satisfied: pyobjc-framework-Quartz==8.5 in ./opt/anaconda3/lib/python3.9/site-packages (from pyobjc>=2.4->pyttsx3) (8.5)
Requirement already satisfied: pyobjc-framework-IOSurface==8.5 in ./opt/anaconda3/lib/python3.9/site-packages (from pyobjc>=2.4->pyttsx3) (8.5)
Requirement already satisfied: pyobjc-framework-Accounts==8.5 in ./opt/anaconda3/lib/python3.9/site-packages (from pyobjc>=2.4->pyttsx3) (8.5)
Requirement already satisfied: pyobjc-framework-ServiceManagement==8.5 in ./opt/anaconda3/lib/python3.9/site-packages (from pyobjc>=2.4->pyttsx3) (8.5)
Requirement already satisfied: pyobjc-framework-SecurityInterface==8.5 in ./opt/anaconda3/lib/python3.9/site-packages (from pyobjc>=2.4->pyttsx3) (8.5)
Requirement already satisfied: pyobjc-framework-ColorSync==8.5 in ./opt/anaconda3/lib/python3.9/site-packages (from pyobjc>=2.4->pyttsx3) (8.5)
Requirement already satisfied: pyobjc-framework-SafariServices==8.5 in ./opt/anaconda3/lib/python3.9/site-packages (from pyobjc>=2.4->pyttsx3) (8.5)
Requirement already satisfied: pyobjc-framework-ScreenSaver==8.5 in ./opt/anaconda3/lib/python3.9/site-packages (from pyobjc>=2.4->pyttsx3) (8.5)
Requirement already satisfied: pyobjc-framework-OSLog==8.5 in ./opt/anaconda3/lib/python3.9/site-packages (from pyobjc>=2.4->pyttsx3) (8.5)
Requirement already satisfied: pyobjc-framework-Security==8.5 in ./opt/anaconda3/lib/python3.9/site-packages (from pyobjc>=2.4->pyttsx3) (8.5)
Requirement already satisfied: pyobjc-framework-Automator==8.5 in ./opt/anaconda3/lib/python3.9/site-packages (from pyobjc>=2.4->pyttsx3) (8.5)
Requirement already satisfied: pyobjc-framework-AutomaticAssessmentConfiguration==8.5 in ./opt/anaconda3/lib/python3.9/site-packages (from pyobjc>=2.4->pyttsx3) (8.5)
Requirement already satisfied: pyobjc-framework-CoreLocation==8.5 in ./opt/anaconda3/lib/python3.9/site-packages (from pyobjc>=2.4->pyttsx3) (8.5)
Requirement already satisfied: pyobjc-framework-CoreAudioKit==8.5 in ./opt/anaconda3/lib/python3.9/site-packages (from pyobjc>=2.4->pyttsx3) (8.5)
Requirement already satisfied: pyobjc-framework-ContactsUI==8.5 in ./opt/anaconda3/lib/python3.9/site-packages (from pyobjc>=2.4->pyttsx3) (8.5)
Requirement already satisfied: pyobjc-framework-FSEvents==8.5 in ./opt/anaconda3/lib/python3.9/site-packages (from pyobjc>=2.4->pyttsx3) (8.5)
Requirement already satisfied: pyobjc-framework-SoundAnalysis==8.5 in ./opt/anaconda3/lib/python3.9/site-packages (from pyobjc>=2.4->pyttsx3) (8.5)
Requirement already satisfied: pyobjc-framework-NetworkExtension==8.5 in ./opt/anaconda3/lib/python3.9/site-packages (from pyobjc>=2.4->pyttsx3) (8.5)
Requirement already satisfied: pyobjc-framework-ReplayKit==8.5 in ./opt/anaconda3/lib/python3.9/site-packages (from pyobjc>=2.4->pyttsx3) (8.5)
Requirement already satisfied: pyobjc-framework-AdSupport==8.5 in ./opt/anaconda3/lib/python3.9/site-packages (from pyobjc>=2.4->pyttsx3) (8.5)
Requirement already satisfied: pyobjc-framework-InstantMessage==8.5 in ./opt/anaconda3/lib/python3.9/site-packages (from pyobjc>=2.4->pyttsx3) (8.5)
Requirement already satisfied: pyobjc-framework-CoreServices==8.5 in ./opt/anaconda3/lib/python3.9/site-packages (from pyobjc>=2.4->pyttsx3) (8.5)
Requirement already satisfied: pyobjc-framework-UserNotificationsUI==8.5 in ./opt/anaconda3/lib/python3.9/site-packages (from pyobjc>=2.4->pyttsx3) (8.5)
Requirement already satisfied: pyobjc-framework-MediaAccessibility==8.5 in ./opt/anaconda3/lib/python3.9/site-packages (from pyobjc>=2.4->pyttsx3) (8.5)
Installing collected packages: pyttsx3
Successfully installed pyttsx3-2.90
(base) pradeep:~$

```

```python
>>> import pyttsx3
>>> s=pyttsx3.init()
s.say>>> s.say("Hello")
>>> s.runAndWait()
>>> 
```



```python
>>> def myspeak(text):
...     s=pyttsx3.init()
...     s.say(text)
...     s.runAndWait()
... 
>>> myspeak("Welcome to Python!")
>>> 

```

