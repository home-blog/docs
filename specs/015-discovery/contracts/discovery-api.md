# Contracts: 발견 (015)

## 1. 인기 검색어 `GET /api/v1/search/trending` (누구나)
`{ "items": [ { "rank": 1, "keyword": "제주", "change": "up", "delta": 2 }, ... ] }`
- `change`: `up`(delta만큼 올라감) · `down`(delta만큼 내려감) · `new` · `same`. 10개 이하. 없으면 빈 목록.
- 검색(`GET /api/v1/search/posts?q=`)은 그대로이고, 결과가 있으면 서버가 센다.

## 2. 오늘의 이슈 `GET /api/v1/posts/today-issue` (누구나)
`{ "date": "2026-10-19", "posts": [ 글 모음 카드, ... ] }` 3개 이하(date는 고른 한국 날짜). 카드 모양은 첫 화면 글 모음과 같다.

## 3. 문장 수집 (로그인, CSRF)
- `POST /api/v1/me/clips` `{ "postId": 12, "text": "문장" }` → 201(새로 모음) 또는 200(이미 모은 것) `{ "clipId", "text", "createdAt", "post": { "postId", "title", "blogName", "available": true } }`
  - 400 `CLIP_TEXT_INVALID`(비었거나 300자 넘음), 400 `CLIP_NOT_IN_POST`(본문에 없음), 409 `CLIP_LIMIT_EXCEEDED`(1,000개, 다른 한도 오류와 같이 409), 404 `CLIP_NOT_FOUND`(지우기), 404(볼 수 없는 글)
- `GET /api/v1/me/clips?page=1` → `{ "items": [위와 같은 모양], "page", "totalPages", "totalCount" }` 최근 순, 한 쪽 20개
- `DELETE /api/v1/me/clips/{clipId}` → 204. 남의 것·없는 것은 404
- `GET /api/v1/posts/{postId}/clips/collectors` (글 주인) → `{ "collectors": 3 }`. 주인이 아니면 404
