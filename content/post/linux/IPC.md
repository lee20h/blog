---
title: "IPC에 대해 알아보자"
date: 2020-04-25
tags:
  - IPC
  - signal
  - shared memory
  - socket
categories:
  - linux
publishResources: true
---

# IPC: Signal and Shared Memory

## IPC: Inter-process communication

### IPC의 필요성

- Many processes are co-operating
    - 서로 협업하는 프로세스: 한 가지 목적을 위해 여러 프로세스가 함께 동작
        - 큰 프로그램의 모듈화, 병렬 작업을 통한 성능 향상, 사용자 편의성 향상 등
        - 예) Game: video and audio processing, background data loading, chatting, etc.
    - 이를 위해서는 상호 간의 정보 교환이 필수
- 그러나 프로세스들은 가상 메모리 공간 (Virtual Memory Space) 에 갇혀 있음
    - **각 프로세스는 자기 자신만의 독립되고, 고립된 (isolated) 메모리 공간을 가짐**
        - 프로세스들은 서로 다른 프로세스의 space 를 침범할 수 없음! 존재조차 모름.
    - 함께 일을 해야 하는데, 그럼 어떻게 정보를 주고 받지? → **IPC**

### IPC: Inter-Process Communication

- IPC: 프로세스 간 통신
    - 모든 프로세스들의 공간에 접근할 수 있는 OS의 도움을 받아, 프로세스들 간에 서로 정보를 교환하는 것
    - 예) Message queue, shared memory, signal, pipe, socket 등
    - (더 쉬운 예. CTRL+C, V)
- 어떤 medium 을 어떤 방식으로 사용해서 정보를 전달하느냐에 따라 서로 다른 통신 특성을 갖게 됨
    - 커널 및 유저 메모리 공간, 네트워크 등이 medium 이 될 수 있음
    - 예) Signal: 커널 메모리 공간을 통해 아주 단순한 정보만 단방향으로 전달
    - 예) Shared memory: 유저 메모리 공간을 통해 양방향으로 정보 전달 가능
        - 그러나 양쪽이 동시에 데이터를 수정할 수 있기 때문에 동기화 문제 발생 가능
    - 예) Socket: 네트워크를 통해 원격 시스템에서 동작하는 프로세스와 통신 가능
- 협업 방식 및 상황에 따라 적절한 IPC 를 선택할 수 있어야 함

## Signal

### 시그널의 개념

- 시그널
    - 프로세스에 무슨 일이 발생했음을 알리는 간단한 메시지를 **비동기적**으로 보내는 것
    - 예) 메모: “전화왔음” (누구한테?? 거기까진 알 수 없음. 메모 종류도 한정적)
- 발생사유
    - 0으로 나누기처럼 프로그램에서 예외적인 상황이 일어나는 경우
    - Kill(2) 처럼 시그널을 보내는 시스템콜을 호출해 다른 프로세스에 시그널을 보내는 경우
    - 사용자가 CTRL+C 와 같이 키를 입력해 인터럽트를 발생시킨 경우
- 시그널 처리방법
    - **각 시그널에 지정된 기본 동작 수행. 대부분의 기본 동작은 프로세스 종료**
    - 무시: 발생한 시그널을 무시하고 진행
    - 핸들러 호출: 특정 시그널 처리를 위한 함수(핸들러)를 지정해두고, 시그널을 받으면 해당 함수를 호출
    - 블록: 발생한 시그널을 처리하지 않고 그대로 둠. 블록을 해제하면 그때 전달되어 처리

### 시그널 종류

