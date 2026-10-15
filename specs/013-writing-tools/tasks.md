# Tasks: 글쓰기 도구 (v0.7.0)

**Input**: [spec.md](spec.md), [research.md](research.md), [data-model.md](data-model.md), [contracts/writing-tools-api.md](contracts/writing-tools-api.md) · 코드 브랜치 `feature/v0.7-writing-tools` → 코드 PR #72 `develop` merge (ac9619e, 2026-10-15, 리뷰 반영으로 연재 동시 요청 테스트 3개 더함, 서버 새 테스트 71개·화면 37개, `mvn verify` 610개·화면 148개 통과, Playwright 확인)

테스트 전략(`docs/3-설계/테스트-전략.md`)대로 **테스트 작업을 구현 앞에** 둔다.

## 바탕
- [x] T001 Flyway V12(`post_draft`, `series`, `series_post`, `post_image.post_draft_id`), ERD E-10
- [x] T002 설정 `writing.*`, `series.*`, 오류 코드

## US1 글쓰기 잔디
- [x] T003 [테스트] S1-1~S1-5: 날짜별 수, 비공개 제외(주인도), 365일 경계, 한국 0시 10분, 없는 블로그
- [x] T004 `GET /blogs/{id}/activity`
- [x] T005 [테스트] 화면: 5단계 진하기, 칸 설명 글자
- [x] T006 화면: 블로그 머리 아래 잔디(원고지 칸)

## US2 연재
- [x] T007 [테스트] S2-1~S2-7: 이름·소개 경계값, 50개, 옮기기, 보이는 글로 화 번호, 남의 것 404, 연재·글 지우기, 순서 검사
- [x] T008 연재 API(만들기·고치기·지우기·글 넣기·빼기·순서, 목록·한 개·글의 연재), 글 지울 때·블로그 닫을 때 정리
- [x] T009 [테스트] 화면: 이전·다음 화, 화 번호
- [x] T010 화면: 블로그 관리 "연재", 글쓰기·수정의 연재 고르기, 글 읽기 연재 표시, 블로그 화면 연재 목록·연재 화면

## US3 임시 저장
- [x] T011 [테스트] S3-1~S3-6: 남에게 404, 어디에도 안 나옴, 다시 저장, 발행(검사·한 번만), 51번째, 이미지 정리 제외
- [x] T012 임시 저장 API, 발행, 이미지 연결(`DraftContentSavedEvent`, 지울 때는 DB가 떼어 줌), 블로그 닫을 때 정리
- [x] T013 [테스트] 화면: 자동 저장(바뀐 것만, 간격), 저장 시각 표시
- [x] T014 화면: 글쓰기 "임시 저장"·자동 저장, 글 관리 "임시 저장" 칸, 이어 쓰기

## US4 예약 발행
- [x] T015 [테스트] S4-1~S4-6: 시각 경계, 시각 전·후 보이기, 취소, 두 벌 동시(한 번만), 실패 시 예약 풀기, 예약 글의 분류 지우기 거절
- [x] T016 예약 API, 1분마다 올리는 작업(`SKIP LOCKED`), 분류 글 개수에 예약 포함
- [x] T017 화면: 글쓰기 "예약" 시각 고르기, 글 관리에 예약 표시·취소

## 마무리
- [x] T018 CHANGELOG `[0.7.0] - 준비 중`, 버전 0.7.0, Swagger 묶음, REST Docs 흐름
- [x] T019 `mvn verify`, `npm run lint && npm test && npm run build`, Playwright로 화면 확인
- [ ] T020 사용자: 로컬에서 확인 후 "배포해줘"
