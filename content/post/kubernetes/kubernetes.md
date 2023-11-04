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

# Concept

[공식 문서](https://kubernetes.io/docs/concepts/)의 내용을 정리해서 작성하였다.

## Cluster Architecture

![image](https://github.com/lee20h/blog/assets/59367782/df83faa7-6e54-4197-aa5d-165f71a8c549)

### Node

쿠버네티스 클러스터 내에서 워크로드를 실행하는 머신 (물리서버, VM)로, 각 노드는 쿠버네티스에 의해 관리되며 Kubelet, Kube Proxy, Container Runtime Interface (CRI)가 존재한다.

- Kubelet: 노드 상에서 컨테이너가 올바르게 실행되도록 관리하는 에이전트
- Kube Proxy: 네트워크 규칙을 관리하여 Pod 간의 통신을 가능하게 하는 컴포넌트
- CRI: 컨테이너를 실행하는 데 필요한 소프트웨어로, 클러스터 컴포넌트 kubelet과 컨테이너 런타임 사이의 통신을 위한 주요 프로토콜을 정의한다.
  - docker, containerd

### Control Plane

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
### hub and spoke

네트워크, 통신, 물류 등 다양한 분야에서 사용되는 구조적인 패턴으로 중앙 집중식 노드인 허브와 여러 개의 분기점인 스포크로 이루어져있다.

특징으로는 다음과 같다.

- 중앙 집중식 구조
- 단순한 네트워크 구성
- 효율적인 관리

쿠버네티스에서의 hub and spoke는 컨트롤 플레인과 노드 간 통신에서 사용된다. **API 서버**가 hub 역할을 하며, 클러스터의 모든 관리와 제어가 이뤄진다.  
그리고 각 노드와 파드가 spoke 역할로 API 서버와의 통신을 통해 작업을 수행한다.

중앙 집중식 관리로 인해서 효율성 증가, 네트워크 구성의 단순화 같은 장점이 있지만, 허브에 과부하가 발생하거나 문제 발생 시 전체 시스템에 영향을 줄 수 있따는 단점이 있다.

### Lease

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

### Cloud Controller Manager

![image](https://github.com/lee20h/blog/assets/59367782/f2ec5fe0-3821-4fcb-8caa-3725c5ebfea3)

클라우드 컨트롤 매니저는 쿠버네티스의 컨트롤 플레인 구성 요소 중 하나로, 클라우드 특정 제어 로직이 내장되어 있다. 이 컴포넌트는 사용자의 클러스터를 클라우드 제공자의 API와 연결하고, 클라우드 플랫폼과 상호 작용하는 컴포넌트와 클러스터 내에서만 상호 작용하는 컴포넌트를 분리한다.  

- Node controller: 새로운 서버가 클라우드 인프라에 생성되면 Node 객체를 업데이트한다.
  - **v1/Node**: get, list, create, update, patch, watch, delete 권한이 필요
- Route controller: 쿠버네티스 클러스터 내의 다른 노드에 있는 컨테이너들이 서로 통신할 수 있도록 적절한 라우트를 구성한다.
  - **v1/Node**: get 권한이 필요
- Service controller: 서비스 리소스를 선언할 때 필요한 로드 밸런서와 기타 인프라 구성 요소를 설정하기 위해 클라우드 제공자의 API와 상호 작용한다.
  - **v1/Service**: list, get, watch, patch, update 권한이 필요합
  - **v1/Endpoints**: create, get, list, watch, update 권한이 필요
- Cloud Controller Manager
  - **v1/Event**: create, patch, update 권한이 필요
  - **v1/ServiceAccount**: create 권한이 필요

### Control Groups (cgroups)

쿠버네티스에서 cgroups (control groups)는 리눅스에서 프로세스에 할당된 리소스를 제한하는 역할을 한다. 쿠버네티스의 kubelet과 컨테이너 런타임은 cgroups를 사용하여 파드와 컨테이너의 리소스 관리를 수행한다.  
이는 CPU 및 메모리 요청과 제한을 포함한 컨테이너화된 작업에 대한 리소스 관리를 의미한다.

리눅스에는 cgroup v1과 cgroup v2 두 가지 버전의 cgroups가 있다. cgroup v2는 cgroup API의 새로운 세대로, 통합된 제어 시스템과 향상된 리소스 관리 기능을 제공한다. cgroup v2는 cgroup v1에 비해 여러 개선 사항을 제공한다.

- 단일 통합 계층 디자인
- 컨테이너에 대한 안전한 하위 트리 위임
- Pressure Stall Information과 같은 새로운 기능
- 여러 리소스에 걸친 향상된 리소스 할당 관리 및 격리
- 네트워크 메모리, 커널 메모리 등의 다양한 유형의 메모리 할당에 대한 통합 회계

쿠버네티스의 일부 기능은 cgroup v2를 전용으로 사용하여 리소스 관리와 격리를 향상시킨다. 예를 들어, MemoryQoS 기능은 메모리 QoS를 향상시키며 cgroup v2 기본 요소에 의존한다.  
cgroup v2를 사용하려면 리눅스 배포판이 cgroup v2를 기본적으로 활성화하고 사용하도록 설정되어 있어야 한다. 몇몇 배포판(ubuntu, debian, fedora, ...)들은 cgroup v2를 기본적으로 사용하고 있다.  
cgroup v2로 마이그레이션하려면 요구 사항을 충족시키고 cgroup v2를 기본적으로 활성화하는 커널 버전으로 업그레이드해야 한다. kubelet은 OS가 cgroup v2에서 실행되고 있는지 자동으로 감지하고 적절하게 수행한다.

### Container Runtime Interface

쿠버네티스의 **컨테이너 런타임 인터페이스**(CRI)는 kubelet이 다양한 컨테이너 런타임을 사용할 수 있도록 하는 플러그인 인터페이스로, 이를 통해 클러스터 컴포넌트를 다시 컴파일할 필요 없이 컨테이너 런타임과 통신할 수 있다.  
CRI는 kubelet과 컨테이너 런타임 간의 주요 gRPC 프로토콜을 정의한다. 각 노드에는 kubelet이 파드와 그 안의 컨테이너를 시작할 수 있도록 작동하는 컨테이너 런타임이 필요하다.

- API Version: Kubernetes v1.23에서는 CRI가 안정적인 상태로 제공된다. kubelet은 클라이언트로서 gRPC를 통해 컨테이너 런타임에 연결한다.
- CRI Version 선택: kubelet은 컴포넌트의 재시작 시 자동으로 최신 CRI 버전을 선택하려고 시도한다. 만약 실패하면, 위에서 언급한 대로 fallback이 발생한다.
- 업그레이드 시 주의사항: 컨테이너 런타임이 업그레이드되어 gRPC 재연결이 필요한 경우, 컨테이너 런타임은 초기에 선택된 버전도 지원해야 한다. 그렇지 않으면 재연결이 실패할 가능성이 있다.

CRI를 통해 쿠버네티스는 다양한 컨테이너 런타임과의 호환성을 확보하여, 운영자는 원하는 컨테이너 런타임을 사용할 수 있다.

### Garbage Collection

쿠버네티스에서 **가비지 컬렉션**은 클러스터 리소스를 정리하는 데 사용되는 여러 메커니즘을 총칭한다. 가비지 컬렉션을 통해 다음과 같은 리소스를 정리할 수 있다.  

- 종료된 파드(Terminated pods)
- 완료된 작업(Completed Jobs)
- 소유자 참조가 없는 객체(Objects without owner references)
- 사용되지 않는 컨테이너와 컨테이너 이미지(Unused containers and container images)
- StorageClass의 회수 정책이 Delete인 동적 프로비저닝된 PersistentVolumes
- 만료된 CertificateSigningRequests (CSRs)
- 특정 시나리오에서 삭제된 노드(Nodes)
- Node Lease 객체

**가비지 컬렉션의 종류**

- Foreground Cascading Deletion: 삭제 중인 소유자 객체가 먼저 '삭제 진행 중' 상태로 들어간다. 이 상태에서 종속된 객체들이 삭제되고, 모든 종속된 객체가 삭제된 후에 소유자 객체가 삭제된다.
- Background Cascading Deletion: 쿠버네티스 API 서버는 소유자 객체를 즉시 삭제하고, 컨트롤러가 배경에서 종속된 객체를 정리한다.
- Orphaned Dependents: 소유자 객체가 삭제되면 남겨진 종속된 객체들을 '고아 객체'라고 한다.

**컨테이너 및 이미지의 가비지 컬렉션**: kubelet은 사용되지 않는 이미지에 대해 5분마다, 사용되지 않는 컨테이너에 대해 1분마다 가비지 컬렉션을 수행한다. 사용자는 외부 가비지 컬렉션 도구를 사용하지 않아야 하며, 필요한 컨테이너를 삭제하지 않도록 주의해야 한다.

### Mixed Version Proxy (1.28 alpha)

API 서버가 리소스 요청을 동일한 클러스터 내의 다른 버전의 쿠버네티스를 실행하는 동료 API 서버로 프록시할 수 있다. 이는 클러스터 내에서 쿠버네티스의 새 릴리스로의 장기간 롤아웃과 같은 상황에서 유용하다.

특징으로는 다음과 같다.

- 클러스터 관리자는 업그레이드 중에 리소스 요청을 올바른 kube-apiserver로 전달함으로써, 더 안전하게 업그레이드 가능한 고가용성 클러스터를 구성할 수 있다.
- 프록시를 통해 사용자가 업그레이드 과정에서 예상치 못한 404 Not Found 오류를 보지 않도록 한다.

**혼합 버전 프록시 활성화 방법**

- API 서버를 시작할 때 `UnknownVersionInteroperabilityProxy` 기능 게이트를 활성화해야 한다.
- API 서버 간의 프록시 전송 및 인증을 구성해야 한다.

**작동 방식**

- API 서버가 리소스 요청을 받으면 어떤 API 서버가 해당 리소스를 제공할 수 있는지 확인한다.
- 요청된 리소스에 대한 내부 StorageVersion 객체가 없고 APIService가 확장 API 서버로의 프록시를 지정하면, 확장 API에 대한 일반적인 흐름을 따라 프록시가 수행된다.
- 유효한 내부 StorageVersion 객체가 있고 API 서버가 해당 리소스를 제공하지 않는 경우, 처리 중인 API 서버는 해당 리소스를 처리할 수 있는 동료 API 서버를 찾아 요청을 프록시한다.

만약 동료 API 서버가 응답하지 않는 경우 (네트워크 연결 문제 등), 처리 중인 API 서버는 503 (`Service Unavailable`) 오류로 응답하는 경우가 있기 때문에 주의해야한다.