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

Loops: Iteration and Recursion
Iteration with `for` and `while` loops

# For loop
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
>>> emails=[ n + "gmail.com" for n in names if not n.endswith("gmail.com")]
>>> emails
['pradeepgmail.com', 'sumagmail.com']
>>> 

```

