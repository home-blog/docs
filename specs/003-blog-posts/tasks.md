---

description: "003 블로그·분류·글 작업 목록"
---

# Tasks: 블로그·분류·글

**Input**: `/specs/003-blog-posts/`의 설계 문서

**Prerequisites**: [plan.md](plan.md), [spec.md](spec.md), [research.md](research.md), [data-model.md](data-model.md), [contracts/blog-post-api.md](contracts/blog-post-api.md), [quickstart.md](quickstart.md)

**결정 반영 (2026-10-08)**: research의 `D-1 ~ D-7`이 모두 정해졌다 (`D-2`는 2026-10-08). 이 목록은 그 결정을 따른다. 마크다운 표시 도구도 2026-10-08에 정해졌다 (research D-8, T025).

| ID | 결정 | 상태 | 이 목록에서 |
|---|---|---|---|
| D-1 | 마크다운 문법만 그리고, 본문의 HTML은 글자 그대로, `javascript:` 같은 위험한 링크는 막는다 | 결정됨 | T025, T029, T053 |
| D-2 | 본문 글자 수는 **저장하는 원문(마크다운 기호·이미지 주소 포함) 그대로** 센다(A). 공백·줄바꿈만 있는 본문은 **비어 있는 것**으로 보고 "본문을 입력해 주세요" | 결정됨 (2026-10-08) | T017, T019, T020, T024 |
| D-3 | 글마다 주제를 고른다. `post.topic_id` 그대로 (필수) | 결정됨 | T003, T021, T022, T024 |
| D-4 | 회원–블로그 1:1 (`blog.users_id` `UNIQUE`, `V2`에 있음) | 결정됨 | T013 |
| D-5 | `sort_order` 그대로, 이름은 소문자 비교로 중복 불가(`V2`의 `uq_category_blog_name`), `category.visibility` 사용 | 결정됨 | T010, T014, T043, T044 |
| D-6 | 화면이 만든 1회용 요청 번호를 `post.request_key`(E-4)에 저장해 한 번만 만든다 | 결정됨 | T003, T018, T022, T024 |
| D-7 | 글을 지우면 그 글·댓글의 신고 기록도 함께 지운다 (`005`가 듣는 쪽을 만든다) | 결정됨 | T032 |
| D-8 | D-1을 지키는 마크다운 표시 도구: **react-markdown + remark-gfm** | 결정됨 (2026-10-08) | T025, T029 |

**Tests**: `001`·`002`처럼 **사용자 이야기마다 서버 테스트 작업**을 넣었다 (`@SpringBootTest` + MockMvc + `springSecurity()`, 실제 PostgreSQL). 테스트 이름에는 [quickstart.md](quickstart.md)의 시나리오 번호를 붙인다. 화면에는 테스트 도구가 없어서(`frontend/package.json`) 각 단계 끝의 `Checkpoint`에서 손으로(또는 Playwright로) 확인한다.

**Organization**: 사용자 이야기(US)별로 묶었다. PR도 이야기 하나에 하나 (CLAUDE.md `커밋과 올리기`). 코드는 `home-blog/blog`, 이 목록은 `home-blog/docs`에 있다.

## Format: `[ID] [P?] [Story] 설명`

- **[P]**: 동시에 해도 되는 작업 (다른 파일이고, 끝나지 않은 작업에 기대지 않음)
- **[Story]**: 어느 사용자 이야기의 작업인지 (US1 ~ US7)

## 경로 약속

> `002`와 같다. 코드는 `home-blog/blog` 저장소에 있다.

| 줄임 | 실제 경로 |
|---|---|
| `BE/` | `backend/src/main/java/com/myblog/` (서버, Spring Boot) |
| `BE-RES/` | `backend/src/main/resources/` |
| `BE-TEST/` | `backend/src/test/java/com/myblog/` (서버 테스트) |
| `FE/` | `frontend/src/` (화면, React + Vite) |

## 쉬운 설명: 이미 있는 것과 새로 만드는 것

| 이미 있는 것 (`001`, `002`) | 이 기능에서 |
|---|---|
| `blog`, `category` 표 (`V2__auth_tables.sql`): `uk_blog_users_id`(1인 1블로그), `uq_category_blog_name`(소문자 비교), `ck_category_visibility`, `sort_order`, `is_default`, `color_index` | 표는 그대로 쓴다. **새로 만드는 표는 `topic`, `post`뿐** (`V3`) |
| `Blog`, `Category` 엔티티, `BlogRepository`, `CategoryRepository`, `BlogProvisioner`(`MemberRegisteredEvent`를 듣고 블로그와 `미분류`를 만듦) | US1은 **이미 동작한다.** 확인 테스트와 읽기 주소만 더한다. 엔티티에 고치는 메서드를 더한다 |
| `ErrorCode`, `ApiException`, `GlobalExceptionHandler`, `FieldErrorMessages`(설정값으로 문구 숫자 채우기) | 새 오류 이름과 `BlogFieldErrorMessages`, `PostFieldErrorMessages`를 더한다 |
| `AccountProperties`(`@ConfigurationProperties` record, DB 칸보다 크면 서버가 켜지지 않음) | 같은 모양으로 `BlogProperties`, `CategoryProperties`, `PostProperties` |
| `CurrentMemberService`(세션 회원을 DB에서 다시 확인), `MemberPrincipal` | `MemberPrincipal`은 `user` **안쪽**이라 `blog`·`post`가 직접 쓸 수 없다 → `user` 맨 위에 질문 틀을 둔다 (T007) |
| `MemberBlogLookup`(틀은 `user`, 채우기는 `blog`) | 같은 방식: `blog`가 틀(`CategoryPostCounter`)을 두고 `post`가 채운다 (T009, T010) |
| `BlogClosingEvent` (`002` US3 T032가 `blog` 맨 위에 만든다) | `post`가 듣고 **자기 글을 스스로** 지운다 (T036) |
| `createBrowserRouter`(`FE/App.tsx`), `RequireLogin`, `api/client.ts`의 `ApiError`, `useUnsavedChangesPrompt`, `index.css`의 원고지 토큰, `SiteHeader`의 `MEMBER_ONLY_PREFIXES`(`/manage`, `/write`가 이미 있음) | 글쓰기·관리 화면 주소를 `/write…`, `/manage…`로 맞춘다 (가안) |

