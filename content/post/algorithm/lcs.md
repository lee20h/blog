---
title: "LCS에 대해 알아보자"
date: 2022-04-28T21:53:52+09:00
tags:
  - LCS
categories:
  - algorithm
published: true
---

# LCS
LCS는 두 가지로 나뉘어진다.

`최장 공통 부분 문자열(Longest Common Substring)`과 `최장 공통 부분 수열(Longest Common Subsequence)` 두 가지로 나뉘는데 비슷하나 차이점이 뚜렷하다. 그 차이점은 해당 부분의 연속 여부이다. 아래 예시를 봐보자.

A**BCD**FEF  A**BCD**F**EF**  
**BCD**EF	 **BCDEF**

해당 두 개의 문자열이 있다고 가정해보자. 먼저, 최장 공통 부분 문자열의 경우에는 `BCD` 3개를 갖고, 최장 공통 부분 수열의 경우에는 `BCDEF` 5개를 갖게 된다.

## 최장 공통 부분 문자열
![image](https://github.com/lee20h/blog/assets/59367782/2b4e8289-6b31-4698-b6e6-8457dc5f36ef)

### 코드
```cpp
for (int i=1; i<=A.length; i++) {
	for (int j=1; j<=B.length; j++) {
		if(A[i-1] == B[j-1]) {
			LCS[i][j] = LCS[i-1][j-1]+1
			if(ans < LCS[i][j])
				ans = LCS[i][j]
		}
	}
}
```

## 최장 공통 부분 수열
![image](https://github.com/lee20h/blog/assets/59367782/54f11bf6-5059-4db7-bacb-b09b9200c5c4)  
X와 Y는 비교할 각 문자열, i와 j는 문자열의 각 인덱스이다. 구현에 있어 3가지가 필요하다.

1. 처음엔 편의를 위해서 빈 수열로 채워준다.

|   | 0 | A | G | C | A | T |
|---|---|---|---|---|---|---|
| 0 | Ø | Ø | Ø | Ø | Ø | Ø |
| G | Ø |   |   |   |   |   |
| A | Ø |   |   |   |   |   |
| C | Ø |   |   |   |   |   |

2. X와 Y의 문자가 같은 경우이다. 이때 예시를 보자.
```cpp
stirng a = "ABCD"
string b = "AEBD"
LCS("ABCD", "AEBD") = 1 + LCS("ABC", "AEB")
```
`(ABCD와 AEBD의 길이) = (ABC, AEB를 비교했을 때의 길이 + 1)`

3. X와 Y의 문자가 다를 경우
```cpp
LCS("ABC", "AEB") = MAX(LCS("AB", "AEB"), LCS("ABC", "AE"))
```
(AB, AEB 길이)와 (ABC,AE 길이) 중 큰 값을 (ABC, AEB 길이)에 대입한다.

### 코드
이러한 3가지를 코드로 나타내면
```cpp
for(int i=1;i<=A.length;i++) { 
	for(int j=1;j<=B.length;j++) { 
		if (A[i-1] == B[j-1]) { 
			LCS[i][j] = LCS[i-1][j-1] + 1; 
		} 
		else { 
			LCS[i][j] = Math.max(LCS[i][j-1], LCS[i-1][j]); 
		}
	}
}
```

두 문자열의 각 문자를 비교하지 않고 선행 문자를 끼고 문자열로써 비교를 하는 것이다. 따라서 LCS 배열 마지막에는 LCS의 길이를 볼 수 있게 된다.

### LCS 부분 수열
추가적으로, LCS에 해당하는 부분 수열을 알고 싶다면 표로 생각해보자. LCS에서 DP와 같이 전의 값을 이용해서 값을 찾아낸다. 이 때 값이 변하는 구간은 항상 대각선으로 변하게 된다. 따라서, 대각선인 시점을 체크해서 그 전까지는 같은 수열을 갖다가 대각선 이후로 수열이 바뀌는 것을 볼 수 있다. 이것을 코드로 봐보자.

```cpp
for (int i = 1; i <= A.length; i++) {
    for (int j = 1; j <= B.length; j++) {
        if (A[i - 1] == B[j - 1]) {
            LCS[i][j] = LCS[i - 1][j - 1] + 1;
            solution[i][j] = "diagonal";
        }
		else {
            LCS[i][j] = Math.max(LCS[i][j - 1], LCS[i - 1][j]);

            if (LCS[i][j] == LCS[i - 1][j]) 
                solution[i][j] = "top";
            else 
                solution[i][j] = "left";
            
        }
    }
}

int a = A.length;
int b = B.length;

while(solution[a][b] != null) {
    if (solution[a][b] == "diagonal") {
        sb.append(A[a-1]);
        a--;
        b--;
	}
    else if (solution[a][b] == "top") 
        a--;
    else if (solution[a][b] == "left") 
        b--;
}
sb.reverse.toString(); // 최장 공통 부분 수열 리스트
```

여기서 여러 개의 문자열을 비교할려면 배열을 늘려주자.