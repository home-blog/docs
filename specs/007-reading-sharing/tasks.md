# Tasks: 읽기 편하게·널리 퍼지게 (v0.3.0)

**Input**: [spec.md](spec.md), [plan.md](plan.md) · 코드 브랜치 `feat/v0.3-reading-sharing`

- [ ] T001 [US3] 서버: `ManuscriptMeasure` + 설정 `post.manuscript.*`, 글 상세·블로그 글 목록·첫 화면 글 모음 응답에 `sheets`, `readMinutes` (테스트)
- [ ] T002 [US3] 화면: 상세·목록·첫 화면에 "원고지 N매 · M분"
- [ ] T003 [US1] 화면: 다크 모드 토큰, 번쩍임 막는 스크립트, 머리글 전환 버튼, 그래프 색 확인
- [ ] T004 [US2] 화면: 글 목차(2개 이상), 위치 표시, 주소 공유
- [ ] T005 [US4] 서버: `/og/...` 미리보기 HTML (공개·비공개·없는 글, 블로그, 첫 화면) (테스트)
- [ ] T006 [US4] 기본 카드 이미지 `og-default.png`
- [ ] T007 [US5] 서버: `/rss.xml`, `/blog/{id}/rss.xml` (테스트)
- [ ] T008 [US6] 서버: `/sitemap.xml`, `/robots.txt` (테스트)
- [ ] T009 [US4-6] 보안 설정(로그인 없이 GET), Vite 프록시, 화면 컨테이너 Nginx(로봇 → `/og`, XML 주소 → 서버)
- [ ] T010 [US5] 화면: 블로그 화면 RSS 링크와 `<link rel="alternate">`
- [ ] T011 확인: `mvn verify`, `npm run lint`, `npm run build`, Nginx 이미지로 로봇 User-Agent 확인, Playwright로 다크 모드·목차 화면
- [ ] T012 사람 확인: 사용자가 로컬(5173)에서 보고, "배포해줘" 뒤 카카오톡 카드 확인 (SC-2)
