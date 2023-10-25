---
title: "Go Pipeline 패턴에 대해 알아보자"
date: 2023-10-29T00:39:31+09:00
tags:
  - golang
  - concurrency
categories:
  - golang
series:
  - Concurrency
published: true
---

# Go Pipeline Pattern이란?

Go 언어에서의 **파이프라인 패턴**은 동시성 프로그래밍에 있어 중요한 개념으로, 특히 복잡한 데이터 처리와 관련된 작업을 효율적으로 수행하기 위해 사용된다.  
이 패턴은 여러 단계의 독립적인 작업 단위로 나누어진 작업의 흐름을 생성한다. 각 단계는 파이프라인의 다음 단계로 데이터를 **전달**하며, 이러한 방식으로 데이터는 파이프라인을 통해 흐르게 된다.

## 구성

![image](https://github.com/lee20h/blog/assets/59367782/579d1d5a-3caa-4663-8e40-687fb68456ed)

- **단계 생성**: 각 파이프라인 단계는 보통 별도의 고루틴에서 실행되는 함수로 구성된다. 이 함수는 입력 채널로부터 데이터를 읽고, 어떤 처리를 수행한 다음, 출력 채널로 결과를 전송한다.
- **채널 연결**: 파이프라인의 연속적인 단계들은 채널을 통해 연결된다. 한 단계의 출력 채널은 다음 단계의 입력 채널이 된다.
- **동기화와 종료**: 파이프라인의 마지막 단계가 완료되면, 종종 모든 고루틴이 완료될 때까지 기다려야 한다. 이는 `sync.WaitGroup`을 사용하여 동기화할 수 있으며, 종료 신호를 전달하여 고루틴이 종료되도록 할 수 있다.

## 사례

파이프라인은 데이터 처리, 병렬 계산, 비동기 작업 흐름 등 다양한 컨텍스트에서 유용하다. 예를 들어, 네트워크 요청을 처리하거나, 큰 데이터 세트를 분석하거나, CPU 집약적인 작업을 병렬로 수행해야 하는 경우에 파이프라인을 사용할 수 있다.  
파이프라인 패턴을 사용하면 복잡한 작업 흐름을 여러 단계로 나눌 수 있어 코드의 가독성과 유지 관리성이 향상되며, 시스템 자원을 효율적으로 활용할 수 있다.

## 예제 

### 함수 작성 예제

```go
package main

import (
	"fmt"
	"sync"
)

// 첫 번째 단계: 정수 생성
func gen(nums ...int) <-chan int {
	out := make(chan int)
	go func() {
		for _, n := range nums {
			out <- n
		}
		close(out)
	}()
	return out
}

// 두 번째 단계: 정수 제곱
func sq(in <-chan int) <-chan int {
	out := make(chan int)
	go func() {
		for n := range in {
			out <- n * n
		}
		close(out)
	}()
	return out
}

// 세 번째 단계: 결과 출력
func print(ch <-chan int) {
	for n := range ch {
		fmt.Println(n)
	}
}

func main() {
	// 파이프라인 설정
	nums := gen(2, 3, 4) // 정수 생성
	squares := sq(nums)  // 정수 제곱

	// 파이프라인 실행 및 동기화
	var wg sync.WaitGroup
	wg.Add(1)
	go func() {
		print(squares) // 결과 출력
		wg.Done()
	}()
	wg.Wait()
}
```

### 파이프 인터페이스 작성 예제

```go
package main

import (
	"fmt"
	"log"
)

func main() {
	// 새 파이프라인 생성
	p := New(func(out chan interface{}) {
		// 초기 데이터 제공
		for i := 1; i <= 5; i++ {
			out <- i
		}
		close(out) // 중요: 채널을 닫아야 함
	})

	// 파이프라인에 단계 추가
	p = p.Pipe(func(in interface{}) (interface{}, error) {
		// 정수를 제곱합니다.
		if num, ok := in.(int); ok {
			return num * num, nil
		}
		return nil, fmt.Errorf("invalid input")
	}).Pipe(func(in interface{}) (interface{}, error) {
		// 제곱된 정수에 10을 더합니다.
		if num, ok := in.(int); ok {
			return num + 10, nil
		}
		return nil, fmt.Errorf("invalid input")
	})

	// 파이프라인 실행 및 결과 수집
	out := p.Merge()

	// 결과 출력
	for o := range out {
		fmt.Println(o)
	}
}

type Executor func(interface{}) (interface{}, error)

type Pipeline interface {
	Pipe(executor Executor) Pipeline
	Merge() <-chan interface{}
}

type pipeline struct {
	dataC     chan interface{}
	errC      chan error
	executors []Executor
}

func New(f func(chan interface{})) Pipeline {
	inC := make(chan interface{})
	go f(inC)
	return &pipeline{
		dataC:     inC,
		errC:      make(chan error),
		executors: []Executor{},
	}
}

func (p *pipeline) Pipe(executor Executor) Pipeline {
	p.executors = append(p.executors, executor)
	return p
}

func (p *pipeline) Merge() <-chan interface{} {
	for i := 0; i < len(p.executors); i++ {
		p.dataC, p.errC = run(p.dataC, p.executors[i])
	}
	return p.dataC
}

func run(
	inC <-chan interface{},
	f Executor,
) (chan interface{}, chan error) {
	outC := make(chan interface{})
	errC := make(chan error)

	go func() {
		defer close(outC)
		for v := range inC {
			res, err := f(v)
			if err != nil {
				errC <- err
				continue
			}
			outC <- res
		}
	}()
	return outC, errC
}

```

## ref

- [go-concurrency-pattern-pipeline](https://syafdia.medium.com/go-concurrency-pattern-pipeline-635dfae01af1)