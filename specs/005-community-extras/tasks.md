---

description: "005 소통과 부가 기능 (댓글·좋아요·태그·신고·이미지) 작업 목록"
---

# Tasks: 소통과 부가 기능 (댓글·좋아요·태그·신고·이미지)

**Input**: `/specs/005-community-extras/`의 설계 문서

**Prerequisites**: [plan.md](plan.md), [spec.md](spec.md), [research.md](research.md), [data-model.md](data-model.md), [contracts/community-api.md](contracts/community-api.md), [quickstart.md](quickstart.md)

**결정 반영 (2026-10-08)**: research의 `D-1 ~ D-11`은 **모두 정해졌다.** 7개는 2026-10-07에, 남은 4개(`D-1`, `D-5`, `D-6`, `D-10`)와 D-3의 정리 시간은 2026-10-08 밤에 사용자가 추천대로 정했다 ("추천대로 진행해"). 정리 시간 24시간과 "태그만 바꾼 수정도 수정됨"(T044)은 `가안`이라 사용자가 아침에 다시 확인한다. 태그·이미지 단계는 처음 순서대로 뒤에 둔다.

| ID | 결정 | 상태 | 이 목록에서 |
|---|---|---|---|
| D-1 | 이미지 파일 저장 위치: **개발은 MinIO**, 저장 코드를 `ImageStorage`로 감싸 서버 디스크로 바꿔 끼울 수 있게. 배포 때 어디에 둘지는 배포 환경(`001` D-5)과 함께 (그래서 헌법의 `미정` 목록에는 남는다) | 결정됨 (2026-10-08, 추천대로) | T002, T049, T051 |
| D-2 | 댓글 5초 간격은 DB의 마지막 댓글 시각으로 세고, 동시 요청은 회원별 잠금으로 막는다. Redis 안 씀 | 결정됨 | T014, T015, T018 |
| D-3 | 이미지는 먼저 올려 "아직 글 없음"으로 기록하고, 글을 저장할 때 본문에 남은 것만 연결한다. 오래 연결되지 않은 것은 `@Scheduled`가 지운다. `post_image`는 팀 ERD 요청 T-1 모양으로 만든다 | 결정됨 | T052, T055, T057, T058 |
| (값) | D-3의 **"몇 시간 뒤에 지울지"**: **24시간** (`가안`, 사용자 아침 확인). `상세/05` `기본값` 표에 먼저 적었다 | 결정됨 (2026-10-08, 가안) | T002, T049, T053, T058 |
| D-4 | 본문 안 이미지는 마크다운 이미지 문법, 주소는 **우리 서버 주소만** (`003` D-1과 같음) | 결정됨 | T057, T059 |
| D-5 | 글 삭제 때 DB와 파일이 어긋나면: **A. DB 먼저 지우고, 끝난 뒤 파일 삭제. 실패한 파일은 정리 작업이 다시 지운다** | 결정됨 (2026-10-08, 추천대로) | T002, T049, T058 |
| D-6 | 이미지 형식 확인: **B. 파일 앞부분의 형식 표시(시그니처)** | 결정됨 (2026-10-08, 추천대로) | T002, T049, T054 |
| D-7 | 팀 ERD `comment`의 `UNIQUE(users_id, post_id)`는 없앤다(팀 ERD 최신판에서 이미 지워짐). `parent_id`, `is_secret`은 두되 쓰지 않는다 | 결정됨 | T015, T016 |
| D-8 | 팀 ERD `comment_report`는 이번에 쓰지 않는다 (표도 만들지 않음) | 결정됨 | T015, T026 |
| D-9 | 탈퇴한 작성자는 회원 줄의 `deleted_at`으로 판단해 "탈퇴한 사용자"로 보여 준다 (`002` D-1과 같음) | 결정됨 | T008, T018, T023 |
| D-10 | 태그 저장 모양: **A. 소문자로 바꿔 저장** (팀 ERD `tag.name`의 `UNIQUE` 그대로 대소문자 무시가 됨) | 결정됨 (2026-10-08, 추천대로) | T002, T041, T042, T043 |
| D-11 | 글을 지우면 그 글의 신고 기록도 함께 지운다 (`003` D-7과 같음) | 결정됨 | T034, T038 |
| `002` D-1, D-5 | 탈퇴해도 남의 글에 단 댓글은 남고 "탈퇴한 사용자"로 보인다. **내가 한 신고는 남긴다.** 내가 누른 좋아요는 지운다 (`MemberWithdrawnEvent` 주석) | 결정됨 | T023, T026, T032, T038 |
| (도구) | 본문 마크다운을 그리는 화면 도구는 `003` research D-8에서 **react-markdown + remark-gfm**으로 정했다(2026-10-08, `003` T025). `003` T025가 merge되기 전에는 본문 안 이미지가 그림으로 보이지 않는다 | 결정됨 (`003`의 일) | T059 |

**Tests**: `001`·`002`·`003`처럼 **사용자 이야기마다 서버 테스트 작업**을 넣었다 (`@SpringBootTest` + MockMvc + `springSecurity()`, 실제 PostgreSQL). 테스트 이름에는 [quickstart.md](quickstart.md)의 시나리오 번호를 붙인다. 동시 요청을 보는 테스트는 클래스 전체에 `@Transactional`을 걸지 않는다 (`002` T021과 같은 이유). 화면에는 테스트 도구가 없어서(`frontend/package.json`) 각 단계 끝의 `Checkpoint`에서 손으로(또는 Playwright로) 확인한다.

**Organization**: 사용자 이야기(US)별로 묶었다. PR도 이야기 하나에 하나 (CLAUDE.md `커밋과 올리기`). 코드는 `home-blog/blog`, 이 목록은 `home-blog/docs`에 있다. **단계 순서는 명세의 우선순위와 조금 다르다**: 정해지지 않은 것이 없던 이야기(US1 → US2 → US3 → US6)를 먼저 하고, 결정을 기다리던 US5(태그, `D-10`)와 US4(이미지, `D-1`·`D-5`·`D-6`)를 뒤로 미뤘다. 2026-10-08에 모두 정해졌지만 순서는 그대로 둔다.

## Format: `[ID] [P?] [Story] 설명`

- **[P]**: 동시에 해도 되는 작업 (다른 파일이고, 끝나지 않은 작업에 기대지 않음)
- **[Story]**: 어느 사용자 이야기의 작업인지 (US1 ~ US6)

## 경로 약속

> `002`, `003`과 같다. 코드는 `home-blog/blog` 저장소에 있다.

| 줄임 | 실제 경로 |
|---|---|
| `BE/` | `backend/src/main/java/com/myblog/` (서버, Spring Boot) |
| `BE-RES/` | `backend/src/main/resources/` |
| `BE-TEST/` | `backend/src/test/java/com/myblog/` (서버 테스트) |
| `FE/` | `frontend/src/` (화면, React + Vite) |

## 쉬운 설명: 이미 있는 것과 새로 만드는 것

