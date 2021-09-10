---
layout: post
title:  "리눅스에 구글드라이브 연동하여 백업스토리지로 활용하기"
subtitle: "rclone과 crontab을 활용한 데이터베이스 백업 프로세스"
author: "Siner"
catalog: true
header-mask:  0.3
tags:
    - nas
    - backup
    - database
    - rclone
    - google drive
date:   2021-09-10
multilingual: flase
---

작년에 지인이 rclone, google drive, cloudbox를 활용하여 구글드라이브에 올린 영상을 plex라는 어플로 감상할 수 있도록 구축한 경험을 들은 적이 있었다. (구글드라이브 저장소를 친구들과 공유하고, plex계정도 친구초대가 가능해서 이부분에서 시너지가 폭발하는 듯 했다)

나는 영화도 잘 보지 않았기 때문에 그냥 그런 좋은 활용방법이 있구나 하고 넘어갔었는데, 최근에 홈서버를 새로 구축하면서 데이터베이스의 백업이 제대로 이루어지지 않아 문제가 발생한 적이 있어서 이번에 새로 서버를 구축하면서 rclone과 google drive를 연동하여 리눅스에 마운트된 디렉토리에 주기적으로 database backup을 남겨야겠다는 필요성을 느꼈고, 구축 경험을 남기고자 한다.

## rclone
[homepage](https://rclone.org/) / [github](https://github.com/rclone/rclone)

rclone은 클라우드 저장소와 저장소를 동기화시켜주는 오픈소스 프로젝트이다.

지원하는 클라우드 저장소는 amazon s3, googld drive, dropbox, nextcloud 등등 다양하게 지원하고 있다.
전체 목록은 다음 링크에서 확인이 가능하다.
[https://rclone.org/#providers](https://rclone.org/#providers)

rclone과 우분투20.04의 연결 과정은 다음 블로그를 참고했다.
[https://hoodiejun.tistory.com/19](https://hoodiejun.tistory.com/19)

공식 문서 링크를 보고싶다면 다음 링크를 참고하면 된다.
[https://rclone.org/drive/](https://rclone.org/drive/)

## crontab
주기적으로 데이터베이스의 백업을 하기 위해 다음의 스크립트를 작성했다.

```bash
#!/bin/bash
PASSWORD=
DATE=$(date "+%d")
BACKUP_DIR=/mnt/gdrive/ttbkk-backup
docker exec mysql /usr/bin/mysqldump ttbkk --password=$PASSWORD > $BACKUP_DIR/ttbkk-$DATE.sql
```

crontab이 실행되면 클라우드와 마운트 되어 있는 폴더에 백업파일이 생성되며, 클라우드에도 거의 동시에 파일이 생성된다.

파일은 항상 overwrite되기 때문에 logrotate에서 영감을 받아 백업파일 이름 생성규칙을 `ttbkk-%d.sql`로 정했다.

이렇게 하면 최대 백업파일 개수가 31개가 되기때문에 용량관리를 하지 않아도 되면서 최대 한달동안의 백업데이터를 보유할 수 있다.

![스크린샷 2021-09-10 오후 3 33 34](https://user-images.githubusercontent.com/34048253/132810281-76fc4808-a67f-4169-976f-2a125bfb93d3.png)
![스크린샷 2021-09-10 오후 3 34 40](https://user-images.githubusercontent.com/34048253/132810402-a1d20bd3-0a6d-4211-ab75-d3b73dcb2c23.png)

## 결론
조금 더 사용해봐야 알겠지만 이로 인해 리눅스 저장소에 쉽게 접근 할 수 있게 되어 백업스토리지 말고도 무궁무진한 활용이 가능할 것으로 보인다.<br>
실제 개발은 mac에서 하고 배포는 ubuntu를 통해서 하고있기 때문에 개발서버에 종종 운영서버의 db를 내려받을 일도 있는데, 이런 경우에 sftp를 활용하지 않고도 쉽게 파일을 가져올 수 있어서 조금은 더 편해지지 않았나 싶다.

## reference
- rclone: [https://rclone.org](https://rclone.org)
- rclone 으로 google drive 로컬 드라이브 만들기 (리눅스): [https://hoodiejun.tistory.com/19](https://hoodiejun.tistory.com/19)
- G Suite for Education:  [https://namu.wiki/w/Google%20Workspace#s-5](https://namu.wiki/w/Google%20Workspace#s-5)
