# Patch Note v1.0.5 (2026-01-07)

## 1. 개요
`CComboBox`의 `SetItemDataPtr` / `GetItemDataPtr` 기능을 활용하여, 특정 시점의 UI 상태(Check Box, Radio 등)를 저장하고 복원하는 기능을 추가했습니다. 또한 동적 할당된 메모리의 누수를 방지하기 위한 삭제 로직을 구현했습니다.

## 2. 주요 변경 사항

### 기능 추가 (Feature)
* **상태 저장 (Save State):**
    * 현재 대화상자의 상태(`DIS` 구조체)를 힙 메모리에 할당.
    * `SetItemDataPtr`를 이용해 콤보 박스 아이템에 구조체 포인터 바인딩.
    * 아이템의 `ItemDataPtr`가 비어 있으면 `new DIS`로 상태 포인터 연결.
    * 콤보 입력창 텍스트가 목록에 없으면 `AddString`으로 새 아이템을 생성.
    * CheckBox 상태(`check_states`)와 RadioButton 상태(`radio_state`)를 함께 저장.
* **상태 삭제 (Delete State):**
    * 선택된 콤보 박스 아이템과 연결된 메모리를 `delete`하고 리스트에서 제거.
* **전체 초기화 (Clean Up):**
    * `CleanUp()` 또는 프로그램 종료 시 콤보 박스 내 모든 항목의 메모리 해제 로직(`ClearComboData`) 추가.

### 기술적 포인트 (Tech Spec)
* **메모리 수명 주기 관리:**
    * 생성: `new DII` (Save 버튼 클릭 시)
    * 소멸: `delete p_data` (Delete 버튼 클릭, 초기화, 또는 종료 시)
    * **주의:** `DeleteString` 호출 전 반드시 메모리를 먼저 해제해야 함을 보장.

### 수정 (Fix)
* `DIS*` 동적 할당 후 해제 누락으로 발생하던 `Detected memory leaks!` 문제를 해제 루틴 보강으로 해결.