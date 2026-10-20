# Data Model: 방명록 (021)

## 표 `guestbook_entry` (Flyway V30, ERD E-18)
| 칸 | 타입 | 설명 |
|---|---|---|
| `guestbook_entry_id` | BIGINT IDENTITY PK | |
| `blog_id` | BIGINT NOT NULL FK blog (CASCADE) | 어느 블로그의 방명록 |
| `users_id` | BIGINT NOT NULL FK users (CASCADE) | 쓴 사람 (탈퇴는 soft delete라 줄은 남는다) |
| `body` | VARCHAR(300) NOT NULL | 엽서 내용 |
| `stamp` | VARCHAR(10) NOT NULL | `pine`·`wave`·`moon`·`plum`·`crane` (CHECK) |
| `is_secret` | BOOLEAN NOT NULL DEFAULT false | 쓴 사람·주인만 읽음 |
| `reply` | VARCHAR(300) NULL | 주인의 답장 |
| `replied_at` | TIMESTAMPTZ NULL | 답장을 처음 쓴 시각 (CHECK: reply와 함께 비거나 함께 있음) |
| `created_at` | TIMESTAMPTZ NOT NULL | |

- 인덱스 `(blog_id, created_at DESC, guestbook_entry_id DESC)`, `(users_id, created_at DESC)`.

## 칸 바꾸기 `notification` (V30)
- `kind` CHECK에 `guestbook`, `gb_reply`. `guestbook_entry_id BIGINT NULL FK guestbook_entry (CASCADE)`: 엽서를 지우면 소식도 지운다.
