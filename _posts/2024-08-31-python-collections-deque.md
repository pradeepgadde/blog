---
layout: single
title:  "Python Collections Deque"
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
# Python Collections Deque

```py
import collections

myll = collections.deque()

print(f"At the beginning {myll}")

myll.append('first')
print(f"After appending one node {myll}")

myll.append('second')
myll.append('third')
print(f"After appending two mode nodes {myll}")

myll.insert(1,'fourth')
print(f"After inserting one node at the first place {myll}")

myll.insert(0,'fifth')
print(f"After inserting one node at the zeroth place {myll}")

myll.pop()
print(f"After popping {myll}")

myll.remove('fourth')
print(f"After removing fourth {myll}")

myll.reverse()
print(f"After reversing {myll}")

myll.append('third')
myll.append('fourth')

print(myll)
myll.rotate()
print(f"After rotating {myll}")

another_ll=collections.deque([1,2,3,4,5])
print(another_ll)
another_ll.rotate()
print(f"After rotating {another_ll}")
another_ll.appendleft(6)
print(f"After appending at the beginning {another_ll}")

another_ll.popleft()
print(f"After popping at the beginning {another_ll}")

another_ll.pop()
print(f"After popping at the end {another_ll}")
another_ll[1]=4
print(f"After modifying the value at index 1 {another_ll}")

mylist=[12,13,14]
another_ll.extend(mylist)
print(f"After extending at the end {another_ll}")
mytuple=(21,22,23)
another_ll.extendleft(mytuple)
print(f"After extending at the beginning {another_ll}")
secondlist=[31,32]
another_ll.insert(2,secondlist)
print(f"After extending at the specified slot {another_ll}")
another_ll.rotate(2)
print(f"After rotating by two {another_ll}")
print(f"The length of the  list is  {another_ll.maxlen}")
another_ll.clear()
print(f"After clearing the queue {another_ll}")

lldemo=collections.deque([],5)
lldemo.extend([1,2,3,4,5])
print(f"demo of creating a list with max limit {lldemo}")
lldemo.append(6)
print(f"demo of appending a list with max limit {lldemo}")
lldemo.append(7)
print(f"demo of appending a list with max limit {lldemo}")
print(f"The length of the  list is  {lldemo.maxlen}")
```



```sh
(base) pradeep:~$/usr/local/bin/python3 /Users/pradeep/LearnPython/ll_libraies_demo.py
At the beginning deque([])
After appending one node deque(['first'])
After appending two mode nodes deque(['first', 'second', 'third'])
After inserting one node at the first place deque(['first', 'fourth', 'second', 'third'])
After inserting one node at the zeroth place deque(['fifth', 'first', 'fourth', 'second', 'third'])
After popping deque(['fifth', 'first', 'fourth', 'second'])
After removing fourth deque(['fifth', 'first', 'second'])
After reversing deque(['second', 'first', 'fifth'])
deque(['second', 'first', 'fifth', 'third', 'fourth'])
After rotating deque(['fourth', 'second', 'first', 'fifth', 'third'])
deque([1, 2, 3, 4, 5])
After rotating deque([5, 1, 2, 3, 4])
After appending at the beginning deque([6, 5, 1, 2, 3, 4])
After popping at the beginning deque([5, 1, 2, 3, 4])
After popping at the end deque([5, 1, 2, 3])
After modifying the value at index 1 deque([5, 4, 2, 3])
After extending at the end deque([5, 4, 2, 3, 12, 13, 14])
After extending at the beginning deque([23, 22, 21, 5, 4, 2, 3, 12, 13, 14])
After extending at the specified slot deque([23, 22, [31, 32], 21, 5, 4, 2, 3, 12, 13, 14])
After rotating by two deque([13, 14, 23, 22, [31, 32], 21, 5, 4, 2, 3, 12])
The length of the  list is  None
After clearing the queue deque([])
demo of creating a list with max limit deque([1, 2, 3, 4, 5], maxlen=5)
demo of appending a list with max limit deque([2, 3, 4, 5, 6], maxlen=5)
demo of appending a list with max limit deque([3, 4, 5, 6, 7], maxlen=5)
The length of the  list is  5
(base) pradeep:~$
```