**모듈 방향** (Spring Modulith, `ModularityTest`): `user ← blog ← post ← comment/community/image`. `post`는 `blog`·`user`의 **맨 위 패키지**만 부를 수 있고, `blog`는 `post`를 부를 수 없다. 그래서 "분류의 글 개수"는 `blog`가 틀을 두고 `post`가 채우며(T009·T010), 글을 지울 때 댓글·좋아요 등은 `post`가 **이벤트를 내고** `005`의 모듈이 들어서 지운다(T032). 모든 듣는 쪽은 `@EventListener`로 **같은 트랜잭션 안에서** 돈다 (`002` US3와 같음).

---

> **진행 (2026-10-08)**: 바탕 + US1 = 코드 PR #20, US2 = PR #21 (둘 다 merge). US2 리뷰에서 "같은 `requestKey`에 다른 내용이면 `409 POST_ALREADY_SAVED`"를 더했다(contracts 10). US3 = PR #23. US4·US5, US6, US7은 브랜치에 만들어 두고 차례로 올린다. T011의 글 요청 함수(`postApi.ts`)와 `rules.ts`는 US2에서, T012의 `/write`·`/posts/:postId` 주소는 US2·US3에서 더했다. T012의 `useUnsavedChangesPrompt`를 `components/`로 옮기는 것은 하지 않았다(`002` 화면들이 같은 파일을 쓰고 있어, 옮기면 얻는 것보다 바꿀 곳이 많다). 머리글의 `내 블로그`는 `/me/blog`(내 블로그 번호를 물어 이동)로 만들었다.

## Phase 1: Setup (공통 준비)

**Purpose**: 새 프로젝트 준비는 없다 (`001`에서 끝남). 확인만 한다

- [x] T001 시작 전 확인: `002` US2가 `main`에 merge되어 있고 CI가 통과하는지 본다. T036은 `002` US3의 `BlogClosingEvent`(`002` T032)가 merge된 뒤에 한다. US마다 브랜치를 만든다 (`feat/003-us1-blog`, `feat/003-us2-write`, … 가안)
- [x] T002 D-2 결정(2026-10-08, A + 공백만 있으면 비어 있음)이 research D-2·E, plan, spec Assumptions에 반영됐는지 확인한다 (커밋 `a954069`). 남은 문서 정리는 T051

---

## Phase 2: Foundational (모든 이야기의 바탕)

**Purpose**: US1 ~ US7이 함께 기대는 표, 모듈, 설정, 오류, 권한 입구

**⚠️ CRITICAL**: 이 단계가 끝나기 전에는 사용자 이야기 작업을 시작하지 않는다

- [x] T003 [P] Flyway `BE-RES/db/migration/V3__blog_posts.sql`: ① `topic` 표(칸은 Crowfoot `myblog-제안`(666)의 DDL과 같게) + 주제 5줄(여행, 음식, 취미, 운동, 개발, `sort_order` 1 ~ 5) ② `post` 표: 팀 ERD 칸(`category_id` → `category`, `topic_id` → `topic` 외래 키, **연쇄 삭제 없음**, `title` VARCHAR(100), `content` TEXT, `visibility` 기본 `public`, `views` 기본 0, `created_at` NOT NULL, `updated_at` NULL) + T-2(`content` 기본값 없음) + T-3(`ck_post_visibility`) + E-4(`request_key` VARCHAR(36) NULL `UNIQUE`) ③ 인덱스 `post (category_id, created_at, post_id)` (data-model 3 추천, 가안). 맨 위 주석에 T·E 번호를 적는다 (`V2`처럼) (FR-018, FR-028, FR-034, FR-046, SC-005)
- [x] T004 `post` 모듈 뼈대 — T003 다음: `BE/post/package-info.java`(`@ApplicationModule(displayName = "글", allowedDependencies = {"blog", "user", "common"})`), `BE/post/domain/Post.java`, `BE/post/domain/Topic.java`(읽기만), `BE/post/repository/PostRepository.java`, `TopicRepository.java`. `Post.createdAt`은 `@CreatedDate`, **`updatedAt`에는 `@LastModifiedDate`를 쓰지 않는다**(research R-2: 서비스가 바뀌었을 때만 직접 넣는다). `ddl-auto: validate`로 `V3`와 맞는지 확인한다
- [x] T005 [P] 설정값 묶음 (plan `설정값 목록`, 헌법 VI): `BE/blog/config/BlogProperties.java`(`blog.name` 1/30, `blog.intro` 0/200), `BE/blog/config/CategoryProperties.java`(`category.name` 1/20), `BE/post/config/PostProperties.java`(`post.title` 1/100, `post.content` 1/10000)를 `AccountProperties`처럼 record로 만들고 `BE-RES/application.yml`에 더한다. 최대값이 DB 칸(`blog.name` 30, `blog.intro` 500, `category.name` 20, `post.title` 100)보다 크면 서버가 켜지지 않게 한다
- [x] T006 [P] `BE/common/error/ErrorCode.java`에 이 기능의 오류를 더한다 (contracts 문구 그대로, `※`는 제안 문구): `POST_NOT_FOUND`(404 "존재하지 않는 글입니다"), `BLOG_NOT_FOUND`(404), `CATEGORY_NOT_FOUND`(404), `INVALID_CATEGORY`(400), `CATEGORY_NAME_DUPLICATED`(409 "이미 있는 분류입니다"), `DEFAULT_CATEGORY_NOT_DELETABLE`(409), `CATEGORY_HAS_POSTS`(409, `{N}`은 서비스가 채움), `INVALID_CATEGORY_ORDER`(400), 칸별 `TITLE_REQUIRED`("제목을 입력해 주세요"), `TITLE_TOO_LONG`, `CONTENT_REQUIRED`("본문을 입력해 주세요"), `CONTENT_TOO_LONG`, `CATEGORY_REQUIRED`, `TOPIC_REQUIRED`("주제를 골라 주세요"), `VISIBILITY_INVALID`, `BLOG_NAME_REQUIRED`, `BLOG_NAME_TOO_LONG`, `BLOG_INTRO_TOO_LONG`, `CATEGORY_NAME_REQUIRED`, `CATEGORY_NAME_TOO_LONG` (FR-016, FR-026, FR-036, FR-040, FR-042, FR-046)
- [x] T007 [P] **로그인한 회원 번호를 다른 모듈에 알려 주는 틀** `BE/user/LoggedInMember.java`(`user` 맨 위, 가안): `Optional<Long> idOf(Authentication)`(읽기 주소용, 로그인 안 했으면 비움), `Long requireIdOf(Authentication)`(없거나 탈퇴했으면 `401 UNAUTHENTICATED`). `user` 안쪽에서 `CurrentMemberService`로 채운다(DB 다시 확인). `blog`·`post` 컨트롤러는 `MemberPrincipal`을 쓰지 않고 이것만 쓴다 (FR-008, FR-044, `002` research B-1)
- [x] T008 `BE/user/config/SecurityConfig.java`: 누구나 읽는 주소 `GET /api/blogs/**`, `GET /api/posts/{postId}`(숫자만, `/edit`는 빼고)를 `permitAll`에 더한다. 나머지(`/api/me/**`, 변경 요청)는 지금처럼 `anyRequest().authenticated()` → `401` (contracts 공통, FR-008)
- [x] T009 `blog` 모듈의 입구 (맨 위 패키지, 가안): ① `BE/blog/BlogDirectory.java` — `post`가 묻는 것: `myBlog(memberId)`, `category(categoryId)` → `CategoryInfo(categoryId, blogId, ownerId, blogName, name, visibility, isDefault)`, `categoriesOf(blogId)`, `publicCategoryIds(blogId)`. 채우기는 `BE/blog/service/BlogDirectoryAdapter.java` ② `BE/blog/CategoryPostCounter.java` — `blog`가 묻고 `post`가 채우는 틀: `countAll(categoryId)`(비공개 포함), `countVisible(categoryIds)`(공개 글만) (FR-034, FR-040, FR-043, research B-2)
- [x] T010 **"보이는 글" 조건을 한곳에** — T004, T009 다음: `BE/post/service/PostVisibility.java`(research B-3): 주인이면 모두, 아니면 `post.visibility = 'public'` **그리고** `category.visibility = 'public'`. 상세·분류 개수·이전/다음이 모두 이것을 쓴다(`004` 목록·검색도). `BE/post/service/CategoryPostCounterAdapter.java`가 T009의 틀을 이 조건으로 채운다 (FR-026, FR-030, FR-031, FR-043, FR-048, SC-002)
- [ ] T011 [P] 화면 요청 함수와 규칙: `FE/blog/blogApi.ts`(contracts 1 ~ 8), `FE/post/postApi.ts`(contracts 9 ~ 14), 응답 타입은 contracts 그대로. `FE/post/rules.ts`에 글자 수와 문구(서버 설정·contracts와 같게)
- [ ] T012 [P] 화면 주소(가안) `FE/App.tsx`: `/blog/:blogId`(블로그), `/posts/:postId`(글 상세), `<RequireLogin>`으로 `/write`(새 글), `/write/:postId`(수정), `/manage/blog`(이름·소개), `/manage/categories`(분류). `FE/components/SiteHeader.tsx`에 `글쓰기`, `내 블로그` 링크. `FE/account/useUnsavedChangesPrompt.ts`를 `FE/components/`로 옮겨 같이 쓴다 (FR-008, FR-017)

