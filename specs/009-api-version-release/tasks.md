# Tasks: API 주소 버전과 릴리스 기록 (v0.4.0)

> 2026-10-12: 코드 브랜치 `feat/v0.4-api-v1-release-docs` (썸네일 PR #50 위). PR #50이 합쳐지면 PR로 올린다.

**Input**: [spec.md](spec.md)

- [x] T001 [US1] 서버: `ApiVersionConfig`(Spring Framework 7 API 버전, 주소 두 번째 칸, 지원 버전 1, 이미지 파일 주소는 버전 없음), 처리기 주소 `/api/{version}/…`, 보안 규칙 `/api/*/…`·로그아웃 `/api/v1/auth/logout`, 버전 오류 `API_VERSION_UNSUPPORTED`(예전 주소는 로그인 없이도 400)
- [x] T002 [US1] 테스트: `ApiVersionTest` + 기존 테스트 주소를 `/api/v1`로
- [x] T003 [US1] 화면: 요청 주소를 모두 `/api/v1/…`로 (이미지 파일 주소는 그대로), `API_VERSION` 상수
- [x] T004 [US2] `CHANGELOG.md`(v0.1.0, v0.3.0, v0.4.0 준비 중), 서버·화면 버전 0.4.0, 배포 작업이 태그·버전 번호·CHANGELOG를 확인, `deploy/README.md` 릴리스 순서
- [x] T005 [US3] 자동 API 문서(springdoc 3.1.1, 개발 설정에서만 `/swagger-ui.html`, 머리 "v1 · 릴리스 0.4.0", 주소의 `{version}`을 v1로 채움), 화면 바닥글
- [x] T006 docs: 쉬운 설명(`docs/4-가이드/06-API-버전과-릴리스.md`), 계약 문서 6개 머리에 주소 버전 안내와 `thumbnail` 칸
- [x] T007 확인: `mvn verify`, `npm run lint`, `npm run build`, actionlint, Playwright로 가입·로그인·글쓰기·로그아웃(모두 `/api/v1`)과 바닥글
- [ ] T008 PR 올리고 CI·CodeRabbit 확인 뒤 merge
- [ ] T009 사용자가 "배포해줘" 하면: GitHub 릴리스 `v0.4.0`(설명에 CHANGELOG 절) → 배포 → 예시 데이터 `photos`
