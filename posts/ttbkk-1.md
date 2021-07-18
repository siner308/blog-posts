---
layout: post
title:  "떡볶이맵 제작기 #1"
subtitle: "문득 떡볶이가 먹고싶을때, 이곳을 방문하라"
author: "Siner"
catalog: true
header-mask:  0.3
tags:
    - Javascript
    - Typescript
    - TTBKK
    - tteokbokki
date:   2021-07-18
multilingual: false
---

10주전 새로운 토이프로젝트가 뭐가 있을까 고민하다가 1주일에 한번은 먹어야 하는 떡볶이를 아이템으로 한 `떡볶이맵`을 만들어 보기로 결심했다.

<div>
<img src="https://user-images.githubusercontent.com/34048253/126061816-16961267-3c95-4978-883c-8621ba16c29e.png" height=500 />
</div>

도메인을 먼저 정하는게 중요했는데, 떡볶이의 영어발음인 tteokbokki는 너무 길고, 한국인이 저 철자를 다 외우는 사람이 많지는 않다고 생각되어 ttbkk.com라는 간단한 도메인을 구입했다. (12달러로 .com치고는 매우 저렴했다)

<div style='display: flex,flex: wrap'>
<img src="https://user-images.githubusercontent.com/34048253/126061820-33aeb796-379f-4273-ae05-31e890a14a4a.png" height=500 />
<img src="https://user-images.githubusercontent.com/34048253/126061823-c16daeb8-b772-469d-8bdd-fb663113418e.png" height=500 />
</div>

[iitc](https://iitc.app/)라는 지도 관련 오픈소스를 알고 있었기 때문에, 여기에서 쓰이는 코드를 활용하고자 [leaflet](https://leafletjs.com/)이라는 map 라이브러리를 사용해보기로 했고, 기존에 익숙하게 사용하던 react와 요즘 대세?인 recoil을 도입해보기로 했다.
typescript는 지도 라이브러리를 사용하기에 매우 불편했다... 이건 js인가 ts인가...

한국은 현재 국외로 지도반출이 불가하여, 구글맵에서는 9호선 데이터 등의 최신정보가 반영되어 있지 않아서 naver맵이나 kakao맵을 사용해야 했다. 하지만 leaflet과의 연동을 위해서는 lat, lng, zoom 값이 url에 포함된 형태의 restapi를 제공해야 했는데 naver의 경우 header에 authorization용 토큰을 넣어서 전송해주어야 했고, kakao static map api의 경우 zoom level이 반대로 되어있고, lat, lng 표기방식이 osm과 같은 일반적인 open api의 규칙이 아니었기 때문에 leaflet에 적용하기가 힘들어보였다.

![image](https://user-images.githubusercontent.com/34048253/126062310-5a97bc2e-4f1f-4b33-9c3a-f82f0c819d4d.png)

<div style='display: flex,flex: wrap'>
<img src="https://user-images.githubusercontent.com/34048253/126061824-b2a03aba-3a24-4e0a-983a-c833554dc67e.png" height=400 />
<img src="https://user-images.githubusercontent.com/34048253/126062647-f1cf1cc5-7859-44a5-bbe3-a0bfa1d8e395.png" height=500 />
</div>

leaflet 라이브러리를 카카오 맵 api로 교체하였고, maptile도 카카오맵으로 변경했다.
leaflet도 scale bar 등의 여러 편의기능이 있었지만, 카카오맵도 썩 괜찮았다.
네이버맵은 카카오맵에 비해 api doc이 부실해보였고, 동료 프론트 개발자분이 카카오맵을 프로덕트에 적용해두었기 때문에 나도 카카오맵을 쓰는게 유리해보였다.

<img src="https://user-images.githubusercontent.com/34048253/126062711-2664af9c-8864-41c0-af9e-58baa9872b52.png" height=500 />

위에서 언급했던 iitc를 쓰면서 편리하다고 느꼈던 점은, 지도에서 다른 페이지로 넘어가지 않고, 모달이나 사이드바 등을 활용하여 여러가지 기능을 제공하는 점이었다. 장소생성 등 dashboard에서 제공해야 한다고 생각했던 모든 것들을 지도 내에서 개발해보기로 했고, admin dashboard는 정말 최소한의 기능만 포함시키기로 컨셉을 잡았다.

### place
떡볶이를 먹는 장소는 'Place'라는 도메인으로 잡았다.

### brand
가게가 프랜차이즈에 속해있는 경우 'Brand'라는 도메인을 통해 서로 연결된다.
하나의 place는 하나의 brand를 갖는다.
하나의 brand는 여러개의 place를 가질 수 있다.

### hashtag
로컬 떡볶이집만 모아보는 기능을 구현하기 위해 로컬 음식점들은 '로컬'이라는 브랜드를 만들어서 우선 거기에 모아두었다.
하나의 place는 여러개의 hashtag를 갖는다.
하나의 brand는 여러개의 hashtag를 갖는다.

## 현재까지 만든 기능들
~0. leaflet 연동 후 제거~
1. kakao map 기본구성
2. 내 위치로 이동
3. 장소 생성
    - 생성할 위치 클릭시 포크모양 이미지 생성
    - 해시태그 생성/추가 시각적으로 보이게 하기
    - 브랜드 입력시, 이미 생성된 브랜드명 리스트 보여주기
    - 장소 생성시, 새로운 브랜드명이 입력된 경우 브랜드 생성.
 4. 지도상 현재 위치 url로 공유하기 기능 추가
 5. 장소 선택시 modal로 detail정보 보여주기
 6. 모바일에서도 볼수있도록 반응형 적용 (부족하지만...)
 7. google analytics 적용


## 배포
클라이언트는 s3에 업로드 후 cloudfront를 통해 배포.
서버는 홈서버를 통해 배포중이다.
최근 홈서버가 매우 불안정하여 매일매일 상태를 체크하고있다. (1년넘게 문제없던 컴퓨터가 왜인지 종종 꺼지곤 한다...)

[https://ttbkk.com](https://ttbkk.com) 에서 확인할 수 있다.
