---
title: "Go Worker Pool 패턴에 대해 알아보자"
date: 2023-10-28T22:37:13+09:00
tags:
  - golang
  - concurrency
categories:
  - golang
series:
  - Concurrency
published: true
---

# Go Worker Pool이란?

**Worker Pool**은 고정된 수의 작업자를 사용하여 큐에 있는 여러 작업을 실행하여 동시성을 구현하는 패턴이다. Go 생태계에서는 고루틴을 사용하여 작업자를 생성하고 채널을 사용하여 큐를 구현한다.  
정의된 작업자 수는 큐에서 작업을 가져와 작업을 완료하며, 작업이 완료되면 새 작업을 계속 가져와서 진행한다.

![image](https://github.com/lee20h/blog/assets/59367782/112c1f43-9824-4b3a-8c8f-cf61b2506d24)

데이터 교환의 경우에도 채널을 사용해서 동기화 문제를 해결할 수 있다.

## 필요성

우리가 사용하는 물리 리소스는 무한하지 않기 때문에, 필요한 만큼만 사용해야한다. Go에서의 고루틴 객체의 최소 크기는 2KB로, 계속 생성하다보면 언젠가는 리소스가 바닥이 나므로 고루틴의 개수를 **제한**할 필요가 있다.  
만약 제한된 워커풀을 사용하게 되면, 고루틴의 개수가 제한되기 때문에 리소스의 급격한 증가를 줄일 수 있다. 

## 장단점

- 장점
  - 자원 재사용
  - 시스템 안정성 향상
  - 리소스 관리 용이성
  - 작업 처리 시간 감소
- 단점
  - 초기 구성 복잡성
  - 리소스 낭비 가능성
  - 동기화 오버헤드
  - 데드락 및 경쟁 상태
    - 채널을 이용한 해결 방안
    - lock을 이용한 해결 방안

## 예제

```go
package main

import (
	"fmt"
	"sync"
	"time"
)

// Job은 처리할 작업과 결과를 반환할 채널을 포함합니다.
type Job struct {
	task     func() interface{} // 작업자에 의해 처리될 작업
	resultCh chan<- Result      // 작업 결과를 반환할 채널
}

// Result는 작업의 결과를 포함합니다.
type Result struct {
	value interface{} // 작업의 결과
}

// WorkerPool은 워커의 수와 작업 채널을 정의합니다.
type WorkerPool struct {
	MaxWorkers int
	JobQueue   chan Job
}

// NewWorkerPool 함수는 새로운 워커 풀을 생성합니다.
func NewWorkerPool(maxWorkers int) *WorkerPool {
	pool := &WorkerPool{
		MaxWorkers: maxWorkers,
		JobQueue:   make(chan Job),
	}
	return pool
}

// Start 메서드는 워커 풀을 시작합니다.
func (wp *WorkerPool) Start() {
	for i := 0; i < wp.MaxWorkers; i++ {
		go func(workerID int) {
			for job := range wp.JobQueue {
				fmt.Printf("Worker %d: 작업을 시작합니다.\n", workerID)
				result := job.task() // 실제 작업을 수행하고 결과를 가져옴.
				job.resultCh <- Result{value: result} // 결과 반환
				fmt.Printf("Worker %d: 작업이 완료되었습니다.\n", workerID)
			}
		}(i)
	}
}

// AddJob 메서드는 워커 풀에 새로운 작업을 추가합니다.
func (wp *WorkerPool) AddJob(task func() interface{}, resultCh chan<- Result) {
	wp.JobQueue <- Job{task: task, resultCh: resultCh}
}

// Shutdown 메서드는 워커 풀을 종료합니다.
func (wp *WorkerPool) Shutdown() {
	close(wp.JobQueue)
}

func main() {
	pool := NewWorkerPool(5)
	pool.Start()

	var wg sync.WaitGroup
	resultCh := make(chan Result, 10) // 결과를 수신할 채널

	// 여러 작업을 워커 풀에 추가
	for i := 0; i < 10; i++ {
		wg.Add(1)
		job := func() interface{} {
			defer wg.Done()
			// 실제 작업을 수행 
			// 여기서는 시뮬레이션을 위해 Sleep을 사용
			time.Sleep(2 * time.Second)
			return fmt.Sprintf("작업 결과 %d", i+1)
		}
		pool.AddJob(job, resultCh)
	}

	go func() {
		wg.Wait()
		close(resultCh)
	}()

	for result := range resultCh {
		fmt.Println("수신된 결과:", result.value)
	}

	pool.Shutdown()
	fmt.Println("모든 작업이 완료되었습니다.")
}
```

## ref

- [go-concurrency-pattern-worker-pool](https://medium.com/code-chasm/go-concurrency-pattern-worker-pool-a437117025b1)