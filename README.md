# MFC Dialog Communication & Memory Management (Sots01)

## 📝 프로젝트 소개
MFC(Microsoft Foundation Class) 기반의 다이얼로그 통신 및 메모리 동적 할당 예제 프로젝트입니다.
Main Dialog와 Sub Dialog 간의 데이터 교환, 사용자 정의 메시지 처리, 그리고 동적 배열을 활용한 통계 연산 기능을 구현하였습니다.

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

3.  **동적 메모리 할당 및 통계 처리 (Memory & Algorithm)**
    * 사용자 입력에 따른 `double` 배열 동적 할당 (`new`/`delete`).
    * 난수 생성 및 기초 통계 연산 (평균, 표준편차, 최소/최대값) 알고리즘 구현.

## 🔖 버전 / 변경 사항

### v1.0.2
* **Fix:** 난수 생성 개수 변경 시 `m_array_count`가 사용자 입력(`array_count`)과 항상 동기화되도록 보완
* **Fix:** Modeless 대화상자(`SubDlg`) 실행 중 Modal 대화상자 버튼을 눌러도 중복 생성되지 않도록 예외 처리 추가

> 자세한 원인/재현/수정 과정: [`docs/PATCH_NOTE_v1.0.2.md`](docs/PATCH_NOTE_v1.0.2.md)

## 🚀 Usage
1.  **Main Dialog:** 각종 컨트롤(Radio, Check)을 조작하면 리스트에 로그가 남습니다.
2.  **Modal/Modeless:** 하단 버튼을 통해 서브 다이얼로그를 엽니다.
3.  **Generate Random:** 서브 다이얼로그에서 생성할 난수 개수를 입력하고 `Gen Rand` 버튼을 누르면 통계가 산출됩니다.