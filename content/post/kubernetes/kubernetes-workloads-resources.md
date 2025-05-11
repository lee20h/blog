---
title: "Kubernetes Workloads Resources에 대하여"
date: 2023-11-12T19:42:02+09:00
tags:
  - kubernetes
  - container
categories:
  - kubernetes
publishResources: true
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

## DaemonSet

파드를 정의하는 다른 컨트롤러와 다르게 `DaemonSet`은 각 노드마다 파드를 정의하는 컨트롤러이다. 클러스터 운영에 필수적인 네트워킹 도구와 같이 모든 노드에서 적용될 법한 도구들을 위해서 사용한다.  
따라서 원하는 노드들에서 파드의 복사본을 실행하도록 보장하며, 노드가 클러스터에 추가되면 해당 노드에 파드가 추가되고 노드 제거 시 파드도 제거된다.  

클러스터 스토리지나 로그 수집, 노드 모니터링과 같이 모든 노드에서 파드가 존재해야하는 경우 사용하게 된다. 이 때 원하는 노드를 고르는 방법은 `.spec.template.spec.nodeSelector`을 이용하여 원하는 노드를 택하게 되고, 위에서 이야기한 대로 정의된 바로 따라서 각 노드마다 파드를 생성한다.  

```yaml
apiVersion: apps/v1
kind: DaemonSet
metadata:
  name: fluentd-elasticsearch
  namespace: kube-system
  labels:
    k8s-app: fluentd-logging
spec:
  selector:
    matchLabels:
      name: fluentd-elasticsearch
  template:
    metadata:
      labels:
        name: fluentd-elasticsearch
    spec:
      containers:
      - name: fluentd-elasticsearch
        image: quay.io/fluentd_elasticsearch/fluentd:v2.5.2
```

위와 같이 정의하면 `DaemonSet`을 사용할 수 있다. 이 때, 노드 레이블이 변경되면 일치하는 노드에 파드를 추가하고 더 이상 일치하지 않으면 파들르 제거한다. 또, `--cascade=orphan` 옵션을 사용하면 파드는 노드에 남게 된다.  

## Job

`Job`은 쿠버네티스에서 한번 실행하고 완료되는 작업을 관리하는 컨트롤러다. `Job`에서는 파드를 관리하며 파드가 성공적으로 완료되면 잡도 완료된다.  
데이터 처리 및 분석과 같은 배치성 작업과 스케줄링된 작업, 병렬 작업 같은 경우 `Job`으로 사용하는 예시가 될 수 있다.  

단순히 한번만 실행하는 오브젝트로 다음과 같은 특징을 가지고 있다.

- 하나 이상의 파드를 생성하고, 지정된 수의 파드가 완료될 때까지 재시도한다.
- 파드가 실패한 경우, 새로운 파드를 시작할 수 있다.
- `CronJob`을 이용하여 정기적인 작업을 실행할 수 있다.

```yaml
apiVersion: batch/v1
kind: Job
metadata:
  name: pi
spec:
  template:
    spec:
      containers:
      - name: pi
        image: perl
        command: ["perl", "-Mbignum=bpi", "-wle", "print bpi(2000)"]
      restartPolicy: Never
  backoffLimit: 4
```

## CronJob

`CronJob`은 `Job`을 한번 더 추상화한 오브젝트로, 반복적인 일정에 일회성 작업을 한다. 주로 백업, 보고서 생성 등과 같은 작업에 사용된다. 또한, `CronJob`의 스케줄링은 유닉스의 Crontap 파일의 스케줄과 동일하다.  
또한, 단일 크론잡이 여러 개의 동시 작업을 생성할 수 있으며, `Job`과 동일하게 하나 이상의 파드를 생성하고, 지정된 수의 파드가 성공적으로 완료 될 때까지 재시도한다.

```yaml
apiVersion: batch/v1
kind: CronJob
metadata:
  name: hello
spec:
  schedule: "* * * * *"
  jobTemplate:
    spec:
      template:
        spec:
          containers:
          - name: hello
            image: busybox:1.28
            command: ["/bin/sh", "-c", "date; echo Hello from the Kubernetes cluster"]
          restartPolicy: OnFailure
```

## ReplicationController

ReplicationController의 경우에는 ReplicaSet으로 대체하여 관리하며, Deployment와도 연결하여 사용하기 때문에 다루지 않았다.  
간단하게 메모하자면, 두 가지의 차이점이 존재한다.  

1. 셀렉터
2. 롤링 업데이트 유무

ReplicationController는 레이블이 같은지 다른지만 비교하는 반면, ReplicaSe은 집합 기반으로 in, notin, exist와 같은 연산자를 지원한다.  
또한, ReplicationController는 Rolling-update를 지원하지만, ReplicaSet는 Deployment를 통해서 지원한다.