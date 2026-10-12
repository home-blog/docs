# Implementation Plan: 읽기 편하게·널리 퍼지게 (v0.3.0)

**Spec**: [spec.md](spec.md) · **Branch**: `feat/v0.3-reading-sharing`

## 쉬운 설명
- 다크 모드·목차는 **화면만** 바뀐다.
- 원고지 매수는 서버가 글자를 세어 목록·상세 응답에 숫자 두 개를 더 붙인다.
- 미리보기 카드·RSS·사이트 지도는 **로봇이 읽는 것**이다. 로봇은 화면의 자바스크립트를 실행하지 않으므로, 서버가 HTML·XML을 직접 만들어 준다.

## 화면 (frontend)
| 기능 | 어떻게 | 상태 |
|---|---|---|
| 다크 모드 | `index.css` 토큰을 어두운 값으로 한 벌 더(`[data-theme="dark"]`, 그리고 고르지 않았으면 `prefers-color-scheme: dark`). `index.html` 머리의 짧은 스크립트가 그리기 전에 저장된 값을 붙여 번쩍임을 막는다. 고른 값은 `localStorage` | 가안 |
| 목차 | 글이 그려진 뒤 본문의 `h1~h3`을 찾아 `id`를 붙이고 목차를 만든다(마크다운 해석 결과 그대로라 어긋나지 않음). 지금 위치는 `IntersectionObserver` | 가안 |
| 원고지 매수 | 서버가 준 `sheets`, `readMinutes`를 보여 준다 | |
| RSS 찾기 | 블로그 화면에서 `<link rel="alternate" type="application/rss+xml">` (React 19가 `head`로 올림)와 링크 | |

## 서버 (backend, `post` 모듈)
| 기능 | 어떻게 | 상태 |
|---|---|---|
| 원고지 매수 | `ManuscriptMeasure`: `PostPreview.plainText`의 코드 포인트 수로 계산. 설정 `post.manuscript.chars-per-sheet: 200`, `chars-per-minute: 500` | 가안 |
| 미리보기 카드 | `GET /og/`, `/og/blog/{id}`, `/og/posts/{id}`가 `og:`·`twitter:` 메타 태그를 넣은 작은 HTML을 준다. 볼 수 없는 글은 기본 카드. 조회수는 세지 않는다 | |
| RSS | `GET /rss.xml`, `/blog/{id}/rss.xml` (RSS 2.0, `XMLStreamWriter`로 이스케이프) | |
| 사이트 지도 | `GET /sitemap.xml`, `/robots.txt` | |
| 절대 주소 | 요청 주소에서 만든다(배포는 바깥 Nginx가 넘긴 `Host`·`X-Forwarded-Proto`, `forward-headers-strategy: native`) | |
| 보안 | 위 주소는 로그인 없이 GET만 허용 | |

## 화면 컨테이너 Nginx (배포)
- `/rss.xml`, `/sitemap.xml`, `/robots.txt`, `/blog/{id}/rss.xml`은 서버로 넘긴다.
- **미리보기 로봇**(카카오톡·디스코드·슬랙·페이스북·X·텔레그램·라인 등, User-Agent로 구분)이 `/`, `/blog/{id}`, `/posts/{id}`를 열면 서버의 `/og/...`로 넘긴다. 사람과 검색엔진(Google 등은 자바스크립트를 실행함)은 지금처럼 화면을 받는다.
- 개발(Vite)에서는 RSS·sitemap·robots 주소만 서버로 넘긴다. 미리보기 카드는 서버의 `/og/...`를 직접 열어 확인한다.

## 기본 카드 이미지
- `frontend/public/og-default.png` (1200×630, 원고지 무늬 + "글결"). 화면에서 만든 그림을 저장해 둔다.

## 설정값
| 키 | 값 | 상태 |
|---|---|---|
| `post.manuscript.chars-per-sheet` | 200 | 가안 |
| `post.manuscript.chars-per-minute` | 500 | 가안 |
| `share.rss-size` | 20 | 가안 |
| `share.sitemap-max-posts` | 5000 | 가안 |
