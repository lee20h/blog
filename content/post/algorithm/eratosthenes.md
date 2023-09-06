---
title: "에라토스테네스의 체에 대해 알아보자"
date: 2022-05-04T22:18:34+09:00
tags:
  - 에라토스테네스의 체
categories:
  - algorithm
published: true
---

## 에라토스테네스의 체란?

많은 수 중 소수를 빠르고 정확하게 판별하는 알고리즘이다.

### 원리
어떤 수의 소수 여부 확인에 있어서는 특정한 숫자의 제곱근까지만 약수 여부를 검증하면 시간 복잡도 O(N^1/2)으로 빠르게 구할 수 있다.

따라서 2부터 n 까지의 소수를 구한다고 가정한다면, 2부터 √n까지 돌면서 배수들을 제거하는 방식이다.

순차적으로 진행하면 다음과 같다.

```go
package main

import (
	"fmt"
	"math"
)

const n = 1000

func main() {
	var filter []bool
	filter = make([]bool, n+1)
	filter[1] = true

	for i := 2; i <= int(math.Sqrt(float64(n))); i++ {
		if filter[i] {
			continue
		}
		for j := i * 2; j < n; j += i {
			filter[j] = true
		}
	}

	for i := 1; i < n; i++ {
		if !filter[i] {
			fmt.Println(i)
		}
	}
}
```

