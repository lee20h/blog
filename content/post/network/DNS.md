---
title: "DNS에 대해 알아보자"
date: 2020-09-05
tags:
  - DNS
categories:
  - network
published: true
---

# Domain Name System

DNS는 IP 주소 대신에 이름을 사용해서 IP주소를 매핑하는 시스템이다.

IP주소 대신에 이름을 사용하는 방법 3가지

- 방법
    1. 컴퓨터내에 host file 유지
        - 인터넷 전체를 IP와 이름을 각 PC가 하나의 파일에 저장하기는 너무 큼
        - 바뀐 IP주소와 이름간의 매핑을 유지하기 어려움

    2. 중앙 서버에 host file 유지
        - 서버를 하나를 가지고 모든 컴퓨터들이 이용하게 되면 병목현상이 일어난다.

    3. DNS 사용
        - 로드 분산
        - 계층 구조

## Name Space

### Flat Name Space
- 각 주소에 유일한 이름을 할당
- 이름은 문자의 연속이고, 구조적이지 않음
    - ex) 210.117.187.210 hello, 210.117.187.184 hi, ...
- 인터넷과 같이 큰 시스템에는 이름 중복을 방지하거나 애매성을 없애기 적절치 않음
    - 즉, 한마디로 이름 관리가 어렵고, 구현이 효율적이지 않음

### Hierarchical Name Space
- 각 이름이 여러 파트로 나뉘어짐
    - ex) (조직의 특성, 조직 이름, 부서 이름, ..)
- 이름을 할당하고 제어하는 주체가 분산될 수 있다
    - 위의 예시에서 조직의 특성은 A, 조직의 이름은 B가 담당하는 식으로 나눌 수 있다.
    - 즉 관리가 쉽고 구현이 효율적임

Hierarchical Name Space에 대한 예시로 여러 조직에서 같은 이름을 PC에 붙일려고 한다면 이러하게 된다.

- 대학1 : `cse.fhda.edu`
- 대학2 : `cse.berkeley.edu`
- 회사1 : `cse.smart.com`

이러한 방식으로 짓게 된다면 이름을 체계적으로 관리할 수 있다.

## Domain Name Space

Hierarchical Name Space를 위해서 Domain Name Space가 설계되었다. 즉, 도메인을 기반한 name space가 존재한다는 말이다.  

![image](https://user-images.githubusercontent.com/59367782/98500767-a8596d00-2290-11eb-8881-1609706359d5.png)

Domain의 포함 관계를 나타낸 예시로, 포함을 하지만 도메인끼리 교차하는 경우는 없다.

구현 방법은 Label을 갖는 역-트리 (inverted tree) 구조를 이용한다.

### 역 트리 구조

![image](https://user-images.githubusercontent.com/59367782/98500859-efdff900-2290-11eb-933c-3cb5a516c71e.png)

- 동일 레벨에서는 각 이름이 유일해야 한다.
- 각 노드는 63자 이하의 문자열로 구성된 label을 갖는다.

### Domain names and labels

도메인 이름은 `dot(.)`으로 구분된 label의 연속이며 항상 해당 노드에서 루트 방향으로 읽으면 된다.

`challenger.atc.fhda.edu`을 보게 된다면 역 트리 구조로 이루어져 있다.
```
Root
  |
Label: edu         Domain name: edu.
  |
Label: fhda        Domain name: fhda.edu.
  |
Label: atc         Domain name: atc.fhda.edu.
  |
Label: challenger  Domain name: challenger.atc.fhda.edu
```

### Qualified Domain Name

- Fully Qualified Domain Name (FQDN)
    - 호스트의 전체 이름을 포함하는 도메인 이름
    - ex) challenger.atc.fhda.edu.
        - challenger라는 이름을 갖는 컴퓨터의 FQDN
    - DNS server는 FQDN만을 IP 주소로 매핑할 수 있음

- Partially Qualified Domain Name (PQDN)
    - 호스트의 앞쪽 이름으로만 구성
    - Resolver가 나머지 suffix를 제공해서 FQDN을 생성
    - ex) challenger
        - DNS client가 suffix인 atc.fhda.edu를 붙여서 DNS server에 요청

```
FQDN                        PQDN
challnger.atc.fhda.edu.     challenger.atc.fhda.edu
cs.hmme.com.                cs.hmme
www.funny.int.              www
```

## Distribution of Name Spaces

Domain name space를 1대에 컴퓨터에 저장하면 병목현상이 발생하고 안정성이 떨어진다. 따라서 동일한 데이터를 여러 대에 분산시키거나 여러 서버가 각자의 의무를 분산시킨다.

### Hierarchy of name servers

```
                      Root server
arpa server     edu server       com server  us server
        fhda.edu bk.edu   mcgraw.com irwin.com 
```

### 서버의 종류

- 루트 서버(root server)
    - 자신의 zone이 전체 트리를 구성하는 서버
    - 대개 자신의 authority를 다른 서버에 양도 (즉, com 서버, edu 서버 등)하고 그 서버들에 대한 참조만 유지
    - 현재 13개 이상의 루트 서버가 전세계에 존재

- 주 서버(Primary server)
    - 자신이 authority를 갖는 znoe에 대한 파일을 저장하는 서버이며, zone 파일의 생성, 유지, 갱신을 담당

- 보조 서버(secondary server)
    - zone 파일을 주 서버로부터 받아서만갱신하고 자신이 생성하지는 않음

- 주의: 하나의 서버가 Zone A에 대해서 주 서버 역할을 하고, Zone B에 대해서 보조 서버 역할을 할 수도 있음

## DNS In The Internet

Root는 Inverse domain, Generic domains, Country domains로 나뉜다.

### Generic domains