![image](https://user-images.githubusercontent.com/59367782/100050890-d082c780-2e5d-11eb-8d97-38549679d545.png)

- OS에 따라 번호와 종류의 차이가 있을 수 있음
- 사용할 시그널의 자세한 내용은 항상 OS에 따라 확인하고 사용

### 시그널 보내기: kill(2)

```
# kill -9 3255
```

- (이미 배운) kill 명령어
    - 프로세스에 시그널을 보내는 명령 (예. PID가 3255인 프로세스에 9번 SIGKILL 전달)


```c
#include <sys/types.h>
#include <signal.h>

int kill(pid_t pid, int sig);
```

- 시그널 보내기: kill(2)
    - **pid가 0보단 큰 수 : pid로 지정한 프로세스에 시그널 발송**
    - pid가 -1이 아닌 음수 : **프로세스 그룹ID가 pid의 절대값인 프로세스 그룹**에 속하고, 시그널을 보낼 권한을 가지고 있는 모든 프로세스에 시그널 발송
    - pid가 0 : 특별한 프로세스를 제외하고 프로세스 그룹ID가 시그널을 보내는 프로세스의 **프로세스 그룹ID와 같은 모든 프로세스**에게 시그널 발송
    - pid가 -1 : 시그널을 보내는 프로세스의 유효 사용자ID가 root가 아니면, 특별한 프로세스를 제외하고 프로세스의 실제 사용자ID가 시그널을 보내는 프로세스의 **유효 사용자ID와 같은 모든 프로세스**에 시그널 발송

### 시그널 보내기: raise(2) and abort(3)

```c
#include <signal.h>

int raise(int sig);
```

- 시그널 보내기: raise(2)
    - 함수를 호출한 프로세스 자기 자신에게 시그널 발송

```c
#include <stdlib.h>

void abort(void);
```

- 시그널 보내기: abort(3)
    - 함수를 호출한 프로세스에 자기 자신에게 SIGABRT시그널 발송
    - SIGABRT 시그널은 프로세스를 비정상적으로 종료시키고 코어덤프 생성

### 시그널 핸들러: signal(3)

- 시그널 핸들러
    - 시그널을 받았을 때, 기본 동작을 대체하여 이를 처리하기 위해 지정된 함수
    - 시그널을 받으면, 받은 시점에 수행 중이던 코드에서 마치 핸들러를 호출한 것 처럼 동작
    - 프로세스를 종료하기 전에 처리할 것이 있거나, 특정 시그널에 대해 종료하고 싶지 않을 경우 지정

```c
#include <signal.h>

void (*signal(int sig, void (*disp)(int)))(int);
```

- 시그널 핸들러 지정: signal(3)
    - disp : sig로 지정한 시그널을 받았을 때 처리할 방법
        - 시그널 핸들러 함수명
        - SIG_IGN : 시그널을 무시하도록 지정
        - SIG_DFL : 기본 처리 방법으로 처리하도록 지정
    - (어떤 Unix 시스템에서는) 한 번 핸들러가 수행되고 나면, 핸들러 지정이 취소됨

### 시그널 핸들러: sigset(3)

```c
#include <signal.h>

void (*sigset(int sig, void (*disp)(int)))(int);
```
- 시그널 핸들러 지정: sigset(3)
    - disp : sig로 지정한 시그널을 받았을 때 처리할 방법
        - 시그널 핸들러 함수명
        - SIG_IGN : 시그널을 무시하도록 지정
        - SIG_DFL : 기본 처리 방법으로 처리하도록 지정
    - **sigset함수는 signal함수와 달리 시그널 핸들러가 한 번 호출된 후에 기본동작으로 재설정하지 않고, 해당 시그널 핸들러를 자동 재지정한다.**

### 시그널 핸들러: sigaction()

```c
#include <signal.h>

int sigaction(int signum, const struct sigaction *act, struct sigaction *oldact);
```

- sigaction 함수
    - signal이나 sigset 함수처럼 시그널을 받았을 때 이를 처리하는 함수 지정
    - signal, sigset 함수보다 다양하게 시그널 제어 가능
    - signum : 처리할 시그널 번호
    - **act : 시그널을 처리할 방법을 지정한 구조체 주소**
    - oldact : 기존에 시그널을 처리하던 방법을 저장할 구조체 주소
    - 첫번째 인자로 SIGKILL과 SIGSTOP을 제외한 어떤 시그널도 사용 가능

### struct sigaction

```c
struct sigaction {
    int sa_flags;
    union {
        void (*sa_handler)();
        void (*sa_sigaction)(int, siginfo_t *, void *);
    } _funcptr;
    sigset_t sa_mask;
};
```

- 핸들러의 동작을 보다 세밀하고 정확하게 제어하기 위한 정보를 전달
    - sa_flags : 시그널 전달 방법을 수정할 플래그
        - 예) SA_RESETHAND: 시그널 핸들러가 한 번 호출되면 지정이 취소됨
    - **sa_handler**/sa_sigaction : 시그널 처리를 위한 동작 지정
        - sa_flags에 SA_SIGINFO가 설정되어 있지 않으면 sa_handler에 시그널 처리동작 지정
        - sa_flags에 SA_SIGINFO가 설정되어 있으면 sa_sigaction 멤버 사용
    - **sa_mask** : 시그널 핸들러가 수행되는 동안 블록될 시그널을 지정한 시그널 집합

