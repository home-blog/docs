# API 계약: AI 빨간 펜과 관련 글 (014)

모두 `/api/v1/…`, 깨지지 않는 변경(칸·주소 더하기). 오류는 공통 모양.

## 1. 빨간 펜 `POST /api/v1/me/proofread` (로그인, CSRF)

요청 `{ "content": "마크다운 본문", "aiConsent": true }`. `aiConsent`(0.9.0): 화면에서 "본문이 AI로 보내진다"에 동의했는지. 교정기가 바깥 AI(`mode: ai`)인데 참이 아니면 본문을 보내지 않고 409 `PROOFREAD_CONSENT_REQUIRED`(화면은 동의를 묻고 다시 보낸다). 규칙 교정은 보지 않는다.

응답 200
```json
{
  "corrections": [
    { "start": 12, "end": 17, "kind": "spacing", "original": "할수있다", "replacement": "할 수 있다", "reason": "의존 명사 '수'는 띄어 씁니다" }
  ],
  "remaining": 17,
  "cached": false
}
```
- `start`·`end`는 보낸 본문의 UTF-16 위치(자바스크립트 문자열 위치와 같음), 겹치지 않고 앞에서부터.
- `kind`: `spacing`(띄어쓰기) · `spelling`(맞춤법) · `style`(다듬기).
- 오류: 401, 400 `PROOFREAD_EMPTY`·`PROOFREAD_TOO_LONG`, 429 `PROOFREAD_LIMIT_EXCEEDED`, 503 `PROOFREAD_UNAVAILABLE`.

## 2. 빨간 펜 상태 `GET /api/v1/me/proofread` (로그인)
`{ "enabled": true, "mode": "ai", "dailyLimit": 20, "remaining": 17, "maxLength": 10000 }`. `mode`(0.9.0): `ai`(바깥 AI, 화면이 보내기 동의를 묻고 하루 횟수를 센다), `rules`(서버 안의 규칙 교정, 동의·횟수 없음, `remaining`은 늘 하루 한도), `off`(`enabled: false`, 화면은 단추를 숨긴다).
(처음에는 글쓰기 설정 post-form에 칸을 더하려 했으나, 남은 횟수도 함께 주려고 빨간 펜 모듈의 주소로 따로 뒀다. 글 모듈이 AI 모듈을 몰라도 된다)

## 3. 비슷한 글 `GET /api/v1/posts/{postId}/related` (누구나)
글을 볼 수 없으면 404(글 읽기와 같음).
```json
{ "basis": "meaning", "posts": [ /* 첫 화면 글 모음과 같은 카드 한 줄(FeedRow) */ ] }
```
`basis`: `meaning`(뜻) 또는 `tags`(태그·주제 대체). 없으면 `posts: []`.
