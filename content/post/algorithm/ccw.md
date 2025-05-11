---
title: "CCW에 대해 알아보자"
date: 2021-12-16T21:51:07+09:00
tags:
  - CCW
categories:
  - algorithm
publishResources: true
---

# CCW

CCW란, 점 3개가 이차원 평면에 존재하여 점들을 이은 선분이 어떤 방향성을 나타내는지 확인하는 알고리즘이다.

가능한 경우의 수는 반시계 방향, 시계 방향, 일직선 총 3가지가 있다. 각각 반시계 방향인 경우에는 1, 일직선은 0, 시계 방향을 -1이라고 했을 때 그림은 아래와 같다.

![image](https://github.com/lee20h/blog/assets/59367782/3e55918d-6903-483c-8895-30fa0755c830)

## 소스

```cpp
int ccw(int x1, int y1, int x2, int y2, int x3, int y3) {
    int temp = x1*y2+x2*y3+x3*y1;
    temp = temp - y1*x2-y2*x3-y3*x1;
    if (temp > 0) {
        return 1;
    } else if (temp < 0) {
        return -1;
    } else {
        return 0;
    }
}
```

## 사용 예시

### 두 선분의 교차 여부

평면상에서 두 선분이 주어졌을 때 교차 여부를 판단할 때 CCW 알고리즘을 사용하게 되면 쉽게 구할 수 있다.

![image](https://github.com/lee20h/blog/assets/59367782/101e6eda-50b5-4342-a277-abb886631853)

선분 AB를 기준으로 선분 CD의 꼭짓점인 C와 D는 서로 다른 방향에 위치하게 된다. 점 A,B,C는 시계 방향, 점 A,B,D는 시계반대 방향의 방향관계를 갖으므로 CCW의 반환값이 서로 다르다. 따라서 `CCW(A,B,C) * CCW(A,B,D) < 0`을 만족한다면 두 선분은 교차한다.

교차하지 않을 때는 `CCW(A,B,C) * CCW(A,B,D) > 0`을 만족한다는 것을 쉽게 알 수 있다. 선분의 꼭짓점이 다른 선분 위에 위치하게 될 때 `CCW(A,B,C) * CCW(A,B,D) = 0`과 같이 값이 나온다는 것도 같이 알아두면 좋다.

이때 문제점은 선분과 직선의 차이이다. 두 직선인 경우에는 위와 같이 CCW 값의 곱을 통해서 교차 여부를 알 수 있지만 두 선분의 경우에는 만나지 않더라도 CCW의 값의 곱이 음수가 될 수 있다.
![image](https://github.com/lee20h/blog/assets/59367782/c8d9a46c-23bc-4f0d-a54a-1d411af606ca)

### 다각형의 넓이
n각형의 경우 n이 매우 크게 되는 경우나 오목 다각형인 경우에는 넓이를 구하기가 매우 어려워진다. 이때도 CCW 알고리즘을 이용하면 쉽게 해결할 수 있다.

CCW 알고리즘에서는 이차원 평면상 두 벡터의 수직벡터의 z좌표를 기준으로 삼기 때문에 외적을 통해서 z좌표를 구해야하나, 선형대수학에서 사용한 삼각형 넓이 구하는 사선 공식과 결과가 같아서 사선 공식을 그대로 사용하도록 한다.
![image](https://github.com/lee20h/blog/assets/59367782/c909caec-4910-42be-a11b-4103ce027833)

따라서, CCW 함수의 반환값을 절댓값을 취한 뒤 1/2만 곱하면 된다. 다각형 넓이 구하는 CCW 알고리즘을 과정으로 알아보자.

- 고정점 설정
- 고정점 기준으로 시계 or 시계반대 방향으로 꼭짓점을 돌면서 CCW 알고리즘 적용
- 절댓값을 취한 뒤 1/2 곱

내각이 180도보다 큰 다각형인 오목다각형에 경우에도 알고리즘은 똑같이 진행해주면 된다. 왜냐하면 내각이 180도보다 크게 되면 위의 과정대로 진행 중 외부의 넓이를 더하게 되는 경우가 있으나, 다른 꼭짓점들에 대해 CCW 알고리즘을 적용하게 되면 기존의 방향과 반대방향인 부분이 존재해서 해당 외부 넓이만큼 빼서 원하는 다각형의 넓이를 구할 수 있다.

### 볼록 껍질 (Convex Hull)
볼록 껍질이란, 2차원 평면 상에서 주어진 점들을 모두 포함하는 가장 작은 다각형을 의미한다. 다각형의 각 변들은 주어진 점을 양 끝점으로 갖는다.

블록 껍질을 구하기 위해 사용하는 알고리즘은 `그라함 스캔 알고리즘`이다.

- 주어진 점들 중 y좌표가 가장 작거나 혹은 y좌표가 가장 작은 점이 둘 이상이라면 x좌표가 가장 작은 점을 선택
- 선택한 점을 기준으로 나머지 점들을 반시계 방향으로 정렬
- 그라함 스캔 알고리즘 적용

그라함 스캔 알고리즘은 스택 자료구조를 이용한다. 위의 과정대로 처음 선택한 점을 스택에 넣고, 정렬된 점들을 차례대로 스택에 넣는다. 새로운 점을 스택에 push할 때 만약 스택에 점이 두개 이상이 있다면, 가장 최근에 push된 두 점을 이은 직선을 기준으로 새로운 점이 왼쪽에 있다면 push, 오른쪽에 있다면 pop하고 새로운 점을 push한다. 이때 사용하는 알고리즘이 CCW 알고리즘이다. CCW 값이 0보다 큰 경우 즉, 좌회전인 경우에는 스택에 push하게 된다.

코드

```cpp
bool comp_y(pos a, pos b) {
	if (a.y != b.y)	return a.y < b.y;
	else return a.x < b.x;
}

long long ccw(pos a, pos b, pos c) { //ccw
	return a.x * b.y + b.x * c.y + c.x * a.y - (b.x * a.y + c.x * b.y + a.x * c.y);
}

bool comp_c(pos a, pos b) { //반시계 정렬
	long long cc = ccw(v[0], a, b);
	if (cc != 0) return cc > 0; // 각도 작은 순
	else return (a.x + a.y) < (b.x + b.y); //일직선 있을경우 좌표가 작은
}

int main() {
    sort(v, v + N, comp_y);
	sort(v + 1, v + N, comp_c);		
	
	s.push(v[0]);
	s.push(v[1]);	
	pos first, second;

	for (int i = 2; i < N; i++) {
		while (s.size() >= 2) {
			second = s.top();
			s.pop();
			first = s.top();
			if (ccw(first, second, v[i]) > 0) { //좌회전
				s.push(second);
				break;
			}

		}
		s.push(v[i]);
	}

	cout << s.size();
}
```

그라함 스캔 알고리즘이 시간 복잡도가 O(n)이며, 반시계방향 정렬하는 시간 복잡도가 O(nlogn)이다. 따라서 블록 껍질를 구하는 시간 복잡도는 O(nlogn)이다.

[그림 출처](https://wogud6792.tistory.com/12)