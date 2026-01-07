# Changelog

## [1.0.5] - 2026-01-07
### Added
- `CComboBox` 아이템에 구조체 포인터(`DIS*`)를 연결하여 상태를 저장/관리하는 기능 추가.
- CheckBox 상태(`check_states`) 저장/복원 로직 추가.
- RadioButton 상태(`radio_state`) 저장/복원 로직 추가.
- `SetItemDataPtr` 활용 및 이에 따른 메모리 해제(`delete`) 로직 구현.
### Fixed
- **Critical:** 프로그램 종료 시 `CComboBox` Item Data에 할당된 메모리가 해제되지 않아 발생하던 `Detected memory leaks!` 현상 수정.
- `OnDestroy` 핸들러에 `ClearComboData()` 호출 추가.
- **UX:** 상태 저장(`Save`) 후 콤보박스의 선택 항목(CurSel)이 갱신되지 않던 문제 수정.

## [1.0.4] - 2026-01-07
### Added
- **Refactor:** `v1.0.3` AVG 기능 및 구조를 나머지 통계값 함수에도 적용 (예. `GetStandardDeviation`, `GetMinMax`)
### Fixed
- **Critical:** 난수 생성 개수 입력 시 음수를 입력하면 거대한 양수로 오인식되는(Underflow) 문제를 `GetDlgItemInt(..., &bTrans, FALSE)`를 사용하여 해결.

## [1.0.3] - 2026-01-07
### Changed
- **Statistics:** 표준편차(StdDev), 최소/최대(Min/Max) 계산 및 출력 기능을 해당 버튼 이벤트 핸들러에서만 처리하도록 변경
- **Refactor:** 평균 통계 연산 함수(`GetAverage`)에 포인터 연산(Displacement Addressing) 적용.
- 난수 생성 범위를 `-100.0 ~ 100.0`으로 확장.

## [1.0.2] - 2026-01-06
### Fixed
- **Critical:** `SubDlg`에서 동적 배열 재할당 시 멤버 변수(`m_array_count`)가 갱신되지 않던 로직 오류 수정.

## [1.0.1]

### Fixed
- Modeless 대화상자가 실행 중일 때 Modal 대화상자가 생성되지 않도록 예외 처리 추가 (`OnBnClickedModalBtn` 내 `mp_sub_dlg` 체크 로직 적용).
