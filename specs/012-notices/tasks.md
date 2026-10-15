# Tasks: 공지사항 (v0.6.0)

**Input**: [spec.md](spec.md), [research.md](research.md), [data-model.md](data-model.md), [contracts/notice-api.md](contracts/notice-api.md) · 코드 브랜치 `feature/v0.6-notices` → 코드 PR #69 `develop` merge (2026-10-15 구현, 서버 테스트 37개는 구현 전 34개 실패 확인 → 모두 통과, 화면 테스트 37개는 일부러 틀리게 바꿔 실패 확인)

테스트 전략(`docs/3-설계/테스트-전략.md`)대로 **테스트 작업을 구현 앞에** 둔다. 테스트가 먼저 실패하는 것을 보고 구현한다.

## 바탕
- [x] T001 Flyway V11(`users.role`, `notice` 표), `ERD-변경-요청.md` E-9
- [x] T002 설정 `notice.*`(글자 수, 한 쪽 글 수, 릴리스 링크 틀), 새 모듈 `notice`(회원 모듈만 부름)

## US5 관리자
- [x] T003 [테스트] 관리자 확인: 비회원 401, 일반 회원 403, 관리자 통과, 관리자에서 뺀 뒤 403, 탈퇴한 관리자 401. `/auth/me`의 `admin`
- [x] T004 `User.role`, `LoggedInMember.requireAdminIdOf`, `/auth/me`에 `admin`

## US4 쓰기·고치기·지우기
- [x] T005 [테스트] 입력 경계값(제목 0·1·100·101자, 본문 10,000·10,001자, 이모지), 번호 모양(`0.6.0`·`0.10.12` 통과, `v0.6`·`0.6`·`0.6.0.1`·`abc`·`12345.0.0` 거절), 업데이트인데 번호 없음, 공지인데 번호 있음, 모르는 종류. CSRF 없으면 403. 고치기·지우기·없는 번호 404
- [x] T006 `POST·PUT·DELETE /notices`, 검사, 오류 코드

## US1·US2 보기
- [x] T007 [테스트] 목록: 누구나, 탭별, 고정 먼저 + 새 글 먼저, 11개면 2쪽, 모르는 종류는 전체, 쪽 번호 규칙. 읽기: 릴리스 링크(업데이트만), 404. 최신 글 시각(없으면 null)
- [x] T008 `GET /notices`, `GET /notices/{id}`, `GET /notices/latest`, 보안 설정(읽기 공개), Swagger 설명(11. 공지사항)

## 화면
- [x] T009 [테스트] Vitest: 탭·꼬리표 글자·고정 표시, 관리자에게만 글쓰기 버튼, 입력 검사(번호 모양), 새 글 점(처음 방문 없음, 새 글 있으면 있음, 목록을 보면 사라짐)
- [x] T010 `/notices`, `/notices/:id`, `/notices/new`, `/notices/:id/edit`, 머리글 "공지" + 점, 바닥글 링크

## 마무리
- [x] T011 CHANGELOG `[0.6.0] - 준비 중`(화면·API·운영), 서버·화면 버전 0.6.0, REST Docs에 공지 흐름
- [x] T012 `mvn -Pdocs verify`, `npm run lint && npm test && npm run build`, Playwright로 화면 확인(넓은 화면·휴대폰 어둡게): 종류 고르기 글자가 세로로 쪼개지던 것, 읽기 화면 바깥 틀, 링크 색을 고침
- [ ] T013 사용자: 로컬에서 관리자 지정(아래) 후 화면 확인. 배포 뒤 배포 DB에서도 지정
- [ ] T014 릴리스 0.6.0 때 첫 업데이트 글 초안(CHANGELOG를 쉬운 말로) → 관리자 게시

## 사용자가 할 일 (T013)
로컬 DB에서 내 계정을 관리자로:
```bash
docker compose exec postgres psql -U myblog -d myblog -c "UPDATE users SET role='admin' WHERE lower(email)=lower('내 이메일') AND deleted_at IS NULL;"
```
