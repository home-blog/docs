# Contracts: 구독·알림 (016)

## 1. 받아 보기 (로그인, CSRF)
- `PUT /api/v1/blogs/{blogId}/subscription` `{ "emailEnabled": false }`(본문 없으면 false) → 200 `{ "subscribed": true, "emailEnabled": false, "subscriberCount": 12 }`
  - 400 `SELF_SUBSCRIBE_NOT_ALLOWED`, 404 `BLOG_NOT_FOUND`, 409 `SUBSCRIPTION_LIMIT_EXCEEDED`
- `DELETE /api/v1/blogs/{blogId}/subscription` → 200 `{ "subscribed": false, "emailEnabled": false, "subscriberCount": 11 }`
- `GET /api/v1/blogs/{blogId}`(블로그 머리)에 `subscriberCount`, `subscribedByMe`(로그인 안 했으면 false), `subscriptionEmail`(내가 메일로도 받는지) 칸을 더한다.
- `GET /api/v1/me/subscriptions` → `{ "items": [ { "blogId", "blogName", "nickname", "emailEnabled", "createdAt" } ] }` 최근 순
- `GET /api/v1/me/subscriptions/feed?page=` → 첫 화면 글 모음과 같은 모양 `{ totalCount, page, totalPages, pageSize, posts: [카드] }`

## 2. 우편함 (로그인)
- `GET /api/v1/me/notifications?page=` → `{ "items": [ { "notificationId", "kind": "post|comment", "postId", "postTitle", "actorName", "available", "read", "createdAt" } ], "page", "totalPages", "unreadCount" }` 최근 순 20개씩
- `GET /api/v1/me/notifications/unread-count` → `{ "unreadCount": 3 }`
- `POST /api/v1/me/notifications/{id}/read` → 204 (남의 것·없는 것 404) (CSRF)
- `POST /api/v1/me/notifications/read-all` → 204 (CSRF)

## 3. 원고 편지 그만 받기 (로그인 없이)
- 메일 링크: `https://<사이트>/letters/stop?token=…` (화면이 확인을 받고 아래를 부른다)
- `POST /api/v1/newsletter/unsubscribe` `{ "token": "…" }` → 200 `{ "blogName": "바다 일기" }`, 잘못된 값 400 `NEWSLETTER_TOKEN_INVALID`. 이미 꺼졌거나 구독이 없어도 200(같은 결과)
