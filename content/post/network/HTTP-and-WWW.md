---
title: "HTTP, WWW에 대해 알아보자"
date: 2020-09-13
tags:
  - HTTP
  - WWW
categories:
  - network
publishResources: true
---

# HTTP and WWW

## HTTP

HTTP는 TCP port 80번을 이용한다.

### HTTP Transaction

클라이언트가 서버에게 HTTP Request, 서버가 클라어인트에게 HTTP Response 보내는 동작을 Transaction이라고 한다.

![image](https://user-images.githubusercontent.com/59367782/99490355-0979f280-29ad-11eb-8a53-7978a0f16b71.png)

### Request Message

| HTTP Request                        |
|-------------------------------------|
| Request line                        |
| Headers                             |
| A blank line                        |
| Body(present only in some messages) |

Header 이후 new line이 두 개가 나온다면 바로 Body가 나오게 된다.

### Reuqest line

`Request Type(method (GET, POST)) ` (space) `URL` (space) `HTTP version`

### URL

URL : Uniform resource locator

`Method`://`Host`:`Port`/`Path`

`http,https,ftp` :// `www.example.com:2030/test/hello.html`

예제) `GET http://google.com/index.html HTTP/1.1`

### Response message

| HTTP Response |
|---|
| Status line |
| Headers |
| A blank line |
| Body(present only in some messages) |

Request와 동일하게 Header이후에 new line이 두 번 반복하면 Body가 나오게 된다.

### Status line

`HTTP version` (space) `Status code` (space) `Status Phrase`

### Header format

`Header name`: (space) `Header value`

예제) Content-length: 168

### Headers

| HTTP Request | HTTP Response |
|---|---|
|Request line|Status line |
| General Headers|  General Headers |
|Request Headers| Request Headers |
|Entity Headers| Entity Headers |
|A blank line| A blank line |
|Body(present only in some messages)|Body(present only in some messages) |

Example 1)

