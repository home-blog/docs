---

description: "004 글 탐색 (글 목록과 검색) 작업 목록"
---

# Tasks: 글 탐색 (글 목록과 검색)

**Input**: `/specs/004-explore/`의 설계 문서

**Prerequisites**: [plan.md](plan.md), [spec.md](spec.md), [research.md](research.md), [data-model.md](data-model.md), [contracts/explore-api.md](contracts/explore-api.md), [quickstart.md](quickstart.md)

**결정 반영 (2026-10-08)**: research의 `D-6 ~ D-8`은 정해졌고, **`D-1 ~ D-5`는 아직 `미정`이다** (헌법 II). 이 목록은 미정 항목을 정하지 않는다. 미정에 기대는 작업은 "D-n 결정에 따름 (추천: …)"으로 적고 **Phase 10(결정 반영)과 Phase 11(속도)에 모았다.** 그래서 Phase 1 ~ 9는 결정을 기다리지 않고 할 수 있다.

| ID | 무엇을 | 상태 | 추천 (research) | 이 목록에서 |
|---|---|---|---|---|
| D-1 | 50자를 넘는 검색어 | **미정** | A. 화면에서 50자까지만 입력되게 하고, 화면을 거치지 않은 요청은 서버가 `400`으로 거절 (새 문구 필요) | T040 (T027, T034가 자리만 남김) |
| D-2 | 잘못된 페이지 번호(1보다 작음, 숫자 아님, 검색 결과에서 너무 큼)와 남의·없는·비공개 분류 번호 | **미정** | 페이지는 A(1 또는 마지막 페이지로 바꿔 보여 줌), 분류는 가(`404` "존재하지 않는 분류입니다", 새 문구 필요) | T037, T038 (T007, T024, T029가 자리만 남김) |
| D-3 | 검색 결과의 본문 앞부분 길이 | **미정** | A. 글 목록과 같은 100자, 같은 설정값 | T039 (T029가 자리만 남김) |
| D-4 | 2초(NF-09)를 어떤 조건에서 재나, 검색에도 적용하나 | **미정** | A + 가. 조건(글 수, 사용자 수)을 정해 재고, 검색에도 같은 2초를 목표로. 숫자는 사용자가 정한다 | T045 |
| D-5 | 글이 많아져 검색이 느려질 때 | **미정** | A로 시작(단순 포함 검색), D-4 조건으로 재서 느리면 B(trigram 색인 `pg_trgm`) | T047 |
| D-6 | 글 표에 블로그 칸이 없음 | 결정됨 (2026-10-07) | A. 분류를 거쳐 찾는다. ERD 그대로 | T009, T015 |
| D-7 | 분류의 공개 여부 | 결정됨 (2026-10-07, A → B) | B. 비공개 분류의 글은 방문자 목록과 **모든 사람의 검색**에서 뺀다. 비공개 분류는 방문자의 분류 목록에도 없다 | T009, T015, T019, T028 |
| D-8 | 마크다운 본문의 미리보기·검색 | 결정됨 (2026-10-07) | B. 목록 미리보기는 **마크다운 기호를 걷어 낸 글자**로 만들고 100자도 걷어 낸 뒤 센다. 검색은 저장된 원문 그대로 | T008, T028 |

> **결정 요청 순서** (T002): 구현 전에 D-1, D-2, D-3을 묻는다(각각 작은 결정이라 Phase 10에서 한 번에 반영). D-4는 속도를 재기 전에, D-5는 D-4로 잰 뒤에 묻는다.

**Tests**: `001` ~ `003`처럼 **사용자 이야기마다 서버 테스트 작업**을 넣었다 (`@SpringBootTest` + MockMvc + `springSecurity()`, 실제 PostgreSQL). 테스트 이름에는 [quickstart.md](quickstart.md)의 시나리오 번호를 붙인다. 화면에는 테스트 도구가 없어서(`frontend/package.json`) 각 단계 끝의 `Checkpoint`에서 손으로(또는 Playwright로) 확인한다.

**Organization**: 사용자 이야기(US)별로 묶었다. PR도 이야기 하나에 하나 (CLAUDE.md `커밋과 올리기`). 코드는 `home-blog/blog`, 이 목록은 `home-blog/docs`에 있다.

## Format: `[ID] [P?] [Story] 설명`

- **[P]**: 동시에 해도 되는 작업 (다른 파일이고, 끝나지 않은 작업에 기대지 않음)
- **[Story]**: 어느 사용자 이야기의 작업인지 (US1 ~ US7)

## 경로 약속

> `002`, `003`과 같다. 코드는 `home-blog/blog` 저장소에 있다.

| 줄임 | 실제 경로 |
|---|---|
| `BE/` | `backend/src/main/java/com/myblog/` (서버, Spring Boot) |
| `BE-RES/` | `backend/src/main/resources/` |
| `BE-TEST/` | `backend/src/test/java/com/myblog/` (서버 테스트) |
| `FE/` | `frontend/src/` (화면, React + Vite) |

## 쉬운 설명: 이미 있는 것과 새로 만드는 것

이 기능은 **읽기만** 한다. 새 표를 만들지 않는다 (data-model: "아무것도 새로 저장하지 않는다"). 글·분류·블로그는 `003`이 만들고, 이 기능은 그것을 **세고, 10개씩 꺼내고, 찾는다.**

