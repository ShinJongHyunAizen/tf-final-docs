# FO-REG-PAG-001 — 회원가입 구현 플랜

## 전제 조건
- 휴대폰 본인인증 외부 모듈 연동 완료 (회원가입 버튼 클릭 시 본인인증 진행)
- 딥링크 타입 식별 API 존재 (직접 유입 / 회원가입 딥링크 / 데이터 연동 딥링크 구분)

---

## PLAN-01: 사업자 번호 인증 API 연동

사업자 번호 10자리 입력 후 외부 인증 API를 호출하여 설립일, 상호명, 대표자 정보를 반환받는다.
인증 성공 시 사업자 구분(개인/법인)을 번호 가운데 2자리 기준으로 분류하고, 인증된 필드를 변경 불가 처리한다.
인증 실패 시 에러 메시지를 노출하고 고객센터 안내 영역을 표시한다.

참조 섹션: Description 1 — 입력 필드 구성, 사업자 정보 인증 완료 후 노출 문구, 사업자 구분

예상 파일:
- `backend/src/main/java/com/example/application/service/BusinessAuthService.java`
- `backend/src/main/java/com/example/adapter/in/web/BusinessAuthController.java`
- `frontend/src/features/member/services/businessAuthService.ts`
- `frontend/src/features/member/components/BusinessInfoForm.tsx`

테스트 포인트:
- 사업자 번호 10자리 입력 시 인증 API 호출 확인
- 인증 성공 시 설립일/상호명/대표자 자동 입력 및 변경 불가 처리 확인
- 가운데 2자리 01~79/80/89/90~99 → 개인사업자, 81~87 → 법인사업자 분류 확인
- 인증 성공 후 `{사업자명} 님 사업자 정보 인증이 완료되었어요. 회원가입을 진행해 주세요.` 문구 노출 확인
- 에러: `사업자 번호를 '-'없이 숫자 10자리를 입력해 주세요.` 노출 확인

---

## PLAN-02: 아이디 중복 검사

아이디 입력 필드에서 포커스 아웃(blur) 시 중복확인 영역을 노출한다.
[아이디 중복 확인] 버튼 클릭 시 서버에서 대소문자를 무시한 중복 여부를 체크하고 결과에 따라 얼럿을 노출한다.

참조 섹션: Description 1 — 아이디 중복 검사 (#CCNLND-1895, #CCNLND-2585)

예상 파일:
- `backend/src/main/java/com/example/application/service/MemberService.java` (LOWER() 비교 쿼리)
- `backend/src/main/java/com/example/adapter/in/web/MemberController.java`
- `frontend/src/features/member/components/IdDuplicateCheck.tsx`

테스트 포인트:
- 아이디 입력 후 blur 시 중복확인 영역 노출 확인
- `@WaregeT02`와 `@wAregeT02` 동일 처리 확인 (대소문자 무시)
- 중복 아님(true): 얼럿 `사용 가능한 아이디 입니다.`, 중복확인 영역 사라짐 확인
- 중복(false): 얼럿 `가입이 불가한 아이디입니다.\n다른 아이디로 시도해 주세요.`, 영역 유지 확인
- 중복 확인 후 아이디 재입력 시 중복확인 영역 재노출 확인

---

## PLAN-03: 회원가입 폼 UI

사업자 인증 전/후 상태 UI, 비밀번호 마스킹/뷰 토글, 약관 동의, 도움말 문구, 버튼 활성화 조건을 구현한다.
비밀번호 재확인 필드는 없으며 단일 비밀번호 필드만 존재한다.

참조 섹션: Description 1 — 비밀번호 처리, 약관 동의, 가입 버튼, 도움말 문구

예상 파일:
- `frontend/src/features/member/components/SignUpForm.tsx`
- `frontend/src/features/member/components/PasswordField.tsx`
- `frontend/src/features/member/components/TermsAgreement.tsx`

테스트 포인트:
- 비밀번호 뷰 off 아이콘 클릭 → 마스킹 해제 및 on 아이콘 전환 확인
- 비밀번호 뷰 on 아이콘 클릭 → 마스킹 처리 및 off 아이콘 전환 확인
- 사업자명 도움말: `사업자명은 사업자등록증의 전체 사업자명을 입력해 주세요.` 표시 확인
- 대표자 도움말: `공동 대표인 경우, 첫 번째로 기재된 대표자 이름만 입력해 주세요.` 표시 확인
- 필드 에러메시지 노출 시 도움말이 에러 메시지 위로 노출 확인
- 회원가입 버튼 서브 타이틀 `대표자 명의 휴대폰으로 본인인증 후` 확인

---

## PLAN-04: 회원가입 API 연동 (본인인증 포함)

[회원가입] 버튼 클릭 시 대표자 명의 휴대폰 본인인증을 진행하고, 성공 시 회원가입 API를 호출한다.
유효성 검사는 노션 'FO 유효성 검사' 문서 기준으로 처리한다.

참조 섹션: Description 1 — 가입 버튼, 유효성 검사

예상 파일:
- `backend/src/main/java/com/example/application/service/MemberRegistrationService.java`
- `backend/src/main/java/com/example/adapter/in/web/MemberController.java`
- `frontend/src/features/member/services/memberService.ts`
- `frontend/src/pages/SignUpPage.tsx`

테스트 포인트:
- 본인인증 성공 후 회원가입 API 호출 확인
- 유효성 검사 실패 시 각 필드별 에러메시지 노출 확인
- 가입 완료 후 자동 로그인 처리 확인 (#CCNLND-2326)

---

## PLAN-05: 가입 완료 후 팝업 (딥링크 타입별 분기)

회원가입 완료 후 딥링크 유입 타입에 따라 팝업을 분기하여 노출한다.
[닫기] 버튼 클릭 시 데이터 연동하기 화면으로 이동한다 (#CCNLND-2326 변경사항).

참조 섹션: Description 2 — 가입 완료 후 팝업 (#1878)

예상 파일:
- `frontend/src/features/member/components/SignUpCompleteModal.tsx`
- `frontend/src/features/member/hooks/useSignUpComplete.ts`

테스트 포인트:
- 직접 유입 시 `필수몰X & 비담보` 조건 팝업 vs 그 외 팝업 분기 확인
- `필수몰X & 비담보`: 버튼 [닫기], [데이터 제공 동의] 확인
- 그 외: 버튼 [닫기], [판매몰 연동] 확인
- 데이터 연동 딥링크 인입 시 3-1(데이터 제공 동의 → FO-DEP-PAG-004) / 3-2(판매몰 연동 → FO-DIF-PAG-002_1) 분기 확인
- [닫기] 클릭 시 데이터 연동하기 이동 확인 (기존 딥링크로 돌아가지 않음)

확인필요: [딥링크 타입 구분 기준 — 서버에서 응답 플래그로 내려주는지, 프론트 로컬 상태로 판단하는지]

---

## PLAN-06: 통합 테스트

참조 섹션: Description 1 전체, Description 2 전체

테스트 포인트:
- 사업자 번호 인증 → 아이디 중복 확인 → 회원가입 전체 플로우 확인
- 직접 유입 가입 완료 → 팝업 노출 → [닫기] → 데이터 연동하기 이동 확인
- 데이터 연동 딥링크 인입 가입 완료 → 담보/비담보/필수몰 조합별 팝업 분기 확인
- 고객센터 안내 영역 '1:1 문의' 클릭 → 비회원(FO-CSM-PAG-002) / 회원(FO-CSM-PAG-004) 분기 확인
