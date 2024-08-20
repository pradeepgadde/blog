---
layout: single
title:  "Python Bit-wise Operators"
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
# Python Bit-wise Operators
The Operators:

x << y
    Returns x with the bits shifted to the left by y places (and new bits on the right-hand-side are zeros). This is the same as multiplying x by 2**y. 
x >> y
    Returns x with the bits shifted to the right by y places. This is the same as //'ing x by 2**y. 
x & y
    Does a "bitwise and". Each bit of the output is 1 if the corresponding bit of x AND of y is 1, otherwise it's 0. 
x | y
    Does a "bitwise or". Each bit of the output is 0 if the corresponding bit of x AND of y is 0, otherwise it's 1. 
~ x
    Returns the complement of x - the number you get by switching each 1 for a 0 and each 0 for a 1. This is the same as -x - 1. 
x ^ y
    Does a "bitwise exclusive or". Each bit of the output is the same as the corresponding bit in x if that bit in y is 0, and it's the complement of the bit in x if that bit in y is 1. 
    

```py
# x = 5
# print(bin(x))

# print(bin(~x))

# x = -5
# print(bin(x))
# print(bin(~x))

# x = -2
# print(bin(x))
# print(bin(~x))

# x = -1
# print(bin(x))
# print(bin(~x))

# x = -14
# print(bin(x))
# print(bin(~x))

# x = 16
# print(x<<1)
# print(x>>1)
# print(x<<2)
# print(x>>2)

# x = 32
# print(x>>3)
# print(x<<2)

# x = 32
# y = 45
# print(bin(x))
# print(bin(y))
# print(x ^ y)
# print(x & y)
# print(x | y)
# print(bin(x ^ y))
# print( x >> y)
# print( x << y)

# for x in range(100):
#     print (x & 1)

x = 31
while x:
    print(x, bin(x))
    x = x >> 1
    print(x & 1)
   
```

```sh
(base) pradeep:~$/usr/local/bin/python3 /Users/pradeep/LearnPython/bitwise.py
31 0b11111
1
15 0b1111
1
7 0b111
1
3 0b11
1
1 0b1
0
(base) pradeep:~$
```

```py
x = 32
while x:
    print(x, bin(x))
    x = x >> 1
    print(x & 1)
   
```

```sh
(base) pradeep:~$/usr/local/bin/python3 /Users/pradeep/LearnPython/bitwise.py
32 0b100000
0
16 0b10000
0
8 0b1000
0
4 0b100
0
2 0b10
1
1 0b1
0
(base) pradeep:~$
```

Let's put these together to find the number of one bits in any given number:

```py
def count_bits(x):
    num_bits = 0
    while x:
        num_bits += x & 1
        x >>= 1
    return num_bits

print(count_bits(16))
print(count_bits(15))
print(count_bits(32))
print(count_bits(31))
```

```sh
(base) pradeep:~$/usr/local/bin/python3 /Users/pradeep/LearnPython/count_bits.py
1
4
1
5
(base) pradeep:~$
```

```py
x = 5
print(bin(x))

print(bin(~x))

x = -5
print(bin(x))
print(bin(~x))

x = -2
print(bin(x))
print(bin(~x))

x = -1
print(bin(x))
print(bin(~x))

x = -14
print(bin(x))
print(bin(~x))

x = 16
print(x<<1)
print(x>>1)
print(x<<2)
print(x>>2)

x = 32
print(x>>3)
print(x<<2)

x = 32
y = 45
print(bin(x))
print(bin(y))
print(x ^ y)
print(x & y)
print(x | y)
print(bin(x ^ y))
print( x >> y)
print( x << y)

# for x in range(100):
#     print (x & 1)

x = 32
while x:
    print(x, bin(x))
    x = x >> 1
    print(x & 1)
   

```

```sh
(base) pradeep:~$/usr/local/bin/python3 /Users/pradeep/LearnPython/bitwise.py
0b101
-0b110
-0b101
0b100
-0b10
0b1
-0b1
0b0
-0b1110
0b1101
32
8
64
4
4
128
0b100000
0b101101
13
32
45
0b1101
0
1125899906842624
32 0b100000
0
16 0b10000
0
8 0b1000
0
4 0b100
0
2 0b10
1
1 0b1
0
(base) pradeep:~$
```

