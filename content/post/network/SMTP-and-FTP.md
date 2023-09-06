---
title: "SMTP와 FTP에 대해 알아보자"
date: 2020-09-09
tags:
  - SMTP
  - FTP
categories:
  - network
published: true
---

# SMTP and FTP

SMTP: Simple Mail Transfer Protocol  
FTP: File Transfer Protocol

## Electronic Mail

### User Agent

![image](https://user-images.githubusercontent.com/59367782/98677871-02e1ee80-23a1-11eb-9955-0f625b5cdfb5.png)

### MIME (Multipurpose Internet Mail Extensions)

- SMTP는 오직 7-bit ASCII 형식만 전송 가능
    - Extended(혹은 Enhanced) SMTP는 8bit도 가능하지만, 호환성 문제
    - 초기에 e-mail은 text만을 생각하기 때문

- 7-bit로 표현하지 못하는 데이터를 전송을 하기 위해 MIME이 생겨남
    - 한국어, 중국어, 프랑스어 등
    - 비디오나 오디오 데이터 등

- MIME은 SMTP를 대치하는 것이 아니라 보완하기 위함

![image](https://user-images.githubusercontent.com/59367782/98678980-aed80980-23a2-11eb-8752-7f1e0b62be17.png)

```
Email Header
---
MIME-Version: 1.1
Content-Type: type/subtype
Content-Transfer-Encoding: encoding type - 7bit 변경 방법
Content-id: message id
Content-Description: textual explanation of nontextual contents
---
Email body
```

MIME이 EMail이 아닌 다른 곳에서도 쓰일 수 있으므로, encoding-type을 `7Bit`나 `Base64`, `Quoted-printable`을 사용해야 한다.

### Base64

![image](https://user-images.githubusercontent.com/59367782/98679677-b9df6980-23a3-11eb-8bd2-8cbf3caafa92.png)

결과적으로 24비트가 32비트로 변환된다. 즉, 32/24 = 1.33이므로 33% 오버헤드가 발생한다.

4bit -> 6bit로 바꾸기 때문에 64개의 매핑 테이블이 생긴다. 이 갯수 때문에 Base64라고 불린다.

### Quoted-printable

Non-ASCII 데이터, 즉 상위 비트가 1인 바이트만 3바이트로 변환한다. (`=`, 16진수 값의 ASCII 값)

![image](https://user-images.githubusercontent.com/59367782/98679972-1b9fd380-23a4-11eb-8d2b-311698d14f03.png)

Non-ASCII를 보면 상위 비트가 1이므로 16진수로 표현하면 `9D`이다.

![image](https://user-images.githubusercontent.com/59367782/98680057-3a05cf00-23a4-11eb-8c2d-d4eb5ed83b99.png)

전체 데이터 중 Non-ASCII **데이터의 비율이 작을 때** Base64 encoding보다 **효율적**이다.

### E-mail 전달

- E-mail의 실제 전달은 MTA(Mail Transfer Agent)가 맡음
    - 예: UNIX의 경우 sendmail daemon 프로그램
    - 메일을 보내기 위해서는 MTA client가 필요
    - 메일을 수신하기 위해서는 MTA server가 필요
    - MTA client와 MTA server는 SMTP 사용

### Commands and responses

![image](https://user-images.githubusercontent.com/59367782/98680546-e5af1f00-23a4-11eb-9ebf-1d72122e5ed2.png)

예시 : telnet www.example.com port: 25
- MTA server 포트는 25번
- telnet을 이용하여 25번 포트에 TCP 연결
- 키보드로 치는 것을 전송
- telnet에서 ctrl-d를 누르면 TCP 종료

```
S: 220 www.example.com ESMTP Postfix - 서버응답
C: HELO mydomain.com
S: 250 Hello mydomain.com
C: MAIL FROM:<sencder@mydomain.com> - Client
S: 250 OK
C: RCPT TO:<friend@example.com>
S: 250 OK
C: DATA
S: 354 End data with <CR><LF>.<CR><LF>
C: Subject: test message
C: From: sender@mydomain.com
C: To: friend@example.com
C:
C: Hello,
C: This is a test.
C: Goodbye.
C: .
S: 250 OK: queued as 12345
C: QUIT
S: 221 Bye
```

### MTA client and server

A -> B 방향으로 메일을 보내면 ?

```
SMTP   | SMTP    SMTP    |                  |  SMTP     |   Mail Access Protocol (※No SMTP※)
client | server  client  |                  |  server   |   IMAP, POP3
A -> Mail Server ------> Internet ------> Mail server -> B
```

![image](https://user-images.githubusercontent.com/59367782/98681793-7cc8a680-23a6-11eb-8b0e-88fd37c4659a.png)

### Mail Access Protocols

- POP3 (Post Office Protocol version 3)
    - Text로 이뤄짐
- IMAP4 (Internet Mail Access Protocol version 4)
- Web-based Mail
    - naver나 Gmail 등
    - 웹 브라우져 (IE, netscape)로 HTTP를 이용
        - sending mail server에서 receiving mail server까지는 SMTP 사용

## File Transfer

FTP는 TCP를 사용하며 **2개의 TCP 연결**을 갖는다. Port 21를 이용한 제어를 위한 연결, Port 20을 이용하여 데이터를 전송하고 받는다.

ASCII를 이용하여 TCP 연결은 한 뒤 데이터를 전송하기 때문에 구현은 어렵지 않지만 User Interface에 따라 달라지게 된다. 

### Using the control connection

![image](https://user-images.githubusercontent.com/59367782/98682938-d54c7380-23a7-11eb-9121-cef80001f9d2.png)


port 20을 사용한 TCP 연결에서 200 ok를 받게 되면 데이터를 전송한다.

![image](https://user-images.githubusercontent.com/59367782/98683006-e72e1680-23a7-11eb-99a6-07c97d5e2363.png)

anonymous를 지원하므로 연결 상대가 허용하면 익명으로도 연결할 수 있다. 또한, **컨트롤 채널**로 **데이터를 주고 받는 방식**인 것을 기억하자.


## 그림 출처

- McGraw-Hill 출판사 TCP/IP 프로토콜