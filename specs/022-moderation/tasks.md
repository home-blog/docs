# Tasks: 운영 강화 (022)

테스트를 먼저 쓴다(헌법·테스트 전략).

## 바탕
- [ ] T001 Flyway V31(`comment_report`, `comment.hidden_*`, `users.suspended_*`, `editorial_action` 칸·CHECK), 오류 코드, 설정 `moderation.*`

## US1 댓글 신고와 자동 가림
- [ ] T002 [테스트] S1-1~S1-5 (자동 가림 경계 2·3건, 동시 신고)
- [ ] T003 댓글 신고 API, 줄 잠금으로 자동 가림, 목록의 `hidden`·`canReport`, 가린 댓글에 답글 막기
- [ ] T004 틀 `CommentReports`, 편집실 댓글 탭 API(목록·자세히·가리기·반려·되돌리기), 우편함 소식

## US2 회원 정지
- [ ] T005 [테스트] S2-1~S2-5 (로그인 403, 세션 끝내기, 열쇠 끊기, 옛 세션, 기한 경계)
- [ ] T006 `users.suspended_*`, 틀 `MemberModeration`, 로그인·소셜 로그인 막기, `LoggedInMember` 두 번째 벽
- [ ] T007 편집실 회원 탭 API(찾기·정지·풀기), 처리 기록

## US3 운영 현황
- [ ] T008 현황 숫자 두 개

## 화면
- [ ] T009 [테스트] Vitest: 댓글 신고 단추·가린 칸, 편집실 댓글·회원 탭, 로그인 정지 안내
- [ ] T010 화면 구현(먹줄 칸, 정지 도장)

## 마무리
- [ ] T011 CHANGELOG, REST Docs, V32 0.16.0 공지, ERD E-19
- [ ] T012 `mvn verify`, 화면 검사, Playwright, PR → 릴리스 0.16.0 → 배포 → 다시 합침
