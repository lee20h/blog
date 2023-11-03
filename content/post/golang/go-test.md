---
title: "Go Test에 대해 알아보자"
date: 2023-10-30T22:55:12+09:00
tags:
  - golang
  - test
  - testify
  - mock
  - suite
categories:
  - golang
published: true
---

# Golang Test

Go에서는 **go test**라는 내장 테스트 도구를 제공한다. 이 도구로 Go 코드의 단위 테스트, 벤치마크, 예제 등을 쉽게 작성하고 실행할 수 있다.  
그 외에도 여러 테스트 프레임워크가 존재하며, mock을 위한 라이브러리도 많이 존재한다. 이번 포스팅에서는 Go에서의 테스트를 크게 훑어보고, 사용법과 장단점을 비교해보려고 한다.

## Go test

Go에서 제공하는 테스트는 단순하게 작성이 가능하다. 다음의 규칙만 지킨다면, 바로 정의 후 사용할 수 있다.

- `_test.go`로 끝나는 파일로 작성
- `Test`로 시작하는 함수로 정의
- `Benchmark`로 시작하는 함수로 성능 벤치마크 작성

### 사용법

위의 규칙에 맞게 작성하였다면, 명령어만 사용한다면 바로 테스트를 실행하고 확인해볼 수 있다.

```shell
$ go test
```

`go test`는 현재 디렉토리에 있는 모든 테스트를 실행하는 명령으로 바로 결과를 받아 볼 수 있다. 이 명령어에 옵션으로 여러가지 기능들을 추가할 수 있다.

```shell
$ go test -bench .
$ go test -cover
$ go test -run TestFunctionName
```

위에서부터 벤치마크 기능, 테스트 커버리지 확인, 특정 테스트를 실행하는 옵션이다. 특정 테스트의 경우 정규 표현식을 사용해서 원하는 테스트만 실행할 수도 있다.  

### 특징

- 간결하고 일관된 문법
- 테스트 자동 검출
- 테이블 주도 테스트 지원
- 내장 벤치마크 도구
- 테스트 커버리지 지원
- 표준 라이브러리 일부

따라오는 단점으로는 다음과 같다.

- 고급 테스트 기능 부족
  - mock 객체 생성, 테스트 스위트 관리 기능 부재
- 병렬 테스트 제어
  - 기본으로 병렬 실행이지만, 세밀하게 제어 시에 복잡도 높음
- 테스트 리포트 포맷
  - 리포트 서드파티와 통합을 위해서 작업이 필요
- 외부 의존성 관리

Goland와 VScode와 같은 IDE에서 확장 기능을 설치하면 원하는 Test 코드들을 작성하도록 템플릿을 제공하거나 테스트를 실행할 수 있다.

### 예제

- 기본 예제

**main.go**
```go
package main

import "fmt"

func Add(a, b int) int {
    return a + b
}

func main() {
    fmt.Println(Add(2, 3))
}
```
**main_test.go**
```go
package main

import "testing"

func TestAdd(t *testing.T) {
  result := Add(2, 3)
  if result != 5 {
    t.Errorf("Expected 5, but got %d", result)
  }
}
```
- 테이블 기반 테스트 예제

**main.go**
```go
package main

import "fmt"

func Add(a, b int) int {
    return a + b
}

func main() {
    fmt.Println(Add(2, 3))
}
```
**main_test.go**
```go
package main

import "testing"

func TestAdd(t *testing.T) {
    // 테스트 케이스를 정의하는 테이블
    tests := []struct {
        name string // 테스트 케이스의 이름
        a, b int    // 입력값
        want int    // 기대하는 결과값
    }{
        {"Add 1+2", 1, 2, 3},
        {"Add 0+0", 0, 0, 0},
        {"Add -1+2", -1, 2, 1},
        {"Add 2+-2", 2, -2, 0},
    }

    // 테이블 기반 테스트를 실행
    for _, tt := range tests {
        t.Run(tt.name, func(t *testing.T) {
            if got := Add(tt.a, tt.b); got != tt.want {
                t.Errorf("Add(%d, %d) = %d, want %d", tt.a, tt.b, got, tt.want)
            }
        })
    }
}
```

```shell
$ go test
```

명령을 통해서 바로 해당 결과를 얻어볼 수 있다. 

