---
layout: single
title:  "Go Flow Control Statements"
date:   2022-04-24 11:56:04 +0530
categories: Go
tags: programming
toc: true
toc_sticky: true
show_date: true
header:
  teaser: /assets/images/go.png
author:
  name     : "Go"
  avatar   : "/assets/images/go-black.png"

sidebar:
  - title: "Blog"
   
    text: "Checkout other topics"
    nav: my-sidebar

---

# Go Flow Control Statements

In this post, let us go through some of the flow control features of Go language.



## For Loop

Go has only one looping construct, the `for` loop.

The basic `for` loop has three components separated by semicolons:

- the init statement: executed before the first iteration
- the condition expression: evaluated before every iteration
- the post statement: executed at the end of every iteration

The init statement will often be a short variable declaration, and the variables declared there are visible only in the scope of the `for` statement.

The loop will stop iterating once the boolean condition evaluates to `false`.

**Note:** Unlike other languages like C, Java, or JavaScript there are no parentheses surrounding the three components of the `for` statement and the braces `{ }` are always required.

Here is a simple For loop to print the sume of numbers 0 to 9.

```go
pradeep@LearnGo example % cat for.go 
package main

import "fmt"

func main() {
	sum := 0
	for i := 0; i < 10; i++ {
		sum += i
	}
	fmt.Println(sum)
}

```

```go
pradeep@LearnGo example % go run for.go 
45
```

The init and post statements are optional.

So let us try a modified version of the above.

```go
pradeep@LearnGo example % cat for-1.go 
package main

import "fmt"

func main() {
	sum := 1
	for ; sum < 45; {
		sum += sum
	}
	fmt.Println(sum)
}

```

```go
pradeep@LearnGo example % go run for-1.go 
64
```
For is Go's "while"

At that point you can drop the semicolons: C's `while` is spelled `for` in Go.

```go
pradeep@LearnGo example % cat for-2.go 
package main

import "fmt"

func main() {
	sum := 1
	for sum < 45 {
		sum += sum
	}
	fmt.Println(sum)
}
```

```go
pradeep@LearnGo example % go run for-2.go 
64
```



If you omit the loop condition it loops forever, so an infinite loop is compactly expressed.

```go
pradeep@LearnGo example % cat forever.go 
package main

func main() {
	for {
	}
}
```

```go
pradeep@LearnGo example % go run forever.go 
^Csignal: interrupt
```

