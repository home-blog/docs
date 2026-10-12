# 계약: 소셜 로그인 (010)

모든 API 주소는 `/api/v1/…`. 바꾸는 요청은 CSRF 헤더(`X-XSRF-TOKEN`)가 필요하다. 오류 모양은 001 `공통 약속`과 같다.
`{provider}`는 `github`, `google`, `kakao`, `naver` 중 하나.

## 1. 쓸 수 있는 회사 — `GET /api/v1/auth/social/providers` (누구나)
`200 { "providers": ["github", "kakao"] }` — 열쇠가 들어 있는 회사만, 정해진 순서(github, google, kakao, naver).

## 2. 회사 화면으로 — `GET /oauth2/authorization/{provider}` (브라우저 이동)
Spring Security가 회사 화면으로 보낸다. 회사는 `GET /login/oauth2/code/{provider}`로 돌려보내고, 서버는 결과에 따라 화면 주소로 보낸다.

| 결과 | 가는 곳 |
|---|---|
| 연결된 회원, 또는 인증된 같은 이메일 회원(자동 연결) | `/` (로그인됨) |
| 처음 사용자 | `/signup/social` (로그인 안 됨, 세션에 가입 대기) |
| 연결하기 성공 | `/mypage?linked={provider}` |
| 연결하기 실패 | `/mypage?linkError={코드}` (`SOCIAL_ACCOUNT_IN_USE`, `SOCIAL_ALREADY_LINKED`, `SOCIAL_LOGIN_FAILED`) |
| 회사에서 취소·오류 | `/login?socialError=SOCIAL_LOGIN_FAILED` |

## 3. 가입 대기 보기 — `GET /api/v1/auth/social/pending`
`200 { "provider": "github", "email": "a@b.com" | null, "emailVerified": true, "nickname": "제안" | null, "expiresInSeconds": 540 }`
없거나 시간이 지났으면 `404 SOCIAL_PENDING_NOT_FOUND` "※ 시간이 지났습니다. 소셜 로그인을 다시 해 주세요".

## 4. 소셜 가입 — `POST /api/v1/auth/social/signup`
요청 `{ "nickname": "...", "email": "...", "verificationToken": "..." }`
- 인증된 이메일이 있는 대기면 `email`, `verificationToken`은 보내지 않는다(보내도 무시).
- 아니면 001 이메일 인증(`/auth/email-verifications`, `/confirm`)을 마친 `email`과 `verificationToken`이 필요하다.

`201 { "member": { "id": 1, "nickname": "..." } }` — 바로 로그인된다(세션 쿠키).
오류: `400` 닉네임·이메일 형식, `403 EMAIL_NOT_VERIFIED`, `409 EMAIL_ALREADY_REGISTERED`(인증 안 된 이메일이 기존 회원과 같을 때 문구: "이미 가입된 이메일입니다. 그 계정으로 로그인한 뒤 마이페이지에서 연결해 주세요"), `409 NICKNAME_ALREADY_USED`, `409 SOCIAL_ACCOUNT_IN_USE`, `404 SOCIAL_PENDING_NOT_FOUND`.

## 5. 가입 대기 취소 — `DELETE /api/v1/auth/social/pending` → `204`

## 6. 내 연결 — `GET /api/v1/account/social` (로그인)
`200 { "hasPassword": false, "providers": ["github", "kakao"], "linked": [{ "provider": "github", "linkedAt": "2026-10-12T..." }] }`

## 7. 연결하기 시작 — `POST /api/v1/account/social/{provider}/link` (로그인) → `204`
세션에 "이 회원에 연결" 표시를 남긴다(10분). 화면은 이어서 `/oauth2/authorization/{provider}`로 이동한다. 모르는 회사는 `404 SOCIAL_PROVIDER_UNKNOWN`.

## 8. 연결 해제 — `DELETE /api/v1/account/social/{provider}` (로그인) → `204`
`404 SOCIAL_NOT_LINKED`, `409 LAST_LOGIN_METHOD`.

## 바뀐 기존 API (v1 그대로, 깨지지 않는 변경)
- `POST /api/v1/account/password`: 비밀번호가 없는 회원은 `currentPassword`를 보내지 않아도 된다(비밀번호 만들기).
- `POST /api/v1/account/withdrawal`: 비밀번호가 없는 회원은 `password`를 보내지 않아도 된다(동의만).
