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

- x << y
    Returns x with the bits shifted to the left by y places (and new bits on the right-hand-side are zeros). This is the same as multiplying x by 2**y. 
- x >> y
    Returns x with the bits shifted to the right by y places. This is the same as //'ing x by 2**y. 
- x & y
    Does a "bitwise and". Each bit of the output is 1 if the corresponding bit of x AND of y is 1, otherwise it's 0. 
- x | y
    Does a "bitwise or". Each bit of the output is 0 if the corresponding bit of x AND of y is 0, otherwise it's 1. 
- ~ x
    Returns the complement of x - the number you get by switching each 1 for a 0 and each 0 for a 1. This is the same as -x - 1. 
- x ^ y
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

## Finding the Nth bit

```py
def find_nth_bit(x,n):
    return x & (1<<n)
    

def clear_nth_bit(x,n):
    return x & ~(1<<n)

def set_nth_bit(x,n):
    return x | (1<<n)

def toggle_nth_bit(x,n):
    return x ^ (1<<n)

def swap_bits(x,i,j):
    if find_nth_bit(x,i) != find_nth_bit(x,j):
        mask = (1<<i) | (1<<j)
        x =  x ^ mask
    return x

print(find_nth_bit(73,2))

print(clear_nth_bit(73,3))

print(set_nth_bit(73,2))

print(toggle_nth_bit(73,4))

print(swap_bits(73,0,3))

```

```sh
(base) pradeep:~$/usr/local/bin/python3 /Users/pradeep/LearnPython/bitwise_operations
0
65
77
89
64
(base) pradeep:~$
```

## Bit Manipulations

```py
import math

def isOn(S, j):
    return (S & (1<<j))

def setBit(S, j):
    return (S | (1<<j))

def clearBit(S, j):
    return (S & (~(1<<j)))

def toggleBit(S, j):
    return (S ^ (1<<j))

def lowBit(S):
    return (S&(-S))

def setAll(n):
    return ((1<<n)-1)

def modulo(S, N): # returns S % N, where N is a power of 2
    return ((S) & (N-1))

def isPowerOfTwo(S):
    return (not(S & (S - 1)))

def nearestPowerOfTwo(S):
    return 1<<round(math.log2(S))

def turnOffLastBit(S):
    return (S & (S - 1))

def turnOnLastZero(S):
    return ((S) | (S + 1))

def turnOffLastConsecutiveBits(S):
    return ((S) & (S + 1))

def turnOnLastConsecutiveZeroes(S):
    return ((S) | (S-1))


def printSet(vS):           # in binary representation
    print("S = {} = {:b}".format(vS, vS))

def main():
    
    print("1. Representation (all indexing are 0-based and counted from right)")
    S = 34
    printSet(S)
    print()

    print("2. Multiply S by 2, then divide S by 4 (2x2), then by 2")
    S = 34
    printSet(S)
    S = S << 1
    printSet(S)
    S = S >> 2
    printSet(S)
    S = S >> 1
    printSet(S)
    print()


    print("3. Set/turn on the 3-rd item of the set")
    S = 34
    printSet(S)
    S = setBit(S, 3)
    printSet(S)
    print()

    print("4. Check if the 3-rd and then 2-nd item of the set is on?")
    S = 42
    printSet(S)
    T = isOn(S, 3)
    print("T = {}, {}".format(T, "ON" if T else "OFF"))
    T = isOn(S, 2)
    print("T = {}, {}".format(T, "ON" if T else "OFF"))
    print()

    print("5. Clear/turn off the 1-st item of the set")
    S = 42
    printSet(S)
    S = clearBit(S, 1)
    printSet(S)
    print()

    print("6. Toggle the 2-nd item and then 3-rd item of the set")
    S = 40
    printSet(S)
    S = toggleBit(S, 2)
    printSet(S)
    S = toggleBit(S, 3)
    printSet(S)
    print()

    print("7. Check the first bit from right that is on")
    S = 40
    printSet(S)
    T = lowBit(S)
    print("T = {} (this is always a power of 2)".format(T))
    S = 52
    printSet(S)
    T = lowBit(S)
    print("T = {} (this is always a power of 2)".format(T))
    print();

    print("8. Turn on all bits in a set of size n = 6")
    S = setAll(6)
    printSet(S)
    print()

    print("9. Other tricks (not shown in the book)")
    print("8 % 4 = {}".format(modulo(8, 4)))
    print("7 % 4 = {}".format(modulo(7, 4)))
    print("6 % 4 = {}".format(modulo(6, 4)))
    print("5 % 4 = {}".format(modulo(5, 4)))
    print("is {} power of two? {}".format(9, isPowerOfTwo(9)))
    print("is {} power of two? {}".format(8, isPowerOfTwo(8)))
    print("is {} power of two? {}".format(7, isPowerOfTwo(7)))
    for i in range(1, 17):
        print("Nearest power of two of {} is {}".format(i, nearestPowerOfTwo(i)))
    print("S = {}, turn off last bit in S, S = {}".format(40, turnOffLastBit(40)))
    print("S = {}, turn on last zero in S, S = {}".format(41, turnOnLastZero(41)))
    print("S = {}, turn off last consecutive bits in S, S = {}".format(39, turnOffLastConsecutiveBits(39)))
    print("S = {}, turn on last consecutive zeroes in S, S = {}".format(36, turnOnLastConsecutiveZeroes(36)))

main()
```

```sh
(base) pradeep:~$/usr/local/bin/python3 /Users/pradeep/LearnPython/bit_manipulation.py
1. Representation (all indexing are 0-based and counted from right)
S = 34 = 100010

2. Multiply S by 2, then divide S by 4 (2x2), then by 2
S = 34 = 100010
S = 68 = 1000100
S = 17 = 10001
S = 8 = 1000

3. Set/turn on the 3-rd item of the set
S = 34 = 100010
S = 42 = 101010

4. Check if the 3-rd and then 2-nd item of the set is on?
S = 42 = 101010
T = 8, ON
T = 0, OFF

5. Clear/turn off the 1-st item of the set
S = 42 = 101010
S = 40 = 101000

6. Toggle the 2-nd item and then 3-rd item of the set
S = 40 = 101000
S = 44 = 101100
S = 36 = 100100

7. Check the first bit from right that is on
S = 40 = 101000
T = 8 (this is always a power of 2)
S = 52 = 110100
T = 4 (this is always a power of 2)

8. Turn on all bits in a set of size n = 6
S = 63 = 111111

9. Other tricks (not shown in the book)
8 % 4 = 0
7 % 4 = 3
6 % 4 = 2
5 % 4 = 1
is 9 power of two? False
is 8 power of two? True
is 7 power of two? False
Nearest power of two of 1 is 1
Nearest power of two of 2 is 2
Nearest power of two of 3 is 4
Nearest power of two of 4 is 4
Nearest power of two of 5 is 4
Nearest power of two of 6 is 8
Nearest power of two of 7 is 8
Nearest power of two of 8 is 8
Nearest power of two of 9 is 8
Nearest power of two of 10 is 8
Nearest power of two of 11 is 8
Nearest power of two of 12 is 16
Nearest power of two of 13 is 16
Nearest power of two of 14 is 16
Nearest power of two of 15 is 16
Nearest power of two of 16 is 16
S = 40, turn off last bit in S, S = 32
S = 41, turn on last zero in S, S = 43
S = 39, turn off last consecutive bits in S, S = 32
S = 36, turn on last consecutive zeroes in S, S = 39
(base) pradeep:~$
```

