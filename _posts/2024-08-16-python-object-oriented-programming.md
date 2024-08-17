---
layout: single
title:  "Python Object Oriented Programming"
categories: Programming
tags: Python
show_date: true
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

## Polymorphism

```py
name = "Pradeep" # str
print(len(name))

my_list = ["pradeep","python"] # list
print(len(my_list))
# That's polymorphism in action, a single function does now
# how to handle different kinds of objects as expected!
```

```sh
(Python4Networking) (base) pradeep:~$/Users/pradeep/LearnPython/Python4Networking/bin/python /Use
rs/pradeep/LearnPython/poly.py
7
2
(Python4Networking) (base) pradeep:~$
```

## Class Methods and Static Methods

```py
import csv


class Item:
    pay_rate = 0.8 # The pay rate after 20% discount
    all = []
    def __init__(self, name: str, price: float, quantity=0):
        # Run validations to the received arguments
        assert price >= 0, f"Price {price} is not greater than or equal to zero!"
        assert quantity >= 0, f"Quantity {quantity} is not greater or equal to zero!"

        # Assign to self object
        self.name = name
        self.price = price
        self.quantity = quantity

        # Actions to execute
        Item.all.append(self)

    def calculate_total_price(self):
        return self.price * self.quantity

    def apply_discount(self):
        self.price = self.price * self.pay_rate

    @classmethod
    def instantiate_from_csv(cls):
        with open('items.csv', 'r') as f:
            reader = csv.DictReader(f)
            items = list(reader)

        for item in items:
            Item(
                name=item.get('name'),
                price=float(item.get('price')),
                quantity=int(item.get('quantity')),
            )

    @staticmethod
    def is_integer(num):
        # We will count out the floats that are point zero
        # For i.e: 5.0, 10.0
        if isinstance(num, float):
            # Count out the floats that are point zero
            return num.is_integer()
        elif isinstance(num, int):
            return True
        else:
            return False

    def __repr__(self):
        return f"Item('{self.name}', {self.price}, {self.quantity})"

item1 = Item("phone",150,4)
print(item1.is_integer(7))
```



```py

class Item:
    @staticmethod
    def is_integer():
        '''
        This should do something that has a relationship
        with the class, but not something that must be unique
        per instance!
        '''
    @classmethod
    def instantiate_from_something(cls):
        '''
        This should also do something that has a relationship
        with the class, but usually, those are used to
        manipulate different structures of data to instantiate
        objects, like we have done with CSV.
        '''

# THE ONLY DIFFERENCE BETWEEN THOSE:
# Static methods are not passing the object reference as the first argument in the background!


# NOTE: However, those could be also called from instances.

item1 = Item()
item1.is_integer()
item1.instantiate_from_something()
```

## Class Inheritance

```py
import csv

class Item:
    pay_rate = 0.8 # The pay rate after 20% discount
    all = []
    def __init__(self, name: str, price: float, quantity=0):
        # Run validations to the received arguments
        assert price >= 0, f"Price {price} is not greater than or equal to zero!"
        assert quantity >= 0, f"Quantity {quantity} is not greater or equal to zero!"

        # Assign to self object
        self.name = name
        self.price = price
        self.quantity = quantity

        # Actions to execute
        Item.all.append(self)

    def calculate_total_price(self):
        return self.price * self.quantity

    def apply_discount(self):
        self.price = self.price * self.pay_rate

    def __repr__(self):
        return f"{self.__class__.__name__}('{self.name}', {self.price}, {self.quantity})"


class Phone(Item):
    def __init__(self, name: str, price: float, quantity=0, broken_phones=0):
        # Call to super function to have access to all attributes / methods
        super().__init__(
            name, price, quantity
        )

        # Run validations to the received arguments
        assert broken_phones >= 0, f"Broken Phones {broken_phones} is not greater or equal to zero!"

        # Assign to self object
        self.broken_phones = broken_phones

phone1 = Phone("iPhone15Pro", 500, 5, 1)

print(Item.all)
```

```sh
(Python4Networking) (base) pradeep:~$/Users/pradeep/LearnPython/Python4Networking/bin/python /Use
rs/pradeep/LearnPython/inheritance.py
[Phone('iPhone15Pro', 500, 5)]
(Python4Networking) (base) pradeep:~$
```

```py
import csv


class Item:
    pay_rate = 0.8 # The pay rate after 20% discount
    all = []
    def __init__(self, name: str, price: float, quantity=0):
        # Run validations to the received arguments
        assert price >= 0, f"Price {price} is not greater than or equal to zero!"
        assert quantity >= 0, f"Quantity {quantity} is not greater or equal to zero!"

        # Assign to self object
        self.name = name
        self.price = price
        self.quantity = quantity

        # Actions to execute
        Item.all.append(self)

    def calculate_total_price(self):
        return self.price * self.quantity

    def apply_discount(self):
        self.price = self.price * self.pay_rate

    

    def __repr__(self):
        return f"{self.__class__.__name__}('{self.name}', {self.price}, {self.quantity})"


class Phone(Item):
    def __init__(self, name: str, price: float, quantity=0, broken_phones=0):
        # Call to super function to have access to all attributes / methods
        super().__init__(
            name, price, quantity
        )

        # Run validations to the received arguments
        assert broken_phones >= 0, f"Broken Phones {broken_phones} is not greater or equal to zero!"

        # Assign to self object
        self.broken_phones = broken_phones
        

phone1 = Phone("iPhone15Pro", 500, 5, 1)

class Laptop(Item):
    def __init__(self, name: str, price: float, quantity=0, broken_laptops=0):
        super().__init__(
            name, price, quantity
        )
        assert broken_laptops >=0, f"Broken Laptops {broken_laptops} is not greater or equal to zero!"
    
        self.broken_laptops = broken_laptops

laptop1 = Laptop("MacbookAir", 1100, 3,1)    

print(Item.all)
```

```sh
(Python4Networking) (base) pradeep:~$/Users/pradeep/LearnPython/Python4Networking/bin/python /Use
rs/pradeep/LearnPython/inheritance.py
[Phone('iPhone15Pro', 500, 5), Laptop('MacbookAir', 1100, 3)]
(Python4Networking) (base) pradeep:~$
```

