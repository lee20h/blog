---
title: "회문(palindrome)에 대해 알아보자"
date: 2021-09-21T22:17:22+09:00
tags:
  - palindrome
categories:
  - algorithm
published: true
---

# 회문
회문 [SWEA](https://swexpertacademy.com/main/code/problem/problemDetail.do?contestProbId=AV14QpAaAAwCFAYi&categoryId=AV14QpAaAAwCFAYi&categoryType=CODE) 회문2 [SWEA](https://swexpertacademy.com/main/code/problem/problemDetail.do?contestProbId=AV14Rq5aABUCFAYi&categoryId=AV14Rq5aABUCFAYi&categoryType=CODE)  
회문이란 앞으로 읽어도 뒤로 읽어도 같은 단어이다. 회문1 같은 경우에는 8x8 단어판와 길이가 주어지면 그 단어판에서 길이만큼의 회문 갯수를 세면 된다.
## 회문1

```cpp
string temp;
for (int i=n; i<n+len; i++) {
	temp += v[i][m];
}
string temp2 = temp;
reverse(temp.begin(),temp.end());
if(temp == temp2)
	ans++;
```
와 같이 뒤집은 문자와 문자가 같으면 회문으로 처리했다.

## 회문2
회문2의 경우에는 회문1에서 단어판을 100x100으로 키우고 최대의 길이의 회문을 찾으면 된다.
```cpp
int x(int len, int pos_y, int pos_x) {
	for (int i=0; i<len/2; i++) {
		if(v[pos_y][pos_x + i] != v[pos_y][pos_x + len - 1 - i])
			return 0;
	}
	return len;
}
int y(int len, int pos_y, int pos_x) {
	for (int i=0; i<len/2; i++) {
		if(v[pos_y + i][pos_x] != v[pos_y + len - 1 - i][pos_x])
			return 0;
	}
	return len;
}

for (int k=1; k<100; k++) {
	for (int i=0; i<=100-k; i++) {
		for (int j=0; j<100; j++) {
			temp = y(k,i,j);
			len = max(len,temp);
		}
	}
	for (int i=0; i<100; i++) {
		for (int j=0; j<=100-k; j++) {
			temp = x(k,i,j);
			len = max(len,temp);
		}
	}
}
```
k가 길이, i가 열, j가 행으로 잡고 길이가 k일때 (i,j)에서 시작하는 회문을 찾고 있으면 길이를 반환해서 len값과 비교해서 큰 값을 len에 넣는다.  

