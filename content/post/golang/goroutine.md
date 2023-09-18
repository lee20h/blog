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

### 멀티스레드의 종류

- `User-Level`: 유저 스레드의 가장 큰 특징으로는 커널이 스레드의 존재를 알지 못한다는 부분이다. 프로세스 내의 사용자 라이브러리가 스레드를 관리하기 때문에 커널이 관여할 수 없다.
  - 한 프로세스 내에서 스레드가 관리되며, 프로세스마다 Thread Table과 Runtime System을 이용하여 커널이 프로세스 스케줄링 하듯이 관리한다.
  - 다시 말하면 `PCB(Process Control Block)`는 커널에서, `TCB(Thread Control Block)`는 프로세스에서 관리한다는 말과 같다.
  - 프로세스 내부에서 system call이 없이 Context Switching이 일어나므로 오버헤드가 매우 적다. 즉, 속도가 빠르다.
  - 하나의 스레드가 system call로 인해서 block되면 다른 스레드들도 멈춘다. 커널이 프로세스 내부의 스레드의 존재를 모르기 때문에 일어나는 현상이다.
  - OS 스레드는 1개만 사용하므로 멀티 코어를 활용할 수 없다.
  - 따라서, 1개의 OS 스레드 위에서 여러 유저 스레드를 관리한다는 점에서 `many-to-one` 모델이라고 불린다. (1:N)

<figure>
  <img src="https://github.com/lee20h/blog/assets/59367782/e2d2aac0-ecb7-4d5f-b550-411babe54de2" width="700" height="300"/>
  <figcaption>
    User-Level Thread
  </figcaption>
</figure>

- `Kernel-Level`: 커널이 모든 프로세스와 스레드들을 생성, 스케줄링 관리한다.
  - 커널 내에서 전체 PCB, TCB를 관리한다. 
  - 1개의 OS 스레드에 1개의 유저 스레드를 할당하는 `one-to-one` 방식이다 (1:1)
  - 프로세스 내의 한 스레드가 block되더라도 다른 스레드를 중단시키지 않고 계속 실행시킨다.
  - Context Switching 오버헤드가 높아서 속도가 느리다.
  - 만약, CPU 코어가 2개 이상이라면 각각 프로세서에서 한 프로세스의 두 개 이상의 스레드를 수행할 수 있다.
  - 리소스에 따라 생성 가능한 커널, 유저 스레드가 한정되어 있다.
  - 리눅스와 유닉스가 이런 방식을 취한다.

<figure>
  <img src="https://github.com/lee20h/blog/assets/59367782/6c302942-0fa7-40a3-9b57-841c7073cf46" width="600" height="300"/>
  <figcaption>
    Kernel-Level Thread
  </figcaption>
</figure>

- `Combined`: 두 방식의 장점을 혼합해서 관리하는 방법이다.
  - 여러 개의 OS 스레드에 여러 유저 스레드를 관리할 수 있는 `many-to-many` 모델이다. (N:M) 프로세스 1개(유저 스레드 N개)와 M개의 커널 스레드를 할당할 수 있다.
  - 커널 스레드마다 1개 씩 경량 프로세스(LWP)를 가진다. LWP란, 가상 처리기로 커널과 프로세스 간의 중간자 역할을 한다.
  - 따라서, Runtime System이 유저 스레드와 
  - 커널이 LWP를 스케줄링 및 실행, LWP가 대기 중인 스레드를 골라서 실행한다. 또한, 개발자가 직접 스레드와 LWP를 사용할 수 있다.
  - 유저 스레드처럼 작동하면서 병행성과 system call에 의한 block에도 대처할 수 있으며 Context Switching이 많이 일어나지 않는다. 이 때의 Context Switching의 오버헤드는 커널 스레드와 유사하다.
  - 멀티 CPU 코어에서 효율적으로 사용하려면, 각 스레드의 LWP 할당과 LWP의 CPU 할당이 별개로 이뤄지므로 각 스레드가 다른 CPU에 할당되도록 하는 매커니즘이 필요하다.

