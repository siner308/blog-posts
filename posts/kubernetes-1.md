---
title: "쿠버네티스 정리 1. 컴포넌트 종류"
subtitle: "쿠버네티스가 각 문제들을 해결하는 방법"
tags:
    - kubernetes
    - container
date: 2022-02-21
image: https://user-images.githubusercontent.com/34048253/154848123-1601c82d-85ee-486e-aa7d-082aa0ce7935.png
---

쿠버네티스 관련 괜찮은 영상이 있어서 시청 하면서 메모한 내용을 정리해두고자 합니다.

<iframe width="560" height="315" src="https://www.youtube.com/embed/X48VuDVv0do" title="YouTube video player" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture" allowfullscreen></iframe>

# k8s란?
쿠버네티스(kubernetes, 또는 k8s)는 구글이 만든 오픈소스 컨테이너 오케스트레이션 툴로, 컨테이너 애플리케이션의 관리를 돕습니다. 물리서버, 클라우드서버 등의 다양한 환경에서 사용이 가능합니다.

아래의 케이스에 해당되는 경우, 고려해볼 수 있습니다.
- 트렌드에 따라 모노리틱 아키텍처(Monilithic Architecture)에서 마이크로 서비스 아키텍처(Microservice Architecture, 또는 MSA)로의 전환을 하고 싶은 경우.
- 컨테이너의 사용이 증가하는 경우.
- 수백개의 컨테이너를 **잘** 관리하고 싶은 경우.

k8s는 다음의 기능들을 제공합니다.
- downtime이 없는 High Availability(HA)
- 스케일링
- 높은 퍼포먼스
- 장애 회복 (disaster recovery)

# k8s 컴포넌트
쿠버네티스에는 정말 많은 구성요소들이 있는데, 이들을 컴포넌트라고 부릅니다.<br>
쿠버네티스의 수많은 컴포넌트들을 하나씩 알아봅시다.

