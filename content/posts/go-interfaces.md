---
title: "Go: interfaces"
date: 2022-10-29T16:41:52+02:00
draft: false
---

# Let's have a first look at interfaces in Go

Suppose we have two structs, Manager and Developer, and we want to have a function, showInfo, that prints certain
information about them. To achieve this we have to use an interface (or generics, which are available since Go 1.18,
but I'll leave that for another post). Without interfaces the code would look like this:

```go
package main

import "fmt"

type manager struct {
	name   string
	salary int
	email  string
	role   string
}

type developer struct {
	name    string
	salary  int
	email   string
	role    string
	manager manager
}

// We are only interested in name, salary and role of our employees

func showInfoM(m manager) {
	fmt.Printf("Name: %s, Salary: %d, Role: %s\n", m.name, m.salary, m.role)
}

func showInfoD(d developer) {
	fmt.Printf("Name: %s, Salary: %d, Role: %s\n", d.name, d.salary, d.role)
}

func main() {
	bob := manager{"Bob", 7000, "bob@example.com", "Manager"}
	joe := developer{"Joe", 4000, "joe@example.com", "Developer", bob}
	marc := developer{"Marc", 4500, "marc@example.com", "Developer", bob}

	showInfoM(bob)
	showInfoD(joe)
	showInfoD(marc)
}
```
Output:
```
Name: Bob, Salary: 7000, Role: Manager
Name: Joe, Salary: 4000, Role: Developer
Name: Marc, Salary: 4500, Role: Developer
```
The output is fine, but it would be nice if we had one function that works with both types instead of one function
for each type. Before we implement our interface, let's have a look at methods which we will need later on to
implement our interface.
```go
func (m *manager) showInfo() {
	fmt.Printf("Name: %s, Salary: %d, Role: %s\n", m.name, m.salary, m.role)
}

func (d *developer) showInfo() {
	fmt.Printf("Name: %s, Salary: %d, Role: %s\n", d.name, d.salary, d.role)
}

// Inside main function
bob.showInfo()
joe.showInfo()
marc.showInfo()
```
Output:
```
Name: Bob, Salary: 7000, Role: Manager
Name: Joe, Salary: 4000, Role: Developer
Name: Marc, Salary: 4500, Role: Developer
```
You can see how the methods resemble OOP code, just instead of implementing the methods inside classes (which don't
exist in Go) you simply define functions that work on receivers, in this case of type *manager and *developer.
But we don't want to have a function that takes the employee as a receiver, we want one that takes the employee
as a parameter. Here is the employee interface:
```go
type employee interface {
	showInfo()
}
```
That's it! Now every type that implements showInfo is automatically also of type employee which means we can define
a function that take type employee as a parameter and pass any type that implements the interface. Since we already
defined showInfo on our types manager and developer, they implement the interface, the only thing left to do
is to define our showInfo function:
```go
func showInfo(e employee) {
	e.showInfo()
}

// Inside main function
showInfo(&bob)
showInfo(&joe)
showInfo(&marc)
```
Output remains the same:
```
Name: Bob, Salary: 7000, Role: Manager
Name: Joe, Salary: 4000, Role: Developer
Name: Marc, Salary: 4500, Role: Developer
```
There you go, we now have one function we (or anyone using our code) call when we want to print employee information
instead of one function for each type, a welcoming simplification.