| 이미 있는 것 (코드) 또는 `003`이 만드는 것 (만드는 중) | 이 기능에서 |
|---|---|
| `blog`, `category` 표와 `idx_category_blog_id`, `ck_category_visibility` (`V2__auth_tables.sql`, 코드에 있음) | 그대로 읽는다. data-model 4의 `idx_category_blog` 제안은 **이미 있다** |
| `post` 표, `ck_post_visibility`(T-3), 인덱스 `post (category_id, created_at, post_id)` (`003` T003의 `V3`) | 그대로 읽는다. 목록 정렬은 이 인덱스를 쓴다. 더 필요한지는 속도를 잰 뒤(T046) |
| `post` 모듈, `Post`, `PostRepository` (`003` T004) | 목록 읽기 메서드를 더한다 (T009). 검색은 따로 읽는 곳을 둔다 (T028) |
| `PostVisibility` — "보이는 글" 조건 한곳 (`003` T010) | 목록은 이 조건을 **그대로** 쓴다. 검색은 주인이어도 비공개를 넣지 않는다 (`003` contracts 15) |
| `BlogDirectory`(`blog` 맨 위, `003` T009): `category`, `categoriesOf`, `publicCategoryIds` | 그대로 쓰고, "블로그가 있나, 주인은 누구인가"를 묻는 `blog(blogId)`를 더한다 (T006) |
| `LoggedInMember.idOf(Authentication)` (`user` 맨 위, `003` T007): 로그인 안 했거나 탈퇴했으면 빈 값 | 목록·검색은 이것으로 "누구인지"만 본다. 없으면 **오류 없이 방문자** (research B-6) |
| `SecurityConfig`: `GET /api/blogs/**`가 로그인 없이 열림 (`003` T008) | 목록 주소 `GET /api/blogs/{blogId}/posts`는 **이미 열려 있다.** 검색 주소 `GET /api/search/**`만 더 연다 (T010) |
| `ErrorCode`의 `BLOG_NOT_FOUND`, `CATEGORY_NOT_FOUND` (`003` T006), `ApiException`, `GlobalExceptionHandler` | 검색어 오류만 더한다 (T005). 지금 `GlobalExceptionHandler`에는 "숫자 자리에 글자가 온 요청" 처리가 없어서 `?page=abc`가 `500`이 된다 → T007, T037에서 막는다 |
| `AccountProperties`처럼 `@ConfigurationProperties` record로 숫자를 한곳에서 읽는 방식 | 같은 모양으로 `ExploreProperties` (T004) |
| 화면: `createBrowserRouter`(`FE/App.tsx`), `/blog/:blogId`·`/posts/:postId`·`/write` 주소와 `BlogHomePage`(`003` T012, T016: "글 목록 자리는 `004`가 채운다"), `blogApi.ts`의 분류 목록 요청(`003` T011), `api/client.ts`의 `ApiError`, `RequireLogin`, `index.css`의 원고지 토큰 | 블로그 화면에 글 목록·분류 고르기·페이지 번호를 넣고, 머리글에 검색창, 새 화면 `/search`를 만든다 |

**모듈 방향** (Spring Modulith, `ModularityTest`): `user ← blog ← post ← comment/community/image`. 목록과 검색은 **`post` 모듈 안에** 둔다 (가안, T003). 글 표가 `post` 모듈의 것이고, 코드의 모듈 목록(CLAUDE.md)에 `search`가 없기 때문이다. `post`는 `blog`·`user`의 **맨 위 패키지**(`BlogDirectory`, `LoggedInMember`)만 부른다. `blog`는 `post`를 부르지 않는다.

**`003`이 아직 만드는 중이다.** 아래 작업의 `003` 클래스·주소 이름은 `specs/003-blog-posts/tasks.md`의 **계획된 이름**이다. `003`이 이름을 바꾸면 이 목록도 같이 고친다 (T042).

---

## Phase 1: Setup (공통 준비)

**Purpose**: 새 프로젝트 준비는 없다 (`001`에서 끝남). `003`이 merge되었는지 확인하고, 결정을 요청한다

- [ ] T001 **시작 조건: `003`의 다음 작업이 `main`에 merge되어 있어야 시작한다** (`home-blog/blog`, CI 통과).
  - **Phase 2 시작 전 (반드시)**: `003` T003 ~ T012 (`003` Foundational 전체: `V3` `post` 표·인덱스, `post` 모듈·`Post`·`PostRepository`, `BLOG_NOT_FOUND`·`CATEGORY_NOT_FOUND`, `LoggedInMember`, `GET /api/blogs/**` 열기, `BlogDirectory`, `PostVisibility`, 화면 `blogApi.ts`·주소), `003` T014 ~ T015 (`BlogQueryService`, `GET /api/blogs/{blogId}`, `GET /api/blogs/{blogId}/categories` — 방문자에게 비공개 분류를 뺀 분류 목록)
  - **화면 작업(T017, T025) 전**: `003` T016 (`BlogHomePage`, 글 목록 자리)
  - **US3·US7 Checkpoint 전**: `003` T022 ~ T024 (글쓰기 `POST /api/posts`, `/write` 화면 — 빈 목록의 `글쓰기` 버튼이 갈 곳), `003` T027 ~ T029 (글 상세 `GET /api/posts/{postId}`, `/posts/:postId` — 목록 한 줄을 누르면 갈 곳)
  - **quickstart S-2a를 화면으로 확인하기 전**: `003` T043 ~ T046 (분류 공개 여부 바꾸기 `PATCH /api/me/blog/categories/{categoryId}`와 화면). 그 전에는 서버 테스트가 DB에서 직접 분류를 비공개로 만든다
  - **탈퇴한 회원의 글이 목록·검색에 없는지 확인하기 전**: `003` T036 (`BlogClosingEvent`를 듣고 글을 지움, research R-5)
  - `003` T040(`PostVisibility` 주석에 "`004` 목록·검색은 이 조건을 쓴다")이 끝났는지 본다. US마다 브랜치를 만든다 (`feat/004-us1-list`, `feat/004-us5-search`, … 가안)
- [ ] T002 **결정 요청** (헌법 II): research `D-1 ~ D-5`의 선택지·추천·이유를 사용자에게 보여 주고 고르게 한다. D-1·D-2·D-3은 구현 전에(Phase 10에서 반영), D-4는 T045 전에, D-5는 T045로 잰 뒤에. 정해지면 research의 해당 `D-` 절과 `E. 요약`, plan `정해야 할 것 요약`·연결표 줄, 이 목록 맨 위 표를 같이 고친다 (T041). 새 문구가 생기면(D-1의 A, D-2의 가·B) **`docs/2-요구사항/상세/04-탐색.md`의 `안내 문구` 표에 먼저 더한다** (contracts 서두 약속)
- [ ] T003 plan이 "구현 때 정한다"로 남긴 두 가지를 이 목록의 가안으로 적고 사용자에게 확인한다 (정하지 못하면 가안 그대로 진행, 결정 항목은 아니다): ① 모듈 — plan `Project Structure`의 `search` 모듈 대신 **`post` 모듈 안에** 목록과 검색을 둔다 (코드 모듈 목록에 `search` 없음, 글 표가 `post`의 것) ② 여러 단어 검색 쿼리 — plan `주요 도구`의 JPA Criteria·Querydsl 대신 **새 의존성 없이 고정된 쿼리 하나에 단어 묶음(배열)을 값으로 넘기는** 방식 (T028). 확인되면 plan 두 줄을 고친다

---

## Phase 2: Foundational (모든 이야기의 바탕)

**Purpose**: US1 ~ US7이 함께 기대는 설정, 오류, 블로그 질문, 페이지 계산, 미리보기, 읽기 입구

**⚠️ CRITICAL**: 이 단계가 끝나기 전에는 사용자 이야기 작업을 시작하지 않는다. T001의 "Phase 2 시작 전" 조건이 먼저다

