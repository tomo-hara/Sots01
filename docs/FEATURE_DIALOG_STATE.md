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