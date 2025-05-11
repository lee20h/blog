---
title: "네트워크 레이어 (network layer)에 대해 알아보자"
date: 2020-08-15
tags:
  - network layer
categories:
  - network
publishResources: true
---

# Network Layer

![network_layer](https://github.com/lee20h/blog/assets/59367782/292f0704-c847-4a74-b341-7fe1fad11c5d)  

# Host-to-Host Delivery : Internetworking, Addressing and Routing

## Internetworks

![Internetwork](https://github.com/lee20h/blog/assets/59367782/35a788ba-de4e-46d7-a5ea-44d9b12cb195)

네트워크간의 연결을 맡아주는 부분이 internetwork라고 한다.  

![network_layer_at_the_sources](https://github.com/lee20h/blog/assets/59367782/bae8c013-bd62-4502-ae45-79087e3a835c)  

위의 그림이 네트워크 레이어에서의 ip 패킷 흐름도라고 볼 수 있다. 라우터 측면에서의 흐름도 비슷하며, 초기엔 유닉스 머신으로, 라우터를 사용했다는 것만 기억해두자.  

목적지 도달시에 헤더를 보고 오류 판단한다. 쪼개진 경우 모으는 Reassemble한 뒤 올려보낸다. 쪼개져 있지 않는다면 바로 올려보내는 모습을 볼 수 있다.  

네트워크 계층에서의 Switching은 Packet switching을 사용하고 있고 Datagram approach를 사용한다.  

![datagram_approach](https://github.com/lee20h/blog/assets/59367782/4d7db90d-8e02-4c7a-ac39-67b737aef10a)  

## Addressing

IP주소는 32bit 주소로 전세계에서 공통적으로 사용한다.  
IP주소를 십진수나 이진수 표기법으로 변환하는 것은 많이 해보았으니 넘긴다.  

Classful addressing : A,B,C,D,E 5개의 클래스로 나눈다.  
클래스를 나누는 기준은 Binary notation인 경우 상위비트로 나누게 된다.  

| A | B | C | D | E |
|---|---|---|---|---|
| 0 | 10 | 110 | 1110 | 1111 | 

이러한 식으로 비트 중 1이 언제 나오냐에 따라 나뉘게 된다. 따라서 클래스마다 이진수를 십진수로 바꾸게되면 십진수의 범위를 알 수 있게 된다.  

| A | B | C | D | E |
|---|---|---|---|---|
| 0 ~ 127 | 128 ~ 191 | 192 ~ 223 | 224 ~ 239 | 240 ~ 255 | 

4 byte ip 주소는 NetworkID 와 HostID 두 개로 나뉜다.  

![ip_table](https://github.com/lee20h/blog/assets/59367782/0a645cd8-df95-4115-b016-5a8a1b605052)

A클래스는 네트워크 ID가 7비트이므로 network block는 2^7. 즉 128 블록을 가질 수 있으며, 0.0.0.0 ~ 127.255.255.255까지 사용이 가능하다. 네트워크 ID가 첫 바이트로, `네트워크ID.0.0.0`를 **Network Address**라고 한다. 호스트 ID는 나머지 비트인 24비트를 사용할 수 있으므로, 2^24개 사용이 가능하다. 추가적으로 A클래스는 대부분 낭비되고 있다.  

B클래스는 네트워크 ID가 14비트이므로 network blcok는 2^14. 즉 16384 블록을 가질 수 있으며, 128.0.0.0 ~ 191.255.255.255까지 사용이 가능하다. 호스트 ID는 나머지 비트인 16비트를 사용할 수 있으므로, 2^16개 만큼 사용이 가능하다. B클래스 또한 낭비되고 있다.  

C클래스의 숫자는 많은 기관에 비해 숫자가 부족하다. 네트워크 ID가 21비트 2^21 = 2097152 블록만큼 사용이 가능하다. 호스트 ID는 8비트로 2^8개 사용이 가능하다. 192.0.0.0 ~ 223.255.255.255까지 사용이 가능하다.  

클래스로 주소를 나눈 이유는 네트워크 주소를 한 기관마다 하나를 할당해서 사용하려는 생각이였다. 하지만 IP주소가 부족해지고 있다는 것을 금방 알아차리게 되었다.  

네트워크 주소와 네트워크 ID는 다르다. 네트워크 주소는 네트워크 ID와 호스트 ID를 갖되, 호스트 ID가 0인 주소이다.  

### IP Level

IP 주소는 2 레벨의 계층 구조를 가지고 있다. network id와 host id 두 개의 계층을 가지고 있다. 먼저 network id를 찾고 host를 찾는 구조를 가지고 있기 때문이다.  

### Subnetwork

주어진 네트워크를 서브네트워크로 나눠서 관리하게 되면 편할 수 있다. 예를 들어서 `141.14.0.0`의 네트워크 주소를 가진 네트워크에서 서브네트워크를 나누게 된다면, `141.14.0.0`, `141.14.64.0`, `141.14.128.0`, `141.14.192.0`와 같이 4등분하여 관리하게 된다면 더욱 수월해질 것이다. IP주소가 충분하다면 가능하다.  

서브넷마스크를 이용해서 IP주소와 비트 &연산을 하게되면 네트워크 주소를 얻어 낼 수 있다. 디폴트마스크는 클래스마다 다르게 주어지는데 기본적으로는 A클래스는 `255.0.0.0` B클래스는 `255.255.0.0` C클래스는 `255.255.255.0` 이런식으로 주어지게 된다.  

망관리자가 디폴트마스크를 조금 바꿔서 사용할 수도 있다. 서브넷마스크를 조금 더 편하게 사용할려고 `/`을 이용해서 표현할 수도 있다. 바로 비트만큼 써주는 것인데 `/8`, `/16`, `/24`와 같이 사용하면 된다. 예를 들어 Default Mask가 255.255.0.0라고 했을 때 Subnet Mask를 지정해주면 255.255.224.0으로 사용할 수 있다.  

### DHCP(Dynamic Host Configuration Protocol)

A, B, C 중 일정영역을 Private networks라고 한다. `10.0.0.0 ~ 10.255.255.255`, `172.16.0.0 ~ 172.31.255.255`, `192.168.0.0 ~ 192.168.255.255` 이러한 private ip address는 내부에서만 패킷 통신이 가능하지만, 외부 통신이 불가능하다. 이러한 내부망을 구성하기 위해서 private 주소를 할당해주는 프로토콜이 DHCP이다. 물론 public한 주소를 할당해주기도 하지만 그 부분은 고정 IP를 직접 할당해줘야한다. private 주소에서 인터넷을 접속하기 위해서는 **NAT**이 필요하다.  

DHCP 클라이언트와 서버간의 흐름  
DHCP 클라이언트가 네트워크에 `DHCP discover`를 보내고 DHCP 서버가 받아서 `DHCP offer`를 클라이언트에 보낸다. 이후 클라이언트가 `DHCP request`를 보낸뒤 서버가 마지막으로 `DHCP ACK`를 보내면서 마무리된다.  

다음 그림으로 쉽게 이해해보자.  

![DHCP_flow](https://github.com/lee20h/blog/assets/59367782/1b693f35-8cea-45c3-b44f-48adf4cecf07)

### NAT(Network Address Translation)

유무선 공유기에는 DHCP 서버와 NAT가 들어있다. DHCP 서버가 IP를 동적으로 할당해주고 NAT가 외부로 향하는 패킷을 고정 IP의 포트번호를 각각 할당해서 테이블에 기록한 뒤 패킷을 전송해준다.

로컬에서는 로컬 주소를 가지고 있는 기기가 외부로 패킷을 보내게되면 Router가 부여받은 IP주소에 포트번호를 할당해줘서 NAT Translation table에 WAN과 LAN에서 들어오는 주소들을 기록해놓고 매핑하여 외부와 내부에 각각에 패킷을 전송해주는 역할이다.  

포트번호는 16비트를 사용해서 상당히 많은 로컬주소를 연결해주고 있다. 하지만 이러한 NAT로 ip주소 부족을 다 채울수는 없으므로 IPv6를 이용해야한다.  

## Routing

### Routing Techniques

라우팅 속도를 늘리기 위해 라우팅 테이블을 줄이고 검색 속도를 늘리기 위한 노력이 많이 있었다.

Static vs Dynamic 알고리즘은 장단점이 명확하다. Dynamic은 매번 라우팅을 바꿔서 찾아주는 것이고, Static은 고정된 라우팅으로만 이뤄지는 것이다. Dynamic의 경우에는 트래픽 양이 많아지면 속도가 오히려 느려질 수 있으므로, 트래픽 양이 적을 때 효과적이다. 따라서 두 개의 알고리즘은 장단점이 명확하다.  

## IP Datagram Format

IP 32bit에 들어있는 정보  
|  |  |  |  |
|:-:|:-:|:-:|:-:|
| ver | head(length) | type of service | length(전체 datagram 길이) |
| 16-bit identifier || flag | flagment offset |
| time to live(ttl) | upper layer | Internet Checksum ||
| 32 bit source IP address |||
| 32 bit destination IP address |||
| Options (if any) ||| 
| data (variable length, typically a TCP or UDP segment) |||

TCP의 오버헤드는 20 bytes of TCP + 20 bytes of IP = 40bytes + app layer overhead


## 그림 출처

- McGraw-Hill 출판사 TCP/IP 프로토콜