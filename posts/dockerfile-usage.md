---
layout: post
title:  "[번역] Dockerfile 레퍼런스 (1) - Usage"
subtitle: "도커 데몬의 작동방법, 이미지 빌드 & 태깅, 캐시이미지 가져오기"
author: "Siner"
header-img: "img/post_headers/dockerfile-commands.png"
catalog: true
header-mask:  0.3
tags:
    - docker
    - architecture
date:   2020-03-29
multilingual: false
---

[공식 레퍼런스](https://docs.docker.com/engine/reference/builder/)를 토대로 작성되었습니다.

---

>도커는 `Dockerfile`을 읽어서 자동으로 이미지를 빌드할 수 있습니다.
>`Dockerfile`은 이미지를 만들기 위해 사용되는 모든 명령어를 담고있는 문서입니다.
>이러한 명령어들은 실제로 유저가 CLI환경에서 한줄씩 입력하여 사용할 수 있는 명령어들입니다.
>`docker build`를 사용하여 여러 줄의 명령어로 이루어진 빌드 과정을 자동으로 진행할 수 있습니다.

이 페이지를 다 읽은 후에, [Dockerfile Best Practices](https://docs.docker.com/engine/userguide/eng-image/dockerfile_best-practices/)를 읽어보는 것을 추천드립니다.

# Usage
[docker build](https://docs.docker.com/engine/reference/commandline/build/) 명령어는 `Dockerfile`과 *context*로부터 정보를 가져와서 이미지를 빌드합니다. context는 파일들이 위치한 위치를 가리키며, `PATH` 또는 `URL`의 형태를 지닙니다.
`PATH`는 로컬 파일시스템 내의 폴더이고, `URL`은 Git 저장소 경로가 됩니다.

context 내용을 불러오는 방식은 재귀적으로 진행됩니다. 그러므로, `PATH`는 모든 하위 폴더를 포함하고, `URL`은 해당 저장소의 모든 하위 모듈를 포함합니다.
현재 폴더를 context로 하는 예시는 다음과 같습니다.
```bash
$ docker build .
Sending build context to Docker daemon 6.51 MB
...
``` 

빌드의 첫번째 과정에서, context의 모든 내용이 Docker daemon으로 전송됩니다. (빌드 과정은 CLI가 아닌, 데몬을 통해 진행됩니다.)
도커 빌드를 처음으로 시작하려 한다면, context를 현재 비어있는 폴더로 두고 Dockerfile을 그 폴더 안에 넣어두는 것을 추천합니다.

context에는 `Dockerfile`을 빌드 하는데 필요한 파일들만 넣는 것을 추천해드립니다.

> *Warning*: `PATH`를 루트 경로(`/`)로 잡지 마세요, 하드 드라이브 전체가 도커 데몬으로 전송됩니다.

context 내부 파일들을 사용할때, `Dockerfile`에서 `COPY`와 같은 명령어를 통해 해당 파일의 경로를 지정해줍니다. 

빌드 퍼포먼스를 향상시키기 위해, `.dockerignore` 파일을 작성하여 context 폴더 내부의 불필요한 파일이나 폴더를 제외시킬 수 있습니다.
이 문서의 하단에서 `.dockerignore`작성에 대한 정보  [다음 링크](https://docs.docker.com/engine/reference/builder/#dockerignore-file)를 클릭하시면 관련 정보를 얻을 수 있습니다. 

기본적으로, context기준 루트경로의 Dockerfile을 사용합니다. `docker build`명령어에 `-f` 옵션을 사용하면, Dockerfile이 파일시스템 어디에 있던지 지정해줄 수 있습니다.  

```bash
$ docker build -f /path/to/a/Dockerfile . 
```

이미지 빌드를 완료하였을때, 특정 도커 저장소와 태그 정보를 담을 수 있게 할 수 있습니다.

```bash
$ docker build -t shykes/myapp .
```

여러개의 저장소와 태그 정보를 지정할 수도 있습니다.
```bash
$ docker build -t shykes/myapp:1.0.2 -t shykes/myapp:latest .
```

도커 데몬이 `Dockerfile`의 명령어를 실행하기 전에, `Dockerfile`의 validation을 실행하고, 문법 오류가 있다면, 에러를 리턴합니다.
```bash
$ docker build -t test/myapp .
Sending build context to Docker daemon 2.048 kB
Error response from daemon: Unknown instruction: RUNCMD
```

도커 데몬은 `Dockerfile`의 명령어를 하나씩 실행하고, 새로운 이미지 빌드를 완료하기 전에, 각 실행 결과를 필요에 따라 새로운 이미지에 커밋합니다. 도커 데몬은 전송된 context를 자동으로 정리합니다.

각 명령어들은 독립적으로 실행되며, 실행 결과는 각각의 새로운 이미지로 존재하게 됩니다. 그러므로, `RUN cd /tmp`와 같은 명령어는, 다음에 진행될 명령어에 어떠한 영향도 주지 못합니다.

도커는 빌드 프로세스를 가속시키기 위해, 이전 이미지 빌드단계에서 생성된 이미지(cache)들을 가능한 한 재활용합니다. 이러한 캐시 사용은 `Using cache`라는 메시지로 콘솔에 나타납니다. 
(더 많은 정보는, [`Dockerfile` best practices guide](https://docs.docker.com/engine/userguide/eng-image/dockerfile_best-practices/)에 적혀있습니다.)

```bash
$ docker build -t svendowideit/ambassador .
Sending build context to Docker daemon 15.36 kB
Step 1/4 : FROM alpine:3.2
 ---> 31f630c65071
Step 2/4 : MAINTAINER SvenDowideit@home.org.au
 ---> Using cache
 ---> 2a1c91448f5f
Step 3/4 : RUN apk update && apk add socat && rm -r /var/cache/
 ---> Using cache
 ---> 21ed6e7fbb73
Step 4/4 : CMD env | grep _TCP= | (sed 's/.*_PORT_\([0-9]*\)_TCP=tcp:\/\/\(.*\):\(.*\)/socat -t 100000000 TCP4-LISTEN:\1,fork,reuseaddr TCP4:\2:\3 \&/' && echo wait) | sh
 ---> Using cache
 ---> 7ea8aef582cc
Successfully built 7ea8aef582cc
```

빌드 캐시는 상위 실행결과에 변동이 없는 경우(local parent chain이 존재하는 경우)에만 사용됩니다. 처음 명령어를 수정했다면, 아래 명령어의 이전 빌드에서 생성되었던 모든 캐시들은 사용이 불가능합니다.
빌드 과정에서 이전 빌드 또는 전체 이미지 체인을 불러오는 과정에서 `docker load`를 사용합니다. 만약 특정 이미지의 캐시를 사용하고 싶다면, `--cache-from` 옵션을 사용할 수 있습니다. `--cache-from` 옵션과 함께 사용된 이미지는 parent chain이 필요없고, 다른 저장소에서 가져와서 사용할 수도 있습니다.

빌드 과정을 마쳤다면, [Pushing a repository to its registry](https://docs.docker.com/engine/tutorials/dockerrepos/#/contributing-to-docker-hub)를 한번 살펴보시길 바랍니다.