**Checkpoint**: `mvn verify`(CI)와 `ModularityTest`가 통과한다(`post → blog → user` 방향만 있음). 로그인하지 않고 `GET /api/blogs/1`이 `401`이 아니다. 새 주소들이 빈 화면으로 열린다

---

## Phase 3: User Story 1 - 가입하면 내 블로그가 만들어진다 (Priority: P1) 🎯 MVP

**Goal**: 가입하면 "{닉네임}의 블로그"와 `미분류`가 생기고(이미 동작), 내 블로그와 분류 목록을 볼 수 있다

**Independent Test**: quickstart S-1

### Tests for User Story 1

- [x] T013 [P] [US1] `BE-TEST/blog/controller/BlogReadTest.java`: S-1의 2·3(`GET /api/me/blog` 이름·빈 소개, 분류는 `미분류` 하나·`isDefault` 참), S-1의 5(같은 회원으로 블로그를 하나 더 저장하면 `uk_blog_users_id`가 거절, D-4), `GET /api/blogs/999999` → `404 BLOG_NOT_FOUND`, 로그인하지 않고 `GET /api/blogs/{id}`·`…/categories` 성공. S-1의 1·4·6(가입 묶음, 실패하면 모두 취소)은 `SignupFlowTest`에 이미 있으면 다시 쓰지 않는다 (FR-001 ~ FR-003, SC-001)

### Implementation for User Story 1

- [x] T014 [US1] `BE/blog/service/BlogQueryService.java`: `myBlog(memberId)`, `blog(blogId, viewerId)`(`isOwner`), `categories(blogId, viewerId)` — `sort_order`, 같으면 `category_id` 순서. 주인이 아니면 **비공개 분류를 빼고**, 글 개수는 `CategoryPostCounter`로 주인이면 비공개 포함·아니면 공개 글만. 이름은 요청마다 표에서 읽는다(캐시 없음) (contracts 1 ~ 3, FR-038, FR-039, FR-043, FR-048, SC-010)
- [x] T015 [US1] `BE/blog/controller/BlogController.java`: `GET /api/me/blog`, `GET /api/blogs/{blogId}`, `GET /api/blogs/{blogId}/categories`. 회원은 T007의 `LoggedInMember`로만 정한다 (FR-001, FR-006)
- [x] T016 [US1] 블로그 화면 `FE/pages/BlogHomePage.tsx` + `blog.css`(원고지 토큰): 이름, 소개, 분류 목록과 글 개수, 주인에게만 비공개 분류에 "비공개" 표시와 `블로그 설정` 버튼. 글 목록 자리는 `004`가 채운다. 마이페이지의 `/blog/{id}` 링크가 여기로 온다

**Checkpoint**: T013이 통과하고 S-1을 화면으로 확인한다 (PR 하나)

---

## Phase 4: User Story 2 - 내 블로그에 글 쓰기 (Priority: P1)

**Goal**: 로그인한 회원이 제목·본문·분류·주제·공개 여부를 정해 글을 쓰고, 저장하면 상세 화면으로 간다. 연속으로 눌러도 한 번만 저장된다

**Independent Test**: quickstart S-2, S-2a(1 ~ 4, 7), S-3, S-4, S-5

### Tests for User Story 2