<figure>
  <img src="https://github.com/lee20h/blog/assets/59367782/a13a7b80-dc46-49a5-98c8-568387148a42" width="700" height="400"/>
  <figcaption>
    Combined
  </figcaption>
</figure>

## Goroutine

고루틴은 Go 런타임에 의해 관리되는 경량 스레드다. 가볍게 특징들을 훑자면 다음과 같다.
- 런타임에 의해 관리되므로 개발자는 OS 레벨의 스레드와 다르게 직접 반납하거나 종료시키는 과정이 없다.
- Combined(M:N)모델을 활용하고 있다. Context Switching, 멀티 코어를 언어의 특징으로 얻어가며, 구현이 어렵다는 단점도 언어의 특징으로 해소하였다.
- 경량 스레드이므로 고루틴의 생성 및 작업 전환에 사용되는 리소스 및 시간이 적어서 다량을 생성해도 부담이 적다.
- 채널을 통해 데이터 전달을 고루틴 간의 데이터 수발신으로 수행하여 공유 변수로 데이터를 전달할 때 발생할 수 있는 경합 상태(race condition), 교착 상태(deadlock), 데이터 오염이나 불일치 등의 발생을 차단하고 데이터의 흐름을 코드 상에서 직관적으로 볼 수 있도록 한다.
  
### GMP

고루틴은 Go 런타임에 의해서 관리된다고 앞서 말했었다. Go 런타임에서 어떻게 다중화된 스레드들을 할당하고 관리하며, 동시성을 이룰 수 있었는지 스케줄러를 통해서 알아보자.  
- G (Goroutine) : 고루틴을 의미
  - 런타임이 고루틴을 관리하기 위해서 사용합니다.
  - 컨텍스트 스위칭을 위해 스택 포인터, 고루틴의 상태 등을 가지고 있습니다.
  - G는 LRQ에서 대기하고 있습니다.
- M (Machine) : OS 스레드를 의미
  - M은 P의 LRQ로부터 G를 할당받아 실행합니다.
  - 고루틴과 OS 스레드를 연결하므로 스레드 핸들 정보, 실행중인 고루틴, P의 포인터를 가지고 있습니다.
- P (Processor) : 논리 프로세서를 의미
  - 최대 GOMAXPROCS개를 가질 수 있습니다.
  - P는 컨텍스트 정보를 담고 있으며, LRQ를 가지고 있어서 G를 M에 할당합니다.
- LRQ(Local run queue) : P마다 존재하는 Run Queue
  - P는 LRQ로 부터 고루틴을 하나씩 POP하여 실행합니다.
  - P마다 하나의 LRQ가 존재하기 때문에 레이스 컨디션을 줄일 수 있습니다.
  - LRQ가 M에 존재하지 않는 이유는 M이 증가하면 LRQ의 수도 증가하여 오버헤드가 커지기 때문입니다.
- GRQ(Global run queue) : LRQ에 할당되지 못한 고루틴을 관리하는 Run Queue
  - 실행 상태의 고루틴은 한번에 10ms까지 실행되는데, 10ms 동안 실행된 고루틴은 대기 상태가되어 GRQ로 이동됩니다.
  - 고루틴이 생성되는 시점에 모든 LRQ가 가득찬 경우 GRQ에 고루틴이 저장됩니다.

Go 언어는 내부 로직들도 전부 Go 언어로 구현되어 있어서 찾아볼 수 있다. 여기서의 GMP 스케줄러는 [runtime/runtime2](https://github.com/golang/go/blob/master/src/runtime/runtime2.go)에서 구현체를 확인할 수 있다.

!! 그림을 넣어야함 !!



## ref

- https://www.crocus.co.kr/1404
- https://happy-chipmunk.tistory.com/20
- https://velog.io/@khsb2012/go-goroutine
- https://d2.naver.com/helloworld/0814313
