---
title: "트라이(trie)에 대해 알아보자"
date: 2022-03-02T22:17:06+09:00
tags:
  - trie
categories:
  - algorithm
publishResources: true
---

# 트라이

문자열에서 검색을 빠르게 해주는 자료구조이다. 정수형 자료형에서 이진색트리를 이용하면 O(logN), 문자열에서 이진검색트리를 사용하면 문자열의 최대 길이가 M 일때 O(MlogN) 시간복잡도를 갖는다.

하지만, 트라이를 이용한 문자열 검색에서는 O(M)의 시간 만에 원하는 문자열을 검색할 수 있다.

![image](https://github.com/lee20h/blog/assets/59367782/001aef20-a403-4916-9f7a-7becb0724436)

종료 노드를 표시한 뒤 트라이 모양을 그려준 그림이다.

트라이는 트리 형태이기 때문에 검색시에 최대 트리의 높이까지 탐색하게 된다. 따라서 시간 복잡도는 O(H)가 된다. 하지만, 트리의 높이는 최대 문자열의 길이가 되기 때문에 O(M)의 시간복잡도를 갖는다고 말할 수 있다.

## 구현 방법

```cpp
struct Trie {
    bool finish;    //끝나는 지점을 표시해줌
    Trie* next[26];    //26가지 알파벳에 대한 트라이
    Trie() : finish(false) {
        memset(next, 0, sizeof(next));
    }
    ~Trie() {
        for (int i = 0; i < 26; i++)
            if (next[i])
                delete next[i];
    }
    void insert(const char* key) {
        if (*key == '\0')
            finish = true;    //문자열이 끝나는 지점일 경우 표시
        else {
            int curr = *key - 'A';
            if (next[curr] == NULL)
                next[curr] = new Trie();    //탐색이 처음되는 지점일 경우 동적할당
            next[curr]->insert(key + 1);    //다음 문자 삽입
        }
    }
    Trie* find(const char* key) {
        if (*key == '\0')return this;//문자열이 끝나는 위치를 반환
        int curr = *key - 'A';
        if (next[curr] == NULL)return NULL;//찾는 값이 존재하지 않음
        return next[curr]->find(key + 1); //다음 문자를 탐색
    }
};
```

가장 보편적인 방법으로, 트라이는 자료구조이기 때문에 주어진 문제의 조건이나 사용해야하는 환경에 따라 변형하여 사용할 수 있다.

이 중 자주 변형하여 사용하는 것은 `find()` 함수로 여러 형식으로 바꾸어서 사용한다.

## 트라이의 장단점

트라이의 장점이라면 시간이다. 위에서 다룬 시간복잡도에서만 봐도 확실히 시간복잡도가 짧다. 한 노드의 하나의 알파벳을 대응시켜서 만드는 자료구조라면, map 즉, 해시 자료구조를 사용하면 되나, 속도가 확실히 느리다.

이어서 단점을 보게되면 위에서 말한 해시 자료구조와 비교하게 되면, 공간복잡도가 차이가 크다는 것을 알 수 있다. 공간복잡도는 **포인터 크기 * 포인터 배열 개수 * 트라이에 존재하는 총 노드의 개수**가 되므로 해시에 비해 많은 차이가 난다.

[그림 및 소스 출처](https://jason9319.tistory.com/129)