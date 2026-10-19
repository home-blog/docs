# Data Model: 편집실 (018)

## 칸 더하기 `post` (Flyway V23, ERD E-15)
| 칸 | 타입 | 설명 |
|---|---|---|
| `locked_at` | TIMESTAMPTZ NULL | 편집실이 내린 시각. 비어 있지 않으면 주인이 공개로 바꿀 수 없다 |
| `lock_reason` | VARCHAR(200) NULL | 내림 사유 |

## 칸 더하기 `post_report` (V23)
| 칸 | 타입 | 설명 |
|---|---|---|
| `resolved_at` | TIMESTAMPTZ NULL | 신고를 닫은 시각. 비어 있으면 검토 대기 |
| `resolution` | VARCHAR(20) NULL | `taken_down`(내림) · `dismissed`(반려). CHECK: 둘 다 비거나 둘 다 있음 |

- 부분 인덱스 `(post_id) WHERE resolved_at IS NULL`.

## 표 `board_report` (V23)
| 칸 | 타입 | 설명 |
|---|---|---|
| `board_report_id` | BIGINT IDENTITY PK | |
| `users_id` | BIGINT NOT NULL FK users (CASCADE) | 신고한 사람 |
| `board_post_id` | BIGINT NOT NULL FK board_post (CASCADE) | 글을 지우면 함께 지운다 |
| `reason` | VARCHAR(20) NOT NULL | SPAM · ABUSE · ADULT · OTHER |
| `detail` | VARCHAR(200) NULL | 기타일 때만 |
| `created_at` | TIMESTAMPTZ NOT NULL | |
| `resolved_at`, `resolution` | | 반려만(`dismissed`). 지우기는 글과 함께 줄이 사라진다 |

- `UNIQUE (users_id, board_post_id)`, 부분 인덱스 `(board_post_id) WHERE resolved_at IS NULL`.

## 표 `editorial_action` (V23)
| 칸 | 타입 | 설명 |
|---|---|---|
| `editorial_action_id` | BIGINT IDENTITY PK | |
| `target_kind` | VARCHAR(10) NOT NULL | `post` · `board` |
| `post_id` | BIGINT NULL FK post (SET NULL) | |
| `board_post_id` | BIGINT NULL FK board_post (SET NULL) | 지운 글은 비어 있다 |
| `target_title` | VARCHAR(100) NOT NULL | 그때의 제목 |
| `action` | VARCHAR(20) NOT NULL | `taken_down` · `dismissed` · `removed` · `unlocked` |
| `reason` | VARCHAR(200) NULL | 내림·지우기 사유 |
| `report_count` | INT NOT NULL | 그때 닫은 신고 수 |
| `users_id` | BIGINT NULL FK users (SET NULL) | 처리한 관리자 |
| `created_at` | TIMESTAMPTZ NOT NULL | |

## 칸 더하기 `notification` (V23)
- `kind` CHECK에 `editorial`을 더한다. `note VARCHAR(200)`: 편집실 소식의 사유.