- [x] T017 [P] [US2] `BE-TEST/post/controller/PostCreateTest.java`: S-2의 1 ~ 5(기본값 `미분류`·주제 `null`·`public`, 마지막 글의 분류·주제, `createdAt`을 보내도 무시), S-2a의 1 ~ 4·7(주제 없음·없는 번호 → `topicId` "주제를 골라 주세요"), S-3의 1 ~ 8(빈 제목·공백 제목, 빈 본문, 둘 다 비면 두 칸, 101자·100자, 앞뒤 공백, 이모지 100자), S-3의 5·6(본문은 **저장 원문 기준** 10,000자 통과·10,001자 거절, 마크다운 기호·이미지 주소도 센다), S-3의 9(공백·줄바꿈만 있는 본문 → `content` "본문을 입력해 주세요") (D-2), S-4의 1·3·4(로그인 안 하면 `401`, 남의 분류 번호 → `400 INVALID_CATEGORY`, 본문의 `blogId` 무시), S-12의 6 ~ 8(SQL 같은 글자 그대로, CSRF 없으면 거절, 화면 없이 101자 거절) (FR-008 ~ FR-016, FR-046, SC-013)
- [x] T018 [P] [US2] `BE-TEST/post/controller/PostRequestKeyTest.java` (**클래스 전체 `@Transactional` 쓰지 않음**, 끝에서 지움): S-5의 2·3 — 같은 `requestKey`로 요청 3개를 동시에 보내면 글은 하나, 세 응답의 `postId`가 같다(처음은 `201`, 나머지는 `200`). 다른 키면 글 두 개 (D-6, FR-018, SC-008)

### Implementation for User Story 2

- [x] T019 [P] [US2] 입력 검사 `BE/post/validation/`: `@ValidTitle`(앞뒤 공백을 지운 뒤 1 ~ `post.title.max-length`, **코드 포인트로 셈**, 비면 `TITLE_REQUIRED`), `@ValidContent`(1 ~ `post.content.max-length`, 앞뒤 공백은 지우지 않음. **저장하는 원문 그대로 코드 포인트로 센다**(마크다운 기호 포함). 공백·줄바꿈만 있으면 `CONTENT_REQUIRED`, D-2), `PostFieldErrorMessages`(문구의 숫자를 `PostProperties`에서) (FR-010, FR-011, FR-016, research B-4)
- [x] T020 [US2] `BE/post/domain/Post.java`에 `create(categoryId, topicId, title, content, visibility, requestKey)`: 제목은 앞뒤 공백 제거, 본문은 `\r\n` → `\n`만 하고 그 밖에는 손대지 않는다 (글자 수는 이렇게 맞춘 원문으로 센다, D-2), `visibility` 없으면 `public`. `created_at`은 서버가 넣는다 (FR-010, FR-011, FR-013, FR-014)
- [x] T021 [US2] `BE/post/service/PostFormService.java`: 내 블로그의 분류 목록(T009), 주제 목록(`sort_order`), 기본 분류·주제 = 내 블로그에서 **작성 시각이 가장 늦은 글**의 것(없으면 `is_default` 분류, 주제 `null`), `defaultVisibility: "public"`, `limits`는 `PostProperties`에서 (contracts 9, research B-6, FR-012, FR-013, FR-046)
- [x] T022 [US2] `BE/post/service/PostWriteService.create`: ① 블로그는 세션 회원으로(`BlogDirectory.myBlog`), 요청에 블로그 번호를 받지 않음 ② `categoryId`가 내 블로그의 것이 아니면 `INVALID_CATEGORY` ③ `topicId`가 `topic` 표에 없으면 `fieldErrors.topicId` `TOPIC_REQUIRED` ④ 같은 `requestKey`의 글이 있으면 그 번호를 돌려줌, 동시에 들어와 `request_key` 중복으로 DB가 거절하면 다시 찾아 그 번호를 돌려줌 (D-6) ⑤ 저장 (FR-008, FR-009, FR-018, FR-034, FR-046) — T019 ~ T021 다음
- [x] T023 [US2] `BE/post/controller/PostController.java`: `GET /api/me/blog/post-form`, `POST /api/posts`(`201 { postId }`, 같은 키 다시 오면 `200`). 요청 본문 `BE/post/controller/dto/PostRequests.java`에 작성 시각·블로그 번호 칸 없음. 제목·본문이 모두 비면 두 칸을 함께 (contracts 9·10, FR-014, FR-015)
- [x] T024 [US2] 글쓰기 화면 `FE/post/PostEditorPage.tsx` + `post-editor.css`: 제목, 마크다운 본문(글자 입력 칸), 분류·주제 고르기(주제는 처음에 비어 있음), 공개/비공개. 화면을 열 때 `crypto.randomUUID()`로 `requestKey`를 한 번 만든다. 저장 중 버튼 잠금, 칸별 오류(`ApiError.messageFor`), 실패해도 입력 유지, 성공하면 `/posts/:postId`, 나가기 확인(`useUnsavedChangesPrompt`). 화면의 글자 수 표시는 **입력한 원문 그대로**(코드 포인트, 서버와 같게), 공백만 있으면 "본문을 입력해 주세요" (D-2) (FR-008 ~ FR-018, FR-046)

**Checkpoint**: T017, T018이 통과하고 S-2 ~ S-5를 화면으로 확인한다. 로그인하지 않고 `글쓰기`를 누르면 로그인 창이 뜨고 돌아온다 (PR 하나)

---

## Phase 5: User Story 3 - 글 읽기 (Priority: P1)

**Goal**: 누구나 공개 글을 읽고 이전·다음 글로 넘어간다. 없는 글과 볼 수 없는 글은 똑같이 "존재하지 않는 글입니다"

**Independent Test**: quickstart S-6, S-12(1 ~ 4)

> 상세 화면은 D-1(결정됨)을 지키는 **도구가 정해진 뒤에** 마크다운을 그린다. 그 전에는 원문 글자 그대로 보여 준다 (plan Constitution Check IV).

- [x] T025 [US3] **마크다운 표시 도구 고르기 (미정)**: D-1의 A(마크다운만 그림, HTML은 글자 그대로, 위험한 링크 차단)를 지키는 화면 도구 후보를 research에 새 `D-항목`으로 적는다 — 선택지, 쉬운 설명, 장단점, 추천, "HTML을 그리지 않는 설정·링크 주소 거르기가 기본인지". 사용자가 고르면 plan `Technical Context`의 `주요 도구` 줄을 고친다

