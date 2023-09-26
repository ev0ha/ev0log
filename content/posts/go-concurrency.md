---
title: "Concurrency basics in Go"
date: 2022-11-13T16:10:41+02:00
draft: false
---

# A look at concurrency primitives in Go

Let's take a look at concurrency primitives in Go: Goroutines, Channels and WaitGroups.
Here is a very simple program with no concurrency features which we will use to iterate on.
We call two functions, each printing a statement and sleeping for a second, from the main function,
which also prints a statement and the time it took the program to run in the end.

```go
package main

import (
	"fmt"
	"time"
)

func main() {
	now := time.Now()
	defer func() {
		fmt.Println(time.Since(now))
	}()

	fmt.Println("Hi from main function!")
	one()
	two()
}

func one() {
	fmt.Println("Hi from function one!")
	time.Sleep(time.Second)
}

func two() {
	fmt.Println("Hi from function two!")
	time.Sleep(time.Second)
}
```
Output:
```
Hi from main function!
Hi from function one!
Hi from function two!
2.000129487s

```
As expected, the functions execute in order and take slightly above 2 seconds to execute, the time they sleep.
But imagine we want function one and two to be run concurrently, in Go we just add 'go' in front of the function call:

```go
	fmt.Println("Hi from main function!")
	go one()
	go two()
```
Output:
```
Hi from main function!
15.99µs
```
Oops! Only the stament from the main function gets printed and the time the program takes to run is only ~16 microseconds!
This happens because when we call a function as a goroutine with 'go', the main function (also a goroutine!) doesn't
wait on the called function to actually execute. This is where WaitGroups come in play! We create a WaitGroup
with the sync package, increment the counter by two before we call our two goroutines, pass the WaitGroup as a pointer
to the functions we run as goroutines and decrement the counter by one inside each function. In the end, we tell our
WaitGroup to wait until all goroutines have run (or rather the counter got decremented to 0).

```go
func main() {
	now := time.Now()
	defer func() {
		fmt.Println(time.Since(now))
	}()

	var wg sync.WaitGroup

	fmt.Println("Hi from main function!")
	wg.Add(2)
	go one(&wg)
	go two(&wg)
	wg.Wait()
}

func one(wg *sync.WaitGroup) {
	fmt.Println("Hi from function one!")
	time.Sleep(time.Second)
	wg.Done()
}

func two(wg *sync.WaitGroup) {
	fmt.Println("Hi from function two!")
	time.Sleep(time.Second)
	wg.Done()
}
```
Output:
```
Hi from main function!
Hi from function two!
Hi from function one!
1.000056387s
```
All the statements get printed and it only took the program ~1 seconds to execute instead of ~2 because we ran
functions one and two concurrently. But functions one and two got executed in different order! Depending on use case
you might not care about the order, but let us assume we do care in this case and we want to synchronize the goroutines.
We will use channels to achieve this. Let us also add a third function. Remember to increment the WaitGroup to 3.
We add a channel 'done' that accepts boolean values, we pass it to our functions as a parameter and send 'true' to
the channel inside each function. In the main function we wait before calling functions two and three until we
receive true from the channel:
```go
func main() {
	now := time.Now()
	defer func() {
		fmt.Println(time.Since(now))
	}()

	var wg sync.WaitGroup

	done := make(chan bool)

	fmt.Println("Hi from main function!")

	wg.Add(3)
	go one(&wg, done)
	<-done
	go two(&wg, done)
	<-done
	go three(&wg, done)
	<-done
	wg.Wait()
}

func one(wg *sync.WaitGroup, done chan bool) {
	fmt.Println("Hi from function one!")
	time.Sleep(time.Second)
	done <- true
	wg.Done()
}

func two(wg *sync.WaitGroup, done chan bool) {
	fmt.Println("Hi from function two!")
	time.Sleep(time.Second)
	done <- true
	wg.Done()
}

func three(wg *sync.WaitGroup, done chan bool) {
	fmt.Println("Hi from function three!")
	time.Sleep(time.Second)
	done <- true
	wg.Done()
}
```
Output:
```
Hi from main function!
Hi from function one!
Hi from function two!
Hi from function three!
3.000301256s

```
We see that the functions now execute in order but they take ~3 seconds, it should take ~1! That happens because we
send true to channel done after we called time.Sleep inside each function, if we call it before we call time.Sleep ...
```go
func one(wg *sync.WaitGroup, done chan bool) {
	fmt.Println("Hi from function one!")
	done <- true
	time.Sleep(time.Second)
	wg.Done()
}

func two(wg *sync.WaitGroup, done chan bool) {
	fmt.Println("Hi from function two!")
	done <- true
	time.Sleep(time.Second)
	wg.Done()
}

func three(wg *sync.WaitGroup, done chan bool) {
	fmt.Println("Hi from function three!")
	done <- true
	time.Sleep(time.Second)
	wg.Done()
}
```
Output:
```
Hi from main function!
Hi from function one!
Hi from function two!
Hi from function three!
1.000673424s

```
Yay, our functions execute in order and concurrently, mission accomplished. This was a first look at basic
concurrency primitives in Go, I hope to learn more advanced patterns in the future and I'm sure I will blog about it
when time comes.
