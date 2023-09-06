---
title: "쓰레드(thread)에 대해 알아보자"
date: 2020-04-30
tags:
  - thread
  - parallel programming
categories:
  - linux
published: true
---

# Thread

## Parallel Programming

### 병렬 프로그래밍

- 공동의 목적을 달성하기 위해, 다수의 실행 주체가 동시에 작업을 수행하는 방식
    - 실행 주체: Process or Thread
    - 사실 컴퓨터 시스템에서 프로그램을 실행하는 주체는 CPU (or Core)!!
    - 프로세스나 쓰레드는 여러 프로그램이 CPU를 공유해서 사용하기 위한 추상적 단위
- 병렬 프로그래밍의 필요성
    - 최근 컴퓨터 시스템은 대부분 다수의 CPU 를 탑재하고 있으며,
    - 여러 CPU를 최대한 활용하여 성능을 높이기 위해서는 병렬 프로그래밍이 필수

### 소켓 프로그래밍에서는?

- 일반적으로 서버는 여러 클라이언트에 대해 동시에 서비스를 제공함
    - 예) 포털에 여러 사람이 동시에 접근하여 각자 서로 다른 서비스를 이용함
- 반복 서버
    - 하나의 프로세스가 동작하며 모든 클라이언트의 요청을 순서대로 처리
    - 클라이언트는 연결 수립을 위해 이전 클라이언트의 요청이 모두 처리될 때까지 대기하여야 함
- 동시 동작 서버 (Parallel or Concurrently running or Multi-user Server)
    - 여러 요청을 동시에 처리해 서비스할 수 있는 서버
    - 서버를 병렬로 동작하도록 구성
    - 일반적인 서버 역할 분담
        - Door-keeper process: 문지기 역할. 접속을 대기하다, 요청이 오면 연결을 수립함. 그리고 수립된 각각의 연결에 대해 서비스 프로세스를 할당함. 이를 반복함.
        - Service thread: 각 연결에 대해 1:1로 실제 서비스를 제공하는 쓰레드

### 반복 서버의 예

- 지난 소켓에서의 서버-클라이언트 예제 동작 방식
    - Server: listen()→ accept() → send() → recv() → close()
    - Client: connect() → recv() → send() → close()
- 서버 프로세스 수정: 반복 처리 서버
    - 하나의 클라이언트 처리가 종료된 후, 다시 accept()로 돌아와서 대기
    - 위 동작을 3번 반복하여, 3개의 클라이언트 요청을 처리하게 함
- 만약 클라이언트가 send()를 하지 않는다면?
    - 서버 프로세스는 recv() 에서 무한 대기하며, 해당 클라이언트로부터 패킷이 전송되길 기다릴 것
- 그때 만약 다른 클라이언트가 동시에 접속을 요청한다면?
    - 서버가 accept()를 대기하고 있기 않기 때문에, 접속 처리를 할 수 없음

### Process-based Parallel Socket Programming

- 프로세스 기반 동시 동작 서버
    - Door-keeper process
        - 기존 서버 코드와 같이 서버 소켓을 열고 bind(), listen() 수행
        - 새로운 연결이 수립되면, child process를 만들어, 아래 service 동작을 수행하게 함
        - 서버는 fork() 이후, 즉각 다시 accept()에서 대기. 이를 3회 반복함
        - 종료 전, 생성된 child 개수만큼 wait()를 수행하여 모든 종료 상태값을 출력 후, 종료
- Service process
    - 기존과 같이 send(), recv() 수행하고, 소켓을 닫고, 종료

## Thread

- 두 가지 종류의 실행 주체(execution unit or entity)
    - Process: 독립적인 메모리 공간을 가진 수행 주체
        - 각각 독립적인 메모리 공간을 생성해주고, 관리해주어야 하기 때문에
        - 생성/제거에 시간이 오래 걸리고, 시스템 자원을 많이 사용함
    - Thread: 프로세스의 단점을 보완하기 위해 새로이 정의된 실행 주체