Root level에 밑에 Generic behavior에 따른 host를 등록한다. 과거에는 7개의 섹션이 존재하는데 필요에 따라 늘어나고 있다.

- Generic domain labels
    - com: Commercial organizations
    - edu: Educational institutions
    - gov: Government institutions
    - int: Internationa organizations
    - mil: Military groups
    - net: Network support centers
    - org: Nonprofit organizations

### Country domains

Root level 밑에 2문자로 나라의 약어를 통해 나라별 도메인을 둔다.

### Inverse domain

IP 주소를 이름으로 매핑하기 위한 도메인으로 다른 domain과 달리 목적이 반대이다. 

## Resolution

클라이언트가 요청하면 서버가 자신이 authority를 가진 경우 자신의 DB를 검색하여 결과를 전달하고, 그렇지 않은 경우 다른 서버에게 요청을 한 후 응답을 전달한다.

![image](https://user-images.githubusercontent.com/59367782/98503337-9202df80-2297-11eb-94bf-d964fc12b9a0.png)

- Recursive resolution
    - authority를 갖는 DNS Server를 계층 구조에서 찾아 요청하고 응답을 전달
    - Recursive Call과 유사
    - 클라이언트는 1번의 요청하면 됨
    - DNS server의 load가 생김 : Scalability 이슈가 있음

![image](https://user-images.githubusercontent.com/59367782/98503313-84e5f080-2297-11eb-86ec-97276230b974.png)

- Iterative resolution
    - 클라이언트가 각 서버에 직접 요청하여 응답을 받는다.
    - DNS server에 load가 적음

### Resolver

- DNS client (or resolver)
    - 가장 가까운 DNS server에 접근해서 mapping request를 수행
        - 서버는 그 요구를 만족시킬 수 있으면 그 정보를 전달하고, 그렇지 않으면 다른 서버에게 그 정보를 요청한 후 정보를 전달
    
    - 요구 종류
        - Mapping names to addresses
            - 도메인 이름에 대한 IP 주소를 요구 받으면 Generic domain 혹은 country domain을 검사하여 돌려줌
            - ex) `nclab.jbnu.ac.kr`을 요구하면 `210.117.187.184`를 돌려줌
        - Mapping addresses to names
            - Inverse domain을 사용
            - ex) `210.117.187.184`을 요구하면 `nclab.jbnu.ac.kr`을 돌려줌

### Caching

첫 요청 시

![image](https://user-images.githubusercontent.com/59367782/98504390-3a19a800-229a-11eb-9ef9-c545fc69003b.png)

두번째 같은 쿼리 요청 시

![image](https://user-images.githubusercontent.com/59367782/98504423-4bfb4b00-229a-11eb-8bd3-ef8f7fe1eb4b.png)

- 일관성 유지 필요
    - Caching한 매핑이 과거의 것일 수 있음
        - DNS 서버는 caching한 것을 보고 응답을 할 때는 `unauthoritative`라고 마킹함
    
    - 다음의 두 기법이 사용됨
        1. Authoritative server가 매핑 결과를 돌려줄 때 TTL(Time-to-live)을 추가시킴
            - 만일 caching 된 것의 TTL이 expire되면 caching 된 것을 사용하지 않고 authoritative server에게 다시 물어봄
        2. DNS server가 주기적으로 cache 데이터 중 TTL이 expire 된 것을 제거함

Caching은 속도가 느린 것에 병목 현상이 있을 때 속도 향상 목적으로 사용된다.

- 폰노이만 아키텍쳐에서 모든 프로그램을 메모리로 가져온 뒤 cpu로 옮겨서 실행해야한다. 이 때 cpu와 메모리에 병목현상이 있게 되는데, cpu에 cache를 둬서 해결한다.

![image](https://user-images.githubusercontent.com/59367782/98504837-4f430680-229b-11eb-871d-e70e89863216.png)

- 웹 서버와 클라이언트 사이에 요청이 늘어나면 트래픽이 늘어나게 된다. 이 때 병목 현상이 일어나게 된다. 이 때 클라이언트가 웹 서버에 접근하기 전 프록시 서버에 먼저 접근하여 웹 서버에 접근하게 된다. 이렇게 되면 병목 현상을 해결할 수 있다.

![image](https://user-images.githubusercontent.com/59367782/98504749-1c007780-229b-11eb-8650-bae87a05db0e.png)

![image](https://user-images.githubusercontent.com/59367782/98504767-26bb0c80-229b-11eb-90d2-aaf2b7cdd7e3.png)

### Caching 특징

이러한 caching 기법이 효과를 발휘하기 위해서는 반드시 `locality of reference`를 가져야 함
- Temporal locality: 시간적 지역성
- Spatial locality: 공간적 지역성

위의 예시들은 이러한 지역성들을 만족한다.

### Caching Overhead

Caching 기법이 갖는 overhead는 `Data consistency`이다. 만약 중복된 데이터가 서로 다른 값을 갖게 되면 문제가 생긴다.

따라서 속도 향상을 생각할 때 Caching 기법 적용 가능 유무를 항상 머리속에 가지고 있으면 좋다. OS의 virtual memory에서 Paging 기술도 일종의 caching 기법으로 간주 가능하다.

프록시 서버를 요즘 안 쓰는 이유는 웹 서버와 프록시 서버의 데이터 차이가 생길 수 있는데 이러한 내용이 자주 일어나게 되어 Temporal locality가 떨어지게 되어 사용하기 힘들게 되었다.

Caching overhead가 크면 적용하지 않지만 만약 overhead가 작다면 충분히 도입하면 효율이 좋으므로 생각을 항상 해주는 것이 좋다.

## 그림 출처

- McGraw-Hill 출판사 TCP/IP 프로토콜