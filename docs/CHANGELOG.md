# Changelog
## [1.0.2] - 2026-01-06
### Fixed
- **Critical:** `SubDlg`에서 동적 배열 재할당 시 멤버 변수(`m_array_count`)가 갱신되지 않던 로직 오류 수정.

## [1.0.1]

### Fixed
- Modeless 대화상자가 실행 중일 때 Modal 대화상자가 생성되지 않도록 예외 처리 추가 (`OnBnClickedModalBtn` 내 `mp_sub_dlg` 체크 로직 적용).
