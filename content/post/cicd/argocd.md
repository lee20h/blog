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

### Architecture

![image](https://github.com/lee20h/blog/assets/59367782/b8640f63-1e7f-4d4f-b3ff-5404acad7a45)

