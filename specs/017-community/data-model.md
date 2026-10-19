# Data Model: 글마당 (017)

## 표 `board_post` (Flyway V21, ERD E-14)

| 칸 | 타입 | 설명 |
|---|---|---|
| `board_post_id` | BIGINT IDENTITY PK | |
| `users_id` | BIGINT NOT NULL FK users | 쓴 사람(탈퇴해도 줄은 남는다) |
| `topic_id` | BIGINT NOT NULL FK topic | |
| `kind` | VARCHAR(10) NOT NULL CHECK (`talk`, `question`) | 이야기·물음 |
| `title` | VARCHAR(100) NOT NULL | |
| `content` | TEXT NOT NULL | 마크다운 |
| `views` | BIGINT NOT NULL DEFAULT 0 | |
| `comment_count` | INT NOT NULL DEFAULT 0 | |
| `accepted_comment_id` | BIGINT NULL FK board_comment ON DELETE SET NULL | 채택한 댓글 |
| `created_at`, `updated_at` | TIMESTAMPTZ NOT NULL | |

- `INDEX (created_at DESC)`, `INDEX (topic_id, created_at DESC)`, `INDEX (users_id)`.

## 표 `board_comment` (V21)

| 칸 | 타입 | 설명 |
|---|---|---|
| `board_comment_id` | BIGINT IDENTITY PK | |
| `board_post_id` | BIGINT NOT NULL FK board_post ON DELETE CASCADE | |
| `users_id` | BIGINT NOT NULL FK users | |
| `content` | VARCHAR(1000) NOT NULL | |
| `created_at` | TIMESTAMPTZ NOT NULL | |

- `INDEX (board_post_id, created_at)`.

## 칸 더하기 `notification` (V21)
- `kind` CHECK에 `board_comment`, `board_accept`를 더한다.
- `board_post_id BIGINT NULL FK board_post ON DELETE SET NULL`. 블로그 글 소식은 `post_id`, 글마당 소식은 `board_post_id`를 쓴다.

## Redis
| 키 | 값 | 수명 |
|---|---|---|
| `view:board:{글}:{방문자}` (통계 모듈) | 30분 안에 본 표시 | 30분 |
