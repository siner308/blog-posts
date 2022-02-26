---
title: "도커와 컨테이너 생태계의 변화과정"
subtitle: "의식의 흐름대로 알아보는 도커와 쿠버네티스의 세부구조"
tags:
    - docker
    - kubernetes
    - container
date: 2021-10-23
image: https://user-images.githubusercontent.com/34048253/138473541-ca9eef72-8d1f-4a24-8115-fe96b01f1eb0.png
---

# container 생태계의 변화

2013년 Docker가 처음으로 세상에 나타난 이후 IT업계는 정말 크게 변했습니다. 어플리케이션의 배포단위는 이제 war, jar, zip 등이 아니라 Docker 이미지가 되었고 Docker를 사용할 수 있는 환경이기만 하면 어플리케이션은 Windows에서든, Ubuntu에서든 동일하게 동작하였습니다. Docker의 이러한 특성을 이용해 수백 수천대의 서버를 운영하는 환경에서 Docker를 도입하는 사례들이 늘어나기 시작했습니다.

하지만 포맷과 런타임에 대한 특정한 규격이 없다 보니 컨테이너의 미래는 불안했습니다. 도커가 사실상의 컨테이너 표준 역할을 했지만 코어OS(CoreOS)는 도커와는 다른 규격으로 표준화를 추진하려 했습니다.

이때 Docker Engine에는 API, CLI, 네트워크, 스토리지 등 많은 기능들이 하나의 패키지에 담겨있었고, Docker 측에서는 Monolithic한 Docker Engine의 구조를 나누는 작업을 시작했습니다.

