---
layout: post
title:  "장고(Django) 공식 이미지로 설명하는 도커(Docker)의 기본적인 사용법"
subtitle: ""
author: "Siner"
header-img: "img/post_headers/2019-02-16-django-docker-hub.png"
catalog: true
header-mask:  0.3
tags:
    - Python
    - Django
    - Docker
date:   2019-02-16
multilingual: false
---

이번 장에서는 `Django 공식 이미지`를 사용하여 `Docker의 기본 개념`에 대해 설명하겠습니다.<br>
Custom 이미지를 생성하여 Django를 배포하는 방법에 대해서는 다음 장에서 다루겠습니다.

---

# 0) Docker

[Docker란 무엇입니까?](https://aws.amazon.com/ko/docker/)
>Docker는 애플리케이션을 신속하게 구축, 테스트 및 배포할 수 있는 소프트웨어 플랫폼입니다. Docker는 소프트웨어를 `컨테이너`라는 표준화된 유닛으로 패키징하며, 이 컨테이너에는 라이브러리, 시스템 도구, 코드, 런타임 등 소프트웨어를 실행하는 데 필요한 모든 것이 포함되어 있습니다. Docker를 사용하면 환경에 구애받지 않고 애플리케이션을 신속하게 배포 및 확장할 수 있으며 코드가 문제없이 실행될 것임을 확신할 수 있습니다.

---

# 1) Docker Hub
Docker Hub에서는 컨테이너 이미지를 받아서 사용하고, 자신만의 이미지를 만들어서 업로드 할 수도 있습니다.<br>
프로젝트에 사용할 Django 이미지를 얻기 위해 [`hub.docker.com`](https://hub.docker.com/_/django)에 접속하여 Django를 검색해보면 Docker에서 공식적으로 지원하는 Django 이미지를 확인할 수 있습니다.
![image](https://user-images.githubusercontent.com/34048253/52415147-94b42c00-2b29-11e9-90ca-bd8a777ff351.png)

아래의 Description 내용을 살펴보면 Django 이미지가 2016년 말에 `Deprecated`되었음을 확인할 수 있습니다.
![image](https://user-images.githubusercontent.com/34048253/52911229-5c101180-32e4-11e9-898c-0f34d4470633.png)

`Deprecated`되었다는 것은, 직접 Django 이미지를 만들지 않는 이상은 Docker에서는 새로운 버전의 Python과 Django를 사용할 수 없게 되었다는 것을 의미합니다.<br>
일단 이번 포스팅에서는 기존의 Django 이미지를 사용해서 배포함으로써, Docker의 기본적인 사용방법에 대해 다뤄보겠습니다.

---

# 2) Dockerfile

이미지를 불러와서 `run`하기 전에, 이미지가 어떻게 구성되어있는지 `Dockerfile`을 한번 살펴볼 필요가 있습니다.

[Dockerfile reference](https://docs.docker.com/engine/reference/builder/)
>Docker는 `Dockerfile`을 읽어 자동으로 이미지를 작성할 수 있습니다. `Dockerfile`은 이미지를 어셈블하기 위해 사용자가 명령 줄에서 호출 할 수있는 모든 명령을 포함하는 텍스트 문서입니다.<br>
`docker build`를 사용하면 몇 가지 명령을 연속적으로 실행하는 자동화 된 빌드를 만들 수 있습니다.

아래 내용은 Django 공식 이미지의 Dockerfile 입니다.<br>
마지막 CMD 명령어에서 `python manage.py runserver 0.0.0.0:8000`명령어를 사용하는 것을 확인할 수 있습니다.

```dockerfile
# Dockerfile

FROM python:3.4

RUN apt-get update \
    && apt-get install -y --no-install-recommends \
        postgresql-client \
    && rm -rf /var/lib/apt/lists/*

WORKDIR /usr/src/app
COPY requirements.txt ./
RUN pip install -r requirements.txt
COPY 2019-02-16-django-docker-hub .

EXPOSE 8000
CMD ["python", "manage.py", "runserver", "0.0.0.0:8000"]
```

# 3) Dockerfile 없이 Django를 띄워보자

Django 공식 이미지를 가져와서 빌드 해보도록 합시다.

<script id="asciicast-XlAQaw2BxeUo6JeYqKQsfoFMc" src="https://asciinema.org/a/XlAQaw2BxeUo6JeYqKQsfoFMc.js" async></script>

```bash
docker run --name my_django_container -v "$PWD":/usr/src/app -w /usr/src/app -d -p 8000:8000 django bash -c "pip install django && django-admin startproject mydjangoproject && cd mydjangoproject && python manage.py runserver 0.0.0.0:8000"
```

위의 명령어는 다음과 같은 뜻을 지닙니다.

- `docker run` : Docker 이미지를 실행합니다. (local에 이미지가 없으면 pull 하고 실행.)
- `--name my_django_container` : 컨테이너 이름은 **my_django_container** 으로 합니다.
- `-v "$PWD":/usr/src/app` : 호스트의 **$PWD(현재 경로)**와 컨테이너의 **/usr/src/app**을 연결해줍니다.
- `-w /usr/src/app` : 컨테이너 내부의 **워킹 디렉토리**를 /usr/src/app 으로 설정합니다.
- `-d` : 해당 명령어를 백그라운드로 실행시키고, 컨테이너 ID를 리턴합니다.
- `-p 8000:8000` : 호스트의 **8000번 포트(앞)**와 컨테이너의 **8000번 포트(뒤)**를 연결해줍니다.
- `django` : 타겟 이미지를 django:latest로 합니다. (뒤에 태그를 **생략하면 latest**를 가져와서 실행함)
- `bash -c` : 뒤의 명령어를 컨테이너 안에서 실행시킵니다.

# 4) ALLOWED_HOSTS

![image](https://user-images.githubusercontent.com/34048253/52415825-70f1e580-2b2b-11e9-85a5-a9876bf33dd8.png)

하지만 지난번과 마찬가지로 `ALLOWED_HOSTS`에러가 나타났네요.<br>
프로젝트를 생성하고 아무것도 안했으니 그럴 수밖에요!

하지만 우리는 -v와 -w 옵션을 통해서 컨테이너 내부에서 생성된 Django 프로젝트를 컨테이너 밖에서도 접근 가능하게 했기 때문에, 수정하여 반영하는 것이 가능합니다.

`settings.py` 에서 `ALLOWED_HOSTS` 부분을 수정해주도록 합니다.

<script id="asciicast-aLv4kVJi7vU8IQTc8D7HaqBEI" src="https://asciinema.org/a/aLv4kVJi7vU8IQTc8D7HaqBEI.js" async></script>

# 5) 결과

![image](https://user-images.githubusercontent.com/34048253/52416726-9f70c000-2b2d-11e9-9eb9-fb164097b865.png)

수정 결과 제대로 된 화면이 나타나는 것을 확인할 수 있습니다.

이전의 [로켓모양](https://user-images.githubusercontent.com/34048253/51082317-0f1aa780-1748-11e9-91b7-2a6c99a98b4b.png)이 아닌 밋밋한 화면으로 나타나는 것은 `Deprecated`상태인 Django 이미지에서 사용한 `Python 3.4.5`때문입니다.
`Python 3.4.5`를 사용함으로써 `Django 1.10.4`가 설치되었고, 오래된 버전의 빌드화면이 나타나네요.

다음 시간에는 이러한 버전문제를 해결하기 위해 `나만의 Django 이미지 만들기`를 해보도록 하겠습니다.
