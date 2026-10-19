# Tasks: 답글·비밀 댓글 (019)

테스트를 먼저 쓴다(헌법·테스트 전략).

## 바탕
- [x] T001 Flyway V25(`comment.deleted_at`, 부모 색인·검사, `notification.kind` reply)

## US1 답글
- [x] T002 [테스트] S1-1~S1-5
- [x] T003 댓글 쓰기에 `parentId`, 목록 순서·`canReply`, 지우기의 지운 칸·원댓글 줄 잠금, 수 세기에서 지운 칸 빼기
- [x] T004 답글 소식(`CommentWrittenEvent.parentWriterId`, 우편함 `reply`)

## US2 비밀 댓글
- [x] T005 [테스트] S2-1~S2-3
- [x] T006 `secret`·`sealed`, 비밀 부모의 답글은 비밀, 블로그 관리의 `secret`·`reply`

## 화면
- [x] T007 [테스트] 답글·봉한 쪽지·지운 칸·우편함 답글 (Vitest)
- [x] T008 댓글 영역 스레드·답글 입력·봉해서 남기기, 블로그 관리 표시, 우편함 "답글" 우표

## 마무리
- [x] T009 CHANGELOG `[0.14.0]`, 버전, REST Docs, V26 0.14.0 공지, README, ERD E-16
- [x] T010 `mvn verify`, `npm run lint && npm test && npm run build`, Playwright, 별도 리뷰 지적 반영(잠금을 기다리는 사이 지운 칸 → 404 재현 테스트)
- [x] T011 PR #95 → 릴리스 #96 → 배포 0.14.0(2026-10-19, 실행 39280257419) → 다시 합침 #97
