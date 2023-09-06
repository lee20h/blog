---
title: "세그먼트 트리에 대해 알아보자"
date: 2022-01-12T22:10:17+09:00
tags:
  - segment tree
categories:
  - algorithm
published: true
---


# 세그먼트 트리

문제를 예로 들어서 세그먼트 트리를 얘기해보자. [참고](https://www.acmicpc.net/blog/view/9)를 내가 이해한 방향으로 적어보았다.

## 문제
배열 A가 있고, 여기서 다음과 같은 두 연산을 수행해야하는 문제를 생각해보자

1) 구간 l, r (l ≤ r)이 주어졌을 때, A[l] + A[l+1] + ... + A[r-1] + A[r]을 구해서 출력하기
2) i번째 수를 v로 바꾸기. A[i] = v
   수행해야하는 연산은 최대 M번이다.

세그먼트 트리나 다른 방법을 사용하지 않고 문제를 푼다면, 1번 연산을 수행하는데 O(N), 2번 연산을 수행하는데 O(1)이 걸리게 된다. 총 시간 복잡도는 O(NM) + O(M) = O(NM)이 나오게 된다.

2번 연산이 없다고 생각해보자.

수를 바꾸는 경우가 없기 때문에, 합도 변하지 않는다. 따라서, 앞에서부터 차례대로 합을 구해놓는 방식으로 문제를 풀 수 있다.

S[i] = A[1] + ... + A[i] 라고 했을 때, i~j까지 합은 S[j] - S[i-1]이 된다.

i~j까지 합은 A[i] + ... + A[j]인데, S[j] = A[1] + ... + A[j], S[i-1]= A[1] + ... + A[i-1] 이기 때문이다.

```cpp
S[0] = A[0];
for (int i=1; i<n; i++) {
    S[i] = S[i-1] + A[i];
}
```
여기서 2번 연산을 하려면, 수가 바뀔때마다 S를 변경해줘야 한다. 가장 앞에 있는 0번째 수가 바뀐 경우에는 모든 S 배열을 변경해야 하기 때문에, 시간복잡도는 O(N)이 걸리게 된다.

따라서, M과 N이 매우 큰 경우에는 시간이 너무 오래걸리게 된다.

이러한 부분을 해결하기 위해서 사용하는 자료구조가 `세그먼트 트리`이다.


## 세그먼트 트리
세그먼트 트리를 이용하게되면 해당 1번 연산과 2번 연산 모두 O(logN)으로 수행할 수 있다.

세그먼트 트리의 리프 노드와 리프 노드가 아닌 다른 노드의 의미는 다음과 같다.
- 리프 노드 : 배열 그 수 자체
- 다른 노드 : 왼쪽 자식과 오른쪽 자식의 합

따라서 노드의 번호가 `x`라면 왼쪽 자식의 번호는 `2*x`, 오른쪽 자식의 번호는 `2*x+1`가 되게 된다.

리프 노드가 10개 인 경우를 그림으로 보면 다음과 같다.

![image](https://github.com/lee20h/blog/assets/59367782/1a76cb8c-40bd-4ab3-a99b-183062e83d3b)

해당 노드에 적혀있는 숫자는 저장하고 있는 합의 범위를 나타낸 것이다. 노드의 번호의 경우에는 위에서 적은 것과 같이 왼쪽 자식의 번호는 해당 노드에 2를 곱한 값, 오른쪽 자식의 번호는 2를 곱한 뒤 1을 더한 값이 되게된다.

### 트리 제작
만약 N이 2의 제곱꼴이라면 완전 이진 트리로 높이가 `logN`이다. 리프 노드가 N개인 완전 이진 트리는 필요한 노드의 갯수가 `2*N-1`개이다.

2의 제곱꼴이 아니라면, 높이가 H = ⌈logn⌉ 이며, 총 세그먼트 트리를 만드는데 필요한 배열의 크기는 2^(H+1) -1개가 된다.

따라서 다음 코드로 Segment Tree를 만들 수 있다.
```cpp
// a: 배열 a
// tree: 세그먼트 트리
// node: 세그먼트 트리 노드 번호
// node가 담당하는 합의 범위가 start ~ end
long long init(vector<long long> &a, vector<long long> &tree, int node, int start, int end) {
    if (start == end) {
        return tree[node] = a[start];
    } else {
        return tree[node] = init(a, tree, node*2, start, (start+end)/2) + init(a, tree, node*2+1, (start+end)/2+1, end);
    }
}
```
start == end 인 경우는 node가 리프 노드인 경우이다. 리프 노드는 배열의 그 원소를 가져야하기 때문에 `tree[node] = a[start]`가 된다.  
node의 왼쪽 자식은 `node*2`, 오른쪽 자식은 `node*2+1`이 된다. 이때, node가 담당하는 구간이 `[stard,end]`라면 왼쪽 자식은 `[start, (start+end)/2]`, 오른쪽 자식은 `[(start+end)/2 +1, end]`를 담당한다. 따라서 재귀 함수를 이용해서 왼쪽 오른쪽 자식 트리들을 만들어서 합을 저장한다.

### 합 찾기
구간이 `[left, right]`로 주어졌을 때, 합을 찾으려면 루트부터 트리를 순회하면서 각 노드가 담당하는 구간과 left, right 사이의 관계를 봐야한다.

예를 들어서 [0,9]의 합을 구하는 경우에는 루트 노드 하나로 합을 알 수 있다.
![image](https://github.com/lee20h/blog/assets/59367782/2da9ce1d-7be2-4638-8680-f5fdd2fe7b3a)
또, [3,9]의 합을 구하는 경우에는 다음과 같다.  
![image](https://github.com/lee20h/blog/assets/59367782/008cf07a-102e-476a-bc23-6d617f5f78f9)

node가 `[start, end]`을 담당하는 경우에 해당 구간의 합을 구해야한다. 이 때 4가지로 나뉘게 된다.

1) [left, right]와 [start,end]가 겹치지 않은 경우
2) [left, right]와 [start,end]를 완전히 포함하는 경우
3) [start, end]]와 [left, right]를 완전히 포함하는 경우
4) [left, right]와 [start,end]가 겹치져 있는 경우 (1,2,3 제외한 나머지 경우)

