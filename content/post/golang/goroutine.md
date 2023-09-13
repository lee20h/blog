---
title: "고루틴에 대해 알아보자"
date: 2023-09-13T22:48:47+09:00
tags:
  - golang
  - concurrency
  - goroutine
categories:
  - golang
series:
  - Concurrency
published: false
---

# Goroutine 이란?

고루틴(goroutine)에 대해 이야기하기 전에 먼저 스레드(thread)에 대해서 이야기하면 좋을 것 같다.

## Thread

스레드란 추상적으로 프로세스 내에서 실행되는 흐름의 단위 혹은 CPU 스케줄링의 기본 단위라고 할 수 있다.  
따라서, 멀티 스레드는 한 프로세스 내에서 실행된다. 이것을 통해 다음 특징들을 설명할 수 있다.

- 스레드는 Stack 메모리 영역을 가진다.
  - 최소한의 레지스터 상태를 가진다.
- 같은 프로세스의 Heap, Code, Data 메모리 영역을 공유한다.
- 프로세스보다 스레드를 스위칭하는 것은 프로세스보다 가볍다.

스레드에 관한 내용은 이 [포스팅](https://20h.dev/post/linux/thread/)을 참고하자.

### 스레드의 종류

- `User-Level`: User-Level 스레드의 가장 큰 특징으로는 커널이 스레드의 존재를 알지 못한다는 부분이다. 프로세스 내의 사용자 라이브러리가 스레드를 관리하기 때문에 커널이 관여할 수 없다.
  - 한 프로세스 내에서 스레드가 관리되며, 프로세스마다 Thread Table과 Runtime System을 이용하여 커널이 프로세스 스케줄링 하듯이 관리한다.   
  - 프로세스 내부에서 Context Switching이 일어나므로 오버헤드가 매우 적다. 즉, 속도가 빠르다.
  - 하나의 스레드가 syscall로 인해서 block되면 다른 스레드들도 멈춘다. 커널이 프로세스 내부의 스레드의 존재를 모르기 때문에 일어나는 현상이다.
  - OS 스레드는 1개만 사용하므로 멀티 코어를 활용할 수 없다.
  - 따라서, 1개의 OS 스레드 위에서 여러 User-Level 스레드를 관리한다는 점에서 `many-to-one` 모델이라고 불린다. (1:N)

<figure>
  <img src="https://github.com/lee20h/blog/assets/59367782/e47f7ef6-a3f2-40ad-b932-54bd5a2c21b3" width="500" height="300"/>
  <figcaption>
    User-Level Thread
  </figcaption>
</figure>

- `Kernel-Level`
- `Combined`

## Goroutine

고루틴은 Go 런타임에 의해 관리되는 경량 스레드다. 가볍게 특징들을 훑자면 다음과 같다.
- 런타임에 의해 관리되므로 개발자는 OS 레벨의 스레드와 다르게 직접 반납하거나 종료시키는 과정이 없다.
- 


## ref

- https://happy-chipmunk.tistory.com/20
- https://velog.io/@khsb2012/go-goroutine
