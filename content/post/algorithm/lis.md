---
title: "LIS에 대해 알아보자"
date: 2022-05-02T21:53:52+09:00
tags:
  - LIS
categories:
  - algorithm
publishResources: true
---


# LIS
LIS(Longest Increasing Subsequence) : 최장 증가 부분 수열

수열 하나가 주어졌을 때, 가장 긴 증가하는 부분 수열을 구할 때 3가지 방법이 있다.

- 완전 탐색을 이용한 방법(O(2ⁿ))
- DP를 이용한 방법 (O(n²))
- 이진 탐색을 이용한 방법 (O(n logn))

## 완전탐색을 이용한 방법
완전 탐색의 경우에는 하나하나 다 비교해보면 되므로 모든 증가 부분수열을 고려한다. 배열을 받아서 원 배열에서 증가 부분 수열의 첫번째 수를 선택한 뒤, 다음 수의 조건(첫 수보다 원 배열에서 뒤에 있고 큰 후보)에 해당하는 값들의 배열을 꾸려 재귀를 해나가면 될 것 같다.

어떤 수가 나올지는 상관없이 자신보다 작은 숫자인지 확인하고 이전 값에서 1을 계속 더해가면 해당 LCS의 길이를 구할 수 있다.

## DP를 이용한 방법
DP를 이용한 방법은 아래와 같이 알고리즘을 짜볼 수 있다.
1. 수열의 길이와 같은 dp배열을 하나 선언한다.

2. 수열을 처음부터 끝까지 순서대로 1개씩 탐색한다. ( 현재 위치 = i )
    1. dp[i]에 넣을 값을 초기화해준다. (val)

    2. 현재 위치(i)보다 이전에 있는 원소(j) 중에서 현재 원소보다 작은지 체크한다. (크거나 같으면 LIS 불가능)

    3. 현재 원소보다 작다면, dp[j]가 val 보다 큰지 체크한다. 이 때 val보다 크다면 j번째 원소를 포함했을 때가, 지금까지 확인한 최장 증가 부분 수열보다 더 길다는 의미이므로 val에 dp[j]를 할당해준다.

    4. 현재 원소도 포함해주어야 하므로 최종적으로 dp[i]에 val + 1을 할당해준다.
3. dp배열의 원소 중에서 가장 큰 값을 출력한다.

```cpp
for(int i=1;i<N;i++) {
	for(int j=0;j<i;j++) { 
		if (array[i] > array[j] && dp[j] + 1 > dp[i])
			 dp[i] = dp[j] + 1; // 증가 수열
	}
	if (max < dp[i])
		max = dp[i]; 
	
}
```
처음에 dp 배열의 값들을 1로 초기화 해주어야하며, if문의 조건에 `dp[j] + 1 > dp[i]`가 들어간 이유는 예를 들어서 10 20 30 20 인 경우 값이 20을 만나게 되면 값이 줄어드는 것을 볼 수 있다. 이 부분을 보장해주기 위해 해당 조건을 추가한 것이다.

## 이진 탐색을 이용한 방법

해당 부분은 잘 정리된 블로그 포스팅을 참고해서 썼다.

- 배열 마지막 요소보다 새로운 수가 크다면, 배열에 넣는다.
- 그렇지 않다면, 그 수가 들어갈 자리에 넣는다. (이분 탐색을 통해 들어갈 자리를 찾는다)

LIS의 갯수만 구할 때의 가장 이해하기 쉬운 코드를 찾아보았다.
```cpp
dp[0] = array[0];
int idx = 0;
for (int i = 1; i < n; i++) {
	if (dp[idx] < array[i]) { 
		dp[++idx] = array[i]; 
	}
	else {
		int ii = lower_bound(array.begin(), array.end(), idx);
		dp[ii] = array[i];
	} 
}
// ans => idx + 1
```

혹은 이런 방식으로 짧게 구현하는 방법도 있다
```cpp
for (int i=0; i<n; i++) {
	vector<int>::iterator iter = lower_bound(dp.begin(), dp.end(), arr[i]);
	if(iter == dp.end()) dp.push_back(arr[i]);
	else *iter = arr[i];
}
cout << dp.size();
```
`lower_bound`을 통해서 못 찾으면 push_back, 찾으면 해당 값을 배열의 값으로 치환해준다.

## LIS 경로 추적
![image](https://github.com/lee20h/blog/assets/59367782/2d8b0bbb-645c-4912-9377-71c5f3cb3712) 
위 그림을 통해서 소스의 전체적인 느낌을 파악한 뒤 다른 문제에서 요하는 경로 추적 또한 코드로 보았다.

```cpp
dp[0] = array[0];
tracking[0] = new Pair(0, array[0]);
int idx = 0;
for (int i = 1; i < n; i++) { 
	if (dp[idx] < array[i]) { 
		dp[++idx] = array[i];
		tracking[i] = new Pair(idx, array[i]);
	}
	else { 
		int ii = lower_bound(idx, array[i]);
		dp[ii] = array[i];
		tracking[i] = new Pair(ii, array[i]);
	}
}
```
이후에는 스택에서 인덱스와 pair의 first을 비교하여 스택에 넣어준 다음 꺼내주게 되면 순서대로 출력할 수 있다.

```cpp
lis.push_back(v[0]);
p.push_back({0, v[0]});

for (int i=1; i<n; i++) {
	int idx = lower_bound(lis.begin(), lis.end(), v[i]) - lis.begin();
	if(idx >= lis.size())  {
		lis.push_back(v[i]);
	}
	else {
		lis[idx] = v[i];
	}
	p.push_back({idx, v[i]});
}
stack<int> st;
int len = lis.size();
cout << len << '\n';
for (int i=n-1; i>=0; i--) {
	if (p[i].first == len-1) {
		st.push(p[i].second);
		len--;
	}
}
while(!st.empty()) {
	cout << st.top() << ' ';
	st.pop();
}
```
나는 이런 식으로 만들어서 pair로 추적하는 아이디어를 집어 넣어서 구현하였다.

[그림 및 소스 출처](https://mygumi.tistory.com/303)  