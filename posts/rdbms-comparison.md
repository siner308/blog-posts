---
layout: post
title:  "주요 RDBMS의 종류"
subtitle: "mysql vs mariadb vs postgresql vs sqlite vs oracle vs mssql"
author: "Siner"
catalog: true
header-mask:  0.3
tags:
    - rdbms
date:   2021-10-11
multilingual: flase
---

# 이전 게시물인 [RDBMS의 특징](https://blog.siner.io/2021/10/10/rdbms-features)를 보고 오면 좋습니다.

>RDBMS DB 엔진 중 유명한 것들을 비교해보고, 특징을 알아보자.
>이 게시글을 보는 모두가 데이터베이스를 고르는 과정에 도움이 되었으면 한다. 

![image](https://user-images.githubusercontent.com/34048253/136786870-419bd656-9630-49a9-95b4-da4f4555f36f.png)

PostgreSQL과 MariaDB의 상승세가 뚜렷하다

![image](https://user-images.githubusercontent.com/34048253/136786988-02b51c7e-3f57-4376-a494-85ff62817646.png)

- [DB-Engines Ranking](https://db-engines.com/en/ranking)

# Oracle
    
Oracle사에서 제공하는 DBMS. 온프레미스, 클라우드, 하이브리드 클라우드 등의 환경을 제공한다.
    
오랜 역사와 막강한 영업력으로 데이터베이스 시장 부동의 1위. 기업시장을 위시로 한 오픈소스 데이터베이스를 쓸 수 없는 환경(문제 발생시 책임소재 등의 이유)에서는 대부분 오라클이 쓰인다. 특히 극한의 신뢰성이 요구되는 미션크리티컬 환경은 오라클이 꽉 잡고 있다.
    
카카오뱅크가 이례적으로 오라클과 함께 오픈소스 DB인 MySQL을 부분적으로 사용한다는 점만으로 화제가 될 만큼 공고한 시장장악력을 가지고 있다.
    
온라인 게임의 경우 대표적으로 월드 오브 워크래프트가 오라클로 구성되어 있다.
    
하지만 비싼 가격과 오라클의 갑질 등의 이유로 오라클을 쓰다가 이탈한 고객들도 은근이 많다고 함.
    
티맥스소프트의 티베로가 이 부분을 노린 제품으로, 자국 버프와 저렴한 가격, 오라클과의 호환성 등의 이유로 공공시장에서 오라클의 자리를 일부 뺏어오는데 성공했다고 한다.
    
AWS도 메인 DB를 오라클에서 다른 오픈소스 DB로 교체하는 계획을 진행중에 있다.
    
- ["추락하는 오라클, 날개조차 찾을 수 없다?" - BI KOREA](http://www.bikorea.net/news/articleView.html?idxno=23767)
- [탈 오라클? "대체제 아직까지는 없다"](https://www.fnnews.com/news/202002091245065540)
- [[해설] 오라클의 '초강수', 금융권 IT전략 고민 깊어진다](http://m.ddaily.co.kr/m/m_article/?no=201553)
- [Oracle - Releases and versions](https://en.wikipedia.org/wiki/Oracle_Database#Releases_and_versions)
    
- 특징
  - flashback
    - 커밋 이전 상태로 되돌리는 기능
    - 과거 시점의 데이터 조회도 가능
    ex) SELECT * FORM 고객 AS OF TIMESTAMP (SYSTIMESTAMP - INTERVAL '1' HOUR)
    - 특정 시점에 존재했던 레코드를 조회해 새로운 테이블을 생성-추가 할 수 있음
    ```
    CREATE TABLE 고객_BACKUP AS SELECT * FROM 고객 AS OF TIMESTAMP TO_DATE(‘201501101020’, ‘YYYYMMDDHH24MI’)
    ```
  - GATHER_PLAN_STATISTICS
    - SQL Trace를 수행하지 않고도 쿼리 plan을 단계별로 get block을 알수있다. 쿼리 성능을 확인-비교 할 수 있기 때문에 튜닝할때 아주 빈번하게 쓰인다고 한다.
                
![image](https://user-images.githubusercontent.com/34048253/136787268-70b5322b-9d06-4796-ab5f-8335e1342178.png)                
    
- 장점
  - ~~돈내고 쓰는만큼 서포트를 해준다~~
  - 고성능 트랜잭션 처리를 제공하여 속도가 빠르다
  - 대규모 데이터베이스 지원
  - SQL문을 실행하는 가장 효율적인 방법을 선택한다. 쿼리비용 최소화를 위한 테이블 인덱싱 분석
- 단점
  - 구동에 고사양의 장비가 필요하다 (메모리 너무 많이 먹어서 컨테이너로 띄울수없다)
  - 비싸다 (특히 클라우드)
    
# MySQL

> MySQL의 실질적인 소유주는 오라클이다.
> 오라클은 자체 상용 DBMS인 오라클 데이터베이스를 가지고 있고, 오픈 소스에 대해 호의적이지 않은데다 프로그램이 갈수록 복잡해지고 있어서 MySQL 사용자들 사이에서도 불안감이 커지고 있다. 
> 그래서 오픈 소스 진영에서 MySQL을 모태로 MariaDB라는 DBMS를 만들었다. 리눅스 배포판 중 페도라와 오픈수세는 MySQL을 버리고 MariaDB를 장착했다. 애플은 OS X 서버 버전에서 MySQL을 버리고 PostgreSQL을 채용했다.
 

- 인수 히스토리
  1. 1995년 설립된 MySQL AB에서 개발되었다
  2. 2008년 썬마이크로시스템즈에 의해 인수되었다
  3. 2010년 오라클에 의해 인수되었다 (이 시점에 mariaDB와 갈라지게 되었다)

- 특징
  - top n개의 레코드를 가지고 오는 케이스에 특화되어있다
  - 웹 애플리케이션으로서의 MySQL의 인기는 PHP의 인기도와 맞물려있다.
  - 복잡한 알고리즘은 가급적 지원하지 않는다
  - 간단한 처리속도를 향상시키는 것을 추구한다
  - 정확함
- 장점
  - 오픈소스로 무료 사용이 가능하다
  - 오픈소스이다?
- 단점
  - 문자열 비교에서 대소문자를 구분하지 않는다
  - nested loop join만 지원한다
    - 바깥 테이블의 처리 범위를 하나씩 접근하면서 추출된 값으로 안쪽 테이블을 조인하는 방식
    - 중첩 루프문과 동일한 원리
    - 좁은 범위에 유리하다
    - 순차적으로 처리한다
  - 과거의 치명적 문제
    - [https://gywn.net/2013/02/mysql-innodb-auto-increment/](https://gywn.net/2013/02/mysql-innodb-auto-increment/)

# MSSQL

- 특징
  - 관대함
- 장점
  - 엔터프라이즈 급 관리 소프트웨어
  - 우수한 데이터 복구 지원
- 단점
  - 비싸다
  - `windows 기반 서버에서만 실행되도록 설계되어있다.`


## MySQL vs MSSQL

  - MYSQL : 정확함
  - MSSQL : 관대함
    
- [[NDC] '리니지2' DB migration : MSSQL과 mysql](https://www.youtube.com/watch?v=pPQto83BoRk)
  
![image](https://user-images.githubusercontent.com/34048253/136787718-b9827308-e3bd-47af-887e-de802142a163.png)
![image](https://user-images.githubusercontent.com/34048253/136787751-2808c067-10cf-449f-95e5-1fb52178b687.png)

  - ODBC
![image](https://user-images.githubusercontent.com/34048253/136787781-aa12d8d9-814c-4b5c-94fa-b644b43d47cd.png)
        
  - 장점
    - 다양한 DBMS를 동일한 인터페이스로 접근 가능
  - 단점
    - 실제 내부 구현이 DBMS마다 서로 다르다
    - mysql: multi-thread safe하지 않음 : setlocale()를 사용 안한다는 옵션을 설정해야 multi-thread를 사용 가능함
  - 문자열
        
![image](https://user-images.githubusercontent.com/34048253/136787880-738e6d74-3061-46e2-a5e9-385eca05fc66.png)

  - 16진수 상수
        
MSSQL에서는 integer로 제대로 반환해주었지만, MYSQL에서는 string으로 판단했음
        
![image](https://user-images.githubusercontent.com/34048253/136787956-b4ca518b-e78e-41ec-b812-401d53bf4e92.png)

![image](https://user-images.githubusercontent.com/34048253/136787972-a12cfdfc-b2e3-4b71-b29e-0bddc6b79002.png)

- 1000배정도 느리다고 함
    
# PostgrSQL
- 특징
  - 다양한 join 방법을 제공한다
    - nested loop join, hash join, sort merge join
  - update시 과거 행을 삭제하고 변경된 데이터를 가진 새로운 행을 추가하는 형태라서 update 성능이 좋지 않다
  - 처리속도를 빠르게 하기 위해 여러 cpu를 활용하여 쿼리를 실행한다
  - 데이터베이스 클러스터 백업 기능을 제공한다
  - 동시성 제공을 위한 MVCC를 제공한다
    - MVCC (Multi-Version Concurrency Control)
  - 데이터 무결성을 보장하기 위한 WAL을 제공한다
    - [WAL (Write-Ahead Logging)](https://postgresql.kr/docs/9.6/wal-intro.html)
  - hot backup + wal replay를 통해 원하는 시점 복구가 가능하다
    
![image](https://user-images.githubusercontent.com/34048253/136788117-51d3af07-778c-4f49-ab99-ef55c3d9aa64.png)

- 강력한 db admin인 pgAdmin
 
# SQLite

SQLite란?
> SQLite는 [self-contained](https://www.sqlite.org/selfcontained.html), [serverless](https://www.sqlite.org/serverless.html), [zero-configuration](https://www.sqlite.org/zeroconf.html), [transactional](https://www.sqlite.org/transactional.html)한 SQL 데이터베이스 엔진입니다.
> SQLite는 오픈소스이고, [유명한 프로젝트](https://www.sqlite.org/famous.html)에서 많이 사용되고 있습니다. 
    
SQLite의 코드는 [international team](https://www.sqlite.org/crew.html)이 풀타임으로 서포트하고 있고, 원한다면 [professional support](https://www.sqlite.org/prosupport.html)도 가능합니다. SQLite 프로젝트는 2000년 5월 9일에 시작되었고, 2050년까지 지원하는 것을 목표로 하고 있습니다.
    
- [C언어로 구현되어있다.](https://sqlite.org/src/dir?ci=trunk) (현재 3.35.5버전까지 릴리즈 되어있다.)
- [지원기능](https://www.sqlite.org/fullsql.html)    
- Transaction [https://gywn.net/2013/08/let-me-intorduce-sqlite/](https://gywn.net/2013/08/let-me-intorduce-sqlite/)
  - Journal Mode
  - WAL Mode

- 장점
  - 파일을 통째로 복사하면 백업이 끝난다.
  - 설정이 필요없다.
  - 빠르다.
  - 작다.
  - db file은 크로스플랫폼을 지원하여 32-bit와 64-bit 시스템 또는, big endian과 little-endian 아키텍처 사이에서 자유롭게 copy and paste를 통해 오갈 수 있음.
  - 라이브러리의 모든 기능을 포함하더라도 600KB 미만으로 매우 경량화되어있음. 적은 메모리 환경에서도 좋은 성능을 보이고, 사용법에 따라 file I/O보다 더 빠를 수도 있음.
  - 임베디드 데이터베이스 엔진이므로, 별도의 서버가 필요없어서 모바일 기기 등 많은곳에서 사용된다.
    
- 단점
  - 한명의 사용자만 동시 접근이 가능함. 멀티유저를 지원하려면 프록시 레이어가 필요함.
  - 복잡하거나 큰 데이터의 보관이 적절치 않음.
  - 유니코드만 사용 가능
  - SQL Parser가 성능저하의 원인이 됨.
  - 룩업테이블의 경우, 다른 데이터베이스는 해시탐색 알고리즘을 사용하여 O(1)이지만, SQLite는 R-Tree를 사용하여 O(log n) 수준임.
  - 룩업 테이블 전체가 메모리에 올라가있는 반면, SQLite는 FileIO 기반이므로 성능을 대폭 저하시킴. → Batch Job에 적절하지 않음.
  - 경량화의 이유로 편의기능은 부족하다.

- [SQLite Backup API](https://www.sqlite.org/backup.html)
    
# mariaDB

오라클 소유의 현재 `불확실한 MySQL의 라이선스 상태에 반발`하여 만들어짐.

5.5버전까지는 MySQL의 모든 기능을 포함하고 있었으나, 이후의 개발판에서는 10.x로 버전을 변경하여 MySQL과 궤를 달리하여 MariaDB의 색깔을 분명히 하고자 했다.
    
→ MySQL의 명령어를 사용하는 mariaDB는 반쪽짜리...
    
MariaDB는 `GPL v2` 라이선스를 따르는 순수한 오픈소스 프로젝트이기에 오라클로부터 자유롭다.
    
MariaDB 커뮤니티는 MySQL과 비교해 애플리케이션 부분 속도가 `약 4~5천배 정도 빠르며`, MySQL이 가지고 있는 모든 제품의 기능을 완벽히 구현하면서도 `성능 면에서는 최고 70%의 향상`을 보이고 있다고 주장한다.
    
- 특징
  - backup
    - MariaBackup
      - Percona XtraBackup의 fork본으로 10.1.23이상 버전에 포함되어있음
    - mysqldump
      - 논리적 백업을 수행. 가장 flexible한 방법이며, 데이터 크기가 상대적으로 작은 경우에 적합함
      - 데이터가 클 경우, 백업 파일이 클 수 있고, 복원 시간이 길어질 수 있음
      - mysqlhotcopy (deprecated)
      - XtraBackup (10.1버전 또는 그 이후부터는 Mariabackup 사용을 권장)
        - 빠르게 hot backup을 수행할 수 있음
      - Filesystem snapshot
        - snapshot중에는 테이블이 lock됨
      - LVM
        - Perl 스크립트를 사용
    
- 다양한 스토리지 엔진
  - XtraDB - 오라클 InnoDB를 대체하기 위해 만들어진 InnoDB 파생 포크 프로젝트
  - Aria - MyISAM 파생 엔진 대체용
  - SphinxSE - 5.2이상에서 지원, Full-Text Searching이 필요할때 사용 가능.
  - Cassandra - 10.0에서 포함되어, nosql 저장엔진을 끌어들이려 함
  - FederatedX - MySQL Federated 파생 엔진, 트랜잭션 제공
  - OQGRAPH - 5.2이상에서 지원
  - TokuDB - 다른 엔진보다 압축률이 높음

## MySQL vs mariaDB
  
- [Benchmark: MariaDB vs MySQL on Commodity Cloud Hardware | MariaDB](https://mariadb.com/ko/resources/blog/benchmark-mariadb-vs-mysql-on-commodity-cloud-hardware/)
    
퍼포먼스 성능 비교
    
- [MariaDB와 MySQL 선택에 대해서 > SIR](https://sir.kr/cm_free/1489073)
    
공통점과 차이점
    
- [MariaDB vs MySQL: [2021] Everything You Need to Know](https://hackr.io/blog/mariadb-vs-mysql)
    
아무튼 mariaDB가 좋음
    

- [RDBMS의 순위와 특징](https://www.notion.so/3c7c0583f7a44ad7bacc9d37d1f3d493)
- [기업별 rdbms 사용현황](https://www.notion.so/4fc295a0b8c34805a980127bd9103219)
- [mysql vs postgresql](https://valuefactory.tistory.com/497)
- [Oracle의 특징](https://ssmsig.tistory.com/37)
- [Nested Loop, Sort-Merge, Hash Join 조인연산](https://needjarvis.tistory.com/162)
- [PostgreSQL, MSSQL, MySQL, Oracle 대용량 DB 비교](https://periar.tistory.com/208)
- [[DBMS comparison] Oracle vs SQL server vs MySQL vs MariaDB vs PostgreSQL part2. Performance with single client](https://opendatabase.tistory.com/entry/DBMS-comparison-Oracle-vs-SQL-server-vs-MySQLinnodb-vs-MariaDBAria-vs-Postgres-part2-one-user)
- [MariaDB Administration - 백업과 복구 개요](https://hobogi.tistory.com/entry/MariaDB-Administration-%EB%B0%B1%EC%97%85%EA%B3%BC-%EB%B3%B5%EA%B5%AC-%EA%B0%9C%EC%9A%94)
- [MariaDB의 InnoDB 엔진, XtraDB엔진 - RastaLion's IT Blog](https://rastalion.me/mysql%EC%9D%98-innodb-%EC%97%94%EC%A7%84-mariadb%EC%9D%98-xtradb%EC%97%94%EC%A7%84/)


# 결론

![image](https://user-images.githubusercontent.com/34048253/136787090-4dece941-62d9-451e-8d9a-8d56584de5ef.png)
