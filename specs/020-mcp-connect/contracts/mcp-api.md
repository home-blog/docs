# 계약: 글결 MCP (020)

## 1. 개인 열쇠 (세션 로그인)
- `GET /api/v1/me/access-keys` → `{ items: [{ keyId, name, hint, scope, createdAt, expiresAt, lastUsedAt }], maxCount }`
- `POST /api/v1/me/access-keys` `{ name, scope: "read"|"draft", days: 30|90|365 }` → 201 `{ keyId, name, hint, scope, createdAt, expiresAt, token }` (`token`은 이때 한 번만)
  - 400 `ACCESS_KEY_INVALID`(이름·범위·기한), 409 `ACCESS_KEY_LIMIT`(5개)
- `DELETE /api/v1/me/access-keys/{keyId}` → 204, 남의 열쇠·없는 열쇠 404 `ACCESS_KEY_NOT_FOUND`

## 2. MCP (`/api/v1/mcp`, 열쇠 로그인)
- `POST` 한 JSON-RPC 메시지. 요청이면 200 `application/json` 응답, 알림·응답이면 202 빈 몸.
- `GET`·`DELETE` 405 (서버가 먼저 보내는 흐름·세션 없음).
- 열쇠가 없거나 틀리면 401 + `WWW-Authenticate: Bearer realm="geulgyeol"`. 다른 사이트 Origin이면 403. 1분 60번을 넘으면 429.
- 메서드: `initialize`, `notifications/initialized`, `ping`, `tools/list`, `tools/call`. 그 밖은 -32601.
- 도구 결과는 `content`(사람이 읽을 글)와 `structuredContent`(같은 내용의 JSON)를 함께 준다. 도구 안의 오류(없는 글, 한도)는 `isError: true`.

| 도구 | 입력 | 범위 |
|---|---|---|
| `search_posts` | `query`(2~50자), `mineOnly`, `limit`(1~10, 기본 5) | read |
| `search_passages` | `query`, `mineOnly`, `limit`(1~8, 기본 5) | read |
| `read_post` | `postId` | read |
| `list_my_posts` | `page`(1부터) | read |
| `my_blog_stats` | 없음 | read |
| `save_draft` | `title`(0~100), `content`(마크다운) | draft |
