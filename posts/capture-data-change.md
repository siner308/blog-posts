---
title: "CDC 알아보기"
subtitle: "무중단 데이터베이스 마이그레이션 방법론"
tags:
    - database
date: 2023-12-03
image: https://github.com/siner308/blog-posts/assets/34048253/1d8f48de-5cc5-4b7f-ab05-042467939849
---

이번 글에서는 API 하위호환 지원에 대한 내용은 다루지 않습니다.

# 서론

> 구글과 같이 무중단 마이그레이션을 잘 수행하는 능력을 가진 개발자가 되고싶다.

항상 위와 같은 생각을 하고있었고, 이번기회에 데이터베이스 마이그레이션에 대해 조금 더 깊게 공부하고자 글을 작성하게 되었습니다.

## 단순 마이그레이션의 한계

기존에는 사용하던 데이터베이스에 마이그레이션 쿼리를 적용했고, 아래의 순서를 따릅니다.

1. 서버가 시작되면서 마이그레이션 파일들을 스캔하고, 어디까지 db에 반영되었는지 migration history 테이블을 조회하여 체크한다.
2. 아직 db에 반영되지 않은 마이그레이션 쿼리를 반영한다.
3. 마이그레이션이 제대로 반영되었다면, migration history 테이블에 기록한다.
4. 서버가 실행된다.

위의 과정은 다음과 같은 문제를 발생시킬 수 있습니다.

1. 다음 버전의 서버가 천천히 하나씩 뜨지 않고 한번에 뜨게 되는 경우, 두개 이상의 서버가 **하나의 마이그레이션을 동시에 실행시켜서 문제**를 일으킬 수 있습니다.
2. 적용할 쿼리의 영향 범위와 트랜잭션 격리 레벨에 따라 서비스에 **다운타임이 발생**할 수 있습니다.
3. 롤백을 할 수 없는 **breaking change 쿼리가 있다면** (one way door), **문제가 발생했을 경우 snapshot을 기준으로 롤백**을 해야할 수 있는데, 이는 2번의 케이스보다 더 긴 다운타임이 발생할 수 있습니다.

게임과 같이 모든 사용자가 동시에 같은 버전을 사용해야하는 케이스가 아니라면, 어느 회사나 무중단으로 서비스를 운영하여 이익을 극대화 하고 싶을 것입니다.

# CDC (Change Data Capture)

무중단 마이그레이션이라는 키워드로 검색을 해보면, CDC라는 솔루션이 있는 것을 확인할 수 있습니다.

CDC(Change Data Capture)는 DB의 변경사항을 트래킹하여 이를 활용하는 방법론에 가까운 포괄적인 개념입니다.
이러한 CDC는 애플리케이션 레이어에 구현할 수도 있고, 물리적 스토리지에 구축할 수도 있다고 합니다.

