---
title: "가상 메모리(virtual memory)에 대해 알아보자"
date: 2021-10-04
tags:
   - memory
categories:
   - operation system
publishResources: true
---

# Virtual Memory

프로그램에서 필요한 메모리를 논리적 주소 공간에서 물리적 주소 공간으로 접근을 한다. 예전에는 논리적 주소 공간과 물리적 주소 공간의 크기를 같게 해야 프로그램이 꺼지지 않고 실행이 되었다. 가상 메모리가 도입 되고나서는 가상 메모리 공간에 물리 메모리 공간에서 조금만 가지고 유지하며, 논리적 주소 공간에서 요구 할 때 가상 메모리 공간에 없다면 물리 메모리 공간에 접근해서 가져온다.

![Virtual-Memory](https://github.com/lee20h/blog/assets/59367782/b5402f38-a04b-4684-bbb6-c041ffbaf5d4)

Storage에 들어가는 메모리들은
1) 기존 프로그램 이미지에 있는 경우
2) SWAP영역(변경한 데이터 혹은 동적 할당한 데이터)에 있는 경우

또한 데이터 부분 Code Data Heap Stack들을 가상메모리로 유지중이다.

## Demand Paging

프로세스가 해당 페이지를 접근 하는 순간에 Paging해서 제공하는 것이다.

![vaild-invaild](https://github.com/lee20h/blog/assets/59367782/81b8c2ac-170d-4c91-8c98-129a0c998593)

Valid-Invalid Bit를 두고 운용한다. 실제로 Frame을 할당한 경우 Valid, 안한 경우에는 Invalid로 저장해놓은다. 이 때 Invalid한 Frame에 접근한 경우 `Page Fault`가 일어나게 된다.

![Page-Table](https://github.com/lee20h/blog/assets/59367782/c4848bce-5b10-4554-9a1f-f7521a2e912c)

Invalid한 프레임에 접근하게 되면 Page Fault가 일어난다.

**Page Fault**

![Page-Fault](https://github.com/lee20h/blog/assets/59367782/92d7e17f-f7aa-40cb-a3de-1679161f0aca)

1) CPU에서 요청한 메모리에 접근할 때 MMU가 Page table에서 Valid, Invalid을 체크한다.
2) Invalid일 때 Trap(SoftWare Interrupt)가 일어나게 된다. [대표적인 Software Interrput인 Page Fault] 이 때 System Call 처리한 것과 같이 User mode라면 Kernel mode로 변경하고 Page Fault Handler를 찾는다.
3) Page Fault Handler가 Free Framelist에서 연산에 맞게 빈 공간이나 맞는 공간을 찾아서 해당 Page table에 넣어주고 Valid bit을 Valid로 바꿔 준다.
4) Restart Instruction을 해서 오류가 안나게 실행해준다.

만약, Frame이 부족할 때 Page Fault Handling이 실패한다. 이것이 **Out Of Memory**(OOM)이다.

이러한 내용이 일반적으로 진행되었을 때의 얘기이다.

이제는 예외에 대해 얘기해보자.  
똑같이 `2.`까지 진행했을 때, Free Frame가 부족한 경우  
다른 Frame으로부터 Page을 가져오는 기법을 `Page Replacement`라고 한다. 이어서 Frame을 얻어 왔을 때 맞는 공간을 찾아야한다. 그 공간을 찾는 것을 두 가지로 나눌 수 있다.
1) Program image에서 가져온다.
2) `Storage`(Swap)에서 찾아온다. 그것을 `Swap-in`이라고 한다.
   이 부분이 위 그림의 설명으로 생각하면 좋다. 다음에는 위의 과정인 `3.`에서 Valid bit부터 이어가면 된다.

추가적으로 Page Replacement나 Storage Access가 일어날 때 성능 저하가 일어난다. Storage Access는 Virtual Memory가 설계 될 때 무조건 한번은 일어난다고 가정하고 설계된 것이다. 하지만 Page Table에서 기존의 Frame을 사용할려 할때 기존의 데이터를 Swap영역에 저장할 때 1번 접근하고, Instruction에서 Storage에 접근할 때 2번 접근하게 되어 성능저하가 일어난다. 결국 Storage 접근이 성능을 좌지우지하게 된다.

