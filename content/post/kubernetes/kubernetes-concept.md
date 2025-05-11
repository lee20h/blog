---
title: "Kubernetes Concept에 대하여"
date: 2023-11-03T21:09:36+09:00
tags:
  - kubernetes
  - container
categories:
  - kubernetes
publishResources: true
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

- Cluster Architecture
- Containers

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

---

## Containers

쿠버네티스에서 컨테이너는 애플리케이션과 그 실행에 필요한 모든 종속성을 패키징하는 기술이다. 컨테이너는 어디서든 동일한 동작을 보장하는 반복 가능한 단위로, 애플리케이션을 호스트 인프라에서 분리하여 다양한 클라우드나 OS 환경에서의 배포를 용이하게 한다.

### Container Image

쿠버네티스에서 컨테이너 이미지는 애플리케이션과 그에 필요한 모든 소프트웨어 종속성을 포함하는 이진 데이터를 나타낸다. 컨테이너 이미지는 독립적으로 실행 가능한 소프트웨어 번들로, 런타임 환경에 대해 명확한 가정을 한다.  
일반적으로 사용자는 애플리케이션의 컨테이너 이미지를 생성하고, 이를 레지스트리에 푸시한 후, Pod에서 참조한다.

- 컨테이너 이미지는 실행 준비가 된 소프트웨어 패키지로, 애플리케이션 코드, 런타임, 라이브러리, 설정 등을 포함한다. 
- 컨테이너는 상태가 없고 변경 불가능하게 설계되어야 한다. 
- 실행 중인 컨테이너의 코드를 변경하는 것은 권장하지 않는다.

**이미지 이름**

컨테이너 이미지는 일반적으로 `pause`, `example/mycontainer`, `kube-apiserver`와 같은 이름을 가진다. 레지스트리 호스트명도 포함될 수 있다. 예를 들어, `fictional.registry.example/imagename`과 같이 될 수 있다.

**이미지 업데이트**

처음 Deployment, StatefulSet, Pod 등을 생성할 때, 기본적으로 모든 컨테이너의 pull 정책이 `IfNotPresent`로 설정된다. 이 정책은 이미지가 로컬에 존재하는 경우 이미지를 pull하지 않는다.

**이미지 pull 정책**

- `imagePullPolicy`는 컨테이너의 속성 중 하나로, 이미지를 언제 pull 할지를 결정한다.
- `IfNotPresent`, `Always`, `Never` 등의 값을 가질 수 있다.

**다중 아키텍처 이미지**

컨테이너 레지스트리는 아키텍처별 버전의 컨테이너를 가리키는 이미지 인덱스를 제공할 수 있다. 이를 통해 다양한 시스템이 사용하는 머신 아키텍처에 적합한 바이너리 이미지를 가져올 수 있다.

**프라이빗 레지스트리 사용**

프라이빗 레지스트리는 이미지를 읽기 위해 키가 필요할 수 있다. 이러한 인증 정보는 여러 방법으로 제공될 수 있다.

**이미지 Pull 실패 시**

`ImagePullBackOff` 상태는 쿠버네티스가 컨테이너 이미지를 pull할 수 없어 컨테이너를 시작할 수 없는 상태를 나타낸다.

### Container Environment

