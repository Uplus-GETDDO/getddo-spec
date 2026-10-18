# 관리자 일별 집계 API

- 상태: R1 구현(백엔드 GD-231). 경로와 필드는 프론트 요청 문서의 제안을 따랐고 담당자 확인 전
- 공통 계약: [공통 API 계약](common.md)
- 갱신일: 2026-10-18 — 프론트 요청 R1을 팀 합의로 한 API로 구현

관리자 대시보드가 목록 API를 `size=1`로 호출해 `totalElements`를 세던 방식을 대신한다. 요청 문서는 응모·응모권·차단 집계로 나눠도 된다고 했지만, 팀 합의(2026-10-18)로 한 API로 구현했다.

| ID | 메서드·경로 | 요청 | 성공 | 주요 오류 |
| --- | --- | --- | --- | --- |
| AS01 | `GET /admin/stats/daily` | 필수 `from`,`to`(KST 날짜) | 200 `DailyStats` | 400 기간 오류 `STATS-001`, 403 `STATS-002` |

`ID AS01`은 이 문서에서 처음 붙인 번호이며 확정 전이다.

`DailyStats`: `days:{date:date,entrantCount:long,entryCount:long,activeUserCount:long,grantedTickets:{ATTENDANCE:long,MISSION:long,GAME:long},usedTickets:long,blockedCount:long}[]`, `totalUserCount:long`.

**규칙**
- 활성 관리자만 호출할 수 있고 읽기 전용이다. `from`·`to`는 KST 날짜(`YYYY-MM-DD`, 양끝 포함)로 둘 다 필수이고 `to`가 `from`과 같거나 이후이며 최대 31일이다. 누락은 400 `COMMON-002`, 형식 오류는 400 `COMMON-005`, 기간 규칙 위반은 400 `STATS-001`이다.
- `days`는 기간의 모든 날짜를 오름차순으로 담고 활동이 없는 날은 0으로 채운다. `grantedTickets`는 세 키를 항상 담는다.
- `entrantCount`는 그날 응모에 접수된 사용자 수(중복 제거)이고 응모권을 쓰지 않는 이벤트의 응모도 포함한다. `entryCount`는 접수된 응모 건수다. 거절된 응모는 세지 않는다.
- `activeUserCount`는 그날 응모, 출석, 미션 제출, 게임 시작 중 하나 이상을 한 사용자 수(중복 제거)다. 적립은 포함하지 않는다.
- `grantedTickets`는 그날 지급된 응모권 장수를 지급 근거별로 나눈 값이고 등급은 합산한다. `usedTickets`는 그날 응모에 사용된 장수이며 반환은 빼지 않는 총량이다.
- `blockedCount`는 그날 발생한 어뷰징 탐지 기록 수다. 지급 전 차단을 따로 저장하지 않아 탐지 기록 기준으로 센다. 검토 상태는 구분하지 않으며 어뷰징 구현 방식이 정해지면 정의를 다시 맞춘다.
- `totalUserCount`는 조회 시점의 활성 일반 사용자 수이고 날짜별 값이 아니다.

백엔드 구현 세부(SQL 기준, 판단과 제약, 검증)는 백엔드 저장소 `docs/02-api/admin-daily-stats.md`에서 관리한다.
