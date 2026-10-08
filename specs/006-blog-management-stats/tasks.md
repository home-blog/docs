---

description: "006 블로그 관리와 통계 작업 목록"
---

# Tasks: 블로그 관리와 통계

**Input**: `/specs/006-blog-management-stats/`의 설계 문서

**Prerequisites**: [plan.md](plan.md), [spec.md](spec.md), [research.md](research.md), [data-model.md](data-model.md), [contracts/manage-api.md](contracts/manage-api.md), [quickstart.md](quickstart.md)

**결정 반영 (2026-10-08)**: research의 `D-1`, `D-6 ~ D-9`는 정해졌다. **`D-2 ~ D-5`, `D-10`은 아직 `미정`** 이고, 원본(`상세/06`)의 `확인 필요` 4건(BM-01-3, BM-05-7, BM-06-5, BM-06-8)도 팀과 합의 전이다. 이 목록은 **정하지 않은 것을 정하지 않는다** (헌법 II). 미정에 기대는 작업은 "D-n 결정에 따름 (추천: …)"으로 적고, **정해진 것만으로 할 수 있는 작업을 앞에, 결정을 기다리는 작업을 뒤에** 두었다 (Phase 11, 12).

| ID | 내용 | 상태 | 이 목록에서 |
|---|---|---|---|
| D-1 | "30분 안에 봤다"·"오늘 왔다" 표시는 Redis에 시간제한을 걸어 둔다 (헌법 1.1.0) | 결정됨 | T053 |
| D-2 | 로그인하지 않은 방문자를 "같은 사람"으로 알아보는 방법 | **미정** (추천: A. 무작위 방문자 쿠키, 회원은 회원 번호) | T052, T065 |
| D-3 | 숫자를 바로 DB에 쓸까, Redis에 모았다 옮길까 | **미정** (추천: A. 바로 DB) | T054 |
| D-4 | 글 주인 본인이 자기 글·블로그를 열 때 셀까 | **미정** (추천: A. 세지 않는다, 방문자도 같은 기준) | T055 |
| D-5 | 방문자를 어느 화면에서 셀까 | **미정** (추천: B. 블로그의 모든 방문자용 화면) | T056 |
| D-6 | 인기 글은 글별 일별 조회수 표 `post_daily_stat`(E-5)로 최근 7일 순위, 그 7일 숫자를 보여 준다. 같으면 최근에 쓴 글 먼저 | 결정됨 | T003, T010, T044, T046 |
| D-7 | 누적 조회수·누적 방문자 = `blog_daily_stat` 모든 줄의 합 (지운 글의 조회수도 남는다) | 결정됨 | T044, T046 |
| D-8 | "댓글 관리를 마지막으로 연 시각"은 `blog.comments_read_at`(E-6, `V2`에 이미 있음) | 결정됨 | T032, T033 |
| D-9 | 분류에 색 번호 칸 `category.color_index`(E-3, `V2`에 이미 있음) | 결정됨 | T023, T025 |
| (색 목록) | D-9의 **색 값 목록**(몇 가지, 어떤 순서)은 시안에서 가져와 `상세/06` `기본값` 표에 먼저 적기로 했고 아직 적지 않았다 | **미정** (추천: 시안의 색 목록을 기본값 표에 먼저 적는다) | T025 |
| D-10 | 그래프를 그리는 화면 도구 | **미정** (추천: A. 가벼운 그래프 도구 하나, 구현 때 고른다) | T058 |
| BM-01-3 | 마이페이지와 블로그 관리를 둘로 나누는 구조 | **확인 필요** (팀). 원본 지금 내용대로 둘로 나눈다 | T016, T060 |
| BM-05-7 | 새 댓글 알림은 숫자와 `NEW`까지 (종 모양 목록·메일·푸시 없음) | **확인 필요** (팀). 원본 지금 내용대로 만들지 않는다 | T036, T060 |
| BM-06-5 | 조회수와 방문자를 따로 센다 (하루 글 3개 → 방문자 1, 조회수 3) | **확인 필요** (팀). 원본 지금 내용대로 따로 센다 | T051, T054, T060 |
| BM-06-8 | 유입 경로, 시간대별·기기별 분석은 만들지 않는다 | **확인 필요** (팀). 원본 지금 내용대로 저장도 하지 않는다 | T039, T060 |

**Tests**: `001` ~ `003`처럼 **사용자 이야기마다 서버 테스트 작업**을 넣었다 (`@SpringBootTest` + MockMvc + `springSecurity()`, 실제 PostgreSQL). 테스트 이름에는 [quickstart.md](quickstart.md)의 시나리오 번호를 붙인다. 시각은 `Clock`(`common/TimeConfig`)을 바꿔 끼워 23:59, 0:01(한국 시간)을 만든다. 화면에는 테스트 도구가 없어서(`frontend/package.json`) 각 단계 끝의 `Checkpoint`에서 손으로(또는 Playwright로) 확인한다.

**Organization**: 사용자 이야기(US)별로 묶었다. US6(통계)은 **읽기(Phase 9)** 와 **숫자 세기(Phase 11, 결정 대기)** 로 나눴다. 읽기는 정해진 것만으로 만들 수 있고, 세기는 `D-2 ~ D-5`에 기댄다. PR은 이야기 하나에 하나 (CLAUDE.md `커밋과 올리기`). 코드는 `home-blog/blog`, 이 목록은 `home-blog/docs`에 있다.

## Format: `[ID] [P?] [Story] 설명`

- **[P]**: 동시에 해도 되는 작업 (다른 파일이고, 끝나지 않은 작업에 기대지 않음)
- **[Story]**: 어느 사용자 이야기의 작업인지 (US1 ~ US8)

## 경로 약속

> `002`, `003`과 같다. 코드는 `home-blog/blog` 저장소에 있다.

| 줄임 | 실제 경로 |
|---|---|
| `BE/` | `backend/src/main/java/com/myblog/` (서버, Spring Boot) |
| `BE-RES/` | `backend/src/main/resources/` |
| `BE-TEST/` | `backend/src/test/java/com/myblog/` (서버 테스트) |
| `FE/` | `frontend/src/` (화면, React + Vite) |

## 쉬운 설명: 이미 있는 것과 새로 만드는 것

이 기능은 **다른 기능이 만든 글·분류·댓글을 한 화면에 모아 관리**하고, **숫자(조회수·방문자)를 새로 센다.** 그래서 먼저 있어야 하는 것이 많다. 아래 "만들 예정"은 `003`·`005`의 작업 목록·계획에 적힌 이름이고, 그 기능이 merge된 뒤 실제 이름을 확인한다.

| 이미 있는 것 / 만들 예정인 것 | 이 기능에서 |
|---|---|
| **있음** (`V2__auth_tables.sql`): `blog.comments_read_at`(E-6), `category.sort_order`, `category.color_index`(E-3), `category.visibility` | 표는 그대로 쓴다. **새 표는 `blog_daily_stat`, `post_daily_stat`(E-5) 둘뿐** (T003) |
| **있음** (`002`): `BlogClosingEvent`(`com.myblog.blog`, 주석에 "006 stats: 이 블로그의 일별 통계"라고 적혀 있다), `BlogWithdrawalCleaner`가 블로그를 지우기 **전에** 낸다 | `stats`가 듣고 **그 블로그의 일별 통계를 스스로** 지운다 (T010) |
| **있음** (`001`·`002`): `ErrorCode`, `ApiException`, `GlobalExceptionHandler`, `CurrentMemberService`, `AccountProperties`(`@ConfigurationProperties` record), `TimeConfig`(`Clock`, UTC), Redis 도구(`StringRedisTemplate`, `EmailVerificationStore`의 Lua 스크립트 방식) | 같은 모양으로 `ManageProperties`, `StatsProperties`(T005), 한국 날짜 계산(T006), Redis 표시(T053) |
| **있음** (`001`·`002` 화면): `createBrowserRouter`(`FE/App.tsx`), `RequireLogin`, `api/client.ts`의 `ApiError`, `SiteHeader`의 `MEMBER_ONLY_PREFIXES`(`/manage`가 이미 있음), `index.css`의 원고지 토큰 | 관리 화면 틀 `ManageLayout`(T012)을 `/manage` 아래에 둔다 |
| **만들 예정** (`003` tasks): `post` 표(`V3`), `LoggedInMember`(T007), `BlogDirectory`·`CategoryPostCounter`(T009), `PostVisibility`(T010), `PostDeletingEvent`(T032), `PostBlogClosingCleaner`(T036), 분류 관리 서버·화면(T044 ~ T046, `/api/me/blog/categories`), 블로그 설정 서버·화면(T049, T050, `PUT /api/me/blog`) | 다시 만들지 않는다. 분류 관리(US3)와 블로그 설정(US8)은 **`003`의 주소와 화면을 관리 화면 안에 넣고** 빠진 것(색 점, 위·아래 버튼, "n/200", "저장했습니다")만 더한다 |
| **만들 예정** (`005` tasks): `comment` 모듈(`005` T005, 의존 `post`·`user`·`common`)과 표(`UNIQUE(users_id, post_id)` 없음, `005` D-7), `PostCommentCounter`(`post` 맨 위, `long count(postId)`, `005` T010·T020), `PostLookup`(`005` T009), `MemberNames`(`user` 맨 위, 탈퇴한 작성자는 번호·닉네임 없이, `005` T008·D-9), 댓글 삭제 `DELETE /api/comments/{commentId}`(작성자 또는 그 글의 블로그 주인, 아니면 `403 COMMENT_DELETE_FORBIDDEN`, `005` T024), `PostDeletingEvent`를 듣는 댓글 정리(`005` T025. `BlogClosingEvent`는 따로 듣지 않음, `005` T026) | 댓글 관리(US4)는 `comment` 모듈 **안에** 목록 주소를 더하고, 삭제는 `005`의 것을 부른다. 글 관리의 댓글 수는 `PostCommentCounter`를 넓혀 쓴다 |

