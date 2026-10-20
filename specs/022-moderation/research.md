# Research: 운영 강화 (022)

`가안`은 추천대로 정한 것(2026-10-20 사용자 "끝까지 다 해"). 바꾸고 싶으면 그 줄만 고친다.

| ID | 정할 것 | 고른 것 | 다른 선택지 | 이유 |
|---|---|---|---|---|
| D-1 | 댓글 신고 표의 자리 | 댓글 모듈 안 새 표 `comment_report`, 편집실은 맨 위 틀 `CommentReports`로 묻고 닫는다 `가안` | 좋아요·신고 모듈(community) | 댓글 표를 가진 모듈이 신고·가림을 함께 다뤄야 한 트랜잭션·한 잠금으로 센다 (글마당 신고와 같은 방식, 018 D-5) |
| D-2 | 자동 가림 | 신고를 넣는 트랜잭션 안에서 그 댓글 줄을 잠그고 대기 신고 수를 세어 기준 이상이면 `hidden_at`을 넣는다 `가안` | 1분마다 일괄 | 바로 가려야 피해가 적다. 줄 잠금으로 동시 신고에도 한 번만 |
| D-3 | 가림의 뜻 | 줄은 남기고 `hidden_at`·`hidden_by`(`auto`·`editor`). 목록에서 내용·쓴 사람을 뺀다 `가안` | 지우기 | 되돌릴 수 있다. 답글이 매달린 줄도 그대로 |
| D-4 | 정지 저장 | `users.suspended_until`, `suspend_reason`. "지금 정지인가" = `suspended_until > now` `가안` | 정지 표 따로 | 기한이 지나면 저절로 풀린다. 확인이 칸 하나다 |
| D-5 | 정지의 효과 | 로그인 막기(Spring Security `isEnabled=false` → `DisabledException` → 403 `ACCOUNT_SUSPENDED`), 세션 모두 끝내기(`MemberSessions.expireAll`), 열쇠 끊기, 세션 확인(`LoggedInMember`)에서도 정지는 로그인 안 함 `가안` | 글쓰기만 막기 | 한곳(회원 모듈)에서 끝난다. 다른 모듈은 그대로 "로그인 안 함"으로 처리 |
| D-6 | 모듈 | 편집실은 회원 모듈의 틀 `MemberModeration`(찾기·정지·풀기)으로만 부른다 `가안` | 편집실이 users 표를 직접 | 모듈 규칙 |
| D-7 | 기록 | `editorial_action`에 `comment_id`, `member_id` 칸, 대상 `comment`·`member`, 처리 `hidden`·`restored`·`suspended`·`unsuspended` `가안` | 기록 표 따로 | 처리 기록 탭 하나에 모인다 |
| D-8 | 가린 댓글 소식 | 지금의 편집실 소식(`editorial`)을 쓰고 제목을 "‘글 제목’에 남긴 댓글"로 `가안` | 새 종류 | 우편함 화면을 크게 바꾸지 않는다 |
