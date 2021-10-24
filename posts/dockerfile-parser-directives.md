---
layout: post
title:  "[번역] Dockerfile 레퍼런스 (4) - Parser Directives"
subtitle: "FROM 명령어 이전에 사용할 수 있는 지시문"
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

# Parser directives
파서 지시어는 선택사항이며, 이 지시어 아랫줄부터 영향을 주게 됩니다. 파서 지시어는 빌드를 레이어를 추가하지 않고, 빌드 단계에서 나타나지 않습니다.
파서 지시어는 `# directive=value`와 같은 주석 형태로 표기됩니다. 같은 종류의 지시어를 여러번 사용할 수 없습니다.

지시어 이후에 빈 라인이 나타나거나 빌더 명령어가 실행되면, 도커는 그 이후에 나타나는 파서 지시어를 모두 주석으로 간주하고 유효성 검사를 하지 않습니다.
그러므로, 모든 파서 지시어는 `Dockerfile`의 최상단에 위치해야 합니다.

파서 지시어는 대소문자를 구별하지 않습니다. 하지만, 컨벤션은 소문자를 권장하고 있습니다. 또한 컨벤션은 지시문 이후에 빈 라인을 두는 것을 권장합니다. 빈줄 없는 파서 지시문은 유효하지 않습니다.

다음 오는 예시들은 위의 규칙들을 어긴 것들 입니다. (이렇게 하면 안됩니다)

---
- **연속적인** 라인이라서 무효:
```text
\# direc \
tive=value
```

- **같은 지시문이 두번** 적혀서 무효:
```text
# directive=value1
# directive=value2
FROM ImageName
```

- **빌더 명령어 이후**에 적혀서 주석으로 간주함:
```text
FROM ImageName
# directive=value
```

- **주석 이후**에 파서 지시어가 와서 무효:
```Dockerfile
# About my dockerfile
# directive=value
FROM ImageName
```

- **유효하지 않은 지시어**가 먼저 와서 이것을 주석으로 간주하고, 그 이후에 오는 유효한 지시어까지 무효로 처리된 경우:
```Dockerfile
# unknowndirective=value
# knowndirective=value
```

- 아무리 공백의 차이가 있다고 하지만, 아래의 모든 라인은 **동일한 라인**으로 봅니다:
```Dockerfile
#directive=value
# directive =value
#	directive= value
# directive = value
#	  dIrEcTiVe=value
```

---

