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

## If Else

Go's `if` statements are like its `for` loops; the expression need not be surrounded by parentheses `( )` but the braces `{ }` are required.



```go
pradeep@LearnGo example % cat if.go 
package main

import (
	"fmt"
)

var x int
func main() {
        if x < 10 {
           fmt.Println(x)
           fmt.Println("Less than 10")
} else {
  fmt.Println(x)
  fmt.Println("Greater than or equal to 10")
   
}
}
```

```go
pradeep@LearnGo example % go run if.go
0
Less than 10
```

Since we have not initialized the variable `x` in this code, it will be assigned a value of `0` and the `if` condition results in true, hence the `if` code block got executed.

Let us change the value of variable `x`.

```go
pradeep@LearnGo example % cat if-else.go 
package main

import (
	"fmt"
)

var x int
func main() {
        if x := 20; x < 10 {
           fmt.Println(x)
           fmt.Println("Less than 10")
} else {
  fmt.Println(x)
  fmt.Println("Greater than or equal to 10")
   
}
}

```

```go
pradeep@LearnGo example % go run if-else.go 
20
Greater than or equal to 10
```

Like `for`, the `if` statement can start with a short statement to execute before the condition.

Variables declared by the statement are only in scope until the end of the `if`.

We can see that Variables declared inside an `if` short statement are also available inside any of the `else` blocks.

## Switch

A `switch` statement is a shorter way to write a sequence of `if - else` statements. It runs the first case whose value is equal to the condition expression.

Go's switch is like the one in C, C++, Java, JavaScript, and PHP, except that Go only runs the selected case, not all the cases that follow. In effect, the `break` statement that is needed at the end of each case in those languages is provided automatically in Go. Another important difference is that Go's switch cases need not be constants, and the values involved need not be integers.

Here is an example showing the usage of `switch` in Go.

```go
pradeep@LearnGo example % cat switch.go 
package main

import (
	"fmt"
	"runtime"
)

func main() {
	fmt.Print("Go runs on ")
	switch os := runtime.GOOS; os {
	case "darwin":
		fmt.Println("OS X.")
	case "linux":
		fmt.Println("Linux.")
	default:
		// freebsd, openbsd,
		// plan9, windows...
		fmt.Printf("%s.\n", os)
	}
}

```

```go
pradeep@LearnGo example % go run switch.go 
Go runs on OS X.
```

Switch cases evaluate cases from top to bottom, stopping when a case succeeds.

For example,

```go
switch i {
case 0:
case f():
}
```

does not call `f` if `i==0`.

The following example shows the order of evaluation

```go
pradeep@LearnGo example % cat switch-order.go 
package main

import (
	"fmt"
	"time"
)

func main() {
	fmt.Println("When's Saturday?")
	today := time.Now().Weekday()
	switch time.Saturday {
	case today + 0:
		fmt.Println("Today.")
	case today + 1:
		fmt.Println("Tomorrow.")
	case today + 2:
		fmt.Println("In two days.")
	default:
		fmt.Println("Too far away.")
	}
}

```

```go
Pradeep@LearnGo example % go run switch-order.go 
When's Saturday?
Too far away.
```

Switch without a condition is the same as `switch true`.

This construct can be a clean way to write long `if-then-else` chains.

```go
pradeep@LearnGo example % cat switch-no-condition.go 
package main

import (
	"fmt"
	"time"
)

func main() {
	t := time.Now()
	switch {
	case t.Hour() < 12:
		fmt.Println("Good morning!")
	case t.Hour() < 17:
		fmt.Println("Good afternoon.")
	default:
		fmt.Println("Good evening.")
	}
}

```

```go
pradeep@LearnGo example % go run switch-no-condition.go 
Good morning!
```

## Defer
A `defer` statement defers the execution of a function until the surrounding function returns.

The deferred call's arguments are evaluated immediately, but the function call is not executed until the surrounding function returns.

```go
pradeep@LearnGo example % cat defer.go 
package main

import "fmt"

func main() {
	defer fmt.Println("world")

	fmt.Println("hello")
}
```

```go
pradeep@LearnGo example % go run defer.go 
hello
world
```
Deferred function calls are pushed onto a stack. When a function returns, its deferred calls are executed in `last-in-first-out` order.

```go
pradeep@LearnGo example % cat defer-multi.go 
package main

import "fmt"

func main() {
	fmt.Println("counting")

	for i := 0; i < 10; i++ {
		defer fmt.Println(i)
	}

	fmt.Println("done")
}

```go
pradeep@LearnGo example % go run defer-multi.go 
counting
done
9
8
7
6
5
4
3
2
1
0
pradeep@LearnGo example % 
```
This concludes the discussion on how to control the flow of our code with conditionals, loops, switches and defers.
