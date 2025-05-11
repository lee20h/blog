---
title: "기본적인 네트워크 보안에 대해 알아보자"
date: 2020-09-21
tags:
  - security
categories:
  - network
publishResources: true
---

# Network Security Protocols

## 네트워크 보안 개요

### 인터넷 망구조가 안고 있는 보안 위협

- 인터넷 망은 여러 관리 주체가 분산 소유/관리 함
    - 여러 인터넷 서비스 제공 
    - 망 관리자는 자신의 망을 통과하는 패킷을 모니터링 할 수 있음

- 다른 사람이 자신이 보낸 데이터를 받을 수 있음
    - 엿보기, 허브(hub)의 mirroring 기능 등의 이유

- 다른 사람이 자신이 보낸 것처럼 할 수 있음
    - IP 주소는 ICANN (Internet Corporation for Assigned Names and Numbers)에서 할당하고 관리
    - ISP는 자신에게 할당된 IP 주소들만 사용
    - IP 주소를 알면 해당 컴퓨터 혹은 최소한 특정 지역을 찾아 낼 수 있음
    - 따라서, “기술적으로는“ IP spoofing을 막을 수 있음
    - 그러나, 이 요구사항은 필수가 아니기 때문에 모든 라우터가 IP spoofing을 방지하는 것은 아님
    - IP spoofing은 DOS (Denial-Of-Service) 공격의 온상

### IP Spoofing

