---
layout: single
title:  "Merge Sorting - Python"
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
# Merge Sorting using Python

Here is a sample code to implement merge sorting using Python.

```py

def merge(l1,l2):
    c=[]
    i=0
    j=0
    while i < len(l1) and j < len(l2):
        if l1[i] < l2[j]:
            c.append(l1[i])
            i+=1
        else:
            c.append(l2[j])
            j+=1
    while i < len(l1):
        c.append(l1[i])
        i+=1
    while j < len(l2):
        c.append(l2[j])
        j+=1
    return c

def merge_sort(mylist):
    if len(mylist) == 1:
        return mylist
    mid_index = int(len(mylist)/2)
    left = merge_sort(mylist[:mid_index])
    right = merge_sort(mylist[mid_index:])
    return merge(left, right)

original = [4,6,2,1,7,5]

print("Merging two sorted lists: ", merge([1,3,6,7],[2,4,8,9]))
print("Original list is:", original)
print("Sorting after Mergesort:", merge_sort(original))

```

## Output

```sh
(base) pradeep:~$ cd /Users/pradeep/LearnPython ; /usr/bin/env /usr/local/bin/python3 /Users/pradeep/.vscode/extensions/ms-python.debugpy-2024.12.0-darwin-x64/bundled/libs/debugpy/adapter/../../
debugpy/launcher 59368 -- /Users/pradeep/LearnPython/merge_lists.py 
Merging two sorted lists:  [1, 2, 3, 4, 6, 7, 8, 9]
Original list is: [4, 6, 2, 1, 7, 5]
Sorting after Mergesort: [1, 2, 4, 5, 6, 7]
(base) pradeep:~$
```