### Tests for User Story 3

- [ ] T026 [P] [US3] `BE-TEST/post/controller/PostReadTest.java`: S-6의 1 ~ 4·6·7(로그인 없이 공개 글, `updatedAt` `null`, 1→2→3 순서의 이전·다음, 끝에서는 `null`, 사이에 낀 비공개 글은 건너뜀, 없는 번호 `404`, `isOwner`), S-9의 5·6(남의 비공개 글의 응답이 없는 번호의 응답과 **상태 코드·본문이 글자 단위로 같다**), S-9a의 4(비공개 분류의 공개 글도 같은 `404`) (FR-023 ~ FR-027, FR-031, FR-048, SC-002, SC-003)

### Implementation for User Story 3

- [ ] T027 [US3] `BE/post/service/PostReadService.java`: 글 → `BlogDirectory.category`로 블로그·주인·분류 이름 → `PostVisibility`로 볼 수 없으면 **없는 글과 같은** `POST_NOT_FOUND`. 이전·다음 = 같은 블로그의 공개 분류(`publicCategoryIds`)의 공개 글 중 (작성 시각, 글 번호) 바로 앞·뒤, 주인이 봐도 공개 글만 (research B-5: "이전 = 바로 앞에 쓴 글", 가안). 블로그·분류 이름은 요청마다 읽는다 (FR-023 ~ FR-026, FR-038, FR-048, research R-6)
- [ ] T028 [US3] `PostController`에 `GET /api/posts/{postId}`: 응답은 contracts 11 그대로(`content`는 원문, `topic`, `prevPostId`, `nextPostId`, `isOwner`)
- [ ] T029 [US3] 글 상세 `FE/post/PostDetailPage.tsx` + `FE/post/MarkdownView.tsx`: 분류·제목·블로그 이름·작성 시각(수정 시각은 있을 때만)·본문, 없는 쪽 이전/다음 버튼 숨김, `목록으로`는 `/blog/:blogId`(분류 목록 주소는 `004`가 정하면 바꾼다), `isOwner`일 때만 `수정`·`삭제`. `MarkdownView`는 T025가 정해지기 전에는 **원문 글자 그대로**(`white-space: pre-wrap`), 정해지면 D-1의 A대로 그린다. 제목·이름은 React 기본 이스케이프 (FR-011, FR-023 ~ FR-027, FR-045, SC-012)

**Checkpoint**: T026이 통과하고 S-6, S-12의 1 ~ 4를 화면으로 확인한다. 여기까지가 MVP다 (PR 하나)

---

## Phase 6: User Story 4 - 글 수정과 삭제 (Priority: P2)

**Goal**: 작성자만 글을 고치고 지운다. 바뀐 것이 없으면 수정 시각은 그대로다. 지우면 딸린 것도 한 묶음으로 지운다

**Independent Test**: quickstart S-7, S-8(지금 있는 표만), S-2a(5, 6)

### Tests for User Story 4

- [ ] T030 [P] [US4] `BE-TEST/post/controller/PostEditTest.java`: S-7의 1 ~ 7(제목만 바꾸면 `changed: true`·`updatedAt` 생김·`createdAt` 그대로, 안 바꾸면 `changed: false`이고 **DB 값도 그대로**, 앞뒤 공백만 더하면 안 바뀐 것, 공개 여부만 바꿔도 갱신, 남의 글 `GET …/edit`·`PUT` → `404`이고 DB 그대로, **남의 글에 형식이 틀린 본문을 보내도 `400`이 아니라 `404`**), S-2a의 5·6(주제만 바꾸기, 주제 빼면 `400`) (FR-019, FR-020, FR-033, FR-041, SC-004, SC-009)
- [ ] T031 [P] [US4] `BE-TEST/post/controller/PostDeleteTest.java` (클래스 전체 `@Transactional` 쓰지 않음): 내 글 삭제 `204`, 남의 글 `404`이고 그대로(S-8의 8), 테스트용 `@EventListener`가 `PostDeletingEvent`에서 예외를 던지면 글이 그대로 남음(S-8의 7, 한 묶음). `002` US3 뒤에는 탈퇴하면 그 회원의 글이 모두 지워지는지도 본다(`002` S-8, T036) (FR-021, FR-022, SC-004, SC-007)

### Implementation for User Story 4

- [ ] T032 [P] [US4] 이벤트 `BE/post/PostDeletingEvent.java`(`record(Long postId)`, `post` 맨 위 패키지). 주석에 "`005`의 댓글·좋아요·태그 연결·이미지 기록·글 신고·댓글 신고(D-7)는 이것을 `@EventListener`로 **같은 트랜잭션 안에서** 듣고 자기 표를 지운다. 이미지 파일은 트랜잭션 뒤에 지운다(research R-1)"를 적는다. 지금은 듣는 쪽이 없다 (FR-022, data-model 5)
- [ ] T033 [US4] `BE/post/domain/Post.java`에 `update(categoryId, topicId, title, content, visibility, now)`: 정리한 값을 지금 값과 하나씩 비교해 **하나라도 다를 때만** 바꾸고 `updatedAt = now`(`Clock`), 바뀌었는지를 돌려준다. `createdAt`은 바꾸는 방법이 없다 (research B-7, R-2, FR-020, SC-009)
- [ ] T034 [US4] `BE/post/service/PostEditService.java`: `editView`·`update`·`delete` 모두 **먼저 주인을 확인**하고 아니면 `POST_NOT_FOUND`(contracts `요청 검사 순서`). 그래서 수정 요청 본문은 컨트롤러의 `@Valid`가 아니라 주인 확인 **뒤에** `jakarta.validation.Validator`로 검사한다. 분류·주제 확인은 T022와 같다. 삭제는 `PostDeletingEvent` 발행 → 글 삭제를 한 트랜잭션에서 (FR-019 ~ FR-022, FR-041, FR-044) — T032, T033 다음
- [ ] T035 [US4] `PostController`에 `GET /api/posts/{postId}/edit`, `PUT /api/posts/{postId}`(`{ postId, changed, updatedAt }`), `DELETE /api/posts/{postId}`(`204`) (contracts 12 ~ 14)
- [ ] T036 [US4] 탈퇴 정리: `BE/post/service/PostBlogClosingCleaner.java` — `002` T032의 `BlogClosingEvent`를 `@EventListener`로 받아 그 블로그의 모든 글마다 `PostDeletingEvent`를 내고 글을 지운다(분류보다 **먼저**, 같은 트랜잭션). `002` T035에 적어 둔 약속을 채우는 것이다 (`002` FR-024) — `002` T032 merge 뒤
- [ ] T037 [US4] 화면: `FE/post/PostEditorPage.tsx`에 수정 모드(`/write/:postId`, `GET …/edit`로 채움, 바뀐 것이 없으면 저장 버튼 잠금, `requestKey` 없음), 상세의 `삭제` → `<dialog>` "삭제하면 되돌릴 수 없습니다. 삭제할까요?", **취소하면 요청을 보내지 않음**, 성공하면 내 블로그(`/blog/:blogId`)로 (FR-019 ~ FR-022)