- Thread: **The execution unit in a process**
    - Thread = CPU Register set + Independent Stack
        - 하나의 수행 흐름이 다른 수행 흐름과 분리되기 위해 필수적인 요소들만 모아 Thread로 정의
    - 하나의 프로세스 내에 여러 쓰레드가 함께 동작할 수 있음
        - 쓰레드들은 프로세스의 자원을 공유해서 사용: code, data, heap 영역, File descriptor 등
    - 장점
        - 프로세스보다 가볍다 (a.k.a. LWP: light-weight process)
            - 여러 쓰레드가 동시에 동작할 때, 여러 프로세스의 경우보다 효율적임 (성능 높음)
        - IPC 가 불필요함: 하나의 프로세스 내에서 메모리 공간을 공유하기 때문
    - 단점: 데이터 공유에 따른 동기화 문제, 오류 전파 (error propagation) 등

- Single-threaded Process vs. Multi-threaded Process
    - STP: 전통적인 프로세스의 구조와 동일함
        - 내부에서 명시적으로 쓰레드 관련 서비스를 사용하지 않더라도,
        - 코드가 수행되는 하나의 흐름을 쓰레드라는 개념으로 구체화하였음
        - 프로세스 생성이 완료되면, 기본 쓰레드가 코드를 순차적으로 수행하기 시작함
    - MTP: 쓰레드 개념이 등장하면서 가능해진 구조
        - 여러 쓰레드가 하나의 프로세스 내에서 프로세스의 자원을 공유하며 동시에 수행됨
    - **“프로세스” 는 더 이상 실행 주체 라기보다, 실행 주체인 쓰레드를 담는 그릇과 같은 역할**

### Pthread 서비스

- Pthread (POSIX Thread)
    - POSIX 는 운영체제 API 표준으로, 여러 OS에서 지원하고 있음
    - pthread 라이브러리: POSIX 에서 쓰레드를 관리하기 위한 여러가지 API를 구현한 것
        - 쓰레드 관련 시스템 콜들을 대신 호출하고, 관련 자료 구조를 관리함
    - 가장 일반적으로 널리 이용되는 쓰레드 라이브러리
    - 생성, 종료, 완료 대기, 동기화 관리 (lock) 등의 기능을 제공
    - gcc 컴파일 시, `-l pthread` 옵션을 사용해 명시적으로 라이브러리 지정을 해주어야 함

|          | 프로세스     | 쓰레드                |
|----------|----------|--------------------|
| 생성       | `fork()` | `pthread_create()` |
| 완료 대기    | `wait()` | `pthread_join()`   |
| 수행 코드 변경 | `exec()` | -                  |

### 쓰레드 생성: pthread_create()

- thread: 생성된 쓰레드를 가리키는 구조체. 쓰레드를 가리키기 위한 ID로 사용
    - TID: Thread ID. 숫자가 아니라 구조체로 표현됨
    - pthread_self()를 통해 확인 가능. pthread_equal()을 통해 비교 가능
- attr: struct pthread_attr_t 로 정의된 쓰레드의 속성들. NULL 인 경우, 기본 속성
- start_routine: 쓰레드로 수행할 코드의 함수 포인터
    - void * start_thread (void *arg);
- arg: start_routine 에 전달할 인자
- Return value
    - On success: 0 , On error: error number (not zero)

### 쓰레드의 종료

- 쓰레드를 종료하는 네 가지 방법
    - 쓰레드 생성 시 지정된 start_routine 에서 return 하는 경우
    - pthread_exit() : 자기 자신을 종료
        - 상태 종료값을 retval 로 전달. retval 은 동적으로 할당하여 사용
    - pthread_cancel() : 인자로 지정된 쓰레드를 종료
    - 전체 프로세스가 종료되는 경우
        - main() 함수의 return
        - (어느 thread 에서든) exit() 호출
        - exec() 시리즈로 새로운 코드가 로드된 경우

### 쓰레드 종료 대기: pthread_join()

- thread 의 종료를 대기하고, retval 에 종료 상태값을 전달함
    - 프로세스에서의 wait() 와 유사한 역할
- Return value
    - On success: 0 , On error: error number (not zero)
- pthread_detach
    - 만약 쓰레드를 join 하지 않기로 결정했다면,
    - detach 를 수행하여 쓰레드 종료 시, 즉각 쓰레드를 destroy 하도록 함
    - 예) 프로세스의 경우, wait()를 수행하지 않으면 좀비 프로세스가 됨
        - 쓰레드는 detach 를 시킨 경우, 다른 쓰레드가 join 을 수행하지 않아도 무방함