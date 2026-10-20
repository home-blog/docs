# Contracts: 방명록 (021)

변경 요청은 로그인 + CSRF. 오류 모양은 공통(`{ code, message, fieldErrors? }`).

## 1. 엽서 목록
- `GET /api/v1/blogs/{blogId}/guestbook?page=` (누구나) → 200
  `{ "totalCount", "page", "totalPages", "pageSize", "canWrite", "bodyMaxLength", "stamps": ["pine", …], "items": [ Entry ] }` · 404 `BLOG_NOT_FOUND`
- `Entry` = `{ "entryId", "author": { "id", "nickname", "withdrawn" }, "body", "stamp", "secret", "sealed", "createdAt", "reply", "repliedAt", "canDelete", "canReply" }`
  - 봉한 엽서: `sealed: true`, `body: ""`, `author` 모두 빈 값, `reply: null`.
  - `canWrite`: 로그인했고 주인이 아니다.

## 2. 엽서 쓰기
- `POST /api/v1/blogs/{blogId}/guestbook` `{ "body", "stamp", "secret" }` → 201 `Entry`
  · 401 · 403 `GUESTBOOK_OWN_BLOG` · 404 `BLOG_NOT_FOUND` · 400(내용, `GUESTBOOK_STAMP_INVALID`) · 429 `GUESTBOOK_TOO_FREQUENT`·`GUESTBOOK_DAILY_LIMIT`

## 3. 지우기
- `DELETE /api/v1/guestbook/{entryId}` (쓴 사람·주인) → 204 · 404 `GUESTBOOK_ENTRY_NOT_FOUND`

## 4. 답장
- `PUT /api/v1/guestbook/{entryId}/reply` `{ "body" }` (주인) → 200 `Entry` · 400 · 404
- `DELETE /api/v1/guestbook/{entryId}/reply` (주인) → 204 · 404

## 5. 우편함
- 소식 `kind: "guestbook"`(내 방명록에 새 엽서, `blogId`, `actorNickname`), `kind: "gb_reply"`(내 엽서에 답장, `blogId`, 블로그 이름). 비밀 엽서의 소식도 내용은 싣지 않는다.
