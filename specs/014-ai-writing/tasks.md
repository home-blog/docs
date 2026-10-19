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

## 0.9.0 돈이 들지 않는 AI (2026-10-19 사용자: "결제를 하지 않는 조건에서 최선을 다해줘", research D-16·D-17)
- [x] T014 [테스트] 규칙 교정기: 고칠 말(맞춤법·띄어쓰기·외래어)과 고치지 않을 말(할수록, 놀려고, 안됐다, 아들께, 홀수도, 별것, 선거 …), 코드·주소 안 그대로, 띄어쓰기를 넘는 말(몇 일, 수 밖에)
- [x] T015 `RuleProofreader`, `Proofreader.mode()`(ai·rules·off), 규칙 교정은 횟수·보관 없음, `proofread.without-key: rules`, 화면은 ai일 때만 동의·남은 횟수
- [x] T016 [테스트] Ollama 요청·답 읽기(개수·차원), 글자 조각 임베딩, 배포 설정 묶기(prod는 rules·태그, Ollama 주소가 있으면 ollama)
- [x] T017 `OllamaEmbedder`(bge-m3), `LocalEmbedder`(개발·시험용), `related.provider` auto = Voyage → Ollama → without-key
- [x] T018 배포: compose `ollama` 프로필(메모리 2GB·CPU 2), `remote-deploy.sh`가 `OLLAMA_ENABLED`·남은 메모리·디스크를 보고 켜고 모델을 뒤에서 받음, `deploy.yml`이 Variables 전달
- [x] T019 V16 0.9.0 업데이트 공지, CHANGELOG `[0.9.0]`, 버전 0.9.0, README·deploy/README·.env.example·REST Docs 설명
- [x] T020 PR → 릴리스 0.9.0 → 배포 → 다시 합침. **2026-10-19 v0.9.0 배포**: 기능 PR #80(리뷰 반영: AI 보내기 동의를 서버가 확인 `aiConsent`·409, 아들께요 그대로, 임베딩 숫자 검사 `Vectors.usable`) → 릴리스 PR #81(리뷰 반영: 규칙의 낱말 경계 `-지 않되`·`호수 밖에`·`몇 일본인`·이름 속 `희안`, `몇 일이`도 며칠로, 로컬 Ollama 포트 127.0.0.1) → 배포 실행 39210501832(확인·배포·릴리스 모두 성공, a·b 두 벌 48a5a13 healthy, 태그·GitHub 릴리스 `v0.9.0`) → main을 develop에 다시 합침 PR #82
- [ ] T021 (사용자, 선택) 서버 여유를 보고 강사님께 여쭌 뒤 Variables `OLLAMA_ENABLED=true` → 배포 다시 돌리기
- [x] T022 (015 0.10.0에 넣음) 규칙 교정: `몇 일` 뒤에 허용한 조사·접미사가 연달아 붙는 꼴(`몇 일이나`, `몇 일간의`)도 며칠로. `몇 일본인`·`몇 일자리` 그대로 두는 회귀 테스트 함께 (코드 PR #82 CodeRabbit 제안, 놓치는 쪽이라 급하지 않음)
