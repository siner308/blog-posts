---
title: "[번역] Dockerfile 레퍼런스 (2) - Buildkit"
subtitle: "동시성을 가지며, 캐시 효율적이며 Dockerfile에 독립적인 빌더 툴킷"
tags:
    - docker
date: 2020-03-29
image: https://user-images.githubusercontent.com/34048253/155849229-de13c0fb-a354-4129-96f7-96f346bcb56d.png
---

[공식 레퍼런스](https://docs.docker.com/engine/reference/builder/)를 토대로 작성되었습니다.

---

# BuildKit
도커 버전 18.09부터, [moby/buildkit](https://github.com/moby/buildkit)를 통한 빌드를 지원합니다. Buildkit 백엔드는 기존 빌드방식과 비교해 많은 이점이 있습니다.
- 사용하지 않는 빌드 단계를 탐지하여 실행하지 않음
- 독립적인 빌드 스테이지의 병렬화
- 변경된 context만 전송 
- context안의 사용하지 않는 파일을 탐지하여 전송하지 않음
- 새로운 기능들과 함께 외부 도커파일 사용
- 중간 이미지 및 컨테이너 관련 API 사용함에 있어서 생기는 부작용 방지
- 자동 정리를 위한 캐시 우선순위 지정

Buildkit 백엔드를 사용하려면 `docker build` 명령어를 사용하게 전에 CLI를 통해 `DOCKER_BUILDKIT=1`이라는 환경변수를 지정해야합니다.

Buildkit 기반의 빌드를 가능하게 하는 실험적인 도커파일 문법은 [BuildKit 저장소의 문서](https://github.com/moby/buildkit/blob/master/frontend/dockerfile/docs/syntax.md#dockerfile-frontend-syntaxes)를 참조해주세요.
