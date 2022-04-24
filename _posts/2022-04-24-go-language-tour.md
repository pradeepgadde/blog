---
layout: single
title:  "Go Language Tour"
date:   2022-04-24 11:55:04 +0530
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

# Go Tour

Here is another example to print current time. Here we hav imported one mode module called `time` and using the `time.Now()` function.

```sh
pradeep@LearnGo example % cat time.go 
package main

import (
	"fmt"
	"time"
)

func main() {
	fmt.Println("Welcome to Go!")

	fmt.Println("The time is", time.Now())
}
pradeep@LearnGo example % 

```
Let us run this.
```sh
pradeep@LearnGo example % go run time.go 
Welcome to Go!
The time is 2022-04-24 20:04:17.585982 +0530 IST m=+0.000212275
```

## Packages
Every Go program is made up of packages. Programs start running in package `main`.

Here is a simple program that is using the packages with import paths `fmt` and `math/rand`.

By convention, the package name is the same as the last element of the import path. For instance, the `math/rand` package comprises files that begin with the statement package `rand`.

```go
pradeep@LearnGo example % cat random.go 
package main

import (
	"fmt"
	"math/rand"
)

func main() {
	fmt.Println("My favorite number is", rand.Intn(10))
}
pradeep@LearnGo example % 
```
Let us run it couple of times.
```sh
pradeep@LearnGo example % go run random.go 
My favorite number is 1
pradeep@LearnGo example % go run random.go
My favorite number is 1
pradeep@LearnGo example % go run random.go
My favorite number is 1
pradeep@LearnGo example % 
```

Each time we run the example program `rand.Intn`  returned the same number, `1` in our case. The actual number does not matter.

To see a different number, we need to seed the number generato using  `rand.Seed`.

```go
pradeep@LearnGo example % cat random-with-seed.go 
package main

import (
  "fmt"
  "math/rand"
  "time"
)

func main() {
  // seed to get different result every time
  rand.Seed(time.Now().UnixNano())
  fmt.Println(rand.Intn(50))
  fmt.Println(rand.Float64())
}
pradeep@LearnGo example %
```
> Note: In Go, sinlge line comments starts with `//`

In this program, we are seeding the random function with time and printing a random Integer and a random Floating point.

Let us run this new program couple of times.

```sh
pradeep@LearnGo example % go run random-with-seed.go 
7
0.6662261095500185
pradeep@LearnGo example % go run random-with-seed.go
33
0.44321924444513516
pradeep@LearnGo example % go run random-with-seed.go
37
0.006576635999428069
pradeep@LearnGo example % go run random-with-seed.go
30
0.8171676592299928
pradeep@LearnGo example % 
```
We can see different values with each run.

## Imports
We can group the imports into a parenthesized, *factored* import statement.

We can also write multiple import statements, like:
```go
import "fmt"
import "math"
```
But, according to documentation,  it is good style to use the factored import statement.

```go
pradeep@LearnGo example % cat sqaureroot.go 
package main

import (
	"fmt"
	"math"
)

func main() {
	fmt.Printf("Now you have %g problems.\n", math.Sqrt(7))
}

```



Let us run this,

```sh
pradeep@LearnGo example % go run sqaureroot.go 
Now you have 2.6457513110645907 problems.
```



## Exported Names

In Go, a name is exported if it begins with a capital letter. For  `Pi`, which is exported from the `math` package.

When importing a package, you can refer only to its exported names. Any "unexported" names are not accessible from outside the package.

Let us use a lower case letter for `Pi` and see what happens!

```go
pradeep@LearnGo example % cat exported-names.go 
package main

import (
	"fmt"
	"math"
)

func main() {
	fmt.Println(math.pi)
}
pradeep@LearnGo example % 

```

Run this

```sh
pradeep@LearnGo example % go run exported-names.go 
# command-line-arguments
./exported-names.go:9:19: undefined: math.pi
```

We can see an error ,`undefined: math.pi`.

Let us correct the syntax and re-run

```go
pradeep@LearnGo example % cat exported-names.go 
package main

import (
	"fmt"
	"math"
)

func main() {
	fmt.Println(math.Pi)
}
pradeep@LearnGo example % 

```

```sh
pradeep@LearnGo example % go run exported-names.go 
3.141592653589793
```



## Functions

A function can take zero or more arguments.

In this example, `add` takes two parameters of type `int`.

Notice that the type comes *after* the variable name.

```go
pradeep@LearnGo example % cat add.go 
package main

import "fmt"

func add(x int, y int) int {
	return x + y
}

func main() {
	fmt.Println(add(42, 13))
}
pradeep@LearnGo example % 

```

Let us run this

```sh
pradeep@LearnGo example % go run add.go 
55
pradeep@LearnGo example % 
```

When two or more consecutive named function parameters share a type, you can omit the type from all but the last.

In this example, we can shorten `x int, y int` to `x, y int`.

Let us try this variant also

```go
pradeep@LearnGo example % cat add-1.go 
package main

import "fmt"

func add(x, y int) int {
	return x + y
}

func main() {
	fmt.Println(add(42, 13))
}


pradeep@LearnGo example % 
```

run it 

```sh
pradeep@LearnGo example % go run add-1.go 
55
pradeep@LearnGo example % 
```

We can see same result.



A function can return any number of results.

The `swap` function returns two strings.

```go
pradeep@LearnGo example % cat swap.go 
package main

import "fmt"

func swap(x, y string) (string, string) {
	return y, x
}

func main() {
	a, b := swap("hello", "world")
	fmt.Println(a, b)
}

```

```sh
pradeep@LearnGo example % go run swap.go 
world hello
```

Go's return values may be named. If so, they are treated as variables defined at the top of the function.

These names should be used to document the meaning of the return values.

A `return` statement without arguments returns the named return values. This is known as a **naked** return.

Naked return statements should be used only in short functions, as with the example shown here. They can harm readability in longer functions.

```go
pradeep@LearnGo example % cat naked-reutrn.go 
package main

import "fmt"

func split(sum int) (x, y int) {
	x = sum * 4 / 9
	y = sum - x
	return
}

func main() {
	fmt.Println(split(17))
}
pradeep@LearnGo example % 
```

```sh
pradeep@LearnGo example % go run naked-reutrn.go 
7 10
```

