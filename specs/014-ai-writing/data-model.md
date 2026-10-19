# Data Model: AI 빨간 펜과 관련 글 (014)

## 1. `post_embedding` (새 표, Flyway V14, ERD E-11)

| 칸 | 타입 | 설명 |
|---|---|---|
| `post_id` | BIGINT PK, FK → `post` ON DELETE CASCADE | 글 하나에 한 줄 |
| `model` | VARCHAR(50) NOT NULL | 만든 임베딩 모델 (`voyage-3.5`, 개발은 `fake`). 모델을 바꾸면 다시 만든다 |
| `content_hash` | CHAR(64) NOT NULL | 제목+본문 앞부분의 sha256. 바뀌었는지 본다 |
| `embedding` | REAL[] NOT NULL | 길이 1024 (`related.embedding.dimensions`), 길이 1로 맞춘 값 |
| `updated_at` | TIMESTAMPTZ NOT NULL | |

- 공개 여부는 저장하지 않는다. 고를 때 글·분류·회원을 다시 본다(공개 규칙은 한곳).
- 글을 지우면 같이 지워진다.

## 2. Redis (잃어도 되는 값)

| 키 | 값 | 수명 |
|---|---|---|
| `proofread:count:{회원}:{한국 날짜}` | 그날 교정 횟수 | 다음 날 0시 + 여유 |
| `proofread:result:{sha256(본문)}` | 교정 결과 JSON | `proofread.cache-ttl` (1일) |

## 3. 공지 (Flyway V15)
- 0.8.0 업데이트 글 하나(운영팀 글, `users_id` 비움), 고정. 0.7.0 운영팀 글의 고정을 푼다.
