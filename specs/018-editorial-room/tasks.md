# Tasks: 편집실 (018)

테스트를 먼저 쓴다(헌법·테스트 전략).

## 바탕
- [x] T001 Flyway V23(`post.locked_at`·`lock_reason`, `post_report.resolved_at`·`resolution`, `board_report`, `editorial_action`, `notification.kind` editorial), 새 모듈 `editorial`, 오류 코드, 설정 `editorial.*`

## US1 신고 검토와 내림
- [x] T002 [테스트] S1-1~S1-5
- [x] T003 글 모듈 `PostModeration`(잠그기·풀기, 글 고치기에서 공개로 바꾸기 막기), community 신고 틀(대기 묶음·닫기)
- [x] T004 편집실 API(대기 목록·신고 자세히·내림·반려·처리 기록·잠금 풀기), 우편함 "편집실" 소식
- [x] T005 화면: 내 메뉴 "편집실"(관리자만), `/editorial` 탭·펼침·도장 단추, 글 고치기 화면의 잠금 안내

## US2 글마당 신고
- [x] T006 [테스트] S2-1·S2-2
- [x] T007 글마당 신고 API·편집실 글마당 탭(지우기·반려), 화면 신고 단추

## US3 운영 현황
- [x] T008 [테스트] S3-1·S3-2
- [x] T009 현황 API·숫자 카드

## 마무리
- [x] T010 CHANGELOG `[0.13.0]`, 버전, REST Docs, V24 0.13.0 공지, README, ERD E-15
- [ ] T011 `mvn verify`, `npm run lint && npm test && npm run build`, Playwright
- [ ] T012 PR → 릴리스 0.13.0 → 배포 → 다시 합침
