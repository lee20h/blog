---
title: "문자열 검색에 대해 알아보자"
date: 2022-03-21T22:06:00+09:00
tags:
  - string
categories:
  - algorithm
published: true
---


# 문자열 검색

## Naive Matching
먼저 `Naive Matching 알고리즘`은 찾고자하는 문자열과 주어진 문자열을 하나하나 비교하면서 처음부터 끝까지 확인하는 방식이다. 최악의 경우 시간복잡도 O((n-m)m)이 걸린다. 하나하나 일일이 검사하므로 작은 문자열이라면 시도할만 하다. 하지만 대부분이 큰 문자열이 주어지므로 최대한 피하자.

## Rabin-Karp
두번째론 `라빈-카프` 알고리즘이 있다. 이 알고리즘은 해시를 이용하여 해시끼리 비교하는 알고리즘이다. 먼저 찾으려는 문자열의 해시값을 구하고 주어진 문자열에서 찾으려는 문자열의 크기만큼 잡고 해시값을 구해서 비교해가며 찾으면 된다. 하지만 문자열이 매우 커질 수록 충돌이 일어날 가능성이 커진다고 한다. (약 1억자리 이상)  
그리고 반복되며 찾을 때마다 아래처럼 이용하면 된다. 시간복잡도는 평균적으로 O(n+m)이다.  

![image](https://github.com/lee20h/blog/assets/59367782/37a2fe76-bebc-4647-bd58-438e6e41344a)

## KMP
세번째론 `KMP` 알고리즘이 있다. 이 알고리즘은 접두사와 접미사에 대해 알아야한다. 예를 들어서 ABCAB란 문장이 있다하자.  
접두사는 A, AB, ABC, ABCB, ABCBA  
접미사는 B, BA, BAC, BACB, BACBA 이다.  
즉, 앞에서 자르기 시작한 것과 뒤에서 자르기 시작한 것의 차이이다.  
이제 pi라는 배열을 만들어 줄건데 pi란 배열은 접두사와 접미사가 같은 경우를 조건으로 두고 그 단어를 pi라는 배열에 넣는 것이다. 왜냐 불필요한 부분을 넘기고 그 전에 찾을 문자열이 포함되었는지 확인 해야하기 때문이다. 만약, 무작정 인덱스 크기를 잡고 넘기게 되면 그 넘어간 인덱스 중에 찾아야할 문자열의 일부가 있을 수 있기 때문이다. 시간복잡도는 O(n+m)이다.
```
KMP(A[ ], P[ ]) 
{ 
	preprocessing(P); 
	i  ← 1;  ▷ 본문 문자열 포인터 
	j  ← 1;  ▷ 패턴 문자열 포인터 
	▷ n: 배열 A[ ]의 길이, m: 배열 P[ ]의 길이 
	while (i ≤ n) { 
		if (j = 0 or A[i] = P[j]) 
			then { i++;  j++; }  
			else  j ← π [j];
		if (j = m+1) then { 
 	A[i-m]에서 매치되었음을 알림; 
  	j ← π [j];  
		}  
	} 
} 

preprocessing(P) 
{ 
	i  ← 1;  ▷ 본문 문자열 포인터 
	k  ← 1;  ▷ 패턴 문자열 포인터 
	while (j ≤ m) { 
		if (k = 0 or A[j] = P[k]) 
			then { j++;  k++; π [j] ← k; }  
			else  k ← π [k];
		if (j = m+1) then { 
 	A[i-m]에서 매치되었음을 알림; 
  	j ← π [j];  
		}  
	} 
}
```

## Boyer-Moore
마지막으로, `보이어-무어` 알고리즘인데, 이 알고리즘은 이해하기 어렵다. [참조pdf](http://www.cs.jhu.edu/~langmea/resources/lecture_notes/boyer_moore.pdf) 이 곳을 통해 그림으로 이해를 돕고자 한다.
[보이어-무어 계산](https://personal.utdallas.edu/~besp/demo/John2010/boyer-moore.htm)   
KMP 알고리즘과 비슷한 개념을 가지고 있다. 최대한 많이 건너뛰면서 비교를 하는 것이다. 하지만 다른 알고리즘과 다른 부분은 오른쪽에서 왼쪽으로 문자열을 비교한다. 물론 이동은 왼쪽에서 오른쪽으로 한다. 이 알고리즘이 대부분의 프로그램의 문자열 검색 방법으로 사용되고 있다.  
오른쪽 끝의 문자가 일치 하지 않고 주어진 문자에 찾아야하는 문자가 존재하지 않으면 패턴 길이만큼 이동한다.  
불일치가 발생하면 최대의 효율을 내는 두가지 방법 중 고른다.
1) 나쁜 문자 이동 (Bad Character Shift)
2) 착한 접미부 이동 (Good Suffix Shift)
    - 나쁜 문자 이동은 (나쁜 문자란, 본문 문자 중 패턴과 일치 하지 않은 문자)
        * 패턴의 오른쪽부터 비교해서 불일치 지점 찾기.
        * 본문의 불일치 지점 문자와 일치하는 패턴의 가장 오른쪽 지점만큼 이동 시킨다.
    - 착한 접미부 이동은 (착한 접미부란, 본문 문자 중 패턴의 접미부와 일치하는 문자)
        * 첫번째 경우엔 착한 접미부와 동일한 문자가 왼쪽 문자열중 존재
        * 두번째 경우엔 패턴에 착한 접미부가 없지만 접두부가 이리하는 경우 KMP와 비슷  