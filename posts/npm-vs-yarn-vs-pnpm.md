---
layout: post
title:  "npm vs yarn vs pnpm"
subtitle: "pnpm은 정말로 탄소절감과 ssd수명에 도움을 줄까?"
author: "Siner"
catalog: true
header-mask:  0.3
tags:
    - Javascript
    - Typescript
date:   2020-12-19
multilingual: false
---

# 서론

타입스크립트를 사용하면서 백엔드를 개발하면서 `googleapis@39.2`를 사용하던 도중, 의존성으로 함께 설치되는 `google-auth-library@3.0.0`의 타입을 가져오는데 의문을 가지게 되었다.

```typescript
// npm 으로 설치해서 사용하는 상황
import { google } from 'googleapis';
import { OAuth2Client } from 'google-auth-library';

const auth: OAuth2Client = new google.auth.OAuth2();
```

![스크린샷 2020-12-19 오후 6 34 21](https://user-images.githubusercontent.com/34048253/102686125-e1d6ae00-4228-11eb-8f1d-a29ebcae10d9.png)

의존성으로 가져오는 버전 말고 다른 버전을 별도로 설치하고 있었다면 어떻게 되는거지? 싶어서 `google-auth-library`의 최신버전을(6.1.3) 추가로 설치해보았고, 역시나 아래와 같이 잘못된 버전의 타입을 가져오면서 **TS2739** 에러가 났다.

![node_modules](https://user-images.githubusercontent.com/34048253/102686319-39c1e480-422a-11eb-8b88-f30f2228fe0e.gif)

위의 에러를 해결하기 위해서는 어쩔 수 없이 `googleapis/node_modules/google-auth-library` 를 통해 접근해야 했다. (이게 맞는건가? 싶었다)

그러던 어느날 [reason-seoul](https://github.com/reason-seoul/reason-todomvc) 레포에 다음과 같은 흥미로운 문구가 있어서 javascript의 패키지 매니징 도구들에 대해 알아보는 시간을 가지기로 했다.

```
저장소는 탄소를 절감하고 우리들의 SSD 수명을 조금 더 보존하기 위해 pnpm 패키지 매니져를 사용하고 있습니다 😉
```

# 비교

쉬운 비교를 위해 위에서 설명한 패키지를 다음과 같이 이름 지어서 설명하겠다.

- `foo@39`: `googleapis@39`
- `bar@3.0.0`: `google-auth-library@3.0.0` (googleapis의 의존성)
- `bar@6.1.3`: `google-auth-library@6.1.3`

## npm

#### 1. foo 설치
- node_modules 하위에 `foo`, `bar`가 flat하게 설치되었다.

#### 2. bar@6.1.3 설치
- 기존 `node_modules`/`bar` 폴더의 내용이 `foo/node_modules/bar`로 즉시 이동(mv)하는것이 아닌, `bar@3.0.0` 삭제 후 그 위치에 새로 `bar@6.1.3`을 설치하고, `foo`/`node_modules`/`bar` 경로에 `bar`가 새로 설치되었다. 단순히 mv가 아닌 삭제라고 결론내린 이유는, `bar@6.1.3` 설치 전에 `bar@3.0.0` 하위에 새로운 파일을 생성해보고, `bar@3.0.0`이 가지고있는 README.md 파일에 수정을 가한 뒤에 `bar@6.1.3`을 설치했기 때문이다. ~물론 git reset --hard 후에 이동시켰을 가능성도 없지는 않다.~

#### 3. bar@6.1.3 삭제
- `node_modules`/`bar` 폴더가 제거되었고, `node_modules`/`foo`/`node_modules`/`bar`가 `node_modules`/`bar`로 돌아오지는 않았다.

#### 4. bar@3.0.0을 수동으로 설치
- `node_modules`/`bar` 경로에 `bar@3.0.0` 이 설치되었다.
- `node_modules`/`foo`/`node_modules`/`bar`에는 똑같은 버전의 `bar@3.0.0`이 남아있어서 같은 버전의 패키지가 중복으로 존재하게 된다.

## yarn

#### 1. foo@39 설치
- npm과 마찬가지로 node_modules 하위에 `foo`, `bar`가 flat하게 설치되었다.

#### 2. bar@6.1.3 설치
- npm으로 설치했을 때와 마찬가지로 동작하였다.

#### 3. bar@6.1.3 삭제
- `node_modules`/`bar`에 설치되어있던 `bar@6.1.3`이 삭제되고, `node_modules`/`foo`/`node_modules`/`bar` 하위폴더에 있던 `bar@3.0.0`이 삭제되고, 처음 설치 위치인 `node_modules`/`bar`에 다시 설치되어 `foo`와 flat한 위치에 존재하게 되었다.

#### 4. foo@39가 의존성을 가지는 bar@3.0.0 수동으로 설치
- `node_modules`/`bar`에 같은 버전이 존재하기 때문에 아무 일도 일어나지 않았다.

## pnpm

#### 1. foo@39 설치
- `foo`의 [package.json](https://github.com/googleapis/google-api-nodejs-client/blob/v39.2.0/package.json)에는 `bar@3.0.0`이 의존성으로 명시되었는데, 실제 설치된건 `bar@3.1.2`였다. [이슈](https://github.com/pnpm/pnpm/issues/3033)로 남겨두고 일단 넘어가자...

![스크린샷 2020-12-19 오후 7 57 24](https://user-images.githubusercontent.com/34048253/102687733-6aa71700-4234-11eb-947b-6454ee256e02.png)
<img src="https://user-images.githubusercontent.com/34048253/102688760-17d15d80-423c-11eb-9b0d-99d5ce93daa9.png" width="400" />

- node_modules 하위에 `foo`폴더가 생기고, 의존성을 가지는 패키지들은 `node_modules/.pnpm` 폴더안에 각각의 버전이 포함된 이름 폴더가 생성되었다.

![스크린샷 2020-12-19 오후 8 47 42](https://user-images.githubusercontent.com/34048253/102688652-71855800-423b-11eb-947e-16fcc1a4b6c6.png)

- `.pnpm`/`bar@3.1.2`/`node_modules`/`base64-js` 폴더와 같은 내용이 `.pnpm`/`base64-js@1.5.1`에 존재하는 것을 확인했는데, `심볼릭 링크`인 것을 확인하였다.

![스크린샷 2020-12-19 오후 9 17 07](https://user-images.githubusercontent.com/34048253/102689198-8e238f00-423f-11eb-825e-0a3d1be005d0.png)

- 사실 `node_modules` 하위에 있는 폴더들도 `.pnpm`에서 가져온 것들임을 확인할 수 있다.<br>
(설치되는 모든 버전들은 .pnpm에 관리되고, 코드에서 사용할 것들만 node_modules에 심볼릭 링크 형태로 생성되는 것이다)


#### 2. bar@6.1.3 설치
- `node_modules`/`bar` 폴더가 생성되었고, `node_modules/.pnpm` 폴더에 새로 설치된 `bar@6.1.3`의 의존성 패키지가 추가되었다.<br>
(이미 설치된 버전이 있다면, 새로 설치되지 않을 것으로 추측된다.)

#### 3. bar@6.1.3 삭제
- `node_modules`/`bar` 폴더가 삭제되고, `node_modules`/`.pnpm` 폴더에 설치되었던 `bar@6.1.3`의 의존성 패키지가 사라지며, `.pnpm`에는 1번 과정에서 설치되었던 폴더만 남아있게 되었다.

#### 4. foo@39의 의존성으로 설치된 bar@3.1.2 수동으로 설치
- 잘못 설치된 `bar@3.1.2`를 수동으로 설치해본 결과, `.pnpm`에 존재하던 `bar@3.1.2`의 심볼릭 링크가 `node_modules`/`bar`로 생성되었다.

# 결론

dependency를 한곳에서 관리하면서 동일 버전의 중복생성이나 같은 이름 다른버전의 dependency 생성시 패키지가 삭제되었다가 다른 위치에 새로 생성되는 등의 불필요한 I/O 과정을 없애는 것으로 pnpm의 장점을 확실히 알게 되었다.

이로써 reason-seoul 레포에서 설명한 탄소 절감 및 SSD의 수명을 위해서 pnpm을 사용한다는 이유도 충분히 납득이 되었다.

하지만 pnpm 사용지 dependency가 설치되어있는 node_modules에 접근이 불가능한 점이 있는데, 애초에 npm, yarn사용시 그러한 디렉토리 접근이 제대로된 방법인지 의심되는 상황이었기 때문에, **pnpm의 사용을 적극 권장하고싶다.**

# 참고자료

- [pnpm, 플랫하지 않은 패키지 매니저](https://imch.dev/posts/pnpm-a-manager-what-is-not-flat)
- [NPM vs PNPM vs Yarn](https://rushjs.io/pages/maintainer/package_managers/)
- [reason-seoul](https://github.com/reason-seoul/reason-todomvc)
