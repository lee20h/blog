---
title: "Virtual Cpu"
date: 2023-09-28T01:02:24+09:00
tags:
  - tag
categories:
  - category
series:
  - series
published: false
---

### vCPU

GCP에서의 vCPU는 하이퍼스레드 1개를 의미하고 GKE와 연결하였을 떄 resource에서 의미하는 CPU는 호스트 OS의 스레드를 사용하는 시간의 비율을 뜻합니다.  
또한, Kubernetes에서의 Golang Application 안 `runtime.GOMAXPROCS()`은 가상화 환경의 하이퍼스레드 갯수로 지정됩니다.

따라서, 물리적인 코어 수가 아닌, OS나 가상화 환경에서의 보고하는 사용 가능한 쓰레드나 코어의 수로 지정한다.