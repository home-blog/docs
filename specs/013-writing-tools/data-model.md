# Data Model: 글쓰기 도구 (013)

Flyway `V12__writing_tools.sql`. ERD 기록은 [`ERD-변경-요청.md`](../../docs/3-설계/ERD-변경-요청.md) E-10. 글 표(`post`)는 바꾸지 않는다(research D-1, D-9).

## 1. `post_draft` (새 표, 임시 저장·예약)

| 칸 | 모양 | 규칙 |
|---|---|---|
| `post_draft_id` | `BIGINT` identity, PK | |
| `blog_id` | `BIGINT NOT NULL`, FK → `blog` | 쓴 사람의 블로그(주인 = 블로그 주인) |
| `category_id` | `BIGINT`, FK → `category` `ON DELETE SET NULL` | 고르지 않았으면 비어 있다 |
| `topic_id` | `BIGINT`, FK → `topic` | 고르지 않았으면 비어 있다 |
| `series_id` | `BIGINT`, FK → `series` `ON DELETE SET NULL` | 발행할 때 넣을 연재 |
| `title` | `VARCHAR(100) NOT NULL DEFAULT ''` | 0~100자 |
| `content` | `TEXT NOT NULL DEFAULT ''` | 0자 ~ 글 본문 상한 |
| `visibility` | `VARCHAR(10) NOT NULL DEFAULT 'public'` | `public`·`private` |
| `tags` | `VARCHAR(200) NOT NULL DEFAULT ''` | 태그 글자를 쉼표로 이은 것(발행할 때 글 태그 규칙으로 검사) |
| `publish_at` | `TIMESTAMPTZ` | 비어 있으면 임시 저장, 있으면 예약 |
| `request_key` | `VARCHAR(36) NOT NULL UNIQUE` | 발행할 때 글의 `request_key`로 쓴다(두 번 생기지 않게, D-3) |
| `created_at` | `TIMESTAMPTZ NOT NULL` | |
| `updated_at` | `TIMESTAMPTZ NOT NULL` | 마지막 저장 시각("임시 저장됨 14:32") |

인덱스: `idx_post_draft_blog (blog_id, updated_at DESC)`, 예약 찾기 `idx_post_draft_publish_at (publish_at) WHERE publish_at IS NOT NULL`.

## 2. `series` (새 표, 연재)

| 칸 | 모양 | 규칙 |
|---|---|---|
| `series_id` | `BIGINT` identity, PK | |
| `blog_id` | `BIGINT NOT NULL`, FK → `blog` | |
| `name` | `VARCHAR(30) NOT NULL` | 1~30자. 한 블로그 안에서 겹쳐도 된다 |
| `description` | `VARCHAR(200) NOT NULL DEFAULT ''` | 0~200자 |
| `created_at` | `TIMESTAMPTZ NOT NULL` | |

인덱스: `idx_series_blog (blog_id, created_at)`.

## 3. `series_post` (새 표, 연재에 든 글)

| 칸 | 모양 | 규칙 |
|---|---|---|
| `series_id` | `BIGINT NOT NULL`, FK → `series` `ON DELETE CASCADE` | 연재를 지우면 줄도 지운다(글은 남는다) |
| `post_id` | `BIGINT NOT NULL`, FK → `post`, **UNIQUE** | 글 하나 = 연재 하나 |
| `sort_order` | `INTEGER NOT NULL` | 화 순서(작을수록 앞). 화 번호는 저장하지 않는다(D-10) |

PK `(series_id, post_id)`. 인덱스 `idx_series_post_order (series_id, sort_order)`. 글을 지울 때는 글 모듈이 이 줄을 먼저 지운다.

## 4. `post_image.post_draft_id` (칸 더하기)

| 칸 | 모양 | 규칙 |
|---|---|---|
| `post_draft_id` | `BIGINT`, FK → `post_draft` `ON DELETE SET NULL` | 임시 저장 글 본문에 든 이미지. 글에 연결되면(`post_id`) 그대로 둔다 |

정리 작업(`deleteUnlinkedBefore`)은 `post_id IS NULL AND post_draft_id IS NULL`인 것만 지운다. 연결 전 이미지 인덱스를 `WHERE post_id IS NULL AND post_draft_id IS NULL`로 바꾼다.

## 5. 상태 흐름

```text
(새 글 쓰기) ─임시 저장→ 임시 저장 글 ─발행→ 글 (임시 저장 글 지움)
                            │  ▲
                        예약 │  │ 예약 취소 / 올리기 실패
                            ▼  │
                         예약 글 ─시각이 됨(작업)→ 글 (임시 저장 글 지움)
```
