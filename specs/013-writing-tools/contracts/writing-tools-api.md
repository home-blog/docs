# API 계약: 글쓰기 도구 (013)

주소는 모두 `/api/v1/…`, 기존 API는 바꾸지 않는다(칸·주소를 더하는 깨지지 않는 변경, 009). 오류는 공통 모양(`code`, `message`, `fieldErrors`). GET이 아닌 요청은 CSRF 토큰을 싣는다. `me/…` 주소는 로그인 필요(아니면 401), 남의 것은 모두 404.

## 1. 글쓰기 잔디 `GET /api/v1/blogs/{blogId}/activity` (누구나)

오늘(한국 시간) 포함 365일. 글이 있는 날만 `days`에 담는다(날짜 오름차순). 없는 블로그는 404 `BLOG_NOT_FOUND`.
```json
{
  "blogId": 7,
  "from": "2025-10-16",
  "to": "2026-10-15",
  "days": [ { "date": "2026-10-12", "postCount": 2, "sheets": 18 } ],
  "totalPosts": 41,
  "totalSheets": 380
}
```

## 2. 연재

### 2-1. 블로그의 연재 목록 `GET /api/v1/blogs/{blogId}/series` (누구나)
`postCount`는 보는 사람에게 보이는 글 수. 만든 순서.
```json
{ "items": [ { "seriesId": 3, "name": "제주 한 달 살기", "description": "…", "postCount": 5 } ] }
```

### 2-2. 연재 한 개 `GET /api/v1/series/{seriesId}` (누구나)
글은 화 순서, 보이는 글만, `episode`는 보이는 글로 센 번호(D-10). 주인에게는 `visibility`가 함께 온다.
```json
{
  "seriesId": 3, "blogId": 7, "name": "제주 한 달 살기", "description": "…",
  "posts": [ { "postId": 41, "title": "첫날", "episode": 1, "createdAt": "…", "visibility": "public" } ]
}
```

### 2-3. 글의 연재 정보 `GET /api/v1/posts/{postId}/series` (누구나)
글을 볼 수 없으면 404(글 읽기와 같은 규칙). 연재에 없으면 `{ "series": null }`.
```json
{
  "series": {
    "seriesId": 3, "name": "제주 한 달 살기", "episode": 2, "total": 5,
    "previous": { "postId": 41, "title": "첫날" },
    "next": { "postId": 45, "title": "셋째 날" },
    "posts": [ { "postId": 41, "title": "첫날", "episode": 1 } ]
  }
}
```

### 2-4. 연재 만들기·고치기·지우기 (주인)
- `POST /api/v1/me/blog/series` `{ "name": "…", "description": "…" }` → 201 연재 한 개(2-2 모양). 이름 `SERIES_NAME_REQUIRED`·`SERIES_NAME_TOO_LONG`, 소개 `SERIES_DESCRIPTION_TOO_LONG`, 개수 초과 409 `SERIES_LIMIT_EXCEEDED`.
- `PUT /api/v1/me/blog/series/{seriesId}` 같은 본문 → 200.
- `DELETE /api/v1/me/blog/series/{seriesId}` → 204. 글은 남는다.

### 2-5. 글 넣기·빼기·순서 (주인)
- `PUT /api/v1/me/blog/series/{seriesId}/posts/{postId}` → 204. 다른 연재에 있었으면 옮긴다. 이미 이 연재에 있으면 그대로. 연재의 글이 200개면 409 `SERIES_POSTS_LIMIT_EXCEEDED`.
- `DELETE /api/v1/me/blog/series/{seriesId}/posts/{postId}` → 204(이 연재에 없으면 404).
- `PUT /api/v1/me/blog/series/{seriesId}/order` `{ "postIds": [45, 41, 43] }` → 200 연재 한 개. 그 연재의 글 번호를 모두 한 번씩 보내야 한다. 아니면 400 `SERIES_ORDER_MISMATCH`.

## 3. 임시 저장 (주인)

임시 저장 본문(`DraftRequest`):
```json
{
  "title": "", "content": "", "categoryId": null, "topicId": null, "seriesId": null,
  "visibility": "public", "tags": []
}
```
저장할 때는 글자 수 상한만 본다(제목 100자, 본문은 글 상한, 태그 칸 200자). 분류·주제·연재가 내 것이 아니거나 없으면 그 칸 오류.

- `GET /api/v1/me/drafts` → `{ "items": [ { "draftId": 9, "title": "", "updatedAt": "…", "publishAt": null } ], "count": 1, "maxCount": 50 }` (마지막 저장 순)
- `POST /api/v1/me/drafts` → 201 `{ "draftId": 9, "updatedAt": "…", "publishAt": null }`. 50개면 409 `DRAFT_LIMIT_EXCEEDED`.
- `GET /api/v1/me/drafts/{draftId}` → 본문 칸 전부 + `draftId`, `updatedAt`, `publishAt`.
- `PUT /api/v1/me/drafts/{draftId}` → 200 (POST와 같은 응답). 예약된 글이면 예약은 그대로 둔다(올릴 때 다시 검사하고, 걸리면 예약을 푼다).
- `DELETE /api/v1/me/drafts/{draftId}` → 204.

### 3-1. 발행 `POST /api/v1/me/drafts/{draftId}/publish`
본문은 글쓰기(003 contracts 10)와 같은 모양 + `seriesId`. 보통 글쓰기와 같은 검사를 하고, 글을 만든 뒤 임시 저장 글을 지운다. → 201 `{ "postId": 41 }`. 동시에 두 번 와도 글은 하나(임시 저장 글의 요청 번호). 이미 발행해 임시 저장 글이 없으면 404.

## 4. 예약 (주인)

- `PUT /api/v1/me/drafts/{draftId}/schedule` 본문: 3-1과 같은 모양 + `"publishAt": "2026-10-20T00:00:00Z"`. 보통 글쓰기 검사 + 시각 검사(`PUBLISH_AT_TOO_SOON` 1분 뒤보다 이름, `PUBLISH_AT_TOO_LATE` 30일 뒤보다 늦음, 초 아래는 버림). → 200 임시 저장 응답(`publishAt` 채움).
- `DELETE /api/v1/me/drafts/{draftId}/schedule` → 200 (`publishAt: null`).

## 5. 글쓰기 설정 `GET /api/v1/me/blog/post-form` (기존, 칸 더하기)
`series`(내 연재 `[{ seriesId, name }]`, 연재 고르기)와 `writing`(`draftAutosaveSeconds` 30, `draftMaxCount` 50, `scheduleMaxDays` 30)을 더한다.

## 6. 분류 글 수 (기존 003 contracts 5·8, 뜻 넓힘)
분류 삭제 거절(`CATEGORY_HAS_POSTS`)에 **예약된 글**도 센다(S4-6). 화면에 보이는 분류의 글 수에는 넣지 않는다(예약 글은 아직 목록에 없다). 예약 없는 임시 저장 글의 분류는 지우면 비워진다.
