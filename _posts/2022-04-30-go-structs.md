---
layout: single
title:  "Go Structs"
date:   2022-04-30 00:56:04 +0530
categories: Go
tags: programming
show_date: true
classes: wide
header:
  teaser: /assets/images/golang.png
author:
  name     : "Go"
  avatar   : "/assets/images/go-red.png"

sidebar:
  - title: "Blog"
   
    text: "Checkout other topics"
    nav: my-sidebar

---
# Go Structs

A struct is a collection of fields.

```go
pradeep:~$cat struct.go 
package main

import "fmt"

type Student struct {
	Name string
	Age int
}

func main() {
	fmt.Println(Student{"Pradeep", 20})
}
```

```go
pradeep:~$go run struct.go
{Pradeep 20}
```

Struct fields are accessed using a `dot`.
```go
pradeep:~$cat struct-fields.go 
package main

import "fmt"

type Student struct {
	Name string
	Age int
}

func main() {
	s := Student{"Pradeep",20}
	s.Name = "John Doe"
	fmt.Println(s.Name)
	fmt.Println(s.Age)
}
```
In this example, we initialized a struct with some fields and then modifed one of the fields.

```go
pradeep:~$go run struct-fields.go
John Doe
20
```

Struct fields can be accessed through a struct pointer.

To access the field `X` of a struct when we have the struct pointer `p` we could write ` (*p).X`. However, that notation is cumbersome, so the language permits us instead to write just `p.X`, without the explicit dereference.

```go
pradeep:~$cat struct-pointers.go 
package main

import "fmt"

type Student struct {
	Name string
	Age int
}

func main() {
	s := Student{"Pradeep",20}
	p := &s
	p.Name = "John Doe"
	fmt.Println(s)

}
```

In this example, we set the field of a struct using the pointer.

```go
pradeep:~$go run struct-pointers.go 
{John Doe 20}
```

A `struct` literal denotes a newly allocated struct value by listing the values of its fields.

You can list just a subset of fields by using the `Name:` syntax. (And the order of named fields is irrelevant.)

The special prefix `&` returns a pointer to the struct value.

```go
pradeep:~$cat struct-literals.go   
package main

import "fmt"

type Student struct {
    Name string
    Age int
}

var (
    s1 = Student{"Pradeep",20}
    s2 = Student{Name: "John"}
    s3 = Student{}
    p = &Student{"Pradeep",20}
)

func main() {
    fmt.Println(s1, p, s2, s3)
}

```

```go
pradeep:~$go run struct-literals.go
{Pradeep 20} &{Pradeep 20} {John 0} { 0}
```

In this last example, `s1` has type `Student`, `s2` has default `Age` of `0`, which is implict, because we have set only the `Name` but not the `Age` and `s3` has default vaule of ` ` for `Name` and `0` for `Age`, and `p` has tupe `*Student`.

