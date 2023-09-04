---
title: "Helm에 대해 알아보자"
date: 2021-12-22T23:06:30+09:00
tags:
  - helm
  - kubernetes
categories:
  - devops
series:
  - helm
published: true
---

![](https://github.com/lee20h/blog/assets/59367782/38a4aad9-2386-4027-a99f-902e2622faec)

# Helm이란?
[Helm 공식사이트](https://helm.sh/)에서는 이렇게 소개한다.
> Helm is the best way to find, share, and use software built for Kubernetes.

Helm은 Kubernetes를 위해 만들어진 소프트웨어를 찾고, 공유하고, 사용할 수 있는 **가장 좋은 방법**이다.

왜 이렇게 소개하였는지 아래에서 차차 알아보자.

> helm은 [헬름], [헴], [헤음]으로 읽힌다.

## 특징

1. **Manage Complexity**

Helm 차트는 복잡한 애플리케이션을 쉽게 설치할 수 있으며 반복해서 설치할 수 있도록 도와준다. 또한, Sigle point of authority를 갖는다.

Kubernetes에서 한 애플리케이션을 위해서 여러 오브젝트가 필요한 경우 각각 배포하지 않고 Helm을 통해 한번에 배포하며 관리할 수 있다.

2. **Easy Updates**

배포된 애플리케이션에 대한 업데이트나 커스텀한 Hook을 등록하여 번거롭지 않게 업데이트를 진행할 수 있다.

3. **Simple Sharing**

Helm 차트는 공용, 개인 서버에 쉽게 **공유 및 호스팅**될 수 있다.

[Chartmuseum](https://chartmuseum.com/)과 같이 오픈소스를 사용하여 Private Repository로 사용하거나 [Artifacthub](https://artifacthub.io/)에서 공개 된 차트로 Kubernetes에 바로 배포할 수도 있다.

물론 Artifacthub에서도 Private Repository를 사용할 수 있다.

4. **Rollbacks**

Helm에서는 `helm rollback` 명령어를 이용하여 쉽게 이전 버전으로 롤백할 수 있다.

따라서 Helm을 통해서 설치하고 업데이트할 때마다 리비전으로 관리된다.

## 설치

- HomeBrew를 이용한 설치 (MAC)
```
$ homebrew install helm
```
- 그 외

[helm release](https://github.com/helm/helm/releases) 페이지에서 운영체제에 맞는 설치 파일로 설치할 수 있다.


## 명령어

Helm 설치가 되었다면 명령어를 통해서 Chart Repository 등록부터 외부 차트를 자신의 클러스터에 배포하는 명령어를 알아보자.

### Repository 등록

```
$ helm repo add [NAME] [URL] [flags]
$ helm repo update [REPO1 [REPO2 ...]] [flags]

$ helm repo add bitnami https://charts.bitnami.com/bitnami
$ helm repo update
```
위와 같이 `helm repo add url`로 repo를 등록 후 update하여 최신화한다.

### Chart 검색

```
$ helm search repo [keyword] [flags]

$ helm search repo bitnami
```

`helm search repo ###` 명령어는 로컬에 등록 된 Repo 중 이름이 포함된 Chart들을 찾아준다.

### Chart 정보 확인

```
$ helm show chart [CHART] [flags]

$ helm show chart bitnami/apache
```

차트(디렉터리, 파일 또는 URL)를 검사하고 Charts.yaml 파일의 내용을 보여준다.

```
$ helm show values [CHART] [flags]

$ helm show values bitnami/apache
```

차트(디렉터리, 파일 또는 URL)를 검사하고, values.yaml 파일의 내용을 보여준다.

다음에 다룰 **Helm Template**을 구성하는 모양을 볼 수 있다.
설치 시에 수정하고 싶은 부분을 파일이나 flag로 넘겨줘서 커스텀하게 설치가 가능하다.

또한 공개되어 있는 Repo들은 대부분 [Artifacthub](https://artifacthub.io/)에서 찾아볼 수 있으므로 메인테이너들이 권장하는 기본값들이나 커스텀을 위한 스키마들이 존재하므로 참고하면 좋다.

![](https://github.com/lee20h/blog/assets/59367782/a1b3fcb1-2ec2-4233-b08b-9400bdd25a87)


### 설치
```
$ helm install [NAME] [CHART] [flags]

$ helm install apache bitnami/apache -f myValues.yaml
or
$ helm install apache bitnami/apache --set foo=bar
```

Chart 정보를 확인하여 값을 수정하여 설치하고자 할 때 위와 같이 flag를 두면 파일 혹은 문자열로 설정이 가능하다.

또한 네임스페이스를 지정하여 Kubernetes Clsuter에 배포될 때의 네임스페이스를 정할 수 있다.

### 업그레이드

```
$ helm upgrade [RELEASE] [CHART] [flags]

$ helm upgrade apache bitnami/apache --set bar=foo
```

설치와 마찬가지로 비슷한 flag들을 가지고 있다. 이미 설치된 차트를 새로운 값들을 넣어 업그레이드 시키는 용도로 사용된다.

### 목록

```
$ helm list [flags]

$ helm list -A
```

helm list는 다른 flag가 없다면 현재 컨텍스트의 네임스페이스의 차트 릴리즈들을 보여준다.

-A flag를 통해 모든 네임스페이스의 차트 릴리즈들을 확인할 수 있다.

### 제거

```
$ helm uninstall RELEASE_NAME [...] [flags]

$ helm uninstall apache
```

설치할 때와 마찬가지로 제거할 때도 네임스페이스 스코프로 이뤄진다. 따라서 네임스페이스를 지정하지 않는다면 현재 컨텍스트의 네임스페이스로 잡혀서 릴리즈를 못 찾는 경우가 생길 수도 있다.

---

다음에는 커스텀하게 만드는 Helm Chart와 Template에 관하여 알아보자.

### 레퍼런스
- [helm docs](https://helm.sh/docs/helm/helm/)