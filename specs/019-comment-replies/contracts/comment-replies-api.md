# 계약: 답글·비밀 댓글 (019)

API 주소 버전은 v1 그대로다(칸을 더하기만 한다).

## 1. 댓글 목록 `GET /api/v1/posts/{postId}/comments`
- 순서: 원댓글 오래된 순, 각 원댓글 바로 뒤에 그 답글 오래된 순.
- `count`: 지운 칸을 뺀 수(봉한 쪽지는 센다).
- 항목에 더하는 칸: `parentId`(원댓글이면 null), `secret`, `sealed`(읽을 수 없는 비밀: `body`는 "", `author`는 `{id:null, nickname:null, withdrawn:false}`), `deleted`(지운 칸: 같은 빈 값), `canReply`.
- `canDelete`는 지운 칸이면 거짓.

## 2. 댓글 쓰기 `POST /api/v1/posts/{postId}/comments`
- 본문: `{ "body": "...", "parentId": 12, "secret": true }`. `parentId`·`secret`은 빼도 된다.
- 부모가 답글이면 그 원댓글로 바꿔 저장한다. 부모가 비밀이면 답글도 비밀.
- 404 `COMMENT_NOT_FOUND`: 부모가 없음·다른 글·지운 칸·읽을 수 없는 비밀.

## 3. 댓글 지우기 `DELETE /api/v1/comments/{commentId}` (204)
- 답글이 남은 원댓글은 지운 칸이 된다. 답글을 지워 원댓글이 빈 지운 칸이 되면 함께 지운다.
- 지운 칸을 다시 지우면 404.

## 4. 우편함 `GET /api/v1/me/notifications`
- `kind`에 `reply`가 더해진다.

## 5. 블로그 관리 댓글 `GET /api/v1/manage/comments`
- 지운 칸은 빠진다. 항목에 `secret`, `reply`(답글이면 참)를 더한다.
