# Research: 글결 MCP OAuth (023)

`가안`은 추천대로 정한 것(2026-10-20 사용자 "끝까지 다 해"). 바꾸고 싶으면 그 줄만 고친다.

| ID | 정할 것 | 고른 것 | 다른 선택지 | 이유 |
|---|---|---|---|---|
| D-1 | 인가 서버 | 글결 서버 안에 작은 OAuth 2.1 인가 서버를 직접 둔다(인가 코드 + PKCE S256, 갱신 토큰 돌리기, 동적 등록) `가안` | Spring Authorization Server | MCP 클라이언트는 **초기 토큰 없이 익명으로 동적 등록**하는데 SAS의 등록 끝점은 초기 토큰을 요구하고, 위임장(React 화면)·세션 로그인과 잇는 데 손이 더 간다. 필요한 흐름이 좁아(공개 클라이언트·code·refresh) 직접 두는 쪽이 테스트로 다 덮기 쉽다 |
| D-2 | 토큰 모양 | 무작위 불투명 토큰(`go_`/`gr_` 접두어), DB에는 SHA-256만 `가안` | JWT | 끊으면 바로 막혀야 한다(정지·비밀번호 변경). 020 열쇠와 같은 방식 |
| D-3 | 짧은 값 | 인가 요청(10분)·허락 코드(5분)는 Redis `가안` | DB | 잃어도 되는 짧은 값(헌법 1.1.0). Redis가 멈추면 연결 시작만 503 |
| D-4 | 저장 | 표 `oauth_client`(등록한 앱), `oauth_grant`(회원이 허락한 연결: 범위·갱신 토큰 해시·옛 갱신 토큰 해시), `oauth_access_token`(1시간) `가안` | access_key에 섞기 | 열쇠 5개 한도·목록과 섞이지 않게 |
| D-5 | 끊기 | 연결을 지우면 출입증도 CASCADE. 비밀번호 변경·탈퇴·정지 때 `AccessKeyService.revokeAll`이 OAuth 연결도 끊는다 `가안` | | 한곳에서 |
| D-6 | 주소 | `/.well-known/oauth-protected-resource(/api/v1/mcp)`, `/.well-known/oauth-authorization-server`, `/oauth/register`, `/oauth/authorize`, `/oauth/token`, `/oauth/revoke`. 화면 위임장 `/connect/authorize`. 사이트 주소(issuer)는 설정 `mcp.oauth.issuer`(기본은 `SITE_URL`) `가안` | | 소셜 로그인의 `/oauth2/`와 겹치지 않게 |
| D-7 | CORS | `.well-known`·`/oauth/register|token|revoke`만 모든 출처 허용(쿠키 없음) `가안` | | 브라우저에서 도는 MCP 클라이언트(Inspector 등)도 붙게. 세션 쿠키를 쓰는 API는 그대로 같은 출처만 |
| D-8 | 모양 | 위임장: 원고지 칸 표(누가·무엇을·언제까지), 허락하면 붉은 인장이 쾅, 거절은 먹선 `가안` | 일반 동의 화면 | 원고지·도장 콘셉트 |
