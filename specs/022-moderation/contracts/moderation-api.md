# Contracts: 운영 강화 (022)

`/api/v1/editorial/**`는 관리자만(401·403), 변경 요청은 CSRF.

## 1. 댓글 신고
- `POST /api/v1/comments/{commentId}/reports` `{ "reason", "detail" }` (로그인) → 201 `{ "message", "hidden" }` (`hidden`: 이 신고로 저절로 가려졌나)
  · 401 · 403 `SELF_REPORT_NOT_ALLOWED` · 404 `COMMENT_NOT_FOUND` · 409 `ALREADY_REPORTED` · 400

## 2. 댓글 목록의 바뀐 칸 (005 contracts 1)
- 댓글마다 `hidden`(편집실이 가림). 가렸으면 `body: ""`, `author` 빈 값, `canReply: false`. `canReport`: 로그인했고, 내 댓글이 아니고, 보이는 댓글.

## 3. 편집실 댓글
- `GET /api/v1/editorial/reports/comments?page=` → `{ totalCount, page, totalPages, pageSize, items: [ { "commentId", "postId", "postTitle", "excerpt", "writerId", "writerNickname", "hidden", "hiddenBy", "reportCount", "spam", "abuse", "adult", "other", "firstAt", "lastAt" } ] }`
- `GET /api/v1/editorial/reports/comments/{commentId}` → `{ "commentId", "postId", "postTitle", "body", "writerNickname", "hidden", "hiddenBy", "reports": [ { reason, detail, createdAt } ] }` · 404
- `POST /api/v1/editorial/comments/{commentId}/hide` `{ "reason" }` → 204 · 400 · 404 · 409 `REPORT_ALREADY_RESOLVED`
- `POST /api/v1/editorial/comments/{commentId}/dismiss` → 204 · 404 · 409
- `POST /api/v1/editorial/comments/{commentId}/restore` → 200 `{ "restored": true|false }`

## 4. 회원
- `GET /api/v1/editorial/members?q=&page=` → `{ …, items: [ { "memberId", "nickname", "blogId", "blogName", "role", "joinedAt", "suspended", "suspendedUntil", "suspendReason" } ], "suspendDays": [1,3,7,30,365] }` (q가 비면 최근 가입 순)
- `POST /api/v1/editorial/members/{memberId}/suspend` `{ "days", "reason" }` → 200 `{ "suspendedUntil" }` · 400 `SUSPEND_DAYS_INVALID`·`EDITORIAL_REASON_INVALID` · 404 `MEMBER_NOT_FOUND` · 409 `MEMBER_NOT_SUSPENDABLE`
- `POST /api/v1/editorial/members/{memberId}/unsuspend` → 200 `{ "unsuspended": true|false }` · 404

## 5. 로그인
- `POST /api/v1/auth/login`: 정지 중이고 비밀번호가 맞으면 403 `{ "code": "ACCOUNT_SUSPENDED", "message": "※ 2026-10-27 15:00까지 이용이 정지된 계정입니다. 사유: …" , "suspendedUntil", "reason" }`.
- 소셜 로그인: `/login?error=suspended`로 돌려보낸다.

## 6. 현황·기록
- `GET /api/v1/editorial/summary`에 `pendingCommentReports`, `suspendedMembers`. `pendingReports`는 댓글 신고도 센다.
- 처리 기록 줄에 `commentId`, `memberId`. `targetKind`: `post`·`board`·`comment`·`member`.
