---
title: "Mockery에 대하여"
date: 2023-11-10T19:59:37+09:00
tags:
  - golang
  - test
  - mock
  - mockery
  - suite
categories:
  - golang
published: true
---

# Mockery란?

Go 언어에서 인터페이스를 모킹하기 위한 도구로, 테스트 중에 인터페이스의 구현을 쉽게 대체할 수 있어서 테스트를 더 단순하고 효율적으로 만들 수 있다.  
개발자가 수동으로 모킹 코드를 작성할 필요 없이 만들어져있는 인터페이스 기반으로 명령어로 모킹 클래스를 생성할 수 있어서 생산성이 높아진다. 모킹을 통해 외부 시스템이나 데이터베이스, 네트워크 호출과 같은 의존성을 제하고 테스트를 실행할 수 있다.  

이러한 부분이 다음과 같은 특징을 갖는다.

- 테스트의 격리성과 실행 속도 개선
- 테스트의 예측 가능성 향상
- 인터페이스만 사용 가능
- 실제 구현의 정확성 보장 X

## 사용

```shell
$ go get github.com/vektra/mockery/v2
```

- 예제

인터페이스 정의

```go
package mypackage

type MyInterface interface {
    MyMethod(param1 string, param2 int) error
}
```

Mock 생성

```shell
$ mockery --name=MyInterface
```

테스트 작성

```go
package mypackage_test

import (
    "mypackage"
    "mypackage/mocks"
    "testing"

    "github.com/stretchr/testify/assert"
)

func TestSomething(t *testing.T) {
    mock := &mocks.MyInterface{}
    mock.On("MyMethod", "arg1", 123).Return(nil)

    // 테스트 로직...
    err := someFunctionThatUsesMyInterface(mock)
    assert.NoError(t, err)

    mock.AssertExpectations(t)
}
```

## 설정

추천하는 기본 설정은 다음과 같다.

```yaml
with-expecter: true
packages:
    github.com/your-org/your-go-project:
        # place your package-specific config here
        config:
        interfaces:
            # select the interfaces you want mocked
            Foo:
                # Modify package-level config for this specific interface (if applicable)
                config:
```

### 매개변수 설명

|          name          | templated | default                      | description                                                                                                                                           |
|:----------------------:|:---------:|------------------------------|-------------------------------------------------------------------------------------------------------------------------------------------------------|
|          all           |     X     | false                        | 지정된 패키지의 모든 인터페이스에 대해 모킹 생성합니다.                                                                                                                       |
|    boilerplate-file    |     X     | ""                           | 모든 생성된 모킹 파일 상단에 표시하고 싶은 주석이 포함된 파일의 경로를 지정합니다. 이는 일반적으로 소스 코드 상단에 라이선스 헤더를 표시하는 데 사용됩니다.                                                             |
|         config         |     X     | ""                           | mockery 구성 파일의 위치를 설정합니다.                                                                                                                             |
|          dir           |     O     | "mocks/{{.PackagePath}}"     | 모킹 파일이 출력될 디렉토리입니다.                                                                                                                                   |
| disable-config-search  |     X     | false                        | 구성 파일 검색을 비활성화합니다.                                                                                                                                    |
| disable-version-string |     X     | false                        | 생성된 모킹 파일에서 버전 문자열을 비활성화합니다.                                                                                                                          |
|        dry-run         |     X     | false                        | 수행될 작업을 출력하지만 실제로 작업을 수행하지 않습니다.                                                                                                                      |
|        exclude         |     X     | []                           | recursive: True를 사용할 때 제외할 하위 패키지를 지정합니다.                                                                                                             |
|     exclude-regex      |     X     | ""                           | include-regex와 함께 설정되면, include-regex와 일치하지만 exclude-regex와도 일치하는 인터페이스는 생성되지 않습니다. all이 설정되어 있거나 include-regex가 설정되지 않은 경우, exclude-regex는 효과가 없습니다. |
|        filename        |     O     | "mock_{{.InterfaceName}}.go" | 모킹가 위치할 파일의 이름입니다.                                                                                                                                    |
| include-auto-generated |     X     | true                         | 재귀적 패키지 탐색 중에 자동 생성된 파일을 건너뛰려면 false로 설정합니다. true로 설정하면, mockery는 특정 디렉토리가 가져올 수 있는 패키지인지 결정할 때 자동 생성된 파일을 포함합니다.                                     |
|     include-regex      |     X     | ""                           | 설정되면, 표현식과 일치하는 인터페이스 이름만 생성됩니다. 이 설정은 구성에서 all: True가 지정된 경우 무시됩니다. 생성되는 인터페이스를 더욱 세밀하게 조정하려면 exclude-regex를 사용하세요.                                  |
|       inpackage        |     X     | false                        | 원본 인터페이스와 같은 패키지에 모킹를 생성할 때, inpackage: True를 지정하여 모킹가 원본 인터페이스와 같은 패키지에 배치됨을 mockery에 알려야 합니다.                                                       |
|        mockname        |     O     | "Mock{{.InterfaceName}}"     | 생성될 모킹의 이름입니다.                                                                                                                                        |
|         outpkg         |     O     | "{{.PackageName}}"           | 생성된 모킹의 패키지 이름을 지정하려면 outpkg를 사용하세요.                                                                                                                  |
|       log-level        |     X     | "info"                       | 로거의 수준을 설정합니다.                                                                                                                                        |
|        packages        |     X     | null                         | 모킹를 생성할 패키지와 인터페이스에 대한 구성을 설명하는 사전입니다.                                                                                                                |
|         print          |     X     | false                        | 결과 코드를 디스크에 쓰는 대신 출력하려면 print: True를 사용하세요.                                                                                                           |
|       recursive        |     X     | false                        | 특정 패키지에 대해 true로 설정하면, mockery는 모든 하위 패키지를 재귀적으로 검색하고 해당 패키지를 구성 맵에 주입합니다.                                                                            |
|          tags          |     X     | ""                           | 생성된 모킹의 빌드 태그를 설정합니다.                                                                                                                                 |
|     with-expecter      |     X     | true                         | 모킹에 EXPECT() 메소드를 생성하려면 with-expecter: True를 사용하세요. 이는 모킹 설정을 위한 선호되는 방법입니다.                                                                          |
|      replace-type      |     X     | null                         | 생성 중에 별칭, 패키지 및/또는 유형을 교체합니다.                                                                                                                         |

