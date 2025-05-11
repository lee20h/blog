---
title: "system call에 대해 알아보자"
date: 2020-03-27
tags:
  - system call
categories:
  - linux
publishResources: true
---

# System Call

## 시스템 프로그래밍

- System Call: OS가 제공하는 기능들을 사용하는 것
    - 하드웨어를 제어하거나,
    - 다른 프로세스와의 통신을 수행하거나,
    - 시스템 정보에 접근, 수정하거나,
    - 시스템을 제어하는 기능 등
- 대표적인 시스템 콜
    - 화면 출력: printf() <- C 라이브러리. 내부에서 write() system call 사용
    - 파일 제어: open(), close(), read(), write()
    - 동적 메모리 할당: malloc() <- C 라이브러리. 내부에서 brk(), mmap() 사용
    - 네트워크 통신: socket(), send(), receive()

## System Call vs. Library function

- 시스템 호출
    - OS Kernel이 제공하는 서비스를 이용해 프로그램을 작성할 수 있도록 제공되는 프로그래밍 인터페이스
    - 기본적인 형태는 C 언어의 함수 형태로 제공
        - `리턴값 = 시스템호출명(인자, …);`
    - C Library 가 이러한 형태로 편리하게 이용할 수 있게 제공해주는 것
- 라이브러리 함수
    - 라이브러리 : 미리 컴파일된 함수들을 묶어서 제공하는 특수한 형태의 파일
        - `/lib`, `/usr/lib`에 위치하며 `lib*.a` 또는 `lib*.so` (static object) 형태로 제공
    - 자주 사용하는 기능을 독립적으로 분리하여 구현해둠으로써 프로그램의 개발과 디버깅을 쉽게하고 컴파일을 좀 더 빠르게 할 수 있다
    - OS 커널과 무관하게 단순한 C 코드를 수행
        - 예) strcpy() : 문자열 복사를 위한 연산을 수행. 편의를 위한 라이브러리 함수
- 두 가지 시스템 콜 호출 방법
    - 직접 시스템콜 호출: 본래 시스템콜은 특수한 방식(trap)을 통해 호출하여야 함
        - 이유1: 시스템콜의 코드가 일반 사용자가 접근할 수 없는 OS 커널 영역에 존재하기 때문
        - 이유2: 시스템콜은 다수 존재하고, 번호로 구분하기 때문에 직관적이지 않음
        - 굳이 하려면? : syscall() 함수 이용
    - **라이브러리 함수 사용**
        - C 라이브러리는 시스템콜을 간편히 사용할 수 있게 도와주는 여러 함수 제공
        - 예) printf() : 화면 출력을 위해서는 본래 write() 시스템 콜을 사용해야 함
        - **일반적으로 거의 모든 경우에 라이브러리 함수를 통해 시스템콜을 이용함**

![image](https://user-images.githubusercontent.com/59367782/96449848-666c7680-1250-11eb-9968-46781fa0c72e.png)


### syscall()의 사용

```cpp
#define _GNU_SOURCE
#include <unistd.h>
#include <sys/types.h>
#include <sys/syscall.h>
#include <stdio.h>

int main() {
        printf("My pid: %d\n", getpid());
        printf("My pid: %d by syscall()\n", (int)syscall(39));
        sleep(1);
        return 0;
}
```

- getpid()
    - 현재 프로세스의 PID를 반환
    - C 라이브러리 함수
    - 이 정보는 OS가 관리하기 때문에, 시스템콜을 통해 수행됨
    - 이때 sys_getpid 라는 이름의 시스템콜을 이용하며, 해당 시스템콜 번호는 39

### man page에서의 섹션 구분

![image](https://user-images.githubusercontent.com/59367782/96450868-bc8de980-1251-11eb-9e3d-f3746a2ee180.png)

printf의 경우 section이 1과 3으로 나뉘어지므로 `$ man printf`와 `$ man 3 printf`로 나눠서 볼 수 있다.

## Error handling

### 시스템 호출의 오류 처리방법

- **결과값: 성공하면 0을 리턴, 실패하면 -1을 리턴**
- 실패 시, 전역변수 errno에 오류 코드 저장
    - Extern 을 이용해 C 라이브러리와 사용자 프로그램이 전역 변수를 공유할 수 있음
    - Extern: 해당 소스 파일의 외부에서 선언한 변수를 인용해서 사용하는 것
- 오류 코드의 확인: errno 유틸리티 사용 <- `moreutils` 설치 후  `$ errno -l`
- 예시
```cpp
#include <unistd.h>
#include <stdio.h>
#include <errno.h>

extern int errno;

int main() {
        if (access("unix.txt", F_OK == -1) {
                printf("errno=%d\n", errno);
        }       
        return 0;
}  
```

### 라이브러리 함수의 오류 처리방법

- 오류가 발생하면 NULL을 리턴, 함수의 리턴값이 int 형이면 -1 리턴
- errno 변수에 오류 코드 저장
- 예시
```cpp
#include <stdlib.h>
#include <stdio.h>
#include <errno.h>

extern int errno;

int main() {
        FILE *fp;
        if((fp = fopen("unix.txt", "r")) == NULL) {
                printf("errno=%d\n", errno);
                return 1;
        }
        fclose(fp);

        return 0;
}
```


- fopen()
    - File stream 을 여는 C 라이브러리 함수
    - 해당 파일이 없으면 에러가 나고, NULL 을 리턴

### 보다 편리한 오류 처리

- 오류 메시지 출력 : perror(3)
- Errno 에 따라 에러 메시지를 출력함
- 예시
```cpp
#include <unistd.h>
#include <stdio.h>

int main() {
        if (access("unix.txt", F_OK) == -1) {
                perror("my message");
                return 1;
        }
        return 0;
}
```