`1번`의 경우엔 `if (left > end || right < start)`로 나타낼 수 있다. `left > end`는 `[start, end]` 뒤에 `[left, right]`가 있는 경우다. 이 경우엔 겹치지 않으므로 탐색을 이어갈 필요 없다. 따라서 0을 리턴하고 종료한다.

`2번`의 경우엔 `if (left <= start && end <= right)`로 나타낼 수 있다. 이 경우에도 탐색을 이어갈 필요가 없다. 왜냐하면, 구해야하는 합의 범위는 `[left, right]`인데, `[start, end]`는 그 범위에 모두 포함되고, 그 node의 자식도 포함되기 때문에 더 이상 호출할 필요가 없다. 따라서 tree[node]를 리턴하고 종료한다.

`3번`과 `4번`의 경우는 왼쪽 자식과 오른쪽 자식을 루트로 하는 트리로 다시 탐색을 시작해야 한다.
```cpp
// node가 담당하는 구간이 start~end이고, 구해야하는 합의 범위는 left~right
long long sum(vector<long long> &tree, int node, int start, int end, int left, int right) {
    if (left > end || right < start) {
        return 0;
    }
    if (left <= start && end <= right) {
        return tree[node];
    }
    return sum(tree, node*2, start, (start+end)/2, left, right) + sum(tree, node*2+1, (start+end)/2+1, end, left, right);
}
```

### 수 변경하기
중간에 어떤 수를 변경한다면, 그 숫자가 포함된 구간을 담당하는 노드를 모두 변경해야한다.  
3번째 수를 변경할 때, 변경해야하는 구간을 봐보자.
![image](https://github.com/lee20h/blog/assets/59367782/eb1e62b1-7b6e-4c6b-beb3-db55c753cc53)
다음은 5를 변경할 때 변경해야하는 구간이다.  
![image](https://github.com/lee20h/blog/assets/59367782/9967fae6-9b4c-497b-bf73-7da83b5187e5)

index 번째 수를 val로 변경한다면, 그 수가 얼마만큼 변했는지 알아야한다. 해당 숫자를 `diff`라고 하면, `diff = val - a[index]`로 구할 수 있다.

수 변경은 2가지 경우가 있다.
1) `[start, end]`에 index가 포함된 경우
2) `[start, end]`에 index가 포함되지 않은 경우

node의 구간에 포함되는 경우에는 `diff`만큼 증가시켜 합을 변경해 줄수 있다. `tree[node] = tree[node] + diff` 포함되지 않은 경우는 그 자식도 index가 포함되지 않으므로 탐색을 중단해야 한다.
```cpp
void update(vector<long long> &tree, int node, int start, int end, int index, long long diff) {
    if (index < start || index > end) return;
    tree[node] = tree[node] + diff;
    if (start != end) {
        update(tree,node*2, start, (start+end)/2, index, diff);
        update(tree,node*2+1, (start+end)/2+1, end, index, diff);
    }
}
```
리프 노드가 아닌 경우에는 자식도 변경해줘야 하기 때문에, `start != end`로 리프 노드인지 검사 해줘야 한다.