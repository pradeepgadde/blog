---
layout: single
title:  "Python Functions and Decorators"
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
# Python Functions and Decorators

Intro to Decoratiors

Reference:  https://pythonbasics.org/decorators/

In Python, everything is an object and that applies to functions as well. So, you can assign one function to other.

```py
def hello():
    print ("Hello")

print(type(hello))

greet = hello

print (type(greet))

greet()

hello()

print(id(hello))

print(id(greet))
```

Here, we have a function named `hello` and we have assigned this function object to another object named `greet`. As we can see, `greet` becomes an object of function class and does the same job as that of the original function `hello`.

```sh
(base) pradeep:~$/usr/local/bin/python3 /Users/pradeep/LearnPython/decorators.py
<class 'function'>
<class 'function'>
Hello
Hello
4372973024
4372973024
(base) pradeep:~$
```

Both functions return the same. That’s because they refer to the same object, as seen by the `id` function output.

A function can take another function as input and even return a function. 

This is called decorating a function. A decorator takes a function, extends it and returns.

```py
def hello(func):
    def inside():
        print("Hello")
        func()
    return inside

def participant():
    print("Pradeep")

greet = hello(participant)

greet()
```

```sh
(base) pradeep:~$/usr/local/bin/python3 /Users/pradeep/LearnPython/decorators_1.py
Hello
Pradeep
(base) pradeep:~$
```

In the above example, hello() is a decorator. The function participant() is decorated by the function hello().

It wraps the function in the other function.

```py
def hello(func):
    def inside():
        print("Hello")
        func()
    return inside

def participant():
    print("Pradeep")
                                                                                                                                                                                                    
if __name__ == "__main__":                                                                                  
    greet = hello(participant)
    greet()      

```

```sh
(base) pradeep:~$/usr/local/bin/python3 /Users/pradeep/LearnPython/decorators_1.py
Hello
Pradeep
(base) pradeep:~$
```

There is no difference between the two ways.

Python can simplify the use of decorators with the **@ symbol**.

```py
def hello(func):
    def inside():
        print("Hello")
        func()
    return inside

@hello
def participant():
    print("Pradeep")
                                                                                         
                                                                                                            
if __name__ == "__main__":                                                                                  
    participant()

```

```sh
(base) pradeep:~$/usr/local/bin/python3 /Users/pradeep/LearnPython/decorators_1.py
Hello
Pradeep
(base) pradeep:~$
```

`@hello def participant(): ` is same as `greet = hello(participant)  greet() ` but is much simpler and cleaner.

In both cases we apply the decorator to a function.

Parameters can be used with decorators. If you have a funtion that prints the sum a + b, like this

```py
def sum(a, b):
    result = a + b
    print(result)

if __name__ == "__main__":
    sum(5, 3)
```

```sh
(base) pradeep:~$/usr/local/bin/python3 /Users/pradeep/LearnPython/decorators_2.py
8
(base) pradeep:~$
```

You can decorate this function by calling it in another function

```py
def prettysum(func):
    def pretty(a,b):
        print(str(a) + " + " + str(b) + " is: ", end=" ")
        return func(a,b)
    return pretty

@prettysum
def sum(a, b):
    result = a + b
    print(result)

if __name__ == "__main__":
    sum(5, 3)

```

```sh
(base) pradeep:~$/usr/local/bin/python3 /Users/pradeep/LearnPython/decorators_2.py
5 + 3 is:  8
(base) pradeep:~$
```

The function `sum` is wrapped by the function `prettysum`.  This is indicated with the @ symbol above it.

Call the function `sum`, and see that both the logic of the functions `sum` and `prettysum` are run, with parameters.

Here is a realworld example, 

```py
import time

def measure_time(func):
    def wrapper (*arg):
        t = time.time()
        res = func(*arg)
        print ("Function took " + str(time.time()-t) + " seconds to execute.")
        return res
    return wrapper

@measure_time
def myFunction(n):
    time.sleep(n)

if __name__ == "__main__":
    myFunction(5)
```

```sh
(base) pradeep:~$/usr/local/bin/python3 /Users/pradeep/LearnPython/time.py
Function took 5.0040247440338135 seconds to execute.
(base) pradeep:~$
```

This will output the time it took to execute the function myFunction().  By adding one line of code *@measure_time* we can now measure program execution time.