**Checkpoint**: T030, T031이 통과하고 S-7, S-8(1 ~ 2, 7 ~ 8)을 화면으로 확인한다. S-8의 3 ~ 6은 `005` 뒤에 (PR 하나)

---

## Phase 7: User Story 5 - 글마다 공개·비공개 정하기 (Priority: P2)

**Goal**: 비공개 글은 주인만 본다. 공개로 바꿀 때는 먼저 묻는다

**Independent Test**: quickstart S-9, S-9a(글 쪽)

> 규칙 자체는 T010(`PostVisibility`)과 T027에서 이미 만들었다. 이 단계는 남은 화면과 넓은 확인이다.

- [ ] T038 [P] [US5] `BE-TEST/post/controller/PostVisibilityTest.java`: S-9의 1·2·10(비공개 글을 주인은 읽고, 공개 → 비공개 하면 바로 남에게 `404`), S-9a의 1·6 ~ 8(분류를 비공개로 해도 글의 `visibility`는 그대로, `미분류`도 비공개 가능, 다시 공개하면 보임, 공개 분류로 옮긴 글만 보임), 이전·다음·분류 개수에 비공개가 섞이지 않음 (FR-028 ~ FR-031, FR-048, SC-002, SC-013)
- [ ] T039 [US5] 화면: 수정 모드에서 비공개 → 공개로 바꿔 저장할 때 `<dialog>` "공개로 바꾸면 누구나 볼 수 있습니다", 취소하면 요청을 보내지 않고 계속 비공개. 주인이 보는 상세에 "비공개" 표시 (FR-032, FR-033)
- [ ] T040 [US5] `004`로 넘길 약속을 적어 둔다: `PostVisibility`의 주석과 `004` 작업 목록을 만들 때 "목록·검색은 이 조건을 그대로 쓰고, 검색은 주인이어도 비공개를 넣지 않으며(contracts 15), 주인의 목록에는 `visibility`를 담는다(FR-032)"를 확인한다 (FR-030, FR-032)

**Checkpoint**: T038이 통과하고 S-9(목록·검색 줄은 `004` 뒤), S-9a(1, 4, 6 ~ 8)를 확인한다 (PR 하나, US4와 묶어도 된다)

---

## Phase 8: User Story 6 - 분류 관리 (Priority: P2)

**Goal**: 주인이 분류를 추가·이름 바꾸기·공개 여부 바꾸기·순서 바꾸기·삭제한다. 글이 있는 분류와 `미분류`는 지울 수 없다

**Independent Test**: quickstart S-10, S-9a(2, 3, 9)

### Tests for User Story 6

- [ ] T041 [P] [US6] `BE-TEST/blog/controller/CategoryManageTest.java` (동시 요청 줄은 `@Transactional` 없이): S-10의 1 ~ 15(맨 아래 추가, ` 일상 `·`daily` 중복, 동시 추가 하나만, 다른 블로그와는 같은 이름 허용, 이름 바꾸면 상세에도 반영, 순서, 번호 빠지면 `INVALID_CATEGORY_ORDER`, 글 2개(공개 1·비공개 1)면 "글이 2개 있어", 옮긴 뒤 삭제, `미분류` 삭제 거절·이름 바꾸기 허용, 개수 B 1·A 2, 남의 분류 `404`, 삭제와 글쓰기 동시 → 분류 없는 글 0), S-9a의 2·3·9 (FR-034 ~ FR-043, FR-047, SC-004 ~ SC-006, SC-010)

### Implementation for User Story 6

- [ ] T042 [P] [US6] `BE/blog/validation/ValidCategoryName.java` + 검사기(앞뒤 공백 제거 뒤 1 ~ `category.name.max-length`, 코드 포인트), `BE/blog/validation/BlogFieldErrorMessages.java`(분류·블로그 문구의 숫자를 설정에서, T048도 이 파일에 더한다) (FR-036)
- [ ] T043 [US6] `BE/blog/domain/Category.java`에 `create(blogId, name, sortOrder, visibility)`, `rename(name)`, `changeVisibility(visibility)`, `moveTo(sortOrder)`를 더한다. `미분류`는 `isDefaultCategory()`로 알아본다(이름이 아님) (FR-037, FR-042, FR-047, D-5)
- [ ] T044 [US6] `BE/blog/service/CategoryService.java`: 추가(가장 큰 `sort_order` + 1), 이름 중복은 소문자로 먼저 보고 **DB의 `uq_category_blog_name`이 마지막 방어선** → `CATEGORY_NAME_DUPLICATED`, 자기 자신과는 비교하지 않음. 순서 바꾸기는 내 분류 번호가 빠짐·겹침 없이 다 있어야 하고 1, 2, 3…으로 다시 매김. 삭제는 남의 분류 → `CATEGORY_NOT_FOUND`, 기본 분류 → `DEFAULT_CATEGORY_NOT_DELETABLE`, `CategoryPostCounter.countAll`이 1 이상 → `CATEGORY_HAS_POSTS`("글이 {N}개"), 세고 지우는 사이 글이 들어와 외래 키가 거절하면 다시 세어 같은 오류 (research R-3). 내 블로그는 세션으로만 (FR-035 ~ FR-042, FR-047)
- [ ] T045 [US6] `BE/blog/controller/CategoryController.java`: `POST /api/me/blog/categories`(`201`), `PATCH …/{categoryId}`(보낸 칸만), `PUT …/order`, `DELETE …/{categoryId}`(`204`). PATCH도 T034처럼 **내 분류인지 먼저** 보고 그 뒤에 검사한다 (contracts 5 ~ 8, FR-044)
- [ ] T046 [US6] 분류 관리 화면 `FE/blog/CategoryManagePage.tsx`(`/manage/categories`): 목록(주인 기준 개수, "비공개" 표시), 추가, 이름 바꾸기, 공개/비공개, 위·아래로 순서 바꾸기, `미분류`에는 삭제 버튼 없음, 거절 문구는 서버 것 그대로. `006`의 관리 화면(BM-04)에 들어갈 수 있게 컴포넌트로 만든다 (FR-035 ~ FR-043, FR-047)

