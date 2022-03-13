---
title: "파이썬(Python3)으로 왓쓰리워즈(what3words, W3W)를 이용해보자."
subtitle: "세상의 모든 주소를 세 단어에 담다"
tags:
    - python
    - map
date: 2019-05-20
image: https://user-images.githubusercontent.com/34048253/155849473-edecbad3-31af-4b30-8895-cb0883b7bd96.png
---

## what3words란?

what3words (W3W)는 전 세계 지도 상의 스팟`(3m x 3m)`을 3개의 `무작위` 단어로 표현한 좌표체계입니다.
이 기능을 사용하면 마이크로 단위의 장소 공유와 주소가 없는 지점도 세 단어 주소로 공유/검색이 가능합니다.
현재 `27개 언어`로 W3W 세단어 주소가 제공 중이며, `카카오`에서도 what3words와의 제휴를 통해 `한글` W3W 주소체계를 서비스 하기 시작했습니다.

![image](https://img1.daumcdn.net/thumb/R1280x0/?scode=mtistory&fname=http%3A%2F%2Fcfile2.uf.tistory.com%2Fimage%2F994AD3385CA1AF9C2EDECC)
<iframe title="[카카오맵] what3words 기능 사용 튜토리얼" width="100%" height="360" src="https://play-tv.kakao.com/embed/player/cliplink/397055543?service=kakao_tv" allowfullscreen frameborder="0" scrolling="no" allow="autoplay"></iframe>
<iframe width="100%" height="315" src="https://www.youtube.com/embed/rsVt5sjXqHg" frameborder="0" allow="accelerometer; autoplay; encrypted-media; gyroscope; picture-in-picture" allowfullscreen></iframe>
<iframe width="100%" height="315" src="https://www.youtube.com/embed/KHi4xQpwohY" frameborder="0" allow="accelerometer; autoplay; encrypted-media; gyroscope; picture-in-picture" allowfullscreen></iframe>
<iframe width="100%" height="315" src="https://www.youtube.com/embed/RFRmSwIiiF8" frameborder="0" allow="accelerometer; autoplay; encrypted-media; gyroscope; picture-in-picture" allowfullscreen></iframe>

## API KEY 발급

what3words를 사용하기 위해서는 [https://accounts.what3words.com/ko/register](https://accounts.what3words.com/ko/register)에서 API KEY를 발급 받아야 합니다.

![image](https://user-images.githubusercontent.com/34048253/58021622-5a095900-7b46-11e9-9799-7f803742eaeb.png)

## pip 패키지 설치

공식 문서는 [https://docs.what3words.com/wrapper/python/](https://docs.what3words.com/wrapper/python/)에 있습니다.<br>
what3words에서는 pip 패키지를 지원하여 python에서 매우 편리하게 이 기능을 사용할 수 있습니다.


```bash
pip install what3words
```

## 위경도 <-> W3W 변환

파이썬 코드로 what3words를 사용하는 두가지 예시를 알아보겠습니다.

`language='ko'`를 입력하지 않으면 영어로 된 w3w값이 나타납니다.

<h4>1. 위경도 >> W3W</h4>

![image](https://user-images.githubusercontent.com/34048253/58022435-6a223800-7b48-11e9-8b15-422af3083b4c.png)

```python
import what3words


geocoder = what3words.Geocoder("what3words-api-key", language='ko')
coordinates = what3words.Coordinates(37.5132924,127.1017133)
res = geocoder.convert_to_3wa(coordinates)

print(res)
```
```bash
{
  'country': 'KR',
  'square': {
    'southwest': {
      'lng': 127.101686,
      'lat': 37.513287
    },
    'northeast': {
      'lng': 127.10172,
      'lat': 37.513314
    }
  },
  'nearestPlace': '성남시, 경기도',
  'coordinates': {
    'lng': 127.101703,
    'lat': 37.5133
  }, 
  'words': '중년.펀드.수확', 
  'language': 'ko', 
  'map': 'https://w3w.co/%EC%A4%91%EB%85%84.%ED%8E%80%EB%93%9C.%EC%88%98%ED%99%95'
}
```

성남시 경기도라고 나오는게 뭔가 잘못 된 느낌인데...
속는 셈 치고 response받은 `중년.펀드.수확`을 카카오 지도에 한번 검색해봅니다.

![image](https://user-images.githubusercontent.com/34048253/58022950-92f6fd00-7b49-11e9-9880-2bd950e747f2.png)

잘 됩니다. 이어서 반대로도 한번 해보겠습니다.

<h4>2. W3W >> 위경도</h4>

![image](https://user-images.githubusercontent.com/34048253/58023529-f7668c00-7b4a-11e9-9bb8-29896adb431e.png)


```python
import what3words

geocoder = what3words.Geocoder("what3words-api-key", language='ko')
res = geocoder.convert_to_coordinates('일차적.사인하다.마음먹었던')
print(res)
```
```bash
{
  'country': 'ZZ',
  'square': {
    'southwest': {
      'lng': 125.977585,
      'lat': 37.552151
    }, 
    'northeast': {
      'lng': 125.977619, 
      'lat': 37.552178
    }
  },
  'nearestPlace': 'Yŏnan-ŭp, 황해남도', 
  'coordinates': {
    'lng': 125.977602, 
    'lat': 37.552164
  },
  'words': '일차적.사인하다.마음먹었던', 
  'language': 'ko', 
  'map': 'https://w3w.co/%EC%9D%BC%EC%B0%A8%EC%A0%81.%EC%82%AC%EC%9D%B8%ED%95%98%EB%8B%A4.%EB%A7%88%EC%9D%8C%EB%A8%B9%EC%97%88%EB%8D%98'
}
```

![image](https://user-images.githubusercontent.com/34048253/58024040-452fc400-7b4c-11e9-9295-3d9bf5e4210b.png)

변환이 잘 되는것을 확인 할 수 있습니다.

## 마무리

W3W 주소체계는 매우 편리하지만 아직 많은 사람들이 알지 못하여 아쉬울 따름입니다.
많은 분들이 이 기능을 사용하는 서비스를 많이 만들어서 한국에서도 많이 사용되면 좋겠네요.

이번에 다루지 못한 언어에서의 API 사용법에 대해서는 공식문서(아래 링크)를 참고하시면 됩니다.
[https://docs.what3words.com/api/v3/](https://docs.what3words.com/api/v3/) 
