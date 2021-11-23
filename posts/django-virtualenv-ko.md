---
layout: post
title:  "가상환경(Virtualenv)로 장고(Django) 시작하기"
subtitle: ""
author: "Siner"
header-img: "img/django/django-virtualenv.jpg"
catalog: true
header-mask:  0.3
tags:
    - python
date:   2019-01-12
multilingual: false
---

본 게시물의 내용은 **Ubuntu 16.04** 를 기반으로 작성되었습니다.

---

**virtualenv**를 이용한 **Django** 프로젝트 구성에 필요한 요소는 다음과 같습니다.<br>
- **pip**
- **virtualenv** (**Python** >= 3.5)
- **Django** >= 2.0

---

## 0) virtualenv를 쓰는 이유
**"가상환경으로 안해도 Django 잘 돌아가던데?"** 라고 하시는 분들이 있을겁니다.

만약 이렇다면 어떨까요?

**가상환경 없이** pip 모듈들을 설치해서 사용하고 있었다고 가정해봅시다.<br>
그 당시 최신버전인 **Django 1.11** 버전을 설치하여 프로젝트를 진행했고, 1.11 버전에 새롭게 추가된 **A** 라는 기능을 사용합니다.<br>
수년뒤에 **Django 2.2** 버전이 등장하게 되고, 새로운 기능들이 추가됨과 동시에, 과거에 사용되던 **A** 라는 기능은 더이상 사용할 수 없게 됩니다.<br>
새로운 프로젝트를 진행하려던 저는 새로운 버전인 Django 2.2 를 사용하고 싶은 마음에 **가상환경 없이** 설치된 Django 1.11 버전을 **Django 2.2 버전으로 업그레이드** 해버리고 맙니다.<br>
바로 그순간 Django 1.11 버전으로 배포중이던 프로젝트에서 사용하고 있던 **A** 라는 기능은 작동하지 않게 되어버리죠.

가상환경을 사용하게 되면, **하나의 호스트**에서 **여러 버전**의 python을 동시에 사용할 수 있고,<br>
따라서, **여러 버전**의 Django를 동시에 사용할 수 있습니다.

