# Tasks: 글결 MCP (020)

테스트를 먼저 쓴다(헌법·테스트 전략).

## 바탕
- [x] T001 Flyway V27 `access_key`, 설정 `access-key.*`·`mcp.*`, 오류 코드, 새 모듈 `connect`

## US1 개인 열쇠
- [x] T002 [테스트] S1-1~S1-4
- [x] T003 회원 모듈: 열쇠 만들기·목록·끊기, MCP 전용 보안 사슬(Bearer, 세션 없음, Origin 검사), 탈퇴하면 지우기

## US2 MCP 찾기·읽기
- [x] T004 [테스트] 프로토콜(initialize·tools/list·tools/call·ping·알림·405·잘못된 JSON), S2-1~S2-3
- [x] T005 글 모듈 입구 `PostLibrary`(찾기·단락 찾기·읽기·내 글), 통계 입구, connect 모듈의 JSON-RPC 처리기와 도구 5개, 요청 수 제한
- [x] T006 실제 MCP 클라이언트(공식 TypeScript SDK)로 붙어 보기

## US3 임시 저장
- [x] T007 [테스트] S3-1·S3-2, 읽기만 열쇠에는 도구가 없음
- [x] T008 `save_draft`

## 화면
- [x] T009 [테스트] 마이페이지 AI 비서 연결(만들기·한 번만 보이기·복사·끊기)
- [x] T010 화면, 연결 설정 예(Claude Code·Claude Desktop)

## 마무리
- [x] T011 CHANGELOG `[0.15.0]`(우편함 다시 그리기 포함), 버전, REST Docs, V28 공지, README, ERD E-17, 쉬운 설명 `docs/4-가이드/09-MCP와-RAG.md`
- [x] T012 `mvn verify`, `npm run lint && npm test && npm run build`, Playwright
- [ ] T013 PR → 릴리스 0.15.0 → 배포 → 다시 합침