- 파일시스템: 이미지와 하나 이상의 볼륨의 조합
- 컨테이너 정보: 컨테이너 자체에 대한 정보
  - 컨테이너의 호스트명은 해당 컨테이너가 실행 중인 Pod의 이름으로, hostname 명령어나 libc의 gethostname 함수 호출을 통해 확인할 수 있다.
  - Pod 이름과 네임스페이스는 [downward API](https://kubernetes.io/docs/tasks/inject-data-application/downward-api-volume-expose-pod-information/)를 통해 환경 변수로 사용할 수 있다.
  - Pod 정의에서 사용자가 정의한 환경 변수도 컨테이너에서 사용 가능하다.
- 클러스터의 다른 객체 정보: 클러스터 내의 다른 객체에 대한 정보
  - 컨테이너가 생성될 때 실행 중이었던 모든 서비스의 목록이 환경 변수로 해당 컨테이너에 제공된다. 이 목록은 새 컨테이너의 Pod와 동일한 네임스페이스 내의 서비스와 쿠버네티스 제어 평면 서비스로 제한된다.
  - 서비스는 전용 IP 주소를 가지며, DNS 애드온이 활성화된 경우 컨테이너에서 DNS를 통해 사용할 수 있다.

### Container Runtime

컨테이너 런타임은 쿠버네티스가 컨테이너를 효과적으로 실행하는 데 필요한 기본 구성 요소로, 컨테이너의 실행과 라이프사이클을 관리한다.  
쿠버네티스는 containerd, CRI-O 등의 컨테이너 런타임을 지원한다.

### RuntimeClass

클러스터에서 여러 컨테이너 런타임을 사용해야 하는 경우, Pod에 대해 RuntimeClass를 지정하여 특정 컨테이너 런타임을 사용하도록 할 수 있다.

런타임 클래스를 사용하면, 다른 Pod 간에 다른 런타임 클래스를 설정하여 성능과 보안 간의 균형을 제공할 수 있다. 예를 들어, 특정 작업이 높은 수준의 정보 보안을 필요로 하는 경우, 하드웨어 가상화를 사용하는 컨테이너 런타임에서 실행되도록 해당 Pod를 스케줄링할 수 있다.

**설정(Setup)**

- 노드에서 CRI 구현 구성: 런타임 핸들러 이름이 있는 구성이 필요하며, 이는 RuntimeClass에 의해 참조된다.
- 해당 RuntimeClass 리소스 생성: 각 핸들러에 대해 해당하는 RuntimeClass 객체를 생성한다.

**사용법(Usage)**

런타임 클래스가 클러스터에 구성되면, Pod 스펙에 runtimeClassName을 지정하여 사용할 수 있다.

**CRI 구성(CRI Configuration)**

CRI 런타임은 노드에서 구성되며, containerd, CRI-O 등의 구성 방법이 있다.

**스케줄링(Scheduling)**

런타임 클래스의 scheduling 필드를 지정하여, 해당 RuntimeClass를 지원하는 노드에만 Pod가 스케줄링되도록 제약을 설정할 수 있다.

**Pod Overhead**

Pod Overhead을 통해 Pod를 실행하는 데 연관된 오버헤드 리소스를 지정할 수 있다. 오버헤드를 선언하면, 클러스터(스케줄러 포함)가 Pod 및 리소스에 대한 결정을 내릴 때 이를 고려할 수 있다.

### Container Lifecycle Hooks

쿠버네티스의 컨테이너 라이프사이클 훅은 컨테이너의 관리 라이프사이클 동안 발생하는 이벤트에 의해 트리거되는 코드를 실행하기 위한 프레임워크를 제공한다.

**컨테이너 훅(Container Hooks)**

- **PostStart**: 컨테이너가 생성된 직후에 실행된다. 그러나 ENTRYPOINT 전에 실행될 것이라는 보장은 없다.
- **PreStop**: 컨테이너가 API 요청이나 관리 이벤트로 인해 종료되기 직전에 호출된다.

**훅 핸들러 구현(Hook Handler Implementations)**

컨테이너는 아래와 같은 훅을 구현하고 등록함으로써 훅에 액세스할 수 있다.

- Exec: 컨테이너의 cgroups 및 네임스페이스 내에서 특정 명령(예: pre-stop.sh)을 실행한다.
- HTTP: 컨테이너의 특정 엔드포인트에 대해 HTTP 요청을 실행한다.

**훅 핸들러 실행(Hook Handler Execution)**

- Hook Handler Execution: 컨테이너 라이프사이클 관리 훅이 호출되면, 쿠버네티스 관리 시스템은 훅 액션에 따라 핸들러를 실행한다. **httpGet**과 **tcpSocket**은 kubelet 프로세스에 의해 실행되며, exec는 컨테이너 내에서 실행된다.
- Synchronous Calls: 훅 핸들러 호출은 컨테이너를 포함하는 Pod의 컨텍스트 내에서 동기적으로 이루어진다. 즉, **PostStart** 훅의 경우, 컨테이너의 ENTRYPOINT와 훅은 비동기적으로 실행된다. 그러나 훅이 너무 f오래 실행되거나 멈추면, 컨테이너는 실행 상태에 도달할 수 없다.
- PreStop Hook: PreStop 훅은 컨테이너를 중지시키는 신호로부터 비동기적으로 실행되지 않는다. 훅이 실행을 완료해야 TERM 신호가 전송될 수 있다. **PreStop** 훅이 실행 중에 멈추면, Pod의 상태는 Terminating이 되고, `terminationGracePeriodSeconds`가 만료될 때까지 그 상태를 유지한다.
- Failed Hooks: PostStart 또는 PreStop 훅 중 하나가 실패하면, 컨테이너는 종료된다.
- Recommendation for Lightweight Handlers: 사용자는 훅 핸들러를 가능한 한 경량화해야 한다. 그러나 상태를 저장하는 등의 긴 실행 명령이 필요한 경우도 있다.

**훅 전달 보장(Hook Delivery Guarantees)**

- 훅은 최소한 한 번 이상 호출될 것이라는 의도로 설계되었다. 즉, PostStart나 PreStop과 같은 주어진 이벤트에 대해 훅이 여러 번 호출될 수 있다.
- 일반적으로 단일 전달만 수행된다. 예를 들어, HTTP 훅 수신기가 다운되어 트래픽을 처리할 수 없는 경우, 재전송을 시도하지 않는다.
- 그러나 드물게 두 번 전달될 수 있다. 예를 들어, kubelet이 훅을 전송하는 중간에 재시작하는 경우, kubelet이 다시 시작된 후 훅이 다시 전송될 수 있다.

**훅 핸들러 디버깅(Debugging Hook Handlers)**

- 훅 핸들러의 로그는 Pod 이벤트에 노출되지 않는다.
- 핸들러가 어떤 이유로 실패하면 이벤트를 방송한다. **PostStart**의 경우 `FailedPostStartHook` 이벤트가, **PreStop**의 경우 `FailedPreStopHook` 이벤트가 발생한다.
- 실패한 `FailedPostStartHook` 이벤트를 직접 생성하려면, 'lifecycle-events.yaml' 파일을 수정하여 **postStart** 명령을 "badcommand"로 변경하고 적용하면 된다.
- `kubectl describe pod lifecycle-demo`를 실행하여 볼 수 있는 결과 이벤트의 예시 출력이 제공된다.

라이프사이클 훅을 사용하면, 컨테이너의 생성과 종료 시점에 사용자 정의 로직을 실행할 수 있어, 상태 저장이나 초기화 작업 등을 수행하는 데 유용하다.

---
