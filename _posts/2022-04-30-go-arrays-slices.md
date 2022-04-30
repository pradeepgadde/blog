---
layout: single
title:  "Go Arrays and Slices"
date:   2022-04-30 01:56:04 +0530
categories: Go
tags: programming
show_date: true
classes: wide
header:
  teaser: /assets/images/go-yellow.png
author:
  name     : "Go"
  avatar   : "/assets/images/golang.png"

sidebar:
  - title: "Blog"
   
    text: "Checkout other topics"
    nav: my-sidebar

---



## Go Arrays

According to the offical documentation, the type `[n]T` is an array of `n` values of type `T`.

The expression

```go
var a [10]int
```

declares a variable `a` as an array of ten integers.

An array's length is part of its type, so arrays cannot be resized. This seems to be a limitation, but Go seems to provide a  way of working with re-size of arrays. I am yet to learn about it. When we come there, will discuss again!

```go
pradeep:~$cat array.go 
package main

import "fmt"

func main() {
	var a [2]string
	a[0] = "Hello"
	a[1] = "World"
	fmt.Println(a[0], a[1])
	fmt.Println(a)

	primes := [6]int{2, 3, 5, 7, 11, 13}
	fmt.Println(primes)
}
```

```go
pradeep:~$go run array.go 
Hello World
[Hello World]
[2 3 5 7 11 13]
```

## Go Slices

An array has a fixed size. A slice, on the other hand, is a dynamically-sized, flexible view into the elements of an array. In practice, slices are much more common than arrays.

The type `[]T` is a slice with elements of type `T`.

A slice is formed by specifying two indices, a low and high bound, separated by a colon:

```go
a[low : high]
```

This selects a half-open range which includes the first element, but excludes the last one.

The following expression creates a slice which includes elements 1 through 3 of  `a`:

```go
a[1:4]
```

```go
pradeep:~$cat slices.go 
package main

import "fmt"

func main() {
	primes := [6]int{2, 3, 5, 7, 11, 13}

	var s []int = primes[1:4]
	fmt.Println(s)
}

```



```go
pradeep:~$go run slices.go 
[3 5 7]
```

A slice does not store any data, it just describes a section of an underlying array.

Changing the elements of a slice modifies the corresponding elements of its underlying array.

Other slices that share the same underlying array will see those changes.



```go
pradeep:~$cat slice-pointers.go 
package main

import "fmt"

func main() {
	names := [4]string{
		"John",
		"Paul",
		"George",
		"Ringo",
	}
	fmt.Println(names)

	a := names[0:2]
	b := names[1:3]
	fmt.Println(a, b)

	b[0] = "XXX"
	fmt.Println(a, b)
	fmt.Println(names)
}
```

```go
pradeep:~$go run slice-pointers.go 
[John Paul George Ringo]
[John Paul] [Paul George]
[John XXX] [XXX George]
[John XXX George Ringo]
pradeep:~$
```

