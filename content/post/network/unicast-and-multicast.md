---
title: "unicast and multicast에 대해 알아보자"
date: 2020-08-21
tags:
  - unicast
  - multicast
categories:
  - network
published: true
---

# Unicast and Multicast Routing

- Unicasting : `1:1 방식`, Class A,B,C
- Multicasting : `1:N 방식`으로, Group 형성 방법이 문제가 된다. 모든 계층에서 구현 가능하다. 여기서 설명할 내용은 Network 계층에서의 multicasting 구현 방법. 최근의 multicasting은 응용계층에서 구현한 것이다. Class D
- Broadcasting : `1:All` 방식
- Anycasting : `1:anyone of them` 방식

![overlay_network](https://github.com/lee20h/blog/assets/59367782/6a5ad4b3-72f2-4b92-ba0b-4dd9c23e0ea8)

**Overlay network** : 응용 프로그램 입장에서 실제 물리적망(검은색) 외에 가상의 논리적 망(녹색)으로 간주 할 수 있다. 2의 지수승으로 점점 늘어날 수 있다. 새로운 인터넷 서비스를 제공할 때 마다 새로운 Overlay network 구성을 고려해 볼 수 있다.

## Unicasting

개념만 이해하면 된다. Source가 Destination으로 보내는데 라우터마다 input과 output이 하나 있는 것이다. 라우터가 수신된 패킷을 오직 하나의 포트로 보내는 방식이다.  

Protocol은 불필요해보이므로 생략함. (장비 세팅 때 필요함)  

## Multicast Routing

Router가 Multicast를 어떻게 지원하는 가에 대한 이야기를 해보자.  

- IGMP : 단말과 라우터들간의 가입 요청/해제
- Multicast tree : Source 단말에서 Destination 단말까지의 가는 루트 (트리이므로, loop가 없다)

Multicast routing은 라우터가 수신된 패킷을 트리 구성에 따라 여러개의 포트로 전달한다.  

IGMP 프로토콜은 group management protocol로 그룹에 참여, 탈퇴에 대한 프로토콜이다. 멤버의 정보는 라우터에 저장되므로 IGMP를 지원하는 라우터만이 저장가능하다. 해당 프로토콜의 정보는 소실될 수 있으므로 2번 보낸다.  

### Logical tunneling  

M-bone : Multicast Back-bone을 뜻하는 용어로, 멀티캐스트를 지원하는 라우터들은 멀티 캐스트 데이터를 받게 되면 Logical tunnel을 형성하여 멀티캐스트를 지원하는 라우터끼리만 통신하는 것이다. 멀티캐스트 데이터를 멀티캐스트 지원하지 않는 라우터에게 보내게 되면 라우터들을 기준으로 좌우로 통신이 가능하다. IPv6에서 얘기한 Logical Tunneling와 유사하다고 생각하면 된다.  

기존 인터넷망에서도 멀티캐스트 지원 라우터를 몇 개 추가하는 것으로 멀티캐스트 서비스를 제공한다. 멀티캐스트 지원 라우터를 늘려감에 따라 멀티캐스트 서비스가 확대 된다.  

현재는 잘 사용하지 않고, 서버를 연결하여 어플리케이션 단에서 제공한다. 왜냐하면 경제적인 이유도 있으며, 에러가 발생했던 과거에 의해서 사용을 꺼려한다.

## 그림 출처

- McGraw-Hill 출판사 TCP/IP 프로토콜