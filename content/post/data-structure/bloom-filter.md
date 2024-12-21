---
title: "Bloom Filter에 대해 알아보자"
date: 2024-12-21T12:44:46+09:00
tags:
  - probabilistic-data-structure
  - bloom-filter
categories:
  - data-structure
  - bloom-filter
published: true
---

# Bloom Filter란?
Bloom Filter는 요소가 집합에 포함되어 있는지를 효율적으로 검사하기 위한 확률적 자료 구조예요. 이 자료 구조는 메모리 사용을 최소화하면서도 빠른 검색 속도를 제공해요. 

Bloom Filter는 긍정 오류(false positive)을 허용하지만, 부정 오류(false negative)은 일어나지 않는 특징이 있어요. 즉, 요소가 집합에 포함되어 있지 않더라도 포함되어 있다고 잘못 판단할 수 있지만, 요소가 포함되어 있다고 잘못 판단하는 경우는 없어요.

# 특징
- 효율적인 공간 사용
  - 고정 크기의 비트 배열과 여러 해시 함수를 사용해 존재 여부를 나타내므로 메모리 제약이 있는 상황에 적합해요.
  - 공간복잡도 `O(m)`으로, 여기서 `m`은 비트 배열 크기예요.
- 빠른 연산
  - 삽입 및 쿼리의 연산의 시간복잡도 `O(k)`로, 여기서 `k`는 해시 함수의 개수예요.
- 확률적 성격
  - 긍정 오류(false positive): 가능해요
  - 부정 오류(false negative): 불가능해요
- 삽입 전용
  - 요소를 직접 제거하는 경우 다른 요소와 매핑한 비트가 겹칠 수 있어서 부정 오류이 발생할 위험이 있어요.
  - 제거된 요소를 추적하기 위해 두 번째 Bloom Filter를 유지할 수 있지만, 이 경우에도 부정 오류이 발생할 수 있어요.
  - Counting Bloom Filter를 이용하면 제거가 가능해요. 이 땐 각 비트가 카운터로 대체되어요.
- 성능 저하 요인
  - 요소가 많으면 많을수록 긍정 오류률이 증가해요.
  - 긍정 오류률이 너무 높아지면 필터를 재생성해야 할 수도 있어요.

[위키피디아](https://en.wikipedia.org/wiki/Bloom_filter)에서는 다음과 같은 사례를 이야기하고 있어요.


> Bloom Filter는 정확한 구성원 검사가 필요한 대량의 메모리가 요구되는 상황을 위해 설계되었어요. 
> - 500,000개의 단어로 구성된 하이픈 사전이 있을 때, 이 중 90%는 간단한 규칙을 따르고 나머지 10%는 비싼 디스크 접근이 필요한 경우
>   - 제한된 메모리 상황에서 Bloom Filter는 다음과 같은 효과를 가져올 수 있어요.
>     - 완벽한 오류 없는 해시가 요구하는 메모리의 18%만 사용하면서, 디스크 접근을 87% 절약할 수 있어요.
>
> 일반적으로, 집합 크기나 요소 수에 관계없이 1%의 긍정 오류률을 유지하기 위해서는 요소당 10비트 이하의 메모리가 필요해요.


# 세부 사항

## 1. 데이터 구조
- 크기 `m`의 비트 배열, 모든 값이 0으로 초기화
- `k`개의 독립적인 해시 함수, 각 해시 함수는 입력을 비트 배열의 `m` 위치 중 하나로 매핑

## 2. 연산
- **삽입**:
  - 각 요소를 `k`개의 해시 함수 통과
  - 비트 배열의 해당 `k` 위치를 1로 설정
- **쿼리**:
  - 동일한 `k`개의 해시 함수를 통과
  - 비트 배열의 해당 `k` 위치를 확인
    - 만약 어떤 위치가 0이라면 → 없음을 보장
    - 모든 위치가 1이라면 → 집합에 있을 가능성이 있음 (긍정 오류)

## 3. 긍정 오류 확률
- 긍정 오류의 확률을 결정하는 요소
  - 필터에 추가된 요소 수가 많을수록
  - 비트 배열의 크기 `m`이 충분하지 않을 때
  - 해시 함수 또는 밑에서 후술할 해시 함수의 개수가 적당하지 않을 때

### 긍정 오류 확률 계산

비트 배열의 모든 비트는 처음에 0으로 설정되어 있어요. 요소 `n`개를 추가한 후, 각 비트가 1로 설정될 확률을 계산해보면 다음과 같아요.

[1]<figure style="display: inline-block;"><img src="https://github.com/user-attachments/assets/29d8143a-3f4e-4ae8-bd1e-3e00423bdaa8" align="left"/></figure>

[2]<figure style="display: inline-block;"><img src="https://github.com/user-attachments/assets/43ff2d21-413b-4407-a828-c4ca84c5fb64" align="left"/></figure>

[3]<figure style="display: inline-block;"><img src="https://github.com/user-attachments/assets/b1690714-28c6-49c9-a8e6-11fda7721128" align="left"/></figure>

각 요소가 비트를 1로 설정할 확률은 `1/m`으로, n개의 요소가 추가 된 후 특정 비트가 0인 확률은 [1]와 같아요.  
특정 요소가 Bloom Filter에 없지만 모든 해시 함수가 1로 설정된 비트를 가르킬 확률은 [2]와 같아요. 이를 정리하면 [3]이 나오게 되어요.

## 4. 해시 함수
- `k`개의 독립적인 해시 함수가 이상적이지만 계산 비용이 높음
  - <figure style="display: inline-block;"><img src="https://github.com/user-attachments/assets/6e008215-9cd7-49bb-87f2-fa289500d6ba" align="left"/></figure> [3] 기반으로 k를 구하는 방법 (m: 비트 배열의 크기, n: 삽입할 요소 수)  
  - <figure style="display: inline-block;"><img src="https://github.com/user-attachments/assets/527f7fbb-ebb6-420f-ac3e-47032be27963" align="left"/></figure> 위의 공식에서 p를 일반적인 1/2로 설정하면 사진과 같은 공식이 완성됨 
  - `k`가 너무 적으면 긍정 오류율 증가, 너무 많으면 비트 배열의 사용 효율 떨어짐
  - 해시가 균일 분포될 수록 거짓긍정률이 낮아짐
- 일반적인 최적화 방법
  - 단일 해시 함수를 사용하고 향상된 더블 해싱 또는 트리플 해싱과 같은 기술로 `k`개의 인덱스 유도
  - 약간의 독립성을 완화하여 성능을 개선하되 긍정 오류률에 미치는 영향은 미미하게 유지

삽입과 쿼리 과정을 간단하게 그림으로 그려보았어요. `Found`의 경우에는 오탐이 가능해요.
<figure>
  <img src="https://github.com/user-attachments/assets/88ced0d1-8de7-47e4-8f91-76b4c43c6387" width="800" height="600"/>
</figure>

## 구현

위에서 학습한 내용 기반으로 가벼운 Bloom Filter를 Golang으로 구현해봤어요.
- [repository](https://github.com/lee20h/bloomfilter-practice)


## 참고

- Yifeng Zhu and Hong Jiang, False Rate Analysis of Bloom Filter Replicas in Distributed Systems, In Proceeding of the 2006 International Conference on Parallel Processing (ICPP'06)
