---
title: "GKE를 위한 GCP Network에 대하여"
date: 2023-10-17T23:03:15+09:00
tags:
  - cloud
  - network
  - GKE
  - GCP
categories:
  - cloud
series:
  - cloud
publishResources: true
---

# Google Cloud Platform Network for Google Kubernetes Engine

필자가 Google Kubernetes Engine을 사용할 때 처음으로 만든 Load Balancer는 External Passthrough Network Load Balancer이다. 그 이유[^default-lb]는 [NGINX Ingress Controller](https://docs.nginx.com/nginx-ingress-controller/)를 배포하면서 알게 되었는데, Service type을 `LoadBalancer`로 구성하게 되면 자동으로 GKE가 서비스 앞에 External Passthrough Network Load Balancer를 프로비저닝하게 된다.  

따라서 외부 IP를 가진 부하 분산기를 바로 쓸 수 있어서 NGINX Ingress Controller와 빠르게 조합이 가능했다. External Passthrough Network Load Balancer를 사용하다보니, 4 layer의 프로토콜을 그대로 받아서 Kubernetes 내부에서 처리해야 했다.  
TLS/SSL이 그대로 쿠버네티스에 들어오게 되고 NGINX Ingress Controller에서 TLS termination이 진행 된 뒤 Kubernetes Service에 전달되는 흐름으로 트래픽을 받을 수 있었다.
<figure>
  <img src="https://github.com/lee20h/blog/assets/59367782/b29ab6dc-3e57-497a-b51d-a6e84a6ae4b2"/>
  <figcaption>
   패스 스루 네트워크 부하 분산기 아키텍처 (<a href="https://cloud.google.com/load-balancing/docs/passthrough-network-load-balancer?hl=ko">출처</a>)
  </figcaption>
</figure>

## 문제점

트래픽이 쿠버네티스에 들어와서 복호화를 위한 리소스가 든다는 점 빼고는 Cert-manager를 통해서 Let's Encrypt 인증서 관리만 잘해준다면 크게 신경 쓸 부분이 없었다. 그러다보니, 굳이 다른 부하 분산기까지 들여다볼 일이 없어서 그대로 관리하였었다.  

하지만, 생기게 된 문제점을 나열하면 다음과 같다.

- Kubernetes 업데이트 혹은 장애로 인한 다운타임이 생길 때를 위한 고가용성 지원
- HTTP-01 validate를 이용해서 Let's Encrypt를 사용하면 도메인 하나가 한 IP에 묶이게 될텐데, 외부 IP를 가진 부하 분산기와 Kubernetes 한 클러스터와의 강결합
- TLS Termination과 Traffic Routing을 전부 백엔드에서 진행
- TCP 프로토콜에 대한 SSL을 지원하는 Ingress Controller가 많지 않음

## 방향성

Entry Point 역할을 하는 하나의 부하 분산기가 존재하고, 쿠버네티스마다 부하 분산기를 가져가면 어떨까? 라는 생각에서 시작하였다. 그렇다면, DNS나 Certificate가 앞단에서 해결이 된다면 뒷단에서는 VPC 내에서 비보안/보안 연결을 선택해서 할 수 있을 것 같았다.  
따라서 구글 클라우드의 부하 분산기[^lb]에 대해서 공부하게 되었고 다음과 같이 아키텍처를 구성해보았다. 

## 아키텍처

트래픽이 들어올 때 다음과 같이 통과하여 쿠버네티스에 전달되게 된다.

- Google Compute Firewall
- Cloud DNS
- Certificate Manager (Cert-map)
- GFE(Google Frontend): External Application Load Balancer (HTTPS) & External Proxy Network Load Balancer (SSL/TCP)
- BE: Network Endpoint Group
- Google Kubernetes Engine

![image](https://github.com/lee20h/blog/assets/59367782/3b00521d-74ca-4a69-a75f-8a9dcdb4f9b1)

1. 외부 IP를 예약하고 Cloud DNS 등록 
2. Certificate Manager로 DNS Authorization으로 인증서를 발급
3. 프론트엔드로 External Application Load Balancer (HTTPS) & External Proxy Network Load Balancer (SSL/TCP)를 작성
4. 이 때, GFE와 Cert-map 연결
5. 프론트엔드 포트에 맞춰서 방화벽 작업 
6. 백엔드로 Network Endpoint Group을 쓰기 위해 GKE에서 `servicenetworkendpointgroups` (svcneg) 등록
    - `cloud.google.com/neg: '{"exposed_ports":{"443":{"name":"neg"}}}'`

네트워크 아키텍처를 이렇게 수정하면 부하 분산기와 DNS가 글로벌로 사용할 수 있기 때문에 리전에 상관없이 고가용성을 확보할 수 있게 된다. Network Endpoint Group만 리전 별로 존재하게 되므로 Google Kubernetes Engine과 연결성만 관리해주면 유지보수도 증가할 것으로 보인다.  
또한 외부 IP가 고정인 상태에서 Network Endpoint Group만 갈아 끼운다면 충분히 무중단 노드 업데이트나 장애 발생 시 조치를 취할 수 있을 것으로 보인다.  

위에서 언급한 모든 구글 클라우드 제품들은 테라폼으로 관리가 가능하므로, 테라폼으로 사용해보면 좋을 것 같다.

[^default-lb]: [참고](https://cloud.google.com/kubernetes-engine/docs/concepts/network-overview?hl=ko#ext-lb)
[^lb]: [참고](https://cloud.google.com/kubernetes-engine/docs/concepts/network-overview?hl=ko)