### 시그널 집합: sigset_t

```c
typedef struct {
    unsigned int __sigbits[4];
} sigset_t;
```

- 복수의 시그널을 한 번에 지정, 처리하기 위해 도입한 개념
    - Bit masking: 각 비트가 각각 시그널의 지정 여부를 표시
    - 프로세스 수행 상태에 따라 전체 시그널의 상태가 혼동스러울 수 있으므로, 항상 전체 시그널 모두에 대해 어떤 처리를 지정하도록 하기 위함

```c
int sigemptyset(sigset_t *set);
```

- 시그널 집합 비우기 : sigemptyset(3)
    - 시그널 집합에서 모든 시그널을 0으로 설정

```c
int sigfillset(sigset_t *set);
```
- 시그널 집합에 모든 시그널 설정: sigfillset(3)
    - 시그널 집합에서 모든 시그널을 1로 설정

```c
int sigaddset(sigset_t *set, int signo);
```
- 시그널 집합에 시그널 설정 추가: sigaddset(3)
    - signo로 지정한 시그널을 시그널 집합에 추가

```c
int sigdelset(sigset_t *set, int signo);
```
- 시그널 집합에서 시그널 설정 삭제: sigdelset(3)
    - signo로 지정한 시그널을 시그널 집합에서 삭제

```c
int sigismember(sigset_t *set, int signo);
```
- 시그널 집합에 설정된 시그널 확인: sigismember(3)
    - signo로 지정한 시그널이 시그널 집합에 포함되어 있는지 확인

### 시그널의 블록: sigprocmask(2)

```c
#include <signal.h>

int sigprocmask(int how, const sigset_t *set, sigset_t *oldset);
```
- how : 시그널을 블록할 것인지, 해제할 것인지 여부
    - SIG_BLOCK : set에 지정한 시그널 집합을 시그널 마스크에 추가
    - SIG_UNBLOCK : set에 지정한 시그널 집합을 시그널 마스크에서 제거
    - SIG_SETMASK : set에 지정한 시그널 집합으로 현재 시그널 마스크 대체
- set : 블록하거나 해제할 시그널 집합 주소
- oldset : NULL 또는 이전 설정값을 저장한 시그널 집합 주소
- Obsoleted services: sighold(2) and sigrelse(2)

### 시그널 대기: sigsuspend(2)

```c
#include <signal.h>

int sigsuspend(const sigset_t *set);
```
- 지정한 시그널(들)이 도착할 때까지 대기
    - set : **기다리려는 시그널들을 제외한 다른 모든 시그널들을 지정**한 시그널 집합
    - 정확한 동작: 현재 동작 중인 프로세스의 시그널 마스크를 set 으로 대치하여, 기다리려는 시그널들을 제외한 다른 시그널들은 블록 처리됨. 따라서 기다리려는 시그널만 프로세스로 전달될 수 있게 설정되는 것
