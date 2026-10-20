# Contracts: 글결 MCP OAuth (023)

주소의 `<issuer>`는 설정 `mcp.oauth.issuer`(예: `https://geulgyeol.java21.net`). OAuth 끝점의 오류는 RFC 6749 모양 `{ "error", "error_description" }`.

## 1. 메타데이터 (누구나, CORS)
- `GET /.well-known/oauth-protected-resource/api/v1/mcp` (그리고 `/.well-known/oauth-protected-resource`) →
  `{ "resource": "<issuer>/api/v1/mcp", "authorization_servers": ["<issuer>"], "scopes_supported": ["read","draft"], "bearer_methods_supported": ["header"], "resource_name": "글결" }`
- `GET /.well-known/oauth-authorization-server` →
  `{ "issuer", "authorization_endpoint": "<issuer>/oauth/authorize", "token_endpoint": "<issuer>/oauth/token", "registration_endpoint": "<issuer>/oauth/register", "revocation_endpoint": "<issuer>/oauth/revoke", "response_types_supported": ["code"], "grant_types_supported": ["authorization_code","refresh_token"], "code_challenge_methods_supported": ["S256"], "token_endpoint_auth_methods_supported": ["none"], "scopes_supported": ["read","draft"] }`
- MCP 401: `WWW-Authenticate: Bearer resource_metadata="<issuer>/.well-known/oauth-protected-resource/api/v1/mcp"`

## 2. 동적 등록
- `POST /oauth/register` JSON `{ "client_name", "redirect_uris": [...], "token_endpoint_auth_method"?, "grant_types"?, "response_types"? }` → 201
  `{ "client_id", "client_id_issued_at", "client_name", "redirect_uris", "grant_types": ["authorization_code","refresh_token"], "response_types": ["code"], "token_endpoint_auth_method": "none" }` · 400 `invalid_redirect_uri` · `invalid_client_metadata`

## 3. 인가
- `GET /oauth/authorize?response_type=code&client_id&redirect_uri&code_challenge&code_challenge_method=S256&state&scope?&resource?`
  → 302 `/connect/authorize?request=<id>` · 모르는 클라이언트/돌아갈 주소면 302 `/connect/authorize?error=invalid_client` · 그 밖에는 302 `<redirect_uri>?error=...&state=...`
- `GET /api/v1/oauth/requests/{id}` (로그인) → `{ "clientName", "redirectHost", "scope" (요청한 범위, 없으면 null), "expiresAt" }` · 401 · 404(지났거나 없음)
- `POST /api/v1/oauth/requests/{id}/approve` `{ "scope": "read"|"draft" }` (로그인, CSRF) → `{ "redirectTo": "<redirect_uri>?code=...&state=...&iss=<issuer>" }`
- `POST /api/v1/oauth/requests/{id}/deny` → `{ "redirectTo": "<redirect_uri>?error=access_denied&state=...&iss=<issuer>" }`

## 4. 토큰 (form, CORS)
- `POST /oauth/token` `grant_type=authorization_code&code&redirect_uri&client_id&code_verifier&resource?` →
  `{ "access_token": "go_...", "token_type": "Bearer", "expires_in": 3600, "refresh_token": "gr_...", "scope" }` (`Cache-Control: no-store`)
- `grant_type=refresh_token&refresh_token&client_id` → 같은 모양(갱신 토큰도 새것)
- 오류 400 `invalid_grant` · `invalid_request` · `unsupported_grant_type` · `invalid_target`, 401 `invalid_client`
- `POST /oauth/revoke` `token&client_id` → 200 (모르는 토큰도 200)

## 5. 연결된 앱
- `GET /api/v1/me/connected-apps` → `{ "items": [ { "grantId", "clientName", "scope", "createdAt", "lastUsedAt" } ], "max": 10 }`
- `DELETE /api/v1/me/connected-apps/{grantId}` → 204 · 404
