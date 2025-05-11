---
title: "Helm Chart와 Template에 대해 알아보자"
date: 2021-12-30T23:14:17+09:00
tags:
  - helm
  - kubernetes
categories:
  - devops
series:
  - helm
publishResources: true
---

![image](https://github.com/lee20h/blog/assets/59367782/71942a7b-e70d-47a2-9ed9-1709ac1a2422)

# Helm Chart란?

[이전 포스팅](https://velog.io/@lee20h/Helm%EC%97%90-%EB%8C%80%ED%95%B4-%EC%95%8C%EC%95%84%EB%B3%B4%EC%9E%90)에서 이야기한 Helm를 제대로 사용하려면 Chart에 대해 알아야한다.

공식페이지에서는 다음과 같이 설명한다.

> Chart는 Helm에서 사용하는 패키지 포맷이다.
차트는 쿠버네티스 리소스와 관련된 셋을 설명하는 파일의 모음이다.
하나의 차트는 memcached 파드를 배포하는 것처럼 단순한 형태나 HTTP 서버, 데이터베이스, 캐시 등으로 구성된 완전한 웹앱 같이 복잡한 형태로 사용될수 있다.

즉, Helm에서의 Chart는 애플리케이션 하나를 배포하기 위해 필요한 일련의 **쿠버네티스 리소스**들을 포함한다.

어떻게 보면, Chart 하나가 우리가 생각하는 **애플리케이션 하나**라고 볼 수 있을 것 같다.

> 차트는 특정한 구조를 가진 파일들로 생성되어 배포할 버전에 맞춰서 아카이브할 수 있다.

여기서 나온 **구조**가 이번 포스팅에서 얘기할 핵심이다.

## Chart 구조

```
$ helm pull chartrepo/chartname
or
$ helm create chartname
```

pull 명령어를 통해서 Helm Chart를 로컬에 다운로드 받을 수 있다. 다운로드 받게 되면 압축파일 형태로 받아진다.

압축을 해제하면 볼 수 있는 디렉토리 구조는 다음과 같다.

```
├── Chart.yaml
├── charts
├── templates
│   ├── NOTES.txt
│   ├── _helpers.tpl
│   ├── deployment.yaml
│   ├── ingress.yaml
│   ├── service.yaml
│   └── serviceaccount.yaml
└── values.yaml
```

- Chart.yaml: Chart의 정보를 정의하는 파일로 Chart의 이름, 버전 등을 정의한다. 이 내용은 artifact hub에서 chart에 대한 정보와 동일하다.
- charts: dependency chart 파일들이 해당 디렉토리 아래에 생기게 된다.
- template: k8s 리소스 템플릿이 보관되는 디렉토리
    - NOTES.txt: Chart를 설치 후 출력되는 내용을 정의한다.
    - *.yaml: 클러스터에 띄울 리소스 템플릿 파일
- **values.yaml**: 템플릿에 사용될 변수들을 모아놓은 파일

### values.yaml

직접 Chart를 만들어서 사용하는 것이 아닌 Artifact Hub에서 Chart를 받아서 배포하는 경우에 가장 중요한 것은 **values.yaml**이다.

Chart의 리소스 템플릿 파일들이 이미 구성되어 있을테니 values.yaml 파일만 입맛대로 수정하여 적용하면 원하는 차트를 배포할 수 있기 때문이다.

이전 포스팅에서 언급하였던 명령어들을 통해서 values를 수정한 값으로 배포하거나 몇 가지 플래그들만 바꿀 수 있다.

```
$ helm install apache bitnami/apache -f myValues.yaml
or
$ helm install apache bitnami/apache --set foo=bar
```

## Chart 작성

만약 본인이 차트를 만들었을 때는 어떻게 해야할까?🤷‍♂️

분명 `helm create chartname`을 통해 디렉토리를 구성했을 것이다. 그렇다면 다른 사람들이 만들어 놓은 차트와 같이 `templates` 폴더에 K8s 리소스 YAML 파일과 상위 폴더에 values.yaml 파일도 존재할 것이다.

정해진 스펙으로만 배포한다면 굳이 Template을 통해서 가변적인 값이 필요 없을 것이다.

하지만 **환경별**로 배포해야하거나 같은 차트를 여러 번 다르게 배포해야할 경우 매번 수정하는 것은 거의 불가능하다.

이 때 필요한 것이 **Template**이다. Template을 구성하는 방법에 대해 알아보자.

## Helm Template

K8s 리소스 파일을 작성하다보면 반복되는 형식도 있고 접두사나 접미사만 다른 경우, 한 값만 다른 경우가 생각보다 많다.

이런 경우에 모든 리소스들을 전부 YAML파일로 관리한다면 좋겠지만 **현실적으로 불가능**하다.

정해진 K8s 리소스 파일에 **변수**처럼 구멍을 뚫어놓고 언제나 변수를 채우기만 하면 필요할 때마다 유연하게 사용하면서 관리할 수 있다.

Helm에서 제공하는 이러한 기능을 **Template**이라고 한다.

위에서 이야기한 구조에서 Template에 필요한 부분은 `templates/`와 `values.yaml`이다.

~~직접 구성할 때 templates의 s를 자주 빼먹는다. 그냥 helm create 하자 😅~~

### Template 문법

- YAML 파일들은 모두 Camel 표기법이 아닌 Dash 표기법 `foo-bar.yaml`을 사용해야한다.
    - ex) my-deployment.yaml
- `{{ .foo.bar }}`와 같은 지시문을 사용하며 안에 들어가는 정해진 내장 객체도 있으며, `.Values.`와 같이 **values.yaml**에 의존하는 객체도 있다.
    - [Helm 내장 객체](https://helm.sh/docs/chart_template_guide/builtin_objects/)를 통해서 몇 개정도 알고 있으면 좋다.



```yaml
$ cat templates/deployment.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: {{ .Values.web.name }}
  labels:
    app.kubernetes.io/name: {{ .Values.web.name }}
    app.kubernetes.io/component: web
spec:
# ... (생략)
------
$ cat values.yaml
web:
  name: "my-test-web"
------
$ helm template . -f ./values.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: my-test-web
  labels:
    app.kubernetes.io/name: my-test-web
    app.kubernetes.io/component: web
spec:
# ... (생략)
```

자신이 만든 Template이 제대로 작동하는지 확인하기 위해서는 `helm template . -f ./my-values.yaml --debug`를 통해서 무엇이 문제인지 진단할 수 있다.

### 여담

Helm은 Go Template을 차용하였기 때문에 `{{ . }}` 문법이나 템플릿의 변수들은 Golang의 자료형을 사용한다.

![image](https://github.com/lee20h/blog/assets/59367782/6da28573-e8eb-436d-93f0-7b6a1256c992)

조만간 작성할 CI/CD 파트의 ArgoCD에서도 Helm을 이용한 방법을 포스팅할 예정이다.

---

## 레퍼런스
- [공식페이지](https://helm.sh/ko/docs/topics/charts/)
- [Spoqa 기술 블로그](https://spoqa.github.io/2020/03/30/k8s-with-helm-chart.html)
- [golang-template - chanlee blog](http://chanlee.github.io/2016/04/21/golang-template-package/)