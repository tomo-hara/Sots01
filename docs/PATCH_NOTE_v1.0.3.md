# Patch Note v1.0.3 (2026-01-06)

이번 버전은 아래 4가지 개선 방향을 반영했습니다.

1. 난수값 범위를 **-100.0 ~ 100.0**으로 변경
2. 통계 정보를 **내가 원하는 시점(버튼 클릭)** 에 계산하도록 구조 변경
3. 반복문 내 변수 선언을 줄이기(루프 바디에서의 선언 최소화)
4. 함수 프로토타입은 **포인터(out param)** 로 결과를 받도록 정리

> 이번 커밋에서는 **통계 중 ‘평균(AVG)’만 우선 구현**했습니다. 표준편차/최소·최대는 다음 커밋에서 확장합니다.

---

## 1) 난수 생성 범위 변경 (-100.0 ~ 100.0)

### 변경 내용
* `Gen Rand` 버튼으로 생성되는 난수 범위를 `[-100.0, 100.0]`로 변경했습니다.

### 코드 포인트
```cpp
// SubDlg::OnBnClickedGenRandBtn
srand((unsigned int)time(NULL));

double lower_bound = -100.0;
double upper_bound = 100.0;

for (UINT i = 0; i < m_array_count; i++) {
    mp_array[i] = lower_bound + (upper_bound - lower_bound) * ((double)rand() / RAND_MAX);
}
```

---

## 2) 통계 계산을 ‘원하는 시점’에 수행하도록 변경 (평균만 구현)

### 기존 흐름(요약)
* `Gen Rand` 실행 시점에 통계까지 함께 계산/표시되는 구조

### 변경된 흐름
* `Gen Rand`는 **난수 생성 + 리스트 표시**까지만 담당
* 통계는 각 버튼(`AVG`, 이후 `STD DEV`, `MIN/MAX`) 클릭 시점에 계산

> 이 커밋에서는 `AVG` 버튼에 대해서만 계산/표시 기능 연동을 해제하였습니다.

---

## 3) 평균(AVG) 계산 함수 리팩터링

### 목표
* 평균 계산을 UI 로직과 분리하고, 결과는 **out param(포인터)** 로 전달
* 계산 성공/실패 여부를 반환값으로 알 수 있게 처리

### 변경 내용
* `GetAverage(double* ap_array, int a_count, double* result_AVG)` 형태로 정리
* 루프 바디에서 불필요한 선언을 줄이고, 포인터 기반 반복으로 합계를 계산

### 코드 포인트
```cpp
// 평균 계산 (성공 시 TRUE)
BOOL SubDlg::GetAverage(double* ap_array, int a_count, double* result_AVG)
{
    if (!a_count) return FALSE;

    double* p_value = ap_array;
    double* p_limit = ap_array + a_count;
    double sum = 0.0;

    while (p_value < p_limit) {
        sum += *p_value;
        ++p_value;
    }

    *result_AVG = sum / a_count;
    return TRUE;
}
```

### UI 연결(AVG 버튼)
```cpp
void SubDlg::OnBnClickedAvgBtn()
{
    double result_AVG = 0.0;
    int ok = GetAverage(mp_array, m_array_count, &result_AVG);

    if (ok) {
        wchar_t buffer[MAX_DOUBLE_LENGTH];
        swprintf(buffer, MAX_DOUBLE_LENGTH, L"%2.6f", result_AVG);
        SetDlgItemText(IDC_AVG_STATIC, buffer);
    } else {
        MessageBox(L"평균을 구하지 못했습니다.");
    }
}
```

---

## 4) 테스트 체크리스트

1. SubDlg에서 생성 개수를 입력
2. `Gen Rand` 클릭 → 리스트에 난수가 표시되는지 확인
3. 값이 **음수/양수 모두** 포함되는지 확인(범위: -100.0 ~ 100.0)
4. `AVG` 클릭 → 평균이 표시되는지 확인
5. 생성 개수를 바꾸고 다시 `Gen Rand` → `AVG`를 반복해도 정상 동작하는지 확인

---

## TODO (다음 커밋)
* `STD DEV`(표준편차), `MIN/MAX` 계산도 `AVG`와 동일한 패턴으로:
  * 계산 함수는 out param 기반
  * UI는 버튼 클릭 시점에 계산/표시
