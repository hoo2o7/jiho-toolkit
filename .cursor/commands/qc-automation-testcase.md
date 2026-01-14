# qc-automation-testcase
 
## Overview
대화로 테스트케이스를 설계하고, **selector를 실제 DOM에서 검증(필수)**한 뒤 `qc-automation/testcases/`에 JSON으로 저장합니다.
 
## Steps
1. 테스트할 기능을 정의한다:
   - 기능/페이지(예: auth/login)
   - 시작 URL(예: `/login`)
   - 사전조건(로그인 필요 여부, 데이터 준비 등)
   - 수행 액션(입력/클릭/대기 등)
   - 기대 결과(URL/텍스트/요소 표시 등)
2. 테스트케이스 JSON을 작성해 `qc-automation/testcases/<feature>/TC-XXX.json`으로 저장한다:
 
```json
{
  "id": "TC-001",
  "name": "로그인 성공 테스트",
  "description": "유효한 자격증명으로 로그인이 성공하는지 확인",
  "feature": "auth",
  "priority": "high",
  "tags": ["login", "smoke"],
  "preconditions": ["서버가 실행 중", "테스트 계정이 존재함"],
  "setup": { "requiresAuth": false, "startUrl": "/login" },
  "steps": [
    { "order": 1, "action": "navigate", "target": "/login", "description": "로그인 페이지로 이동" },
    { "order": 2, "action": "fill", "selector": "[data-testid='email-input']", "value": "{{EMAIL}}", "description": "이메일 입력" },
    { "order": 3, "action": "fill", "selector": "[data-testid='password-input']", "value": "{{PASSWORD}}", "description": "비밀번호 입력" },
    { "order": 4, "action": "click", "selector": "[data-testid='login-button']", "description": "로그인 버튼 클릭" },
    { "order": 5, "action": "wait", "type": "navigation", "description": "페이지 이동 대기" },
    { "order": 6, "action": "assert", "type": "url_contains", "expected": "/dashboard", "description": "대시보드로 이동 확인" }
  ],
  "expectedResult": "로그인 성공 후 대시보드 페이지로 이동"
}
```
 
3. (필수) selector를 검증한다 — 추측 금지:
   - `/webapp-testing`로 해당 페이지를 열고 요소 selector를 확인한다
   - 각 selector는 **정확히 1개만 매칭**해야 한다(중복이면 스코프를 더 좁힌다)
   - 모달 내부 요소는 반드시 `[role='dialog']`로 스코프한다
   - 드롭다운 옵션은 `[role='listbox']`, `[role='option']` 컨벤션을 사용한다
4. 테스트케이스를 “사람이 읽기 쉬운 형태”로 요약해 리뷰하고, 수정 사항을 JSON에 반영한다.
5. (변경사항이 생긴 경우) `git status` 확인 → `git add` → `git commit`
 
## Checklist (선택)
- [ ] `data-testid`가 있으면 우선 사용했는가?
- [ ] 모달 내부 selector에 `[role='dialog']` 접두사가 붙어 있는가?
- [ ] 각 selector를 실제 페이지에서 확인했고, 매칭 count=1인가?
- [ ] 동적 UI(모달/드롭다운/토스트 등) 열린 상태의 selector도 검증했는가?
- [ ] (변경사항이 생긴 경우) 커밋까지 완료했는가?
