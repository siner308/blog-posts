---
layout: post
title:  "장고(Django)에서 S3 연동하기"
subtitle: "Amazon Web Service S3"
author: "Siner"
header-img: "img/post_headers/2019-07-17-django-aws-s3.png"
catalog: true
header-mask:  0.3
tags:
    - python
    - django
    - aws
    - s3
    - storage
    - cloud
date:   2019-07-17
multilingual: false
---

출처 : [django 에서 S3에 Static, media 파일 저장하고 사용하기](https://blog.leop0ld.org/posts/django-use-s3/)

소셜미디어 프로젝트를 준비하면서, `Static 파일들에 대한 트래픽 부담`을 줄이기 위해 `S3`를 도입하고자 하였다.

Django에서 S3를 연동하기 위해서는 `boto`라는 라이브러리를 사용해야 한다는 것은 이미 알고 있었으나, 정확한 사용 방법을 몰라서 찾아보았고, 나중에도 사용할 때 까먹지 않기 위해 포스팅으로 작성하려 한다.

## 1. 버킷 생성

![image](https://user-images.githubusercontent.com/34048253/61305707-43217300-a826-11e9-9bd0-b8f0e8b318b3.png)
버킷 이름을 작성하고
![image](https://user-images.githubusercontent.com/34048253/61300445-eff6f280-a81c-11e9-9078-925195c7efb4.png)
넘어가고
![image](https://user-images.githubusercontent.com/34048253/61300563-2d5b8000-a81d-11e9-8847-b0a354278906.png)
모든 유저가 읽을 수 있게 Block 설정을 해제해주자
![image](https://user-images.githubusercontent.com/34048253/61305794-6b10d680-a826-11e9-8067-e78a0060493a.png)
최종 확인 후 생성한다
![image](https://user-images.githubusercontent.com/34048253/61305869-8e3b8600-a826-11e9-9739-d4a1377fdc2f.png)
버킷이 생성되었다


## 2. Policy 설정

![image](https://user-images.githubusercontent.com/34048253/61305966-ba570700-a826-11e9-82b1-8ff449ed69b2.png)
Bucket 선택후 `Permissions` - `Bucket Policy` 에서, 아래처럼 내용을 작성한다

```json
{
    "Statement": [
        {
            "Sid": "PublicReadForGetBucketObjects",
            "Effect": "Allow",
            "Principal": {
                "AWS": "*"
            },
            "Action": "s3:GetObject",
            "Resource": "BUCKET-ARN/*"
        },
        {
            "Effect": "Allow",
            "Principal": {
                "AWS": [
                    "USER-ARN"
                ]
            },
            "Action": "s3:*",
            "Resource": [
                "BUCKET-ARN",
                "BUCKET-ARN/*"
            ]
        }
    ]
}
```
`BUCKET-NAME`은 Bucket Policy 설정페이지에서 확인이 가능하고,<br>
`USER-ARN`은 [https://console.aws.amazon.com/iam/home#/users](https://console.aws.amazon.com/iam/home#/users)에서 유저를 선택하여 확인이 가능하다. (아래 예시)
![image](https://user-images.githubusercontent.com/34048253/61302315-85e04c80-a820-11e9-8345-2629c4a83d33.png)

## 3. Access Key 발급

[https://console.aws.amazon.com/iam/home#/users](https://console.aws.amazon.com/iam/home#/users)에서 키를 발급하고자 하는 유저를 선택한다.<br>
 `security_credentials`탭에 들어가서 `Create access key`버튼을 클릭하여 키를 발급받는다.
![image](https://user-images.githubusercontent.com/34048253/61304122-9ba34100-a823-11e9-9833-14ef46350f53.png)

## 4. Django 설정
```python
''' .envs/.django
docker-compose의 env_file을 통하여 불러오도록 설정했다.
직접 settings.py에 아래의 데이터를 넣어도 되지만, 다음과 같이 분리하는 것을 추천한다.
'''

AWS_REGION=ap-northeast-2
AWS_STORAGE_BUCKET_NAME=sinerbucketforblog
AWS_ACCESS_KEY_ID=xxxxxxxxxxxxxxxxxxxxxxx
AWS_SECRET_ACCESS_KEY=xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx
```

```python
''' config/settings.py
S3와 연관없는 내용은 생략.
'''

# AWS Setting
AWS_REGION = env('AWS_REGION')
AWS_STORAGE_BUCKET_NAME = env('AWS_STORAGE_BUCKET_NAME')
AWS_ACCESS_KEY_ID = env('AWS_ACCESS_KEY_ID')
AWS_SECRET_ACCESS_KEY = env('AWS_SECRET_ACCESS_KEY')

AWS_QUERYSTRING_AUTH = False
AWS_S3_HOST = 's3.%s.amazonaws.com' % AWS_REGION
AWS_S3_CUSTOM_DOMAIN = '%s.s3.amazonaws.com' % AWS_STORAGE_BUCKET_NAME

# Static Setting
STATIC_URL = "https://%s/" % AWS_S3_CUSTOM_DOMAIN
STATICFILES_STORAGE = 'storages.backends.s3boto.S3BotoStorage'

# Media Setting
MEDIA_URL = "https://%s/" % AWS_S3_CUSTOM_DOMAIN
DEFAULT_FILE_STORAGE = 'storages.backends.s3boto.S3BotoStorage'

# Root Setting
STATIC_ROOT = '%s/static' % STORAGE_PATH
MEDIA_ROOT = '%s/media' % STORAGE_PATH
```

## 5. boto 설치

Django에서 boto를 사용할 수 있게 pip 패키지인 `boto`를 설치해주자.
```bash
pip install boto
```

## 6. collectstatic

collectstatic 진행 후 버킷을 살펴보면 static 파일들이 생성된 것을 확인할 수 있다.

![image](https://user-images.githubusercontent.com/34048253/61306220-2b96ba00-a827-11e9-959e-bac1bd314242.png)

## 7. CORS

admin 페이지에 들어가보면 평소와는 살짝 다른 것 같다.
![image](https://user-images.githubusercontent.com/34048253/61306508-b5df1e00-a827-11e9-8ba3-778418c83e28.png)
확인해보니 폰트를 제대로 불러오지 못하고있다.
![image](https://user-images.githubusercontent.com/34048253/61306637-f179e800-a827-11e9-8314-f9929406b58a.png)

콘솔 확인결과 CORS로 판명. S3에서 해당 처리를 해주도록 하자.
![image](https://user-images.githubusercontent.com/34048253/61307877-0eafb600-a82a-11e9-9828-106555a89451.png)
Bucket 선택후 `Permissions` - `CORS configuration` 에서, 아래처럼 내용을 작성한다

```xml
<!-- Sample Policy -->
<CORSConfiguration>
        <CORSRule>
                <AllowedOrigin>*</AllowedOrigin>
                <AllowedMethod>GET</AllowedMethod>
                <MaxAgeSeconds>3000</MaxAgeSeconds>
                <AllowedHeader>Authorization</AllowedHeader>
        </CORSRule>
</CORSConfiguration>
```

## 8. 결과

![image](https://user-images.githubusercontent.com/34048253/61307238-ef645900-a828-11e9-860c-21eb3bf9f7ab.png)
이제 제대로 된다
