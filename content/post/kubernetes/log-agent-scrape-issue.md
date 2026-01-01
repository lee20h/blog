---
title: "Kubernetes Log Agent Slab 이슈에 대해서 알아보자"
date: 2026-01-01T19:08:12+09:00
tags:
  - kubernetes
  - container
  - log
categories:
  - kubernetes
publishResources: true
---

쿠버네티스 환경에서 로그 수집 에이전트를 운영하다 보면 예상치 못한 메모리 이슈를 겪을 수 있습니다. 이 글에서는 Alloy를 사용하여 `/var/log/pods`를 watch하는 과정에서 발생한 slab memory 이슈와 그 해결 과정을 살펴보겠습니다.

# 문제 상황

## 운영 환경 설정

운영 환경에서는 **Alloy**를 사용하여 Kubernetes 컨테이너 로그를 수집하고 있었습니다. Alloy는 `/var/log/pods` 디렉토리를 watch하여 컨테이너 런타임이 생성하는 모든 로그 파일을 모니터링하는 방식이었습니다.

```text
┌─────────────────────────────────────────────────────────────┐
│                    Log Collection Setup                     │
│                                                             │
│  ┌──────────────┐                                           │
│  │              │                                           │
│  │    Alloy     │───▶ Watch /var/log/pods/**/*.log          │
│  │              │    (Deep directory structure)             │
│  └──────────────┘                                           │
│                                                             │
│  /var/log/pods/                                             │
│  └── default_mypod_1234567890abcdef/                        │
│      └── mycontainer/                                       │
│          ├── 0.log                                          │
│          ├── 1.log                                          │
│          └── ...                                            │
└─────────────────────────────────────────────────────────────┘
```

## 문제 증상

운영 중 Node Exporter에서 수집하는 파드 메모리 메트릭이 지속적으로 우상향하는 그래프를 그리고 있었습니다.

```text
Pod Memory Usage (Node Exporter)
│      ╱
│     ╱
│    ╱  
│   ╱    
│  ╱      
│ ╱        
│╱          
└──────────────────────────────▶ Time
```

하지만 파드 내부의 **RSS (Resident Set Size) 메모리**는 증가하지 않고 있었습니다. 이는 매우 이상한 현상이었습니다.

- **Node Exporter 메트릭**: 계속 증가
- **파드 내부 RSS**: 변화 없음
- **실제 프로세스 메모리**: 정상

# Slab Memory란?

## Slab Allocation의 개념

**Slab Allocation**은 리눅스 커널에서 사용하는 메모리 관리 기법입니다. 커널 객체(예: inode, dentry, 파일 디스크립터 등)를 빠르게 할당하고 해제하기 위해 미리 할당된 메모리 풀을 사용합니다.

```text
┌─────────────────────────────────────────────────────────────┐
│                    Slab Allocator                           │
│                                                             │
│  ┌──────────┐  ┌──────────┐  ┌──────────┐                   │
│  │          │  │          │  │          │                   │
│  │  Slab 1  │  │  Slab 2  │  │  Slab 3  │                   │
│  │          │  │          │  │          │                   │
│  │[obj][obj]│  │[obj][obj]│  │[obj][obj]│                   │
│  │[obj][obj]│  │[obj][obj]│  │[obj][obj]│                   │
│  └──────────┘  └──────────┘  └──────────┘                   │
│                                                             │
│  - Dentry Cache                                             │
│  - Inode Cache                                              │
│  - File System Cache                                        │
└─────────────────────────────────────────────────────────────┘
```

## Dentry Cache

**Dentry (Directory Entry)**는 리눅스 VFS (Virtual File System)에서 디렉토리 항목을 나타내는 커널 객체입니다. 파일 시스템 성능을 향상시키기 위해 커널은 자주 접근하는 디렉토리 항목을 메모리에 캐시합니다. 이 캐시를 **dentry cache**라고 부릅니다.

