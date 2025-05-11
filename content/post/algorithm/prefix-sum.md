---
title: "연속합에 대해 알아보자"
date: 2022-01-03T22:13:58+09:00
tags:
  - prefix sum
categories:
  - algorithm
publishResources: true
---
# 연속합
[CSES](https://cses.fi/problemset/task/1643) [BOJ](https://www.acmicpc.net/problem/1912) [SWEA](https://swexpertacademy.com/main/code/problem/problemDetail.do?contestProbId=AWXQm2SqdxkDFAUo&categoryId=AWXQm2SqdxkDFAUo&categoryType=CODE&&&)
## 초기 생각
기본적으로 비슷한 문제로 처음에 떠올린 방법은 시간 복잡도 O(n)을 가졌다. 하지만 시간초과로 제대로 해결하지 못하였는데 이 부분은 O(1)로 구현해야했다. 그래서 밑과 같은 방식으로 복습하여 구현하였다.
## 알고리즘
```cpp
int best = v[0], sum = 0; // best인 값은 초기에 첫번째 배열로 설정하고 합은 0으로 설정한다.
for (int i=0; i<n; i++) {
	if(sum + v[i] < v[i]) sum = v[i]; // if(sum < 0)과 동일 sum이 0보다 작으면 v[i]로 새롭게 초기화시켜준다.
	else sum += v[i]; // sum이 양수거나 0과 같다면 sum에 현재 인덱스부분을 더해준다.
	best = max(best,sum); // best는 best와 sum중 큰 값으로 갱신해준다.
}
```
와 같은 알고리즘으로 구현하게 되면 연속적인 값들의 합을 구할 때 시간복잡도 O(1)로 구할 수 있다.