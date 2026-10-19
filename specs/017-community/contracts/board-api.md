# Contracts: 글마당 (017)

## 1. 읽기 (로그인 없이)
- `GET /api/v1/board/posts?topic=&kind=talk|question&unsolved=true&q=&page=` → `{ "totalCount", "page", "totalPages", "pageSize": 10, "topics": ["여행", …], "posts": [ { "boardPostId", "topic", "kind", "title", "nickname", "createdAt", "views", "commentCount", "solved" } ] }` 최신순
  - 400 `INVALID_INPUT`(없는 주제·종류)
- `GET /api/v1/board/posts/recent` → `{ "posts": [ 위와 같은 줄 5개 ] }`
- `GET /api/v1/board/posts/{id}` → `{ "boardPostId", "topic", "kind", "title", "content", "nickname", "writerId", "createdAt", "updatedAt", "views", "acceptedCommentId", "mine", "canDelete" }` · 404 `BOARD_POST_NOT_FOUND`
  - 상세를 보면 조회 수를 센다(쓴 사람 제외, 30분 한 번)
- `GET /api/v1/board/posts/{id}/comments` → `{ "comments": [ { "boardCommentId", "nickname", "content", "createdAt", "accepted", "mine", "canDelete" } ] }` 채택한 댓글이 맨 위, 나머지는 오래된 순

## 2. 쓰기 (로그인, CSRF)
- `POST /api/v1/board/posts` `{ "topic": "여행", "kind": "question", "title", "content" }` → 201 `{ "boardPostId" }` · 429 `BOARD_POST_TOO_FAST`
- `PUT /api/v1/board/posts/{id}` 같은 본문 → 200 · 403 `BOARD_FORBIDDEN` · 400 `BOARD_ACCEPTED_KIND_LOCKED`
- `DELETE /api/v1/board/posts/{id}` → 204 (쓴 사람·관리자)
- `POST /api/v1/board/posts/{id}/comments` `{ "content" }` → 201 `{ "boardCommentId" }` · 429 `BOARD_COMMENT_TOO_FAST`
- `DELETE /api/v1/board/comments/{commentId}` → 204 · 409 `BOARD_COMMENT_ACCEPTED`(관리자는 됨)
- `PUT /api/v1/board/posts/{id}/accepted` `{ "commentId": 12 }` → 200 `{ "acceptedCommentId": 12 }` · `DELETE …/accepted` → 200 `{ "acceptedCommentId": null }`
  - 400 `BOARD_ACCEPT_INVALID`(이야기 글, 내 댓글, 다른 글 댓글), 403 `BOARD_FORBIDDEN`

## 3. 우편함 (016에 더함)
- 소식 줄에 `kind: "board_comment" | "board_accept"`, `boardPostId`를 더한다. 화면은 `/board/{boardPostId}`로 간다.