**Checkpoint**: T041이 통과하고 S-10, S-9a(2, 3, 9)를 화면으로 확인한다 (PR 하나)

---

## Phase 9: User Story 7 - 블로그 이름과 소개 고치기 (Priority: P3)

**Goal**: 주인이 블로그 이름과 소개를 바꾸고, 모든 화면에 바로 반영된다. 블로그 삭제 기능은 없다

**Independent Test**: quickstart S-11

- [ ] T047 [P] [US7] `BE-TEST/blog/controller/BlogSettingsTest.java`: S-11의 1 ~ 7(바꾼 이름이 블로그·글 상세에 바로, 앞뒤 공백 제거, 빈 이름·공백 이름 거절, 31자·201자 거절, 빈 소개 허용, B의 요청은 B의 블로그만, 삭제 주소 없음) (FR-004 ~ FR-007, SC-004, SC-010)
- [ ] T048 [P] [US7] `@ValidBlogName`(앞뒤 공백 제거 뒤 1 ~ 30), `@ValidBlogIntro`(앞뒤 공백 제거 뒤 0 ~ 200, research B-4 가안 해석) — `BE/blog/validation/`, 문구는 T042의 `BlogFieldErrorMessages`에. DB 칸은 500이지만 서버가 200으로 막는다 (FR-004, FR-005)
- [ ] T049 [US7] `BE/blog/domain/Blog.java`에 `changeProfile(name, intro)`(빈 소개는 하나로 정해 저장, 응답은 `""`, data-model 1), `BE/blog/service/BlogSettingsService.java`, `BlogController`에 `PUT /api/me/blog`. 블로그 삭제 주소는 만들지 않는다 (contracts 4, FR-004 ~ FR-007)
- [ ] T050 [US7] 블로그 설정 화면 `FE/blog/BlogSettingsPage.tsx`(`/manage/blog`): 이름·소개, 바뀐 것이 없으면 저장 잠금, 칸별 오류, 저장 뒤 블로그 화면에 새 값. 삭제 버튼 없음 (FR-004 ~ FR-007)

**Checkpoint**: T047이 통과하고 S-11을 화면으로 확인한다 (PR 하나)

---

## Phase 10: Polish & Cross-Cutting Concerns

- [x] T051 [P] 결정·구현에 맞게 문서를 같이 고친다 (헌법 `작업 흐름`): plan `Technical Context`(버전·테스트 `미정` → `001`·`002`에서 쓰는 것, Redis 줄은 헌법 1.1.0 문구로), `Constitution Check`의 "결정 대기 7건", FR-011·FR-018·FR-045 줄의 `미정`(D-1·D-6은 결정됨), contracts 9·10의 "`D-6`에서 A를 고르면", 11의 "`D-1`이 정해지기 전에는", research B-8의 "`005` 모듈의 입구를 부른다"(→ 이벤트, T032), data-model 1의 "`002`에서 정한다"(→ 지운다), data-model 9(→ `ERD-변경-요청.md`의 T-1 ~ T-3, E-4 기준), 주소·이벤트·설정 이름이 가안에서 바뀌었으면 그것도. research B-4·data-model 6의 "공백만인 본문은 `D-2`"(→ 결정 내용)
  - **2026-10-08 정리함**: plan(버전·테스트·Redis·Constitution Check·FR-011/018/022/045·D-8), research(B-1 가입 이벤트, B-4, B-8 이벤트), data-model(1 탈퇴 때 실제 삭제, 3 `request_key` E-4·`content`, 6, 9 머리말), contracts(9·10 `requestKey`, `POST_ALREADY_SAVED` 추가, 11 마크다운, 8 `postCount` 칸 없음, 숫자 자리 글자 404/400), quickstart S-12의 4
- [x] T052 [P] research B-10의 보안 헤더(Content-Security-Policy) 검토: 선택지와 추천을 research에 적고 사용자 확인을 받는다. 이미지 주소는 `005`와 함께 정한다
  - **2026-10-08**: research `D-9` — A(지금은 걸지 않고 배포 때 화면을 내려주는 곳에), 정책 초안 적음. 사용자 "추천대로"
- [x] T053 [P] quickstart S-12(제목·본문·이름의 `<script>`, `<img onerror>`, `javascript:` 링크, 마크다운 문법, SQL 같은 글자, CSRF, 이상한 JSON)를 화면에서 실행한다 (FR-045, SC-012)
- [x] T054 quickstart S-13: 한 블로그에 글 1,000개를 넣고 상세(이전·다음 포함)와 분류 목록이 2초 안인지 본다. 느리면 T003의 인덱스부터 확인한다 (NF-09)
  - **2026-10-08 (클라우드 세션, Docker PostgreSQL)**: 글 1,003개(비공개 200개 섞음). 상세(이전·다음 포함) 첫 요청 0.52초(서버 예열), 이후 0.03~0.04초. 분류 목록 0.03~0.09초. 이전 글 쿼리는 이 크기에서 순차 읽기 0.36ms라 인덱스가 아직 필요 없다
- [ ] T055 SC-011(제안값 3분): **사람이** 글쓰기 화면을 열어 저장한 글의 상세를 볼 때까지 시간을 잰다 (S-2의 6)
- [ ] T056 quickstart 전체를 실행하고 끝의 `구현 뒤에 채울 것`을 채운다. `tasks.md` 체크박스와 `CLAUDE.md`의 `6. 지금 상태`를 고친다

---

## 요구사항 → 작업 연결표