### Performance of Demand Paging
이전에 알아보았던 EAT(Effective Access Time)을 적용해볼 것이다. 이때 Page Fault Rate을 p라고 했을 때 무조건 일어날 경우 p=1, 안 일어날 경우 p=0으로 성능 차이를 알아보자.
- 무조건 안 일어난 경우 : EAT = (1-p) x Memory access
- 무조건 일어난 경우 : EAT = Memory access + p x (page fault overhead + swap page out + swap page in)

식을 정리하고 예를 들어보자.  
Memory access time = 200ns  
Average page-fault service time = 8ms  
EAT = (1-p) x 200 + p (8ms)  = 200 + p x 7,999,800  
만약 p = 0.001, 0.1% 확률로 일어난다고 했을 때는 EAT = 8.2um이 나온다. p = 0과 비교해보면 약 40배나 느리다고 볼 수 있다.  
같은 식으로 성능 저하를 10%이하로 잡기 위해서는 400,000 메모리 접근마다 한번 page fault가 일어나야 성능 저하가 10% 이하일 수 있다.

위에서 한 내용으로 Page Replacement는 대략적으로 이해가 되었다. 이제 Page Replacement의 알고리즘에 대해 공부한다.
## Page Replacement

### FIFO 알고리즘
![Replace-FIFO](https://github.com/lee20h/blog/assets/59367782/d5296599-cf14-47e4-ab7c-7eb013d864c2)

FIFO는 15번 Page Fault가 일어났다. 그림에서는 Frame을 3개만 할당했다. 직관적으로 Frame이 늘어나면 Page Fault가 줄어든다.
하지만 잘못 관리하면 늘어나게 되는데 이 부분을
`Belady's Anomarly`라고 한다.

### Optimal 알고리즘
![Optimal-algorithm](https://github.com/lee20h/blog/assets/59367782/8d15540a-dbb7-4af6-92ee-75786c4c9d0e)

가장 이상적인 알고리즘은 앞으로의 미래를 예측해서 가장 늦게 쓰일 프레임을 대체하는 것이다. 하지만 우리는 미래를 예측할 수 없어서 이상적인 알고리즘으로만 남았다.

### LRU 알고리즘
![LRU-algorithm](https://github.com/lee20h/blog/assets/59367782/1c22ab27-f289-4be3-b300-6160fb56c6cd)

현실적으로 접근해서 Least Recently Used 알고리즘으로 생각해보자.
전에 배운 LRU을 이용하는 거와 같다. FIFO보다는 적지만 이상적인 알고리즘보다는 많이 일어났다.

하지만 이러한 LRU도 구현하는데에 있어 문제가 있어 `LRU Approximation Algorithms`라는 것을 사용한다.

### LRU 유사 알고리즘
**LRU Approximation Algorithms**은   
page에 Frame이 있으며, Valid-Invalid bit 옆에 `Reference bit`을 둬서 사용한다. 밑에서 이야기할 `Modify bit` 또한 옆에 같이 둔다.  
Reference bit은 초기에 0으로 두고 page가 reference될 때 bit을 1로 설정한다.

이러한 알고리즘을 이용한 방법이 **Second-Chance Algorithm**다. 순환큐를 통해서 page들을 계속 순환하는데 reference bit가 1인 경우 0으로 바꿔주고 0인 경우 해당 page을 빼서 다른 페이지로 대체한다. 한 페이지가 계속 reference 된다면 bit가 0에서 1로 바뀌기 때문에 다음 사이클에서도 대체되지 않을 것이다.

이 알고리즘에서 더 나아가면 **Enhanced Second-Chance Algorithm**이 있다. 이 알고리즘은 Second-Chance에서 Reference bit외에 Modify bit을 두는 것이다. modify bit은 dirty bit과 같으며, Swap 영역에서 Swap-in으로 page에 frame이 넘어왔을 때 이 값이 변경되었는가 확인하는 비트이다. 이 비트가 1이라면 page replacement 되기 전에 Swap-out으로 Swap 영역에 값을 업데이트 해주고 replacement 되어야한다.  
해당 알고리즘을 썼을 때 대체되는 우선 순위는  
(reference, modify)일 때
1) (0, 0)
2) (0, 1)
3) (1, 0)
4) (1, 1)

