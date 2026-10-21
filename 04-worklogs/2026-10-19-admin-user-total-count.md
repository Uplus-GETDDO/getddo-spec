# 관리자 사용자 목록의 전체 활성 일반 사용자 수

- 상태: 검토 대기
- 작성: Codex

## 목적

사용자가 요청한 관리자 사용자 목록 응답의 `totalUserCount` 추가 계약을 공용 API 명세에 반영한다.

## 수행 내용

- [사용자 API](../05-api/user.md)에 AdminUserList 응답과 필터·페이지에 무관한 전체 활성 일반 사용자 수를 명시했다.
- 기존 totalElements 의미와 AO02 상세 응답은 유지한다.
- 검색 결과가 없어도 totalUserCount를 반환하는 JSON 예시와 프론트 영향을 안내했다.

## 확인된 사실

- 사용자는 AO01 응답에 totalUserCount를 추가하고 push하도록 명시적으로 요청했다.
- 백엔드 작업: [GD-241](https://ureca4.atlassian.net/browse/GD-241), [PR #135](https://github.com/Uplus-GETDDO/getddo-be/pull/135).
- 명세 반영과 배포 서버 사용 가능 여부는 별개다.

## 다음 할 일

- 백엔드 PR의 CI·리뷰와 배포 후 응답 확인.
- 프론트에서 새 응답 필드 연결 확인.

## 미결정 사항

- 이번 필드의 집계 기준은 USER·ACTIVE로 명시했으며 추가 미결정 사항은 없다.
