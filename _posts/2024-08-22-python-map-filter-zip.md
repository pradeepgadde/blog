---
layout: single
title:  "Python Built-in Functions: map, filter, zip"
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
# Python Built-in Functions: map, filter, zip
## Map
Make an iterator that computes the function using arguments from each of the iterables. Stops when the shortest iterable is exhausted.

```py
def square(num):
    return num ** 2

numbers = [1,2,3,4,5]
squares=[]
for i in numbers:
    squares.append(square(i))

print(squares)

squares_with_map=map(square,numbers)

print(list(squares_with_map))
```
## Filter
Return an iterator yielding those items of iterable for which function(item) is true. If function is None, return the items that are true.

```python
def even(num):
    if (num % 2) == 0:
        return True
    return False

numbers = [1,2,3,4,5]
even_numbers = []
for i in numbers:
    if even(i):
        even_numbers.append(i)
    
print(even_numbers)

even_numbers_filter = filter(even,numbers)

print(list(even_numbers_filter))
```

## Zip

The zip object yields n-length tuples, where n is the number of iterables passed as positional arguments to zip(). The i-th element in every tuple comes from the i-th iterable argument to zip(). This continues until the shortest argument is exhausted.

If strict is true and one of the arguments is exhausted before the others, raise a ValueError.

```py
squares = [1,4,9,16,25]
cubes = [1,8,37,64,125]

squres_cubes = zip(squares,cubes)

print(list(squres_cubes))
```

## Combined Output

```py
(base) pradeep:~$/usr/local/bin/python3 /Users/pradeep/LearnPython/map_filter_zip_lists.py
[1, 4, 9, 16, 25]
[1, 4, 9, 16, 25]
[2, 4]
[2, 4]
[(1, 1), (4, 8), (9, 37), (16, 64), (25, 125)]
(base) pradeep:~$
```

As we can see, we get the same output with and without the use of `map` and `filter`.
