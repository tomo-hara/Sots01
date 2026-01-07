# Dialog State Management (DIS)

## 📌 개요
사용자가 설정한 대화 상자의 컨트롤 상태(현재는 CheckBox 한정)를 구조체에 저장하고, 필요시 다시 복원하는 기능을 구현했습니다.
`CComboBox`의 `ItemDataPtr` 주소 공간을 활용하여 상태 구조체(`DialogItemState`)의 포인터를 관리합니다.

## 🏗 데이터 구조 (Data Structure)

### `DialogItemState` (DIS)
대화 상자의 상태를 비트 플래그(Bit Flag)와 열거형(Enum)으로 관리하기 위한 구조체입니다.

* **check_states (`uint32_t`):** 32개의 비트 플래그를 사용하여 체크박스 상태 관리.
* **radio_state (`Radio`):** 라디오 버튼 상태 (현재 저장 로직 미구현).
* **Helper Functions:** `SetFlag`, `HasFlag`, `ClearFlags` 등을 통해 비트 연산을 캡슐화.

## 🔄 로직 흐름 (Logic Flow)

### 1. 상태 저장 (Save)
1.  **캡처:** `SetDialogState()` 함수를 통해 현재 UI 컨트롤의 상태를 `m_dlg_state` 멤버 변수에 기록합니다.
2.  **할당:** `new` 연산자를 통해 힙(Heap) 메모리에 `DIS` 객체를 생성하고 데이터를 복사합니다.
3.  **보관:** `CComboBox::SetItemDataPtr`를 사용하여 콤보 박스의 각 항목에 메모리 주소를 매핑합니다.

### 2. 상태 복원 (Load)
1.  **조회:** `CComboBox::GetItemDataPtr`를 통해 현재 선택된 항목에 매핑된 `DIS*` 주소를 가져옵니다.
2.  **복원:** `LoadDialogState()` 함수에서 저장된 비트 플래그를 확인하여 UI 컨트롤(`CButton::SetCheck`)에 반영합니다.

## 💻 주요 구현 코드

### `LoadDialogState`
저장된 `DIS` 포인터가 유효할 경우, 매핑 테이블(`kMap`)을 순회하며 체크박스 상태를 동기화합니다.

```cpp
void CSots01Dlg::LoadDialogState()
{
    CComboBox* p_combo = (CComboBox*)GetDlgItem(IDC_COMBO1);
    if (!p_combo) return;

    const int index = p_combo->GetCurSel();
    if (index == CB_ERR) return;

    // 1. 저장된 상태 구조체 포인터 획득
    DIS* p_dis = reinterpret_cast<DIS*>(p_combo->GetItemDataPtr(index));
    if (!p_dis) return;

    // 2. 멤버 변수 동기화
    m_dlg_state = *p_dis;

    // 3. CheckBox 상태 복원 (Mapping Table 활용)
    struct Map { int ctrl_id; CheckButtonFlags flag; };
    static const Map kMap[] = {
        { IDC_CHECK1, CheckButtonFlags::Check1 },
        { IDC_CHECK2, CheckButtonFlags::Check2 },
        { IDC_CHECK3, CheckButtonFlags::Check3 },
    };

    CButton* p_check;
    for (const auto& m : kMap) {
        p_check = (CButton*)GetDlgItem(m.ctrl_id);
        if (!p_check) continue;

        // 비트 플래그 확인 후 UI 반영
        const bool checked = p_dis->HasFlag(static_cast<uint32_t>(m.flag));
        p_check->SetCheck(checked ? BST_CHECKED : BST_UNCHECKED);
    }
    
    // Note: Radio Button 상태 복원 로직은 데이터 저장 기능 구현 후 추가 예정
}
```

---

## 🔘 RadioButton 상태 저장/복원

### 저장(UI → DIS)
- `GetCheckedRadioButton(IDC_ENABLE_RADIO, IDC_HIDE_RADIO)`로 선택된 라디오 컨트롤 ID를 얻습니다.
- 컨트롤 ID를 `Radio::Enable/Disable/Hide`로 매핑하여 `m_dlg_state.radio_state`에 저장합니다.

```cpp
UINT checked_id = GetCheckedRadioButton(IDC_ENABLE_RADIO, IDC_HIDE_RADIO);
if (checked_id == 0) {
    m_dlg_state.radio_state = Radio::Enable; // 기본값
} else {
    m_dlg_state.radio_state = RadioFromCtrlId(checked_id);
}
```

### 복원(DIS → UI)
- `radio_state`를 컨트롤 ID로 변환한 뒤 `CheckRadioButton`으로 체크를 반영합니다.

```cpp
const UINT id = CtrlIdFromRadio(p_dis->radio_state);
CheckRadioButton(IDC_ENABLE_RADIO, IDC_HIDE_RADIO, id);
```

> 라디오 선택에 따라 다른 컨트롤 Enable/Hide 같은 부수효과가 있다면, 복원 직후 관련 함수(예: `OnSetCheck(id)`)를 함께 호출하여 UI 정책을 동기화합니다.

## 🧩 추가 구현 사항 (Noteworthy)

### 콤보 입력 텍스트 기반 아이템 생성
- Save 버튼 클릭 시 `CComboBox::GetWindowText()`로 입력창 텍스트를 가져옵니다.
- `FindStringExact()` 결과가 `CB_ERR`이면 목록에 없는 문자열이므로 `AddString()`으로 **새 아이템을 추가**합니다.
- 새 아이템에는 `new DIS`로 상태 구조체를 할당한 뒤 `SetItemDataPtr()`로 연결합니다.
- 기존 아이템이라도 `GetItemDataPtr()`가 `nullptr`이면 `DIS*`를 새로 할당해 연결합니다.

---

## 💾 메모리 관리 (Memory Management)

`CComboBox`는 항목(String)만 관리할 뿐, `SetItemDataPtr`로 연결된 외부 메모리(`DIS*`)의 생명 주기는 관리하지 않습니다. 따라서 개발자가 직접 해제해야 합니다.

#### 🚩 누수 방지 전략 (Anti-Leak Strategy)
* **개별 삭제:** `DeleteString` 호출 직전에 `GetItemDataPtr`로 포인터를 가져와 `delete` 수행.
* **일괄 삭제:** 대화 상자 종료(`OnDestroy`) 또는 초기화(`ResetContent`) 시, 모든 항목을 순회하며 메모리 해제.

#### `ClearComboData` 구현
모든 항목의 메모리를 안전하게 해제하는 헬퍼 함수입니다.

```cpp
void CSots01Dlg::ClearComboData()
{
    CComboBox* p_combo = (CComboBox*)GetDlgItem(IDC_COMBO1);
    const int count = p_combo->GetCount();

    for (int i = 0; i < count; i++) {
        // void* -> DIS* 캐스팅 후 메모리 해제
        DIS* p_data = reinterpret_cast<DIS*>(p_combo->GetItemDataPtr(i));
        
        // 유효성 검사 (nullptr 및 초기화되지 않은 값 -1 체크)
        if (p_data && p_data != (DIS*)-1) {
            delete p_data;
        }
    }
    p_combo->ResetContent(); // UI 리스트 초기화
}
```