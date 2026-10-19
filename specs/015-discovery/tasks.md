# Tasks: 발견 (015)

테스트를 먼저 쓴다(헌법·테스트 전략). [P]는 다른 일과 같이 해도 되는 것.

## 바탕
- [x] T001 Flyway V17 `sentence_clip`, 설정 `stats.search-trend.*`·`explore.today-issue.*`·`community.clip.*`(기능마다 모듈의 설정 옆에), 오류 코드

## US1 인기 검색어
- [x] T002 [테스트] S1-1~S1-6: 결과 있는 검색만 센다, 10분 중복, 검색어 맞추기, 금지어, 순위·변화, Redis 멈춤
- [x] T003 `SearchTrends`(세기·순위), 검색 서비스에서 세기, `GET /api/v1/search/trending`
- [x] T004 화면: 첫 화면 옆칸·검색 결과 화면의 인기 검색어(▲▼ NEW)

## US2 오늘의 이슈
- [x] T005 [테스트] S2-1~S2-3: 오늘+어제 조회 합 순, 블로그당 1개, 볼 수 없는 글 빼기, 3개
- [x] T006 `PostFeedService.todayIssue`(글 모듈, research D-9), `GET /api/v1/posts/today-issue`
- [x] T007 화면: 첫 화면 맨 위 큰 카드 1 + 작은 카드 2

## US3 문장 수집·필사 노트
- [x] T008 [테스트] S3-1~S3-8: 401·404, 길이, 본문에 있는 문장만, 같은 문장 다시, 한도, 남의 것 404, 모은 사람 수(주인만), 탈퇴하면 지움
- [x] T009 `SentenceClip` 표·서비스·API(모으기·목록·지우기·모은 사람 수)
- [x] T010 화면: 본문에서 문장을 고르면 "수집" 단추, 마이페이지 필사 노트(원고지 칸 카드), 글 주인에게 모은 사람 수

## 마무리
- [x] T011 014 T022(몇 일이나·몇 일간의) 같이 넣기
- [x] T012 CHANGELOG `[0.10.0]`, 버전, Swagger·REST Docs, V18 0.10.0 업데이트 공지, README
- [ ] T013 `mvn verify`, `npm run lint && npm test && npm run build`, Playwright
- [ ] T014 PR → 릴리스 0.10.0 → 배포 → 다시 합침
