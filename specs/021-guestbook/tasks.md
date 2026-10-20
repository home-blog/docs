# Tasks: 방명록 (021)

테스트를 먼저 쓴다(헌법·테스트 전략).

## 바탕
- [x] T001 Flyway V30(`guestbook_entry`, `notification.kind` guestbook·gb_reply, `notification.guestbook_entry_id`), 새 모듈 `guestbook`, 오류 코드, 설정 `guestbook.*`

## US1 엽서 남기기와 읽기
- [x] T002 [테스트] S1-1~S1-6 (정상·경계 300자·예외)
- [x] T003 엽서 목록·쓰기 API, 회원 잠금으로 간격·하루 한도, 봉한 엽서
- [x] T004 우편함 "방명록" 소식 (이벤트 → 구독 모듈)

## US2 답장과 정리
- [x] T005 [테스트] S2-1~S2-3
- [x] T006 답장 쓰기·고치기·지우기, 엽서 지우기, "답장" 소식

## 화면
- [x] T007 [테스트] Vitest: 목록·봉한 엽서·쓰기·우표 고르기·답장·지우기·오류
- [x] T008 블로그 머리 "방명록" 탭, `/blog/:blogId/guestbook` 엽서함(우표 SVG 5종, 꽂힘·소인 애니메이션, 뒤집어 답장), 우편함 소식 두 가지

## 마무리
- [x] T009 CHANGELOG `[0.16.0]`, REST Docs, 예시 엽서(seed), ERD E-18 — REST Docs·예시 엽서 6장(`guestbook.json`, Actions `예시 데이터` guestbook) 끝. 남은 것: CHANGELOG(릴리스 때), ERD E-18
- [x] T010 `mvn verify`, `npm run lint && npm test && npm run build`, Playwright
