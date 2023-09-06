---
title: "Transport layer에 대해 알아보자"
date: 2020-08-22
tags:
  - transport
categories:
  - network
published: true
---

# Transport Layer

**process-to-process** 한 컴퓨터의 프로세스와 다른 컴퓨터의 프로세스의 통신을 생각하면 좋다.  

Transport layer duties

- Packetizing
- Connection control
- Addressing
- Providing reilability

# Process-to-Process Delivery: UDP and TCP

- Node-to-node : Data link layer  
- Host-to-host : Network layer  
- Process-to-process : Transport layer

transport 계층은 16비트를 사용하므로 2^16개의 포트가 있다. 클라이언트는 16비트 중 아무 번호를 부여한다. 그 이후 데이터에 번호와 함께 서버의 기능에 바인딩된 포트로 보낸다.  

IP 주소 vs port 번호 : IP 주소는 호스트 네트워크 계층에서의 주소, port 번호는 트랜스포트 계층에서의 주소를 결정한다. 주어진 컴퓨터 내에서 여러 프로세스 중 특정 프로세스를 지정할 수 있는 번호이다.  
IP header에는 IP주소가 있으며, Transport-layer header에는 포트 번호가 들어있다.  

포트 번호의 경우에는 0 ~ 65535가 있으며, 0 ~ 1023의 포트 번호는 IANA에 의해서 정해진 포트번호가 있다. 1024 ~ 49151에는 국제적 공인은 아니지만 등록해서 사용할 수 있는 포트 범위이다. 서버들이 사용할 수 있다고 생각하면 된다. 나머지 49152 ~ 65535 부분은 클라이언트들이 사용한다고 생각하면 된다.  

## Socket

Socket이란 Berkeley socket implementation (1983) => Socket library  

넓은 의미로는 5개의 주소가 Socket 주소가 된다. (Source ip Source port, Destination IP, Destination port, protocol) 이 중 하나라도 다르면 서로 다른 Socket 주소이다.  

좁은 의미로는 ip 주소와 port 번호를 합쳐서 socket이라고도 하지만 거의 넓은 의미로 쓰인다.  

## Multiplexing and demultiplexing  

`Processes -> MUX -> IP -> IP -> DEMUX -> Processes`

## Connectionless vs Connection oriented service

UDP vs TCP로도 볼 수 있다.  

Connectionless는 다른 프로세스로 데이터를 보낼 때 연결을 하지 않고 데이터를 전송하는 것이다. 패킷이 소실될 수 있고, 순서가 뒤바뀔 수 있으며, 지연될 수 있는 문제가 있다. 이후에 도착지에서 acknowledgement를 보내주지 않는다. 대표적으로 UDP가 있다. 사용하는 이유는 프로토콜 오버헤드가 없으며, 마음대로 제어가 가능하다.  

Connection oriented service의 대표적인 것은 TCP가 있다. TCP는 3-way-handshake 방식을 가지고 있다. 연결을 할 때 연결 요청과 ack를 주고 받는 방식이다. 이것은 연결을 해제할 때도 똑같이 적용이 된다.  
TCP계층에서 Error Control을 담당한다. 단말은 5계층, 라우터는 3계층이 구현되어 있다. 인터넷 프로토콜은 라우터에 오류 제어 기능이 없으므로, 오류 제어는 **망이 아닌 단말들이 제어**한다. 이유는 인터넷이 핵 공격에 견디게 만들기 위해 망의 기능을 최소로 단순화시켰다. 따라서 오류제어, 혼잡제어 등과 같은 제어는 호스트 쪽에서 이뤄지도록 하였다.  

## UDP

UDP : Connectionless, unreliable protocol, no flow and error control  

UDP datagram format은 Header와 Data로 이루어져있다. Header의 경우 Source Port number, Detination port number, Total length, check sum 16bit씩 가지고 있다.  

UDP는 Flow and Error Control을 제공하지 않지만 UDP를 사용하는 애플리케이션에서 자체 제작하여 사용하면 된다. 멀티미디어 애플리케이션에서 많이 사용하는 방법이다. 또한 NFS (Network File System)에서도 UDP를 사용한다. NFS는 LAN으로 하드디스크들을 연결하는 것인데 사용자에게는 로컬 스토리지를 사용하듯이 사용할 수 있게 해주는 것이다. 이 NFS에서 Default가 UDP인 이유는 LAN 환경에서 패킷 오류가 거의 없고 빠르게 사용할 수 있다.  

과거엔 Multimedia 응용은 거의 UDP를 가정하고 사용했다. 인터넷이 활성화되지 않았으므로, 비디오 데이터는 양이 많아 TCP를 사용하기 쉽지 않았다. UDP가 빠르게 사용할 수 있어서 사용했지만, 최근에는 멀티미디어 응용에도 TCP를 사용한다. 그 이유는 다음과 같다.  
1. 사용자가 고품질 재생을 원한다.
2. 네트워크 대역폭과 품질이 대폭 향상 (Kbps -> Mbps)

클라이언트 버퍼링을 이용하면 패킷소실에 대한 재전송 시간이 있다. 만약 소실된 패킷이 도착하지 않으면 버퍼링을 기다리며 멈춰있다가 수신되면 재생한다.  

## TCP

TCP : Stream Delivery 모델로 기본 단위가 Byte이다. 또한, 순서가 있으며, Reliable transmission이다. 따라서 순서대로 도착하며, 소실시 재전송도 가능하다.  

Sending TCP와 Receiving TCP를 보면 Sender와 Receiver 버퍼를 유지하면서 송수신을 하고 완성되면 프로세스에게 전달한다. 추가적으로 버퍼는 원형큐를 사용한다.  

