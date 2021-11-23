---
layout: post
title:  "포스트그레스(PostgreSQL)로 설명하는 도커 컴포즈(Docker Compose) 초간단 사용법"
subtitle: ""
author: "Siner"
header-img: "img/post_headers/2019-03-02-postgresql-docker-compose.png"
catalog: true
header-mask:  0.3
tags:
    - docker
    - database
date:   2019-03-02
multilingual: false
---

이번 장에서는 `Docker Compose`의 일반적인 사용방법과, 이를 사용하여 PostgreSQL을 배포하는 방법에 대해서 설명하겠습니다.

---

# 0) Compose란?

[Overview of Docker Compose](https://docs.docker.com/compose/overview/)
>`Compose`란 여러개의 도커 컨테이너들을 한꺼번에 관리(빌드, 배포 등) 할 수 있는 Tool입니다.<br>
Compose를 사용하면 `YAML 파일`을 사용하여 응용 프로그램의 서비스를 구성 할 수 있습니다.<br>
YAML 파일을 작성하면 `단 하나의 명령어`만으로 `모든 서비스`을 시작할 수 있습니다.<br>

>
1. `Dockerfile`을 작성하세요. (없어도 무방합니다.)
2. `docker-compose.yml`파일을 작성하여 모든 서비스가 한번에 배포되도록 하세요.
3. `docker-compose up`명령어를 사용하여 모든 서비스를 한번에 배포하세요.

추가 설명을 붙이자면, `docker run ...` 에 들어가는 수많은 옵션들을 하나의 파일로 관리할 수 있다는 장점도 있습니다.<br>
(저는 이 때문에 사용하는게 더 큽니다.)

---

# 1) 설치

docker없이, docker-compose만 설치하게 되면 동작하지 않습니다.

```bash
sudo apt install -y docker.io docker-compose
```

---

# 2) docker-compose.yml

```yaml
version: '2'
services:
  postgres:
    image: postgres:10
    container_name: postgres
    environment:
      - POSTGRES_PASSWORD=password
    ports:
      - '5432:5432'
    volumes:
      - /mnt/postgres/data:/var/lib/postgresql/data
  pgadmin:
    image: dpage/pgadmin4
    container_name: pg_admin
    environment:
      - PGADMIN_DEFAULT_EMAIL=siner@pgadmin.com
      - PGADMIN_DEFAULT_PASSWORD=password
    ports:
      - '5555:80'
    volumes:
      - /mnt/pgadmin/data:/var/lib/pgadmin
```

---

# 3) docker-compose up

작성한 YAML 파일을 배포합니다.
배포는 아래와 같이 서비스를 골라서 배포할 수도 있고, 전부를 배포할 수도 있습니다.
```bash
docker-compose up  # 파일 내 모든 서비스 배포
docker-compose up postgres  # postgres만 배포
docker-compose up pgadmin  # pgadmin만 배포
docker-compose up postgres pgadmin  # postgres, pgadmin 배포
```

<script id="asciicast-avMMMaWlI7T93vAnXsg7XQgqS" src="https://asciinema.org/a/avMMMaWlI7T93vAnXsg7XQgqS.js" async></script>

`-d` 옵션을 사용하면 백그라운드로 실행이 가능합니다.
```bash
docker-compose up -d
docker-compose up -d postgres
docker-compose up -d pgadmin
docker-compose up -d postgres pgadmin
```

<script id="asciicast-GeYpgwoYQyWHQl3eKd1j4Bgob" src="https://asciinema.org/a/GeYpgwoYQyWHQl3eKd1j4Bgob.js" async></script>

---

# 4) docker-compose down

docker-compose로 배포한 서비스들을 중지시킬 수 있습니다.

<script id="asciicast-eIq4bdDIHveF3qVFgsDKJ009M" src="https://asciinema.org/a/eIq4bdDIHveF3qVFgsDKJ009M.js" async></script>

---

다음 장에서는 `docker-compose build` 명령어를 활용하여 여러 이미지를 한번에 빌드하고 배포하는 방법에 대해 살펴보겠습니다.
