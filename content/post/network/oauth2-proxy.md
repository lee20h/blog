---
title: "Oauth2 Proxy에 대하여"
date: 2023-12-11T22:31:15+09:00
tags:
  - network
  - kubernetes
categories:
  - network
published: true
---

# Oauth2 Proxy란?
[OAuth2 Proxy](https://oauth2-proxy.github.io/oauth2-proxy/)는 OAuth2 서버에 대해 인증을 제공해주는 리버스 프록시 구현체이다. OpenID Connect와 같은 OAuth2에 대해서 인증을 제공하는 애플리케이션의 경우에는 바로 프로바이더를 등록하여 사용할 수 있다. 하지만, 애플리케이션에서 인증을 따로 제공하지 않는다면 따로 작업이 필요하다.  



## 기능

그러한 작업들을 쉽게 할 수 있게 앞단에 바로 리버스 프록시를 통한 인증을 도와주는 것이 OAuth2 Proxy이다. 이름 그대로 OAuth2를 이용해서 인증을 제공한다.  
여러 프로바이더들을 통해서 그룹, 이메일, 도메인을 기반으로 사용자 인증을 진행한다. 사용자의 세션을 관리하고 쿠키를 사용하여 로그인 상태를 유지하게 된다. 따라서 앞단에서 정해진 인원들만 접근이 가능하며 SSO와 같이 한번의 로그인으로 세션을 유지할 수 있다. 이후에는 백엔드 서버에 인증된 요청을 리다이렉트하며 필요하면 수정한 트래픽을 보낼 수 있다.  

## 아키텍처

![image](https://github.com/lee20h/blog/assets/59367782/30a68812-6f19-477c-a8b6-750aff0b96e6)

## 작동 순서

1. 인증 요청: 여러 프로바이더(google, github, facebook ...)의 이메일, 도메인, 그룹 기반으로 인증 
2. 인증: 사용자가 프로바이더에 로그인하면, 프로바이더로부터 액세스 토큰 획득
3. 세션 생성: 인증 성공 시, 사용자 세션 생성 및 쿠키 저장
4. 접근 제어: 인증된 사용자 접근
5. 리다이렉트: 백엔드 서버에 리다이렉트 및 추가적인 작업

## 쿠버네티스에서 작업

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: app-oauth2-proxy
  namespace: default
spec:
  selector:
    matchLabels:
      app: app-oauth2-proxy
  template:
    metadata:
      labels:
        app: app-oauth2-proxy
    spec:
      nodeSelector:
        role: app
      containers:
        - name: oauth2-proxy
          image: quay.io/oauth2-proxy/oauth2-proxy:v7.5.1
          args:
            - --provider=oidc
            - --cookie-secure=true
            - --cookie-samesite=lax
            - --cookie-refresh=1h
            - --cookie-expire=4h
            - --cookie-name=auth
            - --set-authorization-header=true
            - --email-domain=example.com
            - --http-address=0.0.0.0:4180
            - --upstream=http://app.default.svc.cluster.local:8080
            - --redirect-url=https://app.example.com
            - --skip-provider-button=true
            - --whitelist-domain=https://app.example.com #URL변경필요
            - --oidc-issuer-url=https://dex.example.com #URL변경필요
            - --cookie-domain=example.com
          env:
            - name: OAUTH2_PROXY_SILENCE_PING_LOGGING
              value: "true"
            - name: OAUTH2_PROXY_REVERSE_PROXY
              value: "true"
            - name: OAUTH2_PROXY_UPSTREAMS
              value: http://app.default.svc.cluster.local:8080
            - name: OAUTH2_PROXY_CLIENT_ID
              value: app
            - name: OAUTH2_PROXY_CLIENT_SECRET
              value: app
            - name: OAUTH2_PROXY_COOKIE_SECRET
              value: secret
            - name: OAUTH2_PROXY_PASS_HOST_HEADER
              value: 'false'
          ports:
            - containerPort: 4180
              protocol: TCP
          readinessProbe:
            periodSeconds: 3
            httpGet:
              path: /ping
              port: 4180
---
apiVersion: v1
kind: Service
metadata:
  name: app-oauth2-proxy
  namespace: default
spec:
  selector:
    app: app-oauth2-proxy
  ports:
    - name: http
      port: 4180
---
```

- istio virtual serviec
```yaml
apiVersion: networking.istio.io/v1alpha3
kind: VirtualService
metadata:
  name: app-example-com
  namespace: default
spec:
  gateways:
    - default/gateway
  hosts:
    - app.example.com
  http:
    - rewrite:
        authority: app-oauth2-proxy.default.svc.cluster.local
      route:
        - destination:
            host: app-oauth2-proxy.default.svc.cluster.local
            port:
              number: 4180
```

- ingress
```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: app-example-com
  namespace: default
  annotations:
    kubernetes.io/ingress.class: nginx
    nginx.ingress.kubernetes.io/rewrite-target: /
    nginx.ingress.kubernetes.io/use-regex: "true"
spec:
  rules:
    - host: app.example.com
      http:
        paths:
          - path: /
            pathType: Prefix
            backend:
              service:
                name: app-oauth2-proxy
                port:
                  number: 4180
```

위의 예제들과 같이 OAuth2 Proxy를 구성한 뒤 환경변수로 리버스 프록시와 업스트림 주소를 등록하여 사용할 수 있다. 그리고 사용하고자하는 OIDC Issuer를 등록하여 사용하면 된다. 예제에서는 dex를 지정해서 사용하는 것처럼 만들었다.  
그 이후에는 네트워크를 OAuth2 Proxy를 향해 접근하도록 하면 된다. 왜냐하면 업스트림으로 원하는 애플리케이션을 향해서 보내도록 설정했기 때문이다.

이렇게 지정하게 되면, OIDC가 설정되어 있다는 가정하에 바로 사용할 수 있다.

---