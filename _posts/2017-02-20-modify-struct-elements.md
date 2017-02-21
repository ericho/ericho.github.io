---
layout: post
title: Modify type elements with interfaces
date: 2017-02-20
---

I'm using Golang in a side project at work and today I faced one of my first encounters with interfaces. I needed to use a function to perform the same operation for a group of types, in this case of structs.

So, considering the type:

```go
type Circle struct {
    r float64
    area float64
}
```

And you want to calculate the area, also let's say that for some reason you also want to save the result of the operation inside the struct in the area field. So a `Area()` function can be defined as:

```go
func (c Circle) Area() {
    c.area = c.r*c.r*3.141592
}
```

And you call this function like this: 

```go

func main() {
	c := Circle{1.0, 0.0}
	c.Area()
	fmt.Println(c)
}
```

This example can be run [here](https://play.golang.org/p/lrQVF8Uvaf)

The output will be: 

```
{1 0}
```

Nothing happened on the area field. If we change the `Area` function adding the `c *Circle` like this 

```go
func (c *Circle) Area() {
    c.area = c.r*c.r*3.141592
}
```

Then the correct value is printed, which means that we just changed the value within the struct. That is the result of adding the pointer receiver to the function.

Now, let's say that add another type, the `Square` type with it;s own `Area()` function.

```go
type Square struct {
    side float64
    area float64
}

func (s *Square) Area() {
    s.area = s.side*s.side
}
```
[Run example](https://play.golang.org/p/K5nsLazQb2)

EVerything works as expected. But now we decided that we need a function that does some work including calculate the area for a set of defined types. Something like:

```go
type ShapeCalc interface {
    Area()
}

func PrintAndCalculateArea(s ShapeCalc) {
    s.Area()
    fmt.Println(s)
}
```
[Run full example](https://play.golang.org/p/bVNdE1uP7M)

And this code will fail with:

```
tmp/sandbox555582754/main.go:37: cannot use c (type Circle) as type ShapeCalc in argument to PrintAndCalculateArea:
	Circle does not implement ShapeCalc (Area method has pointer receiver)
tmp/sandbox555582754/main.go:38: cannot use s (type Square) as type ShapeCalc in argument to PrintAndCalculateArea:
	Square does not implement ShapeCalc (Area method has pointer receiver)
```

This basically is complaining about the pointer receiver in the functions, but we need that in order to change the value inside the struct.

The solution to this problem is simple, just add a `&` before the declaration of the structs, something like: 

```go
c := &Circle{1.0, 0.0}
s := &Square{2.0, 0.0}
```

Also it is useful to define constructors, to have a function that returns the pointer to an initialized struct. 

```go
func NewCircle() (*Circle) {
	return &Circle{0.0, 0.0}
}
```
[See full example](https://play.golang.org/p/5eytxTJqUN)
