---
title: "가상화에 대해 알아보자"
date: 2023-10-02T20:59:38+09:00
tags:
  - cloud
  - virtualization
categories:
  - virtualization
series:
  - virtualization
published: true
---

# 가상화

> 가상화는 서버, 스토리지, 네트워크 및 기타 물리적 시스템에 대한 가상 표현을 생성하는 데 사용할 수 있는 기술입니다. 가상 소프트웨어는 물리적 하드웨어 기능을 모방하여 하나의 물리적 머신에서 여러 가상 시스템을 동시에 실행합니다. - 출처: [AWS](https://aws.amazon.com/ko/what-is/virtualization/) 

가상화는 인용한 말과 동일하게 단일한 물리 하드웨어 시스템에서 여러 시뮬레이션 환경이나 전용 리소스를 생성할 수 있는 기술을 뜻한다.  
예를 들어, **하이퍼바이저**(**hypervisor**)와 같은 소프트웨어가 하드웨어에 직접 연결되어 가상 머신(VM)이라는 별도의 환경으로 분리하는 기술을 말한다.

## 사용하는 이유

가상화가 없던 시절에는 하나의 컴퓨터에서 하나의 OS를 사용하여 운영하였다. 운영을 하다보니, 다음과 같은 문제점들이 생겨났다.

- 자원 사용량이 예측이 어려운 관계로, 서버 자원을 효율적으로 배치하기 어렵다.
- 남는 리소스들은 전부 idle 상태로 낭비된다.
- 인프라 확장 및 유지보수에 있어 하나의 OS에 종속된다.

이러한 문제점들을 하이퍼바이저라는 소프트웨어를 통해서 해결할 수 있었다. 하이퍼바이저는 가상 환경으로부터 물리 리소스를 분리해준다.

## 가상화 종류

가상화 종류에는 Host OS 가상화, 하이퍼바이저 가상화와 컨테이너 가상화가 있다.

### Host OS 가상화

<table border="1">
    <tr>
        <th><center>가상환경</center></th>
        <th><center>가상환경</center></th>
        <th><center>가상환경</center></th>
    </tr>
    <tr>
        <td><center>애플리케이션</center></td>
        <td><center>애플리케이션</center></td>
        <td><center>애플리케이션</center></td>
    </tr>
    <tr>
        <td><center>미들웨어</center></td>
        <td><center>미들웨어</center></td>
        <td><center>미들웨어</center></td>
    </tr>
    <tr>
        <td><center>Guest OS</center></td>
        <td><center>Guest OS</center></td>
        <td><center>Guest OS</center></td>
    </tr>
    <tr>
        <td colspan="3"><center>가상화 소프트웨어</center></td>
    </tr>
    <tr>
        <td colspan="3"><center>Host OS</center></td>
    </tr>
    <tr>
        <td colspan="3"><center>Hardware</center></td>
    </tr>
</table>

물리 머신에서 동작하는 OS를 Host OS라고 하며, Host OS 위에서 가상 머신에 설치된 OS를 Guest OS라고 한다.  

Host OS와 Guest OS 간의 큰 제약사항이 없으나, OS를 통으로 설치하다보니 크기가 매우 클 수 있다. 따라서 가상화를 구현하는데는 큰 문제가 없으나, 유지보수에 있어서 오버헤드가 크다.

### 하이퍼바이저 가상화

<table border="1">
    <tr>
        <th><center>가상환경</center></th>
        <th><center>가상환경</center></th>
        <th><center>가상환경</center></th>
    </tr>
    <tr>
        <td><center>애플리케이션</center></td>
        <td><center>애플리케이션</center></td>
        <td><center>애플리케이션</center></td>
    </tr>
    <tr>
        <td><center>미들웨어</center></td>
        <td><center>미들웨어</center></td>
        <td><center>미들웨어</center></td>
    </tr>
    <tr>
        <td><center>OS</center></td>
        <td><center>OS</center></td>
        <td><center>OS</center></td>
    </tr>
    <tr>
        <td colspan="3"><center>하이퍼바이저</center></td>
    </tr>
    <tr>
        <td colspan="3"><center>Hardware</center></td>
    </tr>
</table>

따로 Host OS가 없으므로, 위에서 언급한 오버헤드가 없다. 또한, 하드웨어와 하이퍼바이저가 직접적으로 연결되어 있어서, 하드웨어를 직접 제어할 수 있다.  
하지만, 직접 제어하는 만큼 관리 기능이 없기 때문에 따로 직접 콘솔이나 자동화된 컴퓨터를 통해 관리를 해줘야한다.

### 컨테이너 가상화

<table border="1">
    <tr>
        <th><center>가상환경</center></th>
        <th><center>가상환경</center></th>
        <th><center>가상환경</center></th>
    </tr>
    <tr>
        <td><center>애플리케이션</center></td>
        <td><center>애플리케이션</center></td>
        <td><center>애플리케이션</center></td>
    </tr>
    <tr>
        <td><center>미들웨어</center></td>
        <td><center>미들웨어</center></td>
        <td><center>미들웨어</center></td>
    </tr>
    <tr>
        <td colspan="3"><center>Container Runtime</center></td>
    </tr>
    <tr>
        <td colspan="3"><center>OS</center></td>
    </tr>
    <tr>
        <td colspan="3"><center>Hardware</center></td>
    </tr>
</table>

OS에 컨테이너 관리 소프트웨어를 설치하여, 컨테이너 단위로 나눠서 가상화한다. 따라서 한 OS를 공유한다.  
컨테이너 단위로 나누다보니, 단일 애플리케이션을 위한 최소한의 환경만 담아서 속도가 빠르며, 컨테이너에서 실행되는 이미지의 생성과 공유가 쉽다.  
하지만 가상머신에서처럼 OS가 나뉘어지지 않아서, 보안적으로 완전히 격리가 되지 않으며, 다양한 OS 커널을 사용할 수 없다.

Docker를 사용하는 경우에 이미지에 OS가 포함되는 경우가 있다. 여기서 이미지에 들어있는 OS는 운영체제 전체를 의미하지 않는다. 기본적으로 호스트 시스템의 커널을 공유하기 때문에, 컨테이너 내의 OS 이미지는 커널을 제외한 사용자 공간(User Space) 구성요소만 포함한다.  
따라서, 이미지에는 애플리케이션과 의존성, 라이브러리, 런타임 등을 포함하여, 애플리케이션을 실행하기 위해 필요한 모든 파일 시스템의 스냅샷을 가지고 있다.  
이런 방식이 컨테이너를 경량하게 만들며, 부팅 시간을 빠르게 해준다.

컨테이너와 가상머신에 대해서는 다음 포스팅에서 더 깊게 다룰 예정이다.

## 하이퍼바이저

하이퍼바이저는 이전에 하나의 컴퓨터에서 하나의 OS만 운영이 가능하던 방식을 해결해주는 소프트웨어이다. 다시 말하면, 하이퍼바이저는 컴퓨터의 운영 체제 및 응용프로그램을 물리적 하드웨어에서 분리해주는 기술이다.  

구성 방식은 아래의 그림과 같이 이루어져있다.

<figure>
  <img src="https://github.com/lee20h/blog/assets/59367782/afe0b2c4-557a-4119-95f8-baa183ea0b8a" width="400" height="300"/>
  <figcaption>
    Hypervisor (<a href="https://www.redhat.com/ko/topics/virtualization/what-is-virtualization">출처</a>)
  </figcaption>
</figure>

1. 하드웨어에서 제공되는 물리적인 레이어를 추상화
2. VM을 통해 가상화된 물리 자원들을 사용
3. 이 때, VM들은 완전히 격리된 환경

모든 하이퍼바이저에서 VM을 실행하려면 메모리 관리 프로그램, 프로세스 스케줄러, I/O(입력/출력) 스택, 기기 드라이버, 보안 관리 프로그램, 네트워크 스택과 같은 운영 체제 수준의 구성 요소가 필요하다. 즉, 하드웨어들을 전체적으로 관장하며, VM들도 관리하는 중간 관리자 역할을 한다. 따라서 가상머신 모니터(Virutal Machine Monitor)라고 부르기도 한다.

할당된 리소스를 각 가상머신에 제공하고, 물리 리소스에 대해 VM 리소스 일정 관리를 한다. 하드웨어는 계속 작업을 수행하므로, 하이퍼바이저가 일정 관리하는 동안 CPU가 VM에서의 요청에 맞춰서 명령을 실행한다.
이러한 특징들을 이용하여 다른 여러 운영 체제를 한번에 구동할 수 있으며, 하이퍼바이저를 통해 가상화된 하나의 하드웨어 리소스를 공유한다.

### 하이퍼바이저 종류

하이퍼바이저 종류가 유형 1, 2 두 개로 나뉘어진다.

1. 유형 1
    - native, bare metal로 불리는 유형 1
    - 하드웨어에서 직접 실행
    - VM 리소스는 하이퍼바이저에 의해 하드웨어에 예약
    - KVM, Microsoft Hyper-V, VMware vSphere
2. 유형 2
    - Host 하이퍼바이저라고 불리는 유형 2
    - 기존 OS가 설치된 하드웨어에서 소프트웨어 레이어 또는 애플리케이션으로 구동
    - VM 리소스는 Host OS에서 예약하여 하드웨어에서 실행
    - 단일 머신에서 여러 OS를 실행하는 경우
    - VMware Workstation, Oracle VirtualBox

## 가상화 유형

- 데이터 가상화
  - 분산된 데이터들을 단일 소스로 통합
- 데스크탑 가상화
  - 중앙 관리자나 자동화된 관리 툴을 통해 모든 가상 데스크탑의 구성, 업데이트, 보안 점검을 수행
- 서버 가상화
- 스토리지 가상화
- 네트워크 가상화
- 운영체제 가상화
  - OS Kernel에서 다른 운영체제들을 실행
- 네트워크 기능 가상화
  - 네트워크의 주요 기능을 분리하여 환경에 배포
  - 특정 기능을 물리 머신으로부터 분리하여 새 네트워크에 함께 패키징하고 할당
  - 스위치, 라우터, 서버, 케이블, 허브 등 여러 개의 독립적인 네트워크 생성
- 애플리케이션 가상화
  - 애플리케이션 스트리밍: 최종 사용자의 디바이스에서 필요할 때만 실행
  - 서버 기반 애플리케이션 가상화: 사용자는 브라우저 또는 클라이언트 인터페이스에서 설치 없이 원격 애플리케이션에 액세스
  - 로컬 애플리케이션 가상화: 애플리케이션 코드가 자체 환경에 포함되어 출고되므로 변경 없이 모든 운영 체제에서 실행

## TL;DR

하드웨어에 종속된 부분들을 가상화를 통해서 여러 가상머신들이 사용할 수 있게 제공하며, 다음과 같은 특징을 통해 운영에 이점을 가져간다.

- Server Consolidation
  - 효율적인 리소스 사용
- Isolation
  - 각 가상 머신들은 독립적으로 실행
  - 자동화 관리 및 재해 복구

나중에 포스팅하겠지만, 가상화는 클라우드 컴퓨팅 기술을 가능하게하는 주요 기술 중 하나이다.
   
## ref

- [redhat](https://www.redhat.com/ko/topics/virtualization/what-is-virtualization)
- [aws](https://aws.amazon.com/ko/what-is/virtualization/)
- [소프트웨어 정책연구소](https://spri.kr/files/1544421791_k7LnlkLc9mgJgots3jU7H3BR_0.pdf)
- [NDS](https://tech.cloud.nongshim.co.kr/2018/09/18/%EA%B0%80%EC%83%81%ED%99%94%EC%9D%98-%EC%A2%85%EB%A5%983%EA%B0%80%EC%A7%80/)