```text
┌─────────────────────────────────────────────────────────────┐
│                    Dentry Cache                             │
│                                                             │
│  /var/log/pods/                                             │
│  ├── default_pod1_abc123/                                   │
│  │   ├── container1/                                        │
│  │   │   ├── 0.log                                          │
│  │   │   └── 1.log                                          │
│  │   └── container2/                                        │
│  │       └── 0.log                                          │
│  └── default_pod2_def456/                                   │
│      └── container1/                                        │
│          └── 0.log                                          │
│                                                             │
│  각 경로 구성 요소마다 dentry 객체 생성                             │
│  → 깊은 디렉토리 구조 = 많은 dentry 객체                           │
└─────────────────────────────────────────────────────────────┘
```

**Dentry cache**는 파일 경로 해석(path resolution)을 빠르게 하기 위한 캐시입니다. 많은 파일을 watch하거나 깊은 디렉토리 구조를 탐색할 때 이 캐시가 급증할 수 있으며, 이는 slab memory 사용량 증가로 이어집니다.

# cgroup과 Slab Memory의 관계

## cgroup v2의 메모리 회계

cgroup v2에서는 커널 메모리도 cgroup에 할당됩니다. 이는 사용자 공간 메모리뿐만 아니라 **커널이 사용하는 메모리(slab memory 포함)**도 해당 cgroup에 속하게 됩니다.

```text
┌─────────────────────────────────────────────────────────────┐
│                    Cgroup Memory Accounting                 │
│                                                             │
│  Pod Cgroup                                                 │
│  ├── User Space Memory (RSS)                                │
│  │   └── Process memory                                     │
│  ├── Kernel Memory (Slab)                                   │
│  │   ├── Dentry cache                                       │
│  │   ├── Inode cache                                        │
│  │   └── File system cache                                  │
│  └── Page cache                                             │
│                                                             │
│  Node Exporter는 cgroup 전체 메모리를 측정                       │
│  → RSS + Slab Memory = Total Memory                         │
└─────────────────────────────────────────────────────────────┘
```

## 왜 커널에서 관리하지 않을까?

일반적으로 slab memory는 커널에서 관리한다고 생각하지만, cgroup v2에서는 **프로세스가 속한 cgroup에 slab memory가 할당**됩니다. 이는 다음과 같은 이유 때문입니다:

1. **리소스 격리**: 각 파드가 사용하는 커널 리소스를 정확히 추적
2. **메모리 제한**: cgroup memory limit에 slab memory도 포함
3. **공정한 할당**: 어떤 파드가 커널 리소스를 많이 사용하는지 파악

# 왜 RSS는 증가하지 않는데 메트릭은 증가하는가?

## 메모리 측정 방식의 차이

```text
┌─────────────────────────────────────────────────────────────┐
│              Memory Measurement Comparison                  │
│                                                             │
│  RSS (Resident Set Size)                                    │
│  ├── 프로세스가 실제로 사용하는 물리 메모리                           │
│  ├── 사용자 공간 메모리만 포함                                    │
│  └── /proc/<pid>/status의 RssAnon, RssFile                   │
│                                                             │
│  Cgroup Memory (Node Exporter)                              │
│  ├── cgroup 전체 메모리 사용량                                   │
│  ├── RSS + Kernel Memory (Slab)                             │
│  └── /sys/fs/cgroup/memory.stat                             │
│                                                             │
│  차이점:                                                      │
│  - RSS: 사용자 공간만                                           │
│  - Cgroup: 사용자 공간 + 커널 공간 (slab 포함)                    │
└─────────────────────────────────────────────────────────────┘
```

## 실제 상황

```bash
# 파드 내부에서 확인
$ cat /proc/self/status | grep Rss
RssAnon:     10240 kB    # 사용자 공간 메모리
RssFile:     2048 kB     # 파일 매핑 메모리
RssShmem:    0 kB

# Node Exporter가 측정하는 메모리
$ cat /sys/fs/cgroup/memory.stat
cache 0
rss 12288
slab 524288    # ← 이 부분이 계속 증가!
```

**RSS는 증가하지 않지만, slab memory가 계속 증가**하여 Node Exporter 메트릭이 우상향하는 것입니다.

