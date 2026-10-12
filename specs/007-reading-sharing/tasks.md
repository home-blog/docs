# Tasks: 읽기 편하게·널리 퍼지게 (v0.3.0)

> 2026-10-12: T001~T011 코드 브랜치 `feat/v0.3-reading-sharing`(커밋 e0a7e6f)에 끝. 사용자 요청으로 **카드 디자인**(원고지 칸 띠, 떠오르기, 제목 형광펜, 차례로 떠오르는 목록)도 같이 넣었다. 사용자가 로컬에서 확인한 뒤 코드 PR #49로 merge(e8652a7)하고, 같은 날 "배포해줘"로 배포했다(Actions `배포` 손 실행, 성공). Nginx 이미지 빌드는 클라우드 세션에서 npm ci가 막혀, 같은 설정 파일을 nginx:1.27-alpine에 붙여 로봇 User-Agent로 확인했다.

**Input**: [spec.md](spec.md), [plan.md](plan.md) · 코드 브랜치 `feat/v0.3-reading-sharing`

- [x] T001 [US3] 서버: `ManuscriptMeasure` + 설정 `post.manuscript.*`, 글 상세·블로그 글 목록·첫 화면 글 모음 응답에 `sheets`, `readMinutes` (테스트)
- [x] T002 [US3] 화면: 상세·목록·첫 화면에 "원고지 N매 · M분"
- [x] T003 [US1] 화면: 다크 모드 토큰, 번쩍임 막는 스크립트, 머리글 전환 버튼, 그래프 색 확인
- [x] T004 [US2] 화면: 글 목차(2개 이상), 위치 표시, 주소 공유
- [x] T005 [US4] 서버: `/og/...` 미리보기 HTML (공개·비공개·없는 글, 블로그, 첫 화면) (테스트)
- [x] T006 [US4] 기본 카드 이미지 `og-default.png`
- [x] T007 [US5] 서버: `/rss.xml`, `/blog/{id}/rss.xml` (테스트)
- [x] T008 [US6] 서버: `/sitemap.xml`, `/robots.txt` (테스트)
- [x] T009 [US4-6] 보안 설정(로그인 없이 GET), Vite 프록시, 화면 컨테이너 Nginx(로봇 → `/og`, XML 주소 → 서버)
- [x] T010 [US5] 화면: 블로그 화면 RSS 링크와 `<link rel="alternate">`
- [x] T011 확인: `mvn verify`, `npm run lint`, `npm run build`, Nginx 이미지로 로봇 User-Agent 확인, Playwright로 다크 모드·목차 화면
- [ ] T012 사람 확인: 사용자가 로컬(5173)에서 보고, "배포해줘" 뒤 카카오톡 카드 확인 (SC-2). 로컬 확인과 배포는 끝(2026-10-12), 카카오톡 카드만 남음
- [x] T013 [US7] 화면: 글쓰기 나란히 미리보기(원고지 종이, 스크롤 맞춤, 실시간 원고지 매수), 좁은 화면 탭, 서식 도구 막대 (커밋 b8b0e3a)
- [x] T014 [US8] 화면: 머리글 글쓰기 버튼 + 내 메뉴(카드 펼침, 형광펜 항목, 새 댓글 점) (커밋 b8b0e3a)
- [x] T015 개발 서버를 켤 때 예시 글이 없으면 자동으로 넣기 `DevDemoSeed` (2026-10-12 사용자: 로컬 DB에 따로 넣기 번거롭다, 커밋 b65fed8)
