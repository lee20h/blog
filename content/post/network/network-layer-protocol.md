---
title: "네트워크 레이어 프로토콜 (network layer protocol)에 대해 알아보자"
date: 2020-08-19
tags:
  - network layer
  - ARP
  - IP
  - ICMP
categories:
  - network
published: true
---

# Network Layer Protocols: ARP, IPv4, ICMPv4, IPv6 and ICMPv6  

Network layer 프로토콜 : IGMP, ICMP, IP, ARP, RARP  

## ARP  

ARP(Address Resolution Protocol) : IP 주소를 브로드캐스트를 통해서 요청하고 해당 IP주소 가진 시스템이 물리주소를 유니캐스트로 응답을 한다.  

## IP

### Datagram

IP 패킷을 Datagram이라고 부르며, 구조는 이전에 본 부분과 같다.  

|  |  |  |  |
|:-:|:-:|:-:|:-:|
| ver | head(length) | type of service | length(전체 datagram 길이) |
| 16-bit identifier || flag | flagment offset |
| time to live(ttl) | Protocol | Internet Checksum ||
| 32 bit source IP address |||
| 32 bit destination IP address |||
| Options (if any) ||| 
| data (variable length, typically a TCP or UDP segment) |||

여기서 16-bit identifier 와 flags, Fragmentation(13bit)이 Framentation와 관련된 필드이다. Protocol 부분은 Datagram을 받을 프로토콜을 기입한다. Datagram의 전체 길이가 16bit이므로, 2^16바이트를 차지하고 Header의 길이는 20 ~ 60 바이트까지 차지할 수 있다. 20바이트는 옵션이 없는 경우 각 필드들의 row가 5개이기 때문이다.  

따라서, datagram의 헤더를 추가한 전체 사이즈가 total length이다. 또한, 헤더의 프로토콜 값을 따라서 목적지가 여러 프로토콜(ICMP, IGMP, TCP, UDP, OSPF)로 나뉜다.  

### Checksum 계산

![checksum_calculation](https://github.com/lee20h/blog/assets/59367782/f03bf4fe-08bf-4383-91e0-ab43df37db66)

CRC가 막강한 에러체크를 할 수 있지만 어떤 프로토콜이 들어올지 모르기 때문에 유지한다. 덧셈 이후 1의 보수를 취해서 해당 값을 Checksum에 기입한다.

### Fragmentation

통신 간에 라우터를 거치게 되는데 라우터마다 최대 패킷 사이즈가 다르면 패킷을 전송할 수 없기 때문에 해당 패킷을 쪼개는 것을 Fragmentation이라고 한다.  

MTU(Maximum length of data to be encapsulated in a frame) : IP 패킷의 최대 크기 (ex) 이더넷 프레임의 최대 사이즈)  

예시를 들어서 보자. 4000 byte datagram, MTU = 1400 bytes  

 | length = 4000 | ID = x | flag 3bit | offset = 0 13bit | ~ |  
 |---------------|--------|-----------|------------------|---|
이러한 datagram이 있을 때 fragmentation을 통해서 쪼갠다.  

 | length = 1400 | ID = x | fragflag = 1 3bit | offset = 0 13bit   | ~ |  
 |---------------|--------|-------------------|--------------------|---|
 | length = 1400 | ID = x | fragflag = 1 3bit | offset = 175 13bit | ~ |  
 | length = 1200 | ID = x | fragflag = 0 3bit | offset = 350 13bit | ~ |  

fragflag로 fragmentation이 일어났는지 확인할 수 있다.  offset*8 = byte offset 크기를 8의 배수로 쪼개서 offset을 지정해줘서 13비트 안에 사용할 수 있게한다.  

이러하게 나뉘어진 datagram이 각자 라우팅되어서 목적지로 가게 된다. 목적지에서 reassemble을 통해 다시 조합되어 원본과 같은 datagram을 전송할 수 있다.  

## ICMP

ICMP란, IP계층에서 에러 발견이나 에러 수정 메커니즘이 부족하기 때문에 생긴 프로토콜이다. 하지만 기능이 매우 제한적이다.  

ICMP는 항상 original source에게 에러를 알려주는 역할을 한다.  

### Error-reporting messages of ICMP

- Destination unreachable : 패킷이 목적지에 도착하지 못한다면 ICMP가 라우터나 호스트가 전달할 수 없다는 내용을 패킷을 보낸 soruce에게 메세지로 보낸다.

