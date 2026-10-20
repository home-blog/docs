# ERD: 팀 공통 요청과 내 확장

> 작성: JaeUng · 2026-10-07 (2026-10-08 다시 정리) · 상태: **팀 확인 전**
> 기준: 팀원들과 만든 ERD **최신판(2026-10-07에 받은 판)**
> 근거: `specs/001` ~ `specs/006`의 명세와 기술 계획의 결정(`research.md`의 `D-항목`), 팀 공통 범위는 `docs/1-팀공통/요구사항-공통.md`

## 정리 원칙 (2026-10-08)

팀 공통 ERD에는 **네 명이 함께 쓰는 기본 기능**(`요구사항-공통.md`의 CF-01 ~ CF-22)에 꼭 필요한 것만 요청합니다. 제 개인 상세 규칙(로그인 잠금, 재가입, 중복 저장 방지, 블로그 관리·통계 등)에 필요한 칸과 인덱스는 **제 블로그를 구현할 때 제 저장소에서 더합니다.** 공통 표에 칸과 인덱스를 **더하기만** 하므로 공통 ERD와 부딪히지 않습니다.

## 0. 이미 반영된 것

이전 판에서 문제로 보았던 것 중 최신판에서 이미 고쳐진 것입니다.

| 표 | 바뀐 것 | 덕분에 지켜지는 요구사항 |
|---|---|---|
| `comment` | `UNIQUE(users_id, post_id)` 삭제 | 한 사람이 한 글에 댓글을 여러 개 달 수 있다 (CF-18) |
| `category` | `sort_order` 추가 | 분류 순서 바꾸기 (CF-08) |
| `post_report` | `UNIQUE(users_id, post_id)` 추가 | 같은 글을 두 번 신고할 수 없다 (CF-21) |
| `blog_daily_stat` | 기본값 0, `UNIQUE(blog_id, stat_date)` 위치 정리 | 하루에 한 줄 |

## 1. 팀에 요청할 것 (공통, 3건)

| # | 표 | 지금 | 제안 | 왜 공통인가 | 관련 |
|---|---|---|---|---|---|
| T-1 | `post_image` | `post_id` 필수 | **검토 요청**: `post_id` NULL 허용 + 올린 사람(`users_id`), 올린 시각(`created_at`) 추가 | 글쓰기 중에 이미지를 넣으려면(CF-22) 아직 없는 글에는 이미지 기록을 만들 수 없다. 누가 구현하든 같은 문제를 만난다. 글을 저장한 뒤에만 올리게 할 거라면 지금 그대로 둬도 된다 | CF-22, `005` D-3 |
| T-2 | `post` | `content` 기본값 `''` | 기본값 없애기 | 본문은 필수(CF-06)라 빈 값으로 저장되면 안 된다 | CF-06 |
| T-3 | `post`, `category`, `post_report`, `comment_report` | 값 목록이 주석에만 있음 | `CHECK` 추가: `visibility IN ('public','private')`, `reason IN ('SPAM','ABUSE','ADULT','OTHER')` | 오타 값(`Public` 등)이 들어가면 비공개 글이 새어 나간다. DB가 값을 막아 준다 | CF-13, CF-21 |

```sql
-- T-1 (검토)
ALTER TABLE post_image ALTER COLUMN post_id DROP NOT NULL;
ALTER TABLE post_image ADD COLUMN users_id BIGINT NOT NULL REFERENCES users (users_id);
ALTER TABLE post_image ADD COLUMN created_at TIMESTAMPTZ NOT NULL;

-- T-2
ALTER TABLE post ALTER COLUMN content DROP DEFAULT;

-- T-3
ALTER TABLE post           ADD CONSTRAINT ck_post_visibility     CHECK (visibility IN ('public', 'private'));
ALTER TABLE category       ADD CONSTRAINT ck_category_visibility CHECK (visibility IN ('public', 'private'));
ALTER TABLE post_report    ADD CONSTRAINT ck_post_report_reason    CHECK (reason IN ('SPAM', 'ABUSE', 'ADULT', 'OTHER'));
ALTER TABLE comment_report ADD CONSTRAINT ck_comment_report_reason CHECK (reason IN ('SPAM', 'ABUSE', 'ADULT', 'OTHER'));
```