**모듈 방향** (Spring Modulith, `ModularityTest`): `user ← blog ← post ← comment ← stats`. `stats`는 **맨 위**라서 다른 모듈의 **맨 위 패키지**(입구 틀, 이벤트)만 부를 수 있고, **다른 모듈의 표에 직접 쓰지 않는다** (읽기도 입구로). 그래서:
- 아래 모듈이 위 모듈에 물어야 하는 것은 **아래 모듈이 틀(인터페이스)을 두고 위 모듈이 채운다** (`003`의 `CategoryPostCounter`와 같은 방식). 예: 글 관리의 댓글 수는 `post`가 틀 `PostCommentCounter`를 두고 `comment`가 채운다 (T008, T019).
- 글 상세가 "읽혔다"는 것은 `post`가 **이벤트**(`PostViewedEvent`)로 알리고 `stats`가 듣는다. `post.views +1`은 `stats`가 `post`의 입구(`PostViewCounter`)를 불러 **`post`가 자기 표에** 쓴다 (T050, T054).
- 관리 화면 머리 정보(블로그 이름 + 새 댓글 수)는 `blog`와 `comment`를 함께 읽어야 하므로 **`stats` 모듈**에 둔다 (T014, T035). `stats`는 "블로그 관리 화면을 모아 보여 주는 곳"도 맡는다.
- 모든 정리 리스너는 `@EventListener`로 **같은 트랜잭션 안에서** 돈다 (`002` US3와 같음). 숫자 세기만 **글 읽기를 망치지 않도록** 따로 트랜잭션으로 돌고 실패를 삼킨다 (T054, T055).

---

## Phase 1: Setup (공통 준비)

**Purpose**: 새 프로젝트 준비는 없다. 먼저 merge되어 있어야 하는 것을 확인하고, 결정을 요청해 둔다

