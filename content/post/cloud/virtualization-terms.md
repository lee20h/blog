---
title: "가상화 관련 용어에 대해 알아보자"
date: 2023-10-14T00:00:00+09:00
tags:
  - cloud
  - virtualization
categories:
  - virtualization
series:
  - virtualization
published: true
---

## On-premise / Off-premise

On-premise: 사내(자체 데이터센터나 전산실)에 서버를 직접 설치하여 사용하는 환경  
Off-premise: 클라우드 서비스 환경을 이용한 방식

여기서 off-premise라는 용어는 거의 사용되지 않으며, 클라우드 환경 혹은 클라우드 서비스 환경이라고 많이 일컫는다.

## Bare metal

아무런 소프트웨어가 설치되지 않은 상태의 컴퓨터를 일컫는 말로, OS를 설치해야 서버로 사용할 수 있다.  
클라우드에서는 Bare Metal Server를 이야기하면, 가상화를 위한 하이퍼바이저 OS 없이 물리 서버를 그대로 제공해주는 경우를 말한다.
즉, 하드웨어를 그대로 임대한다는 의미로 사용한다.

## Virtual Machine(VM)

물리적 하드웨어 시스템에 구축되서 자체 CPU, Memory, Storage, Network interface 등 리소스를 갖추고 가상 컴퓨터 시스템으로 작동하는 가상 환경을 일컫는다.  
가상 머신 내 동작되는 소프트웨어들은 가상 환경인지 인지하지 못하며, 실제 물리 서버처럼 리소스들을 사용한다. 또한, 하나의 물리적인 서버 내에 여러 가상머신이 다른 OS와 애플리케이션을 실행할 수 있다.

## 하이퍼바이저

