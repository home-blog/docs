# Tasks: 소셜 로그인 (v0.5.0)

**Input**: [spec.md](spec.md), [research.md](research.md), [contracts/social-api.md](contracts/social-api.md) · 코드 브랜치 `feature/v0.5-social-login` (2026-10-12 커밋 7c22a14 push, PR은 사용자가 로컬에서 확인한 뒤)

## 바탕
- [x] T001 Flyway V10: `users.password` NULL 허용, `user_social_account` 표(회원·회사·회사 쪽 번호·연결 시각, 중복 불가 둘). `ERD-변경-요청.md` E-8 (Crowfoot 666 반영은 남음: 배포 DB에 V10이 생긴 뒤 DB → 문서 동기화, 정할 것 42)
- [x] T002 설정 `social.*`(회사별 열쇠는 환경 변수, 가입 대기 10분), 의존성 `spring-boot-starter-security-oauth2-client`, 열쇠가 있는 회사만 만드는 회사 목록
- [x] T003 회사 응답을 `SocialProfile`로 맞추기(GitHub 이메일 목록, Kakao·Naver 중첩 값, D-2 인증 판정), 회사 토큰 저장 안 함(D-7)

## US1·US3 로그인과 자동 연결
- [x] T004 성공 처리기: 연결된 회원 → 로그인, 인증된 같은 이메일 → 연결 후 로그인, 처음 사용자 → 가입 대기, 연결하기 모드 → 연결. 실패 처리기
- [x] T005 `GET /auth/social/providers`, 보안 설정(oauth2Login, 회사 주소)
## US2 소셜 가입
- [x] T006 `GET·DELETE /auth/social/pending`, `POST /auth/social/signup`(인증된 이메일 / 001 인증번호), 가입 뒤 로그인
## US4 연결 관리
- [x] T007 `GET /account/social`, `POST /account/social/{p}/link`, `DELETE /account/social/{p}`(마지막 방법 막기)
- [x] T008 비밀번호 없는 회원: 비밀번호 만들기, 이메일 로그인 거절은 그대로
## US5 탈퇴
- [x] T009 탈퇴 때 연결 지우기, 비밀번호 없는 회원은 동의만
## 화면
- [x] T010 로그인·가입 창의 회사 버튼, `/signup/social` 화면, 마이페이지 "연결된 계정"·"비밀번호 만들기"·탈퇴, Vite 프록시
## 배포·기록
- [x] T011 화면 Nginx `/oauth2/`, `/login/oauth2/`, 배포 작업·compose에 `SOCIAL_*` 환경 변수, `.env.example`, `deploy/README.md` 앱 등록 순서
- [x] T012 `CHANGELOG.md` [0.5.0] 준비 중, 서버·화면 버전 0.5.0
- [x] T013 테스트(연결·자동 연결·가입·해제·탈퇴·연결하기 만료, 회사 응답 판정), `mvn verify`, lint, build, actionlint, Playwright(버튼·소셜 가입 두 경우·연결 카드·휴대폰). 실제 회사 화면은 앱 등록 뒤 사용자가 로컬에서 확인
- [ ] T014 사용자: 회사별 앱 등록과 열쇠 넣기 (아래)

## 사용자가 할 일 (T014)
콜백 주소는 로컬 `http://localhost:5173/login/oauth2/code/{회사}`, 배포 `http://geulgyeol.java21.net/login/oauth2/code/{회사}`.
1. **GitHub**: Settings → Developer settings → OAuth Apps → New. 콜백을 하나만 받으므로 로컬용·배포용 앱을 따로 만든다.
2. **Kakao**: Kakao Developers → 애플리케이션 추가 → 카카오 로그인 켜기, Redirect URI 두 개, 동의 항목에서 닉네임·이메일(이메일은 비즈 앱이어야 필수 동의 가능, 없으면 이메일 인증번호로 받는다). 보안 → Client Secret 켜기.
3. **Naver**: Naver Developers → 애플리케이션 등록 → 네이버 로그인, 이메일·별명, 서비스 URL과 Callback URL.
4. **Google**: Google Cloud Console → OAuth 동의 화면 → 사용자 인증 정보 → 웹 애플리케이션. 승인된 리디렉션 URI. 배포 주소는 HTTPS가 붙은 뒤에 넣는다.
5. 열쇠를 로컬 `.env`(`SOCIAL_GITHUB_CLIENT_ID` 등)와 GitHub Secrets에 넣는다. 공개 저장소에는 적지 않는다.
