---
title: "Express.js의 미들웨어(Middleware)"
subtitle: "Express 애플리케이션은 기본적으로 일련의 미들웨어 함수 호출입니다"
tags:
    - node
    - typescript,javascript
date: 2020-01-04
image: https://user-images.githubusercontent.com/34048253/155849505-13269ce9-84ba-4541-994d-1ece20a746fa.png
---

참고자료
* [Express In Action](https://www.manning.com/books/express-in-action)<br>
* [Express.js](https://expressjs.com/ko/)<br>

---
* Express의 `미들웨어`에 대해 다룹니다.

---
# 1. Intro
![image](https://user-images.githubusercontent.com/34048253/71705012-b04eb200-2e20-11ea-8a2e-b5896c6c7ca5.png)
> Express는 자체적인 최소한의 기능을 갖춘 라우팅 및 미들웨어 웹 프레임워크이며, Express 애플리케이션은 기본적으로 `일련의 미들웨어 함수 호출`입니다.<br><br>
> 미들웨어 함수는 [요청 오브젝트](https://expressjs.com/ko/4x/api.html#req)(req), [응답 오브젝트](https://expressjs.com/ko/4x/api.html#res)(res), 그리고 애플리케이션의 요청-응답 주기 중 그 다음의 미들웨어 함수 대한 액세스 권한을 갖는 함수입니다. 그 다음의 미들웨어 함수는 일반적으로 next라는 이름의 변수로 표시됩니다. (next는 생략이 가능합니다.)<br><br>
> 미들웨어의 `로드 순서`는 중요하며, 먼저 로드되는 미들웨어 함수가 먼저 실행됩니다.

```javascript
var express = require('express');
var app = express();

app.get('/', function (req, res) {
  res.send('Hello World!');
});

app.listen(3000);
```
위 코드는 Express의 가장 간단한 Hello World의 예시이며, 아래에서 이에 대한 `두가지 미들웨어(logger, requestTime)`를 다룰 예정입니다.

# 2-1. 예시 (Logger)
**앱이 요청을 수신할 때마다, LOGGED라는 메시지를 터미널에 출력하는 미들웨어를 만들어봅시다.**

```javascript
var express = require('express');
var app = express();

var myLogger = function (req, res, next) {
  console.log('LOGGED');
  next();
};

app.use(myLogger); // 여기서 미들웨어를 호출!!

app.get('/', function (req, res) {
  res.send('Hello World!');
});

app.listen(3000);
```

myLogger라는 함수(미들웨어)는 단순히 메시지를 출력한 후,<br>
`next()` 함수를 호출하여 스택 내의 그 다음 미들웨어 함수에 요청을 전달합니다.

이 함수(미들웨어)를 로드하려면, `app.use()`의 인자로 myLogger를 넣은 후, 호출합니다.

루트 경로(/)로 `라우팅하기 전에` 미들웨어 함수를 로드해야 합니다.<br>
루트 경로에 대한 라우팅 이후에 myLogger가 로드되면, 루트 경로의 라우트 핸들러가 요청-응답 주기를 종료(**res.send**)하므로 요청은 절대로 myLogger에 도달하지 못하며 앱은 **LOGGED**를 출력하지 않습니다.

# 2-2. 예시 (requestTime)
**앱의 루트에 대한 요청을 실행할 때, 타임스탬프를 브라우저에 표시하도록 미들웨어를 추가합니다.**

```javascript
var express = require('express');
var app = express();

var requestTime = function (req, res, next) {
  req.requestTime = Date.now();
  next();
};

app.use(requestTime);

app.get('/', function (req, res) {
  var responseText = 'Hello World!';
  responseText += 'Requested at: ' + req.requestTime + '';
  res.send(responseText);
});

app.listen(3000);
```

이제 앱은 requestTime 미들웨어 함수를 사용합니다. 또한 루트 경로 라우트의 콜백 함수는 `미들웨어 함수가 req(요청 오브젝트)에 추가하는 특성`을 사용합니다.

이와같이 Express는 **일련의 미들웨어 함수들을 호출**하는것으로 요청에대한 응답을 만들어냅니다.

# 3-1. 에러 핸들러로 에러 보내기
![image](https://user-images.githubusercontent.com/34048253/71709162-f152c080-2e38-11ea-96f1-f43d057d5665.png)
>에러가 발생하지 않는다면, 에러를 핸들링하는 미들웨어를 지나칩니다.

![image](https://user-images.githubusercontent.com/34048253/71709085-b05aac00-2e38-11ea-9eef-c40337e7a73f.png)
>에러가 발생한다면, 그 즉시 에러 핸들러로 이동하게 됩니다.

에러 핸들러를 동작시키는 방법은 아래와 같이 두가지가 있습니다.<br>
(아래의 코드는 에러 핸들러가 아닌, `에러 핸들러로 가기 이전 단계의 코드`입니다.)

1. throw를 통해 Error를 발생시켜서 전달.

```javascript
app.get('/', function (req, res) {
  throw new Error('BROKEN') // 에러가 발생하면, Express가 알아서 캐치합니다!
})
```

2. next 함수의 인자로 넘겨서 전달.

```javascript
app.get('/', function (req, res, next) {
  fs.readFile('/file-does-not-exist', function (err, data) {
    if (err) {
      next(err) // Express의 에러핸들러로 이동합니다.
    } else {
      res.send(data)
    }
  })
})
```

# 3-2. 에러 핸들러 만들기
이제 에러 핸들러를 직접 만들어봅시다.<br>
에러 핸들러는 req, res, next 에 `err`라는 인자를 추가로 받습니다. (실제로는 err 인자가 `첫번째로` 위치하게 됩니다.)

```javascript
var bodyParser = require('body-parser')

function logErrors (err, req, res, next) {
  console.error(err.stack)
  next(err)
}

function clientErrorHandler (err, req, res, next) {
  if (req.xhr) {
    res.status(500).send({ error: 'Something failed!' })
  } else {
    next(err)
  }
}

function errorHandler (err, req, res, next) {
  res.status(500)
  res.render('error', { error: err })
}

app.use(bodyParser.urlencoded({
  extended: true
}));
app.use(bodyParser.json());

app.get('/', (req, res, next) => {
  PaidContent.find(function (err, doc) {
    if (err) {
      return next(err);
    } else {
      res.json(doc);
    }
  })
});

app.use(logErrors);
app.use(clientErrorHandler);
app.use(errorHandler);
```

에러 핸들러는 모든 미들웨어 중 `가장 아래에` 배치시킵니다.

아래 두장의 이미지는 Express 미들웨어를 가장 잘 설명하고 있습니다.

![image](https://user-images.githubusercontent.com/34048253/71705274-1daf1280-2e22-11ea-9716-393c2430ae0c.png)
[Node - Express 커스텀 미들웨어 만들기](https://backback.tistory.com/333)
![image](https://user-images.githubusercontent.com/34048253/71705279-2bfd2e80-2e22-11ea-91cc-b4848d79f9d9.png)
[Express Middlewares, Demystified](https://medium.com/@viral_shah/express-middlewares-demystified-f0c2c37ea6a1)

다음 포스팅에서는 라우팅에 대해 다룰 예정입니다.