- Obsoleted service: sigpause (2)

### 알람 시그널: alarm(2)

- 알람 시그널
    - 일정한 시간이 지난 후에 자동으로 시그널이 발생하도록 하는 시그널
    - 일정 시간 후에 한 번 발생시키거나, 일정 간격을 두고 주기적으로 발송 가능

```c
#include <unistd.h>

unsigned int alarm(unsigned int sec);
````

- 알람 시그널 생성: alarm(2)
    - sec : 알람이 발생시킬 때까지 남은 시간(초 단위)
    - 일정 시간이 지나면 SIGALRM 시그널 발생
    - 프로세스별로 알람 시계가 하나 밖에 없으므로 알람은 하나만 설정 가능

## Shared Memory

- 공유 메모리
    - 같은 메모리 공간을 두 개 이상의 프로세스가 공유하는 것
        - 물론 OS의 허가를 받고 공유함. 그런데 처음 허가 한 번 받고 나면 끝. OS는 관여하지 않음
    - 같은 메모리 공간을 사용하므로 이를 통해 데이터를 주고 받을 수 있음
    - 문제점: 동기화
        - A가 데이터를 기록하는 동안, 같은 위치에 B가 데이터를 쓰면?
        - A가 데이터를 아직 다 쓰지 않았는데, B는 다 쓴 줄 알고 데이터를 읽어가면?

### 공유 메모리 생성: shmget(2)

```c
#include <sys/types.h>
#include <sys/ipc.h>
#include <sys/shm.h>
int shmget(key_t key, size_t size, int shmflg);
```
- key : IPC_PRIVATE 또는 ftok()로 생성한 키 값
    - 상호 약속된 키 값을 기입해야 공유 메모리를 접근할 권한을 얻을 수 있음
    - IPC_PRIVATE: 부모-자식 간의 공유 메모리 공유를 위해 사용
    - key_t ftok(const char *pathname, int proj_id);
        - 주어진 파일명(절대 경로)과 proj_id 를 이용해 키 값을 생성함
        - 절대 경로, proj_id 가 동일해야 같은 키 값이 생성됨
- size : 공유할 메모리 크기. 페이지 크기의 배수로 지정.
    - 페이지: 메모리를 관리하는 기본 단위. 보통 4KB
- shmflg : 공유 메모리의 속성을 지정하는 플래그
    - IPC_CREAT: 공유 메모리 공간을 새로 생성함
    - IPC_EXCL: IPC_CREAT 와 함께 사용해서, 만약 기존 공간이 있다면 fail 하도록 설정
- 공유 메모리 식별자를 리턴
    - OS가 공유 메모리 공간을 관리하는 자료 구조를 가리킴. 파일 기술자와 유사.

### 공유 메모리 연결 및 해제

```c
#include <sys/types.h>
#include <sys/shm.h>
void *shmat(int shmid, const void *shmaddr, int shmflg);
```

- 공유 메모리 연결: shmat(2)
    - shmid : 공유 메모리 식별자
    - shmaddr : 공유 메모리를 연결할 주소
    - shmflg : 공유 메모리에 대한 읽기/쓰기 권한
        - 0(읽기/쓰기 가능), SHM_RDONLY(읽기 전용)

```c
#include <sys/types.h>
#include <sys/shm.h>
int shmdt(char *shmaddr);
```
- 공유 메모리 연결 해제: shmdt(2)
    - shmaddr: 연결을 해제할 공유 메모리 주소

### 공유 메모리 제어

```c
#include <sys/types.h>
#include <sys/ipc.h>
#include <sys/shm.h>
int shmctl(int shmid, int cmd, struct shmid_ds *buf);
```

- 공유 메모리 제어: shmctl(2)
    - cmd : 수행할 제어기능
        - IPC_RMID : 공유 메모리 제거 (할당 해제)
        - IPC_SET : 공유 메모리 정보 내용 중 shm_perm.uid, shm_perm.gid, shm_perm.mode 값을 세번째 인자로 지정한 값으로 변경
        - IPC_STAT : 현재 공유 메모리의 정보를 buf에 지정한 메모리에 저장
        - SHM_LOCK : 공유 메모리를 잠근다.
        - SHM_UNLOCK : 공유 메모리의 잠금을 해제한다.

## TCP/IP Overview

- TCP/IP
    - 인터넷 통신을 위한 표준 프로토콜
    - 4계층으로 구성
    - Packet 단위로 데이터를 주고 받음

| 계층 | 이름 |
|---|---|
|L4 | 응용 계층 |
|L3 | 전송 계층 |
|L2 | 인터넷 계층 |
|L1 | 네트워크 엑세스 |

### IP address and hostname

- IP주소와 호스트명
    - IP 주소: 네트워크에서 특정 시스템 혹은 노드 (node)를 식별하는 주소 체계
        - 점(.)으로 구분된 32비트 값 (192.168.0.1)
    - Host name: 로컬 네트워크에서 시스템에 부여된 이름
    - Domain name: 인터넷에서 IP 주소에 대해 Domain Name Service 에 의해 부여된 이름
        - nslookup 명령을 통해 domain name 이 가리키는 IP 주소를 알 수 있음
    - Special IP address 127.0.0.1 (loopback) : 자기 자신을 가리키는 특수한 IP 주소
- Public and Private IPs
    - Public IP: 전체 인터넷 망에서 각 시스템을 구분하는 주소
        - 전세계 어디서든 동일한 IP로 특정 시스템에 접속 가능 (예. 203.254.143.152)
    - Private IP: Private (or local) network 에서 각 시스템을 구분하는 주소
        - 예) 우리집 공유기 안에서만 노트북, 핸드폰, 스마트 TV를 구분 (192.168.0.xxx)
- 호스트 명과 IP주소 간의 변환
    - /etc/hosts 파일 또는 DNS 등
    - /etc/nsswitch.conf 파일에 주소변환을 누가 할 것인지 지정

### Port

- Port: an endpoint of the network communication
    - 실제 통신을 수행하는 주체를 구분하는 번호
        - 주체: 호스트에서 동작하고 있는 서비스
        - 서비스: 한 프로세스가 여러 서비스(여러 포트를 통해)를 운영할 수도 있고, 하나의 서비스가 여러 프로세스를 이용해 동작할 수 있음
    - 2바이트 정수로 0~65535까지 사용 가능
    - `<IP address>:<port number>`형태로 전체 주소를 지정
        - 예) 203.254.143.152:80 -> 해당 호스트에서 동작하는 http 서비스
- 잘 알려진 포트
    - 잘 알려진 서비스들 (well-known services)의 포트 번호 (보통 1024 이하)
    - SSH(22), HTTP(80), FTP(21)
    - /etc/services : 잘 알려진 포트 번호들을 저장

### IP Network Communication

- IP 주소를 통해 네트워크 상에서 특정 호스트를 식별하고,
- Port 번호를 통해 호스트 내에서 특정 서비스를 식별하여,
- 원격 서비스들 간의 통신이 가능함

### TCP and UDP

- IP 상단의 전송 계층(L3)에서 통신을 위한 부가적인 기능을 제공하는 프로토콜들
    - IP 의 역할: 인터넷 상에서 특정 IP 주소를 가진 시스템을 찾아 패킷을 전해주는 것
    - TCP/UDP 의 역할: 단순히 패킷을 전달한다 라는 것을 넘어, 패킷을 순서대로 전달한다거나, 패킷이 전달되었다는 것을 보장한다거나, 패킷을 보내는 속도를 조절하여 성능을 극대화 하는 등의 부가 기능을 수행
    - 일반적으로 프로그램(L4)은 TCP 혹은 UDP 를 이용하여 실제 통신을 수행함
- UDP(User Datagram Protocol): 매우 단순한 패킷 전송 역할만 수행
- TCP(Transmission Control Protocol): 신뢰성있는 통신을 위한 다양한 기능 제공
    - 인터넷에서 TCP를 주로 사용하기 때문에, TCP/IP 라고 부름

| TCP | UDP |
|---|---|
| 연결지향형 | 비연결형 |
| 신뢰성 보장 | 비신뢰성 |
| 흐름 제어 기능 제공 | 흐름 제어 기능 X |
| 순서 보장 | 순서 보장 X |

## Socket

- Socket: an endpoint of the network communication (?!)
    - OS에서 제공하는 네트워크 통신을 위한 IPC abstraction
        - 소켓이 바로 IP:Port 를 이용해서 통신을 수행하는 주체
            - 예) 소켓=핸드폰, IP:port=핸드폰 번호
        - 즉, ip:port 는 결국 특정 소켓을 지정하는 이름이라 할 수 있음
- 프로그램은 고유의 포트 번호를 부여받은 소켓을 이용해서 네트워크 통신을 수행할 수 있음
    - 예) 친구랑 전화를 하고 싶으면?
        - 새 핸드폰을 산다 ≈ 새 소켓을 연다
        - 통신사 등록하여 개통한다 ≈ 소켓에 Port 번호를 부여함 (IP 주소는 이미 있음)
        - 전화를 건다 ≈ TCP 의 경우, 전화를 걸고, 받아서 연결이 수립되어야 통신 가능
        - 통화를 한다 ≈ 소켓을 이용해 패킷 단위로 데이터를 주고 받는다
        - (통신을 다 했으면? 소켓을 닫는다 ≈ 핸드폰을 버린다)

### 소켓: 종류 및 통신 방식

- 소켓의 종류
    - AF_UNIX : 유닉스 도메인 소켓 (시스템 내부 프로세스간 통신)
    - AF_INET : 인터넷 소켓 (네트워크를 이용한 통신)
- 소켓의 통신 방식
    - SOCK_STREAM : TCP 사용
    - SOCK_DGRAM : UDP 사용
- 즉, 아래와 같이 4가지 종류의 소켓을 사용할 수 있음
    - (AF_INET, SOCK_STREAM): 일반적인 TCP 를 이용한 인터넷 통신
    - (AF_INET, SOCK_DGRAM)
    - (AF_UNIX, SOCK_STREAM)
    - (AF_UNIX, SOCK_DGRAM)

### 소켓: 관련 시스템콜

- socket() : 소켓 파일 기술자 생성
- bind() : 소켓 파일 기술자를 지정된 IP 주소/포트번호와 결합(bind)
- listen() : 클라이언트의 접속 요청 대기
- connect() : 클라이언트가 서버에 접속 요청
- accept() : 클라이언트의 접속 허용
- recv() : 데이터 수신(SOCK_STREAM)
- send() : 데이터 송신(SOCK_STREAM)
- recvfrom() : 데이터 수신(SOCK_DGRAM)
- sendto() : 데이터 송신(SOCK_DGRAM)
- close() : 소켓 파일기술자 종료

### 소켓: TCP 통신을 위한 수행 순서

![image](https://user-images.githubusercontent.com/59367782/100572157-2d77f500-3318-11eb-9a51-93087f2623bc.png)

### 1. 소켓 생성: socket(2) 

```c
#include <sys/types.h>
#include <sys/socket.h>
int socket(int domain, int type, int protocol);
```

- domain : 소켓 종류 (AF_UNIX, AF_INET)
- type : 통신 방식 (TCP, UDP)
- protocol : 소켓에 이용할 프로토콜
    - 0: 지정한 통신 방식의 기본 프로토콜
- Return value: 새롭게 생성된 소켓의 file descriptor
- 소켓 또한 파일 형태로 관리가 됨
- 따라서 소켓을 모두 이용한 다음에는 close()를 이용해서 닫기

```c
int sd;
sd = socket(AF_INET, SOCK_STREAM, 0);
close(sd);
```

### 2. 소켓 이름 지정: bind(3)

```c
#include <sys/types.h>
#include <sys/socket.h>
int bind(int sockfd, const struct sockaddr *addr, socklen_t addrlen);
```

- sockfd: socket file descriptor
- addr : 소켓의 이름을 표현하는 구조체 sockaddr 의 주소
- addrlen: struct sockaddr 의 크기

```c
int sd = socket(AF_INET, SOCK_STREAM, 0);

