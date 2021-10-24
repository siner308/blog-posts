---
layout: post
title:  "REST에 대하여"
subtitle: "REST API는 대체 무엇이고, Restful 하다는 건 대체 무엇인가"
author: "Siner"
header-img: "img/post_headers/2020-01-27-what-is-rest.png"
catalog: true
header-mask:  0.3
tags:
    - http
    - architecture
date:   2020-01-27
multilingual: false
---

참고자료
* [Representational state transfer](https://en.wikipedia.org/wiki/Representational_state_transfer)
* [HTTP Status Codes](https://httpstatuses.com/)
* [REST API의 이해와 설계 - 조대협의 블로그](https://bcho.tistory.com/953?category=252770)

---

## 1. Intro
2000년 Roy Fielding의 논문에서 처음 정의된 `REST`는 소프트웨어 아키텍쳐 스타일로, 아래와 같이 간단하게 설명할 수 있습니다.

> 자원(Method)을 정의하고<br>
> 자원에 대한 주소(URI)를 지정하여<br>
> 전송(Request)하면,<br>
> 다음 자원에 대한 표현이 최종 사용자에게 전송(Response)된다.

* REST 규칙을 따르는 API는 Method와 URI만으로 이 요청의 의도와, 결과를 어느정도 예상할 수 있게 합니다.
* REST 아키텍쳐를 적용한 웹서비스를 `RESTful Web Service` 라고 부르고, 이러한 아키텍쳐는 클라이언트와 서버간에 [상호운용성](https://ko.wikipedia.org/wiki/%EC%83%81%ED%98%B8%EC%9A%B4%EC%9A%A9%EC%84%B1) 있는 데이터 통신을 가능케 합니다.
* [Stateless 프로토콜](https://ko.wikipedia.org/wiki/%EB%AC%B4%EC%83%81%ED%83%9C_%ED%94%84%EB%A1%9C%ED%86%A0%EC%BD%9C)을 사용하기 때문에, Restful 시스템은 `성능`, `안정성`, `컴포넌트 재사용에 의한 확장성`을 목표로 합니다.

---

## 2. Methods
서버의 자원에 보내는 요청은 `Create`, `Read`, `Update`, `Delete` 네가지 성질에 따라 분류할 수 있습니다.
각 목적에 맞는 메서드를 선택하여 요청하면 됩니다.

| 메서드   | 의미              | Idempotent |
|--------|------------------|------------|
| POST   | Create           | No         |
| GET    | Read             | Yes        |
| PUT    | Update (replace) | Yes        |
| PATCH  | Update (modify)  | No         |
| DELETE | Delete           | Yes        |

* [idempotent](https://ko.wikipedia.org/wiki/%EB%A9%B1%EB%93%B1%EB%B2%95%EC%B9%99) : 요청을 반복하더라도 결과가 달라지지 않는 성질

---

## 3. Status Codes

[HTTP 상태 코드](https://developer.mozilla.org/ko/docs/Web/HTTP/Status)는 특정 HTTP 요청이 성공적으로 완료되었는지 알려줍니다.
응답은 5개의 그룹으로 나누어집니다. 자세한 내용은 [section 10 of RFC 2616](https://tools.ietf.org/html/rfc2616#section-10)에 정의되어 있습니다.

* 1xx : 정보 전달
* 2xx : 성공적인 응답
* 3xx : 리다이렉트
* 4xx : 클라이언트 에러
* 5xx : 서버 에러

몇몇 서비스는 모든 Status Code가 아닌 Error Code에 대한 레퍼런스만 존재하였습니다.
대표적인 API 서비스들이 사용하는 Error Code는 아래와 같습니다.

|서비스|Error Codes|
|---|---|
|[Google Cloud](https://cloud.google.com/storage/docs/json_api/v1/status-codes)      |400 401 403 404 409 410 500                        |
|[Naver Open API](https://developers.naver.com/docs/common/openapiguide/errorcode.md)|400 401 403 404 405 429 500                        |
|[Twitter](https://developer.twitter.com/en/docs/basics/response-codes)              |400 401 403 404 406 410 420 422 429 500 502 503 504|
|[Twitch](https://dev.twitch.tv/docs/v5#errors)                                      |400 401 403 404 422 429 500 503                    |
|[Riot](https://developer.riotgames.com/docs/portal#web-apis_response-codes)         |400 401 403 404 415 429 500 503                    |

## 4. Examples


#### POST

###### 1. 사용자 생성
`이름이 안정현이고, 국적이 한국인 사용자를 생성한다` 라는 request는 아래와 같이 표현이 가능합니다.

**Request**

```text
POST http://mysite/users
```
```json
{
    "country": "한국",
    "name": "안정현"
}
```

**Response**

```text
HTTP/1.1 201 Created
```
```json
{
    "user": {
        "id": 8,
        "country": "한국",
        "name": "안정현",
        "age": 29
    }
}
```

#### GET

###### 1. 전체 조회
`전체 사용자 데이터를 조회한다` 라는 요청은 아래와 같이 표현이 가능합니다.

**Request**

```text
GET http://mysite/users
```

**Response**

```text
HTTP/1.1 200 OK
```
```json
{
    "users": [
        {
            "id": 1,
            "country": "미국",
            "name": "마리오",
            "age": 24
        },
        {
            "id": 6,
            "country": "한국",
            "name": "펭수",
            "age": 10
        },
        {
            "id": 8,
            "country": "한국",
            "name": "안정현",
            "age": 29
        }
    ]
}
```

###### 2. 개별 조회
`id가 6인 사용자를 조회한다` 라는 요청은 아래와 같이 표현이 가능합니다.

**Request**

```text
GET http://mysite/users/8
```

**Response**

```text
HTTP/1.1 200 OK
```
```json
{
    "user": {
        "id": 8,
        "country": "한국",
        "name": "안정현",
        "age": 29
    }
}
```

###### 3. 검색

`국적이 한국인 전체 사용자를 조회한다`와 같이 검색에 대한 요청은 알아보기 쉽게 표현이 가능합니다.

**Request**
```text
GET http://mysite/users?country=한국
```

**Response**

```text
HTTP/1.1 200 OK
```
```json
{
    "users": [
        {
            "id": 6,
            "country": "한국",
            "name": "펭수",
            "age": 10
        },
        {
            "id": 8,
            "country": "한국",
            "name": "안정현",
            "age": 29
        }
    ]
}
```

#### PUT
PUT 메서드는 수정보다는 **해당 id의 자원을 지금 보내는 자원으로 대체한다** 라는 느낌에 더 가깝습니다. 변경하고싶지 않은 자원이 있다면, 기존의 자원을 그대로 전송해 주면 됩니다.

`id가 6인 사용자의 정보를 변경한다`라는 request는 아래와 같이 표현이 가능합니다.

**기존 데이터**
```json
{
    "id": 6,
    "country": "한국",
    "name": "펭수",
    "age": 10
}
```

**Request**

```text
PUT http://mysite/users/6
```
```json
{
    "country": "남극",
    "name": "웨에오오오옭"
}
```

**Response**

```text
HTTP/1.1 200 OK
```
```json
{
    "user": {
        "id": 6,
        "country": "남극",
        "name": "웨에오오오옭",
        "age": null
    }
}
```

#### PATCH
`id가 8인 사용자의 이름을 김지윤으로, 나이를 26으로 수정한다`라는 request는 아래와 같이 표현이 가능합니다.

**기존 데이터**
```json
{
    "id": 8,
    "country": "한국",
    "name": "안정현",
    "age": 29
}
```

**Request**
```text
PATCH http://mysite/users/8
```
```json
{
    "name": "김지윤",
    "age": 26
}
```

**Response**
```text
HTTP/1.1 200 OK
```
```json
{
    "user": {
        "id": 8,
        "country": "한국",
        "name": "김지윤",
        "age": 26
    }
}
```

#### DELETE
`id가 1인 사용자를 제거한다`라는 request는 아래와 같이 요청이 가능합니다.

**Request**

```text
DELETE http://mysite/users/1
```

**Response**

```text
HTTP/1.1 200 OK
```
