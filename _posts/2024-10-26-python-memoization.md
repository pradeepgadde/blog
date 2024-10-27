---
layout: single
title:  "Memoization - Python"
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
  - title: "Topics"
    nav: my-sidebar
---
# Memoization using Python

Here is a sample code to implement memoization using Python.

```py


#! /Users/pradeep/opt/anaconda3/bin/python3
counter = 0
def fib(n):
    global counter
    counter += 1
    if n==0 or n==1:
        return n
    else:
        return fib(n-1) + fib(n-2)


n = 30
print("\n Fib of ",n, '=', fib(n))   
print("\n Counter: ", counter)   

memo = [None] * 100
new_counter = 0

def fibonacci(n):
    global new_counter
    new_counter += 1

    if memo[n] is not None:
        return memo[n]
    
    if n == 0 or n == 1:
        return n
    
    memo[n] = fibonacci(n-1) + fibonacci(n-2)

    return memo[n]

n = 99
print("\n With Memoization, Fib of ", n, '=', fibonacci(n))   
print("\n With Memoization, Counter: ", new_counter)   
```

## Output

```sh

 Fib of  30 = 832040

 Counter:  2692537

 With Memoization, Fib of  99 = 218922995834555169026

 With Memoization, Counter:  197

```

