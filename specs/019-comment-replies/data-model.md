# Data model: 답글·비밀 댓글 (019)

Flyway `V25__comment_replies.sql`.

| 표 | 바뀌는 것 |
|---|---|
| `comment` | `deleted_at TIMESTAMPTZ` 더함(지운 칸). `parent_id`·`is_secret`을 쓰기 시작한다. `CHECK (parent_id <> comment_id)`. 부분 색인 `idx_comment_parent (parent_id) WHERE parent_id IS NOT NULL` |
| `notification` | `kind`에 `reply`(내 댓글에 답글) 더함 |

규칙
- `parent_id`는 늘 원댓글(부모가 없는 댓글)을 가리킨다.
- 지운 칸: `deleted_at`이 있고 `body`는 빈 글자. 쓴 사람 번호는 남긴다(비밀 답글을 원댓글 쓴 사람이 읽을 수 있게).
- 글을 지우면 그 글의 댓글은 한 문장으로 모두 지운다(답글·원댓글이 같은 문장이라 외래 키가 막지 않는다).

ERD: `ERD-변경-요청.md` E-16.
