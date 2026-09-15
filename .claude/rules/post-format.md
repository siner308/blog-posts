---
paths:
  - "posts/**/*.md"
  - "drafts/**/*.md"
---

# 글 형식

## frontmatter

```yaml
---
title: "한국어 제목 (English Term)"
subtitle: "한 줄 훅"
tags:
    - database
date: 2026-09-15
image: https://raw.githubusercontent.com/siner308/blog-posts/master/assets/<slug>-thumbnail.png
---
```

- `title`: 한국어. 영어 용어는 괄호로 병기한다. "가비지컬렉션(Garbage Collection)의 종류와 특징". 시리즈는 "떡볶이맵 제작기 #2 - 에러 모니터링", "쿠버네티스 정리 1. 컴포넌트 종류"처럼 번호를 붙인다. cheat sheet는 "X 명령어 정리 - X cheat sheet".
- `subtitle`: 질문이나 주장 한 줄. "python은 과연 크롤링하기 좋은 언어인가", "ts-node를 production에서 사용하면 안되는 이유". 짧은 명령어 메모형 cheat sheet는 "잘 사용하지 않아서 까먹지만 필요하면 써야하는"을 그대로 쓴다.
- `tags`: 소문자 영어. 기존 태그를 먼저 고른다: `database`, `docker`, `kubernetes`, `container`, `typescript,javascript`, `python`, `django`, `node`, `golang`, `java`, `network`, `http`, `aws`, `serverless`, `storage`, `crawling`, `map`, `ttbkk`, `cheat sheet`, `oop`, `validation`, `monitoring`, `security`, `agile`, `hr`, `ux`, `git`, `terraform`, `iac`, `graphql`, `nestjs`, `dart`, `android`, `hardware`, `investment`, `computer science`. 새 태그는 한 단어로 만든다.
- `date`: `YYYY-MM-DD`.
- `image`: 필수. 새 글은 `assets/<slug>-thumbnail.png`에 넣고 raw URL로 건다. 예전 글의 user-images.githubusercontent.com 링크는 그대로 둔다.
- `draft: true`: 미완성 글에만 붙인다.

## 파일

- `posts/<kebab-slug>.md`. slug는 영어 소문자와 하이픈만.

## 본문 구조

- 최상위 절은 `#`, 하위는 `##`와 `###`. 서론이 있으면 `# 서론`.
- 앞 글을 읽어야 이해되는 글은 첫 줄에 이전 글 링크를 둔다: "# 이전 게시물인 [RDBMS의 특징](url)을 보고 오면 좋습니다."
- 마지막 절은 `# references` 목록. 형식은 `- [제목](url)`.
- 코드 블록에는 언어를 표시한다.
- 스크린샷은 관찰의 근거로 넣고 본문에서 "(위 스크린샷)", "아래 스크린샷처럼"으로 가리킨다. 긴 코드는 `<details>`로 접을 수 있다.
- 표는 옵션, 명령, 증상과 처방처럼 항목이 나열될 때만 쓴다.