파서 지시어는 **두 종류**가 있습니다.
* [`syntax`](#syntax)
* [`escape`](#escape)

#### syntax
```Dockerfile
# syntax=[원격 저장소의 이미지 레퍼런스]
```

예시는 아래와 같습니다.
```Dockerfile
# syntax=docker/dockerfile
# syntax=docker/dockerfile:1.0
# syntax=docker.io/docker/dockerfile:1
# syntax=docker/dockerfile:1.0.0-experimental
# syntax=example.com/user/repo:tag@sha256:abcdef...
```

이 기능은 [BuildKit](#buildkit)을 사용하는 경우에만 사용할 수 있습니다.

syntax 지시어는 이 도커파일을 빌드하기 위한 빌더의 경로를 지정할 수 있습니다. BuildKit 백엔드는 도커 이미지로 배포되고 컨테이너 샌드 박스 환경 내에서 실행되는 외부 빌더 구현을 원활하게 사용할 수 있습니다.

커스텀 도커파일 구현은 아래의 것들을 가능하게 합니다:
- 데몬 업데이트 없이 자동으로 버그를 픽스
- 당신의 도커파일을 빌드하기 위해 모든 유저가 같은 행동을 한다는 것을 보장할 수 있음
- 실험적인 기능과 서드파티의 기능들을 시도해 볼 수 있음

#### - Official releases
도커에서는 도커파일을 빌드하기 위한 공시 이미지 버전들을 도커 허브에 있는 `docker/dockerfile` 저장소에 배포하고 있습니다.
이 저장소의 이미지들은 크게 안정적인 것과, 실험적인 것의 두 채널로 분류됩니다.

안정적인 채널은 시멘틱 버전 네이밍을 사용합니다.
- docker/dockerfile:1.0.0 - 1.0.0 버전만을 가리킴
- docker/dockerfile:1.0 - 1.0.* 버전들을 가리킴
- docker/dockerfile:1 - 1.. 버전들을 가리킴
- docker/dockerfile:latest - 안정적인 버전중 가장 최신의 것을 가리킴

실험적인 채널은 안정적인 버전의 이름을 따라 가지만, experimental 이라는 이름이 붙습니다.
- docker/dockerfile:1.0.1-experimental - 1.0.1-experimental 버전만을 가리킴
- docker/dockerfile:1.0-experimental - 1.0 이후 가장 최근의 실험적인 버전을 가리킴
- docker/dockerfile:experimental - 가장 최근의 실험적인 버전을 가리킴

마음에 드는 채널을 선택해서 사용하면 됩니다. 만약 버그가 수정된 버전을 사용하고 싶다면, `docker/dockerfile:1.0`을 사용하면 되고, 실험적인 기능들을 사용하고 싶다면, expeerimental 채널을 사용하면 됩니다.
만약 당신이 실험적인 채널을 사용하고 있다면, 새로운 실험적인 릴리즈는 이전 버전과 호환이 되지 않을 수도 있습니다. 그러므로 안정적인 버전을 사용하는 것을 추천드립니다.

이러한 기능들의 릴리즈 관련 소식은 [Builtkit 저장소](https://github.com/moby/buildkit/blob/master/README.md)에 적혀 있습니다.

#### escape

```Dockerfile
# escape=\ (backslash)
```

또는

```Dockerfile
# escape=` (backtick)
```

> `\` 와 `` ` `` 이외의 문자로는 사용이 불가능합니다.

`escape` 지시어는 `Dockerfile` 이스케이프 문을 사용하기 위한 문자입니다. 따로 지정하지 않는다면, 기본 이스케이프 문자는 `\`를 사용합니다.

이스케이프 문자는 문자 내부에서 문자열을 이스케이프 하기 위해서 사용하거나, 새로운 라인으로 이스케이프 하기 위해 사용됩니다. `Dockerfile` 명령어는 멀티 라인을 지원합니다.

`RUN` 명령어의 경우에는 `escape` 파서 지시어를 별도로 지정하더라도 마지막 줄에서만 문자열 이스케이프가 가능합니다. (마지막 줄이 아닌 경우에는, 명령어를 새로운 줄에 이어서 작성하는 목적으로만 사용할 수 있습니다.)  

파서 지시어를 `` ` ``로 지정하는 경우에는 `\`를 경로를 구분하는데 사용하는 `Windows`에서 특히 유용합니다.

아래의 예시를 `Windows`에서 실행하는 경우, 두번째 줄의 두번쨰 `\`문자의 경우 첫번째 `\`를 이스케이프 하라는 뜻이 아닌, 새로운 줄로 이스케이프 하라는 뜻으로 해석됩니다.
마찬가지로, 세번째 줄의 마지막 `\`문자열은 실제 명령으로 처리되었다고 가정하면, 이 도커파일의 결과는 두번째 줄과 세번째 줄을 하나의 명령으로 간주합니다.

```Dockerfile
FROM microsoft/nanoserver
COPY testfile.txt c:\\
RUN dir c:\
```

결과:

```bash
PS C:\John> docker build -t cmd .
Sending build context to Docker daemon 3.072 kB
Step 1/2 : FROM microsoft/nanoserver
 ---> 22738ff49c6d
Step 2/2 : COPY testfile.txt c:\RUN dir c:
GetFileAttributesEx c:RUN: The system cannot find the file specified.
PS C:\John>
```

위의 문제를 해결하는 방법 중 하나는, `\` 대신 `/`를 사용하여 타겟을 지정하는 것입니다. 하지만 이 구문은 `Windows`의 경로에는 자연스럽지 않으므로 혼란스럽고, 최악의 경우 `Windows`의 모든 명령이 `/`를 경로 구분 기호로 지원하지는 않으므로 오류가 발생할 수 있습니다.

아래의 `Dockerfile`예시에서는 `escape` 파서 지시어를 추가함으로써 `Windows`의 경로표시를 자연스럽게 사용하면서도, 빌드에 성공할 수 있습니다.

```Dockerfile
# escape=`

FROM microsoft/nanoserver
COPY testfile.txt c:\
RUN dir c:\
```  

결과:

```bash
PS C:\John> docker build -t succeeds --no-cache=true .
Sending build context to Docker daemon 3.072 kB
Step 1/3 : FROM microsoft/nanoserver
 ---> 22738ff49c6d
Step 2/3 : COPY testfile.txt c:\
 ---> 96655de338de
Removing intermediate container 4db9acbb1682
Step 3/3 : RUN dir c:\
 ---> Running in a2c157f842f5
 Volume in drive C has no label.
 Volume Serial Number is 7E6D-E0F7

 Directory of c:\

10/05/2016  05:04 PM             1,894 License.txt
10/05/2016  02:22 PM    <DIR>          Program Files
10/05/2016  02:14 PM    <DIR>          Program Files (x86)
10/28/2016  11:18 AM                62 testfile.txt
10/28/2016  11:20 AM    <DIR>          Users
10/28/2016  11:20 AM    <DIR>          Windows
           2 File(s)          1,956 bytes
           4 Dir(s)  21,259,096,064 bytes free
 ---> 01c7f3bef04f
Removing intermediate container a2c157f842f5
Successfully built 01c7f3bef04f
PS C:\John>
```

---

아래 예시는 제가 번역을 진행하면서 궁금하여 진행해본 `escape` 명령어 테스트입니다.
* Ubuntu 18.04에서 진행되었습니다.

```Dockerfile
# escape=`
FROM ubuntu:18.04

RUN echo 'asdf' `
    echo '`\'

RUN echo 'ha`h\a\`h`\a'
RUN echo '\\\\'
RUN echo '\\\'
RUN echo '\\'
RUN echo '\'

RUN echo '````'
RUN echo '```'
RUN echo '``'
RUN echo '`'

RUN echo '`\`\'
RUN echo '\`\`'
RUN echo '\\``'
RUN echo '``\\'
```

결과:
 
`````bash
siner@ubuntu:~/test$ docker build .
Sending build context to Docker daemon  2.048kB
Step 1/15 : FROM ubuntu:18.04
 ---> 72300a873c2c
Step 2/15 : RUN echo 'asdf'     echo '`\'
 ---> Running in 63bfd5c9b030
asdf echo `\
Removing intermediate container 63bfd5c9b030
 ---> c59c61a1823c
Step 3/15 : RUN echo 'ha`h\a\`h`\a'
 ---> Running in eebc9c63c671
ha`h\`h`
Removing intermediate container eebc9c63c671
 ---> a2fb4c9e4a9a
Step 4/15 : RUN echo '\\\\'
 ---> Running in 40a00bd479dd
\\
Removing intermediate container 40a00bd479dd
 ---> 35781e6794da
Step 5/15 : RUN echo '\\\'
 ---> Running in a38c68062333
\\
Removing intermediate container a38c68062333
 ---> f0066f67b029
Step 6/15 : RUN echo '\\'
 ---> Running in 187b6170bfd4
\
Removing intermediate container 187b6170bfd4
 ---> a8c405279af9
Step 7/15 : RUN echo '\'
 ---> Running in 6c489b41da38
\
Removing intermediate container 6c489b41da38
 ---> ee80e82c92bf
Step 8/15 : RUN echo '````'
 ---> Running in 8322471eaf3d
````
Removing intermediate container 8322471eaf3d
 ---> d7c2cdc92ee6
Step 9/15 : RUN echo '```'
 ---> Running in 0cbb7d4b92c2
```
Removing intermediate container 0cbb7d4b92c2
 ---> 08982a6c293c
Step 10/15 : RUN echo '``'
 ---> Running in 50dce8984fed
``
Removing intermediate container 50dce8984fed
 ---> 5e72885cee6c
Step 11/15 : RUN echo '`'
 ---> Running in 35465e86315a
`
Removing intermediate container 35465e86315a
 ---> e876758883b2
Step 12/15 : RUN echo '`\`\'
 ---> Running in 7d1db349e7f8
`\`\
Removing intermediate container 7d1db349e7f8
 ---> b79ee49bd297
Step 13/15 : RUN echo '\`\`'
 ---> Running in cf1fb36bf26f
\`\`
Removing intermediate container cf1fb36bf26f
 ---> c4d67030daeb
Step 14/15 : RUN echo '\\``'
 ---> Running in 8c1607bb257a
\``
Removing intermediate container 8c1607bb257a
 ---> 9659dc608e5c
Step 15/15 : RUN echo '``\\'
 ---> Running in ca2262958b0b
``\
Removing intermediate container ca2262958b0b
 ---> e825ce9d6a5c
Successfully built e825ce9d6a5c
siner@ubuntu:~/test$ 
`````
