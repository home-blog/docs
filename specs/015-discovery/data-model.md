# Data Model: 발견 (015)

## Redis (잃어도 되는 값)

| 키 | 값 | 수명 |
|---|---|---|
| `search:count:{10분 칸 번호(epoch초 / 600)}` | 정렬 집합: 검색어 → 센 수 | 70분 |
| `search:seen:{sha256(사람, 검색어, 10분 칸)}` | "1" (사람과 검색어가 글자로 남지 않게 해시) | 10분 |
| (서버 메모리) 오늘의 이슈 | 고른 글 번호와 날짜 | 5분 (Redis 대신 서버 메모리, 첫 화면 머리 분량과 같은 방식) |

## 표 `sentence_clip` (Flyway V17, ERD E-12)

| 칸 | 타입 | 설명 |
|---|---|---|
| `sentence_clip_id` | BIGSERIAL PK | |
| `users_id` | BIGINT NOT NULL FK users ON DELETE CASCADE | 모은 사람 |
| `post_id` | BIGINT NULL FK post ON DELETE SET NULL | 원문 글(지워지면 비움) |
| `post_title` | VARCHAR(100) NOT NULL | 모을 때의 제목 |
| `blog_name` | VARCHAR(30) NOT NULL | 모을 때의 블로그 이름 |
| `text` | VARCHAR(300) NOT NULL | 모은 문장 |
| `created_at` | TIMESTAMPTZ NOT NULL | |

- 같은 사람·같은 글·같은 문장은 하나: `UNIQUE (users_id, post_id, text)`.
- 목록: `INDEX (users_id, created_at DESC)`. 글마다 모은 사람 수: `INDEX (post_id)`.