하이퍼바이저에 대해 이 [포스팅](https://20h.dev/post/cloud/virtualization/)에서 설명하였다.

## vCPU

Virtual CPU로, 클라우드 환경에서의 가상의 CPU 리소스를 뜻한다. 여기에서 vCPU는 물리적인 Core를 기반을 하며, 가상화 기술을 통해 물리적인 CPU Core 하나를 여러 vCPU로 나눠서 사용할 수 있다.  
이렇게 나뉘어진 vCPU를 가상 머신이나 컨테이너에 할당하여서 리소스를 제공한다.  

vCPU 생성과 관리는 CPU 스케줄링에 큰 영향을 받기 때문에, 하이퍼바이저가 하드웨어와 직접적으로 연결되어 있기 때문에, 물리적 CPU를 vCPU로 분배하여 생성하고 관리하는 일은 하이퍼바이저가 한다.

인텔 CPU를 예를 들면, CPU 16 Core인 하드웨어가 있을 때, 클라우드 환경에서 이 서버를 가상화하여 여러 가상 머신에 제공한다고 한다. 하이퍼 스레딩 (Hyper-Threading[^hyper-threading])를 통해 Core 당 논리 Core를 2개씩 가지게 된다.  
가상화 환경에서는 이 논리 Core를 오버 커밋 (Over Commit[^over-commit])으로 더 많은 가상 Core로 쪼개서 사용할 수 있다. 이 부분을 스케줄링을 담당하는 하이퍼바이저가 결정하게 된다. 이렇게 만들어진 vCPU 또한, 하이퍼바이저가 가상 머신이나 컨테이너에 리소스로 할당하게 된다.


vCPU 1개가 의미하는 바는 물리적 CPU Core 1개 동일하지 않으며, 물리적 CPU 연산 능력을 시간 단위로 분할하여 사용할 수 있는 리소스를 의미한다. 즉, 물리적인 CPU Core를 사용할 수 있는 시간 비율을 의미한다.

## 전가상화 (Full Virtualization)

하드웨어를 완전히 가상화하는 방식으로, Hardware Virtual Machine 이라고도 한다. 가상 머신들은 전부 물리 자원에 연결되어 있다고 생각한다.  

그러다보니, 가상 머신들의 **System call**은 전부 하드웨어가 아닌 **하이퍼바이저**가 처리해서 보내주게 된다.

<figure>
  <img src="https://github.com/lee20h/blog/assets/59367782/41ac69cd-7524-46b1-83dc-90f9ec87ec13"/>
  <figcaption>
    Full Virtualization (<a href="https://techdifferences.com/difference-between-full-virtualization-and-paravirtualization.html">출처</a>)
  </figcaption>
</figure>

### 구현 방법

- 하드웨어 자원 가상화
- 소프트웨어 적으로 구현

두 가지 방법으로 구현할 수 있으며, 요즘에는 `전가상화 == 하드웨어 자원 가상화`로 쓰이고 있다.

먼저 `Dual-mode operation(이중 동작 모드)`에 대해서 알고 넘어가자.

<figure>
  <img src="https://github.com/lee20h/blog/assets/59367782/23f8bccc-84b4-4c61-9ac6-e29262e80f0f"/>
  <figcaption>
    Dual-mode operation (출처: operation system concept)
  </figcaption>
</figure>

사용자와 OS는 시스템 자원을 공유하는데 사용자가 제한없이 사용하게 되면 메모리 내의 주요 자원들을 망가뜨릴 수 있다. 이러한 부분을 보호하는 장치이다.

- 사용자 모드
- 커널 모드

두 가지 모드로 동작하게 되며, 애플리케이션은 사용자 모드에서 작동되다가 System Call을 OS에게 요청한 경우, 커널 모드로 전환되어 요청된 서비스를 실행시킨 뒤 다시 사용자 모드로 전환되게 된다.

#### Trap & Emulation

<figure>
  <img src="https://github.com/lee20h/blog/assets/59367782/be474a24-0fe3-49e6-a6f2-ecaf36bbb3b4"/>
  <figcaption>
    Trap & Emulation (<a href="https://suyeon96.tistory.com/53#Full%--Virtualization%---%EC%A-%--%EA%B-%--%EC%--%--%ED%--%---">출처</a>)
  </figcaption>
</figure>

하이퍼바이저는 Root mode, 논리화된 도메인들은 Non-root mode로 동작된다. 여기서 도메인들은 **privileged command**를 실행하면 **Trap**이 발생하고 Trap Handler가 vm exit 명령을 통해 하이퍼바이저가 실행할 수 있게 한다.  
실행이 완료되면 vm enter 명령을 통해 다시 도메인이 실행되도록 하여 하드웨어에 대한 명령을 내릴 수 있는 방식이다. 따라서 하이퍼바이저와 커널의 처리 방식에서 오버헤드가 발생하여 성능 저하가 일어난다.

#### Binary Translation

<figure>
  <img src="https://github.com/lee20h/blog/assets/59367782/7308d85c-6ebc-41b2-8584-bc3c1bbc035c"/>
  <figcaption>
    Binary Translation (<a href="https://suyeon96.tistory.com/53#Full%--Virtualization%---%EC%A-%--%EA%B-%--%EC%--%--%ED%--%---">출처</a>)
  </figcaption>
</figure>

논리화된 도메인에서 Guest OS는 다양한 OS를 사용할 수 있다. 이로 인해서, 가상화된 하드웨어에 요청 인터페이스도 OS마다 다르게 되는데 하나의 형식으로 변환해주는 작업을 **Binary Translation**라고 한다. 


### Software Assisted – Full Virtualization  (BT – Binary Translation )

<figure>
  <img src="https://github.com/lee20h/blog/assets/59367782/5e5850b7-acb8-49d1-8823-117ef178e48f"/>
  <figcaption>
    Binary Translation (<a href="https://www.unixarena.com/2017/12/para-virtualization-full-virtualization-hardware-assisted-virtualization.html">출처</a>)
  </figcaption>
</figure>

소프트웨어 지원 전가상화를 한 경우를 살펴보자. Binary Translation을 이용하여 명령어를 trap과 가상화하여 실행하게 된다. 위의 그림과 동일하게 하드웨어를 emulate하여 명령을 내릴 수 있게 된다. 따라서 오버헤드에 의한 성능 문제가 발생한다.

- VMware 워크스테이션 (32bit guest OS)
- Virtual PC
- VirtualBox (32bit guest OS)
- VMware 서버

### Hardware-Assisted –  Full Virtualization  (VT)

<figure>
  <img src="https://github.com/lee20h/blog/assets/59367782/b7838ee0-79bd-47da-877d-4e0ba8e38db4"/>
  <figcaption>
    Hardware-assisted virtualization (<a href="https://www.unixarena.com/2017/12/para-virtualization-full-virtualization-hardware-assisted-virtualization.html">출처</a>)
  </figcaption>
</figure>

하드웨어 지원 가상화는 Binary Translation을 제거하고 가상화하여 직접 하드웨어와 통신할 수 있게 한다. 그를 이용해, 유저 애플리케이션와 Guest OS에서 trap과 emulate를 이용하여 **privileged command**를 실행 할 수 있게 된다.

## 반가상화 (Para Virtualization)

전가상화의 경우 가상화 단계에서 오버헤드가 발생하여 성능 저하가 발생한다. 이 부분을 해결하기 위해 반가상화가 등장하게 되었다. 여기서의 핵심 키워드는 `Hyper Call`이다.

<figure>
  <img src="https://github.com/lee20h/blog/assets/59367782/4ef08ecc-f451-4328-a746-80d074d1e789"/>
  <figcaption>
    Para Virtualization (<a href="https://techdifferences.com/difference-between-full-virtualization-and-paravirtualization.html">출처</a>)
  </figcaption>
</figure>

Guest OS에서 하이퍼바이저에게 직접 요청을 날리는 부분을 **Hyper Call**이라는 인터페이스라고 하며, OS에서 애플리케이션이 커널에게 System Call을 호출하여 하드웨어를 사용하는 방법과 동일하다.  
요청하는 대상이 Guest OS, 요청받는 대상이 하이퍼바이저라는 점이 다르다. 또한, Guest OS가 하이퍼바이저에게 요청을 하려면 본인이 Guest OS임을 인지해야하므로, 커널을 수정해서 Guest OS를 따로 제작해야한다.

그렇다보니, 반가상화에서는 모든 명령어를 가상화할 필요 없으며, 필요한 명령어만 가상화를 할 수 있다.  
이러한 부분 덕분에 전가상화에 비해 성능이 좋다고 할 수 있다. 여기서 필요한 명령어를 구분하기 위해서 커널의 수정이 필요하며, Guest OS가 하이퍼바이저에게 명령어를 가상화해달라고 하는 요청을 **Hyper Call**이라고 한다.


## ref

- [Rain.i](http://cloudrain21.com/terms-about-virtualization)
- [techDifferences](https://techdifferences.com/difference-between-full-virtualization-and-paravirtualization.html)
- [Suyeon's Blog](https://suyeon96.tistory.com/53)
- [UnixArena](https://www.unixarena.com/2017/12/para-virtualization-full-virtualization-hardware-assisted-virtualization.html)

[^hyper-threading]: Intel이 개발한 기술로, 한 개의 물리적 CPU 코어를 두 개의 논리적 CPU 코어로 분할하여 동시에 두 개의 쓰레드를 처리할 수 있도록 한다. 이 기술은 즉, 물리적인 CPU 코어가 동시에 두 개의 작업을 처리하는 것처럼 운영 체제와 응용 프로그램에 보이게 한다.
[^over-commit]: 하이퍼바이저가 물리적 CPU Core 보다 더 많은 vCPU를 할당하는 경우. 여러 가상 머신이나, 컨테이너가 CPU 리소스를 동시에 100% 사용하지 않는다는 가정하에, 리소스를 효과적으로 활용하는데 도움을 줌
