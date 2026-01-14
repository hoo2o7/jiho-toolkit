# qc-automation-execute
 
## Overview
테스트케이스를 실행 계획으로 묶고, **진행상태(progress) 저장/재개**가 가능한 형태로 자동 실행합니다. 실제 브라우저 실행은 **반드시 `/webapp-testing`**를 사용합니다.
 
## Steps
1. 실행 ID를 만든다(예: `EXEC-YYYYMMDD-001`) 그리고 폴더를 생성한다:
   - `qc-automation/executions/<EXEC-ID>/results/`
   - `qc-automation/executions/<EXEC-ID>/results/screenshots/`
2. `qc-automation/executions/<EXEC-ID>/plan.json` 생성:
 
```json
{
  "executionId": "EXEC-20260113-001",
  "name": "핵심 플로우 스모크 테스트",
  "createdAt": "2026-01-13T00:00:00Z",
  "testCases": ["TC-001", "TC-002"],
  "config": {
    "browser": "chromium",
    "headless": true,
    "baseUrl": "http://localhost:3000",
    "timeout": 30000,
    "retryOnFailure": 1,
    "screenshotOnFailure": true
  },
  "serverCommand": "pnpm dev",
  "serverPort": 3000,
  "serverReadyPattern": "ready"
}
```
 
3. `qc-automation/executions/<EXEC-ID>/progress.json`를 만든다(또는 초기화한다):
 
```json
{
  "executionId": "EXEC-20260113-001",
  "status": "ready",
  "total": 2,
  "completed": 0,
  "currentIndex": 0,
  "currentTestCase": null,
  "summary": { "passed": 0, "failed": 0, "skipped": 0 },
  "completedTests": [],
  "remainingTests": ["TC-001", "TC-002"],
  "lastUpdated": "2026-01-13T00:00:00Z"
}
```
 
4. 각 테스트케이스별로 `qc-automation/scripts/generated/TC-XXX.py`를 생성한다:
   - 입력 값은 `credentials.env`의 환경변수(`BASE_URL`, `EMAIL`, `PASSWORD` 등)를 사용한다
   - 실패 시 스크린샷을 저장하고, 결과 JSON에 경로를 남긴다
5. 테스트 실행 루프(필수 저장 원칙):
   - 실행 전: `progress.json`에 `currentTestCase`, `currentIndex` 업데이트
   - 실행: `/webapp-testing`로 Python 스크립트를 실행한다
     - 예: `/webapp-testing 실행: python qc-automation/scripts/generated/TC-001.py`
   - 실행 후: 결과를 `qc-automation/executions/<EXEC-ID>/results/TC-001.json`으로 저장
   - 진행 갱신: `progress.json`의 `completed`, `summary`, `completedTests`, `remainingTests`를 업데이트
6. 중단/재개:
   - 재개 시 `progress.json`을 읽고 `remainingTests`부터 이어서 실행한다
7. (변경사항이 생긴 경우) `git status` 확인 → `git add` → `git commit`
 
## Checklist (선택)
- [ ] 실제 실행은 `/webapp-testing`로만 수행했는가?
- [ ] 테스트 1개 실행할 때마다 `progress.json`과 결과 JSON이 저장되는가?
- [ ] 중단 후 `remainingTests` 기준으로 재개 가능한가?
- [ ] 실패 시 스크린샷 경로가 결과에 포함되는가?
- [ ] (변경사항이 생긴 경우) 커밋까지 완료했는가?