### 템플릿 함수

Go 패키지 [text/template](https://pkg.go.dev/text/template)를 기반으로 설정파일 내 `package`에서 사용할 수 있다.

| Function    | Argument(s)         | Description                    |
|-------------|---------------------|--------------------------------|
| contains    | string, substr      | 문자열 내에 부분 문자열이 포함되어 있는지 확인합니다. |
| hasPrefix   | string, prefix      | 문자열이 특정 접두사로 시작하는지 확인합니다.      |
| hasSuffix   | string, suffix      | 문자열이 특정 접미사로 끝나는지 확인합니다.       |
| join        | elems, sep          | 지정된 구분자로 요소들을 결합합니다.           |
| replace     | string, old, new, n | 문자열 내의 'old'를 'new'로 n번 교체합니다. |
| replaceAll  | string, old, new    | 문자열 내의 'old'를 'new'로 모두 교체합니다. |
| split       | string, sep         | 문자열을 구분자에 따라 분할합니다.            |
| splitAfter  | string, sep         | 구분자 이후로 문자열을 분할합니다.            |
| splitAfterN | string, sep, n      | 구분자 이후로 문자열을 n개로 분할합니다.        |
| trim        | string, cutset      | 문자열 양쪽에서 지정된 문자 집합을 제거합니다.     |
| trimLeft    | string, cutset      | 문자열 왼쪽에서 지정된 문자 집합을 제거합니다.     |
| trimPrefix  | string, prefix      | 문자열에서 지정된 접두사를 제거합니다.          |
| trimRight   | string, cutset      | 문자열 오른쪽에서 지정된 문자 집합을 제거합니다.    |
| trimSpace   | string              | 문자열 양쪽에서 공백을 제거합니다.            |
| trimSuffix  | string, suffix      | 문자열에서 지정된 접미사를 제거합니다.          |
| lower       | string              | 문자열을 소문자로 변환합니다.               |
| upper       | string              | 문자열을 대문자로 변환합니다.               |
| camelcase   | string              | 문자열을 카멜케이스로 변환합니다.             |
| snakecase   | string              | 문자열을 스네이크케이스로 변환합니다.           |
| kebabcase   | string              | 문자열을 케밥케이스로 변환합니다.             |
| firstLower  | string              | 문자열의 첫 글자를 소문자로 변환합니다.         |
| firstUpper  | string              | 문자열의 첫 글자를 대문자로 변환합니다.         |
| matchString | pattern             | 주어진 패턴과 문자열이 일치하는지 확인합니다.      |
| quoteMeta   | string              | 문자열에서 정규 표현식 메타문자를 이스케이프합니다.   |
| base        | string              | 파일 경로에서 마지막 요소를 반환합니다.         |
| clean       | string              | 파일 경로를 정리합니다.                  |
| dir         | string              | 파일 경로에서 디렉토리 부분을 반환합니다.        |
| expandEnv   | string              | 문자열 내의 환경 변수를 확장합니다.           |
| getenv      | string              | 환경 변수의 값을 반환합니다.               |

### 템플릿 변수

| Variable             | Description                                                                                                                                                     |
|----------------------|-----------------------------------------------------------------------------------------------------------------------------------------------------------------|
| InterfaceDir         | 모킹 대상이 되는 원본 인터페이스의 디렉토리 경로입니다. `dir: "{{.InterfaceDir}}"`을 사용하여 원본 인터페이스와 인접한 위치에 모킹를 배치할 수 있습니다. 외부 인터페이스에는 사용하지 않는 것이 좋습니다.                                  |
| InterfaceDirRelative | 현재 작업 디렉토리에 상대적인 원본 인터페이스의 디렉토리 경로입니다. 현재 작업 디렉토리에 상대적으로 경로를 설정할 수 없는 경우, 이 변수는 PackagePath와 동일하게 설정됩니다.                                                        |
| InterfaceName        | 모킹 대상이 되는 원본 인터페이스의 이름입니다.                                                                                                                                      |
| Mock                 | 인터페이스가 내보내진 경우 'Mock', 내보내지 않은 경우 'mock'인 문자열입니다. 모킹 이름을 다음과 같이 설정할 때 유용합니다: `mockname: "{{.Mock}}{{.InterfaceName}}"`. 이렇게 하면 모킹 이름이 원본 인터페이스의 내보내기 여부를 유지합니다. |
| MockName             | 생성될 모킹의 이름입니다. 이는 단순히 mockname 구성 변수입니다.                                                                                                                        |
| PackageName          | 원본 인터페이스의 패키지 이름입니다.                                                                                                                                            |
| PackagePath          | 원본 인터페이스의 완전한 패키지 경로입니다.                                                                                                                                        |
