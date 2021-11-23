---
layout: post
title:  "프로덕션 레벨의 크롤러 개발 회고"
subtitle: "python은 과연 크롤링하기 좋은 언어인가"
author: "Siner"
catalog: true
header-mask:  0.3
tags:
    - crawling
    - typescript/javascript
    - container
    - aws
date:   2021-04-18
multilingual: false
---

지난 25일동안 지인의 부탁으로 특정 게시판의 게시글들을 전부 크롤링하는 외주를 진행했었다.
이를 진행하면서 얻게 된 크롤링 관련 경험에 대해 적어보려고 한다.

지금까지 나의 크롤러 프로젝트는 python으로 개발이 되어있었는데, 이번엔 typescript로 작성해달라는 요구사항을 받아서 진행했고, 내가 느낀 python 크롤링과 javascript 기반 언어의 크롤링의 차이점에 대해서도 적어볼 수 있겠다.

회고는 크게 3가지 주제로 나눌 예정이다.

1. python 크롤링 vs javascript(typescript) 크롤링 (크롤링 환경 구성)
2. production(컨테이너) 레벨에서의 죽지않는 크롤러 만들기 (컨테이너의 메모리 관리 aka 브라우저 캐시데이터 관리)
3. 크롤러를 만든다는 것 (해당 사이트의 개발 히스토리를 모두 파악하는 것)


