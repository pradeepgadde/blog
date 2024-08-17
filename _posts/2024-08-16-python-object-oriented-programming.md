---
layout: single
title:  "Python Object Oriented Programming"
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
# Python Object Oriented Programming

<iframe width="560" height="315" src="https://www.youtube.com/embed/Ej_02ICOIgs?si=TSEUhkfDF6yAvwhP" title="YouTube video player" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture; web-share" referrerpolicy="strict-origin-when-cross-origin" allowfullscreen></iframe>

```py
import csv
class Item:
    rate = 0.8
    all = []
    def __init__(self,name, price, qty=0):
        # print("I am Created! {}".format(name))
        # print(f"I am a class of type, {name}")
        # assert price >=0, f"Price {price} must be greater than 0"
        # assert qty >=0, f"Qty {qty} must be greater than 0"
        self.name = name
        self.price = price
        self.qty = qty
        Item.all.append(self)

    def calculate_price(self):
        return self.price * self.qty

    def empty_item(self):
        return self.qty ** 2

    def apply_discount(self):
        self.price = self.price * Item.rate

    def __repr__(self):
        return(f"Item({self.name},{self.price}, {self.qty}")
    
    @classmethod
    def instantiate_from_csv(cls):
        with open('items.csv', 'r') as f:
            reader = csv.DictReader(f)
            items = list(reader)
        
        for item in items:
            #print(item)
            Item(
                name=item.get('name'),
                price=item.get('price'),
                qty=item.get('qty')
            )


Item.instantiate_from_csv()
print(Item.all)
# item1 = Item("MacOS", 150, 5)
# item2 = Item("Windows", -1500)
# print(Item.all)
# for instance in Item.all:
#     print(instance.name)
#     print(instance.price)
#item1.apply_discount()
# print(item1.price)
# item2.rate = 0.7
#item2.apply_discount()
# print(item2.price)
# print(Item.rate)
# print(item1.rate)
# print(item2.rate)
# print(Item.__dict__)
# print(item1.__dict__)
# print(item1.name)
# print(item2.name)
# print(item1.calculate_price())
# print(item2.calculate_price())
# print(item1.empty_item())
# print(item2.empty_item())

# item1.name = "Phone"
# item1.price = 150
# item1.qty = 5
# print(item1.calculate_price(item1.price, item1.qty))
# print(type(item1))
# print(type(item1.name))
# print(type(item1.qty))
# name = "pradeep"
# print(type(name))
# print(name.upper())
# item2 = Item()
# item2.name = "Laptop"
# item2.price = 1500
# item2.qty = 3
# print(type(item2))
# print(item2.calculate_price(item2.price, item2.qty))
# print(item1.empty_item(item1.qty))
# print(item2.empty_item(item2.qty))
```