# Alloy의 파일 Watch와 Dentry Cache 급증

## /var/log/pods의 깊은 디렉토리 구조

Kubernetes는 컨테이너 로그를 다음과 같은 구조로 저장합니다:

```bash
/var/log/pods/
└── default_mypod_1234567890abcdef/    # 네임스페이스_파드명_UID
    └── mycontainer/                     # 컨테이너명
        ├── 0.log                        # 재시작 횟수별 로그
        ├── 1.log
        └── 2.log
```

이 구조는 **3단계 깊이**를 가지며, 각 경로 구성 요소마다 dentry 객체가 생성됩니다.

## 파일 Watch와 Dentry 생성

Alloy가 `/var/log/pods/**/*.log`를 watch할 때:

1. **디렉토리 탐색**: 모든 하위 디렉토리를 재귀적으로 탐색
2. **Dentry 생성**: 각 경로 구성 요소마다 dentry 객체 생성
3. **inotify/fanotify**: 파일 변경 감지를 위한 커널 구조체 생성
4. **Dentry cache 증가**: 빠른 경로 해석을 위한 캐시 항목 증가

```text
┌─────────────────────────────────────────────────────────────┐
│              Dentry Cache Growth                            │
│                                                             │
│  Pod 수: 100개                                               │
│  Container 수: 200개                                         │
│  Log 파일 수: 600개 (재시작 포함)                                │
│                                                             │
│  Dentry 객체 수:                                              │
│  - /var/log/pods: 1                                         │
│  - Pod 디렉토리: 100                                          │
│  - Container 디렉토리: 200                                    │
│  - Log 파일: 600                                             │
│  - Total: ~900개                                             │
│                                                             │
│  각 dentry 객체는 slab memory에 저장                            │
│  → Slab memory 급증!                                         │
└─────────────────────────────────────────────────────────────┘
```

## 컨테이너 런타임의 로그 파일 생성

컨테이너 런타임(containerd, Docker 등)은 컨테이너가 재시작될 때마다 새로운 로그 파일을 생성합니다. 이로 인해:

- 새로운 디렉토리 항목 생성
- 새로운 dentry 객체 생성
- Dentry cache 증가
- Slab memory 증가

# 이로 인해 야기되는 문제들

## 1. 메모리 Limit 초과로 인한 메트릭 오류

```text
┌─────────────────────────────────────────────────────────────┐
│              Memory Limit Exceeded in Metrics               │
│                                                             │
│  Pod Memory Limit: 512Mi                                    │
│  ├── RSS: 100Mi (정상)                                       │
│  ├── Slab: 450Mi (급증!)                                     │
│  └── Total (cgroup): 550Mi → Limit 초과!                     │
│                                                             │
│  실제 상황:                                                   │
│  - RSS는 여유있지만 cgroup 메모리 사용량이 limit 초과               │
│  - Node Exporter 메트릭에서 limit 초과로 표시                    │
│  - 실제 프로세스 메모리는 정상이지만 메트릭만 비정상                    │
│  - OOM kill은 발생하지 않음 (RSS가 낮기 때문)                      │
└─────────────────────────────────────────────────────────────┘
```

**문제점:**
- RSS는 여유가 있지만 cgroup memory 사용량이 limit을 초과
- Node Exporter 등 메트릭 수집 도구에서 limit 초과로 표시
- 실제 프로세스 메모리는 정상이지만 모니터링 시스템에서 비정상으로 감지
- OOM kill은 발생하지 않지만, 메트릭 기반 알람이나 대시보드에서 문제로 표시됨

## 2. 커널 메모리 압박 및 시스템 전체 영향

Slab memory는 **커널 메모리**를 사용하므로, 단순히 파드의 메트릭 문제를 넘어서 시스템 전체에 악영향을 줄 수 있습니다.

