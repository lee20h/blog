---
title: "Kubernetes Networking에 대하여"
date: 2023-11-14T22:10:54+09:00
tags:
  - kubernetes
  - container
  - network
categories:
  - kubernetes
publishResources: false
---

# Kubernetes Network

앞 포스트에서는 쿠버네티스 워크로드에 대해서만 이야기했었는데, 워크로드만 존재하면 통신이 가능하지는 않다. 그렇다면 통신을 가능케하고 유지해주는 리소스에 대해서 알아보자.  

## Kubernetes Network Model

쿠버네티스에서는 각 파드마다 고유한 클러스터 전체 IP 주소가 할당된다. 따라서 파드 간에 명시적으로 링크를 생성할 필요가 없으며, 대부분 컨테이너 포트를 호스트 포트에 매핑할 필요가 없다.  
이렇게 되면 파드들을 VM이나 물리적 호스트처럼 취급할 수 있어서 포트 할당, 네이밍, 서비스 디스커버리, 로드 밸런싱, 애플리케이션 구성 및 마이그레이션 측면에서 간단하고 호환 가능하다.  

쿠버네티스는 모든 파드가 NAT 없이도 쿠버네티스 내 모든 파드와 통신할 수 있어야 한다는 요구사항이 있다. 왜냐하면 노드의 kubelet이나 데몬같은 에이전트도 모든 파드와 통신이 가능해야하기 때문이다.  
또 쿠버네티스에서 IP 주소는 파드 범위에서 존재한다. 따라서, 하나의 파드 내의 컨테이너들은 네트워크 네임스페이스를 공유하게 된다. IP 주소와 MAC 주소를 포함한 네트워크 설정이 파드 내의 모든 컨테이너들에게 동일하게 적용된다. 그러면 파드 내의 컨테이너들은 서로 포트에 로컬호스트를 통해 접근할 수 있다.  
이러한 구조는 파드 내의 컨테이너들이 포트 사용을 조정해야한다. VM 내의 프로세스들이 포트를 조정하는 것과 유사하며, 이러한 방식을 `파드당 IP` (IP-per-pod) 모델이라고 한다.

쿠버네티스 네트워킹의 4가지 컨셉을 나열해보면 다음과 같다.

- 파드 내의 컨테이너들은 loopback을 통해 네트워킹을 사용하여 서로 통신하게 된다.
  - 파드 내부에서만 접근할 수 있는 내부 네트워크로, 파드의 다른 컨테이너와의 통신에 사용되며, 외부 네트워크와 분리되어 있다.
- 클러스터 네트워킹은 다른 파드들 간의 통신을 제공한다.
- 서비스 API는 클러스터 외부에서 실행 중인 애플리케이션에 접근할 수 있게 해준다.
  - Ingress는 HTTP 애플리케이션, 웹 사이트 및 API를 외부에 노출시키기 위한 추가 기능을 제공한다.
  - Gateway API는 서비스 네트워킹을 모델링하기 위한 표현적이고 확장 가능하며 역할 중심의 API를 제공한다. 
- 서비스를 사용하여 클러스터 내에서만 사용될 서비스를 공개할 수도 있다.

> loopback이란?  
> 컴퓨터 네트워킹에서 자기 자신으로 데이터를 보내고 받는 통신 방식으로, 주로 네트워크 인터페이스에서 사용되어 컴퓨터가 자신의 네트워크 장치를 통해 자신에게 데이터를 보내고 그 데이터를 받는 과정을 말한다.  
> 네트워크가 없을 때도 네트워크 서비스 통신을 할 수 있게 해주며, 네트워크를 통하지 않고 자신에게 돌아오게 된다. 주소는 `127.0.0.1` 혹은 `::1`로 설정되어 있다.


## Service

클러스터 내에서 실행되는 애플리케이션을 단일 외부 지향 엔드포인트 뒤에 노출하는 방법으로, 실행 중인 하나 이상의 파드로 구성된 네트워크 애플리케이션을 노출시킨다.  
예를 들어 쿠버네티스에서 파드들은 원하는 상태와 일치하도록 생성과 파괴가 반복된다. **CNI(Container Network Interface)**에 의해 파드들은 동적 IP 주소를 할당 받는다. 그렇다면 네트워크를 통해 어떤 파드인지 알고 접근할 수 있을까 라는 의문이 생긴다.  

이러한 의문을 쿠버네티스 서비스 API가 해결해준다. 서비스 객체가 논리적인 엔드포인트 세트와 파드에 접근하는 방법에 대해 정의한다. 일반적으로는 `Selector`에 의해서 어떤 파드 세트를 타겟으로 할지 정하게 되는데, Selector가 없는 서비스도 존재한다.

