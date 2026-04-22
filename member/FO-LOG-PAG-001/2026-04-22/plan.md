# FO-LOG-PAG-001 — 로그인 계정 잠금 구현 플랜

## 전제 조건
- 로그인 API 엔드포인트 존재 (계정 잠금 상태 응답값 포함)
- 세션 식별 가능한 환경 (세션 ID 또는 IP 기반 추적 가능)

확인필요: [공통 전제(10회) vs Description 1(15회) — 잠금 기준 횟수 불일치, 어느 쪽을 구현 기준으로 삼을지 확인 후 진행]
확인필요: [Description 1(10분간 로그인 불가) vs Description 2(5분간 로그인 불가) — 잠금 지속 시간 불일치, 확인 후 진행]

---

## PLAN-01: 로그인 시도 횟수 추적 (세션 기반 — 케이스 a)

동일 세션에서 1분 내 로그인 시도 횟수를 서버 측에서 추적한다.
Description 1 기준 **15회** 이상 실패 시 **10분**간 해당 세션의 로그인을 불가 처리한다.
다른 세션에서의 시도는 별도로 카운팅하며, 케이스 a 잠금은 세션 단위로 독립 처리한다.

참조 섹션: 공통 전제 — 케이스 a, Description 1 — 계정을 무작위로 틀렸을 때

예상 파일:
- `backend/src/main/java/com/example/application/service/LoginLockService.java`
- `backend/src/main/java/com/example/domain/repository/LoginAttemptRepository.java`

테스트 포인트:
- 동일 세션에서 1분 내 15회 실패 시 잠금 처리 확인
- 잠금 후 동일 세션에서 로그인 시도 시 불가 확인
- 다른 세션에서 시도 시 로그인 가능 확인 (케이스 a 예외)
- 1분 경과 후 카운터 초기화 확인

확인필요: [공통 전제(10회) vs Description 1(15회) 불일치 — 구현 기준 횟수 확정 전 구현 보류]

---

## PLAN-02: 로그인 시도 횟수 추적 (서버 IP 기반 — 케이스 b)

서버 요청 IP 기준으로 1분 내 로그인 시도 횟수를 추적한다.
Description 1 기준 **15회** 이상 실패 시 **10분**간 해당 IP의 로그인을 불가 처리한다.
케이스 a와 독립적으로 동작하며 병행 적용된다.

참조 섹션: Description 1 — 계정을 무작위로 틀렸을 때 케이스 b

예상 파일:
- `backend/src/main/java/com/example/application/service/LoginLockService.java`
- `backend/src/main/java/com/example/domain/repository/LoginAttemptRepository.java`

테스트 포인트:
- 동일 IP에서 1분 내 15회 실패 시 잠금 처리 확인
- 잠금 후 동일 IP에서 로그인 시도 시 불가 확인
- 다른 IP에서 시도 시 케이스 b 잠금 미적용 확인

---

## PLAN-03: 동일 아이디 비밀번호 오류 횟수 추적 (케이스 c)

동일 아이디에 대한 비밀번호 오류 횟수를 서버에서 누적 추적한다.
Description 1 기준 **15회** 이상 틀린 경우 해당 아이디를 **10분**간 잠금 처리한다.
세션이 달라도 동일 아이디 기준으로 합산하며, 잠금 이후 다른 세션에서 시도해도 잠금 유지.

참조 섹션: 공통 전제 — 케이스 c, Description 1 — 동일 아이디를 계속 틀렸을 때

예상 파일:
- `backend/src/main/java/com/example/application/service/LoginLockService.java`
- `backend/src/main/java/com/example/domain/repository/LoginAttemptRepository.java`

테스트 포인트:
- 동일 아이디로 비밀번호 15회 오류 시 잠금 처리 확인
- **다른 세션에서 시도해도 총 시도 횟수 합산 확인** (동일 아이디 기준, 총 15번 기준)
- **잠금 이후 다른 세션에서 시도해도 잠금 유지 확인**
- 10분 경과 후 잠금 해제 확인

확인필요: ['각각 다른 세션에서 10번 시도를 하더라도 총 15번이면 잠금' 원문 — 세션당 10번/총합 15번 관계 불명확]

---

## PLAN-04: 계정 잠금 응답 처리 (API 응답값 정의)

로그인 API에서 계정 잠금 상태와 잠금 만료 시간을 응답값으로 내려준다.
10번째 시도에서 실패 확인 + 잠금 처리가 동시에 이루어지며, 잠금 리턴값을 반환한다.
프론트엔드에서 잔여 시간 계산에 사용할 잠금 만료 타임스탬프를 포함한다.

참조 섹션: Description 2 — 노출 조건

예상 파일:
- `backend/src/main/java/com/example/adapter/in/web/LoginController.java`
- `backend/src/main/java/com/example/application/dto/LoginResponse.java`

테스트 포인트:
- 잠금 조건 충족 시 잠금 상태(`LOCKED`) + 만료 타임스탬프 응답 확인
- 이미 잠긴 상태에서 시도 시 동일 응답 포맷 확인
- 정상 로그인 시 잠금 상태 미포함 응답 확인

확인필요: [Description 2 얼럿 노출 조건 '10번째' vs Description 1 잠금 기준 '15회' 불일치 — 잠금 응답 반환 시점 확정 필요]

---

## PLAN-05: 로그인 잠금 얼럿창 UI

잠금 응답 수신 시 잠금 얼럿창을 노출한다.
잔여 시간을 `05:00`부터 `HH:MM` 형식으로 카운트다운하여 표시한다.
`[확인]` 클릭으로 얼럿을 닫아도 카운팅은 계속 진행되며, 다시 `[로그인]` 시도 시 잔여 시간을 이어서 표시한다.

참조 섹션: Description 2 — 얼럿 문구, 버튼, 남은 시간 카운팅 표기

예상 파일:
- `frontend/src/features/member/components/LoginLockAlert.tsx`
- `frontend/src/features/member/hooks/useLoginLockTimer.ts`

테스트 포인트:
- 잠금 시 얼럿 문구 노출 확인 (원문 그대로):
  ```
  아이디 또는 비밀번호 입력이 10회 이상 틀려 계정이 보호(잠금) 처리되었습니다.
  안전한 서비스 이용을 위해 5분 후 로그인을 다시 시도할 수 있습니다.
  잠시 기다려 주시기 바랍니다.

  남은 시간 : 04:59
  ```
- `[확인]` 클릭 후 얼럿 닫힘 확인
- 얼럿 닫은 후 다시 로그인 시도 시 → 잔여 카운팅 유지 확인
- 카운트다운 만료 후 로그인 가능 상태 확인

확인필요: [얼럿 문구 내 '5분'과 Description 1의 '10분간 로그인 불가' 불일치 — 카운트다운 기준 시간(05:00 vs 10:00) 확정 필요]

---

## PLAN-06: 통합 테스트

참조 섹션: Description 1 전체, Description 2 전체

테스트 포인트:
- 케이스 a: 동일 세션 15회 실패 → 잠금 → 얼럿 → 만료 후 해제 전체 플로우 확인
- 케이스 a 예외: 잠금된 세션 A에서 다른 세션 B로 시도 → 로그인 가능 확인
- 케이스 b: 동일 IP 15회 실패 → 잠금 → 얼럿 전체 플로우 확인
- 케이스 c: 동일 아이디 비밀번호 15회 오류 → 잠금 → 다른 세션에서 시도해도 잠금 유지 확인
- 잠금 중 `[확인]` 클릭 → 다시 로그인 시도 → 잔여 카운팅 이어서 노출 확인
