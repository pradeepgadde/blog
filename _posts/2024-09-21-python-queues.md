---
layout: single
title:  "Queues - Python"
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
# Queues using Python

Here is a sample code to implement Queue datastructure using Python.

```py
class Node:
    def __init__(self, value):
        self.value = value
        self.next = None

class Queue:
    def __init__(self, value):
        new_node = Node(value)
        self.first = new_node
        self.last = new_node
        self.length = 1

    def print_queue(self):
        temp = self.first
        while temp is not None:
            print(temp.value)
            temp = temp.next
    
    def enqueue(self, value):
        new_node = Node(value)
        if self.first is None:
            self.first = new_node
            self.last = new_node
        else:
            self.last.next = new_node
            self.last = new_node
        self.length += 1

    def dequeue(self):
        if self.length == 0:
            return None
        temp = self.first
        if self.length == 1:
            self.first = None
            self.last = None
        else:
            self.first = self.first.next
            temp.next = None
        self.length -=1
        return temp


        

myqueue = Queue(5)
#myqueue.print_queue()
myqueue.enqueue(3)
myqueue.enqueue(7)
myqueue.enqueue(8)
myqueue.print_queue()
myqueue.dequeue()
myqueue.dequeue()
myqueue.dequeue()
myqueue.print_queue()
```

```sh
(base) pradeep:~$/usr/local/bin/python3 /Users/pradeep/LearnPython/queue_demo.py
5
3
7
8
8
(base) pradeep:~$
```

