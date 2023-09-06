---
title: "메인 메모리(main memory)에 대해 알아보자"
date: 2021-09-28
tags:
  - memory
categories:
  - operation system
published: true
---

# Main Memory
메모리에 프로세스를 할당할 때, 메모리를 관리하는 내용이라고 볼 수 있다.

![Multiple-partition](https://github.com/lee20h/blog/assets/59367782/100d8ca9-29f9-49d3-a9ce-5392381ae694)

## Memory Management Focus
처음에는 메모리에 연속되게 할당을 하였으나, 여러가지 이슈가 생기면서 새로운 방법을 찾아갔다.  
Memory Management에 두 가지 Focus에 맞춰서 볼 예정이다.
### Utilization
: 물리 메모리를 얼마나 아껴쓰는지  
- `Fragmentation`  
+ *External Fragmentation* : 필요한 메모리만큼 비어있는 메모리가 존재하나, 연속적이지 않은 경우  
+ *Internal Fragmentation* : 필요한 메모리보다 더 많은 메모리를 할당하고 남은 부분을 사용하지 않은 경우  
### Performance
: 메모리 접근 속도가 얼마나 빠른지  
- Address translation (logical to physical), swapping  
+ `Swapping` : Memory -> Disk, Disk -> Memory로 적합한 순서로 재배열하는 기법이다.  
![Swapping](https://github.com/lee20h/blog/assets/59367782/2cd4ce51-39e6-47c7-a218-7a5a7dd37414)

## Memory Management Method
메모리 관리 기법 또한, 두 가지로 나눠서 볼 수 있다.
1) Segmentation
2) Paging

### Segmentation

