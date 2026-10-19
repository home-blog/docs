# Contracts: 편집실 (018)

모든 `/api/v1/editorial/**`는 관리자만: 로그인하지 않았으면 401, 관리자가 아니면 403. 변경 요청은 CSRF.

## 1. 운영 현황
- `GET /api/v1/editorial/summary` → `{ "members", "membersToday", "publicPosts", "publicPostsToday", "boardPosts", "pendingReports" }`

## 2. 블로그 글 신고
- `GET /api/v1/editorial/reports/posts?page=` → `{ "totalCount", "page", "totalPages", "pageSize", "items": [ { "postId", "blogId", "blogName", "title", "visibility", "locked", "reportCount", "spam", "abuse", "adult", "other", "firstAt", "lastAt" } ] }` 신고가 많은 글 먼저
- `GET /api/v1/editorial/reports/posts/{postId}` → `{ "postId", "blogName", "title", "excerpt", "visibility", "locked", "lockReason", "reports": [ { "reason", "detail", "createdAt" } ] }` · 404
- `POST /api/v1/editorial/posts/{postId}/take-down` `{ "reason" }` → 204 · 400 `EDITORIAL_REASON_INVALID` · 409 `REPORT_ALREADY_RESOLVED`
- `POST /api/v1/editorial/posts/{postId}/dismiss` → 204 · 409
- `POST /api/v1/editorial/posts/{postId}/unlock` → 200 `{ "unlocked": true }` (이미 풀려 있었으면 `false`, 기록도 남기지 않음)

## 3. 글마당 신고
- `POST /api/v1/board/posts/{id}/reports` `{ "reason", "detail" }` (로그인) → 201 `{ "message" }` · 403 `SELF_REPORT_NOT_ALLOWED` · 409 `ALREADY_REPORTED` · 404 · 400(사유·설명)
- `GET /api/v1/editorial/reports/board?page=` → 위와 같은 모양(`boardPostId`, `topic`, `title`, 사유별 수)
- `GET /api/v1/editorial/reports/board/{id}` → `{ "boardPostId", "topic", "title", "excerpt", "reports" }`
- `POST /api/v1/editorial/board/{id}/remove` `{ "reason" }` → 204 · 409(대기 신고 없음)
- `POST /api/v1/editorial/board/{id}/dismiss` → 204 · 409

## 4. 처리 기록
- `GET /api/v1/editorial/actions?page=` → `{ …, "items": [ { "actionId", "targetKind", "postId", "boardPostId", "title", "action", "reason", "reportCount", "createdAt" } ] }` 최근 순

## 5. 그 밖에 바뀐 응답
- `GET /api/v1/posts/{postId}/edit`에 `locked`, `lockReason`. 잠긴 글을 `PUT /api/v1/posts/{postId}`로 공개로 바꾸면 400 `POST_LOCKED_BY_EDITOR`.
- 우편함 소식에 `kind: "editorial"`, `note`(사유). 편집실 소식은 볼 수 없는 글이어도 `postTitle`을 준다.
