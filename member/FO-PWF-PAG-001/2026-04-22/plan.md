# FO-PWF-PAG-001 — 비밀번호 찾기 구현 플랜

## 전제 조건
- 휴대폰 본인인증 외부 모듈 연동 완료
- CI값 + 아이디 조합으로 회원 검증 가능한 API 존재
- 기존 사업자번호 기반 인증 흐름 제거 (인증 방식 변경)

---

## PLAN-01: 비밀번호 찾기 API 연동 (아이디 + CI값 검증)

아이디 입력값과 본인인증으로 획득한 CI값을 서버에서 검증한다.
아이디와 CI값이 일치하면 비밀번호 재설정 화면(FO-PWF-PAG-002)으로 이동한다.
불일치 케이스(CI값 가입 이력 없음 / 아이디 불일치)에 따라 각각 다른 팝업을 노출한다.

참조 섹션: Description 1 — 3-3. 휴대폰 본인인증

예상 파일:
- `backend/src/main/java/com/example/application/service/PasswordFindService.java`
- `backend/src/main/java/com/example/adapter/in/web/PasswordFindController.java`
- `frontend/src/features/member/services/passwordFindService.ts`

테스트 포인트:
- 아이디 + 본인인증 성공 → FO-PWF-PAG-002 이동 확인
- CI값으로 가입 이력 없음 → 3-4 팝업 노출 확인
- 아이디와 CI값 불일치 → 3-5 팝업 노출 확인

---

## PLAN-02: 비밀번호 찾기 화면 UI

안내 문구와 아이디 입력 필드, `[휴대폰 본인인증]` 버튼을 표시한다.
버튼 클릭 시 아이디 값을 포함하여 외부 본인인증 모듈을 호출한다.

참조 섹션: Description 1 — 3-1. 안내 문구, 3-2. 아이디 입력 필드, 3-3. 휴대폰 본인인증

예상 파일:
- `frontend/src/features/member/components/PasswordFindForm.tsx`
- `frontend/src/pages/PasswordFindPage.tsx`

테스트 포인트:
- 안내 문구 노출 확인:
  ```
  가입한 아이디를 입력해 주세요.
  휴대폰 본인인증을 통해
  비밀번호를 변경할 수 있습니다.
  ```
- 아이디 필드 플레이스홀더 `아이디` 노출 확인
- `[휴대폰 본인인증]` 버튼 클릭 시 외부 모듈 호출 확인

---

## PLAN-03: 실패 팝업 — CI값 가입 이력 없음 (3-4)

본인인증 CI값으로 가입된 아이디를 찾을 수 없는 경우 3-4 팝업을 노출한다.
`[확인]` 클릭 시 팝업이 닫히고 FO-PWF-PAG-001 화면으로 돌아온다.
`[1:1 문의하기]` 클릭 시 FO-CSM-PAG-002(1:1문의)로 이동한다.

참조 섹션: Description 1 — 3-4. 실패 팝업 — CI값으로 가입 이력 없음

예상 파일:
- `frontend/src/features/member/components/PasswordFindFailCIModal.tsx`

테스트 포인트:
- 3-4 팝업 문구 노출 확인:
  ```
  본인 명의 휴대전화로 가입된 아이디를 찾을 수 없어요.
  고객센터 문의 혹은 비밀번호 찾기를 다시 시도해 주세요.
  ```
- `[확인]` 클릭 → 팝업 닫힘 + FO-PWF-PAG-001 화면 유지 확인
- `[1:1 문의하기]` 클릭 → FO-CSM-PAG-002 이동 확인

---

## PLAN-04: 실패 팝업 — 아이디 불일치 (3-5)

입력한 아이디와 본인인증 CI값이 불일치하는 경우 3-5 팝업을 노출한다.
`[확인]` 클릭 시 팝업이 닫히고 FO-PWF-PAG-001 화면으로 돌아온다.
`[1:1 문의하기]` 클릭 시 FO-CSM-PAG-002(1:1문의)로 이동한다.

참조 섹션: Description 1 — 3-5. 실패 팝업 — 아이디 불일치

예상 파일:
- `frontend/src/features/member/components/PasswordFindFailIdModal.tsx`

테스트 포인트:
- 3-5 팝업 문구 노출 확인:
  ```
  아이디가 일치하지 않아요.
  고객센터 문의 혹은 비밀번호 찾기를 다시 시도해 주세요.
  ```
- `[확인]` 클릭 → 팝업 닫힘 + FO-PWF-PAG-001 화면 유지 확인
- `[1:1 문의하기]` 클릭 → FO-CSM-PAG-002 이동 확인

---

## PLAN-05: 통합 테스트

참조 섹션: Description 1 전체

테스트 포인트:
- 아이디 입력 → 본인인증 성공 + 일치 → FO-PWF-PAG-002 이동 전체 플로우 확인
- 본인인증 성공 + CI값 가입 이력 없음 → 3-4 팝업 → `[확인]` → 찾기 화면 복귀 확인
- 본인인증 성공 + 아이디 불일치 → 3-5 팝업 → `[1:1 문의하기]` → FO-CSM-PAG-002 이동 확인
- 사업자번호 입력 필드 미노출 확인 (기존 인증 방식 제거 검증)
