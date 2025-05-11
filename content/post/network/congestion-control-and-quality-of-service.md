---
title: "혼잡제어 (congestion control)에 대해 알아보자"
date: 2020-08-25
tags:
  - congestion control
  - quality of service
categories:
  - network
publishResources: true
---

# Congestion Control and Quality of Service

- 혼잡제어는 라우터의 버퍼가 넘치는 경우를 제어해서 없도록 하는 것이다.  
- QOS는 프로토콜 상에선 존재하지 않지만 라우터가 적용할려고 노력한다. 필요하기 때문이다.

## Data Traffic

- 네트워크 혼잡(Congestion)과 품질(QoS)은 사용자로부터 들어오는 데이터 트래픽 양을 조절하거나 서비스 양을 달리하는 것
- 양을 조절하려면 얼마나 어떤 식으로 들어오는 지 정량화 필요
- 가장 간단한 정량화 방법은 평균을 구하는 것이다.
    - `Average data rate = amount of data / time`
    - 문제점 : 평균은 단위 시간에 들어오는 양이 일정할 때 유용하지만 실제 인터넷 트래픽은 그렇지 않다.
    - 인터넷 트래픽은 bursty하다고 할 수 있다.

![image](https://user-images.githubusercontent.com/59367782/97305648-bb3b6d00-18a0-11eb-8138-5a18dd263ae9.png)

### 데이터 트래픽 기술 방법

- 평균의 경우
    - 기준이 되는 시간은 경우에 따라 알맞게 사용한다
        - 과금이 목적 : 시간은 1달
        - TCP 성능 측정이 목적 : 시간은 RTT 시간

- 데이터 트래픽을 기술하는 일반적인 방법
    - 시간 구간에 대해 최대량을 기술
    - Amount = f(t)
        - 함수 f는 non-decreasing function

![image](https://user-images.githubusercontent.com/59367782/97305848-f89ffa80-18a0-11eb-8401-403116e9203a.png)


- 트래픽을 라우터가 쉽게 측정하기 위해서는 간단한 non-decreasing 함수만 사용
    - 가장 널리 쓰이는 형태
        - `Amount = max(line A, line B)`

![image](https://user-images.githubusercontent.com/59367782/97305958-1f5e3100-18a1-11eb-941d-76c5978d7d34.png)

데이터 트래픽 기술 방법은 이해하면 좋다.

### CBR(Constant-bit-rate) traffic

- 90년대에는 네트워크의 망에 맞춰서 비디오 품질을 낮췄다.
- 예전에는 비디오 데이터를 주로 사용했다. 압축하게 되면 매 프레임마다 데이터량이 일정하지 않음
- 억지로 일정하게 하므로 퀄리티가 좋았다가 나빴다가 반복하게 된다.

![image](https://user-images.githubusercontent.com/59367782/97306250-9398d480-18a1-11eb-8f11-af6b5fe7cc65.png)

### VBR(Variable-bit-rate) traffic

- 필요시에 데이터량이 증가했다가 감소한다.
- 비디오 데이터를 VBR traffic으로 그대로 전송하는게 일반적이다.

![image](https://user-images.githubusercontent.com/59367782/97306516-ed999a00-18a1-11eb-8624-335591d12d9e.png)

### Bursty traffic

- 인터넷 트래픽은 이러한 양상을 띔

![image](https://user-images.githubusercontent.com/59367782/97306764-40735180-18a2-11eb-86fa-9ccb2c6dafad.png)

## Congestion

- 혼잡은 네트워크에 걸리는 로드가 네트워크 용량을 초과하면 발생할 수 있음
- 혼잡 제어는 네트워크 용량이 초과될 때 네트워크 처리 용량 한계 이하로 낮추는 매커니즘
    - 네트워크 용량이 초과되는 것을 인지 (타이머를 통해 패킷이 손실된 경우)
    - 진입을 막음
- 혼잡이 발생하는 이유는 고속도로에 많은 차가 진입하면 교통체증을 겪는 것과 동일한 이유
- 라우터 버퍼는 일시적인 혼잡을 해결하기 위한 것
    - 전체적인 입력 데이터 양이 네트워크 처리 용량을 초과하지 않지만 순간적으로 많은 양이 들어오는 것을 처리
    - 따라서 라우터 버퍼는 일시적인 bursty 데이터를 저장할 크기 정도가 적당함

- 패킷의 지연과 네트워크 로드

![image](https://user-images.githubusercontent.com/59367782/97308516-4a964f80-18a4-11eb-884e-fdd58592fb8e.png)

- 입력에 대한 출력 양상

![image](https://user-images.githubusercontent.com/59367782/97308695-80d3cf00-18a4-11eb-878d-ebf74a1cd154.png)

## Congestion Control in TCP

- end-end control로 네트워크의 도움 없이 일어남
- 혼잡제어에 쓰이는 슬라이딩 윈도우는 동적이며 receiver 윈도우보다 작다.
- **sender가 혼잡을 인지하는 방법**
    - packet loss event = TCP의 timeout or 3 duplicate acks (같은 ack가 3번 날라온 경우)
- 혼잡제어 방법
     - **AIMD** (Additive Increase Multiplicative Decrease)
     - **Slow Start**
     - Conservative after timeout events

### TCP AIMD

- Additive Increase : ACK가 잘 돌아오면 혼잡 윈도우 사이즈를 MSS(Maximum Segment Size)씩 증가시킨다.
- Multiplicative Decrease : `packet loss event`가 일어나게 되면 혼잡 윈도우 사이즈를 절반으로 줄인다.
    - 라우터 버퍼가 혼잡이 발생한 시점과 TCP Sender 측에서 혼잡이 인지한 시점의 차이가 있으므로 그 시차간 혼잡의 정도를 상쇄시키기 위해서 절반으로 줄인다.

![image](https://user-images.githubusercontent.com/59367782/97310094-318e9e00-18a6-11eb-8a56-209d537b1e32.png)

### TCP Slow Start

- 처음에는 TCP 연결이 시작하면 혼잡 윈도우 사이즈를 `1`부터 네트워크 최대량까지 눈에 띄게 증가한다. (2의 거듭제곱만큼)
- 이후 loss event가 일어나게 되면 절반을 줄인 뒤 그 다음은 위에서 설명한 양상과 같다.
- 이러한 부분을 설명하기 위해선 FTP로 여러 파일을 전송하는 경우 한 파일을 전송이 끝나고 다음 파일을 전송할 때 TCP 연결이 끊겼다 다시 연결되므로 느리지만, 여러 파일을 압축을 한 뒤 보내면 속도가 일정하게 빠른 것을 알 수 있다.

### 무선환경에서의 TCP 혼잡제어의 문제점

- 유선환경에서는 TCP가 거의 완벽하게 동작
- 무선(wireless) 환경(WiFi, 4G, 5G)에서는 문제가 생김
    - 무선환경은 패킷 전송 오류가 빈번하게 발생
    - TCP Sender는 단순한 패킷 전송 오류를 `혼잡`으로 착각
        - 쓸데없이 전송율을 절반으로 줄임
        - 예) 무선랜 12 Mbps에서 혼자 사용하지만 5 Mbps만 나오는 경우
- 대응 방법
    - 무선환경에 적응하는 Wireless TCP 프로토콜 개발
        - 호환성의 문제점 존재
    - 무선환경 자체에 패킷 전송 오류를 보완하는 방법
        - 패킷 오류가 일어나지 않게 만듬
        1. 프레임에 Forward Error Correction 넣기 : 비트가 깨지면 복원
        2. Transmission Schedule를 통해 기지국에서 오류가 발생한 것을 확인하여 재전송하여 오류를 일어나지 않게 조절함

![image](https://user-images.githubusercontent.com/59367782/97313186-afa07400-18a9-11eb-9f63-55cb456cf4ad.png)

## Quality of Service

### Flow characteristics

Flow란?

- 일반적으로 응용 프로그램 연결 (FTP 연결, Video 연결 등)을 뜻함
- 망 사업자 관점에서 사용자 (사용자 A의 컴퓨터에서 오는 트래픽)
- IPv4에는 표준 정의가 없으나, IPv6에는 표준 정의가 내려져있음

정의된 Flow에 대해서 어떤 QoS를 제공할 것인가를 생각하는 것이 Flow Characteristics이다.

Flow Characteristics

- Reliability : 패킷 소실이 발생하지 않을 확률
- Delay
- Jitter : 패킷간 도착 간격의 차이
- Bandwidth

B-ISDN의 QoS의 모델은 하부망에서 ATM(Asynchronous Transfer Mode)이 인터넷의 QoS 보장 모델이 적용되지 않는 이유다. ATM이 가격 문제와, 고장이 자주 날 수 있어서 국내에선 사용하지 않는다.

### 인터넷의 QoS 모델

- IETF에서 표준화한 모델로, 두 가지 모델을 정의했다.
    - QoS 보장 서비스(Guaranteed Service) model : Flow 별로 전송 오류가 0%되고 보장한 대역폭을 100% 보장하는 모델
    - Qos 차별화 서비스(Differentiated Service) model : Flow들을 몇 개의 class로 나누고 각 클래스에 대해서 서비스를 차별화하는 모델

각 모델의 방식을 알아보자.

- QoS 보장 서비스

![image](https://user-images.githubusercontent.com/59367782/97831031-3e7e1800-1d12-11eb-8b6c-7a6fb7749368.png)

1개의 queue가 아니라 Flow 개수만큼 queue를 유지하므로 각 Flow에 QoS를 보장할 수 있다. Core 라우터의 경우 flow 개수가 1백만개 이상이다. 그렇다면 queue 또한 1백만개 이상이 생긴다. 우선순위 큐를 사용한 방법이므로 복잡도가 `O(log N)`이 되게 된다. 1백만개 넘게 되면 속도에 문제가 생기므로 크기가 제한적일 수 있다. 이러한 문제를 `Scalability Problem`이라한다.

- QoS 차별화 서비스

![image](https://user-images.githubusercontent.com/59367782/97831219-b9473300-1d12-11eb-9200-082311a3f1b6.png)

1개의 queue가 아니라 Class 개수만큼 queue를 유지한다. 망 사업자가 골드, 실버, 브론즈 서비스를 제공한다. 이 경우에는 3개의 queue만 가지고 있으면 되므로 구현이 간단하고 Scalability Problem이 없다. 하지만 만약 한 클래스에 다 몰리게 되면 차등별로 나눈 클래스의 의미가 없어진다. 이러한 문제를 해결하기 위해 각각의 클래스는 하위 클래스보다는 더 나은 서비스를 제공한다고 보장을 하는 식으로 해결할려고 하고 있다. 또한 각 Flow에 보장하는 QoS는 무엇인가가 아직도 문제로 남아있다.

### Techniques to Improve QoS

현재 라우터는 큐를 하나 사용하여 FIFO 방식으로 순서대로 제공하는 식으로 빠르고 싸게 구현하였다. 하지만 큐 별로 QoS를 제공할 수 없다. 따라서 멀티 큐로 우선순위 큐, 가중치를 준 공정 큐를 통해서 QoS를 제공하는 방법을 고안하기도 했다.

### Integrated Services

- Best-effort service (오늘날의 인터넷 서비스)
- Guaranteed service
    - Signaling
    - Flow Specification
    - Admission
    - Service Classes
    - RSVP(Reservation Protocol) : signaling을 위해서 사용

### RSVP

일대일 통신이 아니라 다자간 통신으로 Source와 여러 Receiver가 존재한다. 먼저 Path를 설정한 뒤 자원을 예약한다. 따라서 먼저 Path messages로 경로를 설정 한 뒤 역방향으로 Resv messages를 통해서 예약을 할 수 있게한다.

- Path messages
- Resv messages

### Differentiated Service

통합서비스를 대체하기 위해 차별화 서비스를 사용할려고 한다. 차별화 서비스는 flow-based가 아닌 class-based로 QoS 모델을 디자인한다.


## 그림 출처

- McGraw-Hill 출판사 TCP/IP 프로토콜