| 이미 있는 것 (`001`, `002`, 그리고 만드는 중인 `003`) | 이 기능에서 |
|---|---|
| `users` 표의 `deleted_at` (`V2__auth_tables.sql`). 탈퇴하면 회원 줄은 남고 닉네임은 `탈퇴한사용자{id}`로 바뀐다 (`002` D-1, D-2, `User.withdraw`) | 댓글 작성자가 탈퇴했는지는 `deleted_at`으로 본다 (D-9). 바뀐 닉네임을 그대로 보여 주지 않고 "탈퇴한 사용자"로 바꾼다 (T008, T018) |
| `MemberWithdrawnEvent` (`user` 맨 위, `002` T030). 주석: "005 comment 모듈이 내가 누른 좋아요를 지운다. 내가 한 신고는 남긴다(D-5). 남는 댓글은 탈퇴한 사용자로(D-1)" | 좋아요를 지우는 듣는 쪽을 만든다 (T032). 댓글과 신고는 **듣지 않는다** (남기는 것이 규칙이므로) (T026, T038) |
| `BlogClosingEvent` (`blog` 맨 위, `002` T032). `003` T036의 `PostBlogClosingCleaner`가 듣고 **그 블로그의 글마다 `PostDeletingEvent`를 낸다** | 이 기능은 `BlogClosingEvent`를 따로 듣지 않고 `PostDeletingEvent`만 듣는다. 그래서 글 하나를 지울 때와 탈퇴할 때가 **같은 길**로 지워진다 (T023, T026). 이벤트 주석을 실제에 맞게 고친다 (T026) |
| `003`이 만드는 것: `post`, `topic` 표(`V3`), `Post` 엔티티, `PostVisibility`(볼 수 있는 글 조건), `BlogDirectory`, `LoggedInMember`(`user` 맨 위, 로그인한 회원 번호), `PostDeletingEvent`(`post` 맨 위), `PostWriteService`·`PostEditService`·`PostReadService`, `PostDetailPage`·`PostEditorPage` | 다시 만들지 않는다. `post` 맨 위에 **입구 몇 개를 더하고**(T009, T010, T057) 글 상세·글쓰기 화면에 영역을 붙인다 |
| `ErrorCode`, `ApiException`, `GlobalExceptionHandler`, `FieldErrorMessages`(설정값으로 문구 숫자 채우기) | 새 오류 이름, `CommentFieldErrorMessages` 등, 큰 파일 오류(`MaxUploadSizeExceededException`) 처리를 더한다 |
| `AccountProperties`(`@ConfigurationProperties` record, DB 칸보다 크면 서버가 켜지지 않음, `@ConfigurationPropertiesScan`) | 같은 모양으로 `CommentProperties`, `ReportProperties`, `TagProperties`, `ImageProperties` (`community.*`, plan `설정값 목록`) |
| `CurrentMemberService`(세션 회원을 DB에서 다시 확인) → `003` T007의 `LoggedInMember`로 다른 모듈에 알려 줌 | 모든 컨트롤러가 `LoggedInMember`로만 회원 번호를 얻는다. 주소나 본문에서 받지 않는다 |
| `createBrowserRouter`(`FE/App.tsx`), `RequireLogin`, `useLoginPrompt`(`FE/auth/loginPrompt.ts`, 로그인 창 띄우기), `api/client.ts`의 `ApiError`, `index.css`의 원고지 토큰 | 댓글·좋아요·신고 영역은 글 상세에, 태그 입력·이미지 올리기는 글쓰기 화면에 붙인다. 새 화면 주소는 `/tags/:tagName` 하나 (가안) |
| `docker-compose.yml`에는 PostgreSQL, Redis만 있다 | 개발용 MinIO를 T051에서 더한다 (`D-1`) |

**모듈 방향** (Spring Modulith, `ModularityTest`): `user ← blog ← post ← comment/community/image`. 이 목록은 코드 저장소 CLAUDE.md의 모듈 이름을 따라 **댓글은 `comment`, 좋아요·신고는 `community`, 이미지는 `image`** 에 두고, **태그는 `post` 안에** 둔다 (2026-10-08 결정. 태그는 글 저장과 한 묶음이고 태그별 목록은 글 목록이기 때문. plan `Project Structure`도 이대로 고쳤다). 아래 모듈(`post`)은 위 모듈(`comment` 등)을 부르지 않는다. 그래서 글 상세의 댓글 수·좋아요 수는 `post`가 **질문 틀**을 두고 위 모듈이 채우며(T010, `003`의 `CategoryPostCounter`와 같은 방식), 글 삭제와 글 저장은 `post`가 **이벤트를 내고** 위 모듈이 듣는다(`PostDeletingEvent`, T057의 이벤트). 모든 듣는 쪽은 `@EventListener`로 **같은 트랜잭션 안에서** 돈다. 이미지 **파일** 지우기만 트랜잭션이 끝난 뒤다 (`D-5`).

---

## Phase 1: Setup (공통 준비)

**Purpose**: 먼저 merge되어 있어야 하는 것, 사용자에게 물어야 하는 것, 문서에 먼저 적어야 하는 것

