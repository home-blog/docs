# Tasks: 글 카드 썸네일과 글결 마크 (v0.4.0)

> 2026-10-12: 코드 브랜치 `feat/v0.4-thumbnail-cards` → 코드 PR #50 **merge** (2026-10-12, 넓은 화면 채우기, 머리글 메뉴 다듬기 포함). CodeRabbit 지적으로 코드 블록·인라인 코드 안의 이미지 문법은 썸네일·대표 후보에서 뺐다. 소셜 로그인(N-07)과 같은 v0.4.0에 묶는다.

**Input**: [spec.md](spec.md)

- [x] T001 [US1] 서버: `PostThumbnail`(제목 "대표"인 이미지, 없으면 첫 이미지. 이 사이트 경로와 http(s)만) → 글 모음·인기 글·블로그 목록·검색·태그 목록 응답의 `thumbnail`. OG 카드 그림도 같은 규칙 (테스트). 표 칸은 더하지 않는다: 대표 표시는 본문 이미지의 제목 칸 `![설명](주소 "대표")`
- [x] T002 [US1] 화면: `Thumbnail`(테이프 붙인 사진 / 첫 문장 원고지, 주제별 칸 선 색, 불러오기 실패 시 원고지) → 첫 화면 카드
- [x] T003 [US2] 화면: `FeaturedPost` 첫 페이지 맨 위 큰 카드
- [x] T004 [US3] 화면: `PostRow` 오른쪽 작은 썸네일 (블로그 목록·검색·태그)
- [x] T005 [US4] 화면: 글쓰기 `CoverPicker` (본문 이미지 고르기 → 본문의 "대표" 표시를 옮긴다)
- [x] T006 [US5] 예시 글 사진: `deploy/seed/photos.json` → `demo-photos.sql`(몇 번 실행해도 같음). 개발 서버는 켤 때마다 맞추고, 배포 DB는 Actions `예시 데이터` → `photos`. 사진은 Lorem Picsum(Unsplash 사진, 무료) 주소를 그대로 건다 (가안)
- [x] T007 [US6] 마크: `favicon.svg`, 머리글 `BrandMark`(다크 모드 색), `apple-touch-icon.png`, `og-default.png`
- [x] T008 확인: `mvn verify`, `npm run lint`, `npm run build`, Playwright로 첫 화면(밝게·어둡게·휴대폰)·블로그 목록·글쓰기 대표 고르기 화면 (클라우드 세션은 picsum에 못 나가 사진 자리에 대신 그림)
- [ ] T009 사람 확인: 사용자가 로컬(5173)에서 확인 (SC-2). 마크 시안(E·F·H) 가운데 F로 넣음, 바꾸고 싶으면 고친다
- [ ] T010 배포 뒤: Actions `예시 데이터` → `photos`로 배포 DB 예시 글에 사진 넣기
