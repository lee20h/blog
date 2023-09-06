---
title: "application layer에 대해 알아보자"
date: 2020-09-02
tags:
  - application layer
categories:
  - network
published: true
---

# Application Layer

- **Client-server paradigm**
- Addressing
- Different servcies

## Client-Server Model: Socket Interface

Client와 Server가 Internet을 통해서 통신하는 방식을 의미한다. 대부분 여러 Client와 한 Server와 통신을 얘기한다. 가장 흔한 이러한 방식이 Web Server이다.

### Connectionless iterative server

Client가 연결 요청을 할 때 Server측에서 UDP 요청을 받아서 Client에게 결과를 반환하는 식인데 구현을 While을 통해서 구현되기 때문에 한번에 하나의 요청에만 답할 수 있으므로, 여러 요청시 순서대로 하나씩 응답할 수 있다. 이러한 방식은 구현이 간단하지만 Server의 성능이 비효율적이다. 그리고 UDP를 통한 요청이므로, 간단한 정보만 통신하기 좋다.

### Connection-oriented concurrent server

Client가 연결 요청 시 Server측과 TCP로 연결한 뒤 통신을 하게 되는데 Server가 자신을 Fork하여 Child Server를 통해서 한 Client당 하나의 Child Server를 둬서 통신을 유지한다. 이때 Server의 한 포트를 통해서 사용한다. Fork되는 여러 프로세스를 CPU 스케쥴링할 때 Context-switching에서 생기는 오버헤드 때문에 많은 양의 Server를 가질 수 없다. 이러한 Scalability Problem를 해결하기위해 프로세스를 Fork하지 않고 새로운 Thread를 만들어 Server 역할을 하게 되었다.

해당 방법을 Multi-threaded server라고 한다. 오늘날 기본은 Multi-threaded server가 기본이며, 두 방법을 혼용해서 사용하기도 한다.

### Cluster 방식의 Server implementation

해당 구조가 네이버나 구글같은 사용량이 높은 기업에서도 사용하는 구조이다.

![image](https://user-images.githubusercontent.com/59367782/97833305-6d978800-1d18-11eb-8beb-1fe948034829.png)

Client가 Web Accelerator(Load Balancer)가 HTTP 요청을 받은 뒤 Front-end Server들에 분산시켜서 요청을 보낸다. 이 때 Front-end Server가 다 독립적으로 사용하면 각각의 동기화 문제가 생기기 때문에 Back-end Server를 통해서 DB에 접근하는 구조로 되어있다. Client가 늘어나게 되면 Front-end Server를 늘리게 되면 부하를 이겨낼 수 있다.

## Socket Interface

소켓의 구조는 
||||
|---|---|---|
| Family | Type| Protocol |
|Local Socket address|||
|Remote Socket address|||

소켓 타입은 다음 그림과 같다. 화살표에 적힌 타입으로 코딩을 하게된다.

![image](https://user-images.githubusercontent.com/59367782/97835693-1ac0cf00-1d1e-11eb-9970-35c7bf8721a9.png)

### Socket interface for connectionless iterative server

![image](https://user-images.githubusercontent.com/59367782/97835823-5c517a00-1d1e-11eb-90d1-dff977602f34.png)

Server에서 요청을 받아야 결과를 응답할 수 있다.

### Socket interface for connection-oriented concurrent server

![image](https://user-images.githubusercontent.com/59367782/97835952-a3d80600-1d1e-11eb-8dd4-4d73d7d48fd7.png)

![image](https://user-images.githubusercontent.com/59367782/97835984-ae929b00-1d1e-11eb-815e-523d25a2d860.png)

Server에서 Fork하여 사용하는 이유

서버의 PCB에 레지스터 외에도 I/O Handler가 존재한다. I/O Handler에서 연결된 소켓을 핸들링하게 되는데, fork한 child와 server는 서로 다른 객체지만 child에서 I/O Handler를 똑같이 접근할 수 있으므로 연결된 소켓을 사용할 수 있게 된다.

- 운영체제에서 Process들은 서로 독립적
    - 서로의 메모리 접근 불가
    - 독립적인 프로세스가 자료를 공유하는 방법은 shared memory/message passing 방법이 있지만 서로 인지해야한다. 하지만 Server는 서로 다른 타입의 수많은 child가 언제 어떻게 데이터를 공유할지 알기 어려움
    - 따라서 코딩이 복잡하고 난해하므로 Fork-exec 조합이 server가 child에게 일을 넘기고 사용하기 좋은 방법

## 그림 출처

- McGraw-Hill 출판사 TCP/IP 프로토콜