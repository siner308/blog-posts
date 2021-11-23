---
layout: post
title:  "class-validator를 사용한 타입스크립트(Typescript) Validation"
subtitle: "Typescript Data Validation"
author: "Siner"
header-img: "img/post_headers/2019-12-17-typescript-class-validator.png"
catalog: true
header-mask:  0.3
tags:
    - typescript/javascript
    - validation
date:   2019-12-17
multilingual: false
---

참고자료 : [**typestack/class-validator**](https://github.com/typestack/class-validator)

`Express`를 사용하는 `Typescript` 환경에서 `class-validator`를 사용하여 `Request Data`를 `Validation`하는 과정을 다루고 있습니다.

---

## 1. Data Validation

>[Data Validation(데이터 유효성 검사)](https://en.wikipedia.org/wiki/Data_validation)이란,<br>
>다루는 데이터가 올바른 `Format`을 가지는지 확인하는 과정입니다.

## 2. class-validator
>[typestack/class-validator](https://github.com/typestack/class-validator)<br>
>데코레이터 및 비 데코레이터 기반 검증을 사용할 수 있습니다.<br>
>내부적으로 [validator.js](https://github.com/chriso/validator.js)를 사용하여 유효성 검사를 수행합니다.<br>
>class-validator는 브라우저 및 node.js 플랫폼 모두에서 작동합니다.

```bash
npm install class-validator --save
```

## 3. Format
정상적인 데이터는 아래의 `Format`을 충족해야 한다고 설정해봅시다.
```yaml
title      : string (minLength=10, maxLength=20)
text       : string (hello 를 포함해야함)
rating     : number (minValue=0, maxValue=10)
email      : string (Email)
site       : string (FQDN)
userId     : string (UUID ver.4)
createDate : date
```

## 4. Sample Data
정상적인 Request Data `Sample`은 아래와 같습니다.

```json
{
  "title": "My Best Title",
  "text": "this is a great post about hello world!!",
  "rating": 8,
  "email": "siner308@gmail.com",
  "site": "siner308.github.io",
  "userId": "1fb0e256-ca80-4e2d-9528-ac63bedaede8",
  "createDate": "2019-12-17",
}
```

## 5. Validator Class
이제 class-validator에서 사용할 `Class`를 작성합니다.<br>
~~도입 초기에 Typeorm의 Entity와 헷갈리는 바람에 고생을 했습니다...~~

```typescript
import { IsUUID, Contains, IsInt, Length, IsEmail, IsFQDN,
  IsDate, Min, Max } from "class-validator";

export class Post {

  @Length(10, 20)
  public title: string;

  @Contains('hello')
  public text: string;

  @IsInt()
  @Min(0)
  @Max(10)
  public rating: number;

  @IsEmail()
  public email: string;

  @IsFQDN()
  public site: string;

  @IsUUID('4')
  public userId: string;

  @IsDate()
  public createDate: Date;

}

```

## 6. Handling

데이터 처리 로직은 아래처럼 구현합니다.

```typescript
// postController.ts

import { validate } from 'class-validator';
import { Request, Response, NextFunction, Router } from 'express';
import { PostService, IPost } from '../services/PostService';

const postRouter: Router = Router();

postRouter.post('/post', async (req: Request, res: Response, next: NextFunction) => {
  const title: string = req.body.title;
  const text: string = req.body.text;
  const rating: number = req.body.rating;
  const email: string = req.body.email;
  const site: string = req.body.site;
  const userId: string = req.body.userId;
  const createDate: Date = new Date(req.body.createDate);

  // validation
  const post: Post = new Post();
  post.title = title;
  post.text = text;
  post.rating = rating;
  post.email = email;
  post.site = site;
  post.userId = userId;
  post.createDate = createDate;

  const errors: ValidationError[] = await validate(post); // errors is an array of validation errors
  if (errors.length > 0) {
    console.log('validation failed. errors: ', errors);
    return next(errors);
  } else {
    console.log('validation succeed');
  }
    
  // continuous logic
  const postService: PostService = new PostService();
  try {
    const createdPost: IPost = await postService.createPost(title, text, rating, email, site, userId, createdDate);
    res.status(200).json({
      msg: 'created',
      alertPanel: createdPost,
    });
  } catch (err) {
      res.status(400).json({
        msg: err.message,
      });
  }
})

export default postRouter;
```

## 7. Valid Case
![image](https://user-images.githubusercontent.com/34048253/71055139-e1c84c00-2197-11ea-88cb-7df60c16b8f7.png)

```bash
validation succeed
```

정상 데이터의 경우 아래와 같이 성공 로그가 출력됩니다.

## 7. Invalid Case

![image](https://user-images.githubusercontent.com/34048253/71055039-81390f00-2197-11ea-8fbe-39582a0aec75.png)

![image](https://user-images.githubusercontent.com/34048253/71055830-63b97480-219a-11ea-9dd6-d37220a25b1a.png)

![image](https://user-images.githubusercontent.com/34048253/71055071-a4fc5500-2197-11ea-89e3-b0b4dad1a0db.png)

유효하지 않은 데이터가 들어왔을 경우, 콘솔에서 Validation에 통과되지 않은 `원인`을 확인할 수 있습니다.
