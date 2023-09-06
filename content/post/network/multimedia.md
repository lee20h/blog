---
title: "Multimedia에 대해 알아보자"
date: 2020-09-18
tags:
  - file
  - multimedia
categories:
  - network
published: true
---

# Multimedia

### 기술

- Video 데이터
    - 640x480 (VGA) x RGB (3byte) x 30 fps x 3600s(1 hour) = 약 100GB
        - 1990년 초 HDD 1MB 당 1만원 산정시 10억
    - 고효율 압축은 선택이 아닌 필수
    - B-ISDN 서비스의 경우 초기에 저해상도, 저프레임, CBR 영상 서비스 추진
        - 90년대 Video-On-Demand(VOD) 서비스 추진 -> 사업접기 -> 2000년대 다시 추진
            - 당시 video data는 HDD, 테이프 등에 저장 후 전송하는 연구
            - 당시의 인프라, 하드웨어 환경, 사회적 배경을 기준으로 당시 선택을 평가해야 하고, 미래에 대한 안목의 중요성을 알 수 있음

- 90년대 이후 Video 데이터 압축 표준 진행
    - MPEG-1 (기존 TV 품질 압축)
        - 인텔 펜티엄 CPU 성능으로 소프트웨어 압축을 시연
    - MPEG-2 (HDTV 품질 압축)
    - MPEG-3 (mp3 – 음성 압축)
    - MPEG-4 (.mp4 – 객체기반 압축)
        - 개념은 화면 단위가 아닌 한 화면내 객체 단위로 압축
        - 실제 구현은 Layered encoding (base layer + enhanced layer)
        - T-commerce의 미래로 기대했으나 아직 미미한 실정


### Internet audio/video

- Internet audio/video
    - Streaming stored audio/video
        - ex) VOD
    - Streaming live audio/video
        - ex) radio, TV
    - Interactive audio/video
        - ex) zoom

## Audio/Video Compression

### JPEG gray scale

![image](https://user-images.githubusercontent.com/59367782/99933755-4bd17400-2d9f-11eb-9da7-2989f1955ec6.png)

- 이미지 하나를 압축하는 방식
- 블락 단위로 인코딩
- 전송 중에 데이터가 깨지더라도 해당 프레임만 영향을 받는다.

### JPEG process

![image](https://user-images.githubusercontent.com/59367782/99933825-77545e80-2d9f-11eb-9c5e-1e2e92770dff.png)

### process

- uniform gray scale

![image](https://user-images.githubusercontent.com/59367782/99933842-863b1100-2d9f-11eb-9f50-905856d5f137.png)

- two sections

![image](https://user-images.githubusercontent.com/59367782/99933843-8804d480-2d9f-11eb-9d28-9cee5456db4f.png)

- gradient gray scale

![image](https://user-images.githubusercontent.com/59367782/99933849-89ce9800-2d9f-11eb-88a2-d18c8fab344b.png)

### Reading the table

![image](https://user-images.githubusercontent.com/59367782/99933904-b5ea1900-2d9f-11eb-8d30-baef25814167.png)

해당 데이터를 압축한 것이 JPEG라고 할 수 있다.

### MPEG frames

![image](https://user-images.githubusercontent.com/59367782/99933935-ce5a3380-2d9f-11eb-9446-088ca30281aa.png)

- construction

![image](https://user-images.githubusercontent.com/59367782/99933938-cf8b6080-2d9f-11eb-8f76-5aff029c994b.png)

- 여러 패턴이 존재
- I 프레임은 다른 프레임을 참조하지 않고 자기 자신만을 압축
- B 프레임은 I 프레임을 참조
- P 프레임은 B, I 프레임 참조
- 만약 I 프레임이 깨진 경우 전부 깨짐

## Streaming Stored Audio/Video

### Using a Web server

![image](https://user-images.githubusercontent.com/59367782/99935728-bafd9700-2da4-11eb-8ed2-5f99229eed37.png)

- Using a Web server with a metafile

![image](https://user-images.githubusercontent.com/59367782/99935749-c650c280-2da4-11eb-811b-5a811489f085.png)

- Using a media server

![image](https://user-images.githubusercontent.com/59367782/99935783-d9639280-2da4-11eb-828f-de7ab756b07d.png)

- Using a media server and RTSP

![image](https://user-images.githubusercontent.com/59367782/99935810-e97b7200-2da4-11eb-8428-f33790720275.png)

## Voice Over IP

- SIP(Session Initiation Protocol) : 멀티미디어 세션 제어 프로토콜
- H.323 : 시청각 통신 세션을 제공하는 프로토콜 (ex. 화상 회의)

### SIP Messages

- INVITE
- ACK
- BYE
- OPTIONS
- CANCLE
- REGISTER

**전화 -> SIP**

### SIP formats

- IPv4 address
- Email address
- Phone number

### H.323 architecture

![image](https://user-images.githubusercontent.com/59367782/99936015-6eff2200-2da5-11eb-8452-f76f91d76f41.png)

- 인터넷 상에서 화상 회의에 쓰이는 표준 프로토콜이다.

### H.323 protocols

![image](https://user-images.githubusercontent.com/59367782/99936020-758d9980-2da5-11eb-97fb-f4f87916b23d.png)

- 추가적으로 데이터 공유도 포함되어있다.

### example
![image](https://user-images.githubusercontent.com/59367782/99936025-77575d00-2da5-11eb-9767-98ef9a349e5e.png)


## 그림 출처

- McGraw-Hill 출판사 TCP/IP 프로토콜