struct sockaddr_in sin;
memset((char *)&sin, '\0', sizeof(sin));

sin.sin_family = AF_INET;
sin.sin_port = htons(9000);
sin.sin_addr.s_addr = inet_addr("192.168.100.1");

bind(sd, (struct sockaddr *)&sin, sizeof(struct sockaddr));
```

### 네트워크 주소를 표현하는 구조체

- 인터넷 소켓의 주소 구조체

```c
struct sockaddr_in {
    sa_family_t sin_family; // AF_INET
    in_port_t sin_port; // 네트워크 바이트 port
    struct in_addr sin_addr; // 인터넷 주소
};

struct in_addr {
    uint32_t s_addr; // 네트워크 바이트 주소
}
```

- 유닉스 도메인 소켓의 주소 구조체

```c
struct sockaddr_un {
    sa_familyy_t sun_family; // AF_UNIX
    char sun_path[108]; // 통신에 사용할 파일의 경로명
}
```

### 바이트 저장 순서 지정: endian

- Endian
    - 정수를 저장하는 방식 : Big and Little endian
    - Little endian: 메모리의 높은 주소에 정수의 첫 바이트를 위치 -> 인텔, ARM
    - Big endian: 메모리의 낮은 주소에 정수의 첫 바이트를 위치 -> 모토로라, 썬
- 인텔 머신과 썬 머신이 서로 통신을 하면???
    - 인텔: “야 내가 0x0102 (258) 이라고 했잖아!”
    - 썬: “너 0x0201 (513) 라고 했거든!!”
- TCP/IP 표준: Big Endian
    - Host byte order(HBO)
    - Network byte order(NBO)
    - HBO와 NBO가 항시 다를 수 있음을 기억할 것

### 바이트 저장 순서 지정: 관련 서비스

```c
#include <arpa/inet.h>

