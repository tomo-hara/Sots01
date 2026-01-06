# Patch Note v1.0.2 (2026-01-06)

## 1. SubDlg 메모리 동적 할당 로직 개선
### 🚩 문제점 (Issue)
난수 생성(`OnBnClickedGenRandBtn`) 시, 사용자가 입력한 개수(`array_count`)가 기존 배열 크기보다 클 경우 메모리 재할당(`delete` & `new`)은 수행되나, **멤버 변수 `m_array_count`가 갱신되지 않는 버그**가 있었습니다.
이로 인해 다음 연산 시 배열의 크기를 잘못 인식하여 Heap Corruption 또는 잘못된 통계 계산이 발생할 위험이 있었습니다.

### ✅ 해결책 (Solution)
`SubDlg.cpp`의 동적 할당 로직 직후에 멤버 변수를 최신화하는 코드를 명시적으로 추가했습니다.

```cpp
// SubDlg.cpp
void SubDlg::OnBnClickedGenRandBtn()
{
    // ... 기존 코드 ...
    if (array_count > m_array_count)  {
        delete[] mp_array;
        mp_array = new double[array_count];
        /*m_array_count = array_count;*/ // [Fix] 기존 위치
    }
    m_array_count = array_count; // [Fix] 수정된 멤버 변수 동기화 코드 위치
    // ...
}