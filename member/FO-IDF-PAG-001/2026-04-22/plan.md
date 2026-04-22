# FO-IDF-PAG-001 — 아이디 찾기 구현 플랜

## 전제 조건
- 휴대폰 본인인증 외부 모듈 연동 완료
- CI값 기반으로 회원 아이디를 조회할 수 있는 API 존재

---

## PLAN-01: 아이디 찾기 API 연동 (CI값 기반 조회)

휴대폰 본인인증으로 획득한 CI값으로 가입된 아이디를 서버에서 조회한다.
조회 성공 시 FO-IDF-PAG-002(아이디 찾기 완료) 화면으로 이동한다.
CI값으로 가입 이력이 없는 경우 실패 팝업을 노출한다.

참조 섹션: Description 1 — 1-2. 휴대폰 본인인증, 1-3. 아이디 찾기 실패 팝업

예상 파일:
- `backend/src/main/java/com/example/application/service/MemberIdFindService.java`
- `backend/src/main/java/com/example/adapter/in/web/MemberIdFindController.java`
- `frontend/src/features/member/services/idFindService.ts`

테스트 포인트:
- 본인인증 성공 + CI값 일치 회원 존재 → FO-IDF-PAG-002 이동 확인
- 본인인증 성공 + CI값 일치 회원 없음 → 실패 팝업 노출 확인
- 본인인증 실패 → 실패 팝업 노출 확인

---

## PLAN-02: 아이디 찾기 화면 UI

안내 문구와 `[휴대폰 본인인증]` 버튼을 표시한다.
버튼 클릭 시 외부 본인인증 모듈을 호출하고, 결과에 따라 성공/실패 분기를 처리한다.

참조 섹션: Description 1 — 공통 전제, 1-1. 안내 문구, 1-2. 휴대폰 본인인증

예상 파일:
- `frontend/src/features/member/components/IdFindForm.tsx`
- `frontend/src/pages/IdFindPage.tsx`

테스트 포인트:
- 안내 문구 노출 확인:
  ```
  휴대폰 본인인증을 통해
  아이디를 찾을 수 있습니다.
  ```
- `[휴대폰 본인인증]` 버튼 클릭 시 외부 모듈 호출 확인

---

## PLAN-03: 아이디 찾기 실패 팝업 UI

본인인증으로 가입 이력을 찾지 못한 경우 실패 팝업을 노출한다.
`[1:1 문의하기]` 클릭 시 FO-CSM-PAG-002로, `[회원가입]` 클릭 시 FO-REG-PAG-001로 이동한다.

참조 섹션: Description 1 — 1-3. 아이디 찾기 실패 팝업

예상 파일:
- `frontend/src/features/member/components/IdFindFailModal.tsx`

테스트 포인트:
- 실패 팝업 문구 노출 확인:
  ```
  본인 명의 휴대전화로 가입된 아이디를 찾을 수 없어요.
  고객센터 문의 혹은 회원가입을 진행해 주세요.
  ```
- `[1:1 문의하기]` 클릭 → FO-CSM-PAG-002 이동 확인
- `[회원가입]` 클릭 → FO-REG-PAG-001 이동 확인

---

## PLAN-04: 통합 테스트

참조 섹션: Description 1 전체

테스트 포인트:
- 본인인증 성공 + 회원 존재 → FO-IDF-PAG-002 이동 전체 플로우 확인
- 본인인증 성공 + 회원 없음 → 실패 팝업 → `[회원가입]` → FO-REG-PAG-001 이동 확인
- 본인인증 성공 + 회원 없음 → 실패 팝업 → `[1:1 문의하기]` → FO-CSM-PAG-002 이동 확인
- 기존 사업자번호 기반 인증 제거 확인 (변경 전 흐름 미노출 확인)