이렇게 정리된다.

## Copy-on-Write

![Copy-on-Write](https://github.com/lee20h/blog/assets/59367782/ce5166e3-23e7-4350-9026-99dc2f07a2e4)

두 개의 프로세스가 하나의 페이지를 공유할 때(같은 데이터를 공유할 때) 사용한다. Shared Memory와 다른 차이점은 OS가 프레임을 아끼기 위해서 사용한 것이다. 예를 들어서 C Library는 물리 메모리 상 하나인데 여러 프로세스가 Shared Memory하듯이 모두가 자기 테이블 안에 C Library를 참조해서 가져다가 사용한다. 항상 Read-Only일 때만 이렇게 사용할 수 있다.  
하지만 누가 수정을 해야하는 상황이 올 때 사용하는게 Copy-on-Write이다. 예를 들어서 Fork를 사용할 때 부모 프로세스와 자식 프로세스는 같은 데이터를 공유한다. Fork 당시에는 같은 데이터를 공유하므로 같은 물리 메모리를 링크하면서 프레임을 아끼다가, 자신 프로세스에서 해당 데이터 중 조금 수정한다.

![Copy-on-Write2](https://github.com/lee20h/blog/assets/59367782/6d404dc2-ca06-4307-b50c-411256ff0b22)

따라서 목적은
1) 페이지 프레임 아끼기 위함
2) 카피를 안하기 위함 (메모리 접근을 최소화하기 위함 4B라고 가정시, 4B x 1024 x 2)

Write 작업을 하지 않는다면, 페이지 프레임을 아끼며(용량), 카피를 안하면서 카피의 오버헤드를 안 줘서 성능을 높게 유지할 수 있다.

### Memory-Maapped Files
유저가 File을 사용하기 위해서는 OS의 File System을 거쳐서 하드웨어 (Storage)에 접근해서 사용하는 것이 일반적인 방법이다.  
Page fault나 Swapping할 때 Memory Management을 거쳐서 Stroage에 접근했다.

Stroage 장치는 Magnetic tape으로 되어있어서 Sequential하게 접근하는게 일반적인 방법이다. 하지만 가끔은 Random하게 접근할 때가 있다. Random하게 접근한 경우 성능이 제대로 나오지 않는다. Memory Management을 거쳐서 Storage에 접근할 때는 Random하게 접근하는게 성능이 더 잘 나올 수 있었다.

하지만 이러한 내용은 예전 이야기가 되어버렸다. 이제는 하드웨어의 발전으로 이러한 랜덤 액세스 때문에 Memory-Mapped을 사용한다는 것은 틀린 말이다.

사용하는 이유는 Shared Memory을 위해서 라고 생각하면 된다.