## Node
> 쿠버네티스는 컨테이너를 파드내에 배치하고 노드 에서 실행함으로 워크로드를 구동한다. 노드는 클러스터에 따라 가상 또는 물리적 머신일 수 있다. 각 노드는 [컨트롤 플레인](https://kubernetes.io/ko/docs/reference/glossary/?all=true#term-control-plane)에 의해 관리되며 [파드](https://kubernetes.io/ko/docs/concepts/workloads/pods/)를 실행하는 데 필요한 서비스를 포함한다.
>
>일반적으로 클러스터에는 여러 개의 노드가 있으며, 학습 또는 리소스가 제한되는 환경에서는 하나만 있을 수도 있다.<br>
> [kubernetes.com](https://kubernetes.io/ko/docs/concepts/architecture/nodes/)

Node는 물리서버 또는 가상머신으로, 간단하게 말하자면 하나의 서버를 뜻합니다.<br>
Node는 Master Node와 Worker Node로 나뉩니다.

## Cluster
> 컨테이너화된 애플리케이션을 실행하는 [노드](https://kubernetes.io/ko/docs/concepts/architecture/nodes/)라고 하는 워커 머신의 집합. 모든 클러스터는 최소 한 개의 워커 노드를 가진다.<br>
> [kubernetes.io](https://kubernetes.io/ko/docs/concepts/overview/components/)

## Pod
> 파드(Pod) 는 쿠버네티스에서 생성하고 관리할 수 있는 배포 가능한 가장 작은 컴퓨팅 단위이다.<br>
> [kubernetes.com](https://kubernetes.io/ko/docs/concepts/workloads/pods/)

Pod는 k8s에서 가장 작은 유닛입니다. Pod는 container의 상위 레이어이자, container를 추상화 한 것입니다.
일반적으로, 하나의 Pod은 하나의 application을 담고 있습니다. (하나의 application이라는 말이 오직 하나의 container만을 담고있다는 뜻은 아닙니다.)
각 Pod은 고유한 IP주소 를 가지고 있고, 이 IP주소를 통해 통신합니다.
Pod에 문제가 생겨 crash가 발생하면, 새로운 Pod으로 대체되고 새로운 IP주소를 얻습니다.

## Service
Service는 변하지 않는 IP Address를 가지고 있습니다.
Pod은 Service를 가지고 있습니다.
Pod이 죽더라도, Service의 IP주소는 변하지 않습니다.
데이터베이스와 같이 외부 접근을 차단해야 하는 경우엔 내부 서비스로 둘 수 있고, 외부와의 통신을 위해 서비스를 외부로 열어둘 수 있습니다.

## Ingress
Service의 IP주소를 통해 외부에서 접근이 가능하지만, Ingress를 사용하면 siner.io와 같이 도메인 주소를 통한 접근이 가능해집니다.

다음 이미지는 쿠버네티스 공식에서 제공하는 인그레스의 간단한 예시입니다.

<img width="100%" alt="스크린샷 2022-02-20 오후 11 41 52" src="https://user-images.githubusercontent.com/34048253/154848123-1601c82d-85ee-486e-aa7d-082aa0ce7935.png">

## ConfigMap
> 컨피그맵은 키-값 쌍으로 기밀이 아닌 데이터를 저장하는 데 사용하는 API 오브젝트이다. 
> [파드](https://kubernetes.io/ko/docs/concepts/workloads/pods/)는 [볼륨](https://kubernetes.io/ko/docs/concepts/storage/volumes/)에서 환경 변수, 커맨드-라인 인수 또는 구성 파일로 컨피그맵을 사용할 수 있다.<br>
>[kubernetes.io](https://kubernetes.io/ko/docs/concepts/configuration/configmap/)

ConfigMap을 소개하기 위해 하나의 예시를 들어보겠습니다.
백엔드 Service가 데이터베이스 Service에 접근하기 위해서는 데이터베이스의 주소, 유저네임, 비밀번호와 같은 env를 저장해야합니다. service의 endpoint가 변경되면 url이 변경될텐데, 이를 적용해서 rebuild 등의 작업을 거치려면 작은 변화에도 많은것이 변경되기때문에, 이를 위한 Configuration 저장소인 ConfigMap이 있습니다.
하지만 Credential과 같은 중요한 정보는 여기가 아닌 Secret에 저장합니다.

## Secret
>시크릿은 암호, 토큰 또는 키와 같은 소량의 중요한 데이터를 포함하는 오브젝트이다. 
>이를 사용하지 않으면 중요한 정보가 [파드](https://kubernetes.io/ko/docs/concepts/workloads/pods/) 명세나 [컨테이너 이미지](https://kubernetes.io/ko/docs/reference/glossary/?all=true#term-image)에 포함될 수 있다.
>시크릿을 사용한다는 것은 사용자의 기밀 데이터를 애플리케이션 코드에 넣을 필요가 없음을 뜻한다.<br>
>[kubernetes.io](https://kubernetes.io/ko/docs/concepts/configuration/secret/)

ConfigMap에 넣을 수 없는 secret data를 넣기위한 저장소입니다.
base64 형태로 인코딩해서 저장합니다.

## Volume
데이터베이스와 같은 정보는 어딘가에 저장 되어야 하는데 container의 특성상 사라지기 쉽기때문에 Pod의 내부 저장소를 사용하는것은 불가능합니다. 그리고 data를 저장하는 장소는 오랫동안 안정적인 상태를 유지할 수 있어야 합니다.
이를위한 컴포넌트가 Volume입니다.

Volume은 물리 저장소 또는 Pod이 될 수 있습니다.
Volume은 같은 Node 또는 리모트 Node에 존재할 수도 있고, k8s Cluster 외부에 존재할 수도 있습니다.

k8s는 data persistance를 관리해주지 않습니다. -> 개발자가 잘 처리해서 관리해야합니다.

## Deployment
지금까지 말한 것들 만으로도 충분히 서비스가 돌아가는 환경이 되었습니다.
하지만 Pod이 죽으면 재시작되고, 컨테이너가 다시 생기는동안 downtime이 생겨서 유저들이 매우 안좋아할겁니다.

위의 문제들을 해결하기 위해서는 distributed system의 장점을 살려야 하는데, 이를 위해서는 모든것을 replicate 해야합니다.
위에서 설명한 컴포넌트들을 다른 Node에 clone해서 띄워두고, 하나의 서비스로 연결한 뒤 DNS로 관리해서 Pod이 죽어도 서비스 전체가 죽지 않게 잘 해줘야 합니다.
이를 위해 Service는 LoadBalancer의 기능도 제공하고 있습니다.

Deployment를 통해 Pod의 배포 개수 등의 설정값들을 모아둔 blueprint를 만들 수 있습니다.
Deployment는 Pod을 추상화 한 것이라고 할 수 있습니다.

하지만 데이터베이스와 같이 한곳에서 관리되어야 하는 Pod는 replica를 통해 해결할 수 없습니다.
데이터베이스와 같은 애플리케이션의 Consistency를 위해 Shared Data Storage에 접근할 수 있는 매커니즘이 필요하게 되었는데, 이를 해결해주는 컴포넌트가 바로 StatefulSet입니다.

## StatefulSet
StatefulSet은 말그대로 Stateful한 애플리케이션을 위한 컴포넌트입니다.
읽기와 쓰기의 과정에서 Syncronize하게 해줍니다.
하지만 StatefulSet을 배포하는것이 간단한 것은 아니기에, k8s cluster 외부에 호스팅되어있는 데이터베이스를 사용하기도 합니다.

> Deployment for stateLESS Apps<br>
> StatefulSet for stateFUL Apps or Databases

이제 Deployment를 통해 한 서비스의 Pod이 여러개가 되었으니, 하나의 Node가 죽어버려도, 다른 Node가 처리해줄테니 안전합니다.

>avoid downtime!<br>
>avoid data inconsistencies!
