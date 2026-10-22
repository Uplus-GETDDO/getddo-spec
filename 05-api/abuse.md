# 어뷰징 차단 내역 API

- 상태: 2026-10-22 사용자 합의 반영 — AR01/AR02 경로·필드·조회 기준·evidence 세부 계약·업무 오류 코드 채택. 구현·배포는 미확인
- 공통 계약: [공통 API 계약](common.md)
- 도메인 규칙: [응모·어뷰징 규칙](../02-domain/entry.md)
- 갱신일: 2026-10-22 — GD-141 관리자 조회 업무 오류·공통 오류 재사용 계약 확정

AR01/AR02는 아래 응모 자동 차단 이력 조회 계약을 따른다. 차단 정책의 원본은 [응모 요청 일시 차단](../02-domain/entry.md#응모-요청-반복-초과에-따른-일시-차단--gd-141)이다. 게임·출석·미션의 새로운 지급 전 차단 범위와 AR07 사후 처리 계약은 기존 후속으로 유지한다. 계약 채택은 API 배포나 현재 404 해소를 의미하지 않는다.

권한 A. 석종수는 차단 기록·관리자 조회, 전민규는 게임 결과 검증·점수·보상 자격, 윤태형은 실제 응모권 지급을 담당한다. 관련 요청·검증 근거 연결은 공동 경계이므로 구현 전 계약을 확인한다. 알림 계약은 이번 변경 범위에서 제외한다.

| ID | 메서드·경로 | 요청 | 성공 | 주요 오류 |
| --- | --- | --- | --- | --- |
| AR01 | `GET /admin/abuse-cases` | `page,size,userId,eventId,sourceType,from,to` | 200 `Page<AbuseCaseSummary>` | 403 `ABUSE-001` 및 공통 |
| AR02 | `GET /admin/abuse-cases/{caseId}` | 없음 | 200 `AbuseCaseDetail` | 403 `ABUSE-001`, 404 `ABUSE-002` 및 공통 |
| AR03 | 기존 참여 허용·추첨 제외 결정 초안은 구현 범위에서 제외 | 사후 부정 확인 계약은 후속 검토 | — | — |
| AR04 | 이번 버전 제외 | 회수 대상 조회 | — | — |
| AR05 | 이번 버전 제외 | 회수 결정·처리 | — | — |
| AR06 | 기존 검토 결정 이력 조회 초안은 후속 재검토 | 사후 처리 계약·과거 이력 호환성 확인 | — | — |
| AR07 | `POST /admin/abuse-cases/{caseId}/game-play-invalidations` 기존 초안 유지 | 필수 `gamePlayId:UUID`, `reason:string` | 200 `GameInvalidationResult` | 409 부정 미확정·연결 불일치 |

## 차단 내역 조회 — AR01/AR02

실제 자동 일시 차단이 시작된 이력만 조회한다. 기존의 단순 탐지·검토 기록을 자동 차단 내역으로 섞지 않는다. 만료된 차단도 이력으로 조회하며 새 차단은 새 이력이다. 기존 검토·사후 판단 데이터는 신규 응답에서 제외하되 삭제·덮어쓰지 않는다.

두 API는 활성 ADMIN 전용이다. 공통 접두사 /api/v1, 사용자 문맥·ResponseEnvelope·Page 형식은 [공통 계약](common.md)을 따른다. ID는 UUID 문자열, instant는 UTC Z 형식이다. ?가 붙은 필드도 응답에서 생략하지 않고 값이 없으면 null을 반환한다.

### AbuseCaseSummary

| 필드 | 타입 | 의미 |
| --- | --- | --- |
| id | UUID | 차단 이력 ID |
| userId | UUID | 차단 대상 사용자 |
| userNickname | string | 사용자 표시 닉네임. 조회 시 사용자 정보에서 조인해 제공 |
| eventId? | UUID | 차단을 유발한 관련 이벤트. 사용자 전체 차단을 해당 이벤트로 한정하는 값이 아님 |
| eventTitle? | string | 관련 이벤트 제목. 조회 시 이벤트 정보에서 조인해 제공 |
| sourceType | string | 이번 계약의 값은 ENTRY, 이벤트 응모 요청을 의미 |
| detectionType | string | 이번 계약의 값은 REPEATED_RATE_LIMIT_EXCEEDED, 일반 빈도 제한 거절의 반복 누적을 의미 |
| detectionReason | string | 관리자가 읽을 수 있는 차단 사유 |
| blockedAt | instant | 최초 차단 시작 시각 |
| blockedUntil | instant | 차단 종료 예정 시각. Redis의 실제 현재 차단 상태 보장 값은 아님 |

이번 응답에는 기존 제안의 sourceId·occurredAt·detectedAt을 포함하지 않고 차단 시작·종료 시각으로 표시한다. requestId와 reviewStatus·reviewedBy·reviewedAt·reviewReason도 신규 목록·상세에 포함하지 않는다. sourceType=ENTRY를 출석·미션·게임 차단 제공의 근거로 해석하지 않는다. 현재 차단 여부 boolean은 제공하지 않는다.

### AbuseCaseDetail

AbuseCaseSummary의 모든 필드와 evidence 객체를 반환한다. evidence는 차단 당시 관측 구간·거절 횟수·적용 기준·규칙 버전을 설명하는 관리자 전용 구조다. 토큰·원본 요청 본문·무관한 개인정보를 넣거나 DB JSON 전체를 임의로 노출하지 않는다.

아래 세부 필드명·타입은 2026-10-22 사용자 결정으로 기존 제안을 그대로 채택했다. evidence는 관리자 전용 객체이며 아래 항목을 반환한다. 채택은 구현·배포 완료를 의미하지 않는다.

| 필드 | 타입 | 의미 |
| --- | --- | --- |
| ruleVersion | string | 적용한 차단 판정 규칙 버전 |
| triggerScope | string | USER=사용자 전체 기준, EVENT=같은 이벤트 기준 |
| windowStartedAt, windowEndsAt | instant | 차단 판정에 사용한 집계 구간 |
| requestLimit | int | 해당 범위의 일반 요청 허용 한도 |
| rejectedCount | int | 차단 판정 시 관측한 거절 횟수 |
| rejectionThreshold | int | 차단 진입 거절 횟수 기준 |

```json
{
  "ruleVersion": "v1",
  "triggerScope": "EVENT",
  "windowStartedAt": "2026-10-21T06:00:00Z",
  "windowEndsAt": "2026-10-21T06:01:00Z",
  "requestLimit": 10,
  "rejectedCount": 10,
  "rejectionThreshold": 10
}
```

예시는 같은 이벤트 기준 차단의 evidence다. 이는 목록이나 상세 응답 전체가 아니며 실제 저장·배포 결과도 아니다. 2026-10-22 사용자 결정으로 두 기준이 한 평가에서 동시에 충족되면 사용자 전체 기준(USER)만 대표 근거로 기록하고 차단 이력은 한 건만 만든다. 이때 집계 구간·일반 요청 한도·거절 횟수·거절 임계값은 사용자 전체 기준 값만 사용하며 이벤트 기준 근거를 함께 추가하지 않는다. 사용자 전체 기준만 충족되면 USER, 이벤트 기준만 충족되면 EVENT를 기록한다. 대표 기준 선택과 위 세부 JSON 필드명·타입은 모두 확정됐다. DB 저장 재시도에서도 최초 근거를 유지한다.

### 필터·페이징·오류

- AR01 필터는 page,size,userId,eventId,sourceType,from,to다. 페이지는 1부터 시작하며 Page의 items,page,size,totalElements를 사용한다. size 기본값·상한은 공통 계약의 제안을 따르며 구현 시 검증한다.
- 기본 정렬은 blockedAt DESC, id DESC다. 날짜 필터는 blockedAt 기준 [from,to)이며 시간대가 있는 instant를 받는다.
- sourceType 값은 이번 범위에서 ENTRY다. 별도 detectionType·reviewStatus 필터는 추가하지 않는다. 닉네임·이벤트 제목은 BE가 제공하며 FE의 추가 조회 조인을 필수로 요구하지 않는다.
- AR01/AR02의 업무 오류·기존 공통 오류 적용 기준은 [조회 오류 계약](#조회-오류-계약--ar01ar02)을 따른다. 오류 코드 채택과 실제 구현·배포는 구분한다.
- 일반 사용자에게 반환하는 응모 제한 오류는 기존 429 COMMON-006과 Retry-After다. 일반 한도 초과에서는 집계 구간의 남은 초, 일시 차단 중에는 남은 차단 초를 전달한다. FE는 내부 탐지 수치나 관리자용 evidence를 사용자 안내에 사용하지 않는다.
- 구현·배포 일정과 배포 환경 라우팅은 별도 확인한다. 문서 변경만으로 실 API 연동 가능·404 해소로 표시하지 않는다.

### 조회 오류 계약 — AR01/AR02

2026-10-22 사용자 결정으로 아래 두 업무 오류 코드·공개 메시지를 채택한다.

| 적용 API | HTTP | 업무 코드 | 적용 상황 | 공개 메시지 |
| --- | --- | --- | --- | --- |
| AR01, AR02 | 403 | `ABUSE-001` | 사용자 문맥 검증을 통과했으나 DB 사용자가 ADMIN이 아니거나 ACTIVE가 아님 | 활성 관리자 권한이 필요합니다. |
| AR02 | 404 | `ABUSE-002` | 요청 caseId에 해당하는 조회 대상 자동 차단 이력이 없음 | 차단 이력을 찾을 수 없습니다. |

사용자 문맥 검증 뒤 활성 관리자 권한을 먼저 확인하고, 권한을 통과한 경우에만 상세 이력의 존재 여부를 조회한다. 사용자 헤더의 역할을 권한으로 신뢰하거나 문맥 검증 오류를 `ABUSE-001`로 일괄 치환하지 않는다. `ABUSE-002`는 등록된 AR02에서 업무 이력을 찾지 못한 경우이며, 경로 미구현·라우팅 실패의 HTTP 404를 뜻하지 않는다.

다음 오류는 새 어뷰징 코드를 만들지 않고 [공통 오류 계약](common.md#오류)과 [사용자 문맥 계약](common.md#시연용-사용자-헤더--사용자-요청을-반영한-권고안)을 재사용한다. 기존 코드·상태·메시지 정의의 원본은 공통 계약이다.

| 적용 상황 | 기존 오류 적용 |
| --- | --- |
| 필수 사용자 문맥 헤더 누락·미등록 사용자·역할 또는 멤버십 불일치 | 해당 `USER-*` 코드와 HTTP 상태를 공통 계약 그대로 유지 |
| caseId·userId·eventId UUID, 날짜·시간 또는 page·size 숫자의 형식·타입 해석 실패 | 400 `COMMON-005` |
| 해석 가능한 page·size의 범위 위반, 지원하지 않는 sourceType 등 필터 값 검증 실패 | 400 `COMMON-002` |
| 예상하지 못한 DB 조회 실패 등 서버 내부 오류 | 500 `COMMON-001` |

실패 응답은 [공통 ResponseEnvelope](common.md#응답-봉투--현행-코드)를 사용하며 실제 4xx·5xx 상태와 `data=null`을 유지한다. DB 오류·예외 원문이나 내부 차단 근거를 실패 메시지에 노출하지 않는다.

- AR01의 검색 결과가 없으면 오류가 아니라 200과 빈 items 목록을 반환한다.
- 만료된 자동 차단도 과거 이력이므로 정상 조회하며 만료 자체를 404·409로 처리하지 않는다.
- 조회 API에 별도 상태 충돌 409 코드를 추가하지 않는다. 기존 사용자 문맥 오류의 HTTP 상태는 위 공통 계약을 그대로 유지한다.
- 응모 요청의 일반 제한·자동 차단에는 기존 429 `COMMON-006`을 사용하며, 이를 관리자 조회의 새 오류 코드로 대체하지 않는다.
- 명세 채택만으로 실제 API 경로가 구현·배포되거나 배포 환경의 404가 해소된 것은 아니다.

## 참여 허용·추첨 제외 초안 제외

기존 AR03의 `ALLOW/CONFIRM`, `excludeFromEvent`, `userNoticeReason`을 이용한 참여 허용·이벤트 추첨 제외 결정은 제공하지 않는다. 관련 `eligibilityStatus`, `refundedTicketCount` 등 제외·반환 결과 DTO도 신규 계약에서 제외한다. AR03·AR06 번호는 기존 참조 보존을 위해 남기며 다른 기능의 번호로 재사용하지 않는다. 지급 후 부정 확인·점수 처리에 필요한 판단과 이력 계약은 후속 확인하며 삭제나 폐지로 임의 확정하지 않는다.

## 응모권 회수 제외

AR04·AR05는 이번 버전에서 제외한다. 회수 API·DTO·처리를 제공하지 않는다. 이벤트 취소에 따른 반환은 유지하며 추첨 제외에 따른 반환은 제공하지 않는다.

## 부정 게임 점수 무효화 — 기존 정책 유지, 후속 확인

지급 후 발견한 부정의 게임 점수 무효화·판단 근거·이력 계약은 후속 확인한다. 결정 전에는 AR07의 기존 초안과 점수 처리 정책을 유지한다. 관리자 사유에 따른 개별 당첨 취소·빈 자리 재추첨은 [추첨 API](drawing.md#당첨-취소와-즉시-재추첨)에서 제공하며 어뷰징 확인·기록 연결을 필수 조건으로 요구하지 않는다. 선택적인 어뷰징 근거 연결은 후속 확인한다. 당첨 취소·재추첨을 이유로 응모권을 반환하거나 회수하지 않는다.

`GameInvalidationResult`: `gamePlayId:UUID`, `status:INVALID`, `statistics:MyGameStatistics`, `processedAt:instant`. 부정이 확인되고 연결된 플레이일 때만 무효화한다. 최고점·누적점수·횟수의 정확한 재계산과 최초/변경 결과 이력을 보존하며 응모권 회수는 수행하지 않는다. 실제 확인 근거와 이력·재조회·복구 계약은 담당자 확인 후 구현한다.

결정 기록: [ADR-018](../03-decisions/018-pre-issuance-abuse-blocking-and-no-draw-exclusion.md).
