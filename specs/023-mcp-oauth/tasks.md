# Tasks: 글결 MCP OAuth (023)

테스트를 먼저 쓴다(헌법·테스트 전략).

## 바탕
- [x] T001 Flyway V33(`oauth_client`, `oauth_grant`, `oauth_access_token`), 설정 `mcp.oauth.*`(issuer, 토큰 수명, 연결 한도), 오류 응답 모양

## US1 커넥터로 붙이기
- [x] T002 [테스트] S1-1~S1-7 (메타데이터, 등록 검사, PKCE 맞음/틀림, 코드 두 번, 다른 클라이언트·주소, 갱신 돌리기·옛 갱신 토큰 재사용, resource, 읽기만 범위)
- [x] T003 메타데이터·등록·인가·토큰·폐기 끝점, MCP 보안 사슬에 OAuth 출입증, 401 `resource_metadata`, CORS
- [x] T004 화면 위임장 `/connect/authorize`(로그인 먼저, 범위 고르기, 허락·거절, 인장 애니메이션)

## US2 연결된 앱 관리
- [x] T005 [테스트] S2-1~S2-3 (남의 연결 404, 끊은 뒤 401, 10개 한도, 비밀번호·정지·탈퇴 때 끊김)
- [x] T006 연결된 앱 API, 마이페이지 목록·끊기, "Claude 앱에 붙이기" 탭
- [x] T007 실제 MCP SDK 클라이언트의 OAuth 흐름으로 끝까지 확인(등록 → 인가 → 토큰 → 도구 → 갱신)

## 마무리
- [x] T008 화면 nginx·vite 프록시에 `/.well-known/`·`/oauth/`, CHANGELOG `[0.17.0]`, 버전, V34 공지, REST Docs, 쉬운 설명 `09-MCP와-RAG.md`에 커넥터, ERD E-20
- [x] T009 `mvn verify`, `npm run lint && npm test && npm run build`, Playwright, PR → 릴리스 → 배포 → 다시 합침
