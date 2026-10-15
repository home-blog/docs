# Data Model: 공지사항 (012)

Flyway `V11__notice.sql`. ERD 기록은 [`ERD-변경-요청.md`](../../docs/3-설계/ERD-변경-요청.md) E-9.

## 1. `users.role` (칸 더하기)

| 칸 | 모양 | 규칙 |
|---|---|---|
| `role` | `VARCHAR(10) NOT NULL DEFAULT 'member'` | `member` 또는 `admin` (`ck_users_role`). 이미 있는 회원은 모두 `member` |

관리자 지정 (운영자가 DB에서, D-3):
```sql
UPDATE users SET role = 'admin' WHERE lower(email) = lower('관리자 이메일') AND deleted_at IS NULL;
```

## 2. `notice` (새 표)

| 칸 | 모양 | 규칙 |
|---|---|---|
| `notice_id` | `BIGINT` identity, PK | |
| `kind` | `VARCHAR(10) NOT NULL` | `update`·`notice` (`ck_notice_kind`) |
| `title` | `VARCHAR(100) NOT NULL` | 1~100자(코드 포인트, PostgreSQL도 글자 단위로 센다. 글 제목과 같다) |
| `content` | `TEXT NOT NULL` | 1~10,000자, 마크다운 원문 |
| `release_version` | `VARCHAR(20)` | `update`일 때만 있다, `notice`면 비어 있다 (`ck_notice_release_version`). 모양 `^\d{1,4}\.\d{1,4}\.\d{1,4}$` |
| `pinned` | `BOOLEAN NOT NULL DEFAULT false` | 고정 |
| `users_id` | `BIGINT NOT NULL`, FK → `users` | 쓴 관리자 |
| `created_at` | `TIMESTAMPTZ NOT NULL` | |
| `updated_at` | `TIMESTAMPTZ` | 고쳤을 때 |

인덱스: `idx_notice_list (kind, pinned DESC, created_at DESC)`, 전체 탭용 `idx_notice_order (pinned DESC, created_at DESC)`.

## 3. 상태 흐름
쓰기 → (고치기)* → 지우기(줄 삭제). 고정은 고치기로 켜고 끈다.