uint32_t htonl(unit32_t hostlong);
uint32_t ntohl(unit32_t netlong);
uint16_t htons(unit16_t hostshort);
uint16_t ntohs(unit16_t netshort);
```

- htonl :32비트 HBO를 32비트 NBO로 변환
- ntohl : 32비트 NBO를 32비트 HBO로 변환
- htons : 16비트 HBO를 16비트 NBO로 변환
- ntohs : 16비트 NBO를 16비트 HBO로 변환
- Endian 을 변환해주어야 하는 정보
    - Type 이 지정되어 저장되는 정보
        - IP주소, 포트 번호 (int port = 8080;)
    - 실제 데이터를 주고 받을 때는 문자열 배열과 같은 형태의 바이트 집합을 주고받기 때문에 변환이 불필요함 (unsigned char buf[80];)

### 네트워크 주소 변환 서비스

```c
#include <sys/socket.h>
#include <netinet/in.h>
#include <arpa/inet.h>

in_addr_t inet_addr(const char *cp);
char *inet_ntoa(const struct in_addr in);
```
- IP주소의 형태
    - 192.168.10.1과 같이 점(.)으로 구분된 형태
    - 시스템 내부 저장 방법 : 이진값으로 바꿔서 저장
    - 외부에서는 일반적으로 문자열로 표기
- 문자열 형태의 IP주소를 NBO 32비트 값으로 변환 : inet_addr(3)
- 구조체 형태의 NBO IP주소를 문자열 형태로 변환: inet_ntoa(3)

### 3. 연결 대기 및 수락: listen(3) and accept(3)

```c
#include <sys/types.h>
#include <sys/socket.h>