![Memory-Mapped](https://github.com/lee20h/blog/assets/59367782/e1cd62f8-d06b-4186-b300-09f2740de3f9)

프로세스 A에서 고친 내용이 어느 순간이 disk file에 적용이 된다. 이때 오픈 대신에 mmap()으로 오픈하며, read, write가 아닌 값을 그냥 할당하거나 memset(), memcopy()로 메모리 값을 바꿀 수 있다.  
따라서 여러 프로세스들이 같은 파일에 대해서 공유해서 작업을 할때 Memory Mapped Files을 사용하면 **편리**하다. 원래는 동시에 접근하게되면 한 쪽은 읽기전용이 되지만 이 경우에는 공동작업이 편리하게 가능하다.

### Allocating Kernel Memory
커널 메모리는 물리적으로 Contiguous하게 될 필요가 있다. 그리고 다양한 크기의 구조체의 메모리를 필요로 한다.  
물리적으로 연속적으로 되어야 하는 이유는 I/O장치 때문이다. DMA (Direct Memory Access)가 OS 대신 I/O처리를 해준다. 이 때 DMA는 Virtual Memory을 모르고 Physical Memory로만 접근을 하기 때문에 연속적이여야 한다. 커널에서 물리 메모리에 크기만큼 연속적으로 복사해서 DMA에 넘겨준다. 이러한 작업을 줄이기 위해서 연속적으로 유지한다.

### Buddy System
물리적으로 연속된 페이지들을 효율적으로 할당하기 위해서 사용하는 시스템이다. 연속적인 공간을 필요로 할 때 필요 공간보다 큰 제일 작은 2의 제곱수로 분할하게 되면 트리가 구성되는데 맨 왼쪽의 리프노드에서 할당해주고 남은 노드들을 연속적으로 유지한다.
리눅스에서도 지금도 쓰이고 있다.

![Buddy-System](https://github.com/lee20h/blog/assets/59367782/16b7fea6-a08b-4bf8-bcbe-817a86a7deb8)

### Slab Allocation
메모리 공간을 빠르게 이용하기 위해서 사용하는 기법이다. 메모리를 매번 할당해서 사용하지 않고 미리 여러 공간을 할당하여 사용하는 방법(Pooling)  
예시) PCB

Slab Allocation은 Polling을 이용한 방법  

![Slab-Allocation](https://github.com/lee20h/blog/assets/59367782/377585bc-b9c6-4753-b103-0c63a4ed0469)

Cache을 다 쓰기 전에 Cache의 크기를 조금씩 먼저 늘린다. 할당과 해지는 시간이 걸리기 때문에 빈번히 일어나는 소프트웨어의 경우 Kernel object을 먼저 할당 받아 놓고 사용하는게 overhead을 줄일 수 있는 방법이다.

추가적으로, 소프트웨어 시간이 많이 걸리는 구간
1) I/O
2) Memory Copy
3) Dynamic Memory Allocation

## 정리

정리해보자면, `Page Falut` 일 때 frame이 없으면 `Page replacement`해야하는데 Page replacement Algorithm에 따라 Page Fault 발생할 확률이 다르고 심지어는 Belady's Anormaly가 일어날 만큼 큰 영향을 준다.  
가장 이상적인(Optimal) 알고리즘은 미래에 가장 늦게 선택되는 Page를 대체하는 것인데 미래는 예측하기 어려워 LRU 알고리즘을 사용한다.  
LRU방식은 과거를 보고 미래를 예측하는 방법인데 실제로 아주 빠른 시스템에서는 구현이 불가능해서 `LRU Approximation Algorithm`을 사용한다.  
reference bit을 사용하는 것이 `Second-Chance algorithm`이며, 거기서 reference와 modify비트를 사용하는게 `Enhanced Second-Chance` 알고리즘이다.

Demand paging을 제대로 사용할려면 Page fault가 적게 일어나야한다. 우리가 할 수 있는 일은
1) 메모리 많이꽂는다.
2) 효율적으로 replacement 알고리즘을 수행한다.

두 가지를 달성해야 Demand Paging을 높은 성능으로 돌아갈 수 있다.



### 추가적인 Issue
Program Structure  
극단적인 예지만 쉽게 이해할 수 있는 예시다.  
`int data [128][128]`이며, 각각의 row가 저장되는 한 페이지의 크기는 512B라고 가정 후 두 가지 프로그램이 있다고 한다.  

![Program-Structure](https://github.com/lee20h/blog/assets/59367782/90b7c8de-ef6f-46d9-8355-edce8be2fdb1)

1)
```c
for (j = 0; j < 128; j++)
   for (i = 0; i < 128; i++)
      data[i][j] = 0;
```
128 x 128 = 16,384 page faults  
페이지의 크기가 512이기 때문에 가로로 접근하며, 127개 적재 후 128번 째 다시 0번 col 적재할려하면 page replacement가 일어나서 page fault가 반복된다.
2)
```c
for (i = 0; i < 128; i++)
   for (j = 0; j < 128; j++)
      data[i][j] = 0;
```
128 page faults

C Compiler가 Page을 수직으로 할당하기 떄문에 이차원 배열에 있어서 `2)`와 같이 접근하는게 훨씬 속도가 빠르다. 그러므로 C를 사용함에 있어서 `1)`과 사용할 때는 page fault가 일어나 속도가 느릴 수 있다. 