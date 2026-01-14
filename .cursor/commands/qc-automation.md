# qc-automation
 
## Overview
Playwright 기반 QA 자동화 작업을 **설계(테스트케이스) → 실행(진행상태 저장/재개) → 리포트**까지 한 번에 진행하기 위한 커맨드입니다. 필요하면 단계별 커맨드(`qc-automation-init`, `qc-automation-testcase`, `qc-automation-execute`, `qc-automation-report`)로 쪼개서 실행합니다.
 
## Steps
1. `qc-automation-init` 실행: 폴더 구조 + 설정/자격증명 템플릿 준비
2. `qc-automation-testcase` 실행: 테스트케이스 JSON 작성 + selector 검증(필수)
3. `qc-automation-execute` 실행: 실행 계획(plan) 생성 + 진행상태(progress) 저장하며 자동 실행(/webapp-testing 사용)
4. `qc-automation-report` 실행: 결과를 HTML/JSON 리포트로 정리
5. (변경사항이 생긴 경우) `git status` 확인 → `git add` → `git commit`
 
## Checklist (선택)
- [ ] `qc-automation/config/settings.json`에 baseUrl/timeout 등이 설정되어 있는가?
- [ ] selector가 “추측”이 아니라 실제 DOM에서 검증되었는가? (각 selector count=1)
- [ ] 실행 중단 시 `executions/<EXEC-ID>/progress.json`으로 재개 가능한가?
- [ ] 실패 시 스크린샷/에러가 결과에 남는가?
- [ ] (변경사항이 생긴 경우) 커밋까지 완료했는가?
