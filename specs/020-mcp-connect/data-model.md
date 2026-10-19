# Data model: 글결 MCP (020)

Flyway `V27__access_key.sql`.

| 칸 | 뜻 |
|---|---|
| `access_key_id` | 번호 |
| `users_id` | 주인 (회원 FK) |
| `name` | 이름 1~40자 |
| `token_hash` | 열쇠 원문의 SHA-256 (64자, 유일) |
| `token_hint` | 앞 8글자(`gk_ab12c`) |
| `scope` | `read` · `draft` |
| `created_at`, `expires_at`, `last_used_at` | 만든·끝나는·마지막 사용 시각 (마지막 사용은 1분에 한 번만 고친다) |

- 회원이 탈퇴하면 그 회원의 열쇠를 지운다(탈퇴 이벤트). 탈퇴했는지는 열쇠를 쓸 때도 다시 본다.
- 설정: `access-key.max-per-member`(5), `access-key.allowed-days`(30, 90, 365), `mcp.rate-limit.per-minute`(60), `mcp.search.*`(후보 50, 단락 600자, 결과 8·글마다 2).

ERD: `ERD-변경-요청.md` E-17.