Sender 버퍼에 쌓인 바이트들을 Segment로 패키징해서 보낸다. Segment는 가변크기로, TCP 모듈에 의해서 크기가 정해져서 전송한다.  

TCP 커넥션에서 전송되는 바이트 수는 TCP에서 랜덤으로 넘버링된다. TCP segment 속에 sequence number의 값의 의미는 그 세그먼트가 포함된 바이트의 시작 위치를 의미한다.  

receiver가 segment를 받으면 acknowledgment를 모듈이 알아서 보내게 된다. acknowledgment의 숫자에 경우에는 누적이 된다.

TCP segment format은 Header와 Data로 이루어져있다. 표로 보자.  
Header  
| Source port address 16 bit | Destination port address 16 bits |
|---|---|
| Sequence number 32bits ||
| Acknowledgment number 32bits ||
| Hlen 4bit, Reserved 6bit, urg,ack,psh,rst,syn,fin | window size 16bits |
| checksum 16bits | urgent pointer 16bits |
| options and padding ||

해당 플래그들의 필드들을 알아보자.  
- URG : Urgent pointer is vaild  
- ACK : Acknowledgment is vaild
- PSH : Request for push
- RST : Reset the connection
- SYN : Synchronize sequence numbers
- FIN : Terminate the connection

예를 들어서 3-way-handshaking을 보면,  

1. Client -> Server Segment 1: SYN (seq: 1200, ack: -)
2. Server -> Client Segment 2: SYN 1 ACK (seq: 4800, ack: 1201)
3. Client -> Server Segment 3: ACK (seq: 1201, ack: 4801)

이어서 연결해제인 half close를 보면,

1. Client -> Server Segment 1: FIN (seq: 2500, ack: -)
2. Server -> Client Segment 2: ACK (seq: 7000, ack: 2501)
3. Server -> Client Segment 3: FIN (seq: 7001, ack: 2501)
4. Client -> Server Segment 4: ACK (seq: 2501, ack: 7002)

2번까지 완성되면 Client -> Server 방향 연결은 끊긴다. 이후에 3~4번이 끝나게 되면 Server -> Client 방향 연결도 끊긴다. 이것을 Half Close이라 한다.  

추가적으로 Simple telnet scenario를 보자.  

1. Host A -> Host B Seq=42, ACK=79, data='C'
2. Host B -> Host A Seq=79, ACK=43, data='C'
3. Host A -> Host B Seq=43, ACK=80

### **Flow Control**  

Sender와 Receiver의 1:1 통신 사이에 Receiver의 버퍼가 넘치지 않게 조절해주는 것이다. 대표적으로 Sliding window Protocol을 사용해서 넘치지 않게 조절해준다.  

Sender 버퍼에서의 슬라이딩 윈도우는 정해진 슬라이딩 윈도우 사이즈만큼만 보낼 수 있으며, ACK로 날라온 값만큼 Receiver가 받았다는 것을 알 수 있으므로, 슬라이딩 윈도우를 그만큼을 쉬프트시킨다. 이때 Sender 버퍼 사이즈가 12byte라면, Receiver 버퍼 사이즈도 12byte이상이 되어야한다. 그렇지 않으면 버퍼가 넘칠 수 있다.  

Receiver window도 Sender 버퍼와 같이 원형 큐로 유지하고 버퍼를 계속 채워가며, 시퀀스 넘버에 이어지지 않는 데이터의 경우 버리게 된다. 받은 만큼을 보내고 원형 큐와 같이 계속 데이터를 받아들인다.  

`Sender Window size <= Receiver Window size`  

Receiver window에 따라서 Sender window size가 바뀐다. 늘어나면 늘어나고, 줄어들면 줄어들게 되어있다. 넘치면 안되기 때문이다.  

실제 사이즈는 window size로 flow control 뿐아니라, congestion control도 하기 때문에 sender의 실제 window size는 receiver window size보다 작다.  

### **Error control**  

데이터 손실을 탐지하고 재전송하는 부분을 뜻한다.  

케이스를 보면
- 전송시 데이터 소실 : checksum
- 라우터가 혼잡 발생하여 세그먼트 소실 : Sequence number
- 세그먼트 순서 뒤바뀜 : Sequence number
- 중복된 세그먼트 : Sequence number

세그먼트 소실  
1. Segment1 (seq: 1201, 200bytes)
2. Segment2 (seq: 1401, 200bytes)
3. Segment3 (seq: 1601, 200bytes) 도중에 ack: 1601날라와서 Segment3 소실

이때 time-out하면 재전송

4. Segment3 (seq: 1601, 200bytes)
5. ack: 1801  

Acknowledgment 소실  
1. seq: 1201, 200bytes
2. seq: 1401, 200bytes
3. seq: 1601, 200bytes 도중에 ack: 1601 소실
4: ack: 1801 날라가다가 소실

보낸 segment 전부 time-out되어 1번부터 다시 보낸다.  

### TCP timer

`Timers = Retransmission + Persistence + Keep-alive + Time-waited`  

`RTT(Round-trip-time) = TCP seg + ACK`

재전송 시간은 RTT의 두배 정도로 잡는다. 또, RTT는 Exponential averaging 기법을 사용하여 `RTT = a(previous RTT) + (1-a)(current RTT)`로 잡는다. 이것은 이전의 RTT를 신뢰하는 정도와 현재 신뢰하는 정도를 나눠서 사용한다. 예를 들어 a가 0.7이라면 이전의 RTT를 70%, 현재의 RTT를 30%만큼 잡는다는 말이다.


## 그림 출처

- McGraw-Hill 출판사 TCP/IP 프로토콜