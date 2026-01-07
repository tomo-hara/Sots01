# MFC Dialog Communication & Memory Management (Sots01)

## 📝 프로젝트 소개
MFC(Microsoft Foundation Class) 기반의 다이얼로그 통신 및 메모리 동적 할당 예제 프로젝트입니다.
Main Dialog와 Sub Dialog 간의 데이터 교환, 사용자 정의 메시지 처리, 동적 배열을 활용한 통계 연산, 그리고 **포인터 기반의 UI 상태 저장/복원** 기능을 구현하였습니다.

## 🛠 개발 환경
* **IDE:** Visual Studio 2017
* **Language:** C++
* **Framework:** MFC (Dialog-based)

## 💡 주요 기능
1.  **다이얼로그 제어 (Dialog Management)**
    * Modal 및 Modeless 다이얼로그 생성 및 생명주기 관리.
    * `WM_USER` 기반의 사용자 정의 메시지(`WM_DESTROY_SUB_DIALOG`, `WM_RELOAD_MAIN_DIALOG_INFO` 등)를 통한 부모-자식 윈도우 간 비동기 통신.

2.  **데이터 동기화 (Data Synchronization)**
    * 구조체(`DII`)를 활용하여 다이얼로그 간 설정 값(Combo Box, Radio, Check Box) 공유 및 복원.
    * `CListCtrl`을 이용한 실시간 사용자 이벤트(UI Control 조작) 로깅.
    * **UI 상태 저장/복원:** `CComboBox`의 `SetItemDataPtr`를 활용하여 체크박스/라디오 버튼 상태를 구조체(`DIS`) 단위로 저장 및 복원.
    * **메모리 누수 방지:** `ClearComboData()`를 통해 콤보박스 아이템에 연결된 힙 메모리를 안전하게 해제.

3.  **동적 메모리 할당 및 통계 처리 (Memory & Algorithm)**
    * 사용자 입력에 따른 `double` 배열 동적 할당 (`new`/`delete`).
    * 난수 생성 및 기초 통계 연산 (평균, 표준편차, 최소/최대값) 알고리즘 구현.
    * **[Update]** 난수 생성 범위 확장 (`-100.0 ~ 100.0`) 및 입력값 유효성 검사 강화.
    * **[Refactor]** 포인터 연산을 활용한 고속 통계 산출 (평균, 표준편차, 최소/최대값).

## 🔖 버전 / 변경 사항

### v1.0.5 (Latest)
* **Feat:** `CComboBox` 아이템에 구조체 포인터(`DIS*`)를 연결하여 UI 상태(Check/Radio) 저장 및 복원 기능 추가
* **Fix:** 프로그램 종료 시 `Detected memory leaks!` 해결 (콤보박스 데이터 메모리 해제 로직 `ClearComboData` 구현)
* **Fix:** 상태 저장(`Save`) 직후 콤보박스 선택 항목이 갱신되지 않던 UX 문제 수정

> 상태 관리 상세 기술 문서: [`docs/FEATURE_DIALOG_STATE.md`](docs/FEATURE_DIALOG_STATE.md)  
> 패치 노트: [`docs/PATCH_NOTE_v1.0.5.md`](docs/PATCH_NOTE_v1.0.5.md)

### v1.0.4
* **Feat:** 기초 통계 연산 기능 구현 완료 (표준편차, 최소/최대값)
* **Fix:** 난수 생성 개수 입력 시 음수 입력으로 인한 메모리 할당 오류 방지 (`GetDlgItemInt` Unsigned 처리)

> 패치 노트: [`docs/PATCH_NOTE_v1.0.4.md`](docs/PATCH_NOTE_v1.0.4.md)

### v1.0.3
* **Refactor:** 통계 연산 함수(`GetAverage` 등)에 포인터 연산(Displacement Addressing) 적용 및 구조 개선
* **Change:** 난수 생성 범위를 `-100.0 ~ 100.0`으로 확장하고, 통계 계산을 버튼 클릭 시점에 수행하도록 변경

> 패치 노트: [`docs/PATCH_NOTE_v1.0.3.md`](docs/PATCH_NOTE_v1.0.3.md)

### v1.0.2
* **Fix:** 난수 생성 개수 변경 시 `m_array_count`가 사용자 입력(`array_count`)과 항상 동기화되도록 보완
* **Fix:** Modeless 대화상자(`SubDlg`) 실행 중 Modal 대화상자 버튼을 눌러도 중복 생성되지 않도록 예외 처리 추가

> 자세한 원인/재현/수정 과정: [`docs/PATCH_NOTE_v1.0.2.md`](docs/PATCH_NOTE_v1.0.2.md)

## 🚀 Usage
1.  **Main Dialog (State Save/Load):**
    * 체크박스나 라디오 버튼 상태를 변경한 후 `Save` 버튼을 누르면 콤보박스에 현재 상태가 저장됩니다.
    * 콤보박스에서 저장된 항목을 선택하면 해당 시점의 UI 상태가 복원됩니다.
    * `Delete` 버튼으로 항목을 삭제할 수 있습니다. (저장된 상태 또한 제거)
    * `Modal` 혹은 `Modeless` 버튼을 통해 서브 다이얼로그를 엽니다.
2.  **Sub Dialog (Statistics):**
    * `Generate Random`에서 개수를 입력하고 `Gen Rand`를 누르면 리스트에 난수가 생성됩니다.
    * `AVG`, `STD DEV`, `MIN/MAX` 버튼을 눌러 각 통계 정보를 확인합니다.