- [ ] T001 시작 전 확인 — **이것들이 `main`에 merge되어 있고 CI가 통과해야** 시작한다: ① `002` US3(`BlogClosingEvent`, PR #19) ② `003` Phase 2와 US1 ~ US4, US6, US7(`V3` `post` 표, `LoggedInMember`, `BlogDirectory`, `CategoryPostCounter`, `PostVisibility`, `PostDeletingEvent`, `PostBlogClosingCleaner`, 분류 관리·블로그 설정 서버와 화면, `FE/App.tsx`의 `/manage/blog`·`/manage/categories`) ③ `005` Phase 2와 US1·US2(`005` T005 ~ T012, T018 ~ T026: `comment` 표와 모듈, `PostLookup`, `PostCommentCounter`, `MemberNames`, 댓글 삭제, `PostDeletingEvent`로 댓글 정리) ④ `004` US1(`004` T015 `PostListService`) — T056에서 D-5가 B일 때만. 이 목록의 `003`·`005` 이름은 두 기능의 tasks.md(2026-10-08 초안) 기준이고, 구현에서 이름이 바뀌었으면 그것을 따른다. US마다 브랜치를 만든다 (`feat/006-us1-manage`, `feat/006-us2-posts`, … 가안)
- [ ] T002 결정 요청을 미리 해 둔다 (작업을 막지 않음): 사용자에게 research `D-2`, `D-3`, `D-4`, `D-5`, `D-10`과 분류 **색 목록**(D-9의 남은 일)을 고르도록 선택지·추천을 보여 준다. 팀에는 BM-01-3, BM-05-7, BM-06-5, BM-06-8과 제안값 SC-011(1분)을 묻는다(`docs/2-요구사항/상세/06-블로그관리-통계.md` 초안 공유). 답이 오면 research `D-n`과 이 목록의 `결정 반영` 표를 고친다. **Phase 11은 D-2 ~ D-5, Phase 12는 D-10, T025는 색 목록이 정해진 뒤에** 한다

---

## Phase 2: Foundational (모든 이야기의 바탕)

**Purpose**: US1 ~ US8이 함께 기대는 표, 모듈, 설정, 날짜 계산, 모듈 사이 입구, 정리 리스너, 관리 화면 틀

**⚠️ CRITICAL**: 이 단계가 끝나기 전에는 사용자 이야기 작업을 시작하지 않는다

- [ ] T003 [P] Flyway `BE-RES/db/migration/V{다음 번호}__blog_stats.sql`(`003`의 `V3`, `005`의 파일 다음 번호, 가안): ① `blog_daily_stat` — 팀 ERD 칸(`blog_stat_id` 자동 증가 기본키, `blog_id` → `blog` 외래 키, `stat_date` DATE NOT NULL, `views`·`visitors` INTEGER NOT NULL **DEFAULT 0**) + **`UNIQUE (blog_id, stat_date)`를 두 칸에** (data-model 6, research R-3, `ERD-변경-요청.md` 요약표) ② `post_daily_stat` — E-5 그대로(`post_id` → `post` 외래 키, `stat_date`, `views` DEFAULT 0, `UNIQUE (post_id, stat_date)`). **연쇄 삭제는 걸지 않는다** (정리는 T010의 리스너가 한다). 맨 위 주석에 E 번호를 적는다 (`V2`처럼) (FR-006 ~ FR-008, FR-032, FR-036)
- [ ] T004 `stats` 모듈 뼈대 — T003 다음: `BE/stats/package-info.java`(`@ApplicationModule(displayName = "통계·블로그 관리", allowedDependencies = {"comment", "post", "blog", "user", "common"})`), `BE/stats/domain/BlogDailyStat.java`, `BE/stats/domain/PostDailyStat.java`, `BE/stats/repository/BlogDailyStatRepository.java`, `PostDailyStatRepository.java`(읽기 쿼리: 날짜 범위의 줄, 블로그의 합, 글 번호 목록의 최근 N일 합 — 값은 파라미터로만). `ddl-auto: validate`로 T003과 맞는지 확인한다
- [ ] T005 [P] 설정값 묶음 (plan `설정값 목록`, 헌법 VI): `BE/common/config/ManageProperties.java`(`manage.page-size` 10, `manage.comment.preview-length` 50, `manage.dashboard.popular-days` 7, `popular-size` 5, `recent-size` 5, `chart-days` 30, `manage.stats.period-options` [7, 30], `period-default` 30 — `post`·`comment`·`stats`가 함께 쓰므로 열린 모듈 `common`에 둔다), `BE/stats/config/StatsProperties.java`(`stats.view.dedupe-window` 30m, `stats.day-zone` `Asia/Seoul`)를 `AccountProperties`처럼 record로 만들고 `BE-RES/application.yml`에 더한다. `period-default`가 `period-options` 안에 없으면 서버가 켜지지 않게 한다. **방문자 쿠키 유지 기간(D-2)과 색 개수(색 목록)는 넣지 않는다** (plan: 기본값 표에 먼저)
- [ ] T006 [P] **한국 날짜는 한곳에서** `BE/stats/time/ServiceDay.java`(가안): `today()`, `dateOf(Instant)`, `startOf(LocalDate)`(그 한국 날짜 0시의 순간), `lastDays(n)`(오늘 포함 n일). `Clock`(UTC)과 `stats.day-zone`으로만 계산하고 서버·DB의 시간대 설정에 기대지 않는다 (research B-4, R-1, FR-036, SC-007)
- [ ] T007 [P] `BE/common/error/GlobalExceptionHandler.java`: 쿼리 값이 정해진 값이 아닐 때(`visibility=abc`, `page=-1`, `days=10`, 날짜 형식이 틀린 `newSince`) 지금은 `Exception` 처리로 **500**이 된다 → `MethodArgumentTypeMismatchException`, `MissingServletRequestParameterException`, `HandlerMethodValidationException`을 **`400 VALIDATION_FAILED`** 로 답하게 더한다(내부 정보 없이). `004` T007(같은 문제, `?page=abc`가 `500`)이 먼저 더했으면 확인만 한다. 새 `ErrorCode`는 만들지 않고 `VALIDATION_FAILED`, `NOT_FOUND`, `003`의 `CATEGORY_NOT_FOUND`·`POST_NOT_FOUND`, `005`의 댓글 오류를 쓴다 (contracts 공통 오류, NF-06, NF-07)
- [ ] T008 [P] `post` 맨 위의 입구 틀 (가안): ① `005` T010의 `BE/post/PostCommentCounter.java`(`post`가 묻고 **`comment`가 채우는** 틀)에 `countByPostIds(postIds) → Map<postId, count>`를 더한다 (글 관리의 댓글 수를 쪽마다 한 번에, FR-013) ② `BE/post/PostSummaryQuery.java` — `stats`·`comment`가 묻고 `post`가 채우는 입구: `postIdsOf(blogId)`, `recent(blogId, size)`(비공개 포함, 제목·작성 시각·`visibility`), `publicAmong(postIds)`(`PostVisibility`로 누구나 볼 수 있는 글만, 제목·작성 시각), `titlesOf(postIds)` (FR-008, FR-009, FR-024). 채우기는 T019, T046
- [ ] T009 [P] `comment` 맨 위의 입구 틀 (가안): `BE/comment/CommentStatsQuery.java` — `stats`가 묻고 `comment`가 채운다: `dailyCounts(postIds, fromInclusive, toExclusive, zone) → Map<LocalDate, count>` (한국 날짜로 묶은 일별 댓글 수, FR-032, FR-036). 채우기는 T041
- [ ] T010 **블로그를 닫거나 글을 지울 때 통계도 지운다** `BE/stats/service/StatsCleaner.java` — T004 다음: ① `@EventListener BlogClosingEvent` → 그 블로그의 `blog_daily_stat`을 모두 지운다 (`002`의 `BlogClosingEvent` 주석 "006 stats: 이 블로그의 일별 통계", `002` T035의 약속. `005` T026이 그 주석을 고칠 때 이 줄은 남긴다) ② `@EventListener PostDeletingEvent`(`003` T032) → 그 글의 `post_daily_stat`을 지운다 (D-6, data-model 4: `post_daily_stat`이 `post`를 가리키므로 먼저 지워야 글을 지울 수 있다). 탈퇴 때는 `003` T036이 글마다 `PostDeletingEvent`를 내므로 `post_daily_stat`도 함께 지워진다. 둘 다 같은 트랜잭션 안에서, 실패하면 탈퇴·글 삭제 전체가 취소된다
- [ ] T011 [P] `BE-TEST/stats/service/StatsCleanerTest.java` (클래스 전체 `@Transactional` 쓰지 않음, 끝에서 지움): 일별 통계·글별 통계 줄이 있는 회원이 탈퇴하면 성공하고 두 표에 그 블로그·글의 줄이 남지 않는다(`002` `WithdrawalFlowTest`처럼), `post_daily_stat` 줄이 있는 글을 지우면 `204`이고 줄이 사라진다, 다른 블로그의 줄은 그대로다
- [ ] T012 [P] 관리 화면 틀: `FE/manage/manageApi.ts`(contracts 1, 2, 3-1, 5, 6, 7의 요청 함수와 응답 타입. 분류·설정은 `003`의 `FE/blog/blogApi.ts`를 그대로 쓴다), `FE/manage/ManageLayout.tsx` + `manage.css`(원고지 토큰): 왼쪽 메뉴 `대시보드`·`글 관리`·`분류 관리`·`댓글 관리`·`통계`·`설정`, 오른쪽 `<Outlet />`, 휴대폰 화면에서는 메뉴가 위로(NF-03). `FE/App.tsx`: `<RequireLogin><ManageLayout/></RequireLogin>` 아래에 `/manage`(대시보드), `/manage/posts`, `/manage/categories`(`003`의 분류 관리 화면), `/manage/comments`, `/manage/stats`, `/manage/blog`(`003`의 블로그 설정 화면). `003` T012가 따로 만든 두 주소를 이 틀 안으로 옮긴다 (FR-001)

**Checkpoint**: `mvn verify`(CI)와 `ModularityTest`가 통과한다(`stats`가 맨 위, 고리 없음). 탈퇴·글 삭제가 통계 줄이 있어도 성공한다(T011). `/manage/...` 주소가 왼쪽 메뉴와 빈 내용으로 열린다

---

## Phase 3: User Story 1 - 블로그 주인만 쓰는 관리 화면에 들어가기 (Priority: P1) 🎯 MVP

**Goal**: 로그인한 블로그 주인이 사용자 메뉴에서 `블로그 관리`를 열어 왼쪽 메뉴와 블로그 이름·버튼을 본다. 로그인하지 않으면 로그인 창, 남의 관리 화면은 없다

**Independent Test**: quickstart S-1 (1 ~ 5)

### Tests for User Story 1

- [ ] T013 [P] [US1] `BE-TEST/stats/controller/ManageHeaderTest.java`: S-1의 4(로그인 없이 `GET /api/manage/blog` → `401 UNAUTHENTICATED`), S-1의 5(회원 B는 **B의 블로그** 이름·번호만 받는다, 주소에 블로그 번호를 넣을 곳이 없다), 응답의 `blogId`·`name`·`intro`·`blogPath`, 탈퇴한 회원의 세션 → `401` (FR-002, FR-003, FR-005, SC-001)

### Implementation for User Story 1

- [ ] T014 [US1] `BE/stats/controller/ManageHeaderController.java`: `GET /api/manage/blog` — 회원은 `LoggedInMember.requireIdOf`(`003` T007)로만, 블로그는 `BlogDirectory.myBlog(memberId)`(`003` T009)로 세션에서 찾는다. `blogPath`는 `003` 화면 주소 `/blog/{blogId}`. `newCommentCount`는 US5(T035)에서 더한다 (contracts 1, research B-1, FR-001 ~ FR-003, FR-005)
- [ ] T015 [US1] `FE/manage/ManageLayout.tsx` 머리: `GET /api/manage/blog`로 메뉴 위에 블로그 이름, "내 블로그 보기"(`blogPath`), "글쓰기"(`/write`). 머리 정보를 아래 화면(설정 등)과 나누는 `FE/manage/ManageBlogContext.tsx`(새로 읽기 `refresh()` 포함). `401`이면 `003`·`002`와 같이 로그인 창 (FR-001, FR-002, FR-005)
- [ ] T016 [US1] `FE/components/SiteHeader.tsx` 사용자 메뉴: `마이페이지`(`/mypage`)와 `블로그 관리`(`/manage`)를 **따로** 둔다. BM-01-3(`확인 필요`)은 원본 지금 내용대로 하고, 팀 답이 달라지면 이 메뉴만 고친다(T060) (FR-004)

**Checkpoint**: T013이 통과하고 S-1의 1 ~ 4를 화면으로 확인한다. S-1의 5 ~ 6(모든 관리 주소)은 T059에서 다시 본다 (PR 하나)

---

## Phase 4: User Story 2 - 글 관리: 비공개 글까지 내 글을 한곳에서 (Priority: P1)

**Goal**: 비공개 포함 내 글을 최신순 10개씩 보고, 공개 여부·분류로 거르고, 보기·수정·삭제한다

**Independent Test**: quickstart S-2

### Tests for User Story 2

- [ ] T017 [P] [US2] `BE-TEST/post/controller/ManagePostListTest.java`: S-2의 1(글 없음 → `items` 빈 배열, `hasAnyPost: false`), 2(공개 6·비공개 5 → 1쪽 10개, 2쪽 1개, 빠짐·겹침 없음, 같은 시각이면 번호 큰 것 먼저), 3(제목·분류·작성일·공개 여부·`views`·`commentCount`), 4(`all`/`public`/`private` → 11/6/5), 5(분류 + `private` 함께 → 둘 다 맞는 글만, 없으면 `items` 비고 `hasAnyPost: true`), 9(`visibility=abc`, `page=-1`, `page=0` → `400`, 내부 정보 없음), 남의 `categoryId` → `404`, 비공개 분류의 글도 주인에게는 나온다, 회원 B는 A의 글을 하나도 받지 않는다 (FR-012 ~ FR-014, FR-017, SC-002)

### Implementation for User Story 2

- [ ] T018 [US2] `BE/post/service/ManagePostQueryService.java` + `PostRepository` 쿼리: 내 블로그 분류 번호(`BlogDirectory.categoriesOf`)에 속한 글을 **비공개 포함**, `visibility`는 정해진 값(`all`/`public`/`private`)만, `categoryId`는 내 블로그 것이 아니면 `CATEGORY_NOT_FOUND`(404), (작성 시각, 글 번호) 최신순, 한 쪽 `manage.page-size`, `hasAnyPost`(거르기 전 내 글이 있는지), 분류 이름은 `BlogDirectory`에서, 댓글 수는 **그 쪽의 글 번호로 한 번에** `PostCommentCounter`에 묻는다 (research B-2, FR-012 ~ FR-014, FR-017, FR-037)
- [ ] T019 [P] [US2] `comment`가 T008에서 넓힌 틀을 채운다: `005` T020의 `BE/comment/service/CommentCounterAdapter.java`에 `countByPostIds` — 글 번호 목록의 댓글 수를 `GROUP BY` 쿼리 하나로 (FR-013)
- [ ] T020 [US2] `BE/post/controller/ManagePostController.java`: `GET /api/manage/posts?visibility&categoryId&page`(`page`는 1부터). 응답은 contracts 3-1 그대로 (FR-012 ~ FR-014, FR-017) — T018, T019 다음
- [ ] T021 [US2] `FE/manage/ManagePostsPage.tsx`(`/manage/posts`): 목록 한 줄(제목, 분류, 작성일, 공개 여부, 조회수, 댓글 수), 공개 여부·분류 고르기(주소의 쿼리에 남겨 새로고침해도 유지), 쪽 넘기기, `보기` → `/posts/:postId`, `수정` → `/write/:postId`, `삭제` → `<dialog>` "삭제하면 되돌릴 수 없습니다. 삭제할까요?", **취소하면 요청을 보내지 않음**, 확인하면 `003`의 `DELETE /api/posts/{postId}` 뒤 목록 다시 읽기, 목록 위 `글쓰기` → `/write`, 빈 상태 "아직 쓴 글이 없습니다"+글쓰기 버튼 / "글이 없습니다" (FR-015 ~ FR-017, SC-003)

**Checkpoint**: T017이 통과하고 S-2를 화면으로 확인한다. 삭제한 글의 댓글·글별 통계도 사라진다(`005`, T010) (PR 하나)

---

## Phase 5: User Story 3 - 분류 관리 (Priority: P2)

**Goal**: `003`의 분류 관리를 관리 화면 안에서 쓰고, 색 점과 위·아래 버튼, 항상 보이는 안내를 더한다

**Independent Test**: quickstart S-3, S-3a

> 추가·이름 변경·공개/비공개·순서·삭제 규칙과 주소는 **`003`이 만든 것**(`/api/me/blog/categories`, `003` T041 ~ T046)을 그대로 쓴다. contracts 4의 `/api/manage/categories`, `…/move`는 만들지 않는다 (contracts 머리말 "`003` plan이 다른 주소를 정하면 그쪽을 따른다", 문서는 T062에서 고친다).

### Tests for User Story 3

- [ ] T022 [P] [US3] `BE-TEST/blog/controller/CategoryColorTest.java`: 분류 목록·추가·고치기 응답에 `colorIndex`가 있다, 순서를 바꿔도 각 분류의 `colorIndex`가 그대로다(S-3의 4), 주인이 보는 글 개수는 비공개 글 포함(S-3a의 1), 회원 B의 요청은 A의 분류를 바꾸지 못한다(S-3a의 5, `404`). 나머지 S-3 줄은 `003` T041 `CategoryManageTest`가 덮으므로 다시 쓰지 않는다 (FR-018 ~ FR-020, FR-042, SC-004). 새 분류의 색 번호 순서 줄은 T025에서 더한다

### Implementation for User Story 3

- [ ] T023 [US3] `003`의 분류 응답(`GET /api/blogs/{blogId}/categories`, 추가·고치기 응답)에 `colorIndex`를 더한다 (`category.color_index`, contracts 4-1). `003` contracts 2·5·6은 T062에서 고친다 (FR-020)
- [ ] T024 [US3] `003`의 `FE/blog/CategoryManagePage.tsx`를 `/manage/categories`에 넣고 더한다: 분류마다 색 점(`colorIndex`), "비공개" 표시, `위`·`아래` 버튼(이웃한 두 분류를 바꾼 **전체 순서**를 `003`의 `PUT /api/me/blog/categories/order`로 보낸다, 맨 위의 `위`·맨 아래의 `아래`는 요청을 보내지 않음), 화면 아래에 늘 "글이 하나라도 있는 분류는 삭제할 수 없습니다. 글은 글 수정에서 다른 분류로 옮길 수 있습니다.", 빈 이름 "분류 이름을 입력해 주세요". **색 값은 T025 전까지 한 가지(`--grid-strong`)로** 칠한다 (FR-018 ~ FR-022, FR-042)
- [ ] T025 [US3] 새 분류의 색을 정해진 순서로 저절로 — **(색 목록) 결정에 따름 (추천: 시안의 색 목록을 `상세/06` `기본값` 표에 먼저 적는다)**: 정해지면 ① `상세/06` `기본값` 표와 plan `설정값 목록`에 색 개수를 적고 ② `003`의 `CategoryProperties`(`003` T005)에 `category.color-count`를 더하고 ③ `003`의 `Category.create`(`003` T043)와 `CategoryService` 추가 처리에서 `colorIndex`를 정한다(가안: research D-9의 예처럼 "그 블로그의 분류 수 % 색 개수") ④ 색 값 목록은 `FE/manage/categoryColors.ts`에 두고 `index.css` 토큰 옆에 맞춘다 ⑤ T022에 "새 분류 둘을 추가하면 색 번호가 차례로"를 더한다 (FR-020, research D-9)

**Checkpoint**: T022가 통과하고 S-3, S-3a를 화면으로 확인한다 (색 순서 줄은 T025 뒤) (PR 하나)

---

## Phase 6: User Story 4 - 댓글 관리 (Priority: P2)

**Goal**: 내 블로그 글에 달린 모든 댓글을 최신순 10개씩 보고, 어느 글의 댓글인지 확인하고, 확인 뒤 지운다

**Independent Test**: quickstart S-4

### Tests for User Story 4

- [ ] T026 [P] [US4] `BE-TEST/comment/controller/ManageCommentListTest.java`: S-4의 1(빈 목록), 2(여러 글의 댓글 11개 → 새 댓글이 위, 10 + 1), 3(닉네임·작성 시각·앞 50자·글 번호와 제목, 60자 댓글은 50자, 이모지도 한 글자로 셈), 5(탈퇴한 작성자 → `withdrawn: true`, `nickname: null`), 6(주인이 남의 댓글을 `005`의 `DELETE /api/comments/{commentId}`로 지움 → `204`), 7(`<script>`가 든 댓글이 글자 그대로 온다), 회원 B는 A의 블로그 댓글을 받지 않는다, `page=0` → `400` (FR-023 ~ FR-026, SC-003)

### Implementation for User Story 4

- [ ] T027 [US4] `BE/comment/service/ManageCommentQueryService.java`: 내 블로그 글 번호(`PostSummaryQuery.postIdsOf`)의 댓글을 (작성 시각, 댓글 번호) 최신순, 한 쪽 `manage.page-size`. `preview`는 **서버가 코드 포인트 기준 앞 `manage.comment.preview-length`자**로 자른다. 작성자는 `005` T008의 `MemberNames`로 한 번에(탈퇴면 `withdrawn: true`, 닉네임 없음, `005` D-9), 글 제목은 `PostSummaryQuery.titlesOf`. 내 블로그는 `LoggedInMember` + `BlogDirectory.myBlog`로 찾으므로 **`comment`의 `allowedDependencies`에 `blog`를 더한다**(`005` T005는 `post`, `user`, `common`. `comment → blog`는 고리가 아니다) (research B-9, FR-023, FR-024)
- [ ] T028 [US4] `BE/comment/controller/ManageCommentController.java`: `GET /api/manage/comments?page` — 응답은 contracts 5-2 모양(`isNew`는 US5의 T034에서) (FR-023, FR-024, FR-026)
- [ ] T029 [US4] `FE/manage/ManageCommentsPage.tsx`(`/manage/comments`): 닉네임(탈퇴했으면 "탈퇴한 사용자"), 작성 시각(한국 시간으로 보여 줌), 앞 50자(글자 그대로), 글 제목 → 그 글의 댓글 위치(`/posts/:postId`의 댓글 자리, 모양은 `005` 화면에 맞춘다), `삭제` → `<dialog>` "댓글을 삭제할까요?", 취소하면 요청 없음, 빈 목록 "아직 달린 댓글이 없습니다", 쪽 넘기기 (FR-023 ~ FR-026, SC-003)

**Checkpoint**: T026이 통과하고 S-4를 화면으로 확인한다 (PR 하나)

---

## Phase 7: User Story 5 - 새 댓글 표시 (Priority: P2)

**Goal**: 남이 단 새 댓글 수가 메뉴 옆·사용자 메뉴·대시보드에 **같은 숫자**로 보이고, 댓글 관리를 열면 읽음이 되어 사라진다

**Independent Test**: quickstart S-5

### Tests for User Story 5

- [ ] T030 [P] [US5] `BE-TEST/comment/controller/NewCommentCountTest.java`: S-5의 1 ~ 6 — 한 번도 열지 않았으면(`comments_read_at` 비어 있음) 남이 단 모든 댓글, 주인이 단 댓글은 세지 않음, `POST /api/manage/comments/read` → `previousReadAt`·`readAt`, 그 뒤 `new-count`·`GET /api/manage/blog`의 숫자가 0, 그 뒤 달린 하나만 1, `newSince`로 쪽을 넘겨도 `isNew`가 같은 댓글에만, CSRF 토큰 없이 읽음 처리 → 거절 (FR-027 ~ FR-029, SC-008)
- [ ] T031 [P] [US5] `BE-TEST/comment/service/CommentsReadRaceTest.java` (클래스 전체 `@Transactional` 쓰지 않음): S-5의 7 — 읽음 처리와 남의 댓글 쓰기를 동시에 여러 번 해도, 그 댓글은 이번 목록에서 `NEW`이거나 다음번 새 댓글 수에 남는다. **보지도 못하고 사라지는 경우 0건**. 읽음 처리 두 개가 동시에 와도 `comments_read_at`이 뒤로 가지 않는다 (research R-4)

### Implementation for User Story 5

- [ ] T032 [US5] `blog` 맨 위 입구 `BE/blog/CommentReadMarks.java`(가안): `readAt(blogId)`, `markRead(blogId) → ReadMark(previous, now)`. 채우기 `BE/blog/service/CommentReadMarksAdapter.java` — 블로그 줄을 잠그고(`@Lock(PESSIMISTIC_WRITE)`) `Blog.markCommentsRead(now)`(새 메서드, 이전 값을 돌려줌), `now`는 `Clock`. `blog.comments_read_at`은 `V2`에 이미 있다 (D-8, research B-6, FR-029)
- [ ] T033 [US5] **새 댓글 수 계산은 하나** `BE/comment/NewCommentCounter.java`(`comment` 맨 위 입구) + `BE/comment/service/NewCommentCounterService.java`: 내 블로그 글의 댓글 중 `created_at > readAt`(비어 있으면 모두) **그리고** `users_id ≠ 블로그 주인`인 것의 수. 메뉴 옆·사용자 메뉴·대시보드가 **모두 이것만** 부른다 (FR-027, FR-028, SC-008)
- [ ] T034 [US5] `ManageCommentController`에 `POST /api/manage/comments/read`(contracts 5-1), `GET /api/manage/comments/new-count`(contracts 6), 목록의 `newSince` → `isNew`(`newSince`보다 늦고 작성자가 주인이 아님. 보여 주기에만 쓰고 권한과 상관없다). "한 번도 연 적 없음"(`previousReadAt: null`)을 목록 요청에 어떻게 실을지는 구현 때 가안으로 정하고 contracts 5-2를 고친다(T062) (FR-027 ~ FR-029)
- [ ] T035 [US5] `ManageHeaderController`(T014)의 응답에 `newCommentCount`를 더한다 — `NewCommentCounter`를 부른다 (contracts 1, FR-028)
- [ ] T036 [US5] 화면: `FE/manage/NewCommentCountContext.tsx`(로그인했을 때 화면을 열면 `new-count`를 한 번 묻는다, 계속 다시 묻지 않음), `SiteHeader` 사용자 메뉴의 `블로그 관리` 옆과 `ManageLayout`의 `댓글 관리` 옆에 숫자. `ManageCommentsPage`는 열 때 **읽음 처리를 먼저** 보내고 받은 `previousReadAt`으로 목록을 읽어 `NEW`를 붙이고, 숫자를 0으로 바꾼다. 종 모양 목록·메일·푸시는 만들지 않는다 (BM-05-7 `확인 필요`, 원본대로) (FR-028 ~ FR-030)

**Checkpoint**: T030, T031이 통과하고 S-5를 화면으로 확인한다 (대시보드 숫자는 US7 뒤) (PR 하나)

---

## Phase 8: User Story 8 - 블로그 설정 (Priority: P3)

**Goal**: 관리 화면의 `설정`에서 블로그 이름·소개를 바꾸고, 바로 모든 화면에 보인다

**Independent Test**: quickstart S-10

> 규칙과 주소는 **`003`의 것**(`PUT /api/me/blog`, `003` T047 ~ T050)이다. contracts 8의 `PATCH /api/manage/blog`는 만들지 않는다(T062에서 고침). 다른 이야기와 기대는 것이 없어 US5 다음에 두었지만 언제 해도 된다.

- [ ] T037 [P] [US8] `003` T047 `BlogSettingsTest`가 S-10의 2 ~ 4(빈 이름, 31자 이름, 201자 소개를 화면 없이)를 덮는지 확인하고, 빠진 줄(200자 소개 통과, 저장 뒤 `GET /api/manage/blog`의 이름이 새 값)만 더한다 (FR-039, FR-040, SC-010)
- [ ] T038 [US8] `003`의 `FE/blog/BlogSettingsPage.tsx`를 `/manage/blog`에 넣고 더한다: 소개 아래 "n/200"(코드 포인트로 셈, 서버와 같게), 이름이 비면 칸 아래 "블로그 이름을 입력해 주세요", 저장하면 "저장했습니다", 저장 뒤 `ManageBlogContext.refresh()`로 관리 화면 위쪽 이름을 바로 바꾼다. "데모 데이터 초기화" 버튼은 없다 (FR-039 ~ FR-041, SC-010)

**Checkpoint**: S-10을 화면으로 확인한다 (이름이 보이는 다른 화면은 `004` 뒤에 다시) (PR 하나, US3와 묶어도 된다)

---

## Phase 9: User Story 6 - 통계 보기 (읽기) (Priority: P2)

**Goal**: 통계 화면에서 7일·30일의 일별 조회수·방문자·댓글 수를 본다. 이 단계는 **쌓인 숫자를 읽기만** 한다 (세기는 Phase 11)

**Independent Test**: quickstart S-8, S-7의 4(댓글 쪽). 숫자 줄은 테스트가 `blog_daily_stat`에 직접 넣는다

### Tests for User Story 6 (읽기)

- [ ] T039 [P] [US6] `BE-TEST/stats/controller/StatsQueryTest.java`: S-8의 1 ~ 2(`days` 없으면 30, `days=7`이면 7개 날짜), 4(기록이 없는 날도 날짜가 빠지지 않고 0), 5(`days=10`, `days=' OR 1=1 --` → `400`), S-7의 4(`Clock`을 한국 시간 오전 8시(= UTC 전날 23시)로 두고 단 댓글이 **오늘** 한국 날짜에 들어감), 응답에 유입 경로·시간대·기기 칸이 없음(S-8의 6, BM-06-8 `확인 필요`, 원본대로), 회원 B는 A의 숫자를 받지 않음 (FR-031, FR-032, FR-036, FR-038, SC-007)

### Implementation for User Story 6 (읽기)

- [ ] T040 [P] [US6] `comment`가 T009의 틀을 채운다: `BE/comment/service/CommentStatsQueryAdapter.java` — 기간의 처음·끝은 **순간(Instant)** 으로 비교하고, 날짜는 `created_at`을 받은 시간대로 바꿔 묶는다(시간대는 파라미터로, 글자를 이어 붙이지 않음) (research R-1, FR-032, FR-036)
- [ ] T041 [US6] `BE/stats/service/StatsQueryService.java`: `ServiceDay.lastDays(days)`의 **모든 날짜**마다 `blog_daily_stat`의 `views`·`visitors`(없으면 0)와 `CommentStatsQuery`의 댓글 수(없으면 0) (research B-5, FR-031, FR-032, FR-036) — T040 다음
- [ ] T042 [US6] `BE/stats/controller/StatsController.java`: `GET /api/manage/stats?days` — `manage.stats.period-options`에 있는 값만, 없으면 `period-default`. 응답은 contracts 7 그대로 (FR-031, FR-038)
- [ ] T043 [US6] `FE/manage/StatsPage.tsx`(`/manage/stats`): 기간 7일 / 30일(처음 30일), "조회수·방문자" 자리와 "댓글 수" 자리 두 개. **그래프는 D-10이 정해진 뒤(T058)** 그리고, 그 전에는 두 자리에 날짜별 숫자 표를 보여 준다 (FR-031, FR-032)

**Checkpoint**: T039가 통과하고 S-8의 1, 2, 4 ~ 6을 화면으로 확인한다 (그래프 줄 3은 T058 뒤) (PR 하나)

---

## Phase 10: User Story 7 - 대시보드 (Priority: P3)

**Goal**: 관리 화면을 열면 오늘·어제·누적 숫자, 최근 30일, 인기 글, 최근 글, 새 댓글을 한 화면에서 본다

**Independent Test**: quickstart S-9 (숫자는 테스트가 통계 표에 직접 넣는다. 실제로 쌓이는 것은 Phase 11 뒤)

### Tests for User Story 7

- [ ] T044 [P] [US7] `BE-TEST/stats/controller/DashboardTest.java`: S-9의 1(새 블로그 → 숫자 모두 0, 두 목록 빈 배열), 2(오늘·어제가 `GET /api/manage/stats`의 같은 날 숫자와 같다, 누적 = 모든 줄의 합(D-7), 지운 글이 있어도 누적은 줄지 않음), 3(`chart`가 오늘 포함 30개 날짜), 4(최근 7일 조회수가 가장 많은 글을 비공개로 바꾸면 인기 글에서 빠짐, 공개 글 5개까지, 같은 수면 최근에 쓴 글 먼저, 8일 전 조회수는 세지 않음), 5(최근 글 5개, 비공개 포함, `visibility`), 6(`newCommentCount`가 `new-count` 주소와 같다, SC-008) (FR-006 ~ FR-011, SC-009)

### Implementation for User Story 7

- [ ] T045 [P] [US7] `post`가 T008의 `PostSummaryQuery`를 채운다: `BE/post/service/PostSummaryQueryAdapter.java` — `publicAmong`은 `PostVisibility`의 "누구나 볼 수 있는 글"(글 공개 **그리고** 분류 공개) 조건을 그대로 쓴다(가안. 비공개 분류의 공개 글을 인기 글에서 뺄지는 명세에 없어 T062에서 확인을 요청한다) (FR-008, FR-009, SC-009)
- [ ] T046 [US7] `BE/stats/service/DashboardService.java`: 오늘·어제(`ServiceDay`)의 `blog_daily_stat` 줄(없으면 0), 누적 = 모든 줄의 합(D-7), `chart` = `manage.dashboard.chart-days`일, 인기 글 = 내 글 번호의 최근 `popular-days`일 `post_daily_stat` 합이 큰 순 → `publicAmong`으로 공개 글만 → 같으면 작성 시각 늦은 것 먼저 → `popular-size`개, 숫자는 그 7일 합(D-6), 최근 글 = `PostSummaryQuery.recent(recent-size)`, 새 댓글 수 = `NewCommentCounter` (data-model 6 `숫자 계산 규칙`, FR-006 ~ FR-011) — T045 다음
- [ ] T047 [US7] `BE/stats/controller/DashboardController.java`: `GET /api/manage/dashboard` — 응답은 contracts 2 모양 (FR-006 ~ FR-011)
- [ ] T048 [US7] `FE/manage/DashboardPage.tsx`(`/manage`): 오늘·어제·누적 조회수와 방문자, 새 댓글 수(0보다 크면 강조, "댓글 보기" → `/manage/comments`), 최근 30일 자리(**그래프는 T058 뒤**, 그 전에는 숫자 표)와 "통계 더 보기" → `/manage/stats`, 인기 글(제목·조회수), 최근 글(작성일, 비공개면 "비공개"), 빈 목록 "아직 쓴 글이 없습니다" (FR-006 ~ FR-011)

**Checkpoint**: T044가 통과하고 S-9의 1, 4 ~ 6을 화면으로 확인한다. 이 시점에는 숫자가 아직 쌓이지 않는다(0) (PR 하나)

---

## Phase 11: User Story 6 - 숫자 세기 (조회수·방문자) (Priority: P2) — **결정 대기**

**Goal**: 방문자가 글을 열면 같은 사람·같은 글 30분에 한 번 조회수를, 같은 사람·같은 블로그 하루에 한 번 방문자를 센다. 세기가 실패해도 글은 보인다

**Independent Test**: quickstart S-6, S-7, S-12

> ⚠ **T002의 `D-2`, `D-3`, `D-4`, `D-5`가 정해진 뒤에** 시작한다. 결정에 기대지 않는 T049 ~ T051, T053은 먼저 해 둘 수 있지만, 이 단계의 PR은 결정 뒤에 한 번에 올린다. 조회수와 방문자를 **따로** 세는 것은 BM-06-5(`확인 필요`)의 원본 지금 내용대로다.

### Tests for User Story 6 (세기)

- [ ] T049 [P] [US6] `BE-TEST/stats/service/ViewCountingTest.java` (클래스 전체 `@Transactional` 쓰지 않음, 테스트용 설정에서 `stats.view.dedupe-window`를 1분으로, 시각은 `Clock`): S-6의 1 ~ 3(회원 B가 열면 글 누적·오늘 블로그·오늘 글 조회수 +1, 1분 안에 다시 열면 그대로, 지나면 다시 +1), 5(같은 글을 동시에 10번 → 정확히 +1), 7(없는 글, 남의 비공개 글 → 그대로), 8(글마다 따로 셈), S-7의 1(같은 날 글 3개 → 방문자 1, 조회수 3, BM-06-5), 3(23:59와 다음 날 0:01 → 두 날짜에 방문자 각각 1). **결정 뒤에 기대값을 채우는 줄**: S-6의 4·S-7의 5(D-2), S-6의 6(D-4), S-7의 2(D-5) (FR-033 ~ FR-037, SC-005 ~ SC-007)
- [ ] T050 [P] [US6] `BE-TEST/stats/service/ViewCountingFailureTest.java`: S-12 — Redis에 연결할 수 없게 하면(`StringRedisTemplate`이 예외를 던지게) `GET /api/posts/{postId}`는 `200`이고 숫자는 그대로, 관리 화면 주소(대시보드, 통계, 글 관리)는 정상 (research B-3, NF-09)

### Implementation for User Story 6 (세기)

- [ ] T051 [P] [US6] `post` 맨 위: `BE/post/PostViewedEvent.java`(`record(postId, blogId, ownerId, viewerId)`, `viewerId`는 로그인 안 했으면 `null`) — `003`의 `PostReadService`(`003` T027)가 **글을 보여 줄 수 있을 때만** 낸다(없는 글·남의 비공개 글은 내지 않음). `BE/post/PostViewCounter.java`(입구) + `BE/post/service/PostViewCounterAdapter.java`: `increment(postId)` = `UPDATE post SET views = views + 1`(DB에서 바로, 읽고 쓰기 아님). 듣는 쪽이 없으면 아무 일도 없다 (contracts 9, research R-3, FR-033, FR-037)
- [ ] T052 [US6] 같은 사람 알아보기 `BE/stats/service/VisitorKeyResolver.java` — **D-2 결정에 따름 (추천: A. 로그인했으면 `m:{회원 번호}`, 아니면 무작위 방문자 쿠키 `g:{값}`)**: A면 쿠키가 없을 때 추측할 수 없는 무작위 값을 만들어 지금 응답에 내려 준다(`HttpOnly`, `SameSite=Lax`, prod는 `Secure`, IP 등 개인 정보 없음, 서버에 저장하지 않음). 쿠키 유지 기간은 **`상세/06` `기본값` 표에 먼저** 적은 뒤 `StatsProperties`에 더한다. 리스너가 요청 안에서 돌므로 지금 요청·응답은 `RequestContextHolder`로 얻는다 (data-model 8·9, research R-2, FR-033, FR-034)
- [ ] T053 [P] [US6] 중복 세기 표시 `BE/stats/service/ViewMarks.java` (D-1 결정됨, 헌법 1.1.0의 "조회수 중복 방지·오늘 방문 표시(가안)"): `stats:view:{postId}:{visitorKey}`를 "없을 때만 저장" + `stats.view.dedupe-window` 만료, `stats:visit:{blogId}:{한국 날짜}:{visitorKey}`를 "없을 때만 저장" + **다음 한국 자정 + 여유** 만료. 처음 만든 쪽만 `true`. `StringRedisTemplate`(`EmailVerificationStore`와 같은 도구), 키 이름은 가안 (data-model 8, research B-3, R-1)
- [ ] T054 [US6] 숫자를 쓰는 **입구 하나** `BE/stats/service/ViewRecorder.java` — **D-3 결정에 따름 (추천: A. 바로 DB)**: A면 따로 트랜잭션(`REQUIRES_NEW`)에서 조회 표시가 새로 생겼을 때 `PostViewCounter.increment` + `blog_daily_stat.views` + `post_daily_stat.views`를 "없으면 만들고 있으면 +1"(`INSERT … ON CONFLICT … DO UPDATE`, 값은 파라미터로)로, 방문 표시가 새로 생겼을 때 `blog_daily_stat.visitors` +1. 조회수와 방문자는 다른 표시·다른 칸이다(BM-06-5). B를 고르면 Redis에 모으고 `@Scheduled`로 옮기는 작업을 이 입구 뒤에 더한다 (research B-3, R-3, R-7, FR-033 ~ FR-035, FR-037)
- [ ] T055 [US6] `BE/stats/service/PostViewListener.java`: `@EventListener PostViewedEvent` → 본인 열람 처리 — **D-4 결정에 따름 (추천: A. `viewerId == ownerId`면 조회수·방문자 모두 세지 않는다)** → `VisitorKeyResolver` → `ViewMarks` → `ViewRecorder`. **모든 예외를 잡아 기록만 하고(개인 정보 없이) 삼킨다** — 글 상세는 그대로 `200` (research B-3, NF-09, FR-033)
- [ ] T056 [US6] 방문자를 세는 화면 — **D-5 결정에 따름 (추천: B. 블로그의 모든 방문자용 화면)**: B면 `BE/blog/BlogVisitedEvent.java`(`record(blogId, ownerId, viewerId)`, `blog` 맨 위)를 `003`의 `GET /api/blogs/{blogId}`(블로그 첫 화면)와 `004`의 `PostListService`(`004` T015, 블로그 글 목록·분류별 목록)가 내고, `stats`가 듣고 **방문자만** 센다(T055와 같은 규칙). `004` plan·작업 목록에도 이 줄을 더한다. A면 글 상세(T055)에서만 센다 (research D-5, FR-034)
- [ ] T057 [US6] quickstart S-6 ~ S-7을 화면으로(로그인한 브라우저 둘, 비회원 브라우저 하나) 확인하고, T049의 "결정 뒤에 채우는 줄"을 결정대로 채운다. quickstart의 "`D-n`가 정해지면 확정" 줄도 고친다

**Checkpoint**: T049, T050이 통과하고 S-6, S-7, S-12를 확인한다. 대시보드·통계·글 관리에 실제 숫자가 쌓인다 (PR 하나)

---

## Phase 12: 그래프 그리기 (US6, US7) — **결정 대기**

**Purpose**: 통계 화면과 대시보드의 숫자 표를 선 그래프로 바꾼다

> ⚠ **T002의 `D-10`이 정해진 뒤에** 한다. 서버 계약(contracts 2, 7)은 도구와 상관없이 같다.

- [ ] T058 [US6] [US7] 그래프 — **D-10 결정에 따름 (추천: A. 선 그래프·마우스 표시·휴대폰 화면을 지원하는 가벼운 그래프 도구 하나를 구현 때 고른다)**: 고른 도구(또는 B면 직접 그린 SVG)로 `FE/manage/charts/DailyLineChart.tsx`를 만든다 — 날짜별 선, 마우스를 올리면 그날 숫자, 휴대폰 화면 맞춤(NF-03), 색은 `index.css` 토큰. `StatsPage`(조회수·방문자 한 그래프, 댓글 수 다른 그래프)와 `DashboardPage`(최근 30일 조회수·방문자)의 숫자 표를 바꾼다. 도구 이름과 버전은 plan `Technical Context`의 `그래프` 줄과 기술스택 문서에 적는다 (FR-007, FR-032)

**Checkpoint**: S-8의 3, S-9의 3을 화면으로 확인한다 (PR 하나, Phase 11과 묶어도 된다)

---

## Phase 13: Polish & Cross-Cutting Concerns

- [ ] T059 [P] `BE-TEST/stats/controller/ManageAccessTest.java`: S-1의 5·6을 **모든 관리 주소**에 — 회원 B로 대시보드·글 목록·댓글 목록·통계를 부르면 B의 것만, A의 글·분류·댓글 번호로 거르기·분류 고치기·순서·삭제를 보내면 `404`, 댓글 삭제는 `005`가 정한 `403 COMMENT_DELETE_FORBIDDEN`이고 A의 데이터가 그대로. S-11의 1(CSRF 토큰 없이 읽음 처리·설정 저장·분류 추가 → 거절), 2(거르기·기간 값에 `' OR 1=1 --` → `400`, DB 그대로), 4(잘못된 JSON·없는 주소 → 예외 이름·쿼리·경로 없음) (FR-002, FR-003, SC-001, NF-02, NF-06, NF-07, NF-11)
- [ ] T060 [P] 팀 확인 결과 반영 — **BM-01-3, BM-05-7, BM-06-5, BM-06-8의 팀 답에 따름**: 답이 원본과 같으면 `상세/06`의 `확인 필요` 표시만 지운다. 다르면 `상세/06` → spec(FR-004, FR-030, FR-035, FR-038) → plan을 **먼저** 고치고, 영향받는 작업(T016 사용자 메뉴, T036 알림, T049·T054 따로 세기, T039·T042 남기지 않는 정보)을 새 작업으로 더한다. 알림 목록·메일·유입 경로처럼 새 기능이면 "추후 확장 후보"에 먼저 적는다 (헌법 V)
- [ ] T061 [P] 결정·구현에 맞게 문서를 같이 고친다 (헌법 `작업 흐름`): plan `Technical Context`(버전 `미정` → Spring Boot 4.1.1 등, 짧게 기억하는 값 줄 "`D-1` 결정"), plan 연결표의 FR-006·FR-008·FR-018·FR-020·FR-027 줄("팀 ERD 요청"·"`D-n` 결정" → E-3, E-5, E-6 내 확장과 결정 내용), research A표의 Redis 줄("이메일 인증번호 저장에만" → 헌법 1.1.0), research B-6의 "저장 위치 `미정`(`D-8`)", data-model 한눈에 보기·8번의 "정하지 않음 (Redis 가안, 헌법과 충돌)"·"`D-1`이 정해지기 전에는 만들지 않음", data-model 6의 누적 "정하지 않음 (`D-7`)", data-model 2·5의 "⚠ 없음"(→ `V2`에 있음), data-model 11(→ `ERD-변경-요청.md`의 T·E 기준), quickstart 준비의 "ERD 변경"·"댓글 주의"(`005` D-7로 `UNIQUE` 없음), 이벤트·입구·설정 이름이 가안에서 바뀌었으면 그것도. `상세/06` `구현 방식` 표(아직 "작성 예정")를 채우고 기술스택 2.4·3장 F와 맞춘다
- [ ] T062 contracts를 실제 주소·모양에 맞춘다: 4(분류)는 `003`의 `/api/me/blog/categories`와 `PUT …/order`(위·아래는 화면이 전체 순서를 보냄, T024), 8(설정)은 `003`의 `PUT /api/me/blog`, 4-1에 `colorIndex`(그리고 `003` contracts 2·5·6에도), 2의 `total`(D-7 결정)·`popularPosts.views`(D-6: 최근 7일 합), 5-2의 "한 번도 연 적 없음"을 싣는 방법(T034), 5-3의 오류(`005` contracts 3은 남의 댓글에 `403 COMMENT_DELETE_FORBIDDEN`, 이 문서는 `404`), 공통 오류 문구("※ 잘못된 요청입니다" ↔ 지금 `VALIDATION_FAILED`의 "입력값을 다시 확인해 주세요"). 인기 글에 비공개 분류의 공개 글을 넣을지(T045)는 사용자 확인을 요청한다
- [ ] T063 성능 (NF-09): 글 1,000개·365일 통계·댓글 수천 개가 있는 블로그로 글 상세(세기 포함, Phase 11 뒤)가 2초 안인지, 대시보드·통계·글 관리·댓글 관리가 느리지 않은지 본다. 느리면 인덱스(`post (category_id, created_at, post_id)`, `005`의 `comment (post_id, created_at)`)와 T018·T027의 쿼리 횟수부터 확인하고, 세기가 느리면 D-3의 B를 다시 검토한다 (research R-7)
- [ ] T064 SC-011(제안값 1분, 팀 확인 필요): **사람이** 로그인 상태에서 `블로그 관리`를 열어 대시보드에서 어제 방문자 수와 새 댓글 수를 찾을 때까지 시간을 잰다 (S-9의 7)
- [ ] T065 [P] quickstart S-11의 3(분류·블로그 이름의 `<script>`가 글자 그대로), 5(방문자 쿠키 속성과 값 — **D-2 결정에 따름**, 추천 A일 때 `HttpOnly`·`SameSite`, 무작위 값)를 화면에서 확인한다 (NF-07)
- [ ] T066 quickstart 전체를 실행하고 끝의 `구현 뒤에 채울 것`을 채운다. `tasks.md` 체크박스와 `CLAUDE.md`의 `6. 지금 상태`(006 줄, 남은 결정)를 고친다

---

## 요구사항 → 작업 연결표

| FR | 작업 |
|---|---|
| FR-001 | T012, T014, T015 |
| FR-002, 003 | T013, T014, T015, T059 |
| FR-004 | T016, T060 (BM-01-3) |
| FR-005 | T013, T014, T015 |
| FR-006 | T044, T046, T047, T048 |
| FR-007 | T044, T046, T048, T058 (D-10) |
| FR-008 | T003, T008, T044, T045, T046 |
| FR-009 | T008, T044, T045, T046, T048 |
| FR-010 | T044, T048 |
| FR-011 | T044, T048 |
| FR-012 | T017, T018, T020, T021 |
| FR-013 | T008, T017, T018, T019 |
| FR-014 | T007, T017, T018, T020, T021 |
| FR-015 | T021 (삭제는 `003`) |
| FR-016 | T021 |
| FR-017 | T017, T018, T021 |
| FR-018 | T022, T024 |
| FR-019 | T022, T024 (규칙은 `003`) |
| FR-020 | T022, T023, T024, T025 (색 목록) |
| FR-021 | T024 |
| FR-022 | T024 (서버 문구는 `003`) |
| FR-023 | T026, T027, T028, T029 |
| FR-024 | T008, T026, T027, T029 |
| FR-025 | T026, T029 (삭제는 `005`) |
| FR-026 | T026, T029 |
| FR-027 | T030, T033, T034 |
| FR-028 | T030, T033, T034, T035, T036 |
| FR-029 | T030, T031, T032, T034, T036 |
| FR-030 | T036, T060 (BM-05-7) |
| FR-031 | T005, T039, T041, T042, T043 |
| FR-032 | T009, T039, T040, T041, T043, T058 (D-10) |
| FR-033 | T049, T051, T052 (D-2), T053, T054 (D-3), T055 (D-4) |
| FR-034 | T049, T052 (D-2), T053, T054, T056 (D-5) |
| FR-035 | T049, T054, T060 (BM-06-5) |
| FR-036 | T006, T039, T040, T041, T049, T053 |
| FR-037 | T018, T049, T051, T054 |
| FR-038 | T039, T042, T060 (BM-06-8) |
| FR-039 | T037, T038 (규칙은 `003`) |
| FR-040 | T015, T037, T038 |
| FR-041 | T038 |
| FR-042 | T022, T024 (규칙은 `003`) |

| SC | 확인하는 작업 |
|---|---|
| SC-001 | T013, T059 |
| SC-002 | T017 |
| SC-003 | T021, T026, T029 |
| SC-004 | T022 (그리고 `003` T041) |
| SC-005 | T049 (비회원 줄은 D-2 뒤) |
| SC-006 | T049 |
| SC-007 | T006, T039, T049 |
| SC-008 | T030, T031, T044 |
| SC-009 | T044 |
| SC-010 | T037, T038 |
| SC-011 | T064 |

---

## Dependencies & Execution Order

### Phase Dependencies

- **Setup (Phase 1)**: T001의 merge 목록(`002` US3, `003`, `005` US1·US2)이 먼저. T002는 결정 요청만 하고 기다리지 않는다
- **Foundational (Phase 2)**: Setup 다음. **모든 사용자 이야기를 막는다**
- **US1 (P1)**: Foundational 다음
- **US2 (P1)**: US1 다음 (관리 화면 틀 안에 들어간다)
- **US3 (P2)**: US1 다음. T025는 **색 목록 결정 뒤**
- **US4 (P2)**: US1 다음
- **US5 (P2)**: US4 다음 (댓글 관리 목록에 `NEW`를 붙인다)
- **US8 (P3)**: US1 다음이면 된다
- **US6 읽기 (Phase 9)**: US1 다음
- **US7 (P3)**: US5(새 댓글 수)와 Phase 9(같은 날 숫자) 다음
- **US6 세기 (Phase 11)**: **D-2 ~ D-5 결정 뒤** + Phase 9. 명세는 "통계를 대시보드보다 먼저"라고 했지만, 대시보드는 **쌓인 숫자를 읽기만** 하므로 세기보다 먼저 만들어도 된다. 결정이 늦어지는 동안 정해진 것부터 끝낸다
- **그래프 (Phase 12)**: **D-10 결정 뒤** + Phase 9, 10
- **Polish**: 원하는 이야기가 끝난 뒤. T060은 팀 답이 온 뒤

### 결정을 기다리는 작업

| 결정 | 기다리는 작업 | 그동안 |
|---|---|---|
| D-2 | T052, T065, T049의 비회원 줄 | 회원 번호로만 세는 테스트를 먼저 쓸 수 있다 |
| D-3 | T054 | 표시(T053)와 이벤트(T051)는 먼저 만들 수 있다 |
| D-4 | T055, T049의 S-6 6 | — |
| D-5 | T056, T049의 S-7 2 | 글 상세에서 세는 흐름은 두 선택지 모두 같다 |
| D-10 | T058 | 통계·대시보드는 숫자 표로 보여 준다 (T043, T048) |
| (색 목록) | T025 | 색 점은 한 가지 색 (T024) |
| BM-01-3, BM-05-7, BM-06-5, BM-06-8 | T060 | 원본 지금 내용대로 만든다 (T016, T036, T054, T042) |

### Within Each User Story

- 테스트 → 입구·도메인 → 서비스 → 주소(컨트롤러) → 화면 순서
- 테스트는 먼저 써 두고 실패하는 것을 본 뒤 구현한다. push 전에 `mvn verify`와 화면 `lint`·`build`를 돌린다
- 한 이야기를 끝내고 Checkpoint를 통과한 뒤 PR을 merge하고 다음으로 간다

### Parallel Opportunities

- Phase 2의 T003, T005, T006, T007, T008, T009는 동시에. 화면의 T012도 동시에
- US2의 T017과 T019는 동시에 (다른 모듈)
- US3(분류), US4(댓글), US8(설정), Phase 9(통계 읽기)는 US1 뒤에 서로 기대지 않는다
- Phase 9의 T039와 T040은 동시에
- Phase 11의 T049, T050, T051, T053은 결정 전에도 동시에 시작할 수 있다

---

## Parallel Example: Phase 2

```text
먼저 동시에:
Task: "T003 V{n}__blog_stats.sql (blog_daily_stat, post_daily_stat)"
Task: "T005 ManageProperties, StatsProperties"
Task: "T006 ServiceDay (한국 날짜 한곳)"
Task: "T007 쿼리 값 오류를 400으로"
Task: "T008 post 입구 틀, T009 comment 입구 틀"
Task: "T012 ManageLayout, manageApi.ts, App.tsx 주소"
그다음: T004 → T010 → T011
```

---

## Implementation Strategy

### MVP First (US1 + US2)

1. Phase 1, 2를 끝낸다 (US1 PR에 같이 넣는다)
2. US1(관리 화면 들어가기) → US2(글 관리)
3. **멈추고 확인**: quickstart S-1(1 ~ 4), S-2
4. 보여 줄 수 있으면 보여 준다

### Incremental Delivery (정해진 것부터)

1. US3(분류 관리, 색 순서 빼고) → US4(댓글 관리) → US5(새 댓글) → 확인
2. US8(설정) → Phase 9(통계 읽기) → US7(대시보드) → 확인. 숫자는 아직 0
3. **D-2 ~ D-5가 정해지면** Phase 11(숫자 세기) → S-6, S-7, S-12
4. **D-10이 정해지면** Phase 12(그래프)
5. **색 목록이 정해지면** T025
6. 팀 답이 오면 T060

---

## Notes

- 작업 하나 또는 묶음 하나를 끝낼 때마다 커밋하고, 커밋 메시지에 작업 ID를 적는다 (예: `006 T018`). PR은 이야기 하나에 하나
- `가안`인 주소(`/api/manage/…`), 설정 이름(`manage.*`, `stats.*`), 입구·이벤트 이름(`PostCommentCounter`, `PostSummaryQuery`, `CommentStatsQuery`, `CommentReadMarks`, `NewCommentCounter`, `PostViewedEvent`, `PostViewCounter`, `BlogVisitedEvent`), Redis 키 이름을 바꾸면 T061·T062처럼 문서도 같이 고친다
- `003`·`005`의 코드(`PostReadService`, 분류 응답, `CategoryManagePage`, `BlogSettingsPage`)를 고치는 작업(T023, T024, T038, T051, T056)은 그 기능의 테스트가 계속 통과하는지 함께 본다
- 남은 결정: **D-2, D-3, D-4, D-5, D-10, 색 목록** (사용자), **BM-01-3, BM-05-7, BM-06-5, BM-06-8, SC-011** (팀). 확인할 위험: research R-1(한국 날짜, T006·T039·T049), R-2(비회원 같은 사람, D-2), R-3(동시 요청, T049·T054), R-4(읽음 처리 순간의 댓글, T031), R-5(지운 댓글은 지난 숫자에서도 빠짐, 받아들임), R-7(글 상세 속도, T063)
