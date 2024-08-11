---
layout: single
title:  "LinkedLists - Python"
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
# LinkedLists using Python

Here is a sample code to implement LinkedLists datastructure using Python.

There are two classes in this code, Node class is to initialize the Linked List Node and the second class is to set head and tails. 

Inside the LinkedList class, we have three functions, one to initialize, one to print and the last one to append to the linkedlist.

```py
class Node:
    def __init__(self, value):
        self.value = value
        self.next = None

class LinkedList:
    def __init__(self, value):
        new_node = Node(value)
        self.head = new_node
        self.tail = new_node
        self.length = 1

    def print_linked_list(self):
        temp = self.head
        # if temp.next is None:
        #     print(temp.value)
        while temp.next is not None:
            print(temp.value)
            temp = temp.next
        if temp.next is None:
            print(temp.value)
    
    def append(self,value):
        new_node = Node(value)
        if self.length ==0 :
            self.head = new_node
            self.tail = new_node
        else:
            self.tail.next = new_node
            self.tail = new_node
        self.length += 1


my_linked_list = LinkedList(4)
my_linked_list.append(3)
my_linked_list.append(5)
my_linked_list.print_linked_list()

```

Here is the sample output when we run the above code

```py
(base) pradeep:~$python dsa_1.py 
4
3
5
(base) pradeep:~$
```

