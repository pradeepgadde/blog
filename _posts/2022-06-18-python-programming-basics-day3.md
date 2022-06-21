---
layout: single
title:  "Python Programming Basics Day#3"
date:   2022-06-18 16:56:04 +0530
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
## Day3
It is Day#3 of Linux World Python Training.

[https://replit.com/](https://replit.com/) an in-browser IDE worth trying!

## Passing a Tuple to Function

```python
>>> def mysum(x):
...     print(x)
... 
>>> mysum(5)
5
>>> mysum(5,6,7,8)
Traceback (most recent call last):
  File "<stdin>", line 1, in <module>
TypeError: mysum() takes 1 positional argument but 4 were given

```

```python
>>> def mysum(*x):
...     print(x)
... 
>>> mysum(5,6,7,8)
(5, 6, 7, 8)
>>> mysum(5)
(5,)
>>> 
```

Passing a variable size tuple and perform some action on the data

```python
>>> def mysum(*data):
...     j=0
...     for i in data:
...             j+=i
...     return j
... 
>>> mysum(5,6,7,8)
26
>>> mysum(5,6,7,8,9)
35
>>> mysum(5)
5
>>> 
```

What if we pass a list as parameter?

```pyt
>>> mysum([5,6,7,8,9])
Traceback (most recent call last):
  File "<stdin>", line 1, in <module>
  File "<stdin>", line 4, in mysum
TypeError: unsupported operand type(s) for +=: 'int' and 'list'
>>> 
```

`j` is an integer and `i` is a list, so adding them together caused the TypeError.

To fix this, just add a `*` before the list data while passing.

```py
>>> mysum(*[5,6,7,8,9])
35
>>> mysum(*[5,6,7,8,9.12])
35.12
>>> mysum(*[5,6,7,8,9,12])
47
>>> 
```

## All Conditions

```python
>>> math=49
>>> phy=40
>>> econ=67
>>> chem=56
>>> school_condition=[ math >50, econ>50, phy>45, chem>50]
>>> school_condition
[False, True, False, True]
```

```python
>>> type(school_condition)
<class 'list'>
>>> 
```


```pyt
>>> all(school_condition)
False
>>> 
```
Lets change the values to make all conditions True
```pyt
>>> school_condition=[ math >40, econ>30, phy>25, chem>50] # List of conditions
>>> school_condition
[True, True, True, True]
>>> all(school_condition)
True
>>> 
```

```python
>>> if all(school_condition):
...     print("pass")
... else:
...     print("fail")
... 
pass
>>> 
```



```python
>>> 5 >3 and 7>2
True
>>> 5 >3 and 7>9
False
>>> 5 >9 and 7>9
False
```



```python
>>> 5 >9 or  7>9
False
>>> 5 >9 or  7>3
True
>>> 
```

## Any Condition

```python
>>> school_condition=[ math >40, econ>30, phy>55, chem>50]
>>> school_condition
[True, True, False, True]
>>> any(school_condition)
True
>>> 
```



## Nested Lists
Multi-dimensional lists
```python
>>> day1=[45,34,45,23]
>>> day2=[34,45,23,45]
>>> month1=[day1,day2]
>>> type(month1)
<class 'list'>
>>> type(day1)
<class 'list'>
>>> month1
[[45, 34, 45, 23], [34, 45, 23, 45]]
>>> 

```

```python
>>> month1[0]
[45, 34, 45, 23]
>>> month1[1]
[34, 45, 23, 45]
>>> 
```

```python
>>> month1[1][0]
34
>>> month1[1][1]
45
>>> month1[1][2]
23
>>> month1[1][3]
45 
>>> month1[1][2:]
[23, 45]
>>> 
```

```python
>>> month1
[[45, 34, 45, 23], [34, 45, 23, 45]]
>>> dir(month1)
['__add__', '__class__', '__class_getitem__', '__contains__', '__delattr__', '__delitem__', '__dir__', '__doc__', '__eq__', '__format__', '__ge__', '__getattribute__', '__getitem__', '__gt__', '__hash__', '__iadd__', '__imul__', '__init__', '__init_subclass__', '__iter__', '__le__', '__len__', '__lt__', '__mul__', '__ne__', '__new__', '__reduce__', '__reduce_ex__', '__repr__', '__reversed__', '__rmul__', '__setattr__', '__setitem__', '__sizeof__', '__str__', '__subclasshook__', 'append', 'clear', 'copy', 'count', 'extend', 'index', 'insert', 'pop', 'remove', 'reverse', 'sort']
>>> 
```

In list, getting a column from each row is possible with `numpy aray`.

## Numpy

```python
>>> import numpy

>>> 
>>> numpy.array(month1)
array([[45, 34, 45, 23],
       [34, 45, 23, 45]])
>>> final_month=numpy.array(month1)
>>> month1
[[45, 34, 45, 23], [34, 45, 23, 45]]
>>> final_month
array([[45, 34, 45, 23],
       [34, 45, 23, 45]])
>>> type(final_month)
<class 'numpy.ndarray'>
>>> dir(final_month)
['T', '__abs__', '__add__', '__and__', '__array__', '__array_finalize__', '__array_function__', '__array_interface__', '__array_prepare__', '__array_priority__', '__array_struct__', '__array_ufunc__', '__array_wrap__', '__bool__', '__class__', '__complex__', '__contains__', '__copy__', '__deepcopy__', '__delattr__', '__delitem__', '__dir__', '__divmod__', '__doc__', '__eq__', '__float__', '__floordiv__', '__format__', '__ge__', '__getattribute__', '__getitem__', '__gt__', '__hash__', '__iadd__', '__iand__', '__ifloordiv__', '__ilshift__', '__imatmul__', '__imod__', '__imul__', '__index__', '__init__', '__init_subclass__', '__int__', '__invert__', '__ior__', '__ipow__', '__irshift__', '__isub__', '__iter__', '__itruediv__', '__ixor__', '__le__', '__len__', '__lshift__', '__lt__', '__matmul__', '__mod__', '__mul__', '__ne__', '__neg__', '__new__', '__or__', '__pos__', '__pow__', '__radd__', '__rand__', '__rdivmod__', '__reduce__', '__reduce_ex__', '__repr__', '__rfloordiv__', '__rlshift__', '__rmatmul__', '__rmod__', '__rmul__', '__ror__', '__rpow__', '__rrshift__', '__rshift__', '__rsub__', '__rtruediv__', '__rxor__', '__setattr__', '__setitem__', '__setstate__', '__sizeof__', '__str__', '__sub__', '__subclasshook__', '__truediv__', '__xor__', 'all', 'any', 'argmax', 'argmin', 'argpartition', 'argsort', 'astype', 'base', 'byteswap', 'choose', 'clip', 'compress', 'conj', 'conjugate', 'copy', 'ctypes', 'cumprod', 'cumsum', 'data', 'diagonal', 'dot', 'dtype', 'dump', 'dumps', 'fill', 'flags', 'flat', 'flatten', 'getfield', 'imag', 'item', 'itemset', 'itemsize', 'max', 'mean', 'min', 'nbytes', 'ndim', 'newbyteorder', 'nonzero', 'partition', 'prod', 'ptp', 'put', 'ravel', 'real', 'repeat', 'reshape', 'resize', 'round', 'searchsorted', 'setfield', 'setflags', 'shape', 'size', 'sort', 'squeeze', 'std', 'strides', 'sum', 'swapaxes', 'take', 'tobytes', 'tofile', 'tolist', 'tostring', 'trace', 'transpose', 'var', 'view']
>>> final_month
array([[45, 34, 45, 23],
       [34, 45, 23, 45]])
>>> final_month[1]
array([34, 45, 23, 45])
>>> final_month[2]
Traceback (most recent call last):
  File "<stdin>", line 1, in <module>
IndexError: index 2 is out of bounds for axis 0 with size 2
>>> final_month[0]
array([45, 34, 45, 23])
>>> final_month[:]
array([[45, 34, 45, 23],
       [34, 45, 23, 45]])
>>> final_month[:,2]
array([45, 23])
>>> final_month[:,1]
array([34, 45])
>>> final_month[:,2]
array([45, 23])
>>> final_month[:,3]
array([23, 45])
>>> final_month[:,4]
Traceback (most recent call last):
  File "<stdin>", line 1, in <module>
IndexError: index 4 is out of bounds for axis 1 with size 4
>>> final_month[:,0]
array([45, 34])
>>> final_month[:,1:3]
array([[34, 45],
       [45, 23]])
>>> final_month
array([[45, 34, 45, 23],
       [34, 45, 23, 45]])
>>> 
>>> final_month[1,1:3]
array([45, 23])
>>> final_month[0,1:3]
array([34, 45])
>>> 
```

Vector is a one-dimensional array.

Matrix is a two-dimensional array.

## If Else
Python supports Comparison Chain.
```python
>>> exp=5
>>> if exp>2 and exp<10:
...     print("Offer")
... else:
...     print("No")
... 
Offer
>>> if 2 < exp <10: ## Alternate and simple condition
...     print("Offer")
... else:
...     print("No")
... 
Offer
>>> 
```

