# Tasks: AI 빨간 펜과 관련 글 (v0.8.0)

**Input**: [spec.md](spec.md), [research.md](research.md), [data-model.md](data-model.md), [contracts/ai-api.md](contracts/ai-api.md) · 코드 브랜치 `feature/v0.8-ai` → 코드 PR #76 (서버 테스트 685개, 화면 169개, Playwright로 가짜 AI 확인)

테스트 전략대로 **테스트 작업을 구현 앞에** 둔다.

## 바탕
- [x] T001 의존성(anthropic-java), 설정 `proofread.*`·`related.*`, 오류 코드, 배포 비밀값(`ANTHROPIC_API_KEY`, `VOYAGE_API_KEY`) 전달
- [x] T002 Flyway V14 `post_embedding`, ERD E-11

## US1 빨간 펜
- [x] T003 [테스트] 교정 검사기: 앞 글자로 위치 찾기, 같은 원문 여러 번, 못 찾음·겹침·같은 말 버림, 코드 블록·인라인 코드·링크 주소 버림, UTF-16 위치
- [x] T004 [테스트] S1-1~S1-6: 401, 빈 본문·10,001자 400, 하루 20번 → 429, 같은 본문 캐시(횟수 그대로), AI 실패 503(횟수 그대로), 꺼짐 503
- [x] T005 교정 API, Claude 교정기(구조화된 출력, 거절 대체), 가짜 교정기(개발), 빨간 펜 상태 `GET /me/proofread`(contracts 2)
- [x] T006 [테스트] 화면: 받아들이기 위치 계산, 교정부호 종류
- [x] T007 화면: "빨간 펜" 단추·동의, 교정지(원고지 위 교정부호), 받아들이기·넘기기·모두, 퇴고 도장

## US2 비슷한 글
- [x] T008 [테스트] S2-1~S2-6: 404, 볼 수 없는 글 빼기, 4개·블로그당 2개, 뜻 순서, 태그 대체, 저장 뒤 임베딩·본문 그대로면 건너뛰기, 정리 작업
- [x] T009 임베딩(Voyage·가짜), 저장 이벤트 뒤 비동기, 10분 정리 작업(잠금), 관련 글 API
- [x] T010 화면: 글 읽기 아래 "이 글과 비슷한 글"

## 마무리
- [x] T011 CHANGELOG `[0.8.0]`, 버전 0.8.0, Swagger 묶음, REST Docs 흐름, V15 업데이트 공지
- [x] T012 `mvn verify`, `npm run lint && npm test && npm run build`, Playwright(가짜 AI)
- [x] T013 배포 (사용자 "알아서 배포까지 다 완료해", 2026-10-19). **2026-10-19 v0.8.0 배포**: 기능 PR #76 develop squash merge → 릴리스 PR #77 main merge commit → 배포 실행 39194893386(확인·배포·릴리스 모두 성공, a·b 두 벌 c5952874c80d healthy, 태그·GitHub 릴리스 `v0.8.0`) → main을 develop에 다시 합침 PR #79. 릴리스 리뷰에서 찾은 배포 막힘(YAML의 따옴표 없는 `off`가 false로 읽혀 prod 서버가 켜지지 않음)을 `ProdSettingsBindingTest`로 재현하고 고쳤다. **AI는 아직 꺼져 있다**: 사용자가 GitHub Secrets에 `ANTHROPIC_API_KEY`, `VOYAGE_API_KEY`를 넣고 Actions `배포`를 번호 없이 한 번 다시 돌리면 켜진다(사용자 할 일)