```text
┌─────────────────────────────────────────────────────────────┐
│              Kernel Memory Pressure                         │
│                                                             │
│  System Memory: 32Gi                                        │
│  ├── User Space: 24Gi                                       │
│  ├── Kernel Space: 8Gi                                      │
│  │   ├── Slab (Reclaimable): 2Gi                            │
│  │   │   └── Dentry cache: 1.5Gi (급증!)                     │
│  │   └── Slab (Unreclaimable): 1Gi                          │
│  └── Available: 6Gi                                         │
│                                                             │
│  문제점:                                                      │
│  - 커널 메모리 풀 고갈                                           │
│  - 다른 프로세스의 커널 객체 할당 실패 가능성                         │
│  - 메모리 회수(reclaim) 오버헤드 증가                             │
│  - 시스템 전체 성능 저하                                         │
└─────────────────────────────────────────────────────────────┘
```

### 커널 메모리 회수(Reclaim) 문제

리눅스 커널은 메모리 압박 상황에서 slab cache를 회수하려고 시도하지만, dentry cache 같은 reclaimable slab도 즉시 회수되지 않을 수 있습니다.

```text
┌─────────────────────────────────────────────────────────────┐
│              Memory Reclamation Process                     │
│                                                             │
│  1. Page Cache 회수                                          │
│     └── Clean pages 즉시 삭제                                 │
│                                                             │
│  2. Slab Cache 회수                                          │
│     ├── Reclaimable: dentry, inode cache                    │
│     │   └── vfs_cache_pressure에 따라 회수 속도 결정             │
│     └── Unreclaimable: 활성 커널 객체                          │
│         └── 회수 불가능                                        │
│                                                             │
│  3. Swap (필요시)                                             │
│     └── Anonymous pages를 디스크로 스왑                         │
└─────────────────────────────────────────────────────────────┘
```

**문제점:**
- **커널 메모리 풀 고갈**: Slab memory가 커널 메모리를 과도하게 사용하면 다른 커널 객체 할당이 실패할 수 있습니다.
- **메모리 회수 오버헤드**: `vfs_cache_pressure`가 기본값(100)일 때도 dentry cache 회수는 시간이 걸리며, 이 과정에서 CPU 오버헤드가 발생합니다.
- **시스템 전체 성능 저하**: 커널 메모리 압박은 파일 시스템 성능, 네트워크 성능 등 전반적인 시스템 성능에 영향을 줍니다.
- **다른 프로세스 영향**: 같은 노드의 다른 파드나 시스템 프로세스도 커널 메모리 풀을 공유하므로, 한 파드의 slab memory 증가가 다른 프로세스에 영향을 줄 수 있습니다.

### vfs_cache_pressure와의 관계

`vfs_cache_pressure`는 커널이 dentry와 inode cache를 회수하는 속도를 제어합니다.

```bash
# 현재 설정 확인
$ cat /proc/sys/vm/vfs_cache_pressure
100  # 기본값

# 값의 의미:
# - 0: dentry/inode cache를 거의 회수하지 않음 (OOM 위험)
# - 100: 다른 cache와 공정한 비율로 회수 (기본값)
# - 200+: dentry/inode cache를 더 적극적으로 회수
```

**주의사항:**
- `vfs_cache_pressure`를 0으로 설정하면 dentry cache가 회수되지 않아 OOM 상황이 발생할 수 있습니다.
- 하지만 값을 높여도 dentry cache가 즉시 회수되는 것은 아니며, 회수 과정 자체가 오버헤드를 발생시킵니다.
- 근본적인 해결책은 dentry cache 자체를 줄이는 것입니다.

## 3. 스케줄링 문제

Kubernetes 스케줄러는 Node Exporter 메트릭을 기반으로 노드의 메모리 사용량을 판단합니다.

```text
┌─────────────────────────────────────────────────────────────┐
│              Scheduling Issue                               │
│                                                             │
│  Node Memory: 8Gi                                           │
│  ├── Allocated: 6Gi (Node Exporter 기준)                     │
│  │   ├── 실제 RSS: 4Gi                                       │
│  │   └── Slab: 2Gi (과다 측정)                                │
│  └── Available: 2Gi (실제로는 4Gi 여유)                        │
│                                                             │
│  결과:                                                       │
│  - 스케줄러가 노드를 메모리 부족으로 판단                             │
│  - 실제로는 여유가 있음에도 파드 스케줄링 실패                         │
│  - 클러스터 리소스 활용률 저하                                     │
└─────────────────────────────────────────────────────────────┘
```

