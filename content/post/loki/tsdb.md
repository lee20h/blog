---
title: "Grafana Loki는 왜 TSDB를 선택했을까?"
date: 2025-06-29T15:04:44+09:00
tags:
  - loki
categories:
  - loki
published: true
---

## TSDB는 어떤 방식으로 문제를 해결했을까?

Loki가 도입한 TSDB는 단순한 저장 방식의 변경이 아니라, 기존 `boltdb-shipper` 구조의 병목을 해결하기 위한 구조적 전환입니다. 특히 인덱스와 로그 청크를 시간 단위 블록 단위로 묶어 저장함으로써 성능과 비용 문제를 동시에 개선했습니다.

---

### 블록 기반 저장 구조 (Block-based layout)

TSDB는 Prometheus에서 영감을 받은 구조로, 일정 시간 단위(2시간)를 기준으로 하나의 **TSDB 블록(block)** 을 구성합니다. 이 블록은 인덱스와 로그 청크 데이터를 포함하며, S3나 GCS 같은 오브젝트 스토리지에 다음과 같이 저장됩니다.

```bash
index/
└── <tenant-id>/
    └── <block-id>/
        └── index-id             # 라벨 기반 역색인 파일

<tenant-id>/
└── <block-id>/
    └── <chunk-id>           # 해당 블록 시간대의 로그 청크 파일
```

> 🔹 `index/` 디렉토리는 역색인(index) 정보를 저장합니다.  
> 🔹 `tenant-id/` 디렉토리는 로그 청크(chunk)를 시간 블록별로 저장합니다.  
> 두 구조는 논리적으로는 하나의 블록 단위로 강하게 연결되어 있지만, 물리적으로는 S3에서 관리 편의성을 위해 디렉토리로 분리되어 저장됩니다.  
> 실제 블록 구조는 `chunks/`, `index`, `meta.json`, `tombstones` 등 더 많은 파일을 포함하지만, 여기서는 핵심 구조만 단순화하여 표현했습니다.

---

### 인덱스는 어떤 역할을 할까?

Loki의 쿼리 성능을 높이기 위한 핵심은 **역색인(Inverted Index)** 입니다.  
역색인은 `{app="nginx"}` 같은 라벨 쿼리를 빠르게 수행할 수 있도록, **라벨 → stream ID → chunk ID** 로 매핑된 정보를 미리 저장한 자료구조입니다.

Loki에서의 인덱싱은 라벨 기반 쿼리를 빠르게 수행하기 위해 역색인 형태의 색인 구조를 생성하는 과정입니다. 단순한 stream ID 매핑이 아닌, 라벨 값으로 청크를 직접 찾아갈 수 있도록 구성되어 있어, 전통적인 인덱스와는 다른 검색엔진 스타일의 인덱싱 방식을 채택했다고 볼 수 있습니다.



역색인 기반 쿼리 처리 흐름

1. 사용자 쿼리 → 예: `{app="nginx"}` over last 1h
2. Querier가 시간 범위에 해당하는 블록 목록을 로딩
3. 각 블록의 `index` 파일에서 stream ID 및 chunk ID를 찾음
4. 해당 청크 ID에 대응되는 로그 파일을 Object Storage에서 다운로드
5. 로그 추출 → 결과 반환

이처럼 Loki는 여전히 인덱스를 기반으로 청크를 찾지만, **블록 단위로 묶어 효율성을 극대화**했습니다.


---

### 기존 방식과 TSDB 방식 비교

| 항목 | 기존 (boltdb-shipper) | TSDB 방식 |
|------|------------------------|------------|
| 인덱스 저장 | BoltDB DB 파일 (`.db`) | TSDB-style index (`index/index-id`) |
| 청크 저장 | 개별 파일 | 블록 경로와 매핑 |
| 쿼리 속도 | 파일 수 증가로 점점 저하 | 블록 병렬 스캔으로 향상 |
| 운영 난이도 | DB 파일 증가, compactor 복잡 | 블록 기준 관리로 간소화 |
| 비용 | 인덱스 파일 많고 커짐 | 인덱스 크기 최대 75% 감소 |

