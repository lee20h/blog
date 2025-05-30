---
title: "Docker에 대해 알아보자"
date: 2022-02-06
tags:
  - docker
  - container
  - virtualization
categories:
  - cloud
series:
  - cloud
publishResources: true
---

IT 업계에 종사하는 사람들이라면 한번 쯤은 들어보거나 봤을 법한 고래 캐릭터와 Docker를 [공식페이지](https://www.docker.com/)에서 대문글로 이렇게 표현한다.

> Developers Love Docker. Businesses Trust It.

개발자들이 사랑한다는 Docker에 대해 알아보자.

## Docker란?

공식 페이지에서는 다음과 같이 설명한다.

> Docker는 반복적이고 일상적인 구성 작업을 없애고 데스크톱 및 클라우드와 같은 빠르고 간편하며 이식 가능한 애플리케이션 개발을 위해 개발 라이프사이클 전반에 걸쳐 사용됩니다. Docker의 포괄적인 엔드 투 엔드 플랫폼에는 전체 애플리케이션 제공 라이프사이클에 걸쳐 함께 작동하도록 설계된 UI, CLI, API 및 보안이 포함됩니다.

항상 공식 설명란은 한번에 이해하기 어려운 부분이 많은 것 같다. 따라서 나의 방식대로 설명하려고 한다.

![image](https://github.com/lee20h/blog/assets/59367782/3fb7f963-a245-477d-a632-05521d5b6b90)

컴퓨터에서 사용하는 알집과 같은 압축 프로그램을 생각해보자. 압축은 일련의 파일들을 하나의 파일로 묶어주는 역할을 한다. 이렇게 만들어진 압축 파일은 압축을 풀게되면 언제 어디서나 같은 파일들을 가지고 있다.

그렇다면 압축파일을 가지고 있으면 언제나 같은 파일들을 받을 수 있다고 생각이 든다. 이 부분을 개발적인 측면으로 풀어보자.

어떤 서버가 있다면 서버를 구성하는 소스 코드들이 존재할 것이다. 이 소스 코드들과 구성 요소들을 압축하듯 박제한 것을 **이미지**라고 부른다. 당연히 이미지를 통해서 나오는 결과물은 서버와 동일하다.

이러한 이미지들을 인터넷(**Docker Hub**)에 올려서 공개 혹은 비공개로 관리도 가능하다. 즉, 직접 만들어서 자기만 사용하거나 다른 사람들이 만든 이미지를 이용할 수 있다는 말이다.

이미지를 만들고 실행시키는 도구가 **Docker**이다.

즉, Docker를 이용하여 같은 이미지를 사용한다면 언제 어디서나 누구나 같은 결과를 가진 서버를 쉽게 만들 수 있다는 것이다. 이러한 이유로 개발자들에게 사랑받는 도구들 중 하나로 자리 매김할 수 있었다.

## 컨테이너?🤷‍♂️

"도커가 사용되는 이유는 대충 알겠는데 사용하려고 검색하니까 **컨테이너**라는게 나오는데 이게 뭐예요?"

막상 사용하려하니까 위와 같은 궁금증이 생길 것이다. 이제 조금 이해하려했는데 새로운 단어가 등장하였다.

그렇다면 컨테이너에 대해서도 알아보자.

![image](https://github.com/lee20h/blog/assets/59367782/55266e0e-44e0-4ff6-b3e7-f17eaa740bc2)

우리가 아는 컨테이너는 화물들을 담아 운송할 때 쓰이는 상자이다. 컨테이너들을 배에 잔뜩 싣고 무역을 하는 모습은 뉴스에서도 자주 보는 모습이다. 화물들을 담고 어딘가로 운송하는 모습이 포스팅 대문 사진과 비슷하지 않은가?

![image](https://github.com/lee20h/blog/assets/59367782/59cde6ba-e5d3-44d1-8ef7-0d7d1ba7cd38)

따라서 컨테이너는 무언갈 담을 수 있고, 우리는 위에서 담을만한 것에 대해 알아보았다. 바로 **이미지**다.

우리가 소스코드와 같은 것들을 묶어서 만든 **이미지**를 **컨테이너**라는 곳에 넣을 수 있는 것으로 보인다. 또한 컨테이너들을 각각 담는 것을 공유하지 않을 것으로 예상된다. 왜냐하면 컨테이너에는 하나의 화물 종류를 넣어야 헷갈리는 일이 없을 것 같다.

따라서 이미지를 컨테이너에 넣게되면, 다른 컨테이너들과 의존관계가 없는 독립된 환경에 놓이게 되고, 같은 이미지를 넣게 되면 항상 같은 결과를 보일 것이다.

> 실제로는 이미지를 실행하면 컨테이너가 생성된다. 따라서 컨테이너란 이미지를 구성하는 것들을 구체화하여 실행하고 있는 가상 환경이다.
> ![image](https://github.com/lee20h/blog/assets/59367782/83e453fe-631a-437a-8ae4-2fb65a471d3f)
[(Docker architecture 공식페이지)](https://docs.docker.com/get-started/overview/)

이 말은 즉, 언제나 어디에서도 같은 이미지를 통해서 만들어진 컨테이너는 항상 같은 산출물이 나온다는 말과 같다.

### 기술적 측면

기술적으로 설명하자면, 컨테이너는 리눅스 커널의 네임스페이스와 cgroup(컨트롤 그룹)을 이용하여 프로세스를 격리하고 사용할 수 있는 리소스(CPU, Memory 등)을 제어할 수 있도록 운영체제 가상화를 통해 이루어져있다.

또한, 이전에 사용되었던 가상머신과 많은 비교가 되고 있다.

가상머신은 하이퍼바이저를 활용하여 물리적 하드웨어를 가상화하게 되는데 이 때, 애플리케이션과 연관된 라이브러리 및 종속 항목과 함께 OS가 실행해야 하는 하드웨어의 가상 사본인 게스트 OS가 포함되어 상당히 무겁다. 이러한 이유로 가상머신보다 컨테이너가 리소스나 이식성, 플랫폼 독립성 등 장점을 가진다.

![image](https://github.com/lee20h/blog/assets/59367782/dae6d1d3-2767-4a46-893b-72eab4fbe730)

이러한 이유로 컨테이너 기술은 많은 도구들의 채택을 받아서 사용되고 있고, 컨테이너 기술을 사용한 도구들 중에서 개발자들이 많이 사용하고 있는 도커에 대해서 알아보았다.

## 맺으며..

최대한 기술적인 내용보다 직접 겪어볼만한 내용으로 이해에 도움이 되었으면 하는 바람으로 포스팅해보았다.

처음 도커를 접하는 분들이 바로 기술적인 내용과 명령어를 배우기 전 간단하게나마 이해에 도움이 되었으면 한다. 🤞

---

## 레퍼런스

- [공식페이지](https://www.docker.com/)
- [Docker(도커)란? - 레드햇](https://www.redhat.com/ko/topics/containers/what-is-docker)
- [Docker가 뭐고 왜 쓰는건가요? - 얄코](https://www.yalco.kr/08_docker/)
- [컨테이너란? - IBM](https://www.ibm.com/kr-ko/cloud/learn/containers)
- [containers-vs-vms - 레드햇](https://www.redhat.com/ko/topics/containers/containers-vs-vms)