| FR | 작업 |
|---|---|
| FR-001 | T013, T015 |
| FR-002, 003 | T013 (`BlogProvisioner`는 이미 있음) |
| FR-004 | T047, T048, T049, T050 |
| FR-005 | T005, T047, T048, T049 |
| FR-006 | T007, T015, T047, T049, T050 |
| FR-007 | T047, T049, T050 |
| FR-008 | T007, T008, T017, T022, T024 |
| FR-009 | T017, T022, T024 |
| FR-010 | T017, T019, T020 |
| FR-011 | T019, T020, T025, T029 (세는 법은 D-2: 원문 기준) |
| FR-012 | T017, T021, T024 |
| FR-013 | T017, T020, T021 |
| FR-014 | T017, T020, T023 |
| FR-015 | T023, T024 |
| FR-016 | T006, T017, T019, T024 |
| FR-017 | T012, T024 |
| FR-018 | T003, T018, T022, T024 |
| FR-019 | T030, T034, T035, T037 |
| FR-020 | T030, T033, T034 |
| FR-021 | T031, T034, T035, T037 |
| FR-022 | T031, T032, T034, T036, T037 (댓글 등은 `005`) |
| FR-023 | T026, T027, T029 |
| FR-024 | T026, T027, T029 |
| FR-025 | T028, T029 |
| FR-026 | T006, T010, T026, T027 |
| FR-027 | T026, T029 |
| FR-028, 029 | T003, T017, T038 |
| FR-030 | T010, T038, T040 |
| FR-031 | T010, T026, T038 |
| FR-032 | T039, T040 |
| FR-033 | T030, T039 |
| FR-034 | T003, T009, T022, T044 |
| FR-035 | T041, T044, T045, T046 |
| FR-036 | T041, T042, T044 |
| FR-037 | T041, T043, T044 |
| FR-038 | T014, T027, T041 |
| FR-039 | T014, T041, T044, T046 |
| FR-040 | T009, T041, T044 |
| FR-041 | T030, T034 |
| FR-042 | T041, T043, T044, T046 |
| FR-043 | T010, T014, T041 |
| FR-044 | T007, T008, T017, T030, T031, T034, T041, T045, T047 |
| FR-045 | T025, T029, T053 |
| FR-046 | T003, T017, T021, T022, T024 |
| FR-047 | T041, T043, T044, T046 |
| FR-048 | T010, T014, T026, T038 |

| SC | 확인하는 작업 |
|---|---|
| SC-001 | T013 |
| SC-002 | T026, T038 |
| SC-003 | T026 |
| SC-004 | T030, T031, T041, T047 |
| SC-005 | T003, T041 |
| SC-006 | T041 |
| SC-007 | T031 (글까지. 댓글·좋아요·태그·이미지는 `005` 뒤 S-8) |
| SC-008 | T018 |
| SC-009 | T030 |
| SC-010 | T014, T041, T047 |
| SC-011 | T055 |
| SC-012 | T029, T053 |
| SC-013 | T017, T026, T038 |

---

## Dependencies & Execution Order

### Phase Dependencies

- **Setup (Phase 1)**: 확인만 한다
- **Foundational (Phase 2)**: Setup 다음. **모든 사용자 이야기를 막는다**
- **US1 (P1)**: Foundational 다음
- **US2 (P1)**: Foundational 다음
- **US3 (P1)**: US2 다음 (읽을 글이 있어야 한다). 마크다운 그리기는 T025 결정 뒤
- **US4 (P2)**: US3 다음. T036은 `002` T032 merge 뒤
- **US5 (P2)**: US4 다음 (수정 화면에 확인 창을 더한다)
- **US6 (P2)**: US1 다음이면 된다 (분류 삭제 거절 확인에는 US2의 글쓰기가 필요)
- **US7 (P3)**: US1 다음이면 된다
- **Polish**: 원하는 이야기가 끝난 뒤

### Within Each User Story

- 테스트 → 도메인 → 서비스 → 주소(컨트롤러) → 화면 순서
- 테스트는 먼저 써 두고 실패하는 것을 본 뒤 구현한다. push 전에 `mvn verify`와 화면 `lint`·`build`를 돌린다
- 한 이야기를 끝내고 Checkpoint를 통과한 뒤 PR을 merge하고 다음으로 간다

### Parallel Opportunities

- T003, T005, T006, T007은 동시에. 화면의 T011, T012도 동시에
- US2의 T017, T018, T019는 동시에
- US4의 T030, T031, T032는 동시에
- US6(분류)과 US7(블로그 설정)은 서로 기대지 않는다. 단 `BlogFieldErrorMessages`(T042, T048)를 같이 고치므로 한 사람이 하거나 차례로

---

## Parallel Example: User Story 2

```text
먼저 동시에:
Task: "T017 PostCreateTest (S-2, S-2a, S-3, S-4)"
Task: "T018 PostRequestKeyTest (S-5, 동시 요청)"
Task: "T019 @ValidTitle, @ValidContent (본문은 원문 기준, 공백만이면 비어 있음)"
그다음: T020 → T021 → T022 → T023 → T024
```

---

## Implementation Strategy

### MVP First (US1 ~ US3)

1. Phase 1, 2를 끝낸다 (US1 PR에 같이 넣는다)
2. US1(내 블로그) → US2(글쓰기) → US3(글 읽기)
3. **멈추고 확인**: quickstart S-1 ~ S-6
4. 보여 줄 수 있으면 보여 준다

### Incremental Delivery

1. US4(수정·삭제) + US5(공개 범위) → 확인
2. US6(분류 관리) → 확인
3. US7(블로그 이름·소개) → 확인
4. `004`를 만들 때 T040의 약속대로 목록·검색에 `PostVisibility`를 쓰고 S-9(3, 4), S-9a(5)를 다시 확인한다
5. `005`를 만들 때 `PostDeletingEvent`를 듣는 쪽을 더하고 S-8 전체를 다시 확인한다

---

## Notes

- 작업 하나 또는 묶음 하나를 끝낼 때마다 커밋하고, 커밋 메시지에 작업 ID를 적는다 (예: `003 T022`). PR은 이야기 하나에 하나
- `가안`인 주소(`/api/me/blog/…`, `/write…`, `/manage…`), 오류 이름, 설정 이름, 입구·이벤트 이름(`LoggedInMember`, `BlogDirectory`, `CategoryPostCounter`, `PostDeletingEvent`)을 바꾸면 T051처럼 문서도 같이 고친다
- 남은 결정: **마크다운 표시 도구 이름**(T025). D-1 ~ D-7은 모두 결정됨. 확인할 위험: research R-1(이미지 파일, `005`), R-2(수정 시각, T033), R-3(분류 삭제와 글쓰기 동시, T044), R-6(비공개 글이 다른 길로 드러남, T026)
