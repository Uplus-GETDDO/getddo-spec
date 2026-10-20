# GD-199 병합·배포 상태 동기화 — 2026-10-20

## 목적

- 추첨 API 문서의 "PR #108 OPEN·병합 후 적용 예정" 표기를 실제 병합·배포 상태로 갱신한다.
- 관리자 추첨·어뷰징 경로의 실배포 여부를 확인해 FE 연동(GD-253)의 기준으로 삼는다.

## 확인된 사실

- [BE PR #108](https://github.com/Uplus-GETDDO/getddo-be/pull/108) `GD-199 관리자 단일 발표 요청과 최초 추첨 공개 지원`은 2026-10-17T04:52:50Z dev에 병합됐다(머지 커밋 `e1342051`). 해당 커밋은 main에도 포함된다(`compare/main...e1342051` → `behind`).
- BE Container CD는 dev 기준으로 실행되며 최신 성공은 2026-10-20T00:59:17Z(`c6287fb8`)다. GD-199 병합 이후 CD가 성공했다.
- 2026-10-20 배포 API(`https://dzc8oneey9mmx.cloudfront.net/api/v1`) 헤더 없는 호출 확인:

  | 경로 | 결과 |
  | --- | --- |
  | `GET /admin/draws/{id}`, `/failures`, `/execution-audits`, `/participants`, `/checks` | 401(인증 요구) — 라우팅됨 |
  | `GET /admin/events/{id}/draws`, `/result-changes` | 401 — 라우팅됨 |
  | `GET /admin/wins/{id}/cancellations`, `/admin/cancellations/{id}/redraws`, `/admin/draws/{id}/publication` | 405(POST 전용) — 라우팅됨 |
  | `POST /admin/draws/{id}/result-confirmation`(미등록 UUID 헤더) | USER-002 — 라우팅됨 |
  | `GET /admin/abuse-cases`, `/admin/abuse-cases/{id}` | 404 — 미배포 |
  | `GET /admin/definitely-not-real-endpoint` | 404 — 미존재 경로 대조군 |

- AR01·AR02는 미배포이고 상세 필드 계약도 여전히 담당자 검토 대기다. GD-141 설계 확인 전까지 목업 표시 필드를 계약으로 확정하지 않는다.

## 변경 문서

- `05-api/drawing.md` — 갱신일, 코드 대조 기준, AD 표의 GD-199 행, 후속 문구, `GD-199 PR 대조 계약` 절 제목·본문을 병합·배포 상태로 갱신.
- `00-requirements/pending-decisions.md` — GD-169 후속 항목과 수동 최초 공개 후속 항목의 "리뷰·병합 대기/OPEN" 표기를 병합·배포 완료와 잔여 FE 연동(GD-253)으로 갱신.
- `03-decisions/022-manual-initial-publication.md` — 바뀐 절 제목의 앵커 링크 갱신.

## 다음 할 일·미결정

- FE 연동은 GD-253 범위다. 최초 발표는 최신 검토 `runId`로 본문·추가 멱등 헤더 없는 `result-confirmation` 한 번이다.
- AR01·AR02의 요약/상세 필드(차단 사유·관련 요약·검증 근거·권한)는 담당자 확인 항목으로 유지한다. 배포 전까지 하이브리드 목업을 유지한다.
- `PUBLICATION-002`의 최초 결과 문구 정렬은 담당자 후속으로 유지한다.
