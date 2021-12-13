---
title: "[번역] Dockerfile 레퍼런스 (3) - Format"
subtitle: "가독성을 높이는 명령어 포맷"
tags:
    - docker
date: 2020-03-29
---

[공식 레퍼런스](https://docs.docker.com/engine/reference/builder/)를 토대로 작성되었습니다.

---

# Format
Dockerfile의 포맷은 다음과 같습니다.
```bash
# Comment
INSTRUCTION arguments
```
Instruction은 대문자 소문자를 가리지는 않습니다, 하지만 이러한 컨벤션을 지킴으로써, arguments를 좀더 쉽게 파악할 수 있습니다.
도커는 `Dockerfile`에 있는 instruction을 순서대로 실행시킵니다. `Dockerfile`은 **반드시 \`FROM\`명령어로 시작해야합니다.**
FROM 명령어 이전에 [parser directives](#parser-directives), [comments](#format), [ARG](#arg)를 사용할 수도 있습니다.

* parser directives, comments, ARG에 대한 번역은 추후 작성될 예정입니다.
