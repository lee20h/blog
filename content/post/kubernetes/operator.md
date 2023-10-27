---
title: "Kubernetes Operator에 대하여"
date: 2023-10-23T23:08:47+09:00
tags:
  - kubernetes
  - operator
  - pattern
categories:
  - kubernetes
published: true
---

# Operator(오퍼레이터) 패턴에 대하여

오퍼레이터 패턴은 Custom Resource를 이용하여 애플리케이션 및 해당 컴포넌트를 관리하는 쿠버네티스 확장이다. 

쿠버네티스 원칙 중 컨트롤 루프를 따른다. 여기서 컨트롤 루프에 대해서 가볍게 알아보자.

## Control loop

컨트롤 루프란 일반적으로 자동 제어 시스템에서 사용되는 용어로, 특정 시스템이나 프로세스의 동작을 제어하는데 사용된다. 이것이 원하는 결과를 얻기 위해 지속적으로 조정하고 모니터링하는 과정을 포함한다.  

여기에서의 기본 개념은 피드백 매커니즘이 중요한 역할을 한다.

- Sensor: 시스템의 현재 상태 감지
- Controller: 센서로부터의 입력 기반으로 결정을 내림. 시스템의 상태가 원하는 목표치나 설정값과 다른지 평가
- Actuator: 컨트롤러의 지시에 따라 물리적인 조치를 취함

쿠버네티스 오퍼레이터도 유사하게 작동하며 각 역할을 하는 구성 요소들이 존재한다.

- Kubernetes API Server: 센서, 액추에이터 역할
- Kubernetes Operator: 컨트롤러 역할

1. 쿠버네티스 API 서버가 특정 커스텀 리소스에 대한 변경 사항 지속적 감시
2. 오퍼레이터는 커스텀 리소스 변경되거나 요청이 발생하면, 분석 후 내부 컨트롤러 로직을 실행
3. 컨트롤러 로직에서 애플리케이션의 상태를 조정하여 원하는 상태로 만들기 위해 쿠버네티스 API 서버에 요청

## Controller

![image](https://github.com/lee20h/blog/assets/59367782/cc8aac0b-d80d-42db-9c4c-ffe706e0b24e)

- API 서버를 통한 제어
  - Job Controller
- 직접 제어
  - 클러스터 외부의 것들을 컨트롤 할 경우
  - [autoscaler](https://github.com/kubernetes/autoscaler/)

### Design

의도한 상태와 현재 상태 즉, Desired state와 Current state를 비교하여 Desired state로 일정하게 유지시키기 위한 작업들을 하도록 설계 되어있다.

## Operator SDK

> The Operator SDK makes it easier to build Kubernetes native applications, a process that can require deep, application-specific operational knowledge.

쿠버네티스 오퍼레이터를 쉽게 만들고 적용시키기 위해서 [Operator SDK](https://sdk.operatorframework.io/)라는 프레임워크가 존재한다.

- Ansible
- Go
- Helm

3가지의 방법을 통해서 오퍼레이터를 구현할 수 있게 제공한다.

### install

[install page](https://sdk.operatorframework.io/docs/installation/#install-from-homebrew-macos)

Homebrew, Github, master 브랜치에서 install 총 3가지의 방법으로 설치할 수 있다.

```shell
$ brew install operator-sdk
```

macOS에서는 위와 같이 Homebrew로 바로 설치하여 사용할 수 있다.

여기서는 Golang을 이용하여 기본적인 프레임워크 사용법에 대해서 알아볼려고 한다.

### Go

원하는 디렉토리에서 다음과 같이 실행하면 바로 Go를 이용해서 오퍼레이터를 생성하기 위한 준비가 끝난다.

```shell
$ operator-sdk init --domain example.com --repo github.com/example/operator
$ operator-sdk create api --group cache --version v1alpha1 --kind CustomOperator --resource --controller
```

원하는 값을 넣어주면 Go module과 Kubernetes CRD, 필요한 RBAC, Custom Resource를 관리해주는 Manager 애플리케이션까지 한 레포지토리에서 관리할 수 있도록 제공해준다.  

또한, Makefile에 많은 기능들이 정의되어 있어서 읽어보면 개발 및 배포를 편하게 진행할 수 있는 유틸성도 존재한다.

main.go를 통해서 실행하면 `sigs.k8s.io/controller-runtime` 라이브러리를 통해서 애플리케이션이 로컬에서 바라보고 있는 디폴트 쿠버네티스 클러스터와 연결해서 테스트 혹은 디버깅 또한 쉽게 할 수 있다.

## ref

- https://kubernetes.io/ko/docs/concepts/extend-kubernetes/operator/
- https://kubernetes.io/ko/docs/concepts/architecture/controller/
