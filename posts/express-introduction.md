---
layout: post
title:  "Express.js란 무엇인가 (Javascript 백엔드 프레임워크)"
subtitle: "Node.js를 위한 빠르고 개방적인 간결한 웹 프레임워크"
author: "Siner"
header-img: "img/post_headers/2020-01-03-express-introduction.png"
catalog: true
header-mask:  0.3
tags:
    - express
    - javascript
    - framework
date:   2020-01-03
multilingual: false
---

참고자료
* [Express In Action](https://www.manning.com/books/express-in-action)<br>
* [Express.js](https://expressjs.com/ko/)<br>

---
* Node.js, Express가 등장한 `배경`과 `철학`에 대해 설명합니다.

---
# 1. JS 엔진과 Node.js
Express에 관해 얘기하기 전에, [Node.js](https://nodejs.org/ko/)에 대한 얘기를 먼저 해야 합니다.<br>
이를 위해 [Express 인액션](https://www.manning.com/books/express-in-action)의 일부 내용을 발췌하였습니다.
>`자바스크립트 엔진` : JS는 주로 웹 브라우저 내에서 사용된다. 웹 페이지의 일부를 수정하기 위한 간단한 스크립트 언어로 시작했지만 애플리케이션과 라이브러리가 가득 찬 복잡한 언어로 성장했다. 모질라와 구글 같은 많은 브라우저 공급자는 자원을 빠른 JS 런타임에 쏟아붓기 시작했고, 그 결과로 브라우저는 더 빠른 JS 엔진을 갖게 되었다.<br>
>`Node.js의 등장` : 2009년에 등장한 Node.js는, 구글 크롬의 강력한 JS 엔진인 V8을 채용했고 서버에서 이를 실행할 수 있도록 했다. 개발자들은 이제 서버 사이드 애플리케이션을 개발할 때 JS도 선택할 수 있게 되었고, 브라우저와 서버 간에 코드를 공유하는 능력 덕분에, Node.js의 인기는 높아졌다. (비동기 코딩 스타일과 많은 라이브러리 또한 큰 장점이다.)<br>
>`Node.js의 단점` : Node.js는 애플리케이션을 만드는 데 필요한 일련의 저수준 기능을 제공한다. 그러나 브라우저 기반의 JS처럼, 저수준의 기능은 장황하고 사용하기 어려울 수 있다.

# 2. Express의 철학

> `jQuery` : Express는 철학적으로 [jQuery](https://jquery.com/)와 유사하다. 사람들은 자신의 웹 페이지에 동적 요소를 추가하고 싶지만, 바닐라 브라우저 API는 복잡하고 기능이 제한적이다. jQuery는 브라우저에 대한 API를 단순화하고 유용한 새로운 기능을 추가해 이러한 장황한 코드를 줄였다. 기본적으로 이게 전부다.<br>
> `Express` : Express도 jQuery와 마찬가지로, Node.js의 API를 단순화하고 유용한 새로운 기능을 추가해 이러한 장황한 코드를 줄였다. Express는 확장성을 지향하기 때문에 불필요한 간섭을 하지 않으며, 서드파티 라이브러리로 쉽게 확장된다.

이제 다음 포스팅에서 진짜 Express를 만나봅시다.
