# 이벤트 API 초안

- 상태: 이번 정책 변경 검토 완료 (2026-10-11) / 상세 API 계약은 담당자 검토 대기
- 공통 계약: [공통 API 계약](common.md)
- 도메인 규칙: [이벤트 규칙](../02-domain/event.md)

경로·메서드·DTO·업무 오류 코드는 구현 전 제안이다. 확정된 업무 정책은 연결된 요구사항·도메인 문서를 따른다.

## 사용자 이벤트 조회

| ID | 권한 | 메서드·경로 | 요청 | 성공 | 주요 오류 |
| --- | --- | --- | --- | --- | --- |
| E01 | R | `GET /events` | `page,size,status,eventType,membershipRule` | 200 `Page<EventSummary>` | 공통 |
| E02 | R | `GET /events/{eventId}` | 없음 | 200 `EventDetail` | 공통 |

| DTO | 필드 |
| --- | --- |
| `EventSummary` | `id:UUID`, `title:string`, `imageUrl:string?`, `eventType:NO_TICKET/TICKET`, `weightingEnabled:boolean`, `membershipRule:excellent/vip/vvip`, `startsAt:instant`, `endsAt:instant`, `status:EventStatus`, `serverTime:instant`, `myEntered:boolean`, `winnerCount:long` |
| `EventDetail` | EventSummary에서 `myEntered`, `winnerCount`를 제외한 필드 + `description:string`, `maxTicketsPerUser:int?`, `prizes:Prize[]` |
| `Prize` | `id:UUID`, `rank:int`, `name:string`, `description:string?`, `imageUrl:string?`, `winnerCount:int` |

### 이벤트와 겟또타임 구분 (R4)

화면의 기본 이벤트는 `eventType=NO_TICKET`(응모권 미사용), 겟또타임은 `eventType=TICKET`(응모권 사용)으로 구분한다. E01·AE01의 기존 `eventType` 필터로 각각 조회하며 별도 분류 필드는 추가하지 않는다. 겟또타임은 응모권 사용 이벤트 전체를 뜻하므로 가중치 여부나 월말 소진용 여부로 제한하지 않는다. 용어는 [공통 용어](../02-domain/glossary.md)를 따른다.

### 사용자 목록의 내 응모 여부 (R7)

`myEntered`는 E01의 각 `EventSummary`에 항상 반환하는 boolean이다. 사용자 상세 E02와 관리자 목록·상세의 `AdminEvent`에는 포함하지 않는다.