- [ ] T001 시작 전 확인: **`003`의 아래 작업이 `main`에 merge되어 있고 CI가 통과하는지** 본다 (`003`은 다른 세션이 만드는 중). 이야기마다 필요한 것이 다르므로 그 이야기를 시작할 때 다시 본다. 브랜치는 이야기마다 (`feat/005-us1-comment`, `feat/005-us2-comment-delete`, `feat/005-us3-like`, `feat/005-us6-report`, `feat/005-us5-tag`, `feat/005-us4-image`, 가안)
  - **모든 이야기**: `003` Phase 2 **T003 ~ T012** (`V3`의 `post`·`topic` 표, `post` 모듈과 `Post`·`PostRepository`, `ErrorCode.POST_NOT_FOUND`, `LoggedInMember`, `SecurityConfig`의 읽기 주소, `BlogDirectory`, `PostVisibility`, `postApi.ts`, `/posts/:postId` 화면 주소)
  - **US1, US3, US6** (글 상세에 붙음): `003` US3 **T026 ~ T029** (`PostReadService`, `GET /api/posts/{postId}`, `PostDetailPage`)
  - **US2, US3, US6, US5, US4** (글 삭제 때 함께 지움): `003` US4 **T030 ~ T037**. 특히 **T032**(`PostDeletingEvent`), **T034**(`PostEditService.delete`가 이벤트 발행 → 글 삭제를 한 트랜잭션에서), **T036**(`PostBlogClosingCleaner`, 탈퇴 때 글마다 `PostDeletingEvent`)
  - **US5, US4** (글 저장·수정에 붙음): `003` US2 **T017 ~ T024** (`PostWriteService.create`, `PostRequests`, `PostEditorPage`)와 US4 **T033 ~ T035, T037** (`PostEditService.update`, 수정 화면)
  - **US4의 본문 안 이미지 보이기**: `003` **T025**(마크다운 표시 도구, `003` research D-8: react-markdown + remark-gfm)와 **T029**의 `MarkdownView`
  - `002`의 `MemberWithdrawnEvent`(T030), `BlogClosingEvent`(T032)는 이미 merge됨 (PR #19)
- [x] T002 **미정 항목을 사용자에게 묻는 때를 정해 둔다** (헌법 II. 2026-10-08 끝: 사용자가 모두 추천대로 골랐고 research·plan·`상세/05`·`상세/07`·기술스택 2.6을 맞췄다. 헌법의 `미정` 목록은 배포 위치가 남아 그대로): ① US5(Phase 7)를 시작하기 전에 `D-10` ② US4(Phase 8)를 시작하기 전에 `D-1`, `D-5`, `D-6`, D-3의 **정리 시간 숫자**. 물을 때는 research의 선택지·추천·이유를 그대로 보여 준다. 고르면 research D-항목과 E 표, plan `정해야 할 것 요약`과 FR 연결표의 해당 줄을 같이 고친다. `D-1`을 고르면 헌법의 `미정` 목록, `상세/05`·`상세/07` 구현 방식, `기술스택-아키텍처.md` 2.6도 같이 맞춘다 (research D-1 `영향`)
- [ ] T003 [P] ERD 문서 맞추기 (CLAUDE.md `ERD를 고칠 때`): `post_report`의 `UNIQUE(users_id, post_id)`(FR-019, SC-003, data-model 4)는 2026-10-08 사용자 결정으로 `docs/3-설계/ERD-변경-요청.md`에 내 확장 **E-7**로 더했다. 남은 것: Crowfoot `myblog-제안`(666)에 E-7 반영. `post_image.storage_key` 중복 불가(data-model 5)도 같이 묻는다. 팀 요청 T-1(`post_image`)과 T-3(`reason` CHECK)은 팀 답이 없어도 `003`이 T-2·T-3을 그랬듯 내 코드에는 먼저 넣는다 (T035, T052)
- [ ] T004 [P] `※` 문구를 `docs/2-요구사항/상세/05-소통-부가.md`의 `안내 문구` 표에 먼저 올린다 (**사용자 확인 뒤**, contracts 머리말: "화면을 만들기 전에 먼저 추가"): 댓글 빈 칸·500자 초과·5초 제한, 댓글 삭제 권한 없음·없는 댓글, 자기 글 좋아요·신고, 신고 사유 없음·설명 200자 초과, 태그 5개 초과·형식·중복, 이미지 10장 초과, 저장소 연결 실패. 문장은 "~합니다 / ~해 주세요" (spec Assumptions)

---

## Phase 2: Foundational (모든 이야기의 바탕)

**Purpose**: US1 ~ US6이 함께 기대는 모듈, 설정, 오류, 입구

**⚠️ CRITICAL**: 이 단계가 끝나기 전에는 사용자 이야기 작업을 시작하지 않는다. `003`의 T003 ~ T012가 먼저 merge되어 있어야 한다 (T001)

- [x] T005 [P] (comment PR #29, community PR #30) 모듈 뼈대: `BE/comment/package-info.java`(`@ApplicationModule(displayName = "댓글", allowedDependencies = {"post", "user", "common"})`), `BE/community/package-info.java`(`displayName = "좋아요·신고"`, 같은 의존). `image` 모듈은 US4에서 만든다 (T050)
- [x] T006 [P] (CommentProperties PR #29, ReportProperties PR #31) 설정값 묶음 (plan `설정값 목록`, 헌법 VI): `BE/comment/config/CommentProperties.java`(`community.comment`: `min-length` 1, `max-length` 500, `min-interval` 5s), `BE/community/config/ReportProperties.java`(`community.report.detail-max-length` 200)를 `AccountProperties`처럼 record로 만들고 `BE-RES/application.yml`에 더한다. 최대값이 DB 칸(`comment.body` 500, `post_report.detail` 200)보다 크면 서버가 켜지지 않게 한다. 태그·이미지 설정은 그 이야기에서 더한다 (T043, T053)
- [ ] T007 [P] (댓글 오류는 PR #29, 나머지는 각 이야기 PR) `BE/common/error/ErrorCode.java`에 이 기능의 오류를 더한다 (contracts 문구 그대로, `※`는 제안 문구, T004에서 확정되면 바꿈): `COMMENT_EMPTY`("※ 댓글 내용을 입력해 주세요"), `COMMENT_TOO_LONG`(숫자는 설정값에서), `COMMENT_TOO_FREQUENT`(429), `COMMENT_NOT_FOUND`(404, 남의 댓글을 지우려 할 때도 이것), `SELF_LIKE_NOT_ALLOWED`(403), `SELF_REPORT_NOT_ALLOWED`(403), `ALREADY_REPORTED`(409 "이미 신고한 글입니다"), `REPORT_REASON_REQUIRED`, `REPORT_DETAIL_TOO_LONG`, `TAG_TOO_MANY`, `TAG_INVALID`, `TAG_DUPLICATED`, `IMAGE_LIMIT_EXCEEDED`(409), `INVALID_IMAGE`(400 "이미지는 5MB 이하의 jpg, png, gif, webp만 올릴 수 있습니다"), `STORAGE_UNAVAILABLE`(503 "※ 잠시 뒤 다시 시도해 주세요") (FR-002, FR-004, FR-008, FR-012, FR-014, FR-017 ~ FR-019, FR-024, FR-025)
- [x] T008 [P] **회원 이름을 다른 모듈에 알려 주는 틀** `BE/user/MemberNames.java`(`user` 맨 위, 가안): `Map<Long, MemberName> namesOf(Collection<Long> memberIds)`, `MemberName(Long id, String nickname, boolean withdrawn)`. 탈퇴한 회원(`deleted_at` 있음)은 `withdrawn = true`이고 **번호와 닉네임을 비운다** (contracts 1, D-9, `002` D-1). `user` 안쪽(`BE/user/service/MemberNamesAdapter.java`)에서 `UserRepository`로 한 번에 읽어 채운다(댓글마다 따로 읽지 않음) (FR-003, FR-007)
- [x] T009 **볼 수 있는 글인지 알려 주는 입구** `BE/post/PostLookup.java`(`post` 맨 위, 가안): `Optional<PostRef> findVisible(Long postId, Long viewerIdOrNull)` → `PostRef(postId, blogId, ownerId)`. 볼 수 없으면 비어 있고, 부르는 쪽이 **없는 글과 같은** `POST_NOT_FOUND`로 답한다 (research B-1의 2단계, R-3, CF-09-4). `BE/post/service/PostLookupAdapter.java`가 `003` T010의 `PostVisibility`와 `BlogDirectory`(글 → 분류 → 블로그 → 주인, research B-1)로 채운다 (FR-029)
- [x] T010 **글 상세에 더할 칸의 질문 틀** (`post` 맨 위, 가안): `BE/post/PostCommentCounter.java`(`long count(postId)`, `comment`가 채움), `BE/post/PostLikeSummary.java`(`LikeSummary summary(postId, viewerIdOrNull)` → `likeCount`, `likedByMe`, `community`가 채움). `003`의 `PostReadService`와 글 상세 응답(`003` contracts 11)에 `commentCount`, `likeCount`, `likedByMe`를 더한다 (contracts 7). 채우는 쪽이 아직 없으면 0·`false` (`ObjectProvider`로 받음) (FR-003, FR-011)
- [x] T011 `BE/user/config/SecurityConfig.java`: 누구나 읽는 주소 `GET /api/posts/{postId}/comments`, `GET /api/tags/**`, `GET /api/images/**`를 `permitAll`에 더한다. 나머지 변경 요청은 지금처럼 `401` (contracts 공통, FR-001, FR-009, FR-017)
- [x] T012 [P] (commentApi·rules PR #29, communityApi PR #30) 화면 요청 함수와 규칙: `FE/comment/commentApi.ts`(contracts 1 ~ 3), `FE/community/communityApi.ts`(4 ~ 6), 응답 타입은 contracts 그대로. `FE/comment/rules.ts`에 글자 수·문구(서버 설정·contracts와 같게). 글 상세 응답 타입(`FE/post/postApi.ts`)에 T010의 칸을 더한다

**Checkpoint**: `mvn verify`(CI)와 `ModularityTest`가 통과한다(`comment`·`community` → `post` → `blog` → `user` 방향만 있음). 글 상세 응답에 `commentCount: 0`, `likeCount: 0`, `likedByMe: false`가 보인다

---

## Phase 3: User Story 1 - 댓글 쓰고 읽기 (Priority: P1) 🎯 MVP

**Goal**: 로그인한 회원이 글 상세 아래에 댓글을 쓰고, 누구나 오래된 순서로 읽는다. 로그인하지 않으면 입력칸 대신 안내와 로그인 버튼

**Independent Test**: quickstart S-1(1 ~ 6), S-2, S-3, S-12(1, 2, 4)

**먼저 merge**: `003` T003 ~ T012, T026 ~ T029

### Tests for User Story 1

- [x] T013 [P] [US1] `BE-TEST/comment/controller/CommentWriteTest.java`: S-1의 2 ~ 6(로그인 안 하면 `401`이고 줄이 안 생김, 맨 아래에 닉네임·시각·내용, 줄바꿈 그대로, 오래된 순, 같은 회원이 한 글에 두 번째 댓글도 됨(D-7)), S-2의 1 ~ 6(공백만·줄바꿈만 → `COMMENT_EMPTY`, 500자 통과, 501자 `COMMENT_TOO_LONG`, 이모지 섞인 500자 통과(코드 포인트로 셈, research B-2), 남의 비공개 글 → 없는 번호와 **상태 코드·본문이 같은** `404`), S-12의 1·2·4(`<script>`·`<img onerror>`·`' OR 1=1 --`를 **글자 그대로** 저장하고 돌려줌), CSRF 없이 쓰기 거절, 글 상세의 `commentCount` (FR-001 ~ FR-003, FR-029, FR-030, SC-001, SC-010)
- [x] T014 [P] [US1] `BE-TEST/comment/service/CommentIntervalTest.java` (**클래스 전체 `@Transactional` 쓰지 않음**, 끝에서 지움, 간격은 테스트 설정으로 줄여도 됨): S-3의 1 ~ 5 — 바로 다시 쓰면 `429 COMMENT_TOO_FREQUENT`, 간격이 지나면 성공, 다른 글이어도 막힘(사람 기준), 다른 두 회원은 둘 다 성공, **같은 회원의 요청 5개를 동시에 보내면 한 건만** (D-2, FR-008, SC-007)

### Implementation for User Story 1

- [x] T015 [US1] Flyway `BE-RES/db/migration/V{다음 번호}__comment.sql` (번호는 그때 `main`의 마지막 다음): `comment` 표를 팀 ERD 칸 그대로(`comment_id` 자동 증가, `users_id` → `users`, `post_id` → `post`, `parent_id` NULL → `comment`, `body` VARCHAR(500) NOT NULL, `is_secret` BOOLEAN NOT NULL 기본 false, `created_at` TIMESTAMPTZ NOT NULL). **`UNIQUE(users_id, post_id)`는 없다**(D-7), 외래 키에 연쇄 삭제 없음(`003`과 같음), `comment_report`는 만들지 않는다(D-8). 인덱스 `(post_id, created_at, comment_id)`, `(users_id, created_at)` (data-model 1, 가안). 맨 위 주석에 D-7·D-8을 적는다 (FR-003, FR-008)
- [x] T016 [US1] `BE/comment/domain/Comment.java`, `BE/comment/repository/CommentRepository.java`: `Comment.write(postId, memberId, body, now)` — 줄바꿈을 `\n`으로 맞추고 앞뒤 공백·줄바꿈을 지운 값 저장, `parentId`는 늘 비움, `isSecret`은 늘 거짓(D-7). 수정 메서드는 만들지 않는다(FR-005). 쿼리: 글의 댓글 오래된 순(`created_at`, 같으면 `comment_id`), 글의 댓글 수, 회원의 마지막 댓글 시각 (FR-002, FR-003)
- [x] T017 [P] [US1] 입력 검사 `BE/comment/validation/ValidCommentBody.java` + 검사기(T016과 같은 정리 뒤 1 ~ `community.comment.max-length`, **코드 포인트로 셈**, 비면 `COMMENT_EMPTY`), `CommentFieldErrorMessages`(문구의 숫자를 `CommentProperties`에서) (FR-002, research B-2)
- [x] T018 [US1] `BE/comment/service/CommentService.java`: **쓰기** = `PostLookup.findVisible`(아니면 `POST_NOT_FOUND`) → 검사 → **회원별 잠금**(D-2. 방법은 가안: 트랜잭션 잠금 `pg_advisory_xact_lock(회원 번호)`처럼 회원마다 한 줄로 세우는 것, 구현 때 확인) → 마지막 댓글 시각에서 `min-interval`이 지나지 않았으면 `COMMENT_TOO_FREQUENT` (`Clock`) → 저장. **목록** = 오래된 순, 작성자는 `MemberNames`로 한 번에(탈퇴면 `withdrawn: true`, 닉네임·번호 없음, D-9), `canDelete`(로그인한 회원이 작성자이거나 `PostRef.ownerId`), `count` (contracts 1·2, FR-001 ~ FR-003, FR-007, FR-008)
- [x] T019 [US1] `BE/comment/controller/CommentController.java`: `GET /api/posts/{postId}/comments`(누구나), `POST /api/posts/{postId}/comments`(`201`, 목록의 댓글 하나와 같은 모양). 회원은 `LoggedInMember`로만. **댓글 수정 주소는 만들지 않는다** (contracts 1·2, FR-005)
- [x] T020 [US1] `BE/comment/service/CommentCounterAdapter.java`가 T010의 `PostCommentCounter`를 채운다 → 글 상세에 `commentCount` (FR-003)
- [x] T021 [US1] 댓글 영역 `FE/comment/CommentSection.tsx` + `comment.css`(원고지 토큰)를 `003`의 `PostDetailPage` 아래에 붙인다: 댓글 수, 오래된 순 목록(닉네임, 작성 시각, 내용), 탈퇴면 "탈퇴한 사용자". 내용은 **글자로만** 그린다(`white-space: pre-wrap`, `dangerouslySetInnerHTML` 쓰지 않음, research B-4). 로그인했으면 입력칸(글자 수 표시, 코드 포인트), 등록 중 버튼 잠금, 성공하면 맨 아래에 붙이고 수 +1, 실패 문구는 서버 것 그대로. 로그인 안 했으면 입력칸 대신 "로그인한 회원만 댓글을 쓸 수 있습니다"와 로그인 버튼(`useLoginPrompt`, 로그인하면 이 글로 돌아옴) (FR-001 ~ FR-003, FR-007, FR-030)

**Checkpoint**: T013, T014가 통과하고 S-1(1 ~ 6), S-2, S-3, S-12(1, 2, 4)를 화면으로 확인한다. 여기까지가 MVP다 (PR 하나)

---

## Phase 4: User Story 2 - 댓글 삭제와 정리 (Priority: P2)

**Goal**: 댓글 작성자 본인과 그 글의 블로그 주인만 댓글을 지운다. 글이 지워지면 댓글도 지워지고, 작성자가 탈퇴해도 남의 글에 단 댓글은 남는다

**Independent Test**: quickstart S-4, S-5, S-11(댓글 줄)

**먼저 merge**: `003` T030 ~ T037 (특히 T032, T034, T036)

### Tests for User Story 2

- [x] T022 [P] [US2] `BE-TEST/comment/controller/CommentDeleteTest.java`: S-4의 3 ~ 9 서버 쪽(작성자 `204`이고 줄이 실제로 없음, 블로그 주인 `204`, 다른 회원 `404 COMMENT_NOT_FOUND`(없는 댓글과 같은 응답)이고 그대로, 남의 블로그 주인 `404`, 로그인 안 하면 `401`, 남의 비공개 글의 댓글 → `404 COMMENT_NOT_FOUND`, 댓글 `PUT`·`PATCH` 주소가 없음(`405`)) (FR-004, FR-005, FR-029, SC-006)
- [x] T023 [P] [US2] `BE-TEST/comment/service/CommentCleanupTest.java` (클래스 전체 `@Transactional` 쓰지 않음): S-11의 댓글 줄 — 글을 지우면 그 글의 댓글 0건, 테스트용 듣는 쪽이 `PostDeletingEvent`에서 예외를 던지면 글과 댓글이 **모두 그대로**(한 묶음). S-5의 1 ~ 5 — C가 탈퇴하면 A의 글에 단 C의 댓글은 남고 `withdrawn: true`·닉네임 없음, C의 블로그 글에 B가 단 댓글은 블로그와 함께 없음(`BlogClosingEvent` → `003` T036 → `PostDeletingEvent` → T025), 새 회원이 C의 옛 닉네임으로 가입해도 C의 댓글은 "탈퇴한 사용자" (FR-006, FR-007, SC-005, D-9, `002` D-1)

### Implementation for User Story 2

- [x] T024 [US2] `CommentService.delete` + `CommentController`에 `DELETE /api/comments/{commentId}`(`204`): 댓글을 찾고 → 그 글을 요청한 사람이 볼 수 없으면 `COMMENT_NOT_FOUND`(댓글이 있다는 것도 알리지 않음, contracts 3) → 작성자도 블로그 주인도 아니면 **같은 `COMMENT_NOT_FOUND`**(남의 댓글이 있다는 것도 알리지 않음, 2026-10-08 결정 — `006`과 같음) → 줄을 **실제로** 지운다 (research B-3, FR-004, SC-006)
- [x] T025 [US2] `BE/comment/service/CommentPostCleaner.java`: `003` T032의 `PostDeletingEvent`를 `@EventListener`로 받아 그 글의 댓글을 지운다(같은 트랜잭션, `@Modifying` 쿼리) (FR-006, data-model 7의 1번)
- [x] T026 [US2] 탈퇴 정리 맞추기 (`002` T035에 적어 둔 약속): 댓글은 **`MemberWithdrawnEvent`를 듣지 않는다**(남의 글에 단 댓글은 남는 것이 규칙, FR-007, `002` D-1). 내 블로그 글의 댓글은 `003` T036이 낸 `PostDeletingEvent`로 T025가 지운다. 그래서 `BlogClosingEvent`도 따로 듣지 않는다(가안). `BE/user/MemberWithdrawnEvent.java`와 `BE/blog/BlogClosingEvent.java`의 주석을 실제 듣는 쪽에 맞게 고친다: "005 comment: `PostDeletingEvent`로 글의 댓글 / community: `PostDeletingEvent`로 좋아요·글 신고, `MemberWithdrawnEvent`로 내가 누른 좋아요 / 댓글 신고는 쓰지 않음(005 D-8)" (FR-006, FR-007)
- [x] T027 [US2] 화면: `CommentSection`의 댓글마다 `canDelete`일 때만 `삭제` 버튼 → `<dialog>` "댓글을 삭제할까요?", **취소하면 요청을 보내지 않음**, 성공하면 목록에서 빼고 수 -1. 댓글 고치기 버튼은 없다 (FR-004, FR-005)

**Checkpoint**: T022, T023이 통과하고 S-4, S-5를 화면으로 확인한다 (PR 하나, US1과 묶어도 된다)

---

## Phase 5: User Story 3 - 글에 좋아요 누르기 (Priority: P2)

**Goal**: 로그인한 회원이 남의 글에 좋아요를 누르고 다시 누르면 취소한다. 개수와 "내가 눌렀나"가 보인다

**Independent Test**: quickstart S-6, S-11(좋아요 줄)

**먼저 merge**: `003` T026 ~ T029, T032, T034, T036

### Tests for User Story 3

- [x] T028 [P] [US3] `BE-TEST/community/controller/PostLikeTest.java` (동시 요청 줄은 `@Transactional` 없이): S-6의 1 ~ 5·7·8(누르면 `likeCount` +1·`likedByMe` 참, 취소, 누르기 5번 연속·동시 5개여도 줄 하나, 자기 글 `403 SELF_LIKE_NOT_ALLOWED`이고 줄 없음, 로그인 안 하면 `401`, 남의 비공개 글 `404`), 누르지 않은 채 취소해도 `200`, S-11의 좋아요 줄(글을 지우면 0건), **탈퇴하면 그 회원이 남의 글에 누른 좋아요가 모두 지워짐**(`MemberWithdrawnEvent`, `002` T035) (FR-009 ~ FR-012, FR-028, SC-001, SC-002, SC-005)

### Implementation for User Story 3

- [x] T029 [US3] Flyway `V{다음 번호}__post_like.sql`: `post_like` 표를 팀 ERD 칸 그대로(`post_like_id` 자동 증가, `post_id`, `users_id`, `created_at`) + 설명으로만 적혀 있던 `UNIQUE(users_id, post_id)`를 **실제 제약**으로(data-model 2). 외래 키에 연쇄 삭제 없음 (FR-010, SC-002)
- [x] T030 [US3] `BE/community/domain/PostLike.java`, `BE/community/repository/PostLikeRepository.java`, `BE/community/service/LikeService.java`: `like` = `PostLookup.findVisible` → `ownerId == 나`면 `SELF_LIKE_NOT_ALLOWED` → 이미 있으면 아무것도 안 함, 동시에 들어와 중복 제약에 걸려도 성공으로 본다(research R-1, B-8) → `{ likeCount, likedByMe }`. `unlike` = 볼 수 있는 글인지 확인 → 있으면 지움 → 같은 모양 (FR-009 ~ FR-012)
- [x] T031 [US3] `BE/community/controller/LikeController.java`: `PUT /api/posts/{postId}/like`, `DELETE /api/posts/{postId}/like` (둘 다 `200`). `BE/community/service/PostLikeSummaryAdapter.java`가 T010의 `PostLikeSummary`를 채운다 → 글 상세에 `likeCount`, `likedByMe` (contracts 4·5·7, FR-011)
- [x] T032 [US3] 좋아요 정리 (`@EventListener`, 같은 트랜잭션): `BE/community/service/LikePostCleaner.java` — `PostDeletingEvent`로 그 글의 좋아요를 지운다(FR-028). `BE/community/service/LikeWithdrawalCleaner.java` — `MemberWithdrawnEvent`로 **그 회원이 누른 좋아요를 모두** 지운다(`002` T035, `MemberWithdrawnEvent` 주석, data-model 7) (FR-028, SC-005)
- [x] T033 [US3] 화면 `FE/community/LikeButton.tsx`를 글 상세에 붙인다: 개수, 눌렀으면 눌린 표시(`aria-pressed`), `likedByMe`를 보고 `PUT`/`DELETE`를 고름(research B-8), 보내는 중 잠금. **자기 글(`isOwner`)에서는 누를 수 없게** 한다(서버도 막음). 로그인 안 했으면 로그인 창을 띄우고, 로그인하면 이 글로 돌아온다(`useLoginPrompt`, `001` contracts 9) (FR-009 ~ FR-012)

**Checkpoint**: T028이 통과하고 S-6을 화면으로 확인한다 (PR 하나)

---

## Phase 6: User Story 6 - 글 신고하기 (Priority: P3)

> 명세의 우선순위는 US4, US5보다 뒤지만, **정해지지 않은 것이 없어서** 먼저 한다.

**Goal**: 로그인한 회원이 남의 글을 사유를 골라 신고하고, 접수만 저장한다. 같은 글은 한 번만

**Independent Test**: quickstart S-10, S-11(신고 줄)

**먼저 merge**: `003` T026 ~ T029, T032, T034, T036. E-7은 2026-10-08 결정(T003)

### Tests for User Story 6

- [x] T034 [P] [US6] `BE-TEST/community/controller/PostReportTest.java` (동시 요청 줄은 `@Transactional` 없이): S-10의 1 ~ 7·10(스팸 → `201` "신고가 접수되었습니다", 다시 → `409 ALREADY_REPORTED`, **동시 5개 → 한 건**, 기타에 빈 설명 접수, 200자 접수·201자 `REPORT_DETAIL_TOO_LONG`, 욕설·혐오에 보낸 설명은 저장 안 됨(research B-10), 자기 글 `403 SELF_REPORT_NOT_ALLOWED`, 사유 없음·없는 값 `REPORT_REASON_REQUIRED`), 로그인 안 하면 `401`, 남의 비공개 글 `404`, 신고 뒤에도 글 상세가 그대로(FR-020), S-11의 신고 줄(글을 지우면 그 글의 신고 0건, D-11, 신고가 있어도 글 삭제가 막히지 않음), **탈퇴해도 그 회원이 한 신고는 남음**(`002` D-5) (FR-017 ~ FR-021, SC-001 ~ SC-003)

### Implementation for User Story 6

- [x] T035 [US6] Flyway `V{다음 번호}__post_report.sql`: `post_report`(`post_report_id` **자동 증가**(data-model 4: ERD에 설명이 빠짐), `users_id`, `post_id`, `reason` VARCHAR(20) NOT NULL, `detail` VARCHAR(200) NULL, `created_at`) + T-3 `CHECK (reason IN ('SPAM','ABUSE','ADULT','OTHER'))` + 내 확장 `UNIQUE(users_id, post_id)`(T003의 E-7). 처리 상태 칸은 두지 않는다(FR-020). 맨 위 주석에 T-3·E-7 (FR-018, FR-019, SC-003)
- [x] T036 [P] [US6] `BE/community/domain/ReportReason.java`(`SPAM`, `ABUSE`, `ADULT`, `OTHER`), 요청 `BE/community/controller/dto/ReportRequest.java`(`reason` 없거나 네 값이 아니면 `REPORT_REASON_REQUIRED`, `detail`은 코드 포인트로 0 ~ `community.report.detail-max-length`), `ReportFieldErrorMessages` (FR-018)
- [x] T037 [US6] `BE/community/domain/PostReport.java`, `PostReportRepository`, `BE/community/service/ReportService.java`, `ReportController`에 `POST /api/posts/{postId}/reports`: 검사 순서(research B-1) 로그인 → `PostLookup.findVisible` → 자기 글이면 `SELF_REPORT_NOT_ALLOWED` → 입력 → 이미 있으면 `ALREADY_REPORTED`, 동시에 들어와 DB 중복 제약에 걸려도 같은 오류(research R-1) → 저장. 설명은 `OTHER`일 때만 저장하고 나머지는 버린다(B-10). 글을 숨기거나 지우는 코드는 **만들지 않는다** (contracts 6, FR-017 ~ FR-021)
- [x] T038 [US6] `BE/community/service/ReportPostCleaner.java`: `PostDeletingEvent`로 그 글의 신고를 지운다(D-11, `003` D-7). `MemberWithdrawnEvent`는 **듣지 않는다** — 내가 한 신고는 남긴다(`002` D-5). 클래스 주석에 이 두 줄을 적는다 (FR-020, data-model 7의 4번)
- [x] T039 [US6] 화면 `FE/community/ReportDialog.tsx`: 글 상세의 `신고` → `<dialog>`에서 사유 넷 중 하나(스팸, 욕설·혐오, 음란물, 기타), 기타일 때만 설명 칸(0 ~ 200자, 글자 수 표시), 성공하면 "신고가 접수되었습니다", `409`면 "이미 신고한 글입니다". 자기 글에서는 버튼을 숨긴다(`isOwner`). 로그인 안 했으면 로그인 창 (FR-017 ~ FR-021)

**Checkpoint**: T034가 통과하고 S-10을 화면으로 확인한다 (PR 하나)

---

## Phase 7: User Story 5 - 글에 태그 붙이고 태그로 찾기 (Priority: P3)

**Goal**: 글에 태그를 0 ~ 5개 붙이고, 태그를 누르면 같은 태그의 공개 글을 모아 본다

**Independent Test**: quickstart S-9, S-11(태그 줄)

**먼저 merge**: `003` T017 ~ T024, T026 ~ T029, T033 ~ T035, T037. (`D-10`은 2026-10-08 결정: 소문자로 저장)

### Tests for User Story 5

- [ ] T040 [P] [US5] `BE-TEST/post/controller/PostTagTest.java`: S-9의 1 ~ 12(`#여행`은 `여행`으로, 태그 없이 저장, 6개 → `TAG_TOO_MANY`이고 글도 저장 안 됨, 16자·공백·쉼표 → `tags[n]` `TAG_INVALID`, 15자와 `#`+15자 통과, `Java`+`java` → `TAG_DUPLICATED`, 태그 목록은 공개 글만 최신순(**주인이 봐도 자기 비공개 글은 안 나옴**), 비공개로 바꾸면 바로 빠짐, 수정에서 빼고 더하기, 화면 없이 6개 거절, DB에서 한 글 5개 초과·겹침 0), S-11의 태그 줄(글을 지우면 연결 0건, `tag` 줄은 남음). S-9의 1에서 보이는 모양은 **소문자 `java`** (D-10 A) (FR-013 ~ FR-016, FR-028, FR-029, SC-004, SC-005, SC-008)

### Implementation for User Story 5

- [x] T041 [US5] **`D-10` 결정 받기** (T002): 2026-10-08 사용자가 추천대로 **A. 소문자로 바꿔 저장**을 골랐다. research·plan의 D-10 줄을 고쳤다
- [ ] T042 [US5] Flyway `V{다음 번호}__tag.sql`: `post_tag`(`post_id` → `post`, `tag_id` → `tag`, 기본키 `(post_id, tag_id)`, 연쇄 삭제 없음) + 인덱스 `post_tag (tag_id)`(data-model 3, 태그로 찾기). `tag`(`tag_id` 자동 증가, `name` VARCHAR(15) NOT NULL)의 중복 불가 규칙은 팀 ERD의 `UNIQUE(name)` 그대로 (D-10 A: 소문자로 저장하므로 대소문자 무시가 된다) (FR-014, SC-004)
- [ ] T043 [P] [US5] `BE/post/config/TagProperties.java`(`community.tag`: `max-per-post` 5, `min-length` 1, `max-length` 15(DB 칸 15보다 크면 서버가 안 켜짐), `page-size` 10) + `application.yml`. `BE/post/tag/TagNormalizer.java`(research B-6 순서): 앞뒤 공백 지우기 → 앞의 `#` 모두 지우기 → 1 ~ 15자(코드 포인트), 공백·쉼표 없음 → 대소문자 무시로 겹침 확인(조용히 합치지 않고 `TAG_DUPLICATED`) → 5개 이하. 오류는 `tags` / `tags[n]` 칸으로. **소문자로 바꿔 저장한다** (D-10 A) (FR-013, FR-014)
- [ ] T044 [US5] `post` 모듈 안의 태그 저장: `BE/post/domain/Tag.java`, `PostTag.java`, `TagRepository`, `PostTagRepository`, `BE/post/service/PostTagService.replaceTags(postId, tags)` — 보낸 목록이 **새 전체 목록**(빠진 연결 삭제, 새것 추가), 이름이 있으면 그 줄, 없으면 새로 만들고 동시에 같은 새 태그가 만들어져 중복 제약에 걸리면 다시 읽어 쓴다(research B-6). `003`의 `PostRequests`(T023)에 `tags`를 더하고 `PostWriteService.create`(T022)·`PostEditService.update`(T034) 안에서 같은 트랜잭션으로 부른다. 수정 화면 응답(`GET …/edit`)에 지금 태그. **글 삭제**는 같은 모듈이므로 `PostEditService.delete`의 삭제 순서에 `post_tag` 지우기를 넣는다(글보다 먼저). `tag` 줄은 남긴다 (FR-013, FR-014, FR-016, FR-028). **태그만 바꾼 수정**도 `changed: true`(수정 시각 갱신, "수정됨")로 본다 (2026-10-08 `가안`, 사용자 아침 확인. `003` FR-020의 판단과 맞춘다)
- [ ] T045 [US5] 태그 보기: 글 상세 응답에 `tags`(contracts 7). `GET /api/tags/{tagName}/posts?page=` (`BE/post/controller/TagPostController.java`, contracts 9): 주소의 이름을 T043과 같은 방법으로 정리한 뒤 찾는다. **보는 사람과 상관없이** `PostVisibility`의 "주인이 아닐 때" 조건(글 공개 + 분류 공개)만 쓴다(SC-008). 최신순, `page-size`개씩, 없는 페이지면 마지막 페이지(`004` CF-10-4를 빌린 가안). 한 줄 정보는 contracts 9 그대로 두고 `004`의 목록 계약이 정해지면 맞춘다. 없는 태그는 빈 목록 `200` (FR-015, SC-008)
- [ ] T046 [US5] 화면: `FE/post/TagInput.tsx`를 `003`의 `PostEditorPage`에 붙인다(Enter·쉼표로 하나씩 더하기, 같은 규칙으로 바로 안내(보조), 서버의 `tags[n]` 오류를 그 태그 아래에). 글 상세 **아래**에 태그 목록(누르면 `/tags/:tagName`). 새 화면 `FE/post/TagPostsPage.tsx`(`/tags/:tagName`, 누구나, 글이 없으면 "글이 없습니다"(`상세/04`)). 보이는 모양은 소문자(D-10 A)이므로 "태그는 소문자로 보입니다"를 입력칸 옆에 안내한다(research D-10 `영향`) (FR-013 ~ FR-016)

**Checkpoint**: T040이 통과하고 S-9를 화면으로 확인한다 (PR 하나)

---

## Phase 8: User Story 4 - 글에 이미지 올리기 (Priority: P2)

> 명세의 우선순위는 P2지만 **`D-1`, `D-5`, `D-6`과 정리 시간을 기다리느라** 맨 뒤에 두었다. 2026-10-08에 모두 정해졌으므로(개발은 MinIO, DB 먼저·파일 나중, 시그니처 확인, 24시간) 결정을 기다릴 작업은 없다.

**Goal**: 글을 쓰는 중에 jpg·png·gif·webp 이미지(5MB 이하, 글당 10장)를 올리고, 본문 안에서 본다. 글을 지우면 이미지도 지워진다

**Independent Test**: quickstart S-7, S-8, S-11(이미지 줄, 10번)

**먼저 merge**: `003` T017 ~ T024, T026 ~ T037. 본문 안에 그림으로 보이는 것은 `003` T025(마크다운 표시 도구) 뒤

### Tests for User Story 4

- [ ] T047 [P] [US4] `BE-TEST/image/controller/ImageUploadTest.java` (저장소는 테스트용 가짜 `ImageStorage`): S-7의 1 ~ 7·10(4.9MB png 통과, **정확히 5MB 통과**, 5.1MB → `400 INVALID_IMAGE`(프레임워크가 먼저 막아도 같은 문구, research R-6), gif·webp 통과, bmp 거절, 새 글 기준 11번째·저장한 글의 수정에서 11번째 → `409 IMAGE_LIMIT_EXCEEDED`, 로그인 안 하면 `401`이고 파일 없음), S-8의 1 ~ 7(이름만 `.png`인 HTML 거절, `.jpg`로 바꾼 png는 png로 저장·응답, `../../test.png` 이름을 쓰지 않음, `nosniff`, 남의 비공개 글 이미지 `404`, 연결 전 남의 이미지 `404`, 오류 응답에 경로·예외 이름 없음), `postId`가 남의 글이면 `404` (FR-022 ~ FR-026, FR-029, SC-004)
- [ ] T048 [P] [US4] `BE-TEST/image/service/ImageCleanupTest.java` (클래스 전체 `@Transactional` 쓰지 않음): S-11의 2·4·7·8(글을 지우면 `post_image` 0건, 파일도 없음(D-5 A: 파일 삭제가 실패했으면 정리 작업 뒤에 확인), 지운 글의 이미지 주소 `404`), S-11의 10(저장하지 않고 나간 글의 이미지는 정리 시간이 지나면 기록·파일 삭제), 글 저장 때 본문에서 뺀 이미지 삭제, S-8의 8(저장소가 꺼지면 `503 STORAGE_UNAVAILABLE`이고 서버가 죽지 않음) (FR-027, SC-005)

### Implementation for User Story 4

- [x] T049 [US4] **결정 받기** (T002): 2026-10-08 사용자가 추천대로 골랐다 — `D-1` 개발은 MinIO(`ImageStorage`로 교체 가능), `D-5` DB 먼저·파일 나중·실패는 정리 작업, `D-6` 파일 앞부분의 형식 표시, D-3 정리 시간 **24시간**(`가안`). 정리 시간은 `상세/05`의 `기본값` 표에 먼저 적고 plan `설정값 목록`에 옮겼다
- [ ] T050 [US4] `image` 모듈 뼈대 `BE/image/package-info.java`(`@ApplicationModule(displayName = "이미지", allowedDependencies = {"post", "user", "common"})`)와 저장소 틀 `BE/image/storage/ImageStorage.java`(가안, research A "저장소 감싸기"): `put(key, InputStream, size, contentType)`, `open(key)`, `delete(key)`. 저장소에 연결할 수 없으면 `STORAGE_UNAVAILABLE`로 바꾼다. 테스트용 메모리 구현 `BE-TEST/image/storage/InMemoryImageStorage.java` (FR-026)
- [ ] T051 [US4] 저장소 구현: **D-1 결정대로 개발은 MinIO** ( `docker-compose.yml`에 MinIO 서비스(`127.0.0.1`에만 열기), `pom.xml`에 AWS SDK for Java v2의 S3 클라이언트, `BE/image/storage/S3ImageStorage.java`, 접속값은 환경 변수(개발 기본값만 yml에). 서버 디스크 구현으로 **설정 하나로 바꿔 끼울 수 있게**(`@ConditionalOnProperty`, 가안). 서버 디스크로 바꿀 때는 `DiskImageStorage`와 저장 폴더 설정). 배포 때의 선택은 `001` D-5와 함께 (research D-1, R-5)
- [ ] T052 [US4] Flyway `V{다음 번호}__post_image.sql`: `post_image`(`post_image_id` 자동 증가, `post_id` **NULL 허용** → `post`, `users_id` NOT NULL → `users`, `storage_key` VARCHAR(500) NOT NULL **중복 불가**, `created_at` NOT NULL). 팀 요청 T-1(D-3)과 T003의 `storage_key` 중복 불가를 맨 위 주석에 적는다. 인덱스 `(post_id)`, 연결 전 이미지 찾기용 `(users_id, created_at) WHERE post_id IS NULL` (가안) (FR-024, FR-027)
- [ ] T053 [P] [US4] `BE/image/config/ImageProperties.java`(`community.image`: `max-size` 5MB, `max-per-post` 10, `allowed-types` jpg·png·gif·webp, **연결 전 이미지 정리 시간 `orphan-ttl` 24시간**(`가안`, `상세/05` 기본값)) + `application.yml`. `spring.servlet.multipart.max-file-size`와 `max-request-size`는 **`${community.image.max-size}`에서 값을 가져온다**(숫자를 두 곳에 쓰지 않음, plan). `GlobalExceptionHandler`에 `MaxUploadSizeExceededException` → `400 INVALID_IMAGE`를 더한다(R-6). 정확히 5MB가 통과하는지 T047로 확인 (FR-023, FR-025)
- [ ] T054 [US4] 형식 확인 `BE/image/service/ImageTypeDetector.java`: **D-6 결정(B)대로 파일 앞부분의 형식 표시로 확인** ( jpg `FF D8 FF`, png `89 50 4E 47 0D 0A 1A 0A`, gif `GIF87a`/`GIF89a`, webp `RIFF....WEBP`). 확장자와 브라우저가 보낸 형식은 믿지 않는다. 저장 확장자와 내려 줄 `Content-Type`은 **확인한 형식**으로. SVG는 받지 않는다 (FR-022, SC-004)
- [ ] T055 [US4] `BE/image/service/ImageUploadService.java` + `BE/image/controller/ImageController.java`의 `POST /api/images`(`multipart/form-data`, `file`, 선택 `postId`, `201 { imageId, url }`): 로그인 → `postId`가 있으면 **내 글**인지(`PostLookup`, 아니면 `POST_NOT_FOUND`) → 용량 → 형식(T054) → 개수(수정 중이면 그 글의 이미지 수, 새 글이면 **아직 연결되지 않은 내 이미지 수**, D-3) → 새 이름 `posts/{UUID}.{확인한 확장자}`(사용자 파일 이름은 쓰지도 돌려주지도 않음) → `ImageStorage.put` → DB 기록(DB가 실패하면 방금 올린 파일을 지움) (contracts 10, research B-5, FR-022 ~ FR-025)
- [ ] T056 [US4] `ImageController`의 `GET /api/images/{fileName}`: 기록을 찾고, 연결된 글이면 `PostLookup.findVisible(postId, 보는 사람)`, 연결 전이면 **올린 사람만**. 아니면 `404`. `Content-Type`은 확인한 형식, `X-Content-Type-Options: nosniff`, 비공개 글·연결 전 이미지는 공용 캐시에 남지 않게(`Cache-Control: private`, 가안) (contracts 11, research B-5, R-7, FR-026)
- [ ] T057 [US4] **글 저장 때 이미지 연결** (D-3, D-4): `post` 맨 위에 이벤트 `BE/post/PostContentSavedEvent.java`(`record(Long postId, Long ownerId, String content)`, 가안)를 두고 `003`의 `PostWriteService.create`·`PostEditService.update`가 같은 트랜잭션에서 낸다. `BE/image/service/ImageLinker.java`(`@EventListener`): 본문에서 **우리 서버 주소(`/api/images/…`)의 마크다운 이미지만** 찾아 내가 올린 연결 전 이미지와 이 글의 이미지를 이 글에 연결하고, 10장을 넘으면 `fieldErrors.content` `IMAGE_LIMIT_EXCEEDED`로 거절(저장 전체 취소, contracts 8), 이 글에 있었는데 본문에서 빠진 이미지는 기록을 지우고 파일은 T058과 같은 방법으로 지운다. 남이 올린 이미지 주소는 연결하지 않는다 (FR-024, FR-026, SC-004)
- [ ] T058 [US4] 이미지 정리: ① `BE/image/service/ImagePostCleaner.java` — `PostDeletingEvent`로 그 글의 `storage_key`를 읽어 두고 기록을 지운다(같은 트랜잭션). **파일 삭제는 D-5 결정(A)대로 트랜잭션이 끝난 뒤** 지우고(`@TransactionalEventListener(phase = AFTER_COMMIT)` 등), 실패하면 로그를 남기고 ②가 다시 지운다) ② `BE/image/service/OrphanImageCleaner.java`(`@Scheduled`, `@EnableScheduling`은 `common`에 한 번): 정리 시간(24시간, T053)보다 오래된 연결 전 이미지의 기록·파일을 지운다. D-5가 A이므로 저장소 목록과 DB를 비교해 주인 없는 파일도 지운다(`ImageStorage`에 목록 보기가 필요하면 T050에 더함). 탈퇴한 회원의 연결 전 이미지도 이 작업이 지운다(따로 듣지 않음, 가안) (FR-027, SC-005, research D-3, D-5)
- [ ] T059 [US4] 화면: `003`의 `PostEditorPage`에 `FE/image/ImageUploadButton.tsx`(파일 고르기 `accept`는 jpg·png·gif·webp, 크기를 화면에서도 먼저 보고(보조), **한 장씩** 보냄, 받은 주소를 커서 자리에 `![](주소)`로 넣음(D-4), 실패 문구는 서버 것 그대로, 수정 중이면 `postId`를 같이 보냄). 글 상세의 본문 안 이미지는 `003` T029의 `MarkdownView`가 그린다 — **`003` T025(마크다운 표시 도구, react-markdown + remark-gfm)가 merge되기 전에는 원문 글자로 보인다.** 그 뒤에는 이미지 주소는 우리 서버 주소만 그리게 한다(D-4, S-12의 5). 글 목록의 대표 이미지는 만들지 않는다 (FR-022 ~ FR-026)

**Checkpoint**: T047, T048이 통과하고 S-7, S-8, S-11(10)을 화면으로 확인한다. 본문 안 그림은 `003` T025 뒤에 S-7의 8을 다시 본다 (PR 하나)

---

## Phase 9: Polish & Cross-Cutting Concerns

- [ ] T060 [P] 결정·구현에 맞게 문서를 같이 고친다 (헌법 `작업 흐름`): research 머리말 D 절의 "2026-10-07 현재 하나도 정하지 않았습니다", plan 머리말의 "아직 하나도 정하지 않았습니다"와 `Constitution Check`의 "결정 대기 11건", plan FR 연결표의 `미정` 줄(FR-007은 D-9, FR-008은 D-2, FR-026은 D-4, FR-020은 D-11로 결정됨), research A·plan의 Redis 줄("이메일 인증번호 저장에만" → 헌법 1.1.0 문구), plan `Technical Context`(버전·테스트 `미정` → `001`·`002`에서 쓰는 것), plan `Project Structure`(좋아요·신고·이미지를 `post` 안에 → `community`·`image` 모듈, 태그는 `post`), data-model 7의 "남의 글에 한 신고는 가안"(→ `002` D-5 결정), data-model 9(→ `ERD-변경-요청.md`의 T-1·T-3과 T003의 E-7), quickstart S-1의 6(`UNIQUE`는 이미 지워짐), S-3의 6(D-2 결정), `상세/05`의 `구현 방식`("작성 예정" → 만든 것), `기술스택-아키텍처.md` 2.2 표(Spring 이벤트 줄에 005 듣는 쪽, `@Scheduled` 줄에 이미지 정리), 주소·이벤트·설정 이름이 가안에서 바뀌었으면 그것도
- [ ] T061 [P] quickstart S-12를 화면에서 실행한다: 댓글의 `<script>`·`<img onerror>`·`javascript:` 링크가 글자로 보임, 댓글·태그의 SQL 같은 글자, 본문 이미지 문법에 바깥 주소·`javascript:` 주소(D-4, `003` T025 뒤), CSRF 없이 댓글·좋아요·신고·이미지 올리기 거절, 화면 없이 남의 댓글 삭제·자기 글 좋아요·신고·남의 글 번호로 이미지 올리기 거절, 이상한 JSON에 내부 정보 없음. S-12의 3(`006` 댓글 관리 화면)은 `006` 뒤에 (FR-029, FR-030, SC-010)
- [ ] T062 [P] `003` T052(보안 헤더 Content-Security-Policy 검토)에 이미지 주소를 알려 준다: 본문 이미지는 우리 서버 주소(`/api/images/…`)만 쓰므로 `img-src 'self'`로 둘 수 있다(가안). 결정은 `003` T052에서 사용자에게 묻는다
- [ ] T063 quickstart S-13: 댓글 200개, 좋아요 100개, 태그 5개가 붙은 글의 상세(댓글 목록 포함)가 2초 안인지 본다. 느리면 T015의 `comment (post_id, created_at, comment_id)` 인덱스와 T010의 개수 세기부터 확인한다 (NF-09, research R-2)
- [ ] T064 글 삭제 연쇄 전체 확인: 005 S-11 전체(댓글 2, 좋아요 2, 태그 3, 이미지 3, 신고 1이 달린 글을 지우면 모두 0건, 도중 오류면 모두 그대로)와 `003` quickstart S-8의 3 ~ 6(`003`이 "`005` 뒤에"로 남겨 둔 줄), `002` 탈퇴(S-8)를 다시 실행한다 (FR-006, FR-027, FR-028, SC-005)
- [ ] T065 SC-009(제안값 1분): **사람이** 글 상세에서 댓글 입력칸을 누른 때부터 등록이 끝날 때까지 시간을 잰다 (S-1의 7)
- [ ] T066 quickstart 전체를 실행하고 끝의 `구현 뒤에 채울 것`(실행 명령, 이미지 저장소 띄우기·들여다보기, 화면 확인 순서, D-항목 반영 문구)을 채운다. `tasks.md` 체크박스와 `CLAUDE.md`의 `6. 지금 상태`를 고친다

---

## 요구사항 → 작업 연결표

| FR | 작업 |
|---|---|
| FR-001 | T011, T013, T018, T019, T021 |
| FR-002 | T006, T007, T013, T016, T017 |
| FR-003 | T008, T010, T013, T016, T018, T019, T020, T021 |
| FR-004 | T007, T022, T024, T027 |
| FR-005 | T016, T019, T022, T027 |
| FR-006 | T023, T025, T026, T064 |
| FR-007 | T008, T018, T021, T023, T026 |
| FR-008 | T006, T007, T014, T015, T018 |
| FR-009 | T011, T028, T030, T033 |
| FR-010 | T028, T029, T030, T033 |
| FR-011 | T010, T028, T030, T031, T033 |
| FR-012 | T007, T028, T030, T033 |
| FR-013 | T040, T043, T044, T046 |
| FR-014 | T040, T042, T043, T044, T046 (저장 모양은 D-10) |
| FR-015 | T040, T045, T046 |
| FR-016 | T040, T044, T046 |
| FR-017 | T007, T034, T037, T039 |
| FR-018 | T006, T034, T035, T036, T037, T039 |
| FR-019 | T003, T007, T034, T035, T037 |
| FR-020 | T034, T035, T037, T038 |
| FR-021 | T034, T037, T039 |
| FR-022 | T047, T054, T055, T059 (확인 방법은 D-6) |
| FR-023 | T047, T053, T055 |
| FR-024 | T007, T047, T052, T055, T057 |
| FR-025 | T007, T047, T053, T059 |
| FR-026 | T050, T056, T057, T059 (본문 안 그림은 `003` T025) |
| FR-027 | T048, T052, T058, T064 (파일 삭제는 D-5, 저장소는 D-1) |
| FR-028 | T028, T032, T040, T044, T064 |
| FR-029 | T009, T011, T013, T022, T028, T034, T040, T047, T061 |
| FR-030 | T013, T021, T061 |

| SC | 확인하는 작업 |
|---|---|
| SC-001 | T013, T028, T034 |
| SC-002 | T028, T029, T034 |
| SC-003 | T034, T035 |
| SC-004 | T040, T047, T054 |
| SC-005 | T023, T028, T034, T040, T048, T064 |
| SC-006 | T022 |
| SC-007 | T014 |
| SC-008 | T040, T045 |
| SC-009 | T065 |
| SC-010 | T013, T061 |

---

## Dependencies & Execution Order

### Phase Dependencies

- **Setup (Phase 1)**: T001(merge 확인)은 이야기마다 다시 본다. T003, T004는 사용자 확인이 필요하지만 US1을 막지 않는다 (US6은 T003 뒤)
- **Foundational (Phase 2)**: `003` T003 ~ T012 merge 뒤. **모든 사용자 이야기를 막는다**
- **US1 (P1)**: Foundational + `003` US3(T026 ~ T029) 다음
- **US2 (P2)**: US1 다음 + `003` US4(T030 ~ T037) merge 뒤
- **US3 (P2)**: Foundational + `003` US3·US4 다음. US1·US2와 서로 기대지 않는다
- **US6 (P3)**: Foundational + `003` US3·US4 + T003 다음. **정해지지 않은 것이 없어** US5·US4보다 먼저 한다
- **US5 (P3)**: `003` US2·US4 merge 뒤 (`D-10`은 결정됨)
- **US4 (P2)**: `003` US2·US4 merge 뒤 (`D-1`, `D-5`, `D-6`, 정리 시간은 결정됨). 본문 안 그림은 `003` T025 뒤
- **Polish**: 원하는 이야기가 끝난 뒤. T064는 US2·US3·US6·US5·US4가 모두 끝난 뒤

### 미정 항목이 막는 작업

`D-10`, `D-1`, `D-5`, `D-6`, 정리 시간과 `003` D-8(마크다운 도구)은 2026-10-08에 정해져서 결정이 막는 작업은 없다. T059의 본문 안 그림과 T061의 S-12 5는 `003` T025가 merge된 뒤에 확인한다.

### Within Each User Story

- 테스트 → 표(Flyway) → 도메인 → 서비스 → 주소(컨트롤러) → 화면 순서
- 테스트는 먼저 써 두고 실패하는 것을 본 뒤 구현한다. push 전에 `mvn verify`와 화면 `lint`·`build`를 돌린다
- 한 이야기를 끝내고 Checkpoint를 통과한 뒤 PR을 merge하고 다음으로 간다

### Parallel Opportunities

- T003, T004는 동시에 (문서). T005 ~ T008, T012는 동시에
- US1의 T013, T014, T017은 동시에
- US2의 T022, T023은 동시에
- US3(좋아요)와 US1·US2(댓글)는 서로 다른 모듈이라 동시에 할 수 있다. 단 `ErrorCode`(T007)와 `SecurityConfig`(T011)는 Foundational에서 한 번에 끝낸다
- US4의 T047, T048, T053은 동시에

---

## Parallel Example: User Story 1

```text
먼저 동시에:
Task: "T013 CommentWriteTest (S-1, S-2, S-12)"
Task: "T014 CommentIntervalTest (S-3, 동시 요청)"
Task: "T017 @ValidCommentBody (코드 포인트, 공백만이면 비어 있음)"
그다음: T015 → T016 → T018 → T019 → T020 → T021
```

---

## Implementation Strategy

### MVP First (US1)

1. Phase 1, 2를 끝낸다 (US1 PR에 같이 넣는다)
2. US1(댓글 쓰고 읽기)
3. **멈추고 확인**: quickstart S-1 ~ S-3
4. 보여 줄 수 있으면 보여 준다

### Incremental Delivery

1. US2(댓글 삭제·정리) → 확인
2. US3(좋아요) → 확인
3. US6(신고) → 확인
4. US5(태그, `D-10` 결정됨) → 확인
5. US4(이미지, `D-1`, `D-5`, `D-6`, 정리 시간 결정됨) → 확인. `003` T025가 merge되면 본문 안 그림을 다시 확인
6. T064로 글 삭제·탈퇴 연쇄 전체를 다시 확인

---

## Notes

- 작업 하나 또는 묶음 하나를 끝낼 때마다 커밋하고, 커밋 메시지에 작업 ID를 적는다 (예: `005 T018`). PR은 이야기 하나에 하나
- `가안`인 주소, 오류 이름, 설정 이름, 입구·이벤트 이름(`MemberNames`, `PostLookup`, `PostCommentCounter`, `PostLikeSummary`, `PostContentSavedEvent`, `ImageStorage`)과 모듈 나누기를 바꾸면 T060처럼 문서도 같이 고친다
- `003`은 다른 세션이 만드는 중이다. `003`의 클래스·이벤트 이름이 계획(`003` tasks.md)과 달라지면 이 목록의 이름도 그에 맞춘다
- 남은 결정: 이 기능의 D-항목은 없다 (2026-10-08 모두 결정. 정리 시간 24시간과 태그만 바꾼 수정의 "수정됨"은 사용자 아침 확인). `003`의 마크다운 표시 도구도 정해졌다(react-markdown + remark-gfm). 확인할 위험: research R-1(좋아요·신고 동시 요청, T030·T037), R-2(글 상세 2초, T063), R-3(남의 비공개 글에 직접 요청, T009), R-6(5MB 경계, T053), R-7(이미지 주소, T056)
