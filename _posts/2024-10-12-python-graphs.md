---
layout: single
title:  "Graph - Python"
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
# Graphs using Python

Here is a sample code to implement Graph datastructure using Python.

```py
class Graph:
    def __init__(self):
        self.adj_list = {}
    
    def print_graph(self):
        for vertex in self.adj_list:
            print(vertex, ':', self.adj_list[vertex])

    def add_vertex(self, vertex):
        if vertex not in self.adj_list.keys():
            self.adj_list[vertex] = []
            return True
        return False
    
    def add_edge(self,v1,v2):
        if v1 in self.adj_list.keys() and v2 in self.adj_list.keys():
            self.adj_list[v1].append(v2)
            self.adj_list[v2].append(v1)
            return True
        return False

    def remove_edge(self,v1,v2):
        if v1 in self.adj_list.keys() and v2 in self.adj_list.keys():
            try:
                self.adj_list[v1].remove(v2)
                self.adj_list[v2].remove(v1)
            except ValueError:
                pass
            return True
        return False

    def remove_vertex(self, vertex):
        if vertex in self.adj_list.keys():
            for other_vertex in self.adj_list[vertex]:
                self.adj_list[other_vertex].remove(vertex)
            del self.adj_list[vertex]
            return True
        return False


my_graph = Graph()

my_graph.add_vertex('A')
my_graph.add_vertex('B')
my_graph.add_vertex('C')
my_graph.add_vertex('D')
my_graph.add_edge('A','B')
my_graph.add_edge('A','C')
my_graph.add_edge('A','D')
my_graph.add_edge('B','C')
my_graph.add_edge('D','C')
my_graph.add_edge('D','B')
my_graph.print_graph()
print("After removing an edge:")
my_graph.remove_edge('B','C')
my_graph.print_graph()
my_graph.remove_vertex('D')
print("After removin a vertex:")
my_graph.print_graph()

```

```sh
(base) pradeep:~$ cd /Users/pradeep/LearnPython ; /usr/bin/env /usr/local/bin/python3 /Users/pradeep/.vscode/extensions/ms-python.python-2022.18.2/pythonFiles/lib/python/debugp
y/adapter/../../debugpy/launcher 54160 -- /Users/pradeep/LearnPython/graph_demp.py 
A : ['B', 'C', 'D']
B : ['A', 'C', 'D']
C : ['A', 'B', 'D']
D : ['A', 'C', 'B']
After removing an edge:
A : ['B', 'C', 'D']
B : ['A', 'D']
C : ['A', 'D']
D : ['A', 'C', 'B']
After removin a vertex:
A : ['B', 'C']
B : ['A']
C : ['A']
(base) pradeep:~$
```