---

### 시간 범위 최적화

TSDB 구조에서 각 청크(chunk)에는 `minTime`, `maxTime`이라는 메타 정보가 포함되어 있습니다. 이 정보를 통해 Loki는 쿼리 시 아래와 같은 방식으로 동작합니다:

1. 쿼리 요청이 들어오면, 해당 시간 범위에 포함되는 블록들만 선별합니다.
2. 각 블록의 인덱스를 스캔하여 라벨 조건에 맞는 스트림(stream)을 조회합니다.
3. 그 스트림에 매핑된 청크들 중에서, 쿼리 시간 범위와 겹치는 청크만 가져옵니다.

이 방식 덕분에 Loki는 **불필요한 청크를 다운로드하지 않으며**,  
필요한 청크만 병렬로 스캔할 수 있어 **I/O 효율이 매우 높아집니다**.

> 이 최적화 방식은 쿼리 성능의 핵심이며, 실제로 Loki Cloud 환경에서는  
> BoltDB 대비 4배 더 빠른 쿼리 성능을 실현했다고 Grafana는 밝혔습니다.

---

### 성능 특성 비교

TSDB 도입 이후의 성능 향상은 여러 지표로 확인할 수 있습니다.

| 항목 | BoltDB-Shipper | TSDB |
|------|------------------|------|
| 인덱스 스캔 속도 | 느림 (전체 `.db` 순차 조회) | 블록 단위 병렬 쿼리 |
| 쿼리 CPU 사용량 | 높음 | 상당한 절감 (병렬 처리 최적화) |
| 읽기 병목 | 존재 (파일 수 증가) | 거의 없음 |
| 평균 쿼리 응답 시간 | 수 초 이상 | 수 밀리초 ~ 수 초 |

- 인덱스 구조가 단순화되면서 **쿼리 속도는 빨라지고**,  
- 병렬 처리 기반 구조로 **CPU 사용량이 효율적으로 관리**됩니다.
- 특히 병렬 청크 스캔 성능은 **규모가 커질수록 압도적인 차이**를 보입니다.

---

### 메모리 사용량과 캐싱 전략

TSDB는 성능을 최적화하기 위해 **메타데이터 prefetch**와 **로컬 디스크 캐시**를 적극 활용합니다.

- **Querier, Compactor**는 쿼리 시 필요한 블록의 메타정보를 먼저 가져와 미리 필터링합니다.
- 이 과정은 메모리 캐시 또는 로컬 디스크 캐시를 이용해 반복 접근 비용을 줄입니다.

```yaml
querier:
  cache_config:
    memcached:
      max_cache_size: 500MB
    index_cache_validity: 24h
```

- 일반적으로 **쿼리 수행 시점 외에는 메모리 사용량이 낮은 편**이며,
- TSDB 구조 특성상 **쓰기(write) 시점과 읽기(read) 시점을 명확히 분리**할 수 있어 리소스 관리가 용이합니다.


### 요약

- TSDB는 인덱스와 청크를 **시간 기반 블록 단위로 논리적으로 묶어서 저장**합니다.
- 쿼리는 인덱스를 먼저 조회하고 → 청크를 찾아 로드하는 **전통적인 역색인 기반 흐름을 유지**합니다.
- 단지 저장 방식이 개선되어, **성능 향상 + 저장 효율 + 확장성 확보**가 동시에 이루어졌습니다.

### Ref

- [Loki 2.7 Release Note](https://grafana.com/blog/2022/12/01/grafana-loki-2.7-release)
- [Loki 2.8 Release Note](https://grafana.com/blog/2023/04/06/grafana-loki-2.8-release-tsdb-ga-logql-enhancements-and-a-third-target-for-scalable-mode)
- [Loki TSDB](https://grafana.com/docs/loki/latest/operations/storage/tsdb/)
- [Loki migrate to tsdb](https://grafana.com/docs/loki/latest/setup/migrate/migrate-to-tsdb/)