## 4. 모니터링 혼란

```text
┌─────────────────────────────────────────────────────────────┐
│              Monitoring Confusion                           │
│                                                             │
│  Grafana Dashboard                                          │
│  ├── Pod Memory (Node Exporter): 500Mi                      │
│  ├── Pod RSS (cAdvisor): 100Mi                              │
│  └── 차이: 400Mi (어디서 온 메모리?)                             │
│                                                             │
│  문제점:                                                      │
│  - 알람 기준 설정 어려움                                         │
│  - 실제 메모리 사용량 파악 불가                                    │
│  - 리소스 계획 수립 어려움                                        │
└─────────────────────────────────────────────────────────────┘
```

# 해결 방법: /var/log/containers로 전환

## 플랫한 디렉토리 구조

Kubernetes는 `/var/log/containers` 디렉토리에 **심볼릭 링크**를 제공합니다. 이 디렉토리는 **플랫한 구조**를 가지고 있습니다.

```bash
# /var/log/pods (깊은 구조)
/var/log/pods/
└── default_mypod_1234567890abcdef/
    └── mycontainer/
        └── 0.log

# /var/log/containers (플랫한 구조)
/var/log/containers/
└── mypod_default_mycontainer-abcdef1234567890.log
    → /var/log/pods/default_mypod_1234567890abcdef/mycontainer/0.log
```

## Dentry Cache 감소

플랫한 구조로 전환하면:

```text
┌─────────────────────────────────────────────────────────────┐
│              Dentry Cache Reduction                         │
│                                                             │
│  Before (/var/log/pods):                                    │
│  - Pod 디렉토리: 100개                                         │
│  - Container 디렉토리: 200개                                   │
│  - Log 파일: 600개                                            │
│  - Total dentry: ~900개                                      │
│                                                              │
│  After (/var/log/containers):                                │
│  - 심볼릭 링크 파일: 600개                                       │
│  - Total dentry: ~600개                                      │
│                                                             │
│  감소율: ~33%                                                 │
│  → Slab memory 대폭 감소!                                     │
└─────────────────────────────────────────────────────────────┘
```

## Alloy 설정 변경

```yaml
# Before
discovery.file "logs" {
  path_targets = [
    {
      __path__ = "/var/log/pods/**/*.log",
      job = "kubernetes-pods",
    },
  ]
}

# After
discovery.file "logs" {
  path_targets = [
    {
      __path__ = "/var/log/containers/*.log",
      job = "kubernetes-containers",
    },
  ]
}
```

# 심볼릭 링크를 사용하는 것의 트레이드오프

## 장점

### 1. Dentry Cache 감소
- 플랫한 구조로 인한 dentry 객체 수 감소
- Slab memory 사용량 감소
- cgroup memory pressure 완화

### 2. 파일 탐색 성능 향상
- 단일 디렉토리에서 모든 로그 파일 접근
- 재귀적 디렉토리 탐색 불필요
- 파일 시스템 오버헤드 감소

### 3. 메타데이터 추출 용이
- 파일명에 파드명, 네임스페이스, 컨테이너명 포함
- 추가 파싱 없이 메타데이터 추출 가능

```bash
# 파일명 패턴
<pod_name>_<namespace>_<container_name>-<container_id>.log
```

## 단점 및 주의사항

### 1. 심볼릭 링크 해석 오버헤드

심볼릭 링크를 따라가려면 추가적인 시스템 콜이 필요합니다.

```text
┌─────────────────────────────────────────────────────────────┐
│              Symbolic Link Resolution                       │
│                                                             │
│  일반 파일 접근:                                                │
│  open("/var/log/pods/.../0.log") → 파일 열기                   │
│                                                             │
│  심볼릭 링크 접근:                                              │
│  open("/var/log/containers/xxx.log")                        │
│  → readlink() (심볼릭 링크 해석)                                │
│  → open("/var/log/pods/.../0.log") (실제 파일 열기)             │
│                                                             │
│  추가 오버헤드: readlink() 시스템 콜                              │
└─────────────────────────────────────────────────────────────┘
```

