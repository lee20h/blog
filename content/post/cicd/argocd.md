---
title: "ArgoCD에 대해 알아보자"
date: 2023-09-08T21:51:16+09:00
tags:
  - ArgoCD
categories:
  - cd
series:
  - cicd
published: false
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

말 그대로 GitOps로 Kubernetes에 지속적으로 전달해주는 도구며, 가장 많이 사용되다보니 Kubernetes CD 툴 중 de facto standard라고 생각한다.

## Architecture

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

### Feature

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