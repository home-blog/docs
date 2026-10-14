# Research: 무중단 배포 (011)

모두 `가안`이다 (2026-10-14). 바꾸고 싶으면 해당 줄만 고치면 된다.

| ID | 정할 것 | 고른 것 (가안) | 다른 선택지 | 이유 |
|---|---|---|---|---|
| D-1 | 무중단 방식 | **앱 두 벌 상시 + 롤링**(이중화) | 블루-그린(평소 한 벌, 배포 때만 두 벌), 카나리 | 강사님 안내("앱 두 개를 Nginx에 연결"). 한 벌이 죽어도 다른 벌이 받는다. 메모리는 서버 한 벌 1GB로 묶는다 |
| D-2 | 입구(로드 밸런서) | **학원 Nginx `upstream`** (127.0.0.1:8410 a, 8411 b) | 우리 입구 Nginx 컨테이너 하나 더, Spring Cloud Gateway | 학원 Nginx가 5분마다 설정을 다시 읽어 우리가 고칠 수 있다. 컨테이너가 늘지 않는다. Spring Cloud Gateway는 여러 서버(MSA)용이라 과하다 |
| D-3 | 꺼진 벌 피하기 | `proxy_next_upstream error timeout http_502 http_503` + `max_fails=1 fail_timeout=10s` | Nginx Plus의 능동 건강 검사(유료) | 무료 Nginx에서 쓸 수 있는 방법. 꺼진 벌에 보낸 요청을 다른 벌로 다시 보낸다 |
| D-4 | 배포 순서 | 한 벌의 화면 입구를 먼저 끄고(포트가 닫혀 Nginx가 다른 벌로) → 서버를 새 이미지로 → 건강해지면 화면 입구 → 그 포트로 직접 확인 → 15초 뒤 다음 벌 | 서버부터 끄기 | 서버부터 끄면 화면 입구가 502로 답해 POST가 실패한다. 15초는 fail_timeout(10초)보다 길게: Nginx가 방금 켠 벌을 다시 쓰기 시작한 뒤에 다른 벌을 끈다 |
| D-5 | 새 버전이 고장 났을 때 | 그 벌만 옛 이미지로 되돌리고 배포를 멈춘다. 처음 켜는 벌이면 꺼 둔다 | 계속 진행 | 고장 난 벌로 손님이 가지 않게. 다른 벌은 손대지 않았으니 그대로 손님을 받는다 |
| D-6 | 끌 때 | Spring graceful shutdown 30초, 컨테이너 기다림 40초, 화면 Nginx는 SIGQUIT | 기본(Docker 10초 뒤 강제 종료) | 처리 중인 요청을 마치고 꺼진다 |
| D-7 | 두 버전이 같이 도는 동안 | DB는 넓히고 → 좁히기(칸 이름 바꾸기·지우기는 릴리스를 나눈다), API는 v1 약속 유지, 세션 클래스의 `serialVersionUID`는 올리지 않는다 | — | 롤링 중에는 옛 서버가 새 표 위에서, 옛 화면이 새 서버와 돈다 |
| D-8 | `@Scheduled` 겹침 | 잠그지 않는다. 지금 있는 이미지 정리는 두 번 돌아도 결과가 같다 | ShedLock 같은 DB 잠금 | 지금은 필요 없는 의존성을 더하지 않는다(헌법 V). 두 번 돌면 안 되는 일이 생기면 그때 잠근다 |
| D-9 | 어느 벌이 받았는지 | 앱이 응답에 `X-Instance: a/b`, 학원 Nginx가 `X-Upstream`과 로그 `upstream=` | 로그만 | 바깥에서 curl 한 줄로 보인다 |
| D-10 | 처음 넘어갈 때 | 비어 있는 b를 먼저 켜고 → 옛 컨테이너(web·server)를 끄고 → a를 켠다 | 한 번 멈추고 바꾸기 | 학원 Nginx에 b(8411)가 이미 있어 이 전환도 끊기지 않는다 |

## 남은 한계

- **벌을 끄는 순간의 POST**: 한 벌의 화면 입구를 끄면 Nginx가 새 연결을 닫는데, Docker 포트 중계(docker-proxy)는 컨테이너가 다 꺼질 때까지 연결을 받았다가 끊는다(0.1초 안팎). 학원 Nginx는 GET은 다른 벌로 다시 보내지만, 이미 보낸 POST는 두 번 저장되지 않도록 다시 보내지 않는다. 로컬에서 1초에 70번쯤 보낼 때 벌마다 POST 1개가 502였다. 없애려면 화면 컨테이너를 호스트 네트워크로 띄워 포트 중계를 거치지 않게 해야 한다(학원 서버에서 되는지 확인 필요). → 사용자 결정 대기
- 학원 서버가 꺼지거나 DB·Redis가 멈추면 두 벌 모두 멈춘다 (애플리케이션 이중화까지만).

## 구현 방식

- `docker-compose.deploy.yml`: `web-a`(127.0.0.1:8410)·`server-a`, `web-b`(8411)·`server-b`, 같이 쓰는 `redis`·`minio`. 벌마다 네트워크(`slot-a`, `slot-b`)를 따로 두고 서버에 `server` 별명을 붙여, 화면 Nginx(`server:8080`)가 같은 벌의 서버만 찾는다. 서버 `mem_limit` 1g(`SERVER_MEMORY`), `stop_grace_period` 40s.
- `deploy/remote-deploy.sh`: 위 D-4·D-5·D-10 순서. 벌마다 이미지 태그를 `.slots.env`(`IMAGE_TAG_A`, `IMAGE_TAG_B`, `PREVIOUS_TAG`)에 남긴다. `remote-deploy.sh <태그>`로 되돌린다. 지금 두 벌과 바로 전 이미지만 남기고 지운다.
- 서버: `server.shutdown: graceful`, `spring.lifecycle.timeout-per-shutdown-phase: 30s`, `instance.name`(`INSTANCE_NAME`)이 있으면 `X-Instance` 머리글(`common/instance/InstanceHeaderFilter`).
- 학원 Nginx 견본 `deploy/nginx-site.conf.template`: `upstream`, `log_format <이름>_up`(upstream·상태·시간), `proxy_next_upstream`, `X-Upstream`. `make-nginx-conf.sh`는 이미 있는 파일을 덮어쓰지 않는다(Certbot이 덧붙인 HTTPS를 지키려고).
