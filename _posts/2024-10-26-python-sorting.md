---
layout: single
title:  "Basic Sorting Methods - Python"
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
# Basic Sorting Methods using Python

Here is a sample code to implement basic sorting methods using Python.

```py

def bubble_sort(mylist):
    for i in range(len(mylist)-1,0,-1):
        for j in range(i):
            if mylist[j] > mylist[j+1]:
                mylist[j], mylist[j+1] = mylist[j+1], mylist[j]
    return mylist


def bubble_sort_2(mylist):
    n = len(mylist)
    for i in range(n):
        for j in range(0, n-i-1):
            if mylist[j] > mylist[j+1]:
                mylist[j], mylist[j+1] = mylist[j+1],mylist[j]
    return mylist

def selection_sort(mylist):
    n = len(mylist)
    for i in range(n):
        min_index = i
        for j in range(i+1, n):
            if mylist[j] < mylist[min_index]:
                min_index = j
            mylist[i], mylist[min_index] = mylist[min_index], mylist[i]
    return mylist

def inverted_selection_sort(mylist):
    n = len(mylist)
    for i in range(n):
        max_index = i
        for j in range(i+1,n):
            if mylist[j] > mylist[max_index]:
                max_index = j
            mylist[i], mylist[max_index] = mylist[max_index], mylist[i]
    return mylist

def selection_sort_2(mylist):
    for i in range(len(mylist)-1):
        min_index = i
        for j in range(i+1, len(mylist)):
            if mylist[j] < mylist[min_index]:
                min_index = j
        if i != min_index:
            mylist[i], mylist[min_index] = mylist[min_index],mylist[i]
    return mylist

def insertion_sort(mylist):
    for i in range(1,len(mylist)):
        temp = mylist[i]
        j = i-1
        while temp < mylist[j] and j > -1:
            mylist[j+1] = mylist[j]
            mylist[j] = temp
            j -= 1
    return mylist


print("Range function with start,stop, and step arguements: ")
for i in range(10,0,-1):
    print(i)

print("Range function with stop only arguement: ")
for i in range(10):
    print(i)

print("Sorted list using Bubble sort:", bubble_sort([5,6,4,1,3,2]))

print("Sorted list using Bubble sort:", bubble_sort([9,8,7,6,5,4,3,2,1]))

print("Sorted list using Bublle Sort:", bubble_sort_2([9,8,7,6,5,4,3,2,1]))

print("Sorted list using Selection sort:", selection_sort([9,8,7,6,5,4,3,2,1]))

print("Sorted list using Inverted Selection sort:", inverted_selection_sort([5,6,4,1,3,2]))

print("Sorted list using Selection sort2:", selection_sort([9,8,7,6,5,4,3,2,1]))

print("Sorted list using Insertion sort:", insertion_sort([9,8,7,6,5,4,3,2,1]))

```

## Output

```sh
(base) pradeep:~$ cd /Users/pradeep/LearnPython ; /usr/bin/env /usr/local/bin/python3 /Users/pradeep/.vscode/extensions/ms-python.debugpy-2024.12.0-darwin-x64/bundled/libs/debugpy/adapter/../../
debugpy/launcher 56910 -- /Users/pradeep/LearnPython/bubble_sort.py 
Range function with start,stop, and step arguements: 
10
9
8
7
6
5
4
3
2
1
Range function with stop only arguement: 
0
1
2
3
4
5
6
7
8
9
Sorted list using Bubble sort: [1, 2, 3, 4, 5, 6]
Sorted list using Bubble sort: [1, 2, 3, 4, 5, 6, 7, 8, 9]
Sorted list using Bublle Sort: [1, 2, 3, 4, 5, 6, 7, 8, 9]
Sorted list using Selection sort: [1, 2, 3, 4, 5, 6, 7, 8, 9]
Sorted list using Inverted Selection sort: [6, 5, 4, 2, 3, 1]
Sorted list using Selection sort2: [1, 2, 3, 4, 5, 6, 7, 8, 9]
Sorted list using Insertion sort: [1, 2, 3, 4, 5, 6, 7, 8, 9]
(base) pradeep:~$
```

