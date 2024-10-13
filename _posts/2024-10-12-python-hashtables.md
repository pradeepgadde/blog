---
layout: single
title:  "Hash Table - Python"
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
# Hash Tables using Python

Here is a sample code to implement Hash Table datastructure using Python.

```py
class HashTable:
    def __init__(self, size=7):
        self.data_map = [None] * size
    
    def __hash(self, key):
        my_hash = 0
        for letter in key:
            my_hash = (my_hash + ord (letter)*23) % len(self.data_map)
        return my_hash

    def print_table(self):
        for i, val in enumerate(self.data_map):
            print(i, ":", val)
    
    def set_item(self, key, value):
        index = self.__hash(key)
        if self.data_map[index] == None:
            self.data_map[index] = []
        self.data_map[index].append([key, value])
    
    def get_item(self, key):
        index = self.__hash(key)
        if self.data_map[index] is not None:
            for i in range(len(self.data_map[index])):
                if self.data_map[index][i][0] == key:
                    return self.data_map[index][i][1]
        return None

    def keys(self):
        all_keys = []
        for i in range(len(self.data_map)):
            if self.data_map[i] is not None:
                for j in range(len(self.data_map[i])):
                    all_keys.append(self.data_map[i][j][0])
        return all_keys


my_hash_table = HashTable()
my_hash_table.set_item('SRX',1600)
my_hash_table.set_item('MX',80)
my_hash_table.set_item('QFX',10016)
my_hash_table.set_item('ACX',7100)

my_hash_table.print_table()
print(my_hash_table.get_item('MX'))
print(my_hash_table.get_item('SRX'))
print(my_hash_table.get_item('QFX'))
print(my_hash_table.get_item('ACX'))

print(my_hash_table.keys())

```

```sh
(base) pradeep:~$ cd /Users/pradeep/LearnPython ; /usr/bin/env /usr/local/bin/python3 /Users/pradeep/.vscode/extensions/ms-python.python-2022.18.2/pythonFiles/lib/python/debugp
y/adapter/../../debugpy/launcher 53420 -- /Users/pradeep/LearnPython/htdemo.py 
0 : None
1 : [['MX', 80]]
2 : [['SRX', 1600], ['QFX', 10016]]
3 : None
4 : None
5 : None
6 : [['ACX', 7100]]
80
1600
10016
7100
['MX', 'SRX', 'QFX', 'ACX']
(base) pradeep:~$
```