```yaml
apiVersion: v1
kind: Service
metadata:
  name: my-service
spec:
  selector:
    app.kubernetes.io/name: MyApp
  ports:
    - protocol: TCP
      port: 80
      targetPort: 9376
```

이러한 모양으로 **Selector**가 있는 서비스는 컨트롤러가 **Selector**와 일치하는 파드를 지속적으로 검색해서 `Endpoint` 오브젝트에 서비스 디스커버리 정보를 넣어주게 된다.  

```yaml
apiVersion: v1
kind: Service
metadata:
  name: my-service
spec:
  ports:
    - protocol: TCP
      port: 80
      targetPort: 9376
---
apiVersion: discovery.k8s.io/v1
kind: EndpointSlice
metadata:
  name: my-service-1 # 관행적으로, 서비스의 이름을
  # 엔드포인트슬라이스 이름의 접두어로 사용한다.
  labels:
    # "kubernetes.io/service-name" 레이블을 설정해야 한다.
    # 이 레이블의 값은 서비스의 이름과 일치하도록 지정한다.
    kubernetes.io/service-name: my-service
addressType: IPv4
ports:
  - name: '' # 9376 포트는 (IANA에 의해) 잘 알려진 포트로 할당되어 있지 않으므로
    # 이 칸은 비워 둔다.
    appProtocol: http
    protocol: TCP
    port: 9376
endpoints:
  - addresses:
      - "10.4.5.6" # 이 목록에 IP 주소를 기재할 때 순서는 상관하지 않는다.
      - "10.1.2.3"
```

**Selector**가 없는 서비스는 `EndpointSlice`와 같이 사용하면 다른 종류의 백엔드들을 추상화할 수 있다. 예를 들어 클러스터 내에 서비스와 클러스터 외부에 있는 서비스를 지정하듯이, 클러스터 안팎으로 연결해야하는 경우에 사용된다.  
이럴 때는 **ExternalName** 서비스를 사용하기도 한다.

## EndpointSlice Kubernetes v1.21 [stable]

네트워크 엔드포인트들의 부분 집합을 의미하며, 이전에는 `Endpoint`와 서비스가 연결이 되어있으나, 엔드포인트의 양이 많아지면 감당이 안되므로 `EndpointSlice` 개념이 생겨나게 되었다. 앞에서 말했듯이 엔드포인트들의 부분집합인 이유는 엔드포인트를 용량 이상 갖게 된다면 새로운 엔드포인트슬라이스를 생성하여 분리해서 관리하게 된다.  

**Selector**가 없는 서비스의 경우 이러한 `EndpointSlice`를 정의하여 사용한다는 것은 `Endpoint`가 매핑이 되지 않으므로 수동으로 설정해서 사용한다는 말을 뜻한다는 것을 알 수 있다.

## Ingress

클러스터 외부에서 클러스터 내의 서비스로 HTTP 및 HTTPS 경로를 노출한다. 트래픽 라우팅은 정의된 규칙에 의해서 제어된다. 여기에서 인그레스 혼자서 사용되는 것은 불가능하고, 무조건 인그레스 컨트롤러가 필요하다. 인그레스 컨트롤러에는 Nginx나 Treafik과 같은 여러 구현체가 존재한다.  
인그레스 컨트롤러마다 다르지만 정한 규칙에 의해서 트래픽을 컨트롤하는 부분은 동일하다.