이전에 혼자서 무중단 마이그레이션에 대해 고민해볼 때, [PostgreSQL의 WAL](https://www.postgresql.org/docs/current/wal-intro.html)이나 [MySQL의 Redo Log](https://dev.mysql.com/doc/refman/8.0/en/innodb-redo-log.html)와 같은 개념을 적용하면 되지 않을까 라는 생각을 했었는데, CDC도 이러한 개념을 사용하고 있었습니다.

## 방법론

> 설명을 돕기 위해 최초에 존재했던 db를 v1, 새로 이전할 장소의 db를 v2라고 부르겠습니다.

마이그레이션을 진행한다고 하면, 특정 시점을 기반으로 v1 snapshot을 기준으로 v2를 생성하고, v2에 마이그레이션을 반영하게 될 것입니다. 하지만 그러는 동안에도 v1에는 새로운 데이터가 쌓이게 될 것이고, 이러한 데이터들을 놓치지 않고 v2로 잘 마이그레이션을 해 주어야 합니다.

위에서 언급한 것 처럼, v1에 기록된 데이터 중 어디까지가 v2에 반영되었는지를 알 수 있어야 나머지 v1에 있는 데이터를 잘 옮길 수 있을 것입니다.

위키피디아 CDC 문서에는 다음과 같은 방법론이 제시되고 있습니다.

- **Timestamps on rows**: 마지막 변경된 시점을 저장하고 있는 컬럼을 기반으로 변경사항을 탐지합니다.
- **Version numbers on rows**: 변경되는 매 row를 버전 번호로 표시합니다.
- **Status indicators on rows**: timestamp와 version number 방법의 상위호환으로, row에 status 컬럼을 구성하여 업데이트 되면 안되는 상황에도 대응이 가능합니다.
- **Time/version/status on rows**: 위의 세가지 방식을 합쳐놓은 방식으로 더 강력합니다. **`2023-12-02 12:00 ~ 2023-12-02 13:00` 사이에 변경된 `2.1` 버전의 모든 데이터를 capture 한다.** 와 같은 로직을 사용할 수 있습니다.
- **Triggers on tables**: 변경된 데이터에 대해 외부에서 탐지할 수 있도록 pub/sub 패턴을 적용할 수 있습니다. publish된 데이터를 playback 하여 적용할 수도 있는데, 이러한 publish 기록들을 테이블에 저장하는 방식으로 구현한다고 하면 Id, TableName, RowId, Timestamp, Operation 데이터를 저장하는 스키마를 가집니다. (예시 데이터: 1, Accounts, 76, 11/02/2008 12:15:00, Update)
- **Event programming**: 특정 조건에 부합하는 데이터에만 처리하는 등의 로직을 적용할 수 있습니다. 예를 들면, "특정 컬럼이 특정 값으로 변경된 후" 등이 있습니다.
- **Log scanners**: 위에서 언급했던 WAL과 Redo Log와 같은 트랜잭션 로그에 해당되는 부분으로, 위키피디아의 표현을 빌리자면, **거슬리지 않는 방식**으로 변경을 포착할 수 있습니다. 하지만, 이 방법은 트랜잭션 로그를 지원하고, 트랜잭션 로그의 구조, 내용 등이 특정 데이터베이스 시스템에 한정되는 문제가 있습니다. (트랜잭션 로그에는 표준이 없고, 대부분의 데이터베이스 시스템은 자사의 트랜잭션 로그에 대한 내부 포맷을 문서화하지 않고, 로그에 접근하는 인터페이스만 제공하고 있다고 합니다.) 트랜잭션 로그를 사용한 CDC 솔루션은 코드와 스키마를 변경할 필요가 없고, 무결성을 보장해주고, 낮은 레이턴시에 DB에 최소한의 영향을 주는 장점이 있는 반면, 로그에 대한 읽기, 아카이브, 변환 등에 문제가 있고, 추가적으로 트랜잭션 로그상에서 COMMIT 되었다가 ROLLBACK 된 내용은 변경되지 않은 것으로 처리하는 등 다양한 문제가 있습니다.

- **대안**: [SDC(Slowly Changing Dimension)](https://en.wikipedia.org/wiki/Slowly_changing_dimension)을 하나의 방식으로 사용하는 경우도 있습니다. 이는 Row의 상태를 표시하는 컬럼을 사용하는 방법과 유사해보입니다.

# 정리
이러한 CDC 방법론을 적용하면 다운타임이 없고 견고한 서비스를 유지하는 것이 가능합니다.
물론 마이그레이션의 영향도가 낮거나, 다운타임에 대한 리스크가 크지 않은 제품에 대해서는 불필요한 비용이 될 수도 있습니다.
이러한 CDC를 잘 적용하기 위해서는 incremental primary key를 사용하는 것이 아닌, UUID와 같은 timestamp 기반의 id를 사용하는 것이 두 데이터베이스간의 무결성을 유지하는데 도움이 될 것 같습니다.

# References
- [wikipedia: Change data capture (en)](https://en.wikipedia.org/wiki/Change_data_capture)
- [wikipedia: Change data capture (ko)](https://ko.wikipedia.org/wiki/%EB%B3%80%EA%B2%BD_%EB%8D%B0%EC%9D%B4%ED%84%B0_%EC%BA%A1%EC%B2%98)
- [29CM: API V2 전환과 DB 무중단 마이그레이션 후기](https://medium.com/29cm/api-v2-%EC%A0%84%ED%99%98%EA%B3%BC-db-%EB%AC%B4%EC%A4%91%EB%8B%A8-%EB%A7%88%EC%9D%B4%EA%B7%B8%EB%A0%88%EC%9D%B4%EC%85%98-%ED%9B%84%EA%B8%B0-8b39eb0db566)
- [DATANET: 대용량 DB 다운타임 최소화 위한 마이그레이션 전략](https://www.datanet.co.kr/news/articleView.html?idxno=181922)
- [DATNET: 4차 산업 시대 중요성 커진 '실시간 데이터 동기화'](https://www.datanet.co.kr/news/articleView.html?idxno=155922)
- [SAMSUNG SDS: 무중단 데이터 마이그레이션을 위한 필수 솔루션, CDC(Capture Data Change)](https://www.samsungsds.com/kr/insights/migration_cdc.html)