![image](https://github.com/lee20h/blog/assets/59367782/8e5e18e0-d465-49ea-8bcd-d71fb6976418)

## testify

Go에서 자주 사용하는 테스팅 프레임워크로 `testify`가 있다. Go 기본 테스팅 패키지를 확장하여 다양한 기능을 제공하고 있다.

### 특징

- Assertion: assertion 함수를 제공하여 테스트 코드를 간결하고 명확하게 작성
- Mocking: `testify/mock` 패키지로 인터페이스이 mock 구현을 쉽게 생성 관리
- Suites: `testify/suite` 패캐지로 테스트 스위트 구성하여 관련된 테스트 케이스들을 그룹화
- 읽기 쉬운 에러 메시지


단점으로는 다음과 같은 부분들을 꼽는다.

- 러닝 커브: 기본 테스팅 패키지에 익숙하다면, `testify`의 추가 기능들을 익히는데 시간이 필요할 수 있다.
- 의존성: 당연한 말이지만, 외부 라이브러리르 써야해서 의존성이 증가한다.
- 순수성: Go 테스팅 철학은 간결하고 최소한의 기능만을 제공하는 반면에, `testify`는 다양한 기능을 제공한다.

### 예제

### assert

```shell
$ go get github.com/stretchr/testify/assert
```

#### main.go

```go
package main

import "fmt"

func Add(a, b int) int {
    return a + b
}

func main() {
    fmt.Println(Add(2, 3))
}
```

#### main_test.go

```go
package main

import (
    "testing"
    "github.com/stretchr/testify/assert"
)

func TestAdd(t *testing.T) {
    assert := assert.New(t)

    // 테스트 케이스
    assert.Equal(3, Add(1, 2), "Add(1, 2) should be 3")
    assert.Equal(0, Add(0, 0), "Add(0, 0) should be 0")
    assert.Equal(1, Add(-1, 2), "Add(-1, 2) should be 1")
    assert.Equal(0, Add(2, -2), "Add(2, -2) should be 0")
}
```

### mock

```shell
$ go get github.com/stretchr/testify/mock
```

#### main.go

```go
package main

import "fmt"

type Messenger interface {
    SendMessage(to, message string) error
}

func Notify(messenger Messenger, to, message string) error {
    return messenger.SendMessage(to, message)
}

func main() {
    // 실제 구현
    fmt.Println("Notify function")
}
```

#### main_test.go

```go
package main

import (
    "testing"
    "github.com/stretchr/testify/mock"
)

type MockMessenger struct {
    mock.Mock
}

func (m *MockMessenger) SendMessage(to, message string) error {
    args := m.Called(to, message)
    return args.Error(0)
}

func TestNotify(t *testing.T) {
    mockMessenger := new(MockMessenger)

    mockMessenger.On("SendMessage", "receiver", "Hello!").Return(nil)

    err := Notify(mockMessenger, "receiver", "Hello!")

    mockMessenger.AssertExpectations(t)
    mockMessenger.AssertNoUnexpectedCalls()
    assert.Nil(t, err)
}
```

### suite

```shell
$ go get github.com/stretchr/testify/suite
```

#### main.go

```go
// main.go
package main

func Add(a, b int) int {
    return a + b
}

func Subtract(a, b int) int {
    return a - b
}
```

#### main_test.go

```go
package main

import (
    "testing"
    "github.com/stretchr/testify/suite"
)

type CalculatorSuite struct {
    suite.Suite
}

func (suite *CalculatorSuite) TestAdd() {
    suite.Equal(3, Add(1, 2))
    suite.Equal(0, Add(-1, 1))
}

func (suite *CalculatorSuite) TestSubtract() {
    suite.Equal(1, Subtract(3, 2))
    suite.Equal(-2, Subtract(1, 3))
}

func TestCalculatorSuite(t *testing.T) {
    suite.Run(t, new(CalculatorSuite))
}
```

## mockery

인터페이스를 모킹하기 위해서 사용하는 라이브러리로, `testify/mock`과 같이 자주 사용된다.

### 예제

#### myservice.go

```go
package mypackage

type MyService interface {
    DoSomething(input string) (string, error)
}
```

```shell
$ go get github.com/vektra/mockery/v2/.../
$ mockery --name=MyService
```

#### mocks/MockMyService.go

```go
package mocks

import mock "github.com/stretchr/testify/mock"
import mypackage "mypackage"
import string "string"

// MockMyService is an autogenerated mock type for the MyService type
type MockMyService struct {
    mock.Mock
}

// DoSomething provides a mock function with given fields: input
func (_m *MockMyService) DoSomething(input string) (string, error) {
    ret := _m.Called(input)

    var r0 string
    if rf, ok := ret.Get(0).(func(string) string); ok {
        r0 = rf(input)
    } else {
        r0 = ret.Get(0).(string)
    }

    var r1 error
    if rf, ok := ret.Get(1).(func(string) error); ok {
        r1 = rf(input)
    } else {
        r1 = ret.Error(1)
    }

    return r0, r1
}
```

#### myservice_test.go

```go
package mypackage

import (
    "testing"
    "github.com/stretchr/testify/assert"
    "github.com/stretchr/testify/mock"
    "mypackage/mocks"
)

func TestDoSomething(t *testing.T) {
    // Mock 객체 생성
    mockService := new(mocks.MockMyService)

    // 메서드 호출 시 반환값 설정
    mockService.On("DoSomething", "input").Return("output", nil)

    // 실제 테스트 대상 함수 호출
    result, err := mockService.DoSomething("input")

    // 반환값 검증
    assert.NoError(t, err)
    assert.Equal(t, "output", result)

    // 모킹된 메서드가 예상대로 호출되었는지 확인
    mockService.AssertExpectations(t)
}
```

