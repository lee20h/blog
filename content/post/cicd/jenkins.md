---
title: "Jenkins에 대해 알아보자"
date: 2022-01-13
tags:
  - jenkins
categories:
  - ci
series:
  - cicd
published: true
---


![image](https://github.com/lee20h/blog/assets/59367782/f59af189-0a61-48c9-8b30-25befa85178c)


## Jenkins 란?


[CD Foundation](https://cd.foundation)의 프로젝트 중 하나인 Jenkins는 오래 전부터 사랑받아온 CI/CD Tool 중 하나이다.

[공식사이트](https://www.jenkins.io)에서는 Jenkins를 다음과 같이 설명한다.

> 업계를 선도하는 오픈 소스 자동화 서버인 Jenkins는 프로젝트의 구축, 도입 및 자동화를 지원하는 수백 개의 플러그인을 제공합니다.

Jenkins는 Java 언어로 작성되었으며 CI Tool 춘추전국시대인 현재 그래도 사랑받고 있으며 10년이 넘는 기간동안 생태계가 많이 발전하여 유연하게 사용이 가능하다.

[플러그인](https://plugins.jenkins.io)을 통해 쿠버네티스에 직접 배포하거나 CI 이후 CD를 준비하는 툴로 쓰이기도 한다.

![image](https://github.com/lee20h/blog/assets/59367782/f176310b-76d6-4cbc-90f7-6e32bbcfc12f)

## Job

Jenkins에서 사용하는 작업의 단위는 Job이라고 한다. Job의 종류에는 여러가지가 있는데 플러그인을 통해서 추가될 수도 있다.

Job을 무엇을 고르냐에 따라 설정값도 달라지며, 여러 브랜치 혹은 한 형상관리 프로젝트에 대해서 진행할 수도 있다.

여러 스텝을 묶어 한 Job에서 실행할 수 있는 Pipeline도 존재하며 Pipeline은 Groovy 언어로 작성한다.

생각보다 자유도가 높으며, 설정의 플러그인 관리를 통해서 필요한 플러그인을 추가하거나 찾아볼 수 있다. 이를 통해 **도커**나 **쿠버네티스** 혹은 **클라우드 서비스**들과 연결이 가능하며 유사시를 위한 백업도 가능하다.

![image](https://github.com/lee20h/blog/assets/59367782/43c71db2-2e2c-4683-b720-563e56b9441c)

## 설정

![image](https://github.com/lee20h/blog/assets/59367782/027722c1-b5e6-4ea6-8b61-f0132035f7c0)

시스템 설정과 환경변수를 설정함으로 Jenkins을 사용할 때 필요한 부분을 설정할 수 있다. 이 두 곳에서 추가된 대부분의 플러그인들의 설정을 등록한다.

### Security

![image](https://github.com/lee20h/blog/assets/59367782/25e9e331-fece-45c2-ac1a-633ae7580172)

여기선 주로 보안과 관련된 설정을 할 수 있다. Jenkins 전반에 관한 보안 설정이나, Job에서 사용하는 Credentials을 등록하거나 삭제할 수 있다.

이 외에는 크게 만질 일도 없고 볼 일도 없다고 생각한다.

---

필자는 Jenkins를 쿠버네티스에서 **Dind** 방식으로 사용하여 Pipeline을 다음과 같이 구성했다.

```yaml
pipeline {
    agent {
        kubernetes {
            yaml '''
apiVersion: v1
kind: Pod
metadata:
  annotations:
    sidecar.istio.io/inject: "false"
spec:
  containers:
  - name: git
    image: alpine/git
    tty: true
    command: ["cat"]
            '''
		}
        stage('Checkout') {
            steps {
                container('git') {
                    ...
                }
            }
        }
    }
}
```

Job을 실행하는 주체인 **Agent**를 쿠버네티스 Pod로 삼아 모든 작업들을 Pod에게 맡겨서 진행하도록 하였다. 작업이 실행될 때마다 Pod 하나가 작업 하나를 맡아서 진행하므로 쿠버네티스에는 Pod가 하나씩 생기고 완료 후 사라지는 모습을 보인다.

이 때 필요한 컨테이너들을 선언한 뒤 steps에서 컨테이너들을 골라서 필요한 명령어들을 실행시키도록 하였다. 이런 방식이 좋았던 점은 다른 이미지들을 통해 컨테이너를 구성하다보니 유연하게 Pipeline을 구성할 수 있었다.

굳이 **Dind**를 이용한 이유는 쿠버네티스 네이티브하게 Jenkins를 이용하고 싶었고, Jenkins를 Helm으로 배포하면서 Jenkins의 설정들을 [jcasc 플러그인](https://www.jenkins.io/projects/jcasc)을 통해 yaml로 관리할 수 있었다는 점이였다.

> 여기서 Dind를 이용할 때, **/var/run/docker.sock** 파일을 이용하였다.
Gitlab에서는 Dind를 Dind 방식과 소켓 방식 두가지를 모두 Dind라고 표현하여 Dind라고 적었지만, 명확히 보면 Dood라고 할 수 있다.

또한, 쿠버네티스에서 스케쥴링되는 Pod이므로 쿠버네티스에서 사용하는 것들을 모두 붙여서 사용할 수 있다는 것이 유틸리티를 높일 수 있었다.

>Dind에 대해서는 Docker 포스팅 다음에 더 깊숙하게 포스팅할 예정이다.

## 맺으며..

Jenkins를 사용하는 방법은 매우 다양해서 모두 소개하기 어렵지만, 여러 플러그인들을 찾아서 접목하다보면 나만의 사용 방법이 생길 수 있을 것이다.

필자는 쿠버네티스에 배포하여 위와 같은 방법을 사용하였지만, 클라우드든 온프렘이든 각자의 환경에 맞게 사용하면 되는 것이라고 생각한다.

여러 방식으로 이용될 수 있어서 **정답은 없는 것 같다.**

![image](https://github.com/lee20h/blog/assets/59367782/3c2dcf57-5a39-42a4-8a99-f225202249e1)

---

## 레퍼런스

- [공식페이지](https://www.jenkins.io/)
- [공식 플러그인 페이지](https://plugins.jenkins.io)
- **~~경험~~**