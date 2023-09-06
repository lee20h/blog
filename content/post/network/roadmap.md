---
title: "네트워크에 대해 알아보자"
date: 2020-08-13
tags:
  - internet
categories:
  - network
published: true
---

## 인터넷이란 무엇인가?  

Computing devices : hosts, end-system(PC, Workstation, server). 네트워크 앱이 실행된다.  

Communication links
- fiber, copper, radio, satellite
- transmission rate = **bandwidth**

Routers: foward packets(chunks of data)  

![WTI](https://github.com/lee20h/blog/assets/59367782/b0ca516d-3522-483f-8c79-ebbadd1afcf5)

구조를 설명하는 그림으로, 라우터들과 서버, 워크스테이션과 모바일로 구성되어있다. 네트워크 시점으론 라우터들과 호스트들로 볼 수 있다. ISP는 KT와 같은 회사를 의미한다.  

IDC(Internet Data Center)는 그림의 regional ISP와 같이 서버를 라우터에 붙여서 설치하기도 하는데 병목현상을 줄이고 데이터들을 전송하기 수월하게 한다. 즉, 대용량 서비스에 유리하다.  

Network protocol : 인터넷의 입장에서 모든 통신활동이 프로토콜에 의해 제어된다.  

protocol이란 패킷의 포맷, 순서가 정의되어 있는 것으로 네트워크에서 레이어마다의 규칙이라고 생각하면 좋다. 사람과의 대화처럼 컴퓨터의 대화에서도 필요한 것이 프로토콜이다.   

## 네트워크 구조

Edge router : end-system에 접속하게 해주는 edge router의 역할은 사용자마다 허용된 트래픽만큼 제한(regulation) 해준다. 그리고 서로 다른 ISP에서 허용 트래픽 이상을 통신할려하면 요금이 더 부과될 수 있으므로 우회하는 방향으로 라우팅을 하기도 한다.  

Core router : Edge router에서 제한을 해줬으므로 Core router의 역할은 전송 속도를 빠르게 해주는 부분이다.  

Access networks : end-system이 Edge router에 접근하기 위해 필요한 망으로, 구리선, 전화선, 무선 등이 있다. 전체 네트워크 사이즈의 80% 이상을 차지한다. 따라서 Backbone은 20% 정도 차지한다고 보면 된다.  

## Network edge

end systems(hosts)
- 웹 이메일와 같이 응용프로그램 구동

client/server model
- 클라이언트가 요청시 서버가 그 요청의 서비스를 제공
- 웹 클라이언트/서버, 이메일 클라이언트/서버
- 비용 생길 수 있음

peer-peer model (p2p)
- 서버없이 단말끼리 통신
- 서버 역할과 클라이언트 역할 동시에 수행
- 비용 절감. 서비스 안정성 문제

## Network Core

신뢰할 수 있는 라우터끼리 연결되어 통신하므로 제한(regulation)문제는 없다. 하지만 라우터들을 거쳐서 통신할 때 문제를 이야기한다.  

**circuit switching**  
- 먼저 연결하고 데이터를 보냄
- 지연이 없고 자원을 예약하여 퀄리티가 좋음
- 음성 전화

**packet-switching**
- 연결없이 패킷단위로 보냄
- 라우터에 버퍼 꽉 찼을시 패킷 버려짐(packet loss)
- out of order 문제
- 인터넷 데이터

virtual circuit switching: 두 가지 방법을 보완한 방법  

## Access network(망)

end-system이 Edge router에 접속하기 위한 망

- ADSL : 전화선을 이용한 방법. 중계기가 300m 미만

- Cable modems : HFC(hybrid fiber coax) 동축 케이블을 사용하되 광 케이블로 끌고가서 연결. 공유하므로 같이 사용하면 속도 저하

![Cable_Network](https://github.com/lee20h/blog/assets/59367782/edcaa84b-8a95-4744-8f3e-6f9c4d523fff)

- LAN : local area network로 대부분 ethernet을 사용한다. 

- Wireless LANs

## Physical Media

`TP(Twisted Pair) < Coaxial < Fiber < Radio(terrestrial microwave, LAN, wide-area, satellite)`

## Protocol Layer

네트워크가 복잡하기 때문에 레이어를 이룬다. 각 레이어들은 자기들만 집중하면 되기 때문에 간단해진다.  

복잡한 시스템을 위해서 레퍼런스 모델로 레이어를 나눈다. 이때 인터넷 계층은 5계층으로 나뉜다.  

|인터넷 5계층|
|---|
| application |
|transport|
|network|
|link|
|physical|  

계층별 예시

**application**
- FTP, SMTP, STTP

**transport**
- TCP, UDP

**network**
- IP, routing protocols

link
- ppp, Ethernet

physical
- bit on the wire

![Layering](https://github.com/lee20h/blog/assets/59367782/c03e08c2-3959-413b-8baf-bc2afe9b3f62) 

end-system은 5계층, router는 3계층을 구현한다. end-system에서는 각 프로토콜에 있는 것은 상대방 peer에 대해서 1:1 통신되는 방식으로 디자인 되어있다.  

## history

- 1967 : ARPAnet
- 1972 : ARPAnet 퍼블리싱
- 1973 : Ethernet 제안
- 1979 : ARPAnet 200개의 노드 갖음
- 1983 : TCP/IP, DNS
- Early 1990's : ARPAnet 국방부 -> 민간
- 1994 : www 표준

## 그림 출처

- McGraw-Hill 출판사 TCP/IP 프로토콜