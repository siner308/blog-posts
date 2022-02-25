---
title: "쿠버네티스 정리 2. 아키텍처"
subtitle: "마스터 노드, 워커 노드의 역할"
tags:
  - kubernetes
  - container
date: 2022-02-25
---

[이전 글](https://blog.siner.io/2022/02/21/kubernetes-1)에 이어서 작성합니다.

쿠버네티스의 Node에는 master, slave 이렇게 두가지 타입이 있습니다.
클러스터 내부에서 master와 slave Node가 각각 어떤 역할을 하는지 알아봅시다.

그리고 이를 통해 쿠버네티스가 어떻게 수동적인 관리 없이 셀프 매니징, 셀프 힐링을 하는지 알아보겠습니다.

---

# Worker Node
각 Node에는 여러개의 Pod들이 존재합니다.
Pod을 관리하는 모든 노드에는 필수로 설치되어야 하는 3개의 프로세스가 있습니다.

## 1. container runtime
컨테이너를 동작시키기 위한 프로그램입니다.
예를 들어, docker와 같은 프로세스를 컨테이너 런타임이라고 합니다.

## 2. kubelet
Kubelet은 Pod에서 컨테이너가 확실하게 동작하도록 관리합니다.

## 3. kubeproxy
Service에서 Pod으로 요청을 넘겨주기 위한 process입니다.
kubeproxy는 overhead를 줄이기 위한 처리들을 해줍니다.

예를 들어, A와 B 두개의 노드가 있을때, A노드와 B노드 양쪽에 같은 Service의 Pod이 존재한다면, 
A에서 해당 Service에 요청을 보낼때 B노드로 보내지 않고, network overhead를 낮추기 위해 A에 있는 서비스로 요청을 넘겨줍니다.

<img src="https://user-images.githubusercontent.com/34048253/155730179-80849efd-dccd-4894-975e-4ce6b518e4c2.png" width="70%">

---

# Master Node
Pod의 동작을 Worker Node에서 담당 해준다면, 다음의 것들은 클러스터에서 어떻게 처리해주는지 생각해봅시다.

1. Pod가 어디에 떠야하는가?
2. 모니터링은 어떻게 되고있는가?
3. Pod의 재배치, 재시작은 누가 처리해주는가?
4. 새로운 노드가 추가된다면 어떻게 해야하는가?

위의 4가지 질문을 처리해주는 노드를 Master 노드라고 합니다.

Worker Node와는 전혀 다른 4개의 프로세스를 가지고 있습니다.

## 1. Api Server
만일 당신이 새로운 애플리케이션을 k8s 클러스터에 배포하고싶다면, 당신은 Api Server와 통신해야합니다.
Api Server 클라이언트로는 k8s dashboard와 같은 UI, kubelet와 같은 CLI 도구, 또는 k8s API가 있습니다.

API Server는 클러스터를 조회하거나, 업데이트할때 진입로 역할을 하기 때문에 Cluster Gateway와도 같습니다.
또한 Authentication도 수행해주기 때문에 게이트키퍼로써의 역할도 합니다.

만약 당신이 k8s 클러스터에 어떤 요청을 보낸다고 하면, 다음의 과정을 거칩니다.

요청 -> API Server -> 요청 검증 (validate) -> ... -> Pod

무조건 API Server를 통해야 한다는 점 때문에 보안상 이점이 있습니다.

## 2. Scheduler
사용자가 API Server를 통해 새로운 Pod을 생성하겠다는 요청을 보냈을때, API Server는 Scheduler에게 요청을 보냅니다.

이후, Scheduler는 Worker Node중 하나를 선택하는데, 
이때 각 노드별 사용 가능한 리소스와, Pod이 필요로 하는 리소스를 비교하여 배치하기 적당한 Node를 정합니다.

<img src="https://user-images.githubusercontent.com/34048253/155732647-7a550481-e417-4173-8250-d4701222018c.png" width="100%">

## 3. Controller manager
Pod이 죽는것을 탐지하고, 다시 회복될 수 있도록 Scheduler 에게 요청합니다.
Scheduler는 적절한 Node에 있는 kubelet에게 Pod을 생성하도록 요청합니다.

<img src="https://user-images.githubusercontent.com/34048253/155733146-8622b1c7-53bd-4ea9-9169-51cfb30b2a80.png" width="100%">

## 4. etcd

cluster의 상태를 저장하는 key value 저장소입니다.
scheduler, controller manager의 동작이 etcd의 data를 기반으로 동작하기 때문에, etcd는 cluster의 뇌라도 할 수 있습니다.

Scheduler가 사용 가능한 리소스가 어떻게 되는지 아는것과, Controller manager가 cluster의 상태가 변했는지 (Pod이 죽었는지 등) 등에 대해 아는것도 etcd 덕분입니다.

**사용자의 애플리케이션 데이터는 etcd에 저장되지 않습니다.** 


클러스터에는 여러개의 Master Node가 있기때문에, etcd는 분산되어 저장되는 스토리지입니다.

# 정리
최소한의 클러스터 구성은 2개의 Master 노드와 3개의 Worker 노드입니다.

Master 노드는 매우 중요하지만, 실제로 연산하는 양이 많지는 않기때문에, 리소스를 크게 잡아먹지 않습니다.

Worker 노드는 실제 애플리케이션 작업을 수행하기 때문에 많은 리소스를 할당해야합니다.

Worker 노드가 많아질수록, Master 노드에 가해지는 부하가 증가하기때문에, 적당한 비율로 같이 늘려가야합니다. (2 Master, 3 Worker -> 3 Master, 6 Worker)
