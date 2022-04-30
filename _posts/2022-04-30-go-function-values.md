---
layout: single
title:  "Go Function Values"
date:   2022-04-30 04:56:04 +0530
categories: Go
tags: programming
show_date: true
classes: wide
header:
  teaser: /assets/images/go-red.png
author:
  name     : "Go"
  avatar   : "/assets/images/golang.png"

sidebar:
  - title: "Blog"
   
    text: "Checkout other topics"
    nav: my-sidebar

---

## Go Function Values

Functions are values too. They can be passed around just like other values.

Function values may be used as function arguments and return values.

```go
pradeep:~$cat function-values.go 
package main

import (
	"fmt"
	"math"
)

func compute(fn func(float64, float64) float64) float64 {
	return fn(3, 4)
}

func main() {
	hypot := func(x, y float64) float64 {
		return math.Sqrt(x*x + y*y)
	}
	fmt.Println(hypot(5, 12))

	fmt.Println(compute(hypot))
	fmt.Println(compute(math.Pow))
}
```

```go
pradeep:~$go run function-values.go 
13
5
81
```

Go functions may be closures. A closure is a function value that references variables from outside its body. The function may access and assign to the referenced variables; in this sense the function is "bound" to the variables.

For example, the adder function returns a closure. Each closure is bound to its own sum variable.

```go
pradeep:~$cat function-closures.go 
package main

import "fmt"

func adder() func(int) int {
	sum := 0
	return func(x int) int {
		sum += x
		return sum
	}
}

func main() {
	pos, neg := adder(), adder()
	for i := 0; i < 10; i++ {
		fmt.Println(
			pos(i),
			neg(-2*i),
		)
	}
}
```

```go
pradeep:~$go run function-closures.go 
0 0
1 -2
3 -6
6 -12
10 -20
15 -30
21 -42
28 -56
36 -72
45 -90
```

This post concludes the discussion of Go language basics.

