# Tasks: API 주소 버전과 릴리스 기록 (v0.4.0)

> 2026-10-12: 코드 브랜치 `feat/v0.4-api-v1-release-docs` → 코드 PR #51 (썸네일 PR #50 merge 뒤).

**Input**: [spec.md](spec.md)

- [x] T001 [US1] 서버: `ApiVersionConfig`(Spring Framework 7 API 버전, 주소 두 번째 칸, 지원 버전 1, 이미지 파일 주소는 버전 없음), 처리기 주소 `/api/{version}/…`, 보안 규칙 `/api/*/…`·로그아웃 `/api/v1/auth/logout`, 버전 오류 `API_VERSION_UNSUPPORTED`(예전 주소는 로그인 없이도 400)
- [x] T002 [US1] 테스트: `ApiVersionTest` + 기존 테스트 주소를 `/api/v1`로
- [x] T003 [US1] 화면: 요청 주소를 모두 `/api/v1/…`로 (이미지 파일 주소는 그대로), `API_VERSION` 상수
- [x] T004 [US2] `CHANGELOG.md`(v0.1.0, v0.3.0, v0.4.0 준비 중), 서버·화면 버전 0.4.0, 배포 작업이 태그·버전 번호·CHANGELOG를 확인, `deploy/README.md` 릴리스 순서
- [x] T005 [US3] 자동 API 문서(springdoc 3.1.1, 개발 설정에서만 `/swagger-ui.html`, 머리 "v1 · 릴리스 0.4.0", 주소의 `{version}`을 v1로 채움), 화면 바닥글
- [x] T006 docs: 쉬운 설명(`docs/4-가이드/06-API-버전과-릴리스.md`), 계약 문서 6개 머리에 주소 버전 안내와 `thumbnail` 칸
- [x] T007 확인: `mvn verify`, `npm run lint`, `npm run build`, actionlint, Playwright로 가입·로그인·글쓰기·로그아웃(모두 `/api/v1`)과 바닥글
- [x] T010 릴리스 주기 (2026-10-12 사용자: 로컬에서 어느 정도 해 두고 버전을 올려 배포. "작게 자주"는 같은 날 바로잡음): 배포 작업 손 실행에 릴리스 번호 → check(번호·버전·CHANGELOG·태그 중복) → deploy → release(태그·GitHub 릴리스를 CHANGELOG 절로, 이 작업만 contents: write). CHANGELOG 머리·deploy/README·가이드 06·CLAUDE.md
- [x] T008 PR 올리고 CI·CodeRabbit 확인 뒤 merge (코드 PR #51, CodeRabbit 지적 5개 반영: v+숫자만 받는 파서, 버전 없는 주소의 CSRF, 9자리 제한, package-lock 확인, README)
- [x] T009 2026-10-12 "배포해줘" → Actions `배포` 번호 `0.4.0`. 첫 시도는 배포 스크립트의 동작 확인 주소가 예전 `/api/auth/csrf`라 실패(서버는 켜짐) → 코드 PR #52로 고치고 다시 성공, GitHub 릴리스 v0.4.0 자동 생성 → 예시 데이터 `photos` 성공
- [x] T011 API 문서 보강 (2026-10-14 사용자: 발표 자료용, 릴리스 0.5.3, 코드 PR #67 `develop` merge b083ad7. 리뷰 반영: 설명에 설정값 숫자 대신 설정 이름): 컨트롤러 22개·API 61개에 `@Tag`(10개 묶음)·`@Operation` 한국어 설명, 문서 머리에 로그인·CSRF·오류 설명, 로그인 필요(`세션 쿠키`)·CSRF 표시(보안 설정과 같은 목록), 자료 모양 이름을 `PostRequests.Write`처럼(겹침 고침), `OpenApiDocsTest` 3개. Spring REST Docs 4.0.1: `ApiDocumentationTest` 22개 흐름, `src/docs/asciidoc/index.adoc`(원고지 모양), `mvn -Pdocs verify` → `backend/target/generated-docs/index.html`
