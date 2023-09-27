---
title: "Go Concurrency에 대해 알아보자"
date: 2023-09-23T21:10:58+09:00
tags:
  - golang
  - concurrency
categories:
  - golang
series:
  - Concurrency
published: true
---

## Go Concurrency

### CSP란 ?

Communicating Sequential Processes (CSP)는 병렬 프로그래밍의 복잡한 문제를 해결하기 위해 고안된 개념이다.  
병렬 실행 프로세스들이 통신 채널을 통해서 서로 통신하도록 설계되었다. 이러한 접근 방식은 전역 메모리를 공유하는 대신, 프로세스 간의 명시적 통신을 중심으로 복잡한 동기화 문제를 회피하는데 초점을 맞추고 있다.

> Do not communicate by sharing memory; instead, share memory by communicating.

동시성 프로그래밍을 이야기하면, 대부분 병렬로 실행되는 멀티 스레드를 생각하게 된다. 이 때, 다양한 스레드 간에 데이터 구조, 변수, 메모리를 공유하는 부분을 무조건 생각해야 한다.  
그 이유는 race condition, memory management, dead lock, random-weird-unexplained-exceptions 등을 해결하기 위함이다.

### Golang

Go에서는 어떻게 할까? 라는 질문에는 다음과 같이 답할 수 있다.  
메모리를 공유하기 위해 변수를 잠그는 방식이 아닌, 변수에서 저장된 값을 스레드에서 스레드로 통신 혹은 전송한다. 이 과정에서 전송하거나 전송 받는 스레드 둘 다 데이터가 도착할 때 까지 대기하게 된다.  
이러한 방식을 통해서 데이터가 교환될 때 스레드 간의 동기화를 강제하게 된다. 즉, 기본적으로 데이터 전송이 완료되기 전에 어느 스레드도 작업을 수행하지 않으므로 race condition과 같은 문제가 발생할 여지가 적습니다.

프레임워크나 라이브러리를 통하지 않고 Go의 기본 기능으로 buffered channel이라는 기능을 제공한다. 따라서 channel을 통해서 스레드가 데이터를 주고 받을 때는 스레드가 lock되거나 동기화되는 것이 아닌 채널에 값이 채워지면 lock, 동기화되도록 한다.  
하지만 Go에서도 [Sync package](https://pkg.go.dev/sync)를 통해서 메모리 공유 모델도 사용할 수 있다. `sync.Mutex`의 `Lock()`, `Unlock()` method를 이용하여 고루틴 사이에 메모리를 공유할 수 있다.

- [Sync 예제](https://go.dev/tour/concurrency/9)

```go
package main

import (
	"fmt"
	"sync"
	"time"
)

// SafeCounter is safe to use concurrently.
type SafeCounter struct {
	mu sync.Mutex
	v  map[string]int
}

// Inc increments the counter for the given key.
func (c *SafeCounter) Inc(key string) {
	c.mu.Lock()
	// Lock so only one goroutine at a time can access the map c.v.
	c.v[key]++
	c.mu.Unlock()
}

// Value returns the current value of the counter for the given key.
func (c *SafeCounter) Value(key string) int {
	c.mu.Lock()
	// Lock so only one goroutine at a time can access the map c.v.
	defer c.mu.Unlock()
	return c.v[key]
}

func main() {
	c := SafeCounter{v: make(map[string]int)}
	for i := 0; i < 1000; i++ {
		go c.Inc("somekey")
	}

	time.Sleep(time.Second)
	fmt.Println(c.Value("somekey"))
}
```

- [Channel 예제](https://go.dev/tour/concurrency/2)

```go
package main

import "fmt"

func sum(s []int, c chan int) {
	sum := 0
	for _, v := range s {
		sum += v
	}
	c <- sum // send sum to c
}

func main() {
	s := []int{7, 2, 8, -9, 4, 0}

	c := make(chan int)
	go sum(s[:len(s)/2], c)
	go sum(s[len(s)/2:], c)
	x, y := <-c, <-c // receive from c

	fmt.Println(x, y, x+y)
}
```

- [Buffered Channel 예제](https://go.dev/tour/concurrency/3)

```go
package main

import "fmt"

func main() {
	ch := make(chan int, 2)
	ch <- 1
	ch <- 2
	fmt.Println(<-ch)
	fmt.Println(<-ch)
}
```

### ref

- https://www.minaandrawos.com/2015/12/06/concurrency-in-golang/
- https://go.dev/blog/codelab-share