**참고 (요청 아님)**: 지금 DDL은 MySQL 문법(`` ` ``, `COMMENT`)과 PostgreSQL 타입(`TIMESTAMPTZ`)이 섞여 있습니다. DB가 PostgreSQL(가안)이면 실제 DDL을 만들 때 PostgreSQL 문법으로 정리하면 됩니다.

## 2. 내 확장 (내 블로그를 구현할 때 내 저장소에서 추가)

팀 ERD는 바꾸지 않고, 제 저장소의 DB 변경 파일(마이그레이션)로 더합니다.

| # | 표 | 더하는 것 | 왜 | 관련 |
|---|---|---|---|---|
| E-1 | `users` | `failed_login_count`(INT, 기본 0), `locked_until`(TIMESTAMPTZ, NULL) | 로그인 5회 실패 10분 잠금. **Redis로 옮길지는 구현 때 다시 정한다** | `001` FR-027, D-3 |
| E-2 | `users` | 기존 `UNIQUE` 대신 소문자 비교 인덱스 ("탈퇴하지 않은 회원" 조건은 있어도 되고 없어도 된다) | `Kim`과 `kim`은 같은 닉네임. 재가입은 탈퇴 때 이메일·닉네임을 바꾸므로 평범한 `UNIQUE`로도 된다 | `001` D-4, `002` D-2 |
| E-3 | `category` | 이름 소문자 비교 인덱스, `color_index`(SMALLINT) | "Java"와 "java"는 같은 분류, 분류 색 | `003` D-5, `006` D-9 |
| E-4 | `post` | `request_key`(VARCHAR(36), NULL, 중복 불가) | 저장을 여러 번 눌러도 한 번만 저장 | `003` D-6 |
| E-5 | **새 표** `post_daily_stat` | `post_id`, `stat_date`, `views` | 최근 7일 인기 글 | `006` D-6 |
| E-6 | `blog` | `comments_read_at`(TIMESTAMPTZ, NULL) | 새 댓글 N개 | `006` D-8 |
| E-7 | `post_report` | `UNIQUE(users_id, post_id)`를 **실제 제약으로** 둔다 | 같은 회원이 같은 글을 두 번 신고할 수 없다(CF-21-3). 팀 ERD 최신판에도 규칙은 적혀 있지만(0절), 내 DB에서는 이 제약이 동시에 두 번 눌러도 한 건만 남게 하는 마지막 방어선이라 내 마이그레이션에 확실히 넣는다 (2026-10-08 사용자 결정) | `005` FR-019, SC-003 |
| E-8 | `users`, **새 표** `user_social_account` | `users.password`를 NULL 허용(소셜로만 가입한 회원). `user_social_account`(`users_id`, `provider` github·google·kakao·naver, `provider_user_id`, `linked_at`), `UNIQUE(provider, provider_user_id)`, `UNIQUE(users_id, provider)`. 회사 토큰은 두지 않는다 | 소셜 로그인 (2026-10-12, 개별 프로젝트 기능) | `010` D-4, D-7, D-10, Flyway V10 |
| E-9 | `users`, **새 표** `notice` | `users.role`(`member`·`admin`, 기본 `member`). `notice`(`kind` update·notice, `title`, `content`, `release_version`은 update일 때만, `pinned`, `users_id` 쓴 관리자(V13부터 비울 수 있음: 운영팀이 처음 넣은 글), `created_at`, `updated_at`) | 공지사항 (2026-10-15, 개별 프로젝트 기능 N-24) | `012` D-1, D-4, D-5, Flyway V11, V13(첫 글 13개) |
| E-10 | **새 표** `post_draft`, `series`, `series_post`, `post_image` | `post_draft`(임시 저장·예약: `blog_id`, 비어 있어도 되는 `category_id`·`topic_id`·`series_id`, `title`, `content`, `visibility`, `tags`, `publish_at` 예약 시각, `request_key` 발행용 고유 번호, `created_at`, `updated_at`). `series`(`blog_id`, `name`, `description`, `created_at`). `series_post`(`series_id`, `post_id` 고유, `sort_order`). `post_image.post_draft_id`(임시 저장 글의 이미지) | 글쓰기 도구 (2026-10-15, 개별 프로젝트 기능 N-08~N-10). 글 표는 바꾸지 않는다 | `013` D-1, D-8, D-9, Flyway V12 |
| E-11 | **새 표** `post_embedding` | `post_embedding`(글의 뜻 정보: `post_id` PK·FK 글 지우면 같이 지움, `model` 임베딩 모델, `content_hash` 제목+본문 앞부분 sha256, `embedding` real[] 1024칸, `updated_at`). pgvector 대신 real[]: 배포 DB(공용 인스턴스의 스키마)에서 확장을 켤 수 없을 수 있어 비슷한 정도는 서버가 계산 | 비슷한 글 (2026-10-19, 개별 프로젝트 기능 N-14) | `014` D-9, Flyway V14 |
| E-12 | **새 표** `sentence_clip` | `sentence_clip`(문장 수집·필사 노트: `users_id` 모은 사람(FK, 탈퇴하면 서버가 지움), `post_id` 원문 글(FK, 비울 수 있음: 글이 지워지면 비움), `post_title`·`blog_name` 모을 때의 제목·블로그 이름, `text` 300자, `created_at`, UNIQUE(`users_id`, `post_id`, `text`)) | 문장 수집 (2026-10-19, 개별 프로젝트 기능 N-15) | `015` data-model, Flyway V17 |
| E-13 | **새 표** `blog_subscription`, `notification`, 칸 `post.first_published_at` | `blog_subscription`(원고 받아 보기: `users_id`, `blog_id`, `email_enabled` 메일로도 받기, `created_at`, UNIQUE(`users_id`, `blog_id`)). `notification`(우편함: `users_id`, `kind` post·comment, `post_id` 글이 지워지면 비움, `post_title`, `actor_name`, `read_at`, `created_at`, 새 글 소식은 한 사람·한 글 하나(부분 UNIQUE)). `post.first_published_at`(처음 공개된 시각, 새 글 소식은 이때 한 번) | 구독·알림 (2026-10-19, 개별 프로젝트 기능 N-16~N-18) | `016` data-model, Flyway V19 |
| E-14 | **새 표** `board_post`, `board_comment`, `notification` 칸 | `board_post`(글마당 글: `users_id`, `topic_id` 블로그 글과 같은 주제 표, `kind` talk·question, `title`, `content`, `views`, `comment_count`, `accepted_comment_id` 채택한 댓글(댓글이 지워지면 비움), `created_at`, `updated_at`). `board_comment`(`board_post_id` 글을 지우면 함께, `users_id`, `content` 1,000자, `created_at`). `notification`에 `board_post_id`(글이 지워지면 비움)와 `kind` board·accepted | 글마당(커뮤니티, 2026-10-19, `요구사항.md` 3.7) | `017` data-model, Flyway V21 |
| E-15 | **새 표** `board_report`, `editorial_action`, 칸 `post.locked_at`·`lock_reason`, `post_report.resolved_at`·`resolution`, `notification.note` | 편집실: 신고 처리 상태(대기·내림·반려), 내린 글 잠금, 글마당 신고(한 사람·한 글 한 번), 처리 기록(글이 지워져도 제목은 남음), 편집실 소식 사유 | 편집실(관리자 화면, 2026-10-19, 상세/05 CF-23-1 다음 단계) | `018` data-model, Flyway V23 |
| E-16 | 칸 `comment.deleted_at`, `comment.parent_id`·`is_secret`을 쓰기 시작, `notification.kind` reply | 답글(한 단계)·비밀 댓글(봉한 쪽지), 답글이 남은 댓글을 지우면 지운 칸 | 답글·비밀 댓글(2026-10-19, `005` D-7의 추후 확장) | `019` data-model, Flyway V25 |
| E-17 | **새 표** `access_key` | 개인 열쇠: AI 비서(MCP)가 내 블로그에 붙는 열쇠. 원문은 저장하지 않고 SHA-256 해시와 앞 8글자만, 범위(read·draft)와 기한 | 글결 MCP(2026-10-20) | `020` data-model, Flyway V27 |
| E-18 | **새 표** `guestbook_entry`, `notification`에 `guestbook_entry_id`·`blog_id` | 방명록 엽서(블로그·쓴 사람·내용 300자·우표·비밀·주인 답장 하나). 엽서를 지우면 소식도 지운다 | 방명록(2026-10-20) | `021` data-model, Flyway V30 |
| E-19 | **새 표** `comment_report`, `comment.hidden_at·hidden_by`, `users.suspended_until·suspend_reason`, `editorial_action.comment_id·member_id` | 댓글 신고(한 사람 한 번, 3건이면 저절로 가림), 회원 정지(기한이 지나면 저절로 풀림), 편집실 처리 기록에 댓글·회원 | 운영 강화(2026-10-20) | `022` data-model, Flyway V31 |
| E-20 | **새 표** `oauth_client`, `oauth_grant`, `oauth_access_token` | Claude 앱 커넥터(MCP OAuth): 동적 등록한 앱, 회원이 허락한 연결(범위·갱신 토큰 해시·옛 갱신 토큰 해시), 1시간 출입증 해시. 허락 요청·코드는 Redis | 글결 MCP OAuth(2026-10-20) | `023` data-model, Flyway V33 |

```sql
-- E-1
ALTER TABLE users ADD COLUMN failed_login_count INTEGER NOT NULL DEFAULT 0;
ALTER TABLE users ADD COLUMN locked_until TIMESTAMPTZ NULL;

-- E-2 (email, nickname의 기존 UNIQUE는 지운다)
CREATE UNIQUE INDEX uq_users_email_active    ON users (lower(email))    WHERE deleted_at IS NULL;
CREATE UNIQUE INDEX uq_users_nickname_active ON users (lower(nickname)) WHERE deleted_at IS NULL;

-- E-3 (기존 UNIQUE(blog_id, name)은 지운다)
CREATE UNIQUE INDEX uq_category_blog_name ON category (blog_id, lower(name));
ALTER TABLE category ADD COLUMN color_index SMALLINT NOT NULL DEFAULT 0;

-- E-4
ALTER TABLE post ADD COLUMN request_key VARCHAR(36) NULL UNIQUE;

-- E-5
CREATE TABLE post_daily_stat (
  post_daily_stat_id BIGSERIAL PRIMARY KEY,
  post_id    BIGINT  NOT NULL REFERENCES post (post_id),
  stat_date  DATE    NOT NULL,
  views      INTEGER NOT NULL DEFAULT 0,
  UNIQUE (post_id, stat_date)
);

-- E-6
ALTER TABLE blog ADD COLUMN comments_read_at TIMESTAMPTZ NULL;

-- E-7 (post_report를 만들 때 함께)
ALTER TABLE post_report ADD CONSTRAINT uq_post_report_user_post UNIQUE (users_id, post_id);
```

> E-2와 E-3은 기존 `UNIQUE`를 **대소문자 무시 비교로 바꾸는** 것입니다. 탈퇴 후 재가입은 탈퇴 때 이메일·닉네임을 바꾸므로(2026-10-08) 공통 ERD의 `UNIQUE` 그대로도 됩니다.

## 3. 바꾸지 않는 것 (공유만)

| 표 | 내용 | 이유 |
|---|---|---|
| `users` | 탈퇴해도 줄을 지우지 않고 `deleted_at`을 넣되, **개인정보는 알아볼 수 없게 바꾼다**: 이메일 `deleted-{번호}@deleted.invalid`, 비밀번호 해시 삭제(맞을 수 없는 값), 닉네임 `탈퇴한사용자{번호}`, 소개 비움. 그 사람의 댓글은 "탈퇴한 사용자"로 보인다 | 2026-10-08 ERD 피드백 반영. 칸 변경 없이 탈퇴 처리 코드로 한다 (`002` D-1, D-2) |
| `blog` | `users_id` `UNIQUE`(회원 1명 = 블로그 1개) 그대로 | 이번 범위는 1인 1블로그 (`003` D-4) |
| `post` | `topic_id` 그대로. 주제는 글마다 고른다 | 2026-10-07 결정 (`003` D-3) |
| `category` | `visibility` 그대로. 비공개 분류의 글은 주인만 본다 | 2026-10-07 팀 답변 (`003` D-5) |
| `post` | `blog_id` 없이 **분류를 거쳐** 블로그의 글을 찾는다 | 지금은 충분하다 (`004` D-6) |
| `comment` | `parent_id`, `is_secret`은 두되 이번에는 쓰지 않는다 | `005` D-7 |
| `comment_report` | 표는 두되 이번에는 쓰지 않는다 | `005` D-8 |
| 외래 키 | `ON DELETE CASCADE` 없이, 글을 지울 때 서버가 한 묶음으로 자식부터 지운다 | `003` D-7, `005` D-11 |

## 4. 팀에 묻고 싶은 것

1. **T-1**: 글을 쓰는 중에 이미지를 올리게 할 건가요, 글을 저장한 뒤에만 올리게 할 건가요?

**답을 받은 것 (2026-10-07)**
- `category.visibility`는 분류를 비공개로 하기 위한 칸이다 → 그대로 쓴다.
- 주제는 글을 쓸 때 고른다 → `post.topic_id`를 그대로 쓴다.

## 5. 이 정리가 끝나면

- 팀이 T-1 ~ T-3에 답하면 `001`의 작업 T006(팀 ERD 반영 확인)이 끝난다. T007(DB 표 만들기)에서는 **팀 ERD + 내 확장(E-1 ~ E-7)** 으로 표를 만든다.
- Crowfoot: `팀 공통 ERD` 문서(팀 ERD + T-1 ~ T-3)와 `myblog-제안 (2026-10-07)` 문서(= 내 확장판, 팀 ERD + T + E 전부)