![image](https://user-images.githubusercontent.com/59367782/100573869-19ce8d80-331c-11eb-8df0-540e0aef5760.png)

### 분산 DOS 공격

![image](https://user-images.githubusercontent.com/59367782/100574026-7762da00-331c-11eb-88d7-cea166d6e717.png)

### 포트 스캐닝

- IANA에서 포트번호와 응용간 매핑을 정의
    - 65536개 중 0번부터 1023번까지 예약되어 있음
    - 예: HTTP:80, FTP:21

- 포트 스캐닝 공격
    - 특정 포트에 특정 메시지를 보내고 반응을 수집함으로써 어느 포트가 열려 있는 지를 검사
    - 해당 포트에 보안 공격을 감행

### 계층별 보안 프로토콜이 필요한 이유

- 하위 계층, 예를 들어 IP 계층에서 제공하면 굳이 TCP나 응용 계층에서 제공할 필요가 없는 것 아닐까?

- 이유 2가지
    - IP 계층 보안은 모든 패킷들을 암호화할 수 있지만 사용자 수준의 보안(인증)을 할 수 없음
        - 즉, “어느 컴퓨터”에서 작업을 요구하는 지는 알아도, “누가” 하는 지는 모름

    - 많은 부분을 IP 계층 보안이 처리할 수 있다손 치더라도 응용들이 원하는 다양한 서비스가 모든 라우터에 설치되기까지는 많은 시간이 소요

## 웹서버 정보 보안

### HTTP 보안

- HTTP “기본” 사용자 인증
    - 서버에 htpasswd 프로그램으로 쉽게 설정
    - 특정 디렉토리에 있는 파일들에 인증을 한 후 접근하게 함
    - Base-64 인코딩하여 전달


- HTTP 기본 사용자 인증은 “안전하지 않음”
    - 사용자 이름/비밀번호를 바로 읽을 수 없도록 인코딩됐지만 디코딩 알고리즘을 돌리면 즉시 해독됨
        - 해독을 위한 **비밀키가 전혀 필요 없음**
    - 심지어 인코딩된 것을 그대로 해커가 재사용해도 그 보안 파일을 접근할 수 있음 (replay 공격)
    - 보안 설정한 파일을 접근할 때 마다 브라우져가 자동으로 사용자 이름/비밀번호를 같이 보내기 때문에 쉽게 공격자가 몰래 복사할 기회가 많음

- HTTP 요약 접근 인증 방법

![image](https://user-images.githubusercontent.com/59367782/100574429-5bac0380-331d-11eb-9bd5-f93deb75f82e.png)

- HTTP 요약 접근 인증 방법의 취약점
    - Man-in-the-middle 공격에 취약
        - 근본적 이유는 서버가 브라우저에게 인증 받지 않기 때문 
- HTTP 요약 접근 인증 방법의 또다른 취약점
    - 암호 자체는 아니라도 관련 데이터를 서버에 저장하기 때문에 위험
- WWW 보안에 완전하지 않음
    - 인증만 제공하고 암호화나 무결성 보장은 하지 않음
    - 처음 사용자와 서버간에 암호를 정하는 방법이 명시 안됨
- 대신 SSL/TLS를 이용한 HTTP 보안이 널리 쓰임

## 전자메일 보안

### 안전한 전자 메일 설계

- 목적
    - 두 사람이 보낸 메일을 타인은 읽을 수 없게 함 (**기밀성**)
    - 받은 메일이 진정 상대방이 보낸 것인지 확신 (**인증**)
    - 받은 메일이 타인이 중간에서 내용 변경한 것이 아님을 확신 (**무결성**)
    - 받은 메일을 상대방이 보낸 것이 아니라고 우기는 것을 방지 (**부인 방지**)

- 대칭키로 문서를 암호화하는 경우와 공용키/개인키로 암호화하는 경우

### 대칭키 1개만 사용하는 방법

![image](https://user-images.githubusercontent.com/59367782/100574847-3bc90f80-331e-11eb-92a0-2a1a33aac52a.png)

- 둘이 모두  “안전하게” 하나의 비밀키를 유지해야 함
- “공유”되는 비밀키 하나로 암호화, 인증, 무결성을 모두 신뢰
- 상대를 완전히 서로 신뢰하는 경우만 적용 가능


### 안전한 전자메일을 위한 PGP

- PGP (Pretty Good Privacy) 프로그램
    - 1991년에 Phil Zimmermann에 의해 작성
        - 3년동안 미국연방 조사의 표적
    - 전자우편을 암호화할 때 사실상(de facto) 표준이 됨
        - PGP는 전자우편뿐 아니라 다른 응용(예:파일암호)에도 적용
    - `www.pgpi.org` 에서 무료로 다운할 수 있음

- 특성
    - 기밀성 : 대칭키 및 공용키 암호화 기술 사용 
    - 인증, 무결성 : 해쉬(hash) 함수와 디지털 서명 사용

### PGP: 보내는 부분

![image](https://user-images.githubusercontent.com/59367782/100574710-f1479300-331d-11eb-8c43-5a2d26cf0bf9.png)

- 춘향의 서명을 요약 대신 e-mail 자체를 사용할 수 있지만 속도가 문제
    - E-mail 자체가 아닌 해싱(hashing)한 e-mail을 대신 (춘향의 개인키로) 서명함으로써 속도 향상

- E-mail과 서명을 몽룡의 공용키로 암호화할 수 있지만 공용키가 너무 커서 암호화 속도가 너무 느림
    - 세션키(춘향이 임의로 생성한 작은 크기의 대칭키)를 대신 암호화에 사용하고 세션키를 몽룡의 공용키로 암호화해서 같이 보냄으로써 속도 향상

### PGP: 받는 부분

![image](https://user-images.githubusercontent.com/59367782/100574961-7af76080-331e-11eb-8915-aeb8e2464d75.png)

### 공용키 검증 방식

- PGP를 이용한 안전한 행위를 위해 가장 주의해야 하는 것은 과연 공용키가 진정 상대방의 것이 맞는 지 확인하는 것

- PGP는 CA (Certification Authority)와 다른 방식

- CA의 공용키 인증 방식
    - CA(예:VerySign)를 통한 검증 방식은 우리가 그 CA를 신뢰하고, CA가 책임지고  (공용키, 소유자)의 짝을 직접 인증하는 방식
    - 현재 웹 브라우져가 채택하는 방식

### CA 공용키 검증 방식

- 각 주체(사람, 라우터)는 자신의 공용키를 CA에 등록
    - 각 주체는 CA에게 “자신에 대한 인증”을 받음
    - CA는 “이것이 이 사람의 공용키임”을 의미하는 인증서를 저장

![image](https://user-images.githubusercontent.com/59367782/100575152-da557080-331e-11eb-9bf4-aec27f3672a2.png)

- 몽룡이 춘향의 공용키를 원하는 경우
    - 춘향의 인증서를 구함 (춘향에게 혹은 다른 곳)
    - CA의 공용키로 적용해서 춘향의 공용키 구함


![image](https://user-images.githubusercontent.com/59367782/100575231-ffe27a00-331e-11eb-9252-aa3d51ac5b22.png)

- CA 공용키 인증서 내용:
    - 시리얼 번호 (CA가 할당하는 유일한 인증서 번호)
    - 인증하는 주체에 관한 정보
    - 인증서 발급한 곳(CA)의 정보
    - 유효일
    - 디지털 서명

### PGP의 공용키 검증 방식

- “Web of Trust”
    - 우리의 신뢰 구조와 비슷: 누가 누구를 신뢰하느냐는 “자신”의 몫
        - 자신이 맞다고 “확인”한 것은 믿음
        - 자신이 신뢰하는 사람이 틀림없다고 한 것은 믿음
    - 각 키는 validity와 trust 항목이 있음
        - Validity: 그 키가 그 사용자임을 말하는 것
        - Trust: 그 사용자를 얼마나 신뢰하느냐를 말하는 것
            - Trust level로 신뢰 정도를 표현

## 전송 계층 보안 프로토콜: SSL, TLS

### 전송 계층 보안 프로토콜 역사

![image](https://user-images.githubusercontent.com/59367782/100575612-ceb67980-331f-11eb-8a3b-3feac68bbb23.png)

### SSL 소개

- SSL (Secure Socket Layer)
    - 웹 클라이언트와 서버 사이에 데이터 암호화와 인증을 위해서 설계한 프로토콜
    - 웹 응용 외에도 TCP위에서 동작하는 모든 응용에 적용 가능
    - 오늘날 거의 모든 브라우저와 웹 서버에 구현됨
        - HTTPS로 시작되는 URL은 SSL을 통해서 HTTP를 구현한 것

### SSL 프로토콜

![image](https://user-images.githubusercontent.com/59367782/100575685-f7d70a00-331f-11eb-92a8-afd6a3391c78.png)

![image](https://user-images.githubusercontent.com/59367782/100575698-ff96ae80-331f-11eb-8b16-1fc0aa93a633.png)


### SSL 속도 성능에 관해

- SSL을 사용했을 때 성능상의 문제점
    - 사용자가 인지할 수 있음
    - SSL세션을 초기화할 때 요구되는 공용키 암호화와 해독에 주로 많은 시간이 걸림
    - HTML 데이터를 암호화하고 및 해독하는 시간
        - 상대적으로 위의 경우보다 덜 심각함

## IP 계층 보안 프로토콜: IPsec

### IPsec 이전 네트워크 보안 프로토콜

- IETF IPSEC WG 탄생 이전의 네트워크 계층 보안 프로토콜
    - Security Protocol 3(SP3)
        - NSA, NIST가 개발한 것으로 주로 미국 군사용 외에 널리 퍼지지 못함

- Network Layer Security Protocol (NLSP)
    - ISO에서 Connectionless Network Protocol (CLNP)을 보호하기 위해서 개발, SP3과 호환 안됨

- Integrated NLSP (I-NLSP)
    - SP3와 비슷하며 NIST가 IP와 CLNP 둘 다에 보안 서비스를 제공하기 위해서 개발

- swIPe 
    - John Ioannidis와 Matt Blaze가 개발한 IP 보안 프로토콜
    - `ftp://ftp.csua.berkeley.edu/pub/cypherpunks/swIPe/swipe.tar.Z`

### IPsec 프로토콜

- 두 가지 종류
    - ESP (Encapsulating Security Payload) 프로토콜
        - 암호화, 무결성, 인증 (AH 보다는 범위가 작음)
    - AH (Authentication Header) 프로토콜
        - 인증, 무결성
        - 암호화는 제공 안함

## VPN

### 사설망

![image](https://user-images.githubusercontent.com/59367782/100576012-9499a780-3320-11eb-9ba0-b17cbe3ff50c.png)

### 혼합망

![image](https://user-images.githubusercontent.com/59367782/100576034-9d8a7900-3320-11eb-8623-e3da5cbd4c9c.png)

### 가상 사설망

![image](https://user-images.githubusercontent.com/59367782/100576051-a5e2b400-3320-11eb-9179-a2a7330b8d36.png)


### 가상 사설망 구현

![image](https://user-images.githubusercontent.com/59367782/100576072-ada25880-3320-11eb-85fe-42339938e70c.png)


## 그림 출처

- McGraw-Hill 출판사 TCP/IP 프로토콜