---
layout: single
title:  "Go Pointers"
date:   2022-04-25 00:56:04 +0530
categories: Programming
tags: Go
show_date: true
classes: wide
header:
  teaser: /assets/images/go.png
author:
  name     : "Go"
  avatar   : "/assets/images/golang.png"

sidebar:
  - title: "Blog"
   
    text: "Checkout other topics"
    nav: my-sidebar

---

# Pointers

Go has pointers. A pointer holds the memory address of a value.

The type `*T` is a pointer to a `T` value. Its zero value is `nil`.

```
var p *int
```

The `&` operator generates a pointer to its operand.

```
i := 42
p = &i
```

The `*` operator denotes the pointer's underlying value.

```
fmt.Println(*p) // read i through the pointer p
*p = 21         // set i through the pointer p
```

This is known as **dereferencing** or **indirecting**.

Unlike C, Go has no pointer arithmetic.



```go
pradeep@LearnGo example % cat pointers.go 
package main

import "fmt"

func main() {
	i, j := 42, 2701

	p := &i         // point to i
	fmt.Println(*p) // read i through the pointer
	*p = 21         // set i through the pointer
	fmt.Println(i)  // see the new value of i

	p = &j         // point to j
	*p = *p / 37   // divide j through the pointer
	fmt.Println(j) // see the new value of j
}

```



```go
pradeep@LearnGo example % go run pointers.go 
42
21
73
```