![image](https://github.com/lee20h/blog/assets/59367782/e1a5a699-88a9-41f5-b05e-73816201f685)

그림과 같이 외부로부터의 트래픽을 파드에 전달하는 매개체 역할을 하며, 규칙에 의거하여 서비스에 전달하는 역할을 한다.

```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: minimal-ingress
  annotations:
    nginx.ingress.kubernetes.io/rewrite-target: /
spec:
  ingressClassName: nginx-example
  rules:
  - http:
      paths:
      - path: /testpath
        pathType: Prefix
        backend:
          service:
            name: test
            port:
              number: 80
```

단일 서비스나, 여러 서비스에 같은 IP를 통해 접근하도록 규칙을 만들 수도 있고, DNS나 가상 호스트 기반으로 규칙을 정할 수 있다. 보안 연결을 위한 TLS를 구현하기 위한 비밀키와 인증서를 Secret에 담아서 사용할 수도 있다.

### Ingress Rule

HTTP 및 HTTPS 트래픽을 쿠버네티스 서비스로 어떻게 라우팅되는지 정의하는 부분으로, 하나의 인그레스가 여러의 서비스로 보내거나 부하 분산, TLS 종료, 이름 기반 가상 호스팅 등 여러 기능들을 구성할 수 있다.  
이때 실제 라우팅은 인그레스 컨트롤러에서 처리하게 된다. 이 인그레스 컨트롤러에서 클라우드 부하 분산기나 리버스 프록시 같은 여러 라우팅 매커니즘이 될 수 있다.  

### Ingress Class

인그레스 클래스는 동일한 쿠버네티스 클러스터 내에서 다양한 인그레스 컨트롤러를 구현할 수 있게 해주는 기능이다. 이 기능은 다양한 목적을 위한 다른 유형의 인그레스 컨트롤러가 필요할 때 쓰인다.  
즉, 인그레스를 생성할 때 클래스를 통해 여러 컨트롤러 중 원하는 컨트롤러를 정해서 사용할 수 있게 한다. 주석과 매개변수를 통해 클래스의 동작을 커스텀하게 정의하여 사용할 수 있다.

```yaml
apiVersion: networking.k8s.io/v1
kind: IngressClass
metadata:
  name: external-lb
spec:
  controller: example.com/ingress-controller
  parameters:
    apiGroup: k8s.example.com
    kind: IngressParameters
    name: external-lb
---
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: minimal-ingress
  annotations:
    nginx.ingress.kubernetes.io/rewrite-target: /
spec:
  ingressClassName: external-lb
  rules:
    - http:
        paths:
          - path: /testpath
            pathType: Prefix
            backend:
              service:
                name: test
                port:
                  number: 80
```

### Ingress Controller

인그레스 작동을 위한 소프트웨어가 바로 인그레스 컨트롤러다. 다른 유형들의 컨트롤러들은 `kube-controller-manager` 바이너리의 일부로 실행되는 것과 달리, 인그레스 컨트롤러는 자동으로 만들어지지 않는다. 따라서 원하는 컨트롤러를 선택하여 관리해야 한다.  
여러 인그레스 컨트롤러가 존재하는데, 대부분 클라우드 벤더사에서 제공하는 로드 밸런서와 연결되는 컨트롤러와 NGINX, Istio, Kong, Cilium, HAProxy 등 여러 인그레스 컨트롤러가 존재한다. 이 컨트롤러들은 클래스 내에 여러 개를 배포할 수도 있고, 기본 인그레스 클래스로 지정할 수 있다.

## Gateway API 

게이트웨이 API는 인그레스와 같이 트래픽 라우팅을 위한 API로, 동적 인프라 프로비저닝이나 고급 트래픽 라우팅 기능을 주로 삼고 있다. 인그레스 후속 API로, 인그레스를 전혀 포함하고 있지 않은 별개의 API로 봐야한다.  
이러한 게이트웨이는 아래의 원칙들을 가지고 설계되었다.

- 역할 지향적: 쿠버네티스 서비스 네트워킹을 관리하는 조직의 역할에 따라 다를 수 있다.
  - 인프라 제공자: 여러 클러스터에서 여러 테넌트에 서비스를 제공할 수 있도록 인프라 관리하는 역할
  - 클러스터 운영자: 클러스터를 관리하며, 정책, 네트워크 접근, 애플리케이션 권한 등 운영에 관련된 역할
  - 애플리케이션 개발자: 클러스터에서 실행되는 애플리케이션 수준 및 서비스 구성하는 역할
- 휴대성: 커스텀 리소스로 정의되어 여러 구현에서 지원된다.
- 표현력: 헤더 기반 매칭, 트래픽 가중치 조정같이 인그레스에서는 어노테이션으로만 구현되었던 기능들이 일반 기능으로 존재한다.
- 확장성: API의 다양한 계층에서 커스텀 리소스를 연결이 가능함으로, 세부적인 맞춤 설정이 가능하다.

위 원칙들을 가지고 만들어진 게이트웨이는 3가지의 리소스를 가지고 사용된다.  
`Gateway`, `GatewayClass`, `HTTPRoute` 3가지의 리소스로 인그레스보다 더 많은 기능들을 지원하며 확장성을 가진다고 볼 수 있다. 아래의 그림은 리소스들의 관계도로, 앞으로 설명할 부분을 전부 보여준다. 

![image](https://github.com/lee20h/blog/assets/59367782/f495ac71-1ae4-46a3-9e5f-2bf9bce7bfb6)

게이트웨이는 게이트웨이 클래스 하나와 연결되고 하나 이상의 라우트 종류와 연결되어야 한다. 즉, 게이트웨이 클래스는 인그레스 컨트롤러와 같은 역할을 하며 들어온 트래픽을 원하는 라우트로 필터링해서 보내게 된다.  
즉, 하나의 클래스를 사용하며 여러 게이트웨이들이 자신들이 라우팅을 필터링할 여러 라우트들을 가지고 트래픽을 보내는 방식으로 진행된다는 것이다. 트래픽의 흐름을 그려보면 아래 그림과 같이 이해할 수 있다.

![image](https://github.com/lee20h/blog/assets/59367782/267338e0-3e70-4819-931f-be81a5afca89)

여기에서 사용되는 리소스들을 예제로 작성해보자.

```yaml
apiVersion: gateway.networking.k8s.io/v1
kind: GatewayClass
metadata:
  name: example-class
spec:
  controllerName: example.com/gateway-controller
---
apiVersion: gateway.networking.k8s.io/v1
kind: Gateway
metadata:
  name: example-gateway
spec:
  gatewayClassName: example-class
  listeners:
  - name: http
    protocol: HTTP
    port: 80
---
apiVersion: gateway.networking.k8s.io/v1
kind: HTTPRoute
metadata:
  name: example-httproute
spec:
  parentRefs:
    - name: example-gateway
  hostnames:
    - "www.example.com"
  rules:
    - matches:
        - path:
            type: PathPrefix
            value: /login
      backendRefs:
        - name: example-svc
          port: 8080
```

## Network Policies

네트워크 정책은 쿠버네티스 클러스터 내에서 트래픽 흐름을 제어하는 데 사용되는 규칙으로 OSI 레이어 3(네트워크) 또는 레이어 4(전송)에서 IP 주소나 포트 수준으로 트래픽을 제어하고자 할 때 유용하다. 네트워크 정책은 파드 간 뿐 아니라 파드와 외부 세계 간의 통신을 제어하는 데 사용된다. 

네트워크 정책을 이루는 구성 요소는 다음과 같이 3가지로 이뤄져있다.  

- Policy Types
  - Ingress: 들어오는 트래픽을 제어한다. 기본적으로 모든 인바운드 연결이 허용되지만, 특정 정책을 적용하여 제한할 수 있다.
  - Egress: 나가는 트래픽을 제어한다. 기본적으로 모든 아웃바운드 연결이 허용되지만, 특정 정책을 적용하여 제한할 수 있다. 
- Selectors
  - Pod Selector: 특정 파드 또는 파드 그룹을 대상으로 정책을 적용한다. 레이블 셀렉터를 사용하여 특정 파드를 선택할 수 있다.
  - Namespace Selector: 특정 네임스페이스의 모든 파드를 대상으로 정책을 적용한다. 이를 통해 네임스페이스 간의 통신을 제어할 수 있다. 
- Allow Rules
  - From/To: 특정 출처(ingress) 또는 목적지(egress)를 지정하여 특정 트래픽을 허용한다.
    - 다른 파드(파드 셀렉터를 사용하여 지정)
    - 다른 네임스페이스(네임스페이스 셀렉터를 사용하여 지정)
    - IP 블록(CIDR 표기법을 사용하여 특정 IP 범위 지정)
  - Ports: 특정 포트 또는 포트 범위에 대한 트래픽을 허용한다. 프로토콜(TCP, UDP, SCTP)과 함께 지정될 수 있다.

### 네트워크 정책의 작동 방식

- 기본적으로, 네트워크 정책이 없는 경우 모든 트래픽이 허용된다. '기본 거부(default deny)' 정책을 만들어 특정 트래픽만 허용하도록 설정할 수 있다.
- 여러 네트워크 정책이 독립적으로 평가되어 하나의 파드에 적용될 수 있다. 이 경우, 허용된 트래픽은 모든 관련 정책에 의해 허용된 트래픽의 합집합으로 적용된다.

### 예시

```yaml
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: test-network-policy
  namespace: default
spec:
  podSelector:
    matchLabels:
      role: db
  policyTypes:
    - Ingress
    - Egress
  ingress:
    - from:
      - ipBlock:
          cidr: 172.17.0.0/16
          except:
            - 172.17.1.0/24
      - namespaceSelector:
          matchLabels:
            project: myproject
      - podSelector:
          matchLabels:
            role: frontend
      ports:
        - protocol: TCP
          port: 6379
  egress:
    - to:
      - ipBlock:
          cidr: 10.0.0.0/24
      ports:
        - protocol: TCP
          port: 5978
```

