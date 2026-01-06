# Patch Note v1.0.4 (2026-01-07)

이번 버전은 아래 4가지 개선 방향을 반영했습니다.

1. 통계 정보를 **내가 원하는 시점(버튼 클릭)** 에 계산하도록 구조 변경

2. 통계 함수 a_count 예외 처리 **(유효성 체크)** 

> ✅ 이번 버전에서 **STDDEV / MIN / MAX** 리팩토링을 완료했습니다.

---

## 1) 통계 계산을 ‘원하는 시점’에 수행하도록 변경

### 기존 흐름(요약)
* `Gen Rand` 실행 시점에 통계까지 함께 계산/표시되는 구조

### 변경된 흐름
* `Gen Rand`는 **난수 생성 + 리스트 표시**까지만 담당
* 통계는 각 버튼(`AVG`, `STD DEV`, `MIN/MAX`) 클릭 시점에 계산

---

## 2) 통계 함수 a_count 예외 처리

통계 함수에 선행하는 `Gen Rand` 버튼 클릭 이벤트 핸들러는 입력 배열과 함께 `a_count` 변수를 초기화하기 때문에, 아래 조건에서 **즉시 실패 처리**하도록 예외 처리를 추가했습니다.

* `a_count == 0` : 연산 불가(데이터 없음)


## TODO (다음 커밋)
> - `a_count == 1` : 평균/최소/최대는 가능하지만, 표준편차는 정의 방식에 따라 의미가 없거나 0 처리 필요
권장 방어 패턴(예시):
```cpp
BOOL SubDlg::GetStandardDeviation(double *ap_array, int a_count, double a_avg, double *result_STDDEV)
{
    if (a_count == 1) return FALSE;
    // 또는
    if (a_count < 2) return FALSE;
}
// stddev의 경우: if (a_count < 2) return FALSE;  // 정책에 따라 0으로 처리해도 OK
```

---

## 3) 생성 개수 입력값 파싱 보강 (GetDlgItemInt)

기존 `GetItemInt`가 **음수 입력을 그대로 반환**할 수 있어,
`UINT`/`size_t`로 변환되는 순간 **언더플로우(예: -1 → 매우 큰 양수)** 로 이어질 여지가 있었습니다.

이를 방지하기 위해, MFC의 `CWnd::GetDlgItemInt`를 사용하면서:

* `lpTrans(bTrans)`로 **변환 성공/실패를 구분**
* 생성 개수는 음수가 의미가 없으므로 `bSigned=FALSE`로 **unsigned 모드**로 처리

적용 코드(요약):
```cpp
BOOL bTrans = FALSE;
UINT array_count = GetDlgItemInt(IDC_GEN_COUNT_EDIT, &bTrans, FALSE);

if (bTrans || array_count > m_array_count)  {
    delete[] mp_array;
    mp_array = new double[array_count];
}
m_array_count = array_count;
```

> 추가 권장: 
- `!bTrans` 또는 `array_count == 0`일 때는 **메시지 출력 후 return** 하는 방식이 UX/안정성 측면에서 더 좋습니다.

## TODO (다음 커밋)
- 권장 패턴(예시):
```cpp
BOOL bTrans = FALSE;
UINT array_count = GetDlgItemInt(IDC_GEN_COUNT_EDIT, &bTrans, FALSE);

	if (bTrans || array_count) {
		if (array_count > m_array_count) {
			MessageBox(L"메모리가 동적할당 되었습니다.");
			delete[] mp_array;
			mp_array = new double[array_count];
		}
	} else MessageBox(L"올바른 값을 입력하세요.");
	
    m_array_count = array_count;
```

---

## TODO (다음 커밋)

> - 현재 `m_array_count`가 조건 없이 매번 갱신되고 있습니다. 이로 인해 이미 충분한 메모리가 할당되어 있더라도, 새로운 할당 크기에 맞춰 불필요한 재할당이 발생합니다.

- 권장 패턴(예시):
```cpp
	// ... 기존 코드 ...
	
	(m_array_count < array_count) ? (m_array_count = array_count) : m_array_count;
```