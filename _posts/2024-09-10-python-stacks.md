---
layout: single
title:  "Stacks - Python"
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
# Stacks using Python

Here is a sample code to implement Stack datastructure using Python.

```py
class Node:
    def __init__(self, value):
        self.value = value
        self.next = None

class Stack:
    def __init__(self, value):
        new_node = Node(value)
        self.top = new_node
        self.height = 1

    def pop(self):
        if self.height == 0:
            return None
        temp = self.top
        self.top = self.top.next
        temp.next = None
        self.height -=1
        return temp

    def push(self, value):
        new_node = Node(value)
        if self.height == 0:
            self.top = new_node
        else:
            new_node.next = self.top
            self.top = new_node
        self.height += 1
    
    def print(self):
        temp = self.top
        while temp is not None:
            print(temp.value)
            temp = temp.next
        
my_stack = Stack(4)
my_stack.push(7)
my_stack.push(3)
my_stack.push(12)
my_stack.push(8)
print("Current Stack")
my_stack.print()
my_stack.pop()
print("After popping")
my_stack.print()
my_stack.push(76)
print("Current Stack after adding one more")
my_stack.print()
```

```sh
(base) pradeep:~$/usr/local/bin/python3 /Users/pradeep/LearnPython/stacks_demo.py
Current Stack
8
12
3
7
4
After popping
12
3
7
4
Current Stack after adding one more
76
12
3
7
4
(base) pradeep:~$
```

