---
title: "분산 컴퓨팅에 대해 알아보자"
date: 2023-10-15T22:36:59+09:00
tags:
  - cloud
  - distrubuted computing
categories:
  - cloud
series:
  - cloud
publishResources: true
---

# 분산 컴퓨팅이란?

다수의 컴퓨터가 공통 문제를 해결하기 위해 협업하도록 만드는 방법으로, 컴퓨터 네트워크는 복잡한 과제를 처리하기 위한 대량의 리소스를 제공하는 하나의 거대한 컴퓨터와 같이 동작한다.
분산 시스템 혹은 분산 데이터베이스라고도 하며, 별도의 노드가 공동의 네트워크를 통해 통신하고 동기화하게 된다. 노드는 일반적으로 별도의 하드웨어 장치를 의미하지만 소프트웨어 프로세스나 캡슐화 시스템을 나타낼 수도 있다.  

## 특징

- 리소스 공유
- 동시 처리
- 확장성
- 오류 감지

## 분산 컴퓨팅 아키텍처

- 클라이언트-서버
- N계층 아키텍처(N-tier)
- P2P 아키텍처
- 서비스 지향 아키텍처(SOA)
- 마이크로 서비스 아키텍처(MSA)

## 사례

- [Kubernetes](https://kubernetes.io/)
- [Ethereum](https://ethereum.org/ko/)

## 병렬 컴퓨팅, 그리드 컴퓨팅

### 병렬 컴퓨팅이란?

하나의 컴퓨터 또는 네트워크 내의 여러 컴퓨터가 동시에 많은 수의 계산 또는 프로세스를 수행하는 컴퓨팅 방법이다. 병렬 컴퓨팅 자체는 분산 컴퓨팅의 한 형태로, 모든 프로세서가 공유 메모리에 접근해서 정보를 교환한다.  

하지만, 분산 처리에서는 각 프로세서마다 분산 메모리를 사용하고, 프로세서끼리 메시지 전달로 정보를 교환한다. 

<figure>
  <img src="https://github.com/lee20h/blog/assets/59367782/0a6fae9a-534d-4b18-b66a-dda5802fb3f7"/>
  <figcaption>
    (a), (b): 분산 시스템 (c): 병렬 시스템 (<a href="https://ko.wikipedia.org/wiki/%EB%B6%84%EC%82%B0_%EC%BB%B4%ED%93%A8%ED%8C%85">출처</a>)
  </figcaption>
</figure>

### 그리드 컴퓨팅이란?

지리적으로 분산된 여러 컴퓨터 네트워크가 함께 작동하면서 공통의 작업을 수행하는 컴퓨팅 방법이다. 결국엔 분산 컴퓨팅의 한 방법으로, 여러 네트워크 간의 연결을 강조한다. 각 그리드 네트워크에서 개별 기능 들을 수행하고 결과를 다른 그리드에 전달하는 방식으로 진행된다.


## Ref

- [AWS: grid-computing](https://aws.amazon.com/ko/what-is/grid-computing/)
- [AWS: distributed-computing](https://aws.amazon.com/ko/what-is/distributed-computing/)
- [Azure: grid-computing](https://azure.microsoft.com/ko-kr/resources/cloud-computing-dictionary/what-is-grid-computing/)