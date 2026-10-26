# GETDDO 공용 명세

GETDDO의 **요구사항·도메인 규칙·API 계약·공용 결정**을 관리합니다.
프론트엔드와 백엔드가 같은 용어와 정책을 사용하도록 기준을 모으고, 구현 세부사항은 각 코드 저장소에서 관리합니다.

[프론트엔드](https://github.com/Uplus-GETDDO/getddo-fe) · [백엔드](https://github.com/Uplus-GETDDO/getddo-be) · [서비스 체험](https://getddo-fe.vercel.app)

## 처음 읽는 순서

1. [구현 범위](00-requirements/scope.md)에서 구현·보류·제외 항목을 확인합니다.
2. [기능 요구사항](00-requirements/functional-requirements.md)과 [용어집](02-domain/glossary.md)으로 기능과 용어를 맞춥니다.
3. [도메인 규칙](02-domain/README.md)과 [공용 결정](03-decisions/README.md)에서 업무 조건과 변경 근거를 확인합니다.
4. [API 계약](05-api/README.md)에서 요청·응답·오류와 개별 계약의 상태를 확인합니다.
5. 미확정 내용은 [미결정 사항](00-requirements/pending-decisions.md)에서 확인합니다.

## 문서 지도

| 경로                                         | 책임                                                                                                                                     |
| -------------------------------------------- | ---------------------------------------------------------------------------------------------------------------------------------------- |
| [00-requirements](00-requirements/README.md) | 기능 요구사항, 구현·보류·제외 범위, 미결정 사항                                                                                          |
| [01-conventions](01-conventions/README.md)   | 코드 저장소의 브랜치·커밋·PR·협업 절차                                                                                                   |
| [02-domain](02-domain/README.md)             | 용어와 이벤트·응모권·응모·추첨·출석·미션·게임·알림·배너 정책                                                                             |
| [03-decisions](03-decisions/README.md)       | 프론트엔드·백엔드에 공통으로 적용되는 결정과 근거                                                                                        |
| [04-worklogs](04-worklogs/README.md)         | 명세 정리 과정·확인한 사실·후속 작업                                                                                                     |
| [05-api](05-api/README.md)                   | 공통 응답·권한·오류와 도메인별 HTTP 계약                                                                                                 |
| [templates](templates/)                      | 공용 [ADR](templates/adr.md)·[작업 기록](templates/worklog.md)·Jira [에픽](templates/jira-epic.md)·[태스크](templates/jira-task.md) 양식 |

## 도메인별 바로가기

| 도메인      | 규칙                                                                                       | API                                                                                                                 |
| ----------- | ------------------------------------------------------------------------------------------ | ------------------------------------------------------------------------------------------------------------------- |
| 이벤트·응모 | [이벤트](02-domain/event.md) · [응모](02-domain/entry.md)                                  | [이벤트](05-api/event.md) · [응모](05-api/entry.md)                                                                 |
| 응모권·추첨 | [응모권](02-domain/ticket.md) · [추첨](02-domain/drawing.md)                               | [응모권](05-api/ticket.md) · [추첨·공개 결과](05-api/drawing.md)                                                    |
| 참여·보상   | [출석](02-domain/attendance.md) · [미션](02-domain/mission.md) · [게임](02-domain/game.md) | [출석](05-api/attendance.md) · [미션](05-api/mission.md) · [게임](05-api/game.md)                                   |
| 운영·안내   | [알림](02-domain/notification.md) · [배너](02-domain/banner.md)                            | [사용자](05-api/user.md) · [알림](05-api/notification.md) · [배너](05-api/banner.md) · [차단 조회](05-api/abuse.md) |

## 계약 상태와 문서 우선순위

계약의 **채택·구현·배포·실측은 서로 다른 상태**입니다.
API 문서 전체를 초안 또는 완료로 묶어 판단하지 않고, 각 도메인 문서의 상태 머리말·개별 확정 표시·변경 이력을 확인합니다.
과거 검토 메모나 작업 기록은 이후 확정된 계약보다 우선하지 않습니다.

내용이 충돌하면 다음 순서로 판단합니다.

1. 확정된 요구사항·구현 범위
2. 확정된 도메인 규칙
3. 채택된 공용 결정
4. 공용 협업 규칙
5. 미결정 사항

해결할 수 없는 충돌이나 미확정 정책은 임의로 확정하지 않습니다.
최신 코드·배포 검증은 [FE 문서](https://github.com/Uplus-GETDDO/getddo-fe/tree/dev/docs)와 [BE 저장소](https://github.com/Uplus-GETDDO/getddo-be)에서 별도로 확인합니다.

## 변경과 협업

- 공용 정책을 바꾸면 변경 전·후와 양쪽 구현에 미치는 영향을 기록합니다.
- 중요한 결정은 ADR로, 확정되지 않은 정책은 미결정 사항으로 분리합니다.
- 같은 내용을 복제하지 않고 기준 문서를 연결합니다. 이동·이름 변경 시 참조도 확인합니다.
- 공용 명세 작업의 목적·수행 내용·확인 사실·다음 작업은 `04-worklogs/`에 기록합니다.
- 이 저장소는 [AGENTS.md](AGENTS.md)에 따라 `main`에서 문서를 변경·커밋하고 `origin/main`에 반영합니다. 별도 작업 브랜치·PR은 만들지 않습니다.
- `01-conventions/`의 작업 브랜치·PR 규칙은 프론트엔드·백엔드 코드 저장소에 적용합니다.

## 저장소별 문서 경계

| 저장소                                                 | 관리 범위                                        |
| ------------------------------------------------------ | ------------------------------------------------ |
| 이 저장소                                              | 요구사항·공용 정책·API 계약·공용 결정·협업 규칙  |
| [getddo-fe](https://github.com/Uplus-GETDDO/getddo-fe) | UI·프론트엔드 구조·실행·테스트·배포·FE 설계 결정 |
| [getddo-be](https://github.com/Uplus-GETDDO/getddo-be) | 서버·DB·내부 구현·빌드·운영·BE 설계 결정         |
