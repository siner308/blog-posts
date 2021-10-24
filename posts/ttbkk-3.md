---
layout: post
title:  "떡볶이맵 제작기 #3 - 지도 API Pagination 적용기"
subtitle: "4000개의 지도 데이터를 느려보이지 않게 불러와서 로딩하기"
author: "Siner"
catalog: true
header-mask:  0.3
tags:
- api
- ttbkk
- ux
date:   2021-10-22
multilingual: flase
---

떡볶이맵의 데이터가 점점 많아지면서 marker cluster용 api를 구축해야 했지만, 어떠한 라이브러리가 좋은지 파악하지 못했고 (내가 직접 구현할 수도 있고) best practice를 공부하지 못했기 떄문에 해당 작업을 잠시 뒤로 미루게 되었다.

하지만 전국 떡볶이 데이터가 1000개를 넘어가던 시점부터 쎄한 느낌이 들었고, 응답까지 상당히 많은 시간이 소요됐기 때문에 지도에 데이터가 늦게 떠서 상당히 불편했다. 페이지네이션을 적용한 이후인 현재 데이터를 4000개정도이다.

![스크린샷 2021-10-22 오후 10 59 00](https://user-images.githubusercontent.com/34048253/138466775-b360166d-c345-478a-95ce-b437de512469.png)

나름 기발한?생각으로 장소 데이터를 가져오는 api에 페이지네이션을 적용해보기로 했다.

테이블 형태의 페이지네이션 api라면 한번의 요청에서 limit만큼의 데이터를 받으면 될 일이지만, 지도에서는 모든 데이터를 불러와야 하기 때문에 아래와 같은 api 시나리오를 구성했다.

1. 해당 지도 bound내의 count를 가져오는 api를 호출한다.
2. 1번페이지부터 count/limit를 반올림한 수의 페이지까지의 데이터를 호출한다.

만약 bound내의 데이터가 355개이고 limit을 100으로 가정한다면 아래와 같다.

```text
1. /count 로 요청하여 355라는 값을 얻는다.
2-1. /places?page=1&limit=100 으로 요청하여 100개의 데이터를 가져오고, 화면에 뿌려준다.
2-2. /places?page=2&limit=100 으로 요청하여 100개의 데이터를 가져오고, 화면에 뿌려준다.
2-3. /places?page=3&limit=100 으로 요청하여 100개의 데이터를 가져오고, 화면에 뿌려준다.
2-4. /places?page=4&limit=100 으로 요청하여 55개의 데이터를 가져오고, 화면에 뿌려준다.
```

<div>
  <video width=100% controls title="API 적용 결과 (서울)">
    <source src="https://user-images.githubusercontent.com/34048253/138471049-92df819a-c880-4a4c-b574-9f4479a3bb90.mov">
    [구현 비디오 링크](https://user-images.githubusercontent.com/34048253/138471049-92df819a-c880-4a4c-b574-9f4479a3bb90.mov)
  </video>
  
  <video width=100% controls title="API 적용 결과 (전국)">
    <source src="https://user-images.githubusercontent.com/34048253/138471152-6e14891e-6efe-404b-b1cd-938f3ce903f8.mov">
    [구현 비디오 링크](https://user-images.githubusercontent.com/34048253/138471152-6e14891e-6efe-404b-b1cd-938f3ce903f8.mov)
  </video>
</div>

---

이렇게 api를 쪼개서 유저가 기다리는 시간을 조금 낮출 수 있었지만, request 횟수가 증가한 만큼 서버에 부하도 많아지게 되어, network error 로그가 조금씩 남고있다.
결국 데이터가 많아지면 cluster api로 변경해야하는건 불가피한 것이다.
