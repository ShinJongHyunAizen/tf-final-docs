# FO-PWF-PAG-002 — 비밀번호 재설정 구현 플랜

## 전제 조건
- FO-PWF-PAG-001 (비밀번호 찾기) 본인인증 성공 후 진입
- 비밀번호 재설정 토큰 또는 세션이 본인인증 성공 시 발급되어 있어야 함

---

## PLAN-01: 비밀번호 재설정 API 연동

본인인증 후 발급된 토큰/세션과 함께 새 비밀번호를 서버에 전송한다.
재설정 성공 시 성공 팝업(3-6)을 노출한다.

참조 섹션: Description 1 — 버튼, 재설정 결과

예상 파일:
- `backend/src/main/java/com/example/application/service/PasswordResetService.java`
- `backend/src/main/java/com/example/adapter/in/web/PasswordResetController.java`
- `frontend/src/features/member/services/passwordResetService.ts`

테스트 포인트:
- 비밀번호 입력 후 `[비밀번호 재설정]` 클릭 시 API 호출 확인
- 재설정 성공 시 3-6 팝업 노출 확인

확인필요: [비밀번호 유효성 검사 규칙 (최소 길이, 특수문자 필수 여부 등) — 문서에 명시 없음]
확인필요: [재설정 실패 케이스 처리 — 문서에 명시 없음]

---

## PLAN-02: 비밀번호 재설정 화면 UI

안내 문구와 비밀번호 입력 필드, `[비밀번호 재설정]` 버튼을 표시한다.
기본 상태는 입력 마스킹 + 뷰 off 아이콘이며, 아이콘 클릭으로 마스킹 토글이 가능하다.
비밀번호를 입력했을 때만 버튼이 활성화된다.

참조 섹션: Description 1 — 안내 문구, 비밀번호 입력 필드, 버튼

예상 파일:
- `frontend/src/features/member/components/PasswordResetForm.tsx`
- `frontend/src/pages/PasswordResetPage.tsx`

테스트 포인트:
- 안내 문구 노출 확인:
  ```
  본인 확인되었습니다.
  비밀번호 재설정을 해주세요.
  ```
- 비밀번호 필드 플레이스홀더 `비밀번호` 노출 확인
- 기본 상태: 마스킹 처리 + 뷰 off 아이콘 확인
- 뷰 off 아이콘 클릭 → 입력값 노출 + 뷰 on 아이콘으로 전환 확인
- 뷰 on 아이콘 클릭 → 마스킹 처리 + 뷰 off 아이콘으로 전환 확인
- 비밀번호 미입력 시 `[비밀번호 재설정]` 버튼 비활성화 확인
- 비밀번호 입력 시 버튼 활성화 확인

---

## PLAN-03: 재설정 성공 팝업 (3-6)

비밀번호 재설정 성공 시 3-6 팝업을 노출한다.
`[로그인]` 클릭 시 FO-LOG-PAG-001(로그인) 화면으로 이동한다.

참조 섹션: Description 1 — 재설정 결과

예상 파일:
- `frontend/src/features/member/components/PasswordResetSuccessModal.tsx`

테스트 포인트:
- 성공 팝업 문구 노출 확인:
  ```
  비밀번호가 재설정되었어요.
  로그인을 진행해 주세요.
  ```
- `[로그인]` 클릭 → FO-LOG-PAG-001 이동 확인

---

## PLAN-04: 통합 테스트

참조 섹션: Description 1 전체

테스트 포인트:
- 비밀번호 입력 → `[비밀번호 재설정]` → 성공 팝업 → `[로그인]` → 로그인 화면 전체 플로우 확인
- 마스킹 토글 off → on → off 반복 동작 확인
- 버튼 활성화/비활성화 상태 전환 확인
