# Data Model: 구독·알림 (016)

## 표 `blog_subscription` (Flyway V19, ERD E-13)

| 칸 | 타입 | 설명 |
|---|---|---|
| `blog_subscription_id` | BIGINT IDENTITY PK | |
| `users_id` | BIGINT NOT NULL FK users | 받아 보는 사람 |
| `blog_id` | BIGINT NOT NULL FK blog | 받아 보는 블로그 |
| `email_enabled` | BOOLEAN NOT NULL DEFAULT false | 원고 편지(메일)도 받기 |
| `created_at` | TIMESTAMPTZ NOT NULL | |

- `UNIQUE (users_id, blog_id)`, `INDEX (blog_id)`.

## 표 `notification` (V19)

| 칸 | 타입 | 설명 |
|---|---|---|
| `notification_id` | BIGINT IDENTITY PK | |
| `users_id` | BIGINT NOT NULL FK users | 받는 사람 |
| `kind` | VARCHAR(10) NOT NULL | `post`(새 글) · `comment`(내 글에 댓글) |
| `post_id` | BIGINT NULL FK post ON DELETE SET NULL | |
| `post_title` | VARCHAR(100) NOT NULL | 그때의 글 제목 |
| `actor_name` | VARCHAR(30) NOT NULL | 새 글이면 블로그 이름, 댓글이면 댓글 쓴 사람 닉네임 |
| `read_at` | TIMESTAMPTZ NULL | |
| `created_at` | TIMESTAMPTZ NOT NULL | |

- `INDEX (users_id, created_at DESC)`, 안 읽은 수: `INDEX (users_id) WHERE read_at IS NULL`.
- 같은 글 새 글 소식은 한 사람에게 하나: `UNIQUE (users_id, post_id, kind) WHERE kind = 'post'`.

## 칸 더하기 `post.first_published_at` (V19)
- 처음 공개된 시각. 이미 있는 공개 글은 `created_at`으로 채운다. 비어 있는 글이 공개될 때만 채우고 `PostPublishedEvent`를 낸다.

## Redis
| 키 | 값 | 수명 |
|---|---|---|
| `newsletter:sent:{yyyy-MM-dd}` | 오늘 보낸 메일 수 | 2일 |
