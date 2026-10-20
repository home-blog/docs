# Data Model: 글결 MCP OAuth (023)

## 표 `oauth_client` (Flyway V33, ERD E-20)
| 칸 | 타입 | 설명 |
|---|---|---|
| `client_id` | VARCHAR(64) PK | 무작위 |
| `client_name` | VARCHAR(100) NOT NULL | |
| `redirect_uris` | TEXT NOT NULL | 줄바꿈으로 나눈 1~5개 |
| `created_at` | TIMESTAMPTZ NOT NULL | |
| `last_used_at` | TIMESTAMPTZ NOT NULL | 등록·인가·토큰 요청 때 고침(1분에 한 번까지). 색인 있음 |

- 연결이 하나도 없고 30일 동안 쓰이지 않은(`last_used_at`) 앱은 매일 지운다 (보안 검토: 등록한 날이 아니라 마지막으로 쓴 날).

## 표 `oauth_grant` (V33)
| 칸 | 타입 | 설명 |
|---|---|---|
| `oauth_grant_id` | BIGINT IDENTITY PK | |
| `users_id` | BIGINT NOT NULL FK users (CASCADE) | |
| `client_id` | VARCHAR(64) NOT NULL FK oauth_client (CASCADE) | |
| `scope` | VARCHAR(10) NOT NULL | `read` · `draft` |
| `refresh_hash` | CHAR(64) NOT NULL UNIQUE | 지금 갱신 토큰의 SHA-256 |
| `refresh_expires_at` | TIMESTAMPTZ NOT NULL | 30일 |
| `created_at`, `last_used_at` | TIMESTAMPTZ | |

## 표 `oauth_refresh_used` (V33)
이미 바뀐 갱신 토큰 전부. 한 대 전 것만 보면 훔친 토큰으로 두 번 넘게 돌렸을 때 피해자의 옛 토큰을 알아채지 못한다 (CodeRabbit 지적, PR #107).

| 칸 | 타입 | 설명 |
|---|---|---|
| `refresh_hash` | CHAR(64) PK | 바뀐 옛 갱신 토큰의 SHA-256 |
| `oauth_grant_id` | BIGINT NOT NULL FK oauth_grant (CASCADE) | 연결이 지워지면 함께 지운다(지난 연결은 매일 지우므로 따로 정리하지 않는다) |
| `next_refresh_hash` | CHAR(64) NOT NULL | 이 토큰 대신 준 다음 토큰. 그것이 지금 토큰이면 "바로 전 것" |
| `rotated_at` | TIMESTAMPTZ NOT NULL | 바뀐 때 |

- 갱신에 성공하면 같은 트랜잭션에서 옛 토큰을 넣는다.
- 지금 토큰이 아닌 갱신 토큰을 **같은 앱**이 보내면: 바로 전 것이고 `rotated_at`에서 `mcp.oauth.refresh-reuse-grace`(60초) 안이면 거절만, 그 밖(유예가 지났거나 더 옛것)이면 연결을 끊는다. 다른 앱이 보내면 거절만.

## 표 `oauth_access_token` (V33)
| 칸 | 타입 | 설명 |
|---|---|---|
| `token_hash` | CHAR(64) PK | SHA-256 |
| `oauth_grant_id` | BIGINT NOT NULL FK oauth_grant (CASCADE) | |
| `expires_at` | TIMESTAMPTZ NOT NULL | 1시간 |

## 표 `users`에 더한 칸 (V33)
| 칸 | 타입 | 설명 |
|---|---|---|
| `oauth_revoked_at` | TIMESTAMPTZ NULL | 연결을 모두 끊은 때(비밀번호 변경·정지·열쇠 모두 끊기). 이보다 먼저 받은 허락 코드는 토큰으로 바꿀 수 없다 |

## Redis (잃어도 되는 값)
- `myblog:oauth:req:{id}` 인가 요청(클라이언트, 돌아갈 주소, state, code_challenge, scope, resource) 10분
- `myblog:oauth:code:{sha256}` 허락 코드(연결 번호, 돌아갈 주소, code_challenge, 클라이언트) 5분, 한 번 꺼내면 지운다(GETDEL). 코드를 만든 때(`issuedAt`)를 함께 둔다
- `myblog:oauth:used:{sha256}` 쓴 코드 표시(클라이언트, 연결 번호) 5분. 토큰을 만든 트랜잭션이 커밋된 뒤에 둔다. 같은 클라이언트가 다시 보낼 때만 그 연결을 끊는다
- `myblog:oauth:error:{id}` 위임장 전 오류(앱 이름, 호스트, 오류, 서버가 만든 돌아갈 주소) 10분