- 요청 사용자가 해당 이벤트에 정상 접수된 응모를 가졌으면 `true`다. 응모권 미사용 이벤트도 포함하며, 차감한 응모권 수가 0인지로 판정하지 않는다.
- 응모하지 않았거나 거절된 요청만 있으면 `false`다. 다른 사용자의 응모는 영향을 주지 않으며, 관리자 계정이 사용자 목록을 조회하면 `false`다.
- 정상 접수 후 이벤트가 취소되거나 응모권이 반환돼도 기존 응모 기록에 따라 `true`를 유지한다.
- 추가 응모 가능 여부나 당첨 여부를 나타내지 않는다. 추가 응모 자격은 [응모 API E07](entry.md#사용자-응모현황)에서 확인한다. 기존 사용자 문맥 필수 조건을 유지한다.

### 조회 상태 표현

`EventStatus`의 제안 값은 `SCHEDULED`, `OPEN`, `CLOSED`, `DRAW_CONFIRMED`, `PUBLISHED`, `CANCELED`, `REDRAWING`, `NO_ENTRANTS`다. 어뷰징 전원 제외 종료 상태는 신규 계약에서 사용하지 않는다. 과거 종료 상태의 조회 호환성은 담당자가 별도로 검토하며 저장 이력을 소급 변경하지 않는다. 현행 DB의 `SUSPENDED`는 이번 요구사항에서 제외된 상태이므로 API 계약에 포함하지 않는다. 관리자에게는 운영 상태를 제공하지만 사용자 목록·상세에서 `REDRAWING`을 그대로 노출하지 않는다. 이미 발표된 이벤트는 공개 상태를 유지하고 최초 발표 전에는 기존 대기 상태로 표현하도록 제안한다. 공개 결과에는 별도 공개 여부를 사용하며 이벤트 상태만으로 당첨 정보를 노출하지 않는다. 구체적인 중간 결과 표시 범위는 [결과 조회](drawing.md#공개-결과와-본인-결과)의 공개 중간 상태를 따른다.

### 사용자 목록의 당첨 예정 인원 합계 — GD-251

- `winnerCount`는 해당 이벤트에 등록된 모든 경품의 `winnerCount` 합계다. 예를 들어 1등 1명·2등 3명이면 `4`다. 추첨으로 실제 선정된 당첨자 수·응모자 수·당첨 확률을 뜻하지 않는다. 응모권 미사용·사용 이벤트에 모두 적용한다.
- 합계는 항상 0 이상의 JSON 정수(`long`, 64비트)로 반환한다. 각 경품의 인원은 기존 `int`지만 복수 경품 합계가 32비트 범위를 넘을 수 있으므로 합계는 `long`으로 제공한다. 경품이 없는 과거 데이터는 `0`이다. 등록·수정 시 경품 필수 조건은 유지한다.
- 합계 필드는 E01에만 추가한다. E02와 관리자 응답은 기존 `prizes[].winnerCount`를 유지하며 최상위 `winnerCount`를 추가하지 않는다.
- 요청 파라미터와 기존 최신 등록순, 상태·유형·멤버십 필터 및 삭제 제외 조건은 유지한다. 페이지 형식은 [공통 목록 계약](common.md#목록-조회)의 `page`, `size`, `items`, `totalElements`를 따른다.
- 당첨 인원순 정렬과 태그·추천 필드는 이번 계약에 추가하지 않는다. AE01의 기존 정렬도 변경하지 않는다.

예: `GET /events?eventType=NO_TICKET&page=1&size=12`의 `data.items[].winnerCount`로 기본 이벤트 카드의 당첨 예정 인원을 표시한다. 변경 전·후와 영향은 [GD-251 작업 기록](../04-worklogs/2026-10-20-event-winner-count.md)을 따른다. 이 문서 반영을 배포·FE 연동 완료로 해석하지 않는다.

## 관리자 이벤트 등록·운영

권한 A. 이벤트 CRUD·시작/마감 상태 전환·취소는 박지훈, 배너는 전민규, 반환은 윤태형, 추첨·수동 공개와 해당 상태 연결은 석종수와 협의한다.

| ID | 메서드·경로 | 요청 | 성공 | 주요 오류 |
| --- | --- | --- | --- | --- |
| AE01 | `GET /admin/events` | `page,size,keyword,status,eventType,from,to,sort,weightingEnabled,createdFrom,createdTo` | 200 `Page<AdminEvent>` | 공통, 400 `EVENT-008` 잘못된 검색 조건 |
| AE02 | `GET /admin/events/{eventId}` | 없음 | 200 `AdminEvent` | 공통 |
| AE03 | `POST /admin/events` | `EventWriteRequest` | 201 `AdminEvent` | 400 기간·경품·조건 오류 |
| AE04 | `PUT /admin/events/{eventId}` | `EventWriteRequest` | 200 `AdminEvent` | 409 수정 불가 상태·시작 앞당김 |
| AE05 | `DELETE /admin/events/{eventId}` | 없음 | 200 `null` | 409 시작/응모 이력 존재, 삭제 이벤트의 알림 처리 계약 |
| AE08 | `POST /admin/events/{eventId}/cancel` | `ReasonRequest` | 200 `EventOperationResult` | 400 취소 사유 오류, 409 `EVENT-017` 이미 취소됨, 반환 실패 시 성공 금지 |

AE06·AE07은 이벤트 일시 중단·재개 기능 제외에 따라 사용하지 않는다. 기존 AE08 식별자는 유지한다.

### 관리자 이벤트 기간 검색

AE01의 `from`, `to`는 각각 선택 `date`이며 KST 기준 `YYYY-MM-DD`로 받는다. 이 API는 공통 기간 필터의 `instant` 규칙에 대한 예외다. 사용자 목록 E01에는 날짜 검색 필터를 추가하지 않는다.

- 입력한 시작일과 종료일을 모두 포함한다. 같은 날짜로 검색할 수 있으며 `from > to`는 거절한다.
- 검색 경계는 KST 시작일 00:00 이상, 종료일 **다음 날** 00:00 미만이다. UTC 시각으로 변환해 이벤트 모집 기간 `[startsAt, endsAt)`과 겹치는 이벤트를 조회한다.
- `from`만 있으면 해당 날짜 00:00 이후까지 모집하는 이벤트(`endsAt > 검색 시작 경계`), `to`만 있으면 종료일 다음 날 00:00 전에 모집을 시작한 이벤트(`startsAt < 검색 종료 경계`)를 포함한다. 두 값을 모두 생략하면 기간으로 제한하지 않는다.
- 날짜 형식 오류·존재하지 않는 날짜·역전된 범위 등 잘못된 검색 입력은 400으로 거절한다.

예를 들어 `from=2026-10-10&to=2026-10-23`은 KST `[2026-10-10T00:00:00+09:00, 2026-10-24T00:00:00+09:00)`, UTC `[2026-10-09T15:00:00Z, 2026-10-23T15:00:00Z)`와 겹치는 모집 기간을 검색한다. 모집 시작일이 10월 10일 전이어도 해당 기간과 겹치면 포함한다.

### 관리자 이벤트 검색 확장 (R2)

AE01에 다음 선택 파라미터를 추가한다. 기존 페이지·권한·응답 구조는 유지하며 E01에는 추가하지 않는다.

| 파라미터 | 타입·허용 값 | 생략 시 |
| --- | --- | --- |
| `sort` | string, `createdAt` / `startsAt` / `endsAt` / `title` 중 하나와 `,asc` / `,desc` 조합. 예: `startsAt,asc` | `createdAt,desc` |
| `weightingEnabled` | boolean, `true` / `false` | 가중치 여부로 제한하지 않음 |
| `createdFrom` | date, KST `YYYY-MM-DD`, 이벤트 등록일 하한 포함 | 등록일 하한 없음 |
| `createdTo` | date, KST `YYYY-MM-DD`, 이벤트 등록일 상한 포함 | 등록일 상한 없음 |

- 정렬은 단일 조건만 받는다. 값이 같으면 이벤트 `id`도 같은 방향으로 정렬하며, 정렬한 결과에 페이지를 적용한다. 허용 목록 외 정렬은 400 `EVENT-008`로 거절한다.
- `weightingEnabled=false`는 가중치를 사용하지 않는 이벤트만 조회한다. 필터 생략과 다르다.
- `createdFrom`·`createdTo`는 모집 기간이 아닌 **이벤트 등록 시각 `createdAt`** 기준이다. KST 시작일 00:00 이상·종료일 다음 날 00:00 미만을 UTC 시각으로 비교하여 입력한 양끝 날짜를 포함한다. 같은 날짜도 검색할 수 있다.
- 등록일 경계를 하나만 보내면 해당 하한 또는 상한만 적용한다. 둘 다 생략하면 등록일로 제한하지 않는다. 기존 `from`·`to`의 모집 기간 겹침 조건과 함께 사용할 수 있으며 모든 검색 조건은 AND로 적용한다. 전체 건수에도 같은 조건을 적용한다.
- 날짜는 실제 존재하는 `YYYY-MM-DD`여야 하며 빈 값·확장 연도를 허용하지 않는다. `createdFrom > createdTo`나 지원 범위를 벗어난 UTC 검색 경계는 400 `EVENT-008`로 거절한다. 타입·형식·입력 검증 오류는 [공통 오류 계약](common.md#오류)을 따른다.

예를 들어 `sort=startsAt,asc&weightingEnabled=true&createdFrom=2026-10-01&createdTo=2026-10-17`은 KST 10월 1~17일에 등록한 가중치 이벤트를 모집 시작 시각 오름차순으로 조회한다. 모집 기간도 제한하려면 `from`·`to`를 별도로 함께 전달한다.

## 이벤트·경품 동시 등록 및 수정

| `EventWriteRequest` 필드 | 타입 | 필수 | 제약 |
| --- | --- | --- | --- |
| title | string | 필수 | 1~200자 |
| description | string | 필수 | 공백만 입력 불가, UTF-8 최대 65,535바이트 |
| imageKey | string/null | 선택 | 최대 500자, 이미지 처리 방식은 담당 기능 계약에 따름 |
| eventType | NO_TICKET/TICKET | 필수 | 이벤트 유형 |
| weightingEnabled | boolean | 필수 | NO_TICKET은 false |
| maxTicketsPerUser | int/null | 필수 | 미사용 null, 사용·미가중치 1, 사용·일반 가중치 5, 사용·월말 소진용 null 제안. 월말 운영 허용 조건은 담당자 계약에 따름 |
| membershipRule | excellent/vip/vvip | 필수 | 최소 등급 |
| startsAt / endsAt | instant | 필수 | 종료 > 시작, 명시적 오프셋 |
| prizes | PrizeWrite[] | 필수 | 비어 있지 않음, 등수 중복 금지 |

`PrizeWrite`: 선택 `id:UUID`(수정 시 기존 경품), 필수 `rank:int(≥1)`, `name:string(1~200)`, `winnerCount:int(≥1)`, 선택 `description:string/null`(UTF-8 최대 65,535바이트), `imageKey:string/null(≤500)`.

이벤트 설명과 각 경품 설명의 상한은 등록·수정에 동일하게 적용한다. 글자 수가 아닌 UTF-8 인코딩 후 바이트 수를 각각 검사하며 초과 값을 잘라 저장하지 않는다. 이벤트 설명 초과는 400 `EVENT-001`, 경품 설명 초과는 400 `EVENT-004`로 거절한다. 경품 설명은 생략하거나 `null`로 보낼 수 있다.

```json
{
  "title": "가을 경품 이벤트",
  "description": "유효한 응모권으로 응모하는 이벤트",
  "imageKey": null,
  "eventType": "TICKET",
  "weightingEnabled": true,
  "maxTicketsPerUser": 5,
  "membershipRule": "vip",
  "startsAt": "2026-09-29T18:00:00+09:00",
  "endsAt": "2026-09-29T18:20:00+09:00",
  "prizes": [
    { "rank": 1, "name": "경품 A", "description": null, "imageKey": null, "winnerCount": 1 },
    { "rank": 2, "name": "경품 B", "description": null, "imageKey": null, "winnerCount": 3 }
  ]
}
```

등록 요청자의 관리자 권한을 확인하고 초기 상태는 서버가 정한다. 이벤트와 경품은 한 트랜잭션으로 등록한다. 독립적인 경품 생성 후 이벤트 등록을 완료하는 API로 쪼개지 않는다. 한 등수에 한 경품 종류와 별도 당첨 인원을 설정한다.

PUT은 수정 가능한 설정 전체를 전달한다. 기존 경품은 ID를 유지하고 다른 이벤트의 경품 ID는 거절한다. 경품 목록 교체도 이벤트와 함께 원자적으로 처리한다. 경품 개수·등수 상한을 임의로 3등까지로 제한하지 않는다.

월말 소진용 이벤트는 별도 구분 필드 없이 `eventType=TICKET`, `weightingEnabled=true`, `maxTicketsPerUser=null` 조합으로 표현한다. `NO_TICKET`의 `maxTicketsPerUser=null`과는 `eventType`으로 구분한다. 서버는 이벤트 유형·가중치 여부·상한값 조합을 검증한다. 월말 소진용 이벤트의 지정 주체와 허용 조건은 담당자가 정한다.

`AdminEvent`: EventDetail 전체 + `drawScheduledAt:instant`, `imageKey:string?`, `createdAt:instant`, `updatedAt:instant`, `canceledAt:instant?`, `prizeImages:{prizeId:UUID,imageKey:string?}[]`. 관리자에게 운영 상태를 보여준다.

수정은 시작 전 **SCHEDULED**에서만 허용하고, 현재 설정된 시작 시각보다 앞당길 수 없다. 취소 상태는 시작 전이어도 수정할 수 없다. 진행·마감 후에는 제목·이미지도 수정할 수 없다. 수정 시 시작 알림 예약과 필요한 변경 알림을 연동한다.

삭제는 시작 전이고 응모 이력이 없을 때만 가능하다. 배너는 자동 삭제하지 않으며 관리자가 별도로 수동 삭제한다. 이미 생성된 알림 전체를 삭제한다는 정책은 최신 ADR에서 철회되었으므로 임의로 삭제하거나 반드시 보존한다고 확정하지 않는다. 삭제 이벤트의 기존 알림 처리 방식은 담당자 계약에 따른다.

### 추첨 시각과 발표 시각 구분 — 2026-10-15

- 변경 전: 이벤트 목록·상세·관리자 응답에 `publicationScheduledAt`을 포함해 마감 +5분을 발표 예정 시각으로 표현했다.
- 변경 후: `publicationScheduledAt`을 모든 이벤트 응답에서 제외한다. 사용자 목록·상세는 추첨/발표 예정 시각이나 카운트다운을 제공하지 않고 마감 후 실제 공개 전까지 결과 발표 대기를 표시한다.
- 관리자에게만 `drawScheduledAt`을 제공한다. 값은 `endsAt +5분`의 최초 자동 추첨 기준 시각이며 실제 추첨 실행·명단 확인·결과 공개가 완료됐다는 의미가 아니다. 취소·삭제·장애 등으로 실제 실행이 없거나 늦을 수 있다.
- 마감 +5분 자동 추첨은 유지한다. 관리자 확인·승인과 실제 수동 공개를 구분하며 시간 경과나 확인만으로 공개하지 않는다.
- 실제 공개 여부와 `publishedAt`은 [사용자 결과 조회 계약](drawing.md#공개-결과와-본인-결과)을 따른다. 이벤트 응답에 추정 발표 시각을 다시 만들지 않는다.

근거는 [이벤트 규칙](../02-domain/event.md#상태-관리), [수동 최초 공개 결정](../03-decisions/022-manual-initial-publication.md)과 [GD-203 작업 기록](../04-worklogs/2026-10-15-gd203-event-time-contract.md)을 따른다. 이 절은 응답 계약의 변경이며 BE 코드 병합·배포 완료를 의미하지 않는다. 프론트는 기존 필드를 제거하고 관리자에서만 새 필드를 사용하도록 연동한다.

## 상태 운영

`ReasonRequest`: 필수 `reason:string`, 공백만 입력 불가, UTF-8 최대 65,535바이트. 바이트 상한 초과는 400 `EVENT-016`으로 거절하며 값을 잘라 저장하지 않는다. `EventOperationResult`: `eventId:UUID`, `previousStatus:EventStatus`, `status:EventStatus`, `refundedTicketCount:long`, `processedAt:instant`.

| 작업 | 처리 계약 |
| --- | --- |
| 취소 | 이미 취소된 `CANCELED`를 제외한 상태에서 허용. 당첨 발표 이후·대상 없는 종료 상태 포함. 취소와 대상 차감분 반환이 함께 완료되어야 200 |

유효한 취소 요청을 이미 `CANCELED`인 이벤트에 다시 보내면 409 `EVENT-017`(`ALREADY_CANCELED`, 이미 취소된 이벤트)로 거절한다. 기존 `canceledAt`·`updatedAt`을 변경하지 않고 응모권 반환·취소 알림·감사 기록도 추가하지 않는다. 이미 취소된 이벤트의 성공 결과를 200으로 재전달하는 방식은 사용하지 않는다.

AE08을 반환 작업 접수만 한 202로 완료 처리하지 않는다. 실패는 취소/반환 모두 완료된 것으로 응답하지 않고 같은 원본 차감에 중복 반환하지 않는다. 성공 응답의 반환 수량은 취소 처리의 확정 결과로 제안한다.
