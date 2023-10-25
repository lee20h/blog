---
title: "Go Semaphore 패턴에 대해 알아보자"
date: 2023-10-26T01:12:25+09:00
tags:
  - golang
  - concurrency
categories:
  - golang
series:
  - Concurrency
published: true
---

# Go Semaphore Pattern이란?

Go 언어에서의 **세마포어**(Semaphore)는 동시성 제어를 위한 고전적인 방법 중 하나로, 특정 자원에 대한 동시 접근을 제한함으로써 여러 고루틴 사이에서의 동기화를 달성한다.  
세마포어는 주로 두 가지 유형이 있으며 다음과 같다.
- 이진 세마포어(Binary Semaphore): 가장 간단한 형태의 세마포어로, 동시에 자원을 사용할 수 있는 스레드의 수를 1개로 제한한다. 이는 상호 배제(Mutex)와 매우 유사한 개념이다.
- 카운팅 세마포어(Counting Semaphore): 이 유형의 세마포어는 동시에 여러 개의 자원을 사용할 수 있지만, 그 수는 제한되어 있다. 예를 들어, 동시에 수행될 수 있는 고루틴의 수를 제한하여 리소스 과부하를 방지할 수 있다.

Go에서 세마포어는 채널과 고루틴을 사용하여 구현할 수 있다. 기본적으로, 세마포어는 정해진 수의 토큰을 가지고 있으며, 각 고루틴은 작업을 수행하기 전에 토큰을 획득(또는 대기)해야 하고, 작업 완료 후 토큰을 반환해야 한다.

## 특징

- 버퍼링된 채널을 이용한 구현: Go는 내장된 세마포어 타입을 제공하지 않지만, 버퍼링된 채널을 사용하여 세마포어를 쉽게 에뮬레이션할 수 있다. 버퍼링된 채널의 크기는 동시에 실행될 수 있는 고루틴의 수를 제한하는 데 사용된다.
- **Acquire**와 **Release**: 세마포어를 사용할 때 일반적으로 두 가지 주요 작업, 즉 Acquire(획득)와 Release(해제)가 있다. Acquire는 세마포어를 획득하여 리소스에 접근하려는 시도를 나타내며, Release는 리소스 사용이 완료된 후 세마포어를 해제한다.
- I/O 제한: Go에서 세마포어는 주로 I/O 작업, 특히 네트워크 호출이나 디스크 작업과 같은 리소스에 대한 동시 접근을 제한하는 데 사용된다. 예를 들어, 동시에 수행될 수 있는 HTTP 요청의 수를 제한하기 위해 세마포어를 사용할 수 있니다.
- 동시성 제어: 세마포어는 고루틴이 공유 리소스에 동시에 접근하는 것을 방지하여 경쟁 조건(race condition)이나 데드락(deadlock)과 같은 동시성 문제를 방지하는 데 도움을 준다.

요약하면, Go의 세마포어는 동시성 제어와 공유 리소스의 안전한 접근을 위한 도구로, 버퍼링된 채널을 사용하여 구현된다. 동시에 실행될 수 있는 작업의 수를 제한하여 시스템의 안정성과 효율성을 높이기 위해 사용된다.

## 장단점

- 장점 
  - 리소스 관리: 세마포어는 한 번에 실행할 수 있는 고루틴의 수를 제한하여 리소스 과부하를 방지하고 시스템의 안정성을 유지하는 데 도움이 된다. 특히 I/O 바운드 작업과 같이 리소스 사용이 높은 작업에서 중요하다. 
  - 동기화 간소화: 세마포어를 사용하면 개발자가 동기화 메커니즘을 직접 관리할 필요 없이 고루틴 간의 동기화를 달성할 수 있다. 코드의 복잡성을 줄이고 버그 발생 가능성을 감소시킨다. 
  - 유연성: 세마포어는 다양한 동시성 제어 시나리오에 적용할 수 있으며, 고루틴의 수를 동적으로 조절하여 시스템의 부하를 조절할 수 있는 유연성을 제공한다.
- 단점:
  - 데드락 위험: 세마포어를 잘못 사용하면 데드락이 발생할 수 있다. 예를 들어, 모든 고루틴이 세마포어에서 자원을 기다리는 동안 자원이 해제되지 않으면 시스템이 완전히 멈출 수 있다. 
  - 성능 저하: 세마포어가 너무 엄격하게 설정되면 시스템의 동시성 수준이 제한되어 성능이 저하될 수 있다. 이는 특히 고루틴이 많이 생성되지만 각각이 많은 작업을 수행하지 않는 경우에 문제가 될 수 있다. 
  - 디버깅 어려움: 세마포어와 같은 동시성 메커니즘은 디버깅이 어려울 수 있다. 문제가 발생하면, 특히 병렬 실행 환경에서, 문제를 추적하고 해결하는 것이 일반적인 순차 코드보다 복잡할 수 있다.

## 구성

![image](https://github.com/lee20h/blog/assets/59367782/0c5892fa-bec4-40ec-8734-783a90f9d271)

- 세마포어 생성: 세마포어는 버퍼 크기가 정해진 채널로 표현된다. 버퍼 크기는 동시에 실행할 수 있는 작업의 최대 수를 나타낸다.
- 토큰 획득: 고루틴은 작업을 시작하기 전에 세마포어에서 토큰을 획득해야 한다. 토큰을 획득하려면 채널에 값을 보낸다.
- 작업 실행: 토큰을 성공적으로 획득한 후, 고루틴은 필요한 작업을 실행할 수 있다. 이 작업은 네트워크 요청, 파일 I/O 또는 CPU 집약적 연산과 같은 것일 수 있다.
- 토큰 반환: 작업이 완료되면, 고루틴은 세마포어에 토큰을 반환하여 다른 고루틴이 토큰을 획득할 수 있도록 해야 한다.

## 예제

```go
package main

import (
	"fmt"
	"sync"
	"time"
)

// Semaphore 인터페이스는 세마포어가 가져야 할 기본 메서드를 정의합니다.
type Semaphore interface {
	Acquire() // 세마포어 획득
	Release() // 세마포어 반환
}

// ChannelBasedSemaphore는 채널을 기반으로 한 Semaphore의 구현체입니다.
type ChannelBasedSemaphore struct {
	sem chan struct{}
}

// NewSemaphore 함수는 새로운 ChannelBasedSemaphore를 생성합니다.
func NewSemaphore(concurrencyLimit int) *ChannelBasedSemaphore {
	return &ChannelBasedSemaphore{
		sem: make(chan struct{}, concurrencyLimit),
	}
}

// Acquire 메서드는 세마포어를 획득합니다.
func (s *ChannelBasedSemaphore) Acquire() {
	s.sem <- struct{}{}
}

// Release 메서드는 세마포어를 반환합니다.
func (s *ChannelBasedSemaphore) Release() {
	<-s.sem
}

func main() {
	concurrencyLimit := 3
	semaphore := NewSemaphore(concurrencyLimit)

	var wg sync.WaitGroup

	for i := 1; i <= 10; i++ {
		wg.Add(1)
		go func(i int) {
			defer wg.Done()

			semaphore.Acquire()
			defer semaphore.Release()

			fmt.Printf("작업 %d 시작\n", i)
			time.Sleep(2 * time.Second) // 작업 시뮬레이션
			fmt.Printf("작업 %d 완료\n", i)
		}(i)
	}

	wg.Wait()
	fmt.Println("모든 작업 완료")
}
```

## ref

- [go-concurrency-pattern-semaphore](https://medium.com/gitconnected/go-concurrency-pattern-semaphore-9587d45f058d)