![image](https://user-images.githubusercontent.com/34048253/138473541-ca9eef72-8d1f-4a24-8115-fe96b01f1eb0.png)

# OCI

이러한 문제를 해결하기 위해 2015년 6월 도커, 코어OS, AWS, 구글, 마이크로소프트, IBM 등 주요 플랫폼 벤더들은 애플리케이션의 이식성(Portability) 관점에서 컨테이너 포맷과 런타임에 대한 개방형 업계 표준을 만들기 위해 OCI(Open Container Initiative)를 구성하였습니다. 이후 컨테이너 시장은 OCI의 런타임 명세와 이미지 명세를 준수하는 방향으로 성장하였고 그 과정에서 2016년 12월 쿠버네티스(Kubernetes)의 컨테이너 런타임을 만들기 위한 CRI(Container Runtime Interface)가 등장했습니다.

- OCI github : [https://github.com/opencontainers](https://github.com/opencontainers)
- image-spec : [https://github.com/opencontainers/image-spec/blob/main/spec.md](https://github.com/opencontainers/image-spec/blob/main/spec.md)
- runtime-spec : [https://github.com/opencontainers/runtime-spec/blob/master/spec.md](https://github.com/opencontainers/runtime-spec/blob/master/spec.md)
- distribution-spec : [https://github.com/opencontainers/distribution-spec/blob/main/spec.md](https://github.com/opencontainers/distribution-spec/blob/main/spec.md)

![image](https://user-images.githubusercontent.com/34048253/138473587-b56cf1cf-df48-4ef2-9483-bb662f0a5918.png)

# container runtime

CRI의 등장 배경을 이해하려면 먼저 컨테이너 런타임에 대해 살펴봐야합니다. 컨테이너를 실행하기 위해서는 다음과 같은 세 단계를 거칩니다.

![image](https://user-images.githubusercontent.com/34048253/138473632-12144664-d834-49c4-bded-51d2e2b1f35f.png)

OCI가 만들어질 당시 비공식적 표준 역할을 하던 도커는 컨테이너 런타임의 표준화를 위해 필요한 모든 단계가 아닌 세 번째 단계인 컨테이너의 실행 부분만 표준화하였습니다. 이로 인해 컨테이너의 런타임은 실제 컨테이너를 실행하는 저수준 컨테이너 런타임인 OCI 런타임과 컨테이너 이미지의 전송 및 관리, 이미지 압축 풀기 등을 실행하는 고수준 컨테이너 런타임으로 나뉘게 되었습니다.

## 저수준 컨테이너 런타임 (Low-Level Container Runtime)

컨테이너는 Linux namespace와 cgroup(control group)을 사용하여 구현합니다. namespace는 각 컨테이너에 대해 파일 시스템이나 네트워킹과 같은 시스템 리소스를 가상화하고 cgroup은 각 컨테이너가 사용할 수 있는 CPU 및 메모리와 같은 리소스 양을 제한하는 역할을 합니다. 저수준 컨테이너 런타임은 이러한 namespace와 cgroup을 설정한 다음 해당 namespace 및 cgroup 내에서 명령을 실행합니다.

![image](https://user-images.githubusercontent.com/34048253/138473662-c26eaa10-2580-4794-890c-a72b496df014.png)

### runC

container와 같은 격리 환경을 제공하기 위해 내부적으로 cgroup과 namespace 등의 linux kernel 기술이 사용됩니다. 그런데 이러한 cgroup과 namespace를 다루는 방법은 시스템마다 다르고, 커널 버전마다 다르기 때문에 LXC(Linux Container)나 libvirt같은 중간 매개채를 통해 간접적으로 관리했습니다.

kernel 관련 가상화 기술들이 docker의 발전에 꼭 필요한 요소인데, 이를 다루는 인터페이스가 외부 솔루션에 의존적이다 보니 어느 순간부터 kernel의 가상화 기술을 다루기 위한 interface를 자체적으로 개발, 관리해야 한다는 필요성이 생겼는데, 이 때문에 개발된것이 libcontainer이고, 이것이 docker v1.11이후 리팩토링 과정을 거쳐 지금의 runC가 되었습니다. 

OCI를 준수하는 저수준 컨테이너 런타임으로 가장 잘 알려진 것은 runC 입니다. runC는 원래 도커에서 컨테이너를 실행하기 위해 개발되었으나, OCI 런타임 표준을 위해 독립적인 라이브러리로 사용되었습니다. 저수준 컨테이너 런타임은 컨테이너를 실제 실행하는 역할을 하지만 이미지로부터 컨테이너를 실행하려면 이미지와 관련된 APi같은 기능이 필요합니다. 이러한 기능은 고수준 컨테이너 런타임에서 제공됩니다.

[GitHub - opencontainers/runc: CLI tool for spawning and running containers according to the OCI specification](https://github.com/opencontainers/runc)

## 고수준 컨테이너 런타임 (High-Level Container Runtime)

고수준 컨테이너 런타임은 원격 어플리케이션이 컨테이너를 실행하고 모니터링 하는 데 사용할 수 있는 데몬 및 API를 제공합니다. 또한 컨테이너를 실행하기 위해 저수준 런타임 위해 배치됩니다.

이처럼 컨테이너를 실행하려면 저수준 및 고수준 컨테이너 런타임이 필요하기 때문에 OCI 런타임과 함께 도커가 그 역할을 했습니다. 도커는 containerd라는 가장 잘 알려진 고수준 컨테이너 런타임을 제공합니다. containerd도 runC와 마찬가지로 도커에서 컨테이너를 실행하기 위해 개발되었으나, 나중에 독립적인 라이브러리로 추출되었습니다.

<div style="display:flex">
  <img src="https://user-images.githubusercontent.com/34048253/138473714-f4ad6361-99d6-4d02-9cfa-9fefeff89d07.png" width="33%" />
  <img src="https://user-images.githubusercontent.com/34048253/138473740-d6581dda-9a95-4bef-b211-d41f540acaf6.png" width="66%" />
</div>

containerd와 CRI-O 이 두가지 Container Runtime이 현재 가장 널리 사용되고 있으며 containerd는 Docker Engine에 기본으로 탑재되어 있어서 지금도 Docker를 사용한다면 내부적으로 사용되는 Container Runtime은 containerd 를 사용하게 됩니다. 참고로 `docker build` 커맨드로 생성되는 이미지들 역시 OCI Image Spec을 준수하기 때문에 별도의 작업없이 containerd로 실행시킬 수 있습니다.

### containerd

Docker에서 만든 Container Runtime이 바로 [containerd](https://containerd.io/) 입니다.

### CRI-O

한편, Red Hat, Intel, SUSE, Hyper, IBM 쪽에서도 OCI 표준에 따라 Kubernetes 전용 Container Runtime을 만들었는데 이것이 CRI-O 입니다.

로컬 쿠버네티스인 minikube도 CRI-O를 사용하고 있습니다.

[cri-o](https://cri-o.io/)

![image](https://user-images.githubusercontent.com/34048253/138473886-10dfc957-a307-43e9-ab05-7979c42fb55e.png)

## 런타임 분리 후 퍼포먼스 차이

이렇게 Container Runtime이 Monolithic 아키텍처에서 분리되어 나오면서 Java 어플리케이션을 배포할 때 JDK보다 가벼운 JRE를 사용하는 것처럼 (물론 이 경우에는 보안과 같은 다른 이유도 있습니다만) 컨테이너를 실행할 때 무거운 Docker Engine이 아닌 containerd나 CRI-O와 같은 가벼운 Container Runtime을 사용하게 되면서 다음과 같은 장점도 나타나게 되었습니다.

<div style="display:flex">
  <img src="https://user-images.githubusercontent.com/34048253/138473928-a10b088f-626f-4e5a-a569-5012007460c4.png" width="33%" />
  <img src="https://user-images.githubusercontent.com/34048253/138473953-30150760-420d-4c08-8daa-626639ff0dca.png" width="33%" />
  <img src="https://user-images.githubusercontent.com/34048253/138473976-4f5f6a10-bbc5-421e-9503-91d425c2b460.png" width="33%" />
</div>

쉽게 말해 containerd로의 전환을 통해 Pod은 더 빨리 시작되고, CPU와 메모리의 사용량은 더 줄었다는 이야기로, containerd로의 전환이 왜 일어나고 있는지를 잘 설명해주고 있습니다.

하지만, containerd를 사용하는 방법은 분명히 Docker에 비해 까다롭습니다. 개발자 입장에서는 예전과 같이 docker build만 실행하면 되기 때문에 크게 달라지는 것이 없다고는 하나, Kubernetes 클러스터를 운영하는 입장에서는 Docker로 한번에 모든 것을 제어할 수 있었던 예전과는 달리, 이제는 runc, containerd, ctr 등 낯선 라이브러리들과 익숙해져야 하는 부담이 생겼습니다.

물론 이 부담은 containerd로의 전환을 망설일만한 부담은 아닐 것입니다 게다가 이제 2021년 하반기 출시예정인 kubernetes 1.23과 함께 Container Runtime으로써의 Docker의 지원이 중단되는 것이 결정된만큼 새로운 흐름을 빨리 받아들이고 혜택을 누리는 것이 필요한 것 같습니다.

![image](https://user-images.githubusercontent.com/34048253/138474107-68b2937a-2c31-40be-ab26-5d45b6154816.png)

# docker binaries

컨테이너 런타임에 대해 아실제로 도커가 어떻게 구현되는지 살펴봅시다.

ubuntu20.04, docker 20.10.8 기준 /usr/bin에 설치되는 도커 관련 파일은 아래와 같습니다.

![image](https://user-images.githubusercontent.com/34048253/138474138-e0f7ce65-f0e7-4c19-8b37-ec7e07771d49.png)

## [dockerd](https://docs.docker.com/engine/reference/commandline/dockerd/)

> dockerd는 컨테이너를 관리하는 영구 프로세스입니다. Docker는 데몬과 클라이언트에 대해 서로 다른 바이너리를 사용합니다. 데몬을 실행하려면 dockerd를 입력하면 됩니다.
> 

volume, image, networking 관리 뿐 아니라 orchestration 까지 처리합니다. client로부터 rest api 형식의 요청을 수신하여 처리합니다.

client와 server(docker daemon)간 통신방식은 기본적으로 unix domain socket(IPC socket)을 사용하며, 이외에도 fd 또는 tcp를 사용할 수 있습니다.

![image](https://user-images.githubusercontent.com/34048253/138474167-9d96ff37-8bb3-454e-b632-0be63885d0dc.png)

## [containerd](https://containerd.io/)

container의 lifecycle을 관리합니다. client로부터의 container 관리 관련 요청은 dockerd를 거쳐 gRPC 통신을 통해 containerd로 전달됩니다. 

![image](https://user-images.githubusercontent.com/34048253/138474216-68b78728-a426-4f11-a5b3-9d1f631f2a29.png)

![image](https://user-images.githubusercontent.com/34048253/138474259-97597e24-a400-4ec6-83d1-9fe5725829b2.png)

그림출처: [Docker for Front-End Developers](http://cloudrain21.com/examination-of-docker-process-binary)

## containerd-shim-runc-v2

실제 동작중인 컨테이너입니다.

![image](https://user-images.githubusercontent.com/34048253/138474296-29111590-2a6d-4c88-9007-b68f41fc6b84.png)

## [docker-init](https://docs.docker.com/engine/reference/run/#specify-an-init-process)

> —init 플래그를 사용하여 컨테이너 내부의 1번 프로세스(PID=1)를 init process로 지정할 수 있습니다.
이를 통해 init system에게 좀비프로세스 처리 등의 책임을 부여합니다.
> 

![image](https://user-images.githubusercontent.com/34048253/138474336-24d3c3ac-6f98-466c-91e6-b6b5c38a5ed8.png)

init옵션을 부여한 컨테이너와 그렇지 않은 컨테이너의 비교

## docker-proxy

컨테이너 외부 포트로 들어오는 요청을 내부포트로 전달해주는 역할을 수행합니다.

ex) -p 8000:80 과 같이 옵션을 주면 host의 8000port를 컨테이너의 80포트로 전달해줍니다.

그리고 이러한 포트변환을 수행해주는게 docker-proxy입니다.

![image](https://user-images.githubusercontent.com/34048253/138474358-6612c78f-688c-4666-b6e1-37bba339b61c.png)

docker host 내부의 container간 통신은 docker0라는 bridge를 통해 기본적으로 가능합니다.

![image](https://user-images.githubusercontent.com/34048253/138474400-5d6a85da-b02e-4b69-b657-be33b0323622.png)

docker-proxy 를 통한 port forwarding (그림출처 : [https://blogs.itemis.com](https://blogs.itemis.com/))

# 쿠버네티스 구조의 변화

## 도커만 사용하던 시절

초기의 도커기반의 쿠버네티스는 아래와 같이 kubelet이 명령을 받으면 docker runtime을 통해서 컨테이너를 생성하거나 삭제하는 것과 같은 생명주기를 관리하는 구조를 가지고 있었습니다. 이로인해 Docker의 새로운 버전이 나올 때마다 크게 영향을 받았습니다. 

kubelet: 쿠버네티스의 각 노드마다 있는 통신용 어플리케이션

![image](https://user-images.githubusercontent.com/34048253/138474443-a201f23a-ec87-4ed5-9d8b-6e9cf555b5d1.png)

## 다양한 컨테이너 기술의 등장

도커 이외의 다양한 컨테이너 기술이 나오면서 다양한 컨테이너 런타임을 지원하기 위해서 그때마다 kubelet의 코드를 수정해야 하는 문제가 생겼습니다.

![image](https://user-images.githubusercontent.com/34048253/138474475-f50b5906-7643-45f3-ae4c-e68613f4aa5b.png)

## CRI 스펙의 등장

kubelet의 코드 변화 없이 새로운 컨테이너 런타임을 플러그인 구조로 추가할 수 있는 구조로 짜기 위해 컨테이너 런타임을 CRI 스펙에 맞게 구현하기 시작했습니다.

docker의 경우엔 docker shim이라는 CRI 인터페이스를 준수하는 구현체를 제공하고있고, rkt의 경우엔 rktlet이라는 이름의 CRI 구현체를 제공했습니다.

하지만 지원되는 컨테이너의 종류가 늘어갈수록 CRI를 다시 구현해야 했기 때문에, 컨테이너 런타임 자체를 표준화 하고자 했고, 그로 인해 정해진 스펙이 OCI입니다.

![image](https://user-images.githubusercontent.com/34048253/138474542-ab72f5a9-ae4e-4ba3-bfda-883c3ea6ea4f.png)

## 쿠버네티스, 도커 지원 중단

쿠버네티스는 CRI로 컨테이너 런타임과 통신하는데 도커는 해당 인터페이스를 지원하지 않아 Dockershim이라는 추가 레이어를 통해 연동하였습니다.

또한 도커의 기능 중 쿠버네티스가 사용하는 부분은 컨테이너 런타임 정도 입니다.

- ~~CLI, API~~ → Kubernetes CLI
- ~~Volume~~ → Kubernetes Volume
- ~~Network~~ → Kubernetes Network

결국 쿠버네티스 v1.20 이후의 버전에서는 Dockershim이 deprecated 되고 맙니다.

![image](https://user-images.githubusercontent.com/34048253/138474565-2eb9d949-1b81-41db-80cc-be58ade9548a.png)

# 한눈에 보는 컨테이너 생태계 변천사

![image](https://user-images.githubusercontent.com/34048253/138474593-a0817037-2e58-43f9-9904-56d9941f3785.png)

![image](https://user-images.githubusercontent.com/34048253/138474634-582500a1-721f-4b1a-82c0-bb85a6d0c647.png)

# references

[흔들리는 도커(Docker)의 위상 - OCI와 CRI 중심으로 재편되는 컨테이너 생태계](https://www.samsungsds.com/kr/insights/docker.html)

[쿠버네티스 CRI (Container Runtime Interface) & OCI (Open container initiative)](https://bcho.tistory.com/1353)

[The differences between Docker, containerd, CRI-O and runc](https://www.tutorialworks.com/difference-docker-containerd-runc-crio-oci/)

[containerd는 무엇이고 왜 중요할까?](https://www.linkedin.com/pulse/containerd%EB%8A%94-%EB%AC%B4%EC%97%87%EC%9D%B4%EA%B3%A0-%EC%99%9C-%EC%A4%91%EC%9A%94%ED%95%A0%EA%B9%8C-sean-lee/?originalSubdomain=kr)

[도커 컨테이너 까보기(3) - Docker Process, Binary - Rain.i](http://cloudrain21.com/examination-of-docker-process-binary)

[컨테이너 기술:: Docker와 Podman](https://naleejang.tistory.com/227)

[Container Runtime | Better Tomorrow with Computer Science](https://insujang.github.io/2019-10-31/container-runtime/)

[Docker(container)의 작동 원리: namespaces and cgroups](https://tech.ssut.me/what-even-is-a-container/)
