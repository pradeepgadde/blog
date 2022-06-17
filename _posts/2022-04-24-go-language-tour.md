---
layout: single
title:  "Go Language Tour"
date:   2022-04-24 11:55:04 +0530
categories: Programming
tags: Go
toc: true
toc_sticky: true
show_date: true
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

# Go Tour

In our previous post on [Go Basics](https://pradeepgadde.com/blog/go/2022/04/24/go-basics.html), we installed Go and ran our first Go code.

Here is another example to print current time. Here we hav imported one mode module called `time` and using the `time.Now()` function.

```go
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
```go
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
```go
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

```go
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

```go
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

```go
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

```go
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

```go
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

```go
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

```go
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

```go
pradeep@LearnGo example % go run naked-reutrn.go 
7 10
```

## Variables
The `var` statement declares a list of variables; as in function argument lists, the type is last.

A `var` statement can be at package or function level. We see both in this example.
```go
pradeep@LearnGo example % cat vardemo.go 
package main

import "fmt"

var c, python, java bool

func main() {
	var i int
	fmt.Println(i, c, python, java)
}
```

```go
pradeep@LearnGo example % go run vardemo.go 
0 false false false
```
In this, code three variables `c`, `python`, and `java` are of type `bool`. One variable `i` is of type `int`.


Since none of these are initialized, we see a value of `0` for integer and `false` for boolean variables.

A var declaration can include initializers, one per variable.

If an initializer is present, the type can be omitted; the variable will take the type of the initializer.

Variables declared without an explicit initial value are given their zero value.

The zero value is:

`0` for numeric types,
`false` for the boolean type, and
`""` (the empty string) for strings.
One more example showing this zero values.

```go
pradeep@LearnGo example % cat zero.go 
package main

import "fmt"

func main() {
	var i int
	var f float64
	var b bool
	var s string
	fmt.Printf("%v %v %v %q\n", i, f, b, s)
}
```

```go
pradeep@LearnGo example % go run zero.go 
0 0 false ""
```

Here is another example with variable initialization.

```go
pradeep@LearnGo example % cat var-with-init.go 
package main

import "fmt"

var i, j int = 1, 2

func main() {
	var c, python, java = true, false, "no!"
	fmt.Println(i, j, c, python, java)
}
pradeep@LearnGo example % 
```
Let us run this
```go
pradeep@LearnGo example % go run var-with-init.go 
1 2 true false no!
```

Inside a function, the `:=` short assignment statement can be used in place of a `var` declaration with implicit type.

Outside a function, every statement begins with a keyword (var, func, and so on) and so the `:=` construct is not available.

Here is another one showing this usage.

```go
pradeep@LearnGo example % cat shortvar.go 
package main

import "fmt"

func main() {
	var i, j int = 1, 2
	k := 3
	c, python, java := true, false, "no!"

	fmt.Println(i, j, k, c, python, java)
}

```

Run this
```go
pradeep@LearnGo example % go run shortvar.go 
1 2 3 true false no!
```

The example shows variables of several types, and also that variable declarations may be "factored" into blocks, as with `import` statements.

```go
pradeep@LearnGo example % cat go-basic-types.go 
package main

import (
	"fmt"
	"math/cmplx"
)

var (
	ToBe   bool       = false
	MaxInt uint64     = 1<<64 - 1
	z      complex128 = cmplx.Sqrt(-5 + 12i)
)

func main() {
	fmt.Printf("Type: %T Value: %v\n", ToBe, ToBe)
	fmt.Printf("Type: %T Value: %v\n", MaxInt, MaxInt)
	fmt.Printf("Type: %T Value: %v\n", z, z)
}
```

Run this 
```go
pradeep@LearnGo example % go run go-basic-types.go 
Type: bool Value: false
Type: uint64 Value: 18446744073709551615
Type: complex128 Value: (2+3i)
```
In this example, we are making use of `Printf` function with `%T` and `%v` to print the type and value of a variable.

## Type Conversion
Unlike in C, in Go assignment between items of different type requires an explicit conversion. The expression `T(v)` converts the value v to the type T.

```go
pradeep@LearnGo example % cat typeconversion.go 
package main

import (
	"fmt"
	"math"
)

func main() {
	var x, y int = 3, 4
	var f float64 = math.Sqrt(float64(x*x + y*y))
	var z uint = uint(f)
	fmt.Println(x, y, z)
}
```

```go
pradeep@LearnGo example % go run typeconversion.go 
3 4 5
```

When declaring a variable without specifying an explicit type (either by using the `:=` syntax or `var = ` expression syntax), the variable's type is inferred from the value on the right hand side.

When the right hand side of the declaration is typed, the new variable is of that same type.

```go
pradeep@LearnGo example % cat type-inference.go 
package main

import "fmt"

func main() {
	v := 42 // change me!
	fmt.Printf("v is of type %T\n", v)
}

```

```go
pradeep@LearnGo example % go run type-inference.go 
v is of type int
```

Let us change the value of `v` to some other type and run it again.
```go
pradeep@LearnGo example % cat type-inference.go 
package main

import "fmt"

func main() {
	v := "Pradeep" // change me!
	fmt.Printf("v is of type %T\n", v)
}
```

```go
pradeep@LearnGo example % go run type-inference.go 
v is of type string
```

## Constants
Constants are declared like variables, but with the `const` keyword.

Constants can be character, string, boolean, or numeric values.

Constants cannot be declared using the `:=` syntax.

```go
pradeep@LearnGo example % cat constants.go 
package main

import "fmt"

const Pi = 3.14

func main() {
	const World = "Kubernetes"
	fmt.Println("Hello", World)
	fmt.Println("Happy", Pi, "Day")

	const Truth = true
	fmt.Println("Go rules?", Truth)
}

```

```go
pradeep@LearnGo example % go run constants.go 
Hello Kubernetes
Happy 3.14 Day
Go rules? true
```

## Numeric Constants
Numeric constants are high-precision values.

An untyped constant takes the type needed by its context.

```go
pradeep@LearnGo example % cat numeric-constants.go 
package main

import "fmt"

const (
	// Create a huge number by shifting a 1 bit left 100 places.
	// In other words, the binary number that is 1 followed by 100 zeroes.
	Big = 1 << 100
	// Shift it right again 99 places, so we end up with 1<<1, or 2.
	Small = Big >> 99
)

func needInt(x int) int { return x*10 + 1 }
func needFloat(x float64) float64 {
	return x * 0.1
}

func main() {
	fmt.Println(needInt(Small))
        fmt.Println(needInt(Big))
	fmt.Println(needFloat(Small))
	fmt.Println(needFloat(Big))
}
```

```go
pradeep@LearnGo example % go run numeric-constants.go 
# command-line-arguments
./numeric-constants.go:20:29: cannot use Big (untyped int constant 1267650600228229401496703205376) as int value in argument to needInt (overflows)

pradeep@LearnGo example % 
```

Note, An int can store at maximum a 64-bit integer, and sometimes less.

Let us remove that line,
```go
pradeep@LearnGo example % cat numeric-constants.go 
package main

import "fmt"

const (
	// Create a huge number by shifting a 1 bit left 100 places.
	// In other words, the binary number that is 1 followed by 100 zeroes.
	Big = 1 << 100
	// Shift it right again 99 places, so we end up with 1<<1, or 2.
	Small = Big >> 99
)

func needInt(x int) int { return x*10 + 1 }
func needFloat(x float64) float64 {
	return x * 0.1
}

func main() {
	fmt.Println(needInt(Small))
	fmt.Println(needFloat(Small))
	fmt.Println(needFloat(Big))
}
```


```go
pradeep@LearnGo example % go run numeric-constants.go 
21
0.2
1.2676506002282295e+29
```

This concludes our Go Tour where in we looked at the basic components of any Go program.