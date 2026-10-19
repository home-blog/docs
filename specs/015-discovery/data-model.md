# Data Model: 발견 (015)

## Redis (잃어도 되는 값)

| 키 | 값 | 수명 |
|---|---|---|
| `search:count:{yyyyMMddHHmm, 10분 단위}` | 정렬 집합: 검색어 → 센 수 | 70분 |
| `search:seen:{사람}:{검색어}` | "1" | 10분 |
| `search:trending` | 순위 JSON(지금·10분 전) | 30초 |
| `today-issue` | 글 번호 목록 | 5분 |

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
