# Data Model: 운영 강화 (022)

## 표 `comment_report` (Flyway V31, ERD E-19)
| 칸 | 타입 | 설명 |
|---|---|---|
| `comment_report_id` | BIGINT IDENTITY PK | |
| `users_id` | BIGINT NOT NULL FK users (CASCADE) | 신고한 사람 |
| `comment_id` | BIGINT NOT NULL FK comment (CASCADE) | 댓글을 지우면 함께 지운다 |
| `reason` | VARCHAR(20) NOT NULL | SPAM · ABUSE · ADULT · OTHER |
| `detail` | VARCHAR(200) NULL | 기타일 때만 |
| `created_at` | TIMESTAMPTZ NOT NULL | |
| `resolved_at`, `resolution` | | `hidden`(가림) · `dismissed`(반려). 둘 다 비면 대기 |

- `UNIQUE (users_id, comment_id)`, 부분 인덱스 `(comment_id) WHERE resolved_at IS NULL`.

## 칸 더하기 `comment` (V31)
| 칸 | 타입 | 설명 |
|---|---|---|
| `hidden_at` | TIMESTAMPTZ NULL | 가린 시각 |
| `hidden_by` | VARCHAR(10) NULL | `auto`(신고가 모여서) · `editor`(관리자). CHECK: hidden_at과 함께 |

## 칸 더하기 `users` (V31)
| 칸 | 타입 | 설명 |
|---|---|---|
| `suspended_until` | TIMESTAMPTZ NULL | 이 시각 전까지 정지 |
| `suspend_reason` | VARCHAR(200) NULL | 정지 사유 |

## 칸 바꾸기 `editorial_action` (V31)
- `comment_id BIGINT NULL FK comment (SET NULL)`, `member_id BIGINT NULL FK users (SET NULL)`.
- `target_kind` CHECK: `post`·`board`·`comment`·`member`. `action` CHECK에 `hidden`·`restored`·`suspended`·`unsuspended`.
- `target_title`: 댓글이면 그 글의 제목, 회원이면 그때의 닉네임.
