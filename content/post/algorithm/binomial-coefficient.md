---
title: "이항계수에 대해서 알아보자 (feat. 페르마의 소정리)"
date: 2022-02-04T23:11:48+09:00
tags:
  - 이항계수
categories:
  - algorithm
publishResources: true
---

# 이항 계수

## 이항 계수란
n개의 원소에서 r개의 원소를 뽑아내는 경우의 수  
즉, 조합. 수식으로는 n!/(r!(n-r)!)

nCr = (n-1)Cr + (n-1)C(r-1)


## 페르마의 소정리
p가 소수, a가 정수 일때,  
![페르마 소정리](https://wikimedia.org/api/rest_v1/media/math/render/svg/7ff656f721894b9a50a2b1d18538463a6a4ec15f)
**(대수학에서의 합동(≡)이란 mod p 연산시에 값이 동일한 경우)**  

양쪽에 a로 나눠주게 되면 

![페르마 소정리](https://wikimedia.org/api/rest_v1/media/math/render/svg/2149302899fcbf99c1b46c536549f7ed7b0a6b2b)  

한번 더 a로 나눠주게 되면 a의 역수가 a^(p-2)란 걸 증명할 수 있다.  
문제의 식인 nCr = n!/(r!(n-r)!) % p  
-> A = n!, B = (r!(n-r)!)  
-> (A*B^(-1)) % p  
-> ((A % p)*(B^(-1) % p)) % p  
-> B^(-1)은 B의 역수  
-> (A*B^(p-2)) % p

## 문제 해결
factorial을 구하고 factorial의 역을 구해서 원하는 값을 집어넣어 답을 구한다.

## 추가적으로..
**거듭 제곱시에 분할 정복을 사용하면**
```cpp
long long power(ll x, ll y) {  
	long long result = 1;  
	while(y) {  
		if (y%2) {  
			result *= x;
			result %= P;
		}
		x *= x;
		x %= P;
		y /= 2;
	}
	return result;
}  
```

### 문제

[SWEA](https://swexpertacademy.com/main/code/problem/problemDetail.do?contestProbId=AWXGKdbqczEDFAUo&categoryId=AWXGKdbqczEDFAUo&categoryType=CODE)
[BOJ](https://www.acmicpc.net/problem/11401)
