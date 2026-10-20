# Data Model: 글결 MCP OAuth (023)

## 표 `oauth_client` (Flyway V33, ERD E-20)
| 칸 | 타입 | 설명 |
|---|---|---|
| `client_id` | VARCHAR(64) PK | 무작위 |
| `client_name` | VARCHAR(100) NOT NULL | |
| `redirect_uris` | TEXT NOT NULL | 줄바꿈으로 나눈 1~5개 |
| `created_at` | TIMESTAMPTZ NOT NULL | |

- 연결이 하나도 없고 30일 지난 앱은 매일 지운다.

## 표 `oauth_grant` (V33)
| 칸 | 타입 | 설명 |
|---|---|---|
| `oauth_grant_id` | BIGINT IDENTITY PK | |
| `users_id` | BIGINT NOT NULL FK users (CASCADE) | |
| `client_id` | VARCHAR(64) NOT NULL FK oauth_client (CASCADE) | |
| `scope` | VARCHAR(10) NOT NULL | `read` · `draft` |
| `refresh_hash` | CHAR(64) NOT NULL UNIQUE | 지금 갱신 토큰의 SHA-256 |
| `previous_refresh_hash` | CHAR(64) NULL | 바로 전 갱신 토큰(다시 오면 도난으로 보고 끊는다) |
| `refresh_expires_at` | TIMESTAMPTZ NOT NULL | 30일 |
| `created_at`, `last_used_at` | TIMESTAMPTZ | |

## 표 `oauth_access_token` (V33)
| 칸 | 타입 | 설명 |
|---|---|---|
| `token_hash` | CHAR(64) PK | SHA-256 |
| `oauth_grant_id` | BIGINT NOT NULL FK oauth_grant (CASCADE) | |
| `expires_at` | TIMESTAMPTZ NOT NULL | 1시간 |

## Redis (잃어도 되는 값)
- `myblog:oauth:req:{id}` 인가 요청(클라이언트, 돌아갈 주소, state, code_challenge, scope, resource) 10분
- `myblog:oauth:code:{sha256}` 허락 코드(연결 번호, 돌아갈 주소, code_challenge, 클라이언트) 5분, 한 번 꺼내면 지운다(GETDEL)