- [ ] T004 [P] 설정값 묶음 `BE/post/config/ExploreProperties.java`(`@ConfigurationProperties(prefix = "explore")` record, `AccountProperties`처럼 잘못된 값이면 서버가 켜지지 않음: 페이지 글 수 1 이상, 미리보기 1 이상, 검색어 1 ≤ 최소 ≤ 최대)를 만들고 `BE-RES/application.yml`에 `explore.list.page-size: 10`, `explore.list.preview-length: 100`, `explore.search.keyword-min-length: 2`, `explore.search.keyword-max-length: 50`을 더한다 (plan `설정값 목록`, 상세/04 `기본값` 표, 헌법 VI). 검색 결과 한 페이지 글 수는 `explore.list.page-size`를 같이 쓴다 (plan 가안, "확인 요청" 그대로). 검색 미리보기 길이 설정은 **D-3 결정 뒤** T039에서 (FR-002, FR-005, FR-010, FR-014)
- [ ] T005 [P] `BE/common/error/ErrorCode.java`에 `SEARCH_KEYWORD_TOO_SHORT`(400, "검색어를 %d자 이상 입력해 주세요" — 숫자는 `explore.search.keyword-min-length`로 서비스가 채운다, `ACCOUNT_LOCKED`와 같은 방식. 기본값으로 "검색어를 2자 이상 입력해 주세요")를 더한다. `BLOG_NOT_FOUND`, `CATEGORY_NOT_FOUND`는 `003` T006의 것을 그대로 쓴다. `SEARCH_KEYWORD_TOO_LONG`(D-1), `INVALID_PAGE`(D-2의 B)는 **결정 뒤** T037, T040에서만 (contracts 2, FR-010)
- [ ] T006 [P] **블로그 질문 더하기**: `BE/blog/BlogDirectory.java`(`003` T009)에 `Optional<BlogInfo> blog(Long blogId)`(`BlogInfo(blogId, ownerId, name)`, `blog` 맨 위 패키지)를 더하고 `BE/blog/service/BlogDirectoryAdapter.java`에서 채운다. `003`이 이미 같은 질문을 만들었으면 그것을 쓴다. 목록의 "없는 블로그 → `404 BLOG_NOT_FOUND`"와 "주인인가"(`ownerId == 로그인한 회원`)에 쓴다 (contracts 1 동작 1·2, FR-006, FR-008, research B-6)
- [ ] T007 [P] **페이지 계산 한곳** `BE/post/service/PageNumbers.java` + `BE-TEST/post/service/PageNumbersTest.java`: 전체 수와 한 페이지 글 수로 마지막 페이지(`ceil(전체 / 10)`, 0개면 1)를 내고, 요청 번호가 마지막보다 크면 **마지막으로 바꾼다** (FR-004, 결정됨). 화면과 서버는 1부터, Spring Data는 0부터 센다(research B-2). 요청 번호는 글자로 받아 여기서 읽는다 — `?page=abc`가 `500`이 되지 않게 (지금 `GlobalExceptionHandler`에 처리 없음). **1보다 작음·숫자 아님·아주 큰 숫자**와 **검색 결과의 큰 번호**는 메서드 하나(`outOfRange…`)로 모으고 동작은 **D-2 결정에 따름 (추천: A, 1 또는 마지막 페이지)** → T037. 테스트는 결정된 줄(0개 → 1페이지, 25개 → 3페이지, 99 → 3)만 먼저 쓴다 (FR-002, FR-004, SC-005)
- [ ] T008 [P] **미리보기 한곳** `BE/post/service/PostPreview.java` + `BE-TEST/post/service/PostPreviewTest.java` (research B-3, D-8의 B): ① 마크다운 기호를 걷어 낸다 — 걷어 내는 규칙(제목 `#`, 굵게·기울임 `**`·`*`·`_`, 인용 `>`, 목록 기호, 코드 표시 `` ` ``, 링크 `[글자](주소)` → 글자, 이미지 `![설명](주소)` → 설명 또는 빼기)은 **가안**으로 research B-3에 적는다 ② 줄바꿈(`\r\n`, `\n`, `\r`)을 공백 하나로 ③ **코드 포인트**로 세어 `explore.list.preview-length`(100)까지, 넘으면 끝에 "…", 이하면 그대로. 본문 전체를 읽어 만든다(10개 × 최대 1만 자). 앞부분만 읽기는 T045에서 느릴 때만 검토 (research B-3). 테스트: 150자·줄바꿈 3개, 80자, 이모지, 기호만 있는 줄, 이미지 주소 (FR-005, quickstart S-1의 8·9)
- [ ] T009 **목록 읽기 입구** — `003` T004·T010 다음: `BE/post/repository/PostRepository.java`에 "분류 번호 묶음 안의 글" 세기와 10개 읽기를 더한다 — 주인이면 그 블로그의 모든 분류, 아니면 `BlogDirectory.publicCategoryIds`(공개 분류만) **그리고** `post.visibility = 'public'`. 이 조건 모양은 `PostVisibility`(`003` T010)에 "목록용"으로 두어 **한곳**을 지킨다 (D-6의 A: 분류를 거쳐 찾기, D-7의 B). 정렬 `created_at DESC, post_id DESC`. 세기와 읽기는 **한 읽기 전용 트랜잭션**에서 (research R-4). 값은 모두 파라미터로 (헌법 IV) (FR-001, FR-006, FR-009, data-model 3)
- [ ] T010 `BE/user/config/SecurityConfig.java`: `GET /api/search/**`를 `permitAll`에 더한다. 목록 `GET /api/blogs/{blogId}/posts`는 `003` T008의 `GET /api/blogs/**`로 이미 열려 있는지 확인만 한다 (FR-019, contracts 공통)
- [ ] T011 [P] 화면 요청 함수와 규칙: `FE/explore/exploreApi.ts`(`getBlogPosts(blogId, { page, categoryId })`, `searchPosts(q, page)`, 응답 타입은 contracts 1·2 그대로), `FE/explore/rules.ts`(`KEYWORD_MIN = 2`, `KEYWORD_MAX = 50`, 문구 "검색어를 2자 이상 입력해 주세요", "글이 없습니다", "검색 결과가 없습니다" — 서버 설정·상세/04 `안내 문구` 표와 같게)
- [ ] T012 [P] 화면 조각 `FE/explore/Pagination.tsx`(현재 페이지 강조 `aria-current="page"`, 첫 페이지에서 `이전`·마지막 페이지에서 `다음` 잠금, FR-003), `FE/explore/PostRow.tsx`(분류, 작성일, 제목, 미리보기, 검색이면 블로그 이름. 제목·미리보기는 **글자로만**(React 기본 이스케이프). 작성일은 `003` T029의 글 상세와 같은 방식(한국 시간, 가안)), `FE/explore/explore.css`(원고지 토큰) (FR-003, FR-005, FR-015)
- [ ] T013 화면 주소(가안) `FE/App.tsx`: `/search`(검색 결과, 로그인 없이)를 더한다. 블로그 글 목록은 `003`의 `/blog/:blogId`에 주소 뒤 값 `?category=`, `?page=`를 담는다 (새로고침·뒤로 가기에도 유지, contracts 1 `화면이 하는 일`) — T011 다음

**Checkpoint**: `mvn verify`(CI)와 `ModularityTest`가 통과한다(`post → blog → user` 방향만 있음). `PageNumbersTest`, `PostPreviewTest`가 통과한다. 로그인하지 않고 `GET /api/search/posts?q=ab`가 `401`이 아니다(아직 `404`여도 된다)

---

## Phase 3: User Story 1 - 글 목록을 최신순으로 보고 페이지 넘기기 (Priority: P1) 🎯 MVP

**Goal**: 블로그의 글이 최신순으로 10개씩, "N개의 글"과 페이지 번호와 함께 보인다. 너무 큰 페이지 번호는 마지막 페이지로

**Independent Test**: quickstart S-1

### Tests for User Story 1

- [ ] T014 [P] [US1] `BE-TEST/post/controller/PostListTest.java`: S-1의 1 ~ 10 — 공개 글 29개(25 + 같은 시각 2 + 긴 글 + 짧은 글)로 "29개의 글"(`totalCount`), 첫 페이지 정확히 10개, `totalPages` 3, 3페이지 9개, 모든 페이지에서 위 글이 같거나 최신, 같은 시각이면 `T2`(큰 `post_id`)가 위(작성 시각은 `JdbcTemplate`로 맞춘다), `page=99` → `200`이고 응답 `page`가 3, 긴 본문 미리보기 100자 + "…" 한 줄, 짧은 본문은 그대로, 한 번도 11개 이상 아님. 그리고 없는 블로그 → `404 BLOG_NOT_FOUND`, 글 0개 → `page` 1·`totalPages` 1 (FR-001 ~ FR-005, FR-009, SC-003 ~ SC-005)

### Implementation for User Story 1

- [ ] T015 [US1] `BE/post/service/PostListService.java` (읽기 전용 트랜잭션): ① `BlogDirectory.blog(blogId)` 없으면 `BLOG_NOT_FOUND` ② `LoggedInMember.idOf`로 보는 사람, `isOwner` ③ 보이는 분류 묶음(주인: `categoriesOf`, 아니면 `publicCategoryIds`) ④ T009로 세기 → `PageNumbers`(T007) → 10개 읽기 ⑤ 한 줄 = `postId`, `title`, `categoryId`, `categoryName`(분류 이름은 `categoriesOf`에서, 요청마다 읽음 — `003` contracts 15), `createdAt`(시간대 포함), `preview`(T008), `visibility`. 블로그·분류 이름을 기억해 두지 않는다 (contracts 1 동작 1 ~ 6, FR-001 ~ FR-006, FR-009) — T004 ~ T009 다음
- [ ] T016 [US1] `BE/post/controller/PostListController.java`: `GET /api/blogs/{blogId}/posts?page=&categoryId=` → `PostListResponse`(`blogId`, `isOwner`, `categoryId`, `totalCount`, `page`, `totalPages`, `pageSize`, `posts`) — `BE/post/controller/dto/ExploreResponses.java`. 회원은 `LoggedInMember`로만 정한다. 요청에 "비공개도 보여 줘" 같은 값을 받지 않는다 (contracts 1, research B-1)
- [ ] T017 [US1] 블로그 화면의 글 목록 `FE/explore/BlogPostList.tsx`를 `003`의 `FE/pages/BlogHomePage.tsx` 글 목록 자리에 넣는다: 위에 "{totalCount}개의 글", `PostRow` 10개(누르면 `/posts/:postId`), 아래 `Pagination`. 서버가 페이지를 바꿔 주면(99 → 3) 주소의 `?page=`도 응답 값으로 바꾼다(`replace`) (FR-001 ~ FR-005, FR-009) — `003` T016 다음

**Checkpoint**: T014가 통과하고 S-1을 화면으로 확인한다 (PR 하나. Phase 2를 같이 넣는다)

---

## Phase 4: User Story 2 - 공개 글은 누구나, 비공개 글은 주인만 (Priority: P1)

**Goal**: 방문자에게는 공개 분류의 공개 글만, 블로그 주인에게는 비공개 글과 비공개 분류의 글도 보인다. 화면이 보낸 값으로 이 규칙을 바꿀 수 없다

**Independent Test**: quickstart S-2, S-2a(1, 2, 6)

> 규칙 자체는 T009(`PostVisibility`의 목록용 조건)와 T015에서 이미 만들었다. 이 단계는 넓은 확인과 화면이다 (`003` US5와 같은 방식).

### Tests for User Story 2

- [ ] T018 [P] [US2] `BE-TEST/post/controller/PostListVisibilityTest.java`: S-2의 1 ~ 6 — 로그아웃·B는 "29개의 글"이고 비공개 없음, A(주인)는 "31개의 글"이고 `visibility: "private"`가 섞여 옴, B가 `visibility=private`·`includePrivate=true`를 붙여도 무시, 쿠키 없으면 방문자, **로그아웃한 뒤의 옛 쿠키·탈퇴한 회원의 세션 → `401`이 아니라 방문자**. S-2a의 1·2·6 — 비공개 분류 `일기`의 공개 글 2개는 방문자에게 없음("0개의 글", "글이 없습니다"), 주인은 보임. 방문자의 분류 목록(`003`의 `GET /api/blogs/{blogId}/categories`)에 `일기`가 없음 (FR-006, FR-019, SC-001, SC-002, SC-010)

### Implementation for User Story 2

- [ ] T019 [US2] `PostListService`(T015) 점검: 주인 판단은 **세션 회원 번호 == `BlogInfo.ownerId`** 하나뿐이고, 방문자 조건(공개 분류 **그리고** 공개 글)은 요청 값과 무관하게 항상 붙는지 본다. 방문자 응답의 `visibility`는 늘 `public`. `PostVisibility` 주석에 "`004` 목록이 이 조건을 쓴다(T009)"가 있는지 본다 (`003` T040) (FR-006, research B-1, B-6)
- [ ] T020 [US2] 화면: `PostRow`에서 `visibility`가 `private`이면 "비공개" 표시(`003`, CF-13-5. 주인에게만 온다) (FR-006)

**Checkpoint**: T018이 통과하고 S-2, S-2a(1, 2, 6)를 확인한다 (US1과 한 PR로 묶어도 된다)

---

## Phase 5: User Story 3 - 글이 없는 목록 안내 (Priority: P2)

**Goal**: 보일 글이 없으면 "글이 없습니다", 내 블로그면 "첫 글을 써 보세요"와 `글쓰기` 버튼도

**Independent Test**: quickstart S-3

- [ ] T021 [P] [US3] `BE-TEST/post/controller/PostListEmptyTest.java`: S-3의 1 ~ 5 — 글 없는 C를 방문자로 → `200`, `totalCount: 0`, `isOwner: false`, C 본인 → `isOwner: true`, 비공개 글만 있는 D를 방문자로 → `totalCount: 0`, D 본인 → 비공개 글이 보임. 오류가 아니다 (FR-008, US3)
- [ ] T022 [US3] 화면 `BlogPostList`: `totalCount`가 0이면 "글이 없습니다", `isOwner`면 "첫 글을 써 보세요"와 `글쓰기` 버튼(`/write`, `003` T024). 분류를 골랐는데 0개여도 같은 안내 (US4-4). "첫 글을 써 보세요"는 상세/04 `안내 문구` 표에 없다(contracts ※) → T041에서 표에 더할지 사용자 확인 (FR-008) — `003` T024 다음

**Checkpoint**: T021이 통과하고 S-3을 화면으로 확인한다 (PR 하나, US4와 묶어도 된다)

---

## Phase 6: User Story 4 - 분류로 좁혀 보기 (Priority: P2)

**Goal**: 분류를 고르면 그 분류의 글만, 페이지를 넘겨도 선택이 유지된다. 다른 블로그의 글은 절대 섞이지 않는다

**Independent Test**: quickstart S-4(1 ~ 6). S-4의 7, S-2a의 3·4는 D-2 결정 뒤(T038)

### Tests for User Story 4

- [ ] T023 [P] [US4] `BE-TEST/post/controller/PostListCategoryTest.java`: S-4의 1 ~ 6 — 고르지 않으면 모든 공개 글, `일상`만 "13개의 글", `일상`을 고른 채 2페이지도 `일상`만, **모든 페이지**에 다른 분류 0건(SC-006), 공개 글 없는 분류는 `totalCount: 0`. 그리고 B 블로그의 분류 번호·없는 번호를 넣으면 **A·B 어느 쪽 글도 섞여 오지 않는다**(응답 모양은 T038에서 확인) (FR-007, SC-006)

### Implementation for User Story 4

- [ ] T024 [US4] `PostListService`에 `categoryId`: **보이는 분류 묶음(T015의 ③) 안에 있을 때만** 조건에 더한다. 묶음 밖(다른 블로그의 분류, 없는 번호, 방문자가 보낸 비공개 분류 번호, 숫자 아닌 값)은 **메서드 하나**(`unknownCategory…`)로 모은다 — 비공개 분류도 없는 분류와 **똑같이** 다뤄 있다는 것을 드러내지 않는다(D-7). 그 메서드의 응답은 **D-2 결정에 따름 (추천: 가, `404 CATEGORY_NOT_FOUND` "존재하지 않는 분류입니다")** → T038. 응답의 `categoryId`에 적용한 값 (FR-006, FR-007, contracts 1 동작 3)
- [ ] T025 [US4] 화면 `FE/explore/CategoryFilter.tsx`(블로그 화면): 분류 고르기는 `003`의 분류 목록(`blogApi.ts`)을 쓴다 — 방문자에게는 비공개 분류가 빠져 온다. 고르면 주소 `?category=`에 담고 `page`는 1로, 페이지를 넘길 때 같은 값을 계속 보낸다, `전체`로 선택 풀기 (FR-007, contracts 1 `화면이 하는 일`) — `003` T016 다음

**Checkpoint**: T023이 통과하고 S-4(1 ~ 6)를 화면으로 확인한다 (PR 하나)

---

## Phase 7: User Story 5 - 키워드로 공개 글 찾기 (Priority: P2)

**Goal**: 여러 블로그의 공개 분류 공개 글을 제목·본문에서 찾는다. 모든 단어가 들어 있는 글만, 대소문자 무시, 최신순 10개씩. 비공개는 주인이어도 빠진다

**Independent Test**: quickstart S-5, S-2a(5, 7). 미리보기 길이는 D-3 결정 뒤(T039)

### Tests for User Story 5

- [ ] T026 [P] [US5] `BE-TEST/post/controller/PostSearchTest.java`: S-5의 1 ~ 8 — `단풍` → K1·K2·K3·K6(블로그가 달라도), `단풍 명소` → K3만, `spring boot`·`SPRING BOOT` 같은 결과, 로그아웃·**A로 로그인한 채** `비밀단어` → 0건, 15건이면 10 + 5이고 최신이 위, 한 줄에 `blogName`·`categoryName`·`createdAt`·`title`·`preview`가 있고 `visibility`는 없음, `없는단어조합xyz` → `200`·`totalCount: 0`. S-2a의 5·7 — 비공개 분류 `일기`의 공개 글은 **주인 D가 로그인해도** 0건, 공개로 바꾸면 나옴. 응답 `keyword`는 앞뒤 공백을 지운 값 (FR-011 ~ FR-016, FR-017, SC-001, SC-002, SC-003, SC-004, SC-007)
- [ ] T027 [P] [US5] **검색어 한곳** `BE/post/service/SearchKeyword.java` + `BE-TEST/post/service/SearchKeywordTest.java` (research B-4, B-5, R-1): 앞뒤 공백 지우기(전각 공백 포함), 길이는 지운 뒤 **코드 포인트**로(가운데 공백 포함), 공백 문자(스페이스, 탭, 전각 공백)로 나누기, 빈 조각 버리기, 같은 단어 하나로, 단어마다 `\` → `\\`, `%` → `\%`, `_` → `\_`로 바꾸고 앞뒤에 `%`. 50자 초과는 **D-1 결정에 따름** → T040 (자리만 남긴다) (FR-010, FR-012, FR-018)

### Implementation for User Story 5

- [ ] T028 [US5] **검색 읽기 입구** `BE/post/repository/PostSearchRepository.java` (가안, T003): **고정된 쿼리 하나**(세기용·10개 읽기용)에 단어 묶음을 배열 값 하나로 넘긴다 — 조건은 "단어 묶음 중 제목에도 본문에도 없는 단어가 **하나도 없다**"(`ILIKE … ESCAPE '\'`, 단어마다 AND, 제목 OR 본문), **항상** `post.visibility = 'public'` **그리고** `category.visibility = 'public'`(주인 여부 안 봄, D-7, `003` contracts 15), `post → category → blog`로 이어 블로그 이름을 같이 읽음(D-6), 정렬 `created_at DESC, post_id DESC`, 건너뛸 수·10개도 값으로. 문자열을 이어 붙여 쿼리를 만들지 않는다(헌법 IV). 표를 SQL로만 읽어 `blog`의 클래스를 부르지 않으므로 모듈 경계는 지켜진다 — `PostVisibility` 주석에 "검색 쿼리도 같은 조건"을 적는다. 검색은 저장된 원문 그대로 찾는다(D-8) (FR-011 ~ FR-014, FR-018, research B-4, R-1, R-2)
- [ ] T029 [US5] `BE/post/service/PostSearchService.java` (읽기 전용 트랜잭션): `SearchKeyword`(T027) → 세기 → `PageNumbers`(T007; **검색 결과에서 너무 큰 번호는 D-2 결정에 따름 (추천: A, 마지막 페이지)** → T037) → 10개 읽기 → 한 줄 = `postId`, `blogId`, `blogName`, `title`, `categoryId`, `categoryName`, `createdAt`, `preview`. 미리보기는 `PostPreview`(T008)로 만들고 **길이는 D-3 결정에 따름 (추천: A, 글 목록과 같은 `explore.list.preview-length`)** → T039 (길이를 읽는 곳을 하나로 둔다). 검색 기록은 남기지 않는다(research R-6) (contracts 2 동작 3 ~ 9, FR-011 ~ FR-015)
- [ ] T030 [US5] `BE/post/controller/PostSearchController.java`: `GET /api/search/posts?q=&page=` → `PostSearchResponse`(`keyword`, `totalCount`, `page`, `totalPages`, `pageSize`, `results`), `ExploreResponses.java`에. 로그인해도 결과가 같다 (contracts 2, FR-019)
- [ ] T031 [US5] 화면: 머리글 검색창 `FE/explore/SearchBox.tsx`를 `FE/components/SiteHeader.tsx`에 넣고(제출하면 `/search?q=입력 그대로`), 검색 결과 `FE/pages/SearchPage.tsx`: 주소의 `q`로 요청, "검색 결과 {totalCount}건", `PostRow`(블로그 이름 포함, 누르면 `/posts/:postId`), `Pagination`(`q` 유지), 0건이면 "검색 결과가 없습니다" (FR-014 ~ FR-017)

**Checkpoint**: T026, T027이 통과하고 S-5, S-2a(5, 7)를 화면으로 확인한다. S-5의 7(미리보기 길이)은 T039 뒤에 다시 (PR 하나)

---

## Phase 8: User Story 6 - 검색어 입력 규칙과 특수문자 (Priority: P2)

**Goal**: 공백만·1자는 검색하지 않고 안내, 앞뒤 공백은 지우고, 검색창에는 입력한 그대로 남는다. 특수문자는 일반 글자로

**Independent Test**: quickstart S-6(1 ~ 9), S-9(1, 3). S-6의 10은 D-1 결정 뒤(T040)

### Tests for User Story 6

- [ ] T032 [P] [US6] `BE-TEST/post/controller/PostSearchKeywordTest.java`: S-6의 1 ~ 9 — `q` 없음·빈 값·공백만·`가`·`%20가%20` → `400 SEARCH_KEYWORD_TOO_SHORT` "검색어를 2자 이상 입력해 주세요"이고 **쿼리를 실행하지 않음**, `  단풍  ` = `단풍` 결과·`keyword: "단풍"`, `100%` → `할인 100%`만(`할인 1000원` 없음), `a_b` → `a_b`만(`axb` 없음), `\`·`'`·`"`·`;`·`--`·`<script>`·`(`·`)` → 모두 `200`이고 응답에 예외 이름·쿼리 없음. S-9의 1 — `' OR 1=1 --`, `'; DROP TABLE post; --`는 글자 그대로 찾고 비공개 없음, 표 그대로 (FR-010, FR-017, FR-018, SC-008, NF-07)

### Implementation for User Story 6

- [ ] T033 [US6] `PostSearchService`에 길이 검사: `SearchKeyword`로 다듬은 뒤 `explore.search.keyword-min-length`(2) 미만이면 **쿼리 전에** `SEARCH_KEYWORD_TOO_SHORT`(T005, 숫자는 설정값). 서버 검사가 마지막 방어선 (헌법 IV, FR-010, research B-5)
- [ ] T034 [US6] 화면 `SearchBox`·`SearchPage`: 요청 전에 같은 검사(앞뒤 공백 지우고 2자 이상, 코드 포인트)를 해서 짧으면 **요청하지 않고** "검색어를 2자 이상 입력해 주세요". 결과 화면 검색창은 주소의 `q`를 **입력한 그대로**(공백 포함) 채운다(새로고침에도). 50자 넘는 입력은 **D-1 결정에 따름** → T040 (FR-010, FR-017)

**Checkpoint**: T032가 통과하고 S-6(1 ~ 9), S-9(1, 3)을 화면으로 확인한다 (PR 하나, US5와 묶어도 된다)

---

## Phase 9: User Story 7 - 쓰는 기능은 로그인으로 안내 (Priority: P3)

**Goal**: 로그인하지 않아도 목록·분류별 목록·검색·페이지 넘기기에서 로그인 창이 뜨지 않는다. 쓰는 기능을 누를 때만 뜬다

**Independent Test**: quickstart S-7

- [ ] T035 [P] [US7] `BE-TEST/post/controller/ExploreAnonymousTest.java`: S-7의 1 — 로그인 없이 목록, 분류별 목록, 검색, 2페이지가 모두 `200`(어느 것도 `401` 아님, SC-010). 30일이 지난 세션·로그아웃한 쿠키로도 `200`. `POST /api/posts`가 로그인 없이 `401 UNAUTHENTICATED`인 것은 `003` T017에 있으면 다시 쓰지 않는다 (FR-019, FR-020, SC-010)
- [ ] T036 [US7] 화면 확인·보완: 로그아웃 상태에서 목록·검색이 `onUnauthenticated`(로그인 창) 신호를 한 번도 내지 않는지, 머리글·빈 목록의 `글쓰기`(`/write`, `RequireLogin`)를 누르면 로그인 창 → 로그인하면 돌아오는지(`001` contracts 9, `003` T012). 댓글·좋아요·신고 버튼은 `005`가 만든 뒤 S-7의 5를 다시 본다 (FR-020)

**Checkpoint**: T035가 통과하고 S-7(1 ~ 4)을 화면으로 확인한다 (US3이나 US4 PR에 넣어도 된다)

---

## Phase 10: 결정 반영 (D-1 ~ D-3)

**Purpose**: T002에서 사용자가 고른 뒤에 한다. **결정 전에는 시작하지 않는다.** 각 작업은 앞 단계가 남겨 둔 "한곳"만 채운다

- [ ] T037 [US1] [US5] **D-2 결정에 따름 — 잘못된 페이지 번호 (추천: A, 1보다 작거나 숫자가 아니면 1페이지, 너무 크면 마지막 페이지. 검색 결과도 같게)**: `PageNumbers`(T007)의 `outOfRange…`를 채우고 `PageNumbersTest`, `PostListTest`, `PostSearchTest`에 S-9의 2(`0`, `-1`, `abc`, `1 OR 1=1`, 아주 큰 숫자 → 서버 오류 없음, 응답에 내부 정보 없음) 줄을 더한다. B를 고르면 `ErrorCode.INVALID_PAGE`(400, 문구는 상세/04 표에 먼저)를 더하고 FR-004의 "큰 번호 → 마지막 페이지"는 그대로 둔다 (FR-004, FR-014, SC-005)
- [ ] T038 [US4] [US2] **D-2 결정에 따름 — 남의·없는·비공개 분류 번호 (추천: 가, `404 CATEGORY_NOT_FOUND` "존재하지 않는 분류입니다")**: T024의 `unknownCategory…`를 채우고 `PostListCategoryTest`에 S-4의 7, S-2a의 3·4(방문자가 비공개 분류 번호를 보낸 응답 = 아무 데도 없는 번호의 응답, **상태 코드·본문이 글자 단위로 같다**), `categoryId=abc`, S-9의 2 줄을 더한다. 선택지 `나`를 고르면 "고르지 않음"으로 보고 전체 글을 보여 준다. 화면 `CategoryFilter`는 그 응답을 받으면 선택을 풀고 안내 (FR-006, FR-007, SC-006)
- [ ] T039 [US5] **D-3 결정에 따름 — 검색 미리보기 길이 (추천: A, 글 목록과 같은 100자·같은 설정값)**: T029의 길이 읽는 곳을 채운다. B면 `explore.search.preview-length`를 `ExploreProperties`·`application.yml`·plan `설정값 목록`·상세/04 `기본값` 표에 같이 더한다. C(검색어 주변 보여 주기)는 추천대로 "추후 확장 후보"에. `PostSearchTest`에 S-5의 7 길이 줄 (FR-015)
- [ ] T040 [US6] **D-1 결정에 따름 — 50자 넘는 검색어 (추천: A, 화면 검색창은 50자까지만 받고, 화면을 거치지 않은 요청은 서버가 `400 SEARCH_KEYWORD_TOO_LONG` "검색어는 50자까지 입력할 수 있습니다"(※ 제안))**: A·C면 문구를 상세/04 `안내 문구` 표에 먼저 더하고 `ErrorCode`, `SearchKeyword`(T027, `explore.search.keyword-max-length`, 코드 포인트), 화면(`SearchBox`의 입력 제한·안내)을 고친다. B면 앞 50자만 잘라 찾고 `keyword`에 자른 값. `PostSearchKeywordTest`에 S-6의 10 (FR-010)
- [ ] T041 결정 내용을 문서에 같이 반영한다 (헌법 `작업 흐름`, T002): research `D-1 ~ D-3` 절과 `E. 요약`, plan 연결표(FR-004, FR-007, FR-010, FR-014, FR-015 줄의 `미정`)와 `정해야 할 것 요약`, contracts 1·2의 `D-n` 문장과 오류 표, quickstart S-2a(3), S-4(7), S-6(10), S-9(2)의 기대 결과와 `구현 뒤에 채울 것`, 상세/04 `안내 문구` 표(새 문구, "첫 글을 써 보세요" 추가 여부 확인), 이 목록 맨 위 표

**Checkpoint**: 바뀐 테스트가 통과하고 S-2a(3, 4), S-4(7), S-6(10), S-9(2)를 확인한다 (각 결정을 해당 이야기의 다음 PR이나 작은 PR 하나로)

---

## Phase 11: Polish & 속도 (D-4, D-5)

- [ ] T042 [P] 문서를 구현과 결정에 맞게 고친다 (헌법 `작업 흐름`): plan 서두 "아직 하나도 정하지 않았습니다"·"새로 `확정`한 것은 없습니다"와 research D 서두(D-6 ~ D-8은 결정됨), plan `정해야 할 것 요약`·`구현 순서 제안`·FR-005 줄의 D-8("그 전에는 글자 그대로" → research D-8의 B), research `E. 요약`의 D-8 줄, plan `Technical Context`(버전·테스트 `미정` → `001` ~ `003`에서 쓰는 것, Redis 줄과 research A의 Redis 줄은 헌법 1.1.0 문구로), `Project Structure`의 `search` 모듈(T003 결과), data-model 4·6(`idx_category_blog`는 `V2`의 `idx_category_blog_id`로 이미 있음, CHECK 요청은 `ERD-변경-요청.md` T-3으로 끝남, 인덱스는 "팀 요청"이 아니라 내 확장), research R-5(`002` 결정: 회원 줄은 남기고 블로그·글은 `BlogClosingEvent`로 지운다). `003`의 이름(`BlogDirectory`, `PostVisibility`, `LoggedInMember`, 주소)이 바뀌었으면 이 목록도
- [ ] T043 [P] `003`에 남겨 둔 연결 채우기: 글 상세의 `목록으로`(`003` T029: "분류 목록 주소는 `004`가 정하면 바꾼다")를 `/blog/:blogId?category=` 로, 글 삭제 뒤 이동(`003` T037)이 내 블로그 글 목록으로 가는지. `FE/pages/HomePage.tsx` 맨 위 주석("글 목록·주제별 탐색은 specs/004")을 고친다 — 주제별 화면은 개인 기능이라 `004` 밖이다
- [ ] T044 [P] quickstart S-9(3 ~ 5)를 화면·DB로 실행한다(제목 `<script>`가 글자로, 서버 로그의 조건에 공개 글 조건이 항상 있음, 잘못된 `visibility` 값은 `ck_post_visibility`가 막음). `003` quickstart S-9(3, 4), S-9a(5)를 목록·검색으로 다시 확인한다 (`003` tasks `Incremental Delivery` 4) (FR-006, FR-011, SC-001, SC-002)
- [ ] T045 **D-4 결정에 따름 — 2초 측정 (추천: A + 가, 정한 글 수·사용자 수로 재고 검색에도 2초)**: 시험용 글을 한 번에 넣는 방법을 만들고(quickstart `구현 뒤에 채울 것`), S-8의 1 ~ 5를 실행한다. 느리면 `EXPLAIN`으로 인덱스가 쓰이는지 본다 (SC-009, NF-09)
- [ ] T046 T045에서 목록이 느릴 때만: data-model 4의 부분 인덱스 `idx_post_public_created`(가안)를 다음 빈 Flyway 번호(`V4…` 가안)로 더한다. 더하기 전에 사용자에게 확인하고, 더하면 `docs/3-설계/ERD-변경-요청.md`의 E-항목과 Crowfoot `myblog-제안`(666)에 같이 적는다. 블로그 전체 목록이 계속 느리면 D-6의 B(`post.blog_id`)를 다시 묻는다 (SC-009, data-model 4)
- [ ] T047 **D-5 결정에 따름 — 검색 속도 대비 (추천: A로 시작, T045에서 느리면 B: `pg_trgm` 확장과 제목·본문 trigram 색인)**: B를 고르면 Flyway에 `CREATE EXTENSION`과 색인을 더한다. 개발용 이미지(`pgvector/pgvector:pg18`)에 `pg_trgm`이 들어 있는지, 배포 DB에서 확장을 만들 권한이 있는지(배포는 `001` D-5 `미정`) 먼저 확인한다. 쿼리 모양(T028)은 그대로 (research D-5)
- [ ] T048 quickstart 전체를 실행하고 끝의 `구현 뒤에 채울 것`을 채운다. `tasks.md` 체크박스와 `CLAUDE.md`의 `6. 지금 상태`를 고친다

---

## 요구사항 → 작업 연결표

| FR | 작업 |
|---|---|
| FR-001 | T009, T014, T015, T017 |
| FR-002 | T004, T007, T014, T015, T017 |
| FR-003 | T012, T017 |
| FR-004 | T007, T014, T017, T037 (1보다 작은 번호 등은 D-2) |
| FR-005 | T004, T008, T012, T014, T015 |
| FR-006 | T006, T009, T015, T018, T019, T020, T024, T038 |
| FR-007 | T023, T024, T025, T038 (남의 분류 번호는 D-2) |
| FR-008 | T006, T021, T022 |
| FR-009 | T009, T014, T015, T017 |
| FR-010 | T004, T005, T027, T032, T033, T034, T040 (50자 초과는 D-1) |
| FR-011 | T026, T028 |
| FR-012 | T026, T027, T028 |
| FR-013 | T026, T028 |
| FR-014 | T026, T028, T029, T037 (큰 번호는 D-2) |
| FR-015 | T012, T026, T029, T030, T031, T039 (미리보기 길이는 D-3) |
| FR-016 | T026, T031 |
| FR-017 | T026, T031, T032, T034 |
| FR-018 | T027, T028, T032 |
| FR-019 | T010, T016, T018, T030, T035 |
| FR-020 | T035, T036 |

| SC | 확인하는 작업 |
|---|---|
| SC-001 | T018, T026, T044 |
| SC-002 | T018, T026, T044 |
| SC-003 | T014, T026 |
| SC-004 | T014, T026 |
| SC-005 | T007, T014, T037 |
| SC-006 | T023, T038 |
| SC-007 | T026 |
| SC-008 | T032, T037 |
| SC-009 | T045, T046, T047 (D-4, D-5) |
| SC-010 | T018, T035, T036 |

---

## Dependencies & Execution Order

### Phase Dependencies

- **Setup (Phase 1)**: T001의 `003` merge 조건을 먼저 본다. T002(결정 요청)는 바로 한다
- **Foundational (Phase 2)**: `003` T003 ~ T012, T014 ~ T015 merge 뒤. **모든 사용자 이야기를 막는다**
- **US1 (P1)**: Foundational 다음. 화면(T017)은 `003` T016 뒤
- **US2 (P1)**: US1 다음 (같은 서비스를 점검한다)
- **US3 (P2)**: US1 다음. 화면(T022)은 `003` T024 뒤
- **US4 (P2)**: US1 다음
- **US5 (P2)**: Foundational 다음 (US1과 따로 해도 된다. `PageNumbers`, `PostPreview`를 같이 쓴다)
- **US6 (P2)**: US5 다음
- **US7 (P3)**: US1, US5 다음. 화면 확인은 `003` T024, T029 뒤
- **결정 반영 (Phase 10)**: 해당 결정(T002) 뒤. T037은 US1·US5, T038은 US4, T039는 US5, T040은 US6 다음
- **Polish (Phase 11)**: T045는 D-4 결정 뒤, T047은 T045와 D-5 결정 뒤. T046은 T045에서 느릴 때만

### Within Each User Story

- 테스트 → 저장소·한곳(설정, 페이지, 미리보기, 검색어) → 서비스 → 주소(컨트롤러) → 화면 순서
- 테스트는 먼저 써 두고 실패하는 것을 본 뒤 구현한다. push 전에 `mvn verify`와 화면 `lint`·`build`를 돌린다
- 한 이야기를 끝내고 Checkpoint를 통과한 뒤 PR을 merge하고 다음으로 간다

### Parallel Opportunities

- T004, T005, T006, T007, T008은 동시에. 화면의 T011, T012도 동시에
- US1(목록)과 US5(검색)는 서버 파일이 겹치지 않아 동시에 할 수 있다 (`ExploreResponses.java`만 같이 고치므로 차례로)
- US5의 T026, T027은 동시에
- 결정이 나오면 T037 ~ T040은 서로 다른 곳을 고쳐 동시에 해도 된다 (T037과 T038은 `PostListCategoryTest`·`PostListTest`를 같이 고치므로 한 사람이)

---

## Parallel Example: User Story 5

```text
먼저 동시에:
Task: "T026 PostSearchTest (S-5, S-2a의 5·7)"
Task: "T027 SearchKeyword (공백 지우기, 단어 나누기, %·_·\ 를 일반 글자로)"
그다음: T028 → T029 → T030 → T031
```

---

## Implementation Strategy

### MVP First (US1 + US2)

1. `003`의 Foundational과 US1이 merge되기를 기다리며 T002(결정 요청), T003을 한다
2. Phase 2를 끝낸다 (US1 PR에 같이 넣는다)
3. US1(목록) + US2(공개 범위) → **멈추고 확인**: quickstart S-1, S-2
4. 보여 줄 수 있으면 보여 준다

### Incremental Delivery

1. US3(빈 목록) + US4(분류) → 확인
2. US5(검색) + US6(검색어 규칙) → 확인
3. US7(로그인 안내) → 확인
4. 결정이 나오는 대로 Phase 10 (D-1 ~ D-3)
5. D-4가 정해지면 T045로 재고, 결과를 보고 D-5를 묻는다

---

## Notes

- 작업 하나 또는 묶음 하나를 끝낼 때마다 커밋하고, 커밋 메시지에 작업 ID를 적는다 (예: `004 T015`). PR은 이야기 하나에 하나
- `가안`인 주소(`/search`, `?category=`, `?page=`), 오류 이름, 설정 이름(`explore.*`), 입구 이름(`BlogDirectory.blog`, `PageNumbers`, `PostPreview`, `SearchKeyword`, `PostSearchRepository`)을 바꾸면 T042처럼 문서도 같이 고친다
- 남은 결정: **D-1 ~ D-5** (모두 `미정`, 추천은 맨 위 표). 이 목록은 추천안을 미리 만들지 않는다. 확인할 위험: research R-1(`%`, `_`, T027·T032), R-2(대소문자와 DB 언어 설정), R-3(페이지 사이 글이 늘고 줄어듦 — 받아들임), R-4(개수와 목록이 잠깐 어긋남, T009), R-5(탈퇴 회원의 글 — `003` T036으로 지워짐), R-6(검색 기록 안 남김)