**영향:**
- 파일 열기 시 약간의 성능 오버헤드
- 대부분의 경우 무시할 수 있는 수준
- 로그 수집 빈도에 따라 영향 다름

### 2. 심볼릭 링크 업데이트 타이밍

컨테이너가 재시작되면 심볼릭 링크가 업데이트됩니다.

```text
┌─────────────────────────────────────────────────────────────┐
│              Link Update Timing                             │
│                                                             │
│  Container Restart:                                         │
│  1. Old log file: /var/log/pods/.../container/0.log         │
│  2. New log file: /var/log/pods/.../container/1.log         │
│  3. Symlink update: /var/log/containers/xxx.log             │
│     → points to 1.log                                       │
│                                                             │
│  잠재적 문제:                                                  │
│  - 링크 업데이트 전에 파일을 열면 이전 로그 파일 참조                   │
│  - 로그 수집기가 파일 변경을 감지하지 못할 수 있음                     │
│  - inode 변경 감지 필요                                        │
└─────────────────────────────────────────────────────────────┘
```

**해결 방법:**
- 로그 수집기가 inode 변경을 감지하도록 설정
- 파일 재오픈 메커니즘 구현
- Alloy, Fluent Bit 등은 이미 이를 처리함

### 3. 파일 시스템 의존성

심볼릭 링크는 원본 파일이 존재해야 합니다.

```text
┌─────────────────────────────────────────────────────────────┐
│              File System Dependency                         │
│                                                             │
│  시나리오:                                                    │
│  1. 파드 삭제                                                 │
│  2. /var/log/pods/.../ 파일 삭제                              │
│  3. 심볼릭 링크는 남아있지만 broken link                          │
│                                                             │
│  문제점:                                                     │
│  - 로그 수집기가 broken link를 만나면 에러 발생                     │
│  - 정기적인 정리 작업 필요                                        │ 
│                                                             │
│  완화 방법:                                                   │
│  - kubelet이 자동으로 정리 (일반적)                              │
│  - 로그 수집기가 broken link 무시                               │
└─────────────────────────────────────────────────────────────┘
```

### 4. 컨테이너 런타임별 차이

다른 컨테이너 런타임에서는 심볼릭 링크 구조가 다를 수 있습니다.

```bash
# containerd
/var/log/containers/pod_namespace_container-id.log
  → /var/log/pods/namespace_pod_uid/container/0.log

# Docker (구버전)
/var/log/containers/pod_namespace_container-id.log
  → /var/lib/docker/containers/container-id/container-id-json.log
```

**고려사항:**
- 컨테이너 런타임 변경 시 테스트 필요
- 링크 해석 경로가 다를 수 있음

# 요약

1. **문제 원인**: `/var/log/pods`의 깊은 디렉토리 구조를 watch하면서 dentry cache가 급증하여 slab memory가 증가했습니다.

2. **메트릭 불일치**: RSS는 증가하지 않았지만, cgroup에 할당된 slab memory가 증가하여 Node Exporter 메트릭이 우상향했습니다.

3. **해결 방법**: `/var/log/containers`의 플랫한 구조로 전환하여 dentry 객체 수를 감소시켰습니다.

4. **트레이드오프**: 심볼릭 링크 사용 시 약간의 성능 오버헤드와 파일 시스템 의존성이 있지만, 대부분의 로그 수집기는 이를 잘 처리합니다.

5. **다른 에이전트**: Fluentd, Filebeat 등도 `/var/log/pods`를 직접 watch하면 동일한 문제가 발생할 수 있으므로, `/var/log/containers` 사용을 권장합니다.

이 이슈를 통해 Kubernetes 환경에서 로그 수집 시 파일 시스템 구조와 커널 메모리 관리의 관계를 이해하는 것이 중요함을 알 수 있습니다.