![python_virtualenv](https://user-images.githubusercontent.com/34048253/51121149-0a461880-185a-11e9-8a5d-6ec7ed58aa60.png)

자, 이제 virtualenv와 함께 Django 프로젝트를 시작해봅시다!

---

## 1) pip 설치
아래의 명령어를 따라 pip를 설치합니다.
  
```bash
curl https://bootstrap.pypa.io/get-pip.py -o get-pip.py
python get-pip.py  # 오류 발생시, 아래의 스크린샷처럼 --user 옵션을 넣어서 입력하면 됩니다.
```

![image](https://user-images.githubusercontent.com/34048253/51082217-18a31000-1746-11e9-94b3-2cdfb6039ca4.png)

---

## 2) virtualenv 설치
pip를 설치했다면, 이번에는 pip를 사용해서 **virtualenv**를 설치 할 것입니다.

```bash
pip install virtualenv  # 오류 발생시, 아래의 스크린샷처럼 --user 옵션을 넣어서 입력하면 됩니다.
```

![image](https://user-images.githubusercontent.com/34048253/51082225-3a9c9280-1746-11e9-9bc4-91a05049b0de.png)

---

## 3) virtualenv 생성
```bash
virtualenv -p python3 venv_for_django
```
 
-p 옵션을 사용하지 않으면, python2 기반으로 virtualenv가 생성될 수 있기 때문에, 버전을 명시해주는 것이 좋습니다.

![image](https://user-images.githubusercontent.com/34048253/51082232-4be59f00-1746-11e9-8eac-5f11b63184f6.png)

---

## 4) virtualenv 활성화
```bash
source venv_for_django/bin/activate
```

![image](https://user-images.githubusercontent.com/34048253/51082234-59028e00-1746-11e9-8200-7bce0f604007.png)

---

## 5) Python 버전 체크
우리가 생성한 virtualenv가 의도대로 python3을 기반으로 설치되었는지 확인합니다.

```bash
python --version
```

**2.0** 이상의 Django을 사용하기 위해서는, virtualenv 안에 **python3** 버전이 반드시 설치되어야 합니다.<br>
그렇기 때문에, virtualenv를 생성할때 **-p python3** 옵션을 넣습니다.

만약 이 옵션을 넣지않아서 python2 버전이 설치된다면, **낮은 버전의 Django (1.11)가 설치될 것입니다.**

<img width="427" alt="image" src="https://user-images.githubusercontent.com/34048253/51124322-63fe1100-1861-11e9-9baa-010c7ddde4a5.png">

![image](https://user-images.githubusercontent.com/34048253/51082240-89e2c300-1746-11e9-9444-018887dd64b2.png)

---

## 6) Django 설치
pip를 사용하여 Django를 설치합니다.

```bash
pip install django
```

![image](https://user-images.githubusercontent.com/34048253/51082246-9c5cfc80-1746-11e9-856e-11fbf902c6a8.png)

---

## 7) Django 프로젝트 생성 (startproject)
Django가 설치된 virtualenv 환경에서 아래의 커맨드를 입력합니다.

```bash
django-admin startproject myproject
```

아래의 스크린샷처럼 프로젝트 이름이 유효하지 않다고 나오는 경우에는, 다른 프로젝트 이름을 사용해봅니다.

![image](https://user-images.githubusercontent.com/34048253/51082256-d0382200-1746-11e9-86bc-5dd2ca6659bc.png)

---

## 8) Django 프로젝트 실행 (runserver)
생성된 프로젝트 안으로 들어가서 **python manage.py runserver** 라는 기본 명령어에 **ip**, **port** 옵션을 넣어서 실행시켜 봅시다.<br>
(외부 접속을 위해 0.0.0.0 이라는 ip를 입력했고, 8000이라는 포트번호를 입력했습니다.)

```bash
python manage.py runserver 0.0.0.0:8000
```

아래의 스크린샷처럼 **migrate** 관련 경고가 나타나지만, 지금 당장은 무시하고 넘어가도 좋습니다.<br>
**makemigrations**과 **migrate**에 관한 내용은 나중에 다루도록 하겠습니다.

![image](https://user-images.githubusercontent.com/34048253/51082269-01b0ed80-1747-11e9-84fe-e045bdb44691.png)

---

## 9) ALLOWED_HOSTS
웹 브라우저를 통해 host ip와 8000번 포트로 접속하면, **DisallowedHost at /** 라는 에러와 함께 디버깅 화면이 나타난 것을 확인할 수 있습니다.

![image](https://user-images.githubusercontent.com/34048253/51082279-5d7b7680-1747-11e9-9674-6e49b2eb9612.png)

우리가 접속할 ip가 **ALLOWED_HOSTS** 리스트에 존재하지 않는다는 것이니, 우리가 접속할 ip인 192.168.0.3을 리스트에 추가해야 합니다.<br>
ALLOWED_HOSTS 설정은 **myproject/settings.py**에 존재합니다.

![image](https://user-images.githubusercontent.com/34048253/51082397-76852700-1749-11e9-9d75-9a4853fee34d.png)

host의 주소(ip, 도메인)가 바뀌어도 접속이 가능하도록, **ALLOWED_HOSTS에 '*'** 를 입력해줍니다.

![image](https://user-images.githubusercontent.com/34048253/51082295-be0ab380-1747-11e9-87cb-fca9b5e81e32.png)

---

## 10) 결과
수정이 끝나고, 다시 **runserver** 커맨드를 실행시키면, 정상적으로 **Django** 서버가 동작하는 것을 확인할 수 있습니다.
![image](https://user-images.githubusercontent.com/34048253/51082317-0f1aa780-1748-11e9-91b7-2a6c99a98b4b.png)
