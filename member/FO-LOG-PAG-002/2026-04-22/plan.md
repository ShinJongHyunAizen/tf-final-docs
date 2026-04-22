# FO-LOG-PAG-002 — 로그인 후 본인인증 구현 플랜

## 전제 조건
- 정상 로그인 처리 완료 후 CI값 존재 여부를 확인할 수 있는 API 또는 로그인 응답값 포함
- 휴대폰 본인인증 외부 모듈 연동 완료
- 비밀번호 변경일, 변경 연장여부, 연장 승인일 조회 가능 (회원 정보 API)

---

## PLAN-01: CI값 기반 본인인증 진입 분기

로그인 성공 후 서버로부터 CI값 존재 여부를 확인한다.
CI값이 빈값인 경우 본인인증 없이 대출 비교하기 화면으로 이동한다.
CI값이 존재하는 경우 본인인증 화면(1번)으로 진입한다.

참조 섹션: Description 1 — CI값 확인 분기

예상 파일:
- `backend/src/main/java/com/example/application/service/LoginPostProcessService.java`
- `frontend/src/features/member/hooks/usePostLoginFlow.ts`

테스트 포인트:
- 로그인 후 CI값 빈값 → 대출 비교하기 이동 확인
- 로그인 후 CI값 존재 → 본인인증 화면 진입 확인

---

## PLAN-02: 본인인증 화면 UI 및 처리

본인인증 안내 문구와 `[본인인증 하기]` 버튼을 표시한다.
버튼 클릭 시 외부 본인인증 모듈을 호출한다.
성공 시 2번 처리로 이동하고, 실패 시 실패 문구를 노출한다.

참조 섹션: Description 1 — 1. 본인인증 화면

예상 파일:
- `frontend/src/features/member/components/IdentityVerificationScreen.tsx`
- `frontend/src/features/member/services/identityVerificationService.ts`

테스트 포인트:
- 안내 문구 노출 확인:
  ```
  대표자 정보가 변경되었습니다.
  안전한 서비스 이용을 위해
  본인인증을 진행해 주세요.
  ```
- `[본인인증 하기]` 버튼 클릭 시 외부 모듈 호출 확인
- 본인인증 실패 시 문구 노출 확인:
  ```
  본인인증에 실패하였습니다.
  다시 한번 시도해 주세요.
  ```

---

## PLAN-03: 본인인증 성공 처리 및 비밀번호 변경 분기

본인인증 성공 후 성공 문구와 `[확인]` 버튼을 표시한다.
`[확인]` 클릭 시 비밀번호 변경 여부 API를 호출하여 3가지 경우로 분기한다.
- 변경 연장여부 'N' + 마지막 비밀번호 변경일 기준 90일 경과 → 2-1 팝업
- 변경 연장여부 'Y' + 연장 승인일 기준 90일 경과 → 2-2 팝업
- 해당 없음 → 마이페이지 이동

참조 섹션: Description 1 — 2. 본인인증 성공 처리, 비밀번호 변경 팝업 분기

예상 파일:
- `backend/src/main/java/com/example/application/service/PasswordChangeCheckService.java`
- `frontend/src/features/member/hooks/usePostVerificationFlow.ts`
- `frontend/src/features/member/components/IdentityVerificationSuccessScreen.tsx`

테스트 포인트:
- 성공 문구 노출 확인:
  ```
  본인인증이 완료되었습니다.
  이제 대표자 권한으로 서비스를 이용하실 수 있습니다.
  ```
- `[확인]` 클릭 후 비밀번호 변경 API 호출 확인
- 연장여부 'N' + 90일 초과 → 2-1 팝업 노출 확인
- 연장여부 'Y' + 연장 승인일 기준 90일 초과 → 2-2 팝업 노출 확인
- 해당 없음 → 마이페이지 이동 확인

---

## PLAN-04: 비밀번호 변경 권고 팝업 (2-1) — 90일 경과

비밀번호 변경 연장여부 'N' + 90일 경과 시 권고 팝업을 노출한다.
`[90일 후 변경]` 클릭 시 마이페이지로 이동하고, `[비밀번호 변경]` 클릭 시 비밀번호 변경 화면으로 이동한다.

참조 섹션: Description 1 — 2-1. 비밀번호 변경 권고 팝업

예상 파일:
- `frontend/src/features/member/components/PasswordChangeRecommendModal.tsx`

테스트 포인트:
- 팝업 문구 노출 확인:
  ```
  비밀번호를 변경하신 지 90일이 경과되었습니다.
  안전한 서비스 이용을 위해 비밀번호 변경을 권장드립니다.
  ```
- `[90일 후 변경]` 클릭 → 마이페이지 이동 확인
- `[비밀번호 변경]` 클릭 → 마이페이지 내 비밀번호 변경 화면 이동 확인

확인필요: [비밀번호 변경 화면 경로 — 마이페이지 내 정확한 화면 ID 불명확]

---

## PLAN-05: 비밀번호 변경 강제 팝업 (2-2) — 180일 경과

비밀번호 변경 연장여부 'Y' + 연장 승인일 기준 90일 경과 시 강제 팝업을 노출한다.
`[비밀번호 변경]` 버튼만 존재하며, 클릭 시 비밀번호 변경 화면으로 이동한다.

참조 섹션: Description 1 — 2-2. 비밀번호 변경 강제 팝업

예상 파일:
- `frontend/src/features/member/components/PasswordChangeForcedModal.tsx`

테스트 포인트:
- 팝업 문구 노출 확인:
  ```
  비밀번호를 변경하신 지 180일이 경과되었습니다.
  안전한 서비스 이용을 위해 비밀번호 변경이 필요합니다.
  ```
- `[비밀번호 변경]` 버튼만 노출 확인
- `[비밀번호 변경]` 클릭 → 비밀번호 변경 화면 이동 확인

확인필요: [2-2 팝업에서 '변경' 없이 닫기 가능한지 여부 — 강제 여부 불명확]

---

## PLAN-06: 통합 테스트

참조 섹션: Description 1 전체

테스트 포인트:
- CI값 없음 → 대출 비교하기 직행 확인
- CI값 있음 → 본인인증 화면 → 성공 → 비밀번호 조건 없음 → 마이페이지 전체 플로우 확인
- CI값 있음 → 본인인증 실패 → 실패 문구 → 재시도 가능 확인
- 연장여부 'N' + 90일 초과 → 2-1 팝업 → `[90일 후 변경]` → 마이페이지 확인
- 연장여부 'Y' + 90일 초과 → 2-2 팝업 → `[비밀번호 변경]` → 변경 화면 확인
