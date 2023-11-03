---
title: "Kubernetes에 대하여"
date: 2023-11-03T21:09:36+09:00
tags:
  - kubernetes
  - container
categories:
  - kubernetes
published: false
---

# Kubernetes란?

최근 많은 기업들에서 운영하기 위해 채택하여 사용하고 있는 Kubernetes에 대해서 알아보자. 쿠버네티스, K8s라고 불리우며, 공식 웹사이트에서는 다음과 같이 설명하고 있다.

> Kubernetes, also known as K8s, is an open-source system for automating deployment, scaling, and management of containerized applications.

쿠버네티스는 오픈 소스로, 컨테이너화된 애플리케이션을 배포 및 관리하는 도구이다. 즉, Container Orchestration 도구 중 하나이다.  
Container Orchestration이 왜 필요한지에 대해서 짧게 정리하자면 다음과 같다.

- 많은 컨테이너의 버전 및 상태 관리
- CI/CD와의 통합
- 네트워크 및 스토리지 인터페이스
- 장애 대응

많은 Container Orchestration 도구가 있지만 그 중에서 구글 프로젝트를 기반으로 만들어진 오픈 소스인 쿠버네티스가 가장 많이 주목받고 사용되고 있다.

필자의 생각으로는 그 이유를 다음과 같이 짐작하고 있다.
- 클라우드 간의 이식성 보장
- 구글 기반 오픈소스 생태계의 성장
- 확장성
- 마스터 노드를 통한 중앙 제어
- 선언적 구성 기반

## Concept

[공식 문서](https://kubernetes.io/docs/concepts/)의 내용을 정리해서 작성하였다.

### Cluster Architecture

![image](https://github.com/lee20h/blog/assets/59367782/df83faa7-6e54-4197-aa5d-165f71a8c549)

#### Node

쿠버네티스 클러스터 내에서 워크로드를 실행하는 머신 (물리서버, VM)로, 각 노드는 쿠버네티스에 의해 관리되며 Kubelet, Kube Proxy, Container Runtime Interface (CRI)가 존재한다.

- Kubelet: 노드 상에서 컨테이너가 올바르게 실행되도록 관리하는 에이전트
- Kube Proxy: 네트워크 규칙을 관리하여 Pod 간의 통신을 가능하게 하는 컴포넌트
- CRI: 컨테이너를 실행하는 데 필요한 소프트웨어로, 클러스터 컴포넌트 kubelet과 컨테이너 런타임 사이의 통신을 위한 주요 프로토콜을 정의한다.
  - docker, containerd

#### Control Plane

쿠버네티스 클러스터를 관리하고 제어하는 컴포넌트의 집합으로, 여러 컴포넌트들을 가지고 있다.

- Kube API Server
- etcd
- Kube Controller Manager
- Kube Scheduler

### 통신 방법

- Node to Control Plane
  - hub-and-spoke API 패턴
  - 노드에서의 모든 API 사용은 API 서버에서 종료
  - Kubelet이 Kube API Server와 통신하여 노드에 배포할 Pod의 정보를 받아오고, 노드의 상태를 API 서버에 보고한다.
  - Kube Proxy는 서비스의 로드 밸런싱과 네트워크 라우팅을 관리하기 위해 API 서버와 통신한다.
- Control Plane to Node
  - API Server에서 클러스터의 각 노드에서 실행되는 kubelet 프로세스로의 경로
  - API 서버의 프록시 기능을 통한 API 서버에서 Node, Pod, Service의 경로
- Controle Plane 내부
  - Kube Api Server는 클러스터의 상태를 etcd에 읽고 쓰며, Kube Controller Manager와 Kube Scheduler도 API 서버를 통해 클러스터 상태를 관리한다.
- Konnectivity
  - SSH 터널을 대체하여 컨트롤 플레인에서 클러스터로의 통신을 위한 TCP 수준의 프록시 제공
  - 컨트롤 플레인 네트워크의 Konnectivity 서버와 노드 네트워크의 Konnectivity 에이전트로 구성
#### hub and spoke

네트워크, 통신, 물류 등 다양한 분야에서 사용되는 구조적인 패턴으로 중앙 집중식 노드인 허브와 여러 개의 분기점인 스포크로 이루어져있다.

특징으로는 다음과 같다.

- 중앙 집중식 구조
- 단순한 네트워크 구성
- 효율적인 관리

쿠버네티스에서의 hub and spoke는 컨트롤 플레인과 노드 간 통신에서 사용된다. **API 서버**가 hub 역할을 하며, 클러스터의 모든 관리와 제어가 이뤄진다.  
그리고 각 노드와 파드가 spoke 역할로 API 서버와의 통신을 통해 작업을 수행한다.

중앙 집중식 관리로 인해서 효율성 증가, 네트워크 구성의 단순화 같은 장점이 있지만, 허브에 과부하가 발생하거나 문제 발생 시 전체 시스템에 영향을 줄 수 있따는 단점이 있다.

#### Lease

분산 시스템에서 종종 공유 리소스를 잠그고 세트의 구성원 간 활동을 조정하는 매커니즘을 제공하는 Lease가 필요하다. 쿠버네티스에서 이 개념은 `coordination.k8s.io` API 그룹의 Lease 객체로 표현된다.
노드의 하트비트와 컴포넌트 수준의 리더 선출과 같은 시스템 중요 기능에 쓰인다.

- Node heartbeats
  - 쿠버네티스는 Lease API를 사용하여 kubelet 노드 하트비트를 쿠버네티스 API 서버에 전달
  - 모든 노드에 `kube-node-lease` 네임스페이스 내에 Lease 오브젝트가 존재
  - 쿠버네티스 컨트롤 플레인은 Lease 오브젝트의 renewTime 필드를 통해서 가용성을 확인
- Leader election
  - Lease를 사용하여 한 번에 하나의 컴포넌트 인스턴스가 실행되도록 보장
  - **kube-controller-manager**와 **kube-scheduler**와 같은 컨트롤 플레인 컴포넌트에서 HA 구성에 활용
- API server identity
  - v1.26부터 각 kube-apiserver는 Lease API를 사용하여 클라이언트에게 **kube-apiserver** 인스턴스 수를 알려줌
- Workloads
  - 컨트롤러 실행 혹은 리더를 선택하거나 선출하기 위해 Lease를 정의하여 사용
