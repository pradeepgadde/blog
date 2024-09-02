---
layout: single
title:  "Python Try Except Finally"
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
# Python Try Except Finally

```py
try:
    result = 3/0

except ZeroDivisionError:
    print("You cant divide by zero")

except FloatingPointError:
    print("There is a Floating point error")

finally:
    print("We are done with it")


def demo():
    try:
        x = int(input("Enter a number: "))
        result = x/0
        print("The division result is {}".format(result))
    except ZeroDivisionError:
        print("You cant divide by zero")
    except ValueError:
        print("Invalid input")
    except Exception as e:
        print(f"The exception is {e}")

# demo()

def divide(x,y):
    try:
        result = x/y
    except ZeroDivisionError:
        print("You cant divide by zero")
    else:
        print(f"The result is {result}")
    finally:
        print(f"I am always executed!")

divide(10,2)
divide(5,0)
```



```sh
(base) pradeep:~$/usr/local/bin/python3 /Users/pradeep/LearnPython/try-except-finally.py
You cant divide by zero
We are done with it
The result is 5.0
I am always executed!
You cant divide by zero
I am always executed!
(base) pradeep:~$
```

Printing the exception as output

```py
def divide(x,y):
    try:
        result = x/y
    except Exception as err:
        print(f"You encountered an error {err}")
    else:
        print(f"The division result is {result}")
divide(15,5)
divide(5,None)
```

```sh
(base) pradeep:~$/usr/local/bin/python3 /Users/pradeep/LearnPython/try-except-finally.py
The division result is 3.0
You encountered an error unsupported operand type(s) for /: 'int' and 'NoneType'
(base) pradeep:~$
```

