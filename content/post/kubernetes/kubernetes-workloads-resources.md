---
title: "Kubernetes Workloads Resources에 대하여"
date: 2023-11-12T19:42:02+09:00
tags:
  - kubernetes
  - container
categories:
  - kubernetes
published: false
---

# Workload Resources

쿠버네티스는 워크로드를 선언적으로 관리하기 위해서 여러 내장 API를 제공한다. 결론적으로는, 애플리케이션은 파드 내부 컨테이너로 실행된다. 따라서 쿠버네티스는 개별 파드들을 관리해야한다.  
예를 들어, 파드가 실패해서 대체하거나 복사해야할 때 쿠버네티스 API를 사용하여 파드보다 더 높은 추상 워크로드 객체를 생성한 뒤 쿠버네티스 컨트롤 플레인이 정의한 워크로드 객체 사양에 따라 파드 객체를 관리한다.  

- Deployment
- ReplicaSet
- StatefulSet
- DaemonSet
- Job & CronJob

## Deployment

쿠버네티스에서 애플리케이션을 선언적으로 업데이트할 때 사용되며, Stateless 애플레케이션을 쉽게 생성하고 업데이트할 수 있다.  

다음과 같은 기능들을 포함하며 yaml로 정의된 바는 다음과 같다.

- 애플리케이션에 필요한 리소스를 생성
- 새 버전의 롤아웃 관리
- 이전 버전 롤백
- 애플리케이션 업데이트 시 자동화된 롤아웃과 롤백
- 업데이트 중에도 사용 가능

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx-deployment
  labels:
    app: nginx
spec:
  replicas: 3
  selector:
    matchLabels:
      app: nginx
  template:
    metadata:
      labels:
        app: nginx
    spec:
      containers:
      - name: nginx
        image: nginx:1.14.2
        ports:
        - containerPort: 80
```

위와 같이 컨테이너들과 스펙을 정의하게 되면 `ReplicaSet`을 생성하여 관리하게 된다. 가장 일반적인 배포 방법으로, 애플리케이션 배포와 업데이트를 쉽게 진행하도록 도와준다.  

관리의 예로는 애플리케이션 새 버전 롤아웃, 이전 버전 롤백, 스케일 업 또는 스케일 다운 해야하는 경우 사용 될 수 있다.  

## ReplicaSet

주요 목적은 안정적인 복제 파드 세트를 유지하기 위함이며, 지정된 수의 동일한 파드의 가용성을 보장하기 위해 사용 된다. 일반적으로 `Deployment`를 정의하여 자동으로 `ReplicaSet`을 관리하도록 한다.  
`ReplicaSet`은 파드를 식별하는 방법을 지정하는 `spec.selector`, 유지해야 할 파드 `spec.replicas` 수, 만들어질 파드의 템플릿으로 구성되어있다. 

```yaml
apiVersion: apps/v1
kind: ReplicaSet
metadata:
  name: frontend
  labels:
    app: guestbook
    tier: frontend
spec:
  replicas: 3
  selector:
    matchLabels:
      tier: frontend
  template:
    metadata:
      labels:
        tier: frontend
    spec:
      containers:
      - name: php-redis
        image: gcr.io/google_samples/gb-frontend:v3
```

`ReplicaSet`을 직접 생성하여 사용하는 경우보다 더 상위 수준의 개념인 `Deployment`를 사용하여 애플리케이션 선언적 관리를 하는 방법이 일반적이다.  

## StatefulSet

다른 컨트롤러와 동일하게 파드를 관리해주는 컨트롤러이나, Stateful 애플리케이션을 관리한다는 부분이 다르다. 또한, 특정 순서로 파드를 배포하고 스케일링하며 각 파드들은 고유한 식별자를 가진다.  

Stateful 애플리케이션을 관리한다는 말 그대로 다음과 같은 특징을 가지고 있다.

- 고유 식별: 스테이트풀셋의 각 파드는 순차적이고 고유한 식별자를 가진다.
- 순서 보장: 네트워크와 스토리지가 준비된 후에 파드가 순차적으로 배포된다.
- 지속적인 스토리지: 파드는 재시작 후에도 동일한 지속적인 스토리지 볼륨에 접근할 수 있다.
- 파드 이름의 일관성: 파드의 이름은 스테이트풀셋의 이름과 파드의 고유 식별자를 결합하여 생성된다.

특징에 이어서 예제 스펙은 다음과 같다.

```yaml
apiVersion: apps/v1
kind: StatefulSet
metadata:
  name: web
spec:
  selector:
    matchLabels:
      app: nginx # has to match .spec.template.metadata.labels
  serviceName: "nginx"
  replicas: 3 # by default is 1
  minReadySeconds: 10 # by default is 0
  template:
    metadata:
      labels:
        app: nginx # has to match .spec.selector.matchLabels
    spec:
      terminationGracePeriodSeconds: 10
      containers:
      - name: nginx
        image: registry.k8s.io/nginx-slim:0.8
        ports:
        - containerPort: 80
          name: web
        volumeMounts:
        - name: www
          mountPath: /usr/share/nginx/html
  volumeClaimTemplates:
  - metadata:
      name: www
    spec:
      accessModes: [ "ReadWriteOnce" ]
      storageClassName: "my-storage-class"
      resources:
        requests:
          storage: 1Gi
```

위와 같은 특징으로 인해서 순서가 중요한 데이터베이스나 클러스터 시스템, 데이터 복제나 샤딩이 필요한 시스템, 안정적인 넨트워크나 지속적인 스토리지가 필요한 애플리케이션 등으로 쓰일 수 있다.  
이러한 예시에서는 레플리카 수나 볼륨 클레임, 파드 템플릿을 고려해서 작성해야한다는 유의 사항이 존재한다.

---