# API 계약: 공지사항 (012)

주소는 모두 `/api/v1/…`. 오류는 공통 모양(`code`, `message`, `fieldErrors`). GET이 아닌 요청은 CSRF 토큰(`X-XSRF-TOKEN`)을 싣는다.

## 1. 목록 `GET /api/v1/notices?kind=&page=` (누구나)

- `kind`: `update` | `notice` | 없음(전체). 모르는 값은 전체로 본다.
- `page`: 1부터. 규칙은 글 목록(004 D-2)과 같다(범위를 넘으면 마지막 쪽).

```json
{
  "items": [
    {
      "noticeId": 3,
      "kind": "update",
      "title": "공지사항 게시판이 생겼어요",
      "releaseVersion": "0.6.0",
      "pinned": true,
      "createdAt": "2026-10-15T09:00:00Z"
    }
  ],
  "page": 1,
  "totalPages": 1,
  "totalCount": 1
}
```

## 2. 최신 글 시각 `GET /api/v1/notices/latest` (누구나)

머리글의 새 글 점(US3)용. 글이 없으면 `latestAt: null`.
```json
{ "latestAt": "2026-10-15T09:00:00Z" }
```

## 3. 읽기 `GET /api/v1/notices/{noticeId}` (누구나)

```json
{
  "noticeId": 3,
  "kind": "update",
  "title": "공지사항 게시판이 생겼어요",
  "content": "마크다운 본문",
  "releaseVersion": "0.6.0",
  "releaseUrl": "https://github.com/home-blog/blog/releases/tag/v0.6.0",
  "pinned": true,
  "createdAt": "2026-10-15T09:00:00Z",
  "updatedAt": null
}
```
없으면 404 `NOTICE_NOT_FOUND`. 공지 글은 `releaseVersion`, `releaseUrl`이 `null`.

## 4. 쓰기 `POST /api/v1/notices` (관리자)

```json
{ "kind": "update", "title": "…", "content": "…", "releaseVersion": "0.6.0", "pinned": true }
```
- 201 `{ "noticeId": 3 }`
- 400 `VALIDATION_FAILED` + `fieldErrors`:

| 칸 | code | message |
|---|---|---|
| `kind` | `NOTICE_KIND_INVALID` | ※ 종류를 골라 주세요 |
| `title` | `TITLE_REQUIRED` / `TITLE_TOO_LONG` | 제목을 입력해 주세요 / ※ 제목은 100자 이하로 입력해 주세요 |
| `content` | `CONTENT_REQUIRED` / `CONTENT_TOO_LONG` | 본문을 입력해 주세요 / ※ 본문은 10,000자 이하로 입력해 주세요 |
| `releaseVersion` | `RELEASE_VERSION_REQUIRED` | ※ 업데이트에는 릴리스 번호를 적어 주세요 (예: 0.6.0) |
| `releaseVersion` | `RELEASE_VERSION_INVALID` | ※ 릴리스 번호는 0.6.0 모양으로 적어 주세요 |
| `releaseVersion` | `RELEASE_VERSION_NOT_ALLOWED` | ※ 공지에는 릴리스 번호를 적지 않습니다 |

- 401 `UNAUTHENTICATED` (로그인 안 함·탈퇴), 403 `FORBIDDEN` (관리자 아님)

## 5. 고치기 `PUT /api/v1/notices/{noticeId}` (관리자)
본문과 오류는 쓰기와 같다. 200 `{ "noticeId": 3 }`. 없으면 404.

## 6. 지우기 `DELETE /api/v1/notices/{noticeId}` (관리자)
204. 없으면 404.

## 7. 로그인 상태 `GET /api/v1/auth/me` (칸 더하기)
```json
{ "member": { "id": 1, "nickname": "글쓴이", "admin": false } }
```
