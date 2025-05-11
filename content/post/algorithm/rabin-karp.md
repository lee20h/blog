---
title: "Rabin Karp에 대해 알아보자"
date: 2022-04-12T22:04:40+09:00
tags:
  - rabin-karp
categories:
  - algorithm
publishResources: true
---

# 라빈-카프(Rabin-Karp)

라빈-카프 알고리즘은 문자열 매칭 알고리즘 중 하나이다. 항상 빠르지는 않지만 일반적인 경우 빠르게 작동하는 간단한 구조의 문자열 매칭 알고리즘이라는 점에서 자주 사용된다.

해시 기법을 사용한다. 해시란, 일반적으로 긴 데이터를 상징하는 짧은 데이터로 바꾸어주는 기법이다. 상징하는 데이터로 바꾸어 처리하게 되면, 충돌이 없다는 가정하에 연산 속도가 O(1)에 달한다.

## 라빈-카프 해시 함수

- abacaaba의 해시 값

97 * 2^7 + 98 * 2^6 + 96 * 2^5 + 99 * 2^4 + 97 * 2^3 + 97 * 2^2 + 98 * 2^1 + 97 * 2^0 = 24,833

즉, 각 문자의 아스키 코드 값에 2의 제곱 수를 차례대로 곱하여 더해 준 것이다.

물론 충돌도 가능하지만 충돌율이 낮아서 사용할 수 있다. 충돌 시에는 포인터를 이용한 연결 자료구조를 이용해 해결한다.

라빈 카프 알고리즘은 여러 개의 문자열을 비교할 때 항상 해시 값을 구하여 비교하고, 해시 값은 거의 일치하지 않기 때문에 긴 글과 부분 문자열의 해시 값이 일치하는 경우에만 문자열을 처음부터 끝까지 재검사하여 일치하는지 확인하면 된다.

## 예시 코드
```cpp
void findString(string parent, string pattern) {
	int parentSize = parent.size();
	int patternSize = pattern.size();
	int parentHash = 0, patternHash = 0, power = 1;
	for (int i=0; i <= parentSize - patternSize; i++) {
		if(i == 0) {
			for (int j=0; j < patternSize; j++) {
				parentHash = parentHash + parent[patternSize - 1 -j] * power;
				patternHash = patternHash + pattern[patternSize - 1 -j] * power;
			}
			if(j < patternSize -1)
				power = power * 2;
		}
		else {
			parentHash = 2* (parentHash - parent[i-1] * power) + parent[patternSize-1+i];
		}
		if(parentHash == patternHash) {
			bool found = true;
			for (int j=0; j<patternSize; j++) {
				if(parent[i+j] != pattern[j]) {
					found = false;
					break;
				}
			}
			if(found)
				cout << i+1 << "번째 위치서 발견\n";
		}
	}
}
```

이 때 해시 값의 일치 여부를 검증하기 위해 나머지 연산을 이용하지만, 일반적으로 생각하면 오버 플로우가 발생해서 값이 음수로 변해도 연산 자체에서 구해지는 값은 같으므로 나머지 연산을 안해줘도 된다. 하지만, 안정적인 프로그램을 작성하거나 필요에 의해서 나머지 연산을 입혀주는 것이 좋다.

추가적으로, 나머지 연산을 해서 배열에 값을 넣어야 할 때, 함수를 통해서 나머지 연산을 하게 된다. 이 때 음수의 값은 배열의 인덱스가 될 수 없으므로 이러하게 나머지 연산을 하게 된다.

```cpp
int mod(long long n) {
	if(n >= 1) return n % MOD;
	return ((-n/MOD+1)*MOD + n) % MOD;
}
```

아직도 제대로 이해는 하지 못했으나 내용을 적어보려한다.  
(n에 근접한 MOD의 배수) + n(음수) = n(음수)의 절댓값과 n보다 크고 n에 근접한 MOD의 배수와의 차이점 = MOD 배수 - |n| = MOD % n(n이 양수인 경우)와 동일하다.

따라서 양수일 때는 `n % MOD`, 음수일 때는 `MOD 배수 - |n| % MOD`로 값이 반환된다는 것을 볼 수 있다.

그 이유로는 절댓값이 같은 양수와 해시 값이 겹치지 않도록 하기 위함으로 보인다.  