- Source quench : 라우터버퍼가 가득차서 보낼 수 없을 때 source에게 메세지를 보내는 경우다. IP는 흐름제어와 혼잡제어를 할 수 없으므로 에러 메시지를 보낸다.  

- Time exceeded : TTL을 통해서 패킷이 만료된 경우 source에게 보내는 메세지다.  

- Parameter problem : 패킷을 받았으나 특정 필드 값이 애매하거나 잃어버린 경우 source에게 알려준다.

- Redirection : 호스트가 정적 라우팅을 사용할 때 default 라우터에게 패킷을 보낸다. 이때 default 라우터를 잘못 지정되었을 때 해당 라우터가 알맞은 라우터를 아는 경우 호스트에게 알맞은 라우터를 알려준다.  

오늘날에는 해킹의 위험으로 거의 서비스 하지 않고 무시하는 경우가 많다. 추가적으로 IP는 흐름제어와 혼잡제어를 못하며, TCP에서 제공하는 것을 알아두면 좋다.  

ICMP에 query를 보내서 답을 하는 Echo나 Time-stamp를 사용할 수 있으나 이 기능 또한 해킹의 위험으로 서비스 하지 않는다.  

## IPv6

### IPv6(IPng)

IPv4는 32bit이지만, IPv6는 128bit를 가진다. IPv4보다 전체적으로 개선되었으며, QOS가 좋아졌다. 인터넷 데이터를 넘어 전화선도 지원한다.  

128 bits 5 16 bytes 5 32 hex digits -> hexadecimal colon notation

`FDEC:BA98:7654:3210:000F:BBFF:0000:FFFF` 로 표기한다. 축약을 하여 표기하면 0을 표기하지 않고 하기도 한다.  

`FDEC:BA98:0074:3210:000F:BBFF:0000:FFFF` => `FDEC:BA98:74:3210:F:BBFF:0:FFFF`  

더 나아가 `FDEC:0:0:0:0:BBFF:0:FFFF` => `FDEC::BBFF:0:FFFF` 로 고치기도 한다.  

### Format  

| ver                                                          | PRI | Flow label  |           | |
|--------------------------------------------------------------|-----|-------------|-----------|-|
| Payload length                                               |     | Next header | Hop limit |
| Source address                                               |     |             |           |
| Destination address                                          |     |             |           |
| Payload extension headers + Data packet from the upper layer |     |             |           |

version, Priority, Flow label (Flow(Session)마다 Quality)

### Comparsion IPv4

IPv4에는 IGMP, ICMP, ARP, RARP가 있었지만 IPv6에서는 ICMPv6만 사용한다.  

### IPv4 -> IPv6 Strategies

두 버전의 호환성 문제가 있다. 문제를 해결하기 위한 전략을 살펴보자.  

- Dual stack : IPv4 시스템과 IPv6 시스템 스택을 두 개를 동시에 유지하여 버전에 맞게 스택을 매핑하여 패킷을 전송하며 사용한다. 문제는 v4 -> v4와 v6 -> v6로 패킷 전송은 가능하지만 v4 -> v6로 패킷 전송이 불가능하다.

- Tunneling : IPv6 호스트 사이에 v4와 v6 둘 다 가능한 라우터를 둬서 IPv4는 평소와 같이 전송을 한다. v6는 라우터가 자신을 source로 하고 상대 라우터를 destination으로 해서 v4 패킷으로 만들어서 전달한 뒤 destination 라우터가 v4패킷을 v6패킷으로 바꿔서 전달한다. Dual stack과 같이 버전사이의 전송은 안되나, 라우터를 그대로 사용할 수 있어서 경제적으로 이점은 있다.

- Header translation : 모든 라우터들은 v6로 설치되며, v4 패킷이 오면 v6로 헤더를 변환하여서 전송한다. 헤더 변환이 가능하므로 버전 사이의 전송이 가능하다. 하지만 flow label problem이 있다. 따라서 일부 패킷만 변환이 가능하고 일반적으로 전부 변환이 가능하지 않다.

결과적으로 IPv6로 전환할려면 라우터를 다 바꿔야한다. 하지만 일시에 라우터를 바꾸는 것은 현실적으로 불가능하다. 따라서 점진적으로 변경해야하므로 기간이 꽤나 걸릴 것으로 예상된다.


## 그림 출처

- McGraw-Hill 출판사 TCP/IP 프로토콜