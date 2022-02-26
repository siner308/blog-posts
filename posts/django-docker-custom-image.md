---
title: "나만의 도커(Docker) 이미지를 만들어서 장고(Django) 서비스 배포하기"
subtitle: ""
tags:
    - python
    - django
    - docker
date: 2019-02-25
image: https://user-images.githubusercontent.com/34048253/155849594-90045484-a15e-4b3b-abf0-2ab382b9a518.png
---

저번 장에서는 `Django 공식 도커 이미지`를 확인해보았고, `Deprecated`되었음을 확인했습니다.<br>
이번 장에서는 `나만의 Django 이미지`를 직접 만들어서 배포해보고, `docker hub`에 업로드까지 해보겠습니다.<br>

본 포스팅의 프로젝트 `소스`는 `아래의 링크`를 통해 얻을 수 있습니다.<br>
[github.com/siner308/django-with-custom-image](https://github.com/siner308/django-with-custom-image)

![docker](https://user-images.githubusercontent.com/34048253/53354047-5a3ff100-3969-11e9-8404-a99701c0f35a.jpg)

- image from [cultivatehq.com](https://cultivatehq.com/posts/docker/)

---

# 0) Dockerfile

도커 이미지를 만들기 위해서는 `Dockerfile`이 필요합니다.
Dockerfile 작성의 힌트를 얻기 위해, Django의 공식 Dockerfile을 확인해봅니다.

>Official Dockerfile (Deprecated)

```dockerfile
FROM python:3.4

RUN apt-get update \
    && apt-get install -y --no-install-recommends \
        postgresql-client \
    && rm -rf /var/lib/apt/lists/*

WORKDIR /usr/src/app
COPY requirements.txt ./
RUN pip install -r requirements.txt
COPY 2019-02-25-django-docker-custom-image .

EXPOSE 8000
CMD ["python", "manage.py", "runserver", "0.0.0.0:8000"]
```

아래는 제가 만든 Dockerfile입니다.
위의 Deprecated 버전과는 다르게 `python3.6`을 기반으로 하여 `Django 2.0 이상`의 버전을 사용하도록 하였습니다.

>My Dockerfile

```dockerfile
FROM python:3.6

RUN apt-get update \
    && apt-get install -y --no-install-recommends \
       postgresql-client \
    && rm -rf /var/lib/apt/lists/*

COPY . /app  # 나의 Django 코드를 컨테이너에 복사합니다.
RUN pip install -r /app/requirements.txt  # requirements.txt에 적혀있는 pip 패키지들을 설치합니다.
RUN chmod 755 /app/start  # start 파일을 실행 가능하게 합니다.
WORKDIR /app  # 워킹디렉토리를 /app으로 합니다.
EXPOSE 8000  # 8000번 포트를 expose합니다.

ENTRYPOINT ["/app/start"]  # /app/start 파일을 실행시킵니다.
```

---

# 1) ENTRYPOINT 

위의 Dockerfile 맨 마지막 줄을 살펴보면, `ENTRYPOINT ["/app/start"]` 라는 명령어가 있습니다.<br>
컨테이너를 실행할 때 `/app/start`를 실행하라는 것을 의미합니다.

`ENTRYPOINT`는 Dockerfile 하나에 `한번밖에` 호출할 수 없는데, 파일을 사용함으로써 여러 명령어를 한번에 실행시킬 수 있게 합니다.

>start

```bash
#!/bin/bash

python manage.py makemigrations
python manage.py migrate

python manage.py runserver 0.0.0.0:8000
```

위와 같이 start 파일에는 `데이터베이스 마이그레이션`과 `배포` 명령어가 함께 들어있습니다.<br>
Django 명령어를 `하나의 파일`로 따로 관리함으로써, Dockerfile을 자주 변경하는 일이 줄어들게 됩니다.

---

# 2) docker build

[Usage - docker build](https://docs.docker.com/engine/reference/commandline/build/)
```bash
docker build [OPTIONS] PATH | URL | -
```

이제 Dockerfile과 start 파일을 중심으로 `나만의 Django 이미지`를 만들어보겠습니다.<br>
우선 Dockerfile이 있는 위치로 이동한 다음, 아래의 명령어를 입력합니다.

>docker build

```bash
docker build -t dockerdjango .
```

docker build 명령어를 입력한 후, `-t 옵션을 넣으면 생성될 이미지의 이름(태그)를 지정`할 수 있습니다.<br>
저의 경우에는 `이미지의 이름을 dockerdjango로` 지정하였습니다.

그 뒤에있는 `.` 는 `Dockerfile과 컨텐츠가 있는 경로`입니다.<br>
Dockerfile이 있는 위치에서 실행하기 때문에 `현재경로`인 `.` 을 입력해줍니다.

<script id="asciicast-VX34izwYhqKwOEzTzFXntgmgo" src="https://asciinema.org/a/VX34izwYhqKwOEzTzFXntgmgo.js" async></script>

`docker images`명령어를 이용해서 `dockerdjango:latest`라는 이름의 이미지와,<br>
Base 이미지인 `python:3.6`이 있는 것을 확인할 수 있습니다.

---

# 3) docker run

[Usage - docker run](https://docs.docker.com/engine/reference/run/)
```bash
docker run [OPTIONS] IMAGE[:TAG|@DIGEST] [COMMAND] [ARG...]
```

방금 생성한 `이미지를 배포`해보도록 하겠습니다.<br>
8000번 포트를 열어 주었기 때문에, `-p 8000:8000` 옵션을 넣어주겠습니다.

>docker run

```bash
docker run -d -p 8000:8000 --name dockerdjango dockerdjango:latest
```

| OPTION | DESCRIPTION |
| ------ | ------ |
| `-d` | 백그라운드 실행 |
| `-p <host port>:<container port>` | 호스트와 컨테이너의 포트를 매핑 |
| `--name <container name>` | 컨테이너의 이름을 지정 |
| `<image name>:<image tag>` | 이미지의 이름과 버전을 지정 |

<script id="asciicast-PHZPC1hTMJut3AJiK8Ye8OvcI" src="https://asciinema.org/a/PHZPC1hTMJut3AJiK8Ye8OvcI.js" async></script>

---

# 4) docker hub에 업로드

[Usage - docker login](https://docs.docker.com/engine/reference/commandline/login/)
```bash
docker login [OPTIONS] [SERVER]
```

[Usage - docker tag](https://docs.docker.com/engine/reference/commandline/tag/)
```bash
docker tag SOURCE_IMAGE[:TAG] TARGET_IMAGE[:TAG]
```

[Usage - docker push](https://docs.docker.com/engine/reference/commandline/push/)
```bash
docker push [OPTIONS] NAME[:TAG]
```

이제 위의 `login`, `tag`, `push` 명령어를 활용하여 나만의 이미지를 다른 곳에서도 사용할 수 있게 `docker hub에 업로드`해 보도록 하겠습니다.

>docker login, tag, push

```bash
docker login
docker tag dockerdjango:latest aan308/dockerdjango:latest
docker push aan308/dockerdjango:latest
```

<script id="asciicast-hQtMYcSk9EZ5hIYQSbssXh8pq" src="https://asciinema.org/a/hQtMYcSk9EZ5hIYQSbssXh8pq.js" async></script>

![image](https://user-images.githubusercontent.com/34048253/53351892-32e72500-3965-11e9-9487-8193219a0dc1.png)

성공적으로 업로드 됐습니다.

---

# 5) docker hub 이미지로 run 하기

docker hub에 공개적으로 업로드 된 이미지는 어디서든 pull 받아서 사용할 수 있습니다.<br>
docker run 커맨드 시, local 저장소에 해당 image가 없다면, `자동으로 pull`을 실행합니다.

>docker run with remote image

```bash
docker run -d -p 8000:8000 --name dockerdjango aan308/dockerdjango:latest
```

<script id="asciicast-OlzBqtf4VPdw87Q0Md28Kfcsd" src="https://asciinema.org/a/OlzBqtf4VPdw87Q0Md28Kfcsd.js" async></script>

---

# 6) Result

![image](https://user-images.githubusercontent.com/34048253/52423832-3b092d00-2b3c-11e9-8e67-24ddc3a7d0e1.png)

배포에 성공하였습니다.

---

다음편에서는 `docker-compose`를 사용하여 `PostgreSQL`과 `Django`를 함께 배포하는 방법에 대해 적어보겠습니다.