int listen(int sockfd, int backlog);
int accept(int sockfd, struct sockaddr *addr, socklen_t *addrlen);
```

- 클라이언트 연결 기다리기: listen(3)
    - backlog : 동시 접속을 허용할 최대 클라이언트 수
- 연결 요청 수락하기: accept(3)
    - addr, addrlen: 접속을 요청한 클라이언트의 IP 정보를 저장할 메모리 주소들
        - Addrlen 은 반드시 구조체 크기 값으로 설정을 해주어야 함
    - Return value: 수락된 연결에 대한 새로운 소켓을 반환
        - 다수 클라이언트와 동시에 통신하기 위해,
        - Sockfd 소켓은 또다른 연결 요청을 대기하기 위해 사용하고,
        - 새로운 소켓을 이용해 클라이언트와 통신을 수행함
- UDP의 경우, 비연결성 프로토콜이기 때문에 “연결” 동작이 불필요함
    - 따라서 listen(), accept(), connect() 를 사용하지 않음

### 4. 연결 요청: connect(3)

```c
#include <sys/types.h>
#include <sys/socket.h>

int connect(int sockfd, const struct sockaddr *addr, int addrlen);
```
- addr : 접속하려는 서버의 IP 정보
- addrlen : addr 구조체의 크기

```c
int sd = socket(AF_INET, SOCK_STREAM, 0);

