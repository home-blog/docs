# Research: 글마당 (017)

`가안`은 추천대로 정한 것(2026-10-19 사용자 "알아서, 색다르게"). 바꾸고 싶으면 그 줄만 고친다.

| ID | 정할 것 | 고른 것 | 다른 선택지 | 이유 |
|---|---|---|---|---|
| D-1 | 모듈 | 새 모듈 `board`(글마당 글·댓글·채택). 주제 이름은 글 모듈의 맨 위 틀 `TopicCatalog`로 묻고, 이름은 `MemberNames`, 관리자 여부는 회원 모듈에 묻는다: `allowedDependencies = post, user, common` `가안` | 기존 `community` 모듈에 넣기 | `community`는 블로그 글의 좋아요·신고·문장 수집이다. 글마당은 블로그와 따로 서는 게시판이다 |
| D-2 | 주제 | 블로그 글과 같은 `topic` 표(FK). 글마당만의 주제는 두지 않는다 `가안` | 글마당 주제 따로 | `요구사항.md` 3.7 탭이 블로그 주제와 같다 |
| D-3 | 조회 수 | 통계 모듈이 `BoardPostViewedEvent`를 듣고 블로그 글과 같은 방문자 키·30분 표시(Redis)로 한 번만 센 뒤 글마당 모듈의 틀 `BoardViewCounter`로 `views + 1` `가안` | 글마당 모듈이 Redis 직접 | 같은 사람 판단(쿠키·회원)이 통계 모듈 한곳에 있다. 통계 모듈 의존에 `board`를 더한다 |
| D-4 | 댓글 수 | `board_post.comment_count`를 댓글 쓰기·지우기와 같은 트랜잭션에서 `+1/-1`(DB에서 바로) `가안` | 목록마다 셈 | 목록이 표 하나만 읽는다 |
| D-5 | 채택 | `board_post.accepted_comment_id`(FK board_comment, ON DELETE SET NULL). 채택한 댓글은 지울 수 없다 `가안` | 댓글에 표시 칸 | 글당 하나가 표 구조로 지켜진다 |
| D-6 | 너무 빨리 올리기 | Redis 키 `board:rate:{post|comment}:{회원}`(창 1분, 횟수 설정) `가안` | DB로 셈 | 잃어도 되는 짧은 값(헌법 1.1.0 Redis 규칙) |
| D-7 | 지우기 | 글·댓글 모두 진짜 지운다(`DELETE`), 댓글은 FK CASCADE `가안` | 지움 표시 | 지운 글을 되살리는 요구가 없다. 탈퇴는 줄을 남기고 이름만 "탈퇴한 회원"(회원 줄이 남으므로 그대로 된다) |
| D-8 | 검색 | `GET /api/v1/board/posts?q=`: 단어마다 `제목·본문·주제 이름·닉네임 ILIKE` AND, 같은 표 목록 쿼리에 조건을 더한다. 바인딩 값만(문자열 이어 붙이지 않음) `가안` | 전문 검색 | 글마당 글 수가 적다. 블로그 글 검색과 같은 방식 |
| D-9 | 우편함 소식 | `notification.kind`에 `board_comment`, `board_accept`를 더하고 `board_post_id`(FK ON DELETE SET NULL) 칸을 더한다. 구독 모듈이 `BoardCommentWrittenEvent`, `BoardAnswerAcceptedEvent`를 커밋 뒤 듣는다 `가안` | 글마당 모듈이 직접 | 우편함은 한곳(016)에 둔다. 구독 모듈 의존에 `board`를 더한다 |
| D-10 | 모양 | 표는 원고지 칸 선 머리, 물음 꼬리표는 먹색 테두리, 채택은 붉은 도장(글 끝 도장과 같은 모양), 해결된 물음은 목록에 작은 `해결` 도장 `가안` | 체크 아이콘 | 원고지·도장 콘셉트 |
