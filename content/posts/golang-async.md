---
title: "Golang Async"
date: "2023-07-21T16:16:31+07:00"
tags: ["tech", "golang"]
draft: false
comment: true
---

> This article is Golang version of [Asynchronous programming with async and await guideline](https://learn.microsoft.com/en-us/dotnet/csharp/asynchronous-programming/). They did really good job to demonstrate how asynchronous programming looks like.
>
> You can find source code [here](https://github.com/ntk148v/testing/tree/master/golang/breakfast).

Throughout this article, we'll write the instructions to make a breakfast.

```unknown
1. Pour a cup of coffee.
2. Heat a pan, then fry two eggs.
3. Fry three slices of bacon.
4. Toast two pieces of bread.
5. Add butter and jam to the toast.
6. Pour a glass of orange juice.
```

Each step takes time to finish, for example, you have to wait X seconds to make a glass of orange juice. Let's write those instructions as Golang:

```go
package main

import (
	"fmt"
	"time"
)

// These structs intentionally empty for the purpose of this example.
// They are simply marker classes for the purpose of demonstration, contain no properties, and serve no other purpose.
type Bacon struct{ fried bool }
type Coffee struct{ poured bool }
type Egg struct{ fried bool }
type Juice struct{ poured bool }
type Toast struct{ toasted, butter, jam bool }

func pourOJ() *Juice {
	fmt.Println("Pouring orange juice")
	time.Sleep(time.Second)
	return &Juice{poured: true}
}

func applyJam(toast *Toast) {
	fmt.Println("Putting jam on the toast")
	time.Sleep(time.Second)
	toast.jam = true
}

func applyButter(toast *Toast) {
	fmt.Println("Putting butter on the toast")
	time.Sleep(time.Second)
	toast.butter = true
}

func toastBread(slices int) *Toast {
	for slice := 0; slice < slices; slice++ {
		fmt.Println("Putting a slices of bread in the toaster")
	}

	fmt.Println("Start toasting...")
	time.Sleep(time.Second * 3)

	return &Toast{toasted: true}
}

func fryBacon(slices int) *Bacon {
	fmt.Printf("Putting %d slices of bacon in the pan\n", slices)
	fmt.Println("Cooking first side of bacon...")
	time.Sleep(3 * time.Second)
	for slice := 0; slice < slices; slice++ {
		fmt.Println("Flipping a slice of bacon")
	}

	fmt.Println("Cooking the second side of bacon...")
	time.Sleep(3 * time.Second)
	fmt.Println("Put bacon on plate")

	return &Bacon{fried: true}
}

func fryEggs(howMany int) *Egg {
	fmt.Println("Warming the egg pan...")
	time.Sleep(3 * time.Second)
	fmt.Printf("Cracking %d eggs\n", howMany)
	time.Sleep(3 * time.Second)
	fmt.Println("Put eggs on plate")

	return &Egg{fried: true}
}

func pourCoffee() *Coffee {
	fmt.Println("Pouring coffee")
	time.Sleep(time.Second)
	return &Coffee{poured: true}
}

func main() {
	start := time.Now()
	cup := pourCoffee()
	fmt.Println("* Coffee is ready:", cup.poured)

	egg := fryEggs(2)
	fmt.Println("* Eggs are ready:", egg.fried)

	bacon := fryBacon(3)
	fmt.Println("* Bacon is ready:", bacon.fried)

	toast := toastBread(2)
	applyButter(toast)
	applyJam(toast)
	fmt.Println("* Toast is ready:", toast.toasted && toast.butter && toast.jam)

	oj := pourOJ()
	fmt.Println("* Orange juice is ready:", oj.poured)
	fmt.Printf("* Breakfast is ready, it took %s\n", time.Since(start))
}
```

![](https://learn.microsoft.com/en-us/dotnet/csharp/asynchronous-programming/media/synchronous-breakfast.png)

Cook it:

```shell
$ go run ./main.go

Pouring coffee
* Coffee is ready: true
Warming the egg pan...
Cracking 2 eggs
Put eggs on plate
* Eggs are ready: true
Putting 3 slices of bacon in the pan
Cooking first side of bacon...
Flipping a slice of bacon
Flipping a slice of bacon
Flipping a slice of bacon
Cooking the second side of bacon...
Put bacon on plate
* Bacon is ready: true
Putting a slices of bread in the toaster
Putting a slices of bread in the toaster
Start toasting...
Putting butter on the toast
Putting jam on the toast
* Toast is ready: true
Pouring orange juice
* Orange juice is ready: true
* Breakfast is ready, it took 19.006261366s
```

We call this one is the **synchronous-breakfast** version. It took roughly 19 seconds (yeah, I known, no one can cook the breakfast in 19 seconds in the real life. But I can't wait for 30 minutes) because the total is the sum of each task.

The computer will block on each statement until the work is complete before moving on to the next statement. That creates an unsatisfying breakfast. The later tasks wouldn't be started until the earlier tasks had been completed. It would take much longer to create the breakfast, and some items would have gotten cold before being served.

If you have experience with cooking, that isn't how we cook. Those instructions should be executed **asynchronously**. You'd start warming the pan for eggs, then start the bacon. You'd put the bread in the toaster, then start the eggs. At each step of the process, you'd start a task, then turn your attention to tasks that are ready for your attention.

Cooking breakfast is a good example of asynchronous work that isn't parallel. One person (or thread/goroutine) can handle all these tasks. Continuing the breakfast analogy, one person can make breakfast asynchronously by starting the next task before the first task completes. For a parallel, you'd need multiple cooks (or threads). One would make the eggs, one the bacon, and so on. Each one would be focused on just that one task.

> To clear up the conflation between _concurrency (asynchronous)_ and _parallel_, you should check out [Rob Pike's talk - Concurrency is not parallelism](https://go.dev/blog/waza-talk).

We need an **asynchronous-breakfast**, but you should know about **Goroutine** before we start. A _goroutine_ is a lightweight thread of execution. The following example is taken from [gobyexample](https://gobyexample.com/goroutines):

```go
package main

import (
    "fmt"
    "time"
)

func f(from string) {
    for i := 0; i < 3; i++ {
        fmt.Println(from, ":", i)
    }
}

func main() {
    // Suppose we have a function call f(s).
    // Here’s how we’d call that in the usual way, running it synchronously.
    f("direct")

    // To invoke this function in a goroutine, use go f(s).
    // This new goroutine will execute concurrently with the calling one.
    go f("goroutine")

    // You can also start a goroutine for an anonymous function call.
    go func(msg string) {
        fmt.Println(msg)
    }("going")

    // Our two function calls are running asynchronously in separate goroutines now.
    time.Sleep(time.Second)
    fmt.Println("done")
}
```

What do we do now? _Put each step in a separate goroutine!_:

```go
package main

import (
	"fmt"
	"time"
)

// These structs intentionally empty for the purpose of this example.
// They are simply marker classes for the purpose of demonstration, contain no properties, and serve no other purpose.
type Bacon struct{ fried bool }
type Coffee struct{ poured bool }
type Egg struct{ fried bool }
type Juice struct{ poured bool }
type Toast struct{ toasted, butter, jam bool }

func pourOJ() *Juice {
	fmt.Println("Pouring orange juice")
	time.Sleep(time.Second)
	return &Juice{poured: true}
}

func applyJam(toast *Toast) {
	fmt.Println("Putting jam on the toast")
	time.Sleep(time.Second)
	toast.jam = true
}

func applyButter(toast *Toast) {
	fmt.Println("Putting butter on the toast")
	time.Sleep(time.Second)
	toast.butter = true
}

func toastBread(slices int) *Toast {
	for slice := 0; slice < slices; slice++ {
		fmt.Println("Putting a slices of bread in the toaster")
	}

	fmt.Println("Start toasting...")
	time.Sleep(time.Second * 3)

	return &Toast{toasted: true}
}

func fryBacon(slices int) *Bacon {
	fmt.Printf("Putting %d slices of bacon in the pan\n", slices)
	fmt.Println("Cooking first side of bacon...")
	time.Sleep(3 * time.Second)
	for slice := 0; slice < slices; slice++ {
		fmt.Println("Flipping a slice of bacon")
	}

	fmt.Println("Cooking the second side of bacon...")
	time.Sleep(3 * time.Second)
	fmt.Println("Put bacon on plate")

	return &Bacon{fried: true}
}

func fryEggs(howMany int) *Egg {
	fmt.Println("Warming the egg pan...")
	time.Sleep(3 * time.Second)
	fmt.Printf("Cracking %d eggs\n", howMany)
	time.Sleep(3 * time.Second)
	fmt.Println("Put eggs on plate")

	return &Egg{fried: true}
}

func pourCoffee() *Coffee {
	fmt.Println("Pouring coffee")
	time.Sleep(time.Second)
	return &Coffee{poured: true}
}

func main() {
	start := time.Now()

	go func() {
		cup := pourCoffee()
		fmt.Println("* Coffee is ready:", cup.poured)
	}()

	go func() {
		egg := fryEggs(2)
		fmt.Println("* Eggs are ready:", egg.fried)
	}()

	go func() {
		bacon := fryBacon(3)
		fmt.Println("* Bacon is ready:", bacon.fried)
	}()

	go func() {
		toast := toastBread(2)
		applyButter(toast)
		applyJam(toast)
		fmt.Println("* Toast is ready:", toast.toasted && toast.butter && toast.jam)
	}()

	go func() {
		oj := pourOJ()
		fmt.Println("* Orange juice is ready:", oj.poured)

	}()

	fmt.Printf("* Breakfast is ready, it took %s\n", time.Since(start))
}
```

```shell
$ go run ./main.go
* Breakfast is ready, it took 5.727µs
```

Oh no, something went wrong here! Nothing was done, we didn't get a breakfast because the program didn't wait for goroutines to finish. To solve this problem, we can use a [sync.WaitGroup](https://gobyexample.com/waitgroups). The WaitGroup is used to wait for all the goroutines launched to finish.

Modify the code and we have a new **asynchronous-breakfast** version:

```go
package main

import (
	"fmt"
	"sync"
	"time"
)

// These structs intentionally empty for the purpose of this example.
// They are simply marker classes for the purpose of demonstration, contain no properties, and serve no other purpose.
type Bacon struct{ fried bool }
type Coffee struct{ poured bool }
type Egg struct{ fried bool }
type Juice struct{ poured bool }
type Toast struct{ toasted, butter, jam bool }

func pourOJ() *Juice {
	fmt.Println("Pouring orange juice")
	time.Sleep(time.Second)
	return &Juice{poured: true}
}

func applyJam(toast *Toast) {
	fmt.Println("Putting jam on the toast")
	time.Sleep(time.Second)
	toast.jam = true
}

func applyButter(toast *Toast) {
	fmt.Println("Putting butter on the toast")
	time.Sleep(time.Second)
	toast.butter = true
}

func toastBread(slices int) *Toast {
	for slice := 0; slice < slices; slice++ {
		fmt.Println("Putting a slices of bread in the toaster")
	}

	fmt.Println("Start toasting...")
	time.Sleep(time.Second * 3)

	return &Toast{toasted: true}
}

func fryBacon(slices int) *Bacon {
	fmt.Printf("Putting %d slices of bacon in the pan\n", slices)
	fmt.Println("Cooking first side of bacon...")
	time.Sleep(3 * time.Second)
	for slice := 0; slice < slices; slice++ {
		fmt.Println("Flipping a slice of bacon")
	}

	fmt.Println("Cooking the second side of bacon...")
	time.Sleep(3 * time.Second)
	fmt.Println("Put bacon on plate")

	return &Bacon{fried: true}
}

func fryEggs(howMany int) *Egg {
	fmt.Println("Warming the egg pan...")
	time.Sleep(3 * time.Second)
	fmt.Printf("Cracking %d eggs\n", howMany)
	time.Sleep(3 * time.Second)
	fmt.Println("Put eggs on plate")

	return &Egg{fried: true}
}

func pourCoffee() *Coffee {
	fmt.Println("Pouring coffee")
	time.Sleep(time.Second)
	return &Coffee{poured: true}
}

func main() {
	start := time.Now()
	var wg sync.WaitGroup

	wg.Add(1)
	go func() {
		defer wg.Done()
		cup := pourCoffee()
		fmt.Println("* Coffee is ready:", cup.poured)
	}()

	wg.Add(1)
	go func() {
		defer wg.Done()
		egg := fryEggs(2)
		fmt.Println("* Eggs are ready:", egg.fried)
	}()

	wg.Add(1)
	go func() {
		defer wg.Done()
		bacon := fryBacon(3)
		fmt.Println("* Bacon is ready:", bacon.fried)
	}()

	wg.Add(1)
	go func() {
		defer wg.Done()
		toast := toastBread(2)
		applyButter(toast)
		applyJam(toast)
		fmt.Println("* Toast is ready:", toast.toasted && toast.butter && toast.jam)
	}()

	wg.Add(1)
	go func() {
		defer wg.Done()
		oj := pourOJ()
		fmt.Println("* Orange juice is ready:", oj.poured)

	}()

	// Wait for all things be done
	wg.Wait()
	fmt.Printf("* Breakfast is ready, it took %s\n", time.Since(start))
}
```

```shell
$ go run ./main.go

Pouring orange juice
Pouring coffee
Warming the egg pan...
Putting 3 slices of bacon in the pan
Cooking first side of bacon...
Putting a slices of bread in the toaster
Putting a slices of bread in the toaster
Start toasting...
* Orange juice is ready: true
* Coffee is ready: true
Putting butter on the toast
Cracking 2 eggs
Flipping a slice of bacon
Flipping a slice of bacon
Flipping a slice of bacon
Cooking the second side of bacon...
Putting jam on the toast
* Toast is ready: true
Put bacon on plate
* Bacon is ready: true
Put eggs on plate
* Eggs are ready: true
* Breakfast is ready, it took 6.00254463s
```

![](https://learn.microsoft.com/en-us/dotnet/csharp/asynchronous-programming/media/whenany-async-breakfast.png)

Breakfast is ready after roughly 6 seconds. Compare to the previous result, it's a huge improvement. The instructions are a bit mess, indicating that the steps were performed concurrency.

<iframe src="https://giphy.com/embed/yJFeycRK2DB4c" width="480" height="384" frameBorder="0" class="giphy-embed" allowFullScreen></iframe><p><a href="https://giphy.com/gifs/yJFeycRK2DB4c">via GIPHY</a></p>

Enjoy your breakfast!