## 1. python 크롤링 vs javascript(typescript) 크롤링
python을 사용한 크롤링 환경 구성시 [`google-chrome-stable_current_amd64.deb`](https://dl.google.com/linux/direct/google-chrome-stable_current_amd64.deb)과 [`chromedriver`](https://chromedriver.chromium.org/downloads) 두가지를 외부 저장소를 통해 받아야 하고, pip를 통해 selenium을 설치해야하고, 
javascript에서는 google-chrome-stable는 똑같이 외부로부터 받아야 하지만, chromedriver과 selenium을 npm을 통해 받을 수 있다는 차이가 있다.

이로 인해 두가지가 편해졌는데, 그중 한가지는 chromedriver 바이너리 파일 관리가 편해졌다는 점이다.
mac에서 mac용 바이너리로 개발하다가, 배포 환경에서는 linux용 amd64바이너리를 받아서 배포해야 했는데, 이 때문에 버전의 변경이 일어났을때 chromedriver에서 다운받아 압축풀고 프로젝트로 복사하는 일련의 과정들이 너무 번거롭게 느껴졌었다. 이를 관리하기 위한 코드도 추가되었던 것은 덤이다.

```python
if env === 'local':
  chromedriver_path = '<현재폴더>/chromedriver_mac' # 개발시엔 /user/<username>/repositories/<repository>/binaries/chromedriver_mac 과 같은 경로에 바이너리를 넣어두었다
else:
  chromedriver_path = '/app/chromedriver' # linux 배포시 도커 이미지 환경이므로 WORKDIR이 app으로 되어있다
```

npm으로 관리시엔 mac, linux 구분없이 `node_modules/chromedriver` 하위 경로에 바이너리가 자동으로 설치되고, 우리가 이걸 몰라도 상관 없도록 설계되어 있었다. 다만 터미널 등의 npm 환경이 아닌 곳에서 노드를 실행한다면, PATH에 `node_modules/.bin`이 포함되지 않아서 바이너리를 찾지 못했다.

![image](https://user-images.githubusercontent.com/34048253/115136647-079b8480-a05c-11eb-9222-c316cddf21ff.png)
![image](https://user-images.githubusercontent.com/34048253/115136769-ec7d4480-a05c-11eb-865f-fd85078501d9.png)

두번째로 편해진 점은, google-chrome-stable 패키지의 버전 추적이 쉬워졌다는 점이다.
컨테이너 이미지 빌드시에 `google-chrome-stable_current_amd64.deb` 설치파일을 매번 외부로부터 받아야 한다면, latest 버전을 받게 될텐데, chromedriver와 버전이 일치해야 실행이 가능하기 때문에 미리 받아둔 설치파일을 repository에 넣어두는 방식을 선호하고있다.
구 버전의 google-chrome-stable 설치파일에 대한 다운로드 url도 제공하고 있는것 같긴 하지만, 이렇게 되면 도커 이미지 빌드 시간도 길어질 뿐더러 마이너 버전까지 상세하게 기입해야 다운로드가 되는 것 같아서 선호하지 않는다. (또한, dl.google.com은 정확한 url이 아니면 접속할 수 없어서 유효한 링크를 찾는 과정이 번거롭다)

그렇기 때문에, npm에 chromedriver의 버전을 기입해 둘 수 있다는 점이 참 편리하다고 느껴졌다. 항상 버전을 확인하려면 cli를 사용해야 했기 때문에...
```bash
$ ./chromedriver --version
ChromeDriver 89.0.4389.23 (61b08ee2c50024bab004e48d2b1b083cdbdac579-refs/branch-heads/4389@{#294})
```

## 2. production(컨테이너) 레벨에서의 죽지않는 크롤러 만들기 (컨테이너의 메모리 관리 aka 브라우저 캐시데이터 관리)
코드 작성이 거의 마무리되고, aws batch를 통해 운영 테스트를 하던 도중, 메모리 릭이 있는것을 발견했다.

![image](https://user-images.githubusercontent.com/34048253/115136984-4cc0b600-a05e-11eb-857c-b7bee361633a.png)

로컬에서 컨테이너를 모니터링 해본 결과, 계속해서 zombie 프로세스가 생기는 것을 발견하였다.

![image](https://user-images.githubusercontent.com/34048253/115137501-7fb87900-a061-11eb-9ac4-b88751a6a224.png)

확인해본 결과 headless 브라우저는 driver.quit() api가 제대로 동작하지 않는 것을 확인하였다.
하나의 브라우저로 계속 크롤링 작업을 했을때 어느 순간 크롤링이 중단되는 문제를 발견하였고, 이를 포함하여 예상치 못한 변수들을 통제하기 위해 일정 주기로 브라우저를 종료 후 다시 실행시키도록 코드를 작성하였는데, 종료가 되지 않고 좀비 프로세스만 늘어나는 상황이 발생한 것이다...

따라서 브라우저를 종료시키지 않고 운영해야 했고, 그렇다면 하나의 브라우저를 어떻게 안정적인 상태로 크롤링 작업을 수행할 수 있게 할수 있을까 하는 고민을 하게 되었다.

우선 메모리 릭 발생 원인을 찾기위해 코드를 샅샅이 둘러보았으나 딱히 문제가 될 만한 곳은 발견되지 않았고, 컨테이너 내부에서 top 명령어를 통해 크롬이 점유하는 메모리는 크게 변하지 않지만, 컨테이너의 메모리만 증가하는것을 확인했다. (위 스크린샷)
따라서 크롬 브라우저가 사용하는 메모리 자체는 증가하지 않지만, 컨테이너의 볼륨이 증가함으로써 외부에서 보았을때 메모리가 증가하는 것으로 관측된 것이다. 크롬 브라우저를 오랫동안 사용하는 작업이므로, 브라우저의 캐시데이터 등이 문제가 되는 것으로 보여졌다.

크롬 브라우저의 캐시데이터를 지우는건 chrome://settings/clearBrowserData에 접속하여 버튼을 누르면 되는데, 이는 [shadow](https://developer.mozilla.org/ko/docs/Web/Web_Components/Using_shadow_DOM) 돔에 의해 막혀있어서 일반적인 크롤링 방법으로는 shadow돔에 접근하기가 쉽지 않았다. 방법들을 찾아보았으나, 실패하고 말았다. 

![image](https://user-images.githubusercontent.com/34048253/115137098-dcfefb00-a05e-11eb-98c3-5ce1ab975277.png)

- 찾아본 shadow root 접근 방법들
  - [css selector를 통한 방법](https://intoli.com/blog/clear-the-chrome-browser-cache/)
  - [스크립트 실행을 통한 방법](https://stackoverflow.com/a/56381495/9906215)

대신 selenium webdriver에 deleteAllCookies api가 존재하는 것을 확인했고, 바로 적용했다.
```typescript
async function clearBrowserData(driver: WebDriver): Promise<WebDriver> {
  await driver.manage().deleteAllCookies();
  console.log('all cookies are deleted');
  return driver;
}
```
하지만 쿠키만으로는 메모리 상승을 크게 억제하기가 힘들었고, 캐시를 줄여야 했다.

[chromium](https://source.chromium.org/chromium/chromium/src/+/master:README.md) 오픈소스를 살펴보면서 [setCacheCapacity](https://source.chromium.org/chromium/chromium/src/+/master:components/web_cache/renderer/web_cache_impl.cc;l=40;bpv=0;bpt=1) 라는 함수를 찾아보기도 했으나, 마땅한 해결책은 보이지 않았다.
컨테이너 내부의 /tmp 경로에 있는 크롬 디렉터리에 캐시데이터가 쌓이는 것을 발견하긴 했지만, 운영 도중 cli로 계속해서 cache를 지우는 방법보다 세련된 방법을 찾고싶었다. (cache 경로에 포함된 hash값을 예측하기 힘든 것도 번거로운 점이었고, 삭제로 인해 또 다른 문제가 발생할 수도 있지 않겠는가...)

![image](https://user-images.githubusercontent.com/34048253/115137345-97dbc880-a060-11eb-9145-2a5ca5bc220a.png)

그러던 도중 chromeOptions에 disk-cache-size를 정할 수 있는 방법이 있다는 것을 찾았고, 아래의 코드가 그 결과이다.
메모리 증가 현상이 계속 있긴 했지만, 크롤러가 끝날때까지는 안정적인 상태를 유지하게 되어 운영 단계에서 일단 사용할수 있다고 판단했다.

```typescript
async function getNewDriver(driver?: WebDriver): Promise<WebDriver> {
  if (driver) await driver.quit();
  // init browser
  const options: Options = new Options();
  options.headless();
  options.addArguments('--disable-dev-shm-usage');
  options.addArguments('--disable-gpu');
  options.addArguments('--no-sandbox');
  options.setUserPreferences({ 'disk-cache-size': 4096 });
  options.windowSize({ width: 1920, height: 1080 });
  return new Builder().forBrowser('chrome').setChromeOptions(options).build();
}
```
 
![image](https://user-images.githubusercontent.com/34048253/115137416-f99c3280-a060-11eb-8998-29af27199b93.png)

위 이미지는 1시간마다 batch job을 실행시킨 결과이다. 하나의 job당 5시간정도 크롤링을 수행하기 때문에 초기 리소스 증가는 어느정도 용인이 가능하다.

무엇보다 크롤러가 죽지않고 무사히 작업을 끝냈다.

## 3. 크롤러를 만든다는 것 (해당 사이트의 개발 히스토리를 모두 파악하는 것)
크롤링을 진행하면서, 몇몇 페이지에만 특정 element가 보이지 않는다는 것을 알았고, 오래된 게시글에서 이러한 증상이 많이 나타나는 것을 알게 되었다. 추측컨데, 이는 해당 웹사이트가 유지보수 과정을 겪으면서 새로운 기능이 추가된 것으로 판단되었다. 유저들은 최신 게시글만을 보기 때문에 상관없지만, 크롤러를 만드는 입장에서는 과거의 데이터까지 전부 수집하기 때문에 이에 대한 예외처리를 해주어야 한다. 어쩔 수 없이 해당 서비스의 모든 히스토리를 파악하게 되는 것이다.
게시판에서 사용중인 에디터에서 사용자의 직접적인 html 수정을 지원했기 때문에 특정 element로 화면을 덮어버리도록 설계된 게시글의 경우, 해당 element를 삭제하고 크롤링 작업을 이어서 수행할 수도 있었지만, 이러한 코드는 특정 게시글에 대한 하드코딩이고, 좋은 방법은 아니라고 생각했다.
또한 이런 예측 불가능한 유저의 행동까지 모두 커버해야 할 만큼 이 특정 게시글이 수집해야 하는 가치가 있는지 다시 확인하게 되었고, 이러한 게시글이 존재한다는 점만 공유하고 해당 게시글을 위한 하드코딩은 작성하지 않았다.

## 마무리
크롤러를 운영하게 되면 해당 사이트가 UI가 변화할때마다(유지보수를 할 때마다) 나도 유지보수를 해주어야 하기 때문에, 내가 작업할 수 없는 시기에 사이트에 변화가 일어난다면 크롤링을 수행할 수 없게 되고, 이는 크롤러 데이터 신뢰도에 영향이 미치게 된다. 나의 경우엔 게시글의 삭제 여부를 판단하여 soft delete 처리를 해야 했는데, 이게 실제로 삭제된건지, 특정 시기에만 connection에 문제가 생겨서 불러오지 못한건지 판단하기 애매하다고 생각했고, crawledAt 필드에 크롤링 event의 startedAt과 비교하여 crawledAt이 더 늦으면 isDeleted를 true로 하되, 다음번 크롤링 수행시에 다시 탐지하게 되었다면 isDeleted를 false로 다시 바꿔주도록 구현했다.

개인적으로 이미 존재하는 데이터를 통해 인사이트를 얻고, 경쟁력을 가지도록 하는 작업은 충분히 의미가 있다고 생각한다.
하지만 수없이 많이 변화하는 (개발자가 아주 많이 있는) 사이트를 크롤링 하는 작업은 매우 고되며, 외주가 아닌 항상 유지보수를 해야하는 작업이 될 수도 있다는 것을 염두해 두어야 할 것이다.

ps. 내가 작업했던 사이트는 상당히 오래된 서비스였고 ui의 변경도 거의 일어나지 않았기에, 아마도 당분간 추가적인 유지보수엔 리소스가 많이 들지 않을 것으로 생각된다.
