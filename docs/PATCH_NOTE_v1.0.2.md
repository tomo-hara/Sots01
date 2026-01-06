# Patch Note v1.0.2 (2026-01-06)

이 문서는 **README에는 핵심만 남기고**, v1.0.2에서 수정한 내용의 **재현/원인/수정 포인트**를 정리한 기록입니다.

---

## 1) 난수 배열 동적할당 시 `m_array_count` 동기화

### 증상
* `Gen Rand` 버튼을 여러 번 누르면서 **생성 개수**를 바꿀 때, 내부에서 사용하는 배열 크기(`m_array_count`)가 사용자 입력값과 어긋나면
  * 통계(평균/표준편차/Min/Max) 계산 대상이 의도와 달라지거나
  * 특정 조건에서 배열 접근이 불안정해질 수 있습니다.

### 재현 방법(예시)
1. SubDlg 실행
2. 생성 개수 입력 → **Gen Rand**
3. 생성 개수를 다른 값으로 변경 → **Gen Rand** 반복

### 원인
* 난수 생성 개수를 읽어오는 `array_count`와, 멤버로 보관하는 `m_array_count`가 **항상 같은 값을 유지해야** 합니다.
* 동적할당(재할당) 분기 내부/외부 위치에 따라 `m_array_count` 갱신이 누락되면, 이후 루프/통계 함수가 **stale count**를 사용하게 됩니다.

### 수정 포인트
* (재)할당 여부와 관계없이 **항상** 멤버를 갱신합니다.

```cpp
UINT array_count = GetDlgItemInt(IDC_GEN_COUNT_EDIT);

if (array_count > m_array_count) {
    delete[] mp_array;
    mp_array = new double[array_count];
}

m_array_count = array_count; // ✅ 항상 동기화
```

### 간단 검증 체크리스트
* 작은 값 → 큰 값 → 작은 값으로 바꿔도
  * 리스트 출력 개수
  * 평균/표준편차/Min/Max
  가 입력값 기준으로 일관되게 계산되는지 확인

---

## 2) Modeless SubDlg 실행 중 Modal 버튼 중복 생성 방지

### 증상
* Modeless `SubDlg`가 열려 있는 상태에서, Main Dialog에서 **Modal 버튼**을 연속 클릭하면
  * 같은 성격의 SubDlg가 중복 생성(또는 의도치 않은 다이얼로그 흐름)될 수 있습니다.

### 원인
* 버튼 핸들러가 “이미 SubDlg가 떠 있는지”를 체크하지 않으면, 클릭마다 생성 로직이 재실행됩니다.

### 수정 포인트
* `Sots01Dlg::OnBnClickedModalBtn`에서 `mp_sub_dlg` 포인터 유효성(존재 여부)을 먼저 검사합니다.

```cpp
void Sots01Dlg::OnBnClickedModalBtn()
{
    // ✅ Modeless SubDlg가 이미 떠 있으면 Modal 생성 금지
    if (!mp_sub_dlg) {
        SetDialogInfo();
        SubDlg ins_dlg(&m_dlg_info, MODAL_DIALOG);
        ins_dlg.DoModal();
    }
}
```

> 참고: Modeless SubDlg가 종료될 때 `WM_DESTROY_SUB_DIALOG` 처리에서 `mp_sub_dlg = nullptr;`로 정리되면,
> 이후에는 Modal 버튼이 정상 동작합니다.

### 간단 검증 체크리스트
* Modeless SubDlg 열기 → Modal 버튼 클릭: **새로 생성되지 않아야 함**
* Modeless SubDlg 닫기 → Modal 버튼 클릭: **정상적으로 Modal SubDlg가 떠야 함**

---

## 관련 파일
* `SubDlg.cpp` : 난수 생성/동적 할당/통계 계산
* `Sots01Dlg.cpp` : Modal/Modeless 버튼 핸들러 및 SubDlg 생명주기 관리
