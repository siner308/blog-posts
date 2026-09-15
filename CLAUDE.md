# blog-posts

blog.siner.io에 올라가는 글 원고. `master`에 push하면 GitHub Actions가 Netlify build hook을 호출해 배포된다. 블로그 구현부는 [siner308/blog](https://github.com/siner308/blog).

## 구조

- `posts/<kebab-slug>.md`: 발행 글. frontmatter는 `.claude/rules/post-format.md`를 따른다.
- `drafts/`: 초고.
- `assets/<slug>-thumbnail.png`: 썸네일. 글의 `image:`가 raw.githubusercontent.com URL로 가리킨다.

## 화법 기준

어투는 `.claude/rules/voice.md`, 형식은 `.claude/rules/post-format.md`에 있다. 두 규칙은 아래 글에서 뽑았으니 판단이 애매하면 이 글들을 다시 읽고 맞춘다.

- 설명 글: `change-data-capture.md`, `container-ecosystem.md`, `kubernetes-1.md`, `node-vs-ts-node.md`, `what-is-rest.md`, `garbage-collection.md`
- 회고와 제작기: `production-level-crawling.md`, `npm-vs-yarn-vs-pnpm.md`, `ttbkk-1.md`, `ttbkk-2.md`, `ttbkk-3.md`, `rclone-google-drive-crontab.md`

화법 기준으로 삼지 않는 글:

- `chip-isa-ecosystem.md`: AI가 쓴 글이라 저자의 어투가 아니다.
- 제목이 `[번역]`으로 시작하는 글 6편: 원문의 어투다.
- `redis-cheat-sheet.md`, `terraform-cheat-sheet.md`, `git-cheat-sheet.md`: 문장이 거의 없는 명령어 메모라 cheat sheet 형식의 기준으로만 쓴다.

## 작업 규칙

- 검토를 요청받으면 고치지 않고 어긋난 문장을 인용해서 보여준다. 수정은 따로 요청받은 뒤에 한다.
- 발행 전 `image:`와 `assets/` 썸네일이 있는지 확인한다.