![image](https://user-images.githubusercontent.com/59367782/99491194-a9844b80-29ae-11eb-8d3b-ea5b077a3b9f.png)

### Persistent vs. non-persistent

- Non-persistent connection
    - 접속한 클라이언트 연결에 대한 응답을 보낸 뒤 바로 끊음
    - 서버는 클라이언트가 이전에 어떤 요청을 했었는지 정보 유지하지 않음
        - 서버 로드가 줄어듦
        - 서버 데이터가 한번에 준비된느 것이 아니라 시간을 두고 준비되는 경우 모든 데이터가 준비될 때 까지 '지연'문제 발생

- Persistent connection
    - 접속한 클라이언트 연결을 응답을 보낸 뒤에도 계속 '유지'하는 것

HTTP version 1.1은 기본적으로 persistent connection을 기본으로 사용한다.

## GET vs. POST Request 매커니즘 이해

- 웹 서비스를 위한 전체 시스템 구성

![image](https://user-images.githubusercontent.com/59367782/99491701-8e660b80-29af-11eb-852b-f93458028639.png)


### HTTP GET Request

![image](https://user-images.githubusercontent.com/59367782/99493475-9c695b80-29b2-11eb-8d30-ac9c52733f8e.png)

`GET http://chonbuk.ac.kr/kor/?menuID=10&pid=254#dept_title HTTP/1.1`  
서버의 클라이언트 입력 정보를 `?key1=value1&key2=value2`의 형태로 제공한다

이러한 GET 파라미터를 UNIX 기준으로 봐보자.

- UNIX environment variable에 저장하게 된다.
- (key, value) 페어를 파싱한 후 이 정보를 기반으로 DB에서 정보를 추출한 후 응답 메시지 작성
- Environment variable은 운영체제에 따라 512 바이트, 1024바이트, 4096바이트 등 그 크기가 다름
- 만일 (key,value) 페어의 길이가 너무 크면 Environment variable에 저장 불가능
- 사용자로부터 이미지나 동영상을 업로드 하는 경우 HTTP 프로토콜은 ASCII기반이므로 MIME encoding을 수행한 후 URL_encoding을 수행하여 ?,=,& 등의 특수 용도로 사용되는 문자들을 %5F와 같은 형태로 변환하는데 그 크기가 수 MB 이상일 수 있음. ex) `?image=A` A <- URL_ENCODE(MIME_encode(image_data))
- 입력 데이터 크기가 크면 GET 방식으로 전달이 불가늫아고 POST 방식으로 전달해야 함

### HTTP POST Request

![image](https://user-images.githubusercontent.com/59367782/99493795-34ffdb80-29b3-11eb-9e1d-e28dc94c5d52.png)

GET과는 달리 HTTP 페이로드(body)에 담겨서 전송이 된다.

Request line: `POST http://chonbuk.ac.kr/kor HTTP1.1`
Body: `?menuID=10&pid=254#dept_title`

- UNIX 표준입력에 저장
- (key, value) 페어를 파싱한 후 이 정보를 기반으로 DB에서 정보를 추출한 후 응답 메시지 작성
- 표준 입력의 크기는 무한대이고 C 언어의 경우 `scanf()`로 읽으면 되며 언어에 맞게 표준 입력 함수를 호출하면 됨
- 사용자 입력 데이터 크기에 무관하게 적용할 수 있고 사용자 브라우저 창에 세부 정보가 직접적으로 출력되지않아 보안적으로 좀 더 안전함
- 사용자로부터 이미지나 동영상을 업로드 하는 경우 GET의 경우와 동일하게 MIME encoding, URL encoding을 해주어야 함.
- 웹 서버를 개발할 때 GET 방식은 웹 브라우저 입력창을 서버 테스트 입력 용도로 사용할 수 있는 장점이 있어 GET으로 개발한 후 POST로 전환하는 방식도 사용

### 정리

- GET/POST방식은 사용자 데이터를 어디에 저장하여 전달하는 지의 차이가 있으며 사용자 데이터 처리 방식은 사실상 동일

![image](https://user-images.githubusercontent.com/59367782/99494189-ee5eb100-29b3-11eb-8bf8-32fc8db50158.png)

## WWW

World-Wide-Web

![image](https://user-images.githubusercontent.com/59367782/99494463-64631800-29b4-11eb-92cd-a6ed4856f6ce.png)

문서에서 문서로 연결되는 것이 거미줄처럼 연결되어 있으면 전 세계로 뻗어있어서 이러한 이름을 갖게 되었다.

### Browser architecture

![image](https://user-images.githubusercontent.com/59367782/99494623-af7d2b00-29b4-11eb-82d4-4c62bd3fdccd.png)

이러한 프로토콜들을 전부 처리할 수 있으며 HTML, JS, JAVA언어를 해석하여 해주는 인터프리터 역할을 해주기도 한다.

### Categories of Web document

- Web document
    - Static: 미리 만들어 놓은 HTML 파일
    - Dynamic
    - Active

서버 프로그램이 동적으로 HTML 파일을 생성 (ex. C 프로그램이 printf() 함수로 HTML 명령어를 출력)

### Static document

클라이언트로부터 Request가 오면 서버측에서 Web document를 그대로 Response로 제공해주는 것이다.

문법은 Mark를 해서 angle bracket으로 속성을 부여할 수 있는데 브라우저마다 속성의 정도가 다를 수 있다.

`<TageName Attribute: Value Artribute: Value> ` Beginning Tag  
`</TagName>` Ending Tag

### Dynamic document

HTTP Request가 오게 되면 서버측에서 프로그램을 통해서 Document를 만들어서 클라이언트에 제공한다. 이런 경우에는 클라이언트가 많아질 수록 서버에 부하가 커지므로 scalability 문제가 있다.

![image](https://user-images.githubusercontent.com/59367782/99496730-2ec02e00-29b8-11eb-8d96-874a07d5baaf.png)

쉘 스크립트로 예시를 들게되면 위의 그림과 같다. 구조만 맞춰주면 쉘 뿐아니라 펄같은 다른 프로그래밍 언어를 사용해도 된다. 

### Active document

HTTP Request가 오게 되면 서버측에서 클라이언트에게 프로그램 코드를 보내 클라이언트에서 프로그램을 이용해서 Document를 만들어서 보게 된다. 이 경우에는 서버에 부하 부담이 없다.

### applet

브라우저가 즉, 클라이언트 사이드에서 해석하여 실행하게 된다. 하지만 속도가 느려서 요즘에 사용 안한다.

![image](https://user-images.githubusercontent.com/59367782/99497243-fb31d380-29b8-11eb-8d03-fda971f76f25.png)

이러한 과정으로 이뤄지게 된다고만 알아두자.

## JAVA vs JavaScript

두 언어는 전혀 다른 언어로 설명을 해본다.

### JAVA

- Sun Microsystems에서 셋톱박스, 전자레인지 등에 활용할 수 있도록 호환성에 초점을 맞춰 개발한 언어

- 기존 임베디드 시스템에서의 제어 프로그래밍
    - 어셈블리(asssembly)언어 -> CPU마다 새롭게 코딩
    - CPU에 무관하게 Java Virtual Machine(JVM)위에 Java byte code를 실행하게 하여 호환성 제공
        - 단 JVM을 울리는 작업은 CPU마다 새롭게 만들어야 함
        - 또한 JVM은 interpretation 방식이므로 느림
            - JIT(Just-In-Time) 컴파일러 등장으로 속도가 개선
                - byte code가 실행될 때 실제 machine instruction으로 compilation을 수행
                - 두번째 실행할될 때는 machine instruction이 바로 실행되므로 빠름
- 안드로이드 앱 개발에 자바 사용 후 주목 받음

### JavaScript

- 넷스케이프 브라우져에 사용자 입력을 받아 브라우저와 상호작용하는 언어를 개발 (라이브 스크립트)
- 이후 넷스케이프에 라이브 스크립트를 통합
- 이후 자바가 인기를 끌자 라이브 스크립트를 자바 스크립트로 명칭 변경
- 자바를 브라우저에서 실행하기 위해서는 별도의 플러그인을 설치해야 함
- 브라우저가 자바스크립트에 쓰여진 언어를 해석해서 실행하며 브라우저에 통합되어 있으므로 플러그인이 필요 없음

## Proxy 서버의 한계점

![image](https://user-images.githubusercontent.com/59367782/99497531-67143c00-29b9-11eb-9f43-68926eeaea22.png)

오늘날에는 Proxy server가 잘 동작하지 않는다.
- Web Server가 dynamic document를 주로 생성하기 때문
- 즉 동일한 HTTP Request에 대해 다른 HTTP Response가 돌아오는 경우가 많다. (caching한 데이터가 무용지물 됨)

## Server-side Programming

- 사용자 요구에 맞게 Dynamic 혹은 Active document를 생성하기 위해 서버쪽에서 행하는 프로그래밍
- 결국 HTTP request 메시지를 수신할 때마다 서버에서 사용자 입력을 가지고 코드를 실행

### 종류

- CGI
- PHP
- JSP
- ASP
- Node.js
- Go
- ...

### CGI

![image](https://user-images.githubusercontent.com/59367782/99930171-7289ad80-2d93-11eb-8a8d-6f4b4ec507ca.png)

- Web server가 test.pl에게 데이터를 넘겨주는 방법이 문제
    - Web serve와 test.pl은 둘 다 process이므로 process communication 문제
    - GET 방식이면 test.pl process의 환경 변수에 저장
    - POST 방식이면 test.pl process의 표준 입력에 저장
- 한 서버에 process 숫자가 100개 넘어서면 **scalability** 문제가 발생

### PHP

![image](https://user-images.githubusercontent.com/59367782/99930395-3b67cc00-2d94-11eb-81e7-eb215b14db4d.png)

- Web server가 test.php를 직접 실행하므로 CGI와 같은 데이터 전달 문제가 없음
- HTTP request가 많아져도 process 숫자가 증가하지 않으므로 **scalability** 문제 완화

### ASP

![image](https://user-images.githubusercontent.com/59367782/99931063-8e428300-2d96-11eb-898a-767fbd3b606d.png)

- 서버가 Microsoft 운영체제이어야만 하며 윈도우 운영체제의 보안문제에 취약
- 윈도우 서버가 scalability가 좋지 않기 때문에 확장성 측면에서 다소 불리
- 윈도우 서버가 유닉스 서버에 비해 설치가 쉽기 때문에 학원에서 많이 다룸
    - 운영체제가 특정 한 회사에 한정되고 표준과 무관하므로 가급적 피하는 것이 좋음

### JSP

![image](https://user-images.githubusercontent.com/59367782/99931097-af0ad880-2d96-11eb-9760-4dd433286f95.png)

- Web server가 test.jsp를 직접 실행하므로 CGI와 같은 데이터 전달 문제가 없음
- Java 언어에 익숙한 사용자가 server side programming할 때 고려
- 생성 -> 변환 -> 실행을 해야하기 때문에 서버 부하가 큼 (대략 PHP 방식보다 2-3배 느림)

### Node.js

![image](https://user-images.githubusercontent.com/59367782/99931141-d2358800-2d96-11eb-853a-ef07dcff3ac2.png)

- Client 웹 페이지에 개발에 사용된 자바스크립트를 서버에도 사용
    - 자바스크립트에 익숙한 개발자들이 서버 프로그래밍할 때 용이
    - 비동기 I/O 사용 (I/O가 많은 작업을 처리할 때 유리)

### Go

![image](https://user-images.githubusercontent.com/59367782/99931189-f729fb00-2d96-11eb-90c3-65aa50922960.png)

- Google이 networked server, tools, system programming에 적합하게 **단순(simplicity)**하고 고성능을 목적으로 개발
- 새로운 언어인 GO를 배워야 하는 부담이 존재
- 클래스 계층구조를 없애고 복잡한 객체는 composition으로 생성
- 일반적으로 현재 **가장 고성능**

### Performance 비교

- 일반적으로 (GO , Node.js )-> (PHP, ASP) -> JSP -> CGI
- 각 sider-side programming은 계속 성능 개선을 하고 있으므로 유의
    - CGI -> FastCGI, PHP -> ReactPHP (event-driven), … 
- 응용의 특성 (예: I/O 요구 특성)에 따라 성능이 달라질 수 있음
- 프로젝트의 특성(예: 보안, 호환성 유지 등)에 따라 맞는 것을 선택

## 그림 출처

- McGraw-Hill 출판사 TCP/IP 프로토콜