# Tasks: 무중단 배포 (v0.5.2)

**Input**: [spec.md](spec.md), [research.md](research.md) · 코드 브랜치 `feature/zero-downtime`

## 학원 Nginx (사용자)
- [x] T001 `~/nginx/geulgyeol.java21.net.conf`에 `upstream`(8410, 8411), `proxy_next_upstream`, `X-Upstream` (2026-10-14 사용자. 처음에 통째로 덮어써 HTTPS가 빠질 뻔해 백업으로 되돌린 뒤 필요한 줄만 고침). 5분 뒤 반영 확인: `x-upstream: 127.0.0.1:8410`, 한 번은 `8411, 8410`(8411이 비어 있어 8410으로 다시 보냄)
- [ ] T002 (선택) 접속 로그에 `log_format geulgyeol_up` 더하기 (`deploy/README.md` "트래픽 보기")

## 코드
- [x] T003 `docker-compose.deploy.yml` 두 벌(web·server a/b), 벌마다 네트워크와 `server` 별명, 메모리 1g, 끌 때 기다림
- [x] T004 `deploy/remote-deploy.sh` 롤링: 화면 입구 먼저 끄기 → 새 이미지 → 그 포트로 확인(API 200, `X-Instance`, 첫 화면) → 15초 → 다음 벌. 실패하면 그 벌만 되돌리고 멈춤. `.slots.env`, `<태그>`로 되돌리기, 두 벌 이전 구성에서 넘어가기
- [x] T005 서버 graceful shutdown 30초, `X-Instance` 머리글(`InstanceHeaderFilter`, 이름 검사), 테스트
- [x] T006 배포 작업 `WEB_PORT_B`, 학원 Nginx 견본·`make-nginx-conf.sh`(두 포트, 로그 모양, 덮어쓰지 않기)
- [x] T007 `deploy/README.md` 무중단 배포·트래픽 보기·학원 Nginx 고치기·되돌리기, `CHANGELOG.md` [0.5.2] 준비 중, 버전 0.5.2

## 확인 (로컬, 학원 Nginx와 같은 설정의 Nginx + 두 벌, 1초에 70번쯤 GET·POST)
- [x] T008 예전 방식 배포: 4,925개 중 1,700개 실패(26초 동안 502)
- [x] T009 두 벌 이전 → 두 벌 전환: 11,248개 중 실패 0 (SC-003)
- [x] T010 롤링 배포: 11,277개 중 2개 실패(벌을 끄는 순간 이미 보낸 POST, research "남은 한계"), GET 실패 0. a 3,717 / b 3,799 (SC-001, SC-004)
- [x] T011 고장 난 버전 배포: 3,780개 중 실패 0, a 벌만 옛 이미지로 되돌아가고 배포 실패로 멈춤 (SC-002)
- [ ] T012 실제 배포 때 사용자가 배포 중 실패 세기 (`deploy/README.md` "트래픽 보기"의 반복 명령)

## 리뷰 반영 (코드 PR #63, 2026-10-14 `develop` merge b8d3708)
- [x] T013 옛 이미지로 되돌리기도 실패하면 그 벌을 끈다 (두 이미지 모두 고장 난 경우 로컬 확인: a 꺼짐, b가 받음)
- [x] T014 되돌릴 옛 이미지와 "바로 전 이미지"를 `.slots.env`가 아니라 실제 컨테이너에서 먼저 읽는다 (끊긴 배포 뒤에도 맞게)

## 사용자 결정 대기
- 벌을 끄는 순간의 POST 502를 없앨지 (화면 컨테이너 호스트 네트워크, research "남은 한계")
- 저장소 Variables `SESSION_COOKIE_SECURE`=`true` (HTTPS가 열렸으므로)
