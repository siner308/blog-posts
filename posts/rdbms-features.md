---
layout: post
title:  "RDMBS의 특징"
subtitle: "데이터베이스 면접 질문은 아니고 용어 정리"
author: "Siner"
catalog: true
header-mask:  0.3
tags:
    - database
date:   2021-10-10
multilingual: flase
---

# 동시성 제어 - MVCC (Multi Version Concurrency Control)
하나의 레코드에 대해 여러 버전이 관리된다는 의미이다. 가장 큰 목적은 Lock을 사용하지 않는 일관된 읽기를 제공하는데 있다.
- [PostgreSQL 9.3.5 문서 - 동시성 제어](https://postgresql.kr/docs/9.3/mvcc-intro.html)

1. Pessimistic Lock
처음 짐 스타키가 구현한 방식으로 MGA : Multi Generation Architecture라고 부른다. MGA는 튜플을 업데이트 할때 새로운 값으로 변경하는 것이 아니라, 즉, 같은 자리에서 Replace로 처리하는 것이 아니라 새로운 튜플을 추가하고 이전 튜플은 유효 범위를 마킹하여 처리한다.
이와 같은 방식은 PostgreSQL, SQL Server, InterBase에서 사용하는 방식이다.

2. Undo Segment
Oracle, InnoDB에서 사용하는 방식으로 **Undo** 라는 영역을 따로 두고 최신 데이터는 데이터 영역에 두고 올드 버전만 Undo 영역에 두어 레코드 갱신에 대한 버전관리를 하는 방식이다. 이것은 1980년대에 오라클의 밥 마이너가 구현한 방식이다.
만약, 변경 작업이 완료되지 않은 상태에서 다른 세션이 같은 영역에 읽기 작업을 하려고 한다면, DBMS는 각 레코드의 SCN 정보를 확인하여 Undo 영역을 찾아서 해당 버전의 레코드를 가져와 메모리에 로드하고 읽을 수 있게 처리해 준다.

<img src="https://user-images.githubusercontent.com/34048253/136663619-e075a596-8a8b-4106-81ab-7775a651e036.png" width="66%"/>

- [https://mysqldba.tistory.com/335](https://mysqldba.tistory.com/335)
- [https://joont92.github.io/db/MVCC/](https://joont92.github.io/db/MVCC/)

# 단일 서버의 트랜잭션 관리 - WAL (Write-Ahead Logging)
Write-Ahead 로깅 (WAL)은 데이터 무결성을 보장하는 표준 방법이다.
WAL을 사용하는 시스템에서 모든 수정은 적용 이전에 로그에 기록된다. 일반적으로 redo 및 undo 정보는 둘 다 로그에 저장된다.
짧게 말해서, WAL의 중심 개념은 변경을 로깅한 후에만, 즉 변경 내용을 설명하는 로그 레코드를 영구적 저장소에 먼저 기록한 후에 데이터 파일(테이블과 인덱스가 있는)의 변경 내용을 작성해야 한다는 것이다.
데이터 페이지에 적용되지 않은 변경 내용은 로그 레코드에서 실행 취소가 가능하다. (이것은 롤포워드roll-forward 복구이며, REDO라고도 한다.)
    
트랜잭션에 의해 변경된 모든 데이터 파일보다는 트랜잭션이 커밋된 것을 보장하기 위해 로그 파일만 디스크에 써야 하기 때문에 WAL을 사용하면 디스크 쓰기 수가 상당히 줄어든다. 로그 파일은 순차적으로 작성되며, 따라서 로그 파일 동기화 비용은 데이터 페이지 쓰기 비용보다 훨씬 적다. 이것은 특히 서버가 데이터 스토어의 서로 다른 부분을 건드리는 소규모 트랜잭션을 다수 처리하는 경우에 그렇다. 또한, 서버가 소규모 동시 트랜잭션을 다수 처리하는 경우에 로그 파일의 fsync 하나로 여러 가지 트랜잭션을 충분히 커밋할 수 있다. (postgres의 경우 1gb를 max wal size를 기본으로 한다)

- [로그 선행 기입 - 위키백과, 우리 모두의 백과사전](https://ko.wikipedia.org/wiki/%EB%A1%9C%EA%B7%B8_%EC%84%A0%ED%96%89_%EA%B8%B0%EC%9E%85)
- [Write-Ahead 로깅(WAL)](https://postgresql.kr/docs/9.6/wal-intro.html)
- [NAVER D2 - DBMS는 어떻게 트랜잭션을 관리할까?](https://d2.naver.com/helloworld/407507)

# MSA 트랜잭션 관리 - `2-Phase Commit`
2개 이상의 서비스에서 데이터를 생성해야 하는 경우, transaction 처리를 안전하게 수행하는 방법.
- [마이크로 서비스에서 분산 트랜잭션](https://giljae.medium.com/%EB%A7%88%EC%9D%B4%ED%81%AC%EB%A1%9C-%EC%84%9C%EB%B9%84%EC%8A%A4%EC%97%90%EC%84%9C-%EB%B6%84%EC%82%B0-%ED%8A%B8%EB%9E%9C%EC%9E%AD%EC%85%98-347af5136c87)

## 2-Phase Commit
Transaction Coordinator가 각 서비스의 commit, rollback을 제어하는 방법
        
![image](https://user-images.githubusercontent.com/34048253/136663755-7f128904-398c-47b4-9efd-60cb8f470b6f.png)

쇼핑몰을 예시로 들면 고객이 결재를 완료 했을 경우
1. 재고관리서비스에서는 재고를 차감합니다.
2. 배송 서비스에서는 배송정보를 생성합니다.
3. 주문정보의 상태를 발송으로 변경합니다.
        
![image](https://user-images.githubusercontent.com/34048253/136663786-98d17707-24c8-481d-93ca-764279aa6ed4.png)

Two Phase Commit 방식에서는 우선 각 데이터 Prepare 상태로 처리 합니다. 각 서비스에서 처리가 완료 되었을 경우 Transaction Coordinator에 Commit 준비 완료 메세지를 보냅니다.
        
![image](https://user-images.githubusercontent.com/34048253/136663808-38f0d1d0-9952-4d22-b8d4-43b24ae5e337.png)

Commit 준비가 완료되면 각 서비스에서 Coomit을 진행합니다.

![image](https://user-images.githubusercontent.com/34048253/136663821-1d9ff609-f7bd-480d-b0ab-b38105aba77d.png)

만일 위와 같이 배송정보 생성 중 오류가 발생할 경우 Transaction Coordinator 실패 메세지를 보냅니다.

![image](https://user-images.githubusercontent.com/34048253/136663832-093af8f2-87ba-4fb7-877d-86ff8fb42444.png)

Transaction Coordinator에서 각 서비스에 Rollback 하도록 지시합니다.
Two Phase Commit의 경우 NoSQL에서는 지원을 하지 않으며, 교착상태가 발생할 우려가 있습니다.
        
[2단계 커밋 프로토콜 - 위키백과, 우리 모두의 백과사전](https://ko.wikipedia.org/wiki/2%EB%8B%A8%EA%B3%84_%EC%BB%A4%EB%B0%8B_%ED%94%84%EB%A1%9C%ED%86%A0%EC%BD%9C)
        
## SAGA

- Choreography

![image](https://user-images.githubusercontent.com/34048253/136663884-de1d7557-68c6-4d38-8a63-d388066ef912.png)

Choreography-based SAGA는 위와 같이 재고 차감이 완료되면 배송서비스에 이벤트를 전달하여, 배송서비스에서 배송정보를 생성 후, 주문서비스에 이벤트를 전달하여 주문정보를 변경하는 순차적으로 이벤트로가 전달되면서 트랙잭션이 관리되는 방식입니다.
                
![image](https://user-images.githubusercontent.com/34048253/136663891-b94f3cbf-f334-4b9f-b29f-8de448793d91.png)

만일 위와 같이 배송정보 생성 중 오류가 발생할 경우, 배송에서는 재고관리 서비스로 재고 복원 이벤트를 전달하게 되며, 재고 복원이 만료되면 결재서비스로 결제 취소 이벤트를 전달하여 결제가 취소되는 방식으로 관리됩니다.

- Orchestration
            
Orchestration-based SAGA는 SAGA Orchestrator에서 각 서비스에게 호출하여 트랜잭션을 관리하는 방식입니다.
            
1. 주문 서비스가 주문을 저장을 하고, SAGA Orchestrator 에게 트랜잭션을 시작하도록 요청합니다.
2. Orchestrator는 지불 명령을 Payment 서비스로 보내고, 지불 완료 메세지로 응답합니다.
3. Orchestrator는 주문 준비 명령을 Order 서비스로 보내고, 주문 준비 메세지로 응답합니다.
4. Orchestrator는 배달 명령을 Delivery 서비스로 보내고, 주문 배달 메세지로 응답합니다.
5. 모든 응답이 완료되면 트랜잭션을 종료합니다.
            
![image](https://user-images.githubusercontent.com/34048253/136664187-7092416e-073d-4c4a-adbb-12e74e5a6227.png)
![image](https://user-images.githubusercontent.com/34048253/136664704-d3c46822-a0fb-4362-b984-5ae1c779dca8.png)

롤백 과정을 살펴 보면
            
1. Stock 서비스에서 품절 메세지를 Orchestrator에 전달합니다.
2. Orchestrator에서 롤백을 시작합니다.
3. Orchestrator는 환불 명령을 Payment 서비스로 보내고, 주문실패 상태로 변경합니다.
            
Orchestration-based SAGA는 트랜잭션을 Orchestrator 집중해서 관리 할 수 있으므로, 복잡도가 줄어들어 롤백을 쉽게 관리할수 있고, 구현 및 테스트가 용이합니다. 그러나 Orchestrator에 모든 트랙잭션이 집중되어 로직이 복잡해 질 수 있고, Orchestrator 추가로 인한 인프라도 고려해야 합니다.

- [Microservices Pattern: Sagas](https://microservices.io/patterns/data/saga.html)
        
## 3-Phase Commit
특정 가정이 충족되었을때, 2PC의 blocking문제를 해결할 수 있는 프로토콜이다.
3PC는 한글 자료도 없다... 관심있으신분들만 보시길
        
- [Three-phase commit protocol - Wikipedia](https://en.wikipedia.org/wiki/Three-phase_commit_protocol)
    
# OLTP vs OLAP

## OLTP (Online Transaction Processing)
        
복수의 사용자 pc에서 발생되는 Tx를 DB서버가 처리하고, 그 결과를 사용자 PC에 결과값을 되돌려주는 과정. (1개의 요청작업을 처리하는 과정을 OLTP라고 한다)
        
## OLAP (Online Analytical Processing)
        
OLTP가 데이터 자체의 처리에 중점이 된 용어라면, OLAP는 이미 저장된 데이터를 기반하여 분석하는데 중점이 된 용어이다.
        
OLAP는 Data Warehouse에 저장되어있는 데이터를 분석하고, 이를 통해 사용자에겡 유의미한 정보를 제공해주는 처리방법을 의미함. 이런 정보(information: data를 분석한 것)를 바탕으로 보다 복잡한 모델링을 가능하게 한다. (Data를 요구와 목적에 맞게 분석하여 Information을 제공하는 것을 OLAP라고 함)
        
![image](https://user-images.githubusercontent.com/34048253/136664336-1f8812df-9e1f-4fb6-9632-f6d04e56f7fa.png)

- [[DATABASE] OLTP와 OLAP의 차이점 & 예시](https://too612.tistory.com/511)
    
# Cold Backup, Hot Backup

1. Cold Backup (닫힌 백업)

서버를 셧다운 시킨 상태에서 백업. 
백업받는 동안은 DB 서비스 불가능.
        
2. Hot Backup (열린 백업)
        
서버가 오픈된 상태에서 백업        
백업중에도 서비스 가능.
REDO로그의 양이 굉장히 증가하므로, 어지간하면 DML작업이 다량으로 일어나는 시간에는 하지 말자.
        
> 열린백업을 받는 동안은 해당 테이블스페이스를 오프라인 시켜 놓는데, 그렇다면 백업 동안은 내용 변경이 불가능할까? 그렇지 않다. 백업 동안의 변경되는 내용은 리두로그파일에 저장된다.
>이는 열린백업 동안 DML작업이 일어나면 많은 양의 리두로그가 발생하게 되고 백업이 오래 지속되면 그 동안에 변경된 내용은 전부 리두로그파일에 저장되었다가 백업이 끝나게 되면 다시 데이터파일에 적용해야 하기 때문에 반드시 아카이브로그모드로 작동해야 하는 것이다.
 
- [[B/R] Cold backup과 Hot backup](https://sksstar.tistory.com/120)