struct sockaddr_in sin;
memset((char *)&sin, '\0', sizeof(sin));

sin.sin_family = AF_INET;
sin.sin_port = htons(9000);
sin.sin_addr.s_addr = inet_addr("192.168.100.1");

connect(sd, (struct sockaddr *)&sin, sizeof(struct sockaddr));
```

### 5. TCP 데이터 송수신: send(3) and recv(3)

```c
#include <sys/types.h>
#include <sys/socket.h>

ssize_t send(int sockfd, const void *buf, size_t len, int flags);

ssize_t recv(int sockfd, void *buf, size_t len, int flags);
```

- buf, len: 데이터가 기록된/기록될 공간의 주소와 크기
- flags: IO 블록킹, 에러 처리 등의 옵션을 쓸 수 있음. 보통 0.
- Return value: 전송한/수신한 데이터의 크기

```c
char *msg = "Send Test\n";
int len = strlen(msg) + 1; //문자열 종료 문자 ‘\0’ 를 포함시키기 위함
if (send(sd, msg, len, 0) == -1) {
    perror("send");
    exit(1);
}
```
```c
char buf[80];
int len=sizeof(buf), rlen=0; //buf[]의 크기를 recv()에 전달해 overflow 방지
if ((rlen = recv(sd, buf, len, 0)) == -1) {
    perror("recv");
exit(1);
}
```

### 6. UDP 데이터 송수신: sendto(3) and recvfrom(3)

```c
#include <sys/types.h>
#include <sys/socket.h>

ssize_t sendto(int sockfd, const void *buf, size_t len, int flags, const struct sockaddr *dest_addr, socklen_t addrlen);

ssize_t recvfrom(int sockfd, void *buf, size_t len, int flags, struct sockaddr *src_addr, socklen_t *addrlen);
```

- TCP의 send(), recv()와 다른 점
    - UDP는 비연결성 프로토콜이기 때문에,
    - 매 호출 시마다 보낼 주소, 받을 주소를 명시하여야 함
    - 1회성으로 정보를 다수 서버에 전송할 경우, 효율적임