*Segment*란 우리가 프로그래밍을 할 때 사용되는 여러가지 논리적 단위로, 예를 들어 `main program` `procedure` `function` `method` 등을 말할 수 있다. 교수님께서는 의미있는 조각이라고 말하셔서 꽤 직관적으로 이해가 되었다.  
Segmentation은 논리적주소로 이루어진 2개의 tuple로 구성되어 있다고 할 수 있는데 그 모양은 이렇다.  
`<segment-number,offset>` segment-number에는 adress space 즉 주소공간을 의미하고 offset은 논리적 주소로 보았을 때 위치를 말한다.  
또, Segment table을 갖는데 `base`와 `limit`을 갖게 된다 base는 물리메모리상에서 시작하는 주소고 limit은 segment의 길이라고 생각하면 된다. 예를 들어 [segment-number][2] 이러한 Segment table이 있다면 [0][0] -> limit, [0][1] -> base를 뜻한다.  
![Segmentation](https://github.com/lee20h/blog/assets/59367782/10e6d85c-7c2c-4ae7-ac42-ee785d09e366)

또 하나의 특징은 Segmentation은 프로그래머가 직접 정하므로 OS의 컨트롤 밖에 위치한다. 그래서 이러한 부분을 다 고려해서 코딩을 하는 프로그래머는 거의 없다고 하셨다.

Focus를 맞췄던 Utilization과 Performance에 대해 이야기해보자.  
먼저 Utilization은 이전에 Contiguous보다는 좋아졌다. 왜냐 segment단위로 나누게 되어 external Fragmentation이 좋아졌기 때문이다. 하지만 Performance의 측면에서는 낮아졌다. Address translation의 측면에서는 Segmentation 하드웨어를 보면 메모리 주소에 두번 접근하기 때문이다. 이러한 Trade-off을 갖는데 어떻게 보면 Swapping에 입장에서 보면 이미 할당된 메모리를 segment단위로 나눠서 자리를 재배열해 다른 메모리가 할당하도록 자리를 내주게 되면 훨씬 속도가 빠른 경우도 있지만 그것은 특정한 경우에만 가능하다.

### Paging

*Paging*은 모든 메모리 공간을 page 단위로 쪼갠다고 생각하면 된다. 바로 Utilization과 Performance에 대한 결과를 보고 그 이유에 대해 알아보자.  
page단위로 쪼개기 때문에 external Fragmentation은 아예 일어나지 않고 internal Fragmentation 또한 최소화 할 수 있다. 따라서 Utilization은 향상한다. 하지만 Segmentation과 같이 Performance에서는 Address translation은 떨어지고 Swapping은 좀 더 좋아 질 수 있다.  
![Logical-Paging](https://github.com/lee20h/blog/assets/59367782/1db0dffe-731b-4885-9079-93652f5eae39)

Paging의 논리적인 모델을 살펴보면 이러하다. 논리 메모리에서 page table을 거쳐 물리메모리로 매칭되는 방식이다.  
![Paging](https://github.com/lee20h/blog/assets/59367782/0df44369-0085-473a-a3ee-7c214acb55d5)

PTBR(Page-table base register)에 Page table이 존재해서 CPU가 Memory에 두 번 접근한다. 먼저 page table에 접근 후 물리 메모리에 접근을 한다. 그리고 Segment에는 limit이 존재했으나, Page에는 limit이 존재하지 않는다.  
또 Free Frames에 대해 적어보면 이 List는 OS가 가지고 운영하며, 아래 사진과 같다.
![Free-Frames](https://github.com/lee20h/blog/assets/59367782/c0a33e66-42c6-4bd9-90a6-0b8b4284acc8)

할당 해제시에는 `Free-Frame list` 아무데나 넣어준다. 왜냐하면 물리 메모리는 어느 곳에 어떻게 접근해도 성능과 특성이 같기 때문이다.  
Swapping에 대해 잠깐 얘기하면 `Page-in`, `Page-out`에 대해 얘기하면  
+ Page in : Disk로 내려갔던 Page 하나를 물리 메모리 공간에 할당한다.  
+ Page out : 물리 메모리 공간 확보를 위해 Page Frame하나를 Disk에 Swap한다.  

Page 하나의 사이즈는 4KB이며,  32bit OS에서 한 프로세스의 Page table의 크기는 4MB이다.

![Paging-table](https://github.com/lee20h/blog/assets/59367782/e0efa60e-3d82-4cf2-83a2-de409470b8b5) 

## Paging 문제점 해결
`TLB`와 `Hierarchical Page Tables` 두 가지로 Paging의 문제점을 해결한다.

두 개념 다 `Paging`기법에서 문제점을 해결하기 위해 사용한다. 먼저 TLb는 Address Translation으로 인한 속도 저하 문제 즉, Performance 측면에서 해결하기 위해 사용되고, Hierarchical Page Tables는 Utilization 측면에서 Page table의 크기가 너무 커지는 문제를 해결하기 위해 사용된다.

### TLB

먼저 TLB에 대해 알아보면, Translation Look-aside Buffers의 약자로 Cache의 일종으로 생각하면 이해하기 좋다. Paging에서의 Frame number와 Page number을 쌍으로 저장하여 Cache와 같은 역할을 하는 버퍼이다. Parallel Search을 하게 되어 동시에 Page number, Frame number 쌍에 동시에 다 접근해서 원하는 Page Number을 통해 Frame Number을 접근하는데 이때, 어떤 number에 접근해도 O(1)으로 시간복잡도가 일정하다. 예전에 공부했던 Context Switch에서의 Overhead중 하나인 TLB Flush가 여기서 일어나게 되는데 프로세스 A의 Page, Frame number 쌍을 저장해놨다가 프로세스 B로 Context Switch한 경우 버퍼를 다 비우고 다시 채우게 된다. Cache 개념으로 처음에 채워지는 쌍들은 모두 Miss이기 때문에 Overhead가 따르게 된다.  이 부분을 생각해서 `Tagged TLB`개념이 나오는데 이 개념은 PID와 비슷한 ASID를 TLB에 둬서 각각의 숫자쌍이 어떤 프로세스의 것인지 표기하는 것이다. 이때는 TLB Flush을 하지 않아서 A -> B -> A로 Context Switch 되었을 때 버퍼에 A의 것이 하나라도 남아있다면 Hit을 해서 속도를 올리겠다는 개념이다. 하지만 버퍼에 공간을 하나 추가한다는 것은 돈이 많이 들어 사용하지 않는다고 하였다.  
이러한 문제를 배우고 다른 특징을 또 배웠는데 그것은 Locality 즉 지역성이다. for loop에 의해서 계속 메모리를 참조하게 되면 근처 지역에서 벗어나지 않고 비슷한 공간을 계속 참조하는 특성을 가지고 있다. 따라서 Hit확률이 거의 90%라고 할 수 있다고 한다. 이러한 특징때문에 TLB는 아직도 사용하고 있으므로 중요하다고 한다.  

![TLB](https://github.com/lee20h/blog/assets/59367782/a3977d40-e847-417b-9396-28e803fab75e)

TLB는 MMU에 붙어있는 장치인데, 위 그림에서 TLB Hit되어 물리메모리의 접근하는 부분이 MMU의 역할이라고 볼 수 있다. 따라서 Hit가 이뤄질 경우 MMU안에서 다 해결이 된다는 말로 속도가 빠르다. 그리고 위에서 말한 것과 같이 TLB에 동시에 접근하는 것을 볼 수 있다. 중요한 것은 TLB miss와 TLB에 동시에 접근해서 TLB에 있으면 miss 부분을 버리고 TLB에 없으면 바로 miss부분에 진입해서 물리메모리에 접근한다는 것이 중요하다고 강조하셨다.  
Effective Access Time(EAT) = TLB access time + Hit case + Miss case (α = hit ratio, t = Access time for TLB, m = Access time for memory)  
= t + α * m + 2 * m * (1-α)  이때의 2는 Page table과 Memory에 접근하는 시간이다.  
contiguous했을 때 100ns로 접근한다고 했을 때 Paging에서 Hit ratio가 매우 높다면 메모리 접근을 아무리 많이해도 Overhead가 크게 차이가 나지 않아서 Paging을 비로서 사용할 수 있다. 메모리 접근을 많이 해도 상관없다는게 포인트로, 다음에 설명할 Hierarchical Page Tables와 연관이 깊다.

### Hierarchical Page Tables

여러가지 Page table의 구조가 있는데 Page 각각의 사이즈가 4KB로 Entry을 다 고려하면 4MB의 공간을 잡아 먹게 된다. 이 용량을 줄이기 위해
- Hierarchical Paging
- Hashed Page Tables
- Inverted Page Tables

이렇게 세 가지가 있으나 Hierarchical Paging을 제외하고는 단점이 존재해서 중심으로 다룰 것은 Hierarchical Paging이다.  
![2-level-P-T](https://github.com/lee20h/blog/assets/59367782/a3a1035d-adcf-4914-9d8f-439887455eae)

개념은 Page Table들을 Paging해서 차지하는 공간을 드라마틱하게 줄인다는 것이다. Logical Address Space에서 0x0000 0000 중 0x0000 0|000으로 나뉘어서 앞의 5개는 Page number, 뒤에 3개는 Offset을 의미한다. 0x0000 0000은 Page Table의 0번의 0으로 indexing하여 접근하게 된다. Level 1의 Page table은 4KB을 차지하고 Frame number로 5000, 5001, 5002을 가지고 있다. 이 Frame number는 물리 주소공간에서 마지막에 연속적이지 않은 Page Table을 구성할 때 사용된다. 그리고 또 4 KB이므로 level 2로 연결 될 때 인덱싱해온 주소에 맞춰서 1024을 곱해서 찾아간다. Level 2에서는 마찬가지로 주어진 Number로 물리 주소공간에 연결을 한다.  
이렇게 다 연결하고 보면 이전에는 4MB를 차지한 반면에 지금은 고작 3 + 3 + 1 page로 7page로 완성을 하였다. 드라마틱하게 공간이 줄어든 것을 볼 수 있었다.

공부한 부분 중 핵심을 꼽은 것은 다 정리했으나, 이제 자잘한 부분은 넘어갔으니 짚고 넘어가보자.  
Page table에 Valid bit을 둬서 Frame number가 값이 어떤 것이 들어있을 때 이 값의 유효성을 체크해준다. v와 i로 valid와 invalid를 체크해주며 valid할 때만 접근을 허용한다.

Share Pages은 프로세스끼리 같은 물리 주소공간에 접근할 수 있게 같은 frame, page number 쌍을 갖게 하는 것이다. 이 때 프로세스 별로 접근권한을 둬서 권한 밖의 행동을 하게 되면 MMU가 제지하고 종료시키도록 할 수 있다.

### 나머지 사용하지 않는 이유

아까 그냥 넘긴 Hashed와 Inverted Page Table에 대해 짧게 말해보면 Hashed는 똑같이 Hashing 기법을 통해서 접근하게 만드는 것이다. 단점으로는 원하는 테이블을 찾기 위해서 값을 계속 연결되어 찾아가야한다는 점이 있다. 이걸 개선하기 위해 page을 더 두게 되면 위에서 한 Hierarchical Paging와 같게된다. Inverted은 말 그대로 Page number -> Frame number얻어 오던 Paging에서 반대로 Frame number -> Page number을 얻게 하는 방식이다. 이 방식은 pid을 두고 찾을 수 있으나, 선형탐색을 통해서 인덱싱을 해야하므로 Overhead가 분명하게 존재한다. 따라서 이러한 방법들은 잘 사용되지 않는다.  