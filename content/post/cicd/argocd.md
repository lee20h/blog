---
title: "ArgoCD에 대해 알아보자"
date: 2023-09-08T21:51:16+09:00
tags:
  - ArgoCD
categories:
  - cd
series:
  - cicd
published: true
---

## Argo 란?

[CNCF](https://www.cncf.io/projects/argo/)의 Graduated Project로, [Argo](https://argoproj.github.io/) 4가지의 오픈 소스 툴을 관리하는 프로젝트다.
공식 사이트에선 다음과 같이 소개한다.

> Open source tools for Kubernetes to run workflows, manage clusters, and do GitOps right.

kubernetes-native한 툴로, Workflow와 클러스터를 관리하고, GitOps를 사용할 수 있게 만들어주는 도구들이다. 

### GitOps란?

> GitOps uses Git repositories as a single source of truth to deliver infrastructure as code

Red Hat에서는 위와 같이 설명한다.  
Git 레포지토리를 단일 진실 공급원으로 사용하여 인프라를 코드로 제공하는 것을 GitOps라고 한다.

![image](https://github.com/lee20h/blog/assets/59367782/6fab1e2e-f33e-4545-adb3-de464edcfdb4)

- Argo Workflows
- **Argo CD**
- Argo Rollouts
- Argo Events

네 가지의 도구 중 이 포스트에서 다룰 도구는 Argo CD이다. 

## ArgoCD 란?

[Argo CD](https://argo-cd.readthedocs.io/en/stable/)를 설명하는 페이지에서는 다음과 같이 일컫는다.

> Argo CD is a declarative, GitOps continuous delivery tool for Kubernetes.

말 그대로 GitOps로 Kubernetes에 지속적으로 전달해주는 도구며, Kubernetes CD 툴 중 가장 많이 쓰여서 de facto standard라고 생각한다.

## [Architecture](https://argo-cd.readthedocs.io/en/stable/#features)

![image](https://github.com/lee20h/blog/assets/59367782/b8640f63-1e7f-4d4f-b3ff-5404acad7a45)

Argo CD는 어플리케이션을 지속적으로 모니터링하여 live 상태와 git repository에 선언된 desired 상태를 비교하는 kubernetes controller로 구현되어있습니다.  
만약, 배포된 어플리케이션이 live 상태와 선언된 desired 상태와 다르다면 `OutOfSync`로 나타냅니다. 다른 상태를 시각화하여 보고하며, 유저가 직접 수동으로 동기화하거나 자동으로 동기화할 수 있게 기능을 제공합니다.  
따라서 Git Repository를 수정함에 따라 원하는 상태로 바꿔서 배포가 가능합니다.

### Component

- API Server
  - Application 관리 및 상태 보고
  - sync, rollback, user-defined action
  - secret을 통해 kubernetes credentials 저장
  - RBAC을 통한 접근 제어
  - git webhook 이벤트의 리스너이자 주최자
  - UI나 CLI를 통해서 REST/gRPC 요청

- Repository Server
  - git repository URL
  - revision (branch, commit, tag)
  - application path
  - helm, kustomize, etc.와 같은 template

- Application Controller
  - Git repository와 현재 ArgoCD에서 배포된 Application을 서로 비교하며 상태를 보여주는 Kubernetes Controller
  - user-defined action (presync, sync, postsync hook) life cycle 관리

### [Feature](https://argo-cd.readthedocs.io/en/stable/#features)

- 특정환경 자동 배포
- helm, kustomize와 같은 템플릿 도구 지원
- 멀티 클러스터 배포 지원
- SSO 통합
- 멀티태넌시와 RBAC을 통해서 인증 기능
- git commit 기반으로 한 rollback
- application health check
- 자동 배포 및 수동 배포
- Web UI 실시간 제공
- Webhook 통합
- blue/green & canary 배포 지원
- Prometheus 메트릭

### [Core Concepts](https://argo-cd.readthedocs.io/en/stable/core_concepts/)

- Kubernetes CRD로 구현된 manifest 관리 도구인 Application
- Git repository를 target, Kubernetes에 배포된 application을 live로 하는 상태 관리
- Kubernetes workload 뿐 아니라, Configuration 까지 관리

## UI

![image](https://github.com/lee20h/blog/assets/59367782/1a4e9f41-cd07-475a-bd96-8f24574ee8f6)

Application 안에 정의된 manifest들을 기준으로 여러 리소스들을 관리한다. 이 때 리소스들은 전부 target reposiory와 비교하여 상태(Sync Status)를 정의한다.
또, Kubernetes에 배포된 리소스들의 상태를 App Health를 통해서 표현한다. 

- Unknown
- Missing
- Degraded
- Healthy
- Progessing
- Suspended

## CLI

### Installation

```shell
$ brew install argocd
```

brew를 이용하여 Argo CD cli를 설치할 수 있다. 다른 cli 툴들과는 다르게 서버와 같이 패키징되어 있지 않고 cli만 다운로드 받을 수 있다.  
그 이유로는, Kubernetes와 함께 사용해야하므로 대부분의 서버 배포는 Kubernetes 내부에 서버가 배포되는 방식이다.  
공식 문서에서 안내하는 [ArgoCD Manifest](https://raw.githubusercontent.com/argoproj/argo-cd/stable/manifests/core-install.yaml)를 그대로 배포하게 되면 UI와 서버를 배포할 수 있다.

### Connection

처음 연결할 때는 다음과 같은 명령어로 연결 할 수 있으며, 혹은 설정에서 admin의 비밀번호를 지정할 수 있다.

```shell
argocd admin initial-password
```

그 이후에는 ArgoCD 서버에 직접 로그인해서 명령어를 통해 API를 요청할 수 있다. 만약, DNS를 연결하지 않았더라면, `kubectl port-forward`를 통해서 localhost로 접근해야한다.

```shell
argocd login <AROGCD_SERVER>
```

### MultiCluster

만약 MultiCluster를 운용하면서 하나의 ArgoCD로 중앙 관리를 한다면, 다음 명령어를 통해 연결하면 된다.

```shell
argocd cluster add <CLUSTER_NAME>
```

여기서 사용하는 cluster의 이름은 `~/.kube/config`에 들어있는 context를 기준으로 사용할 수 있다.


Argo CD cli에 대해서는 다음 포스팅에서 더 자세하게 다뤄볼려고 한다.