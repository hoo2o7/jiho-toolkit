# qc-automation-report
 
## Overview
`qc-automation/executions/<EXEC-ID>/results/`에 저장된 결과를 기반으로 **사람이 읽기 쉬운 HTML 리포트**와 **프로그램용 JSON 요약**을 생성합니다.
 
## Steps
1. 대상 실행 ID(`<EXEC-ID>`)를 정하고, 아래 입력이 존재하는지 확인한다:
   - `qc-automation/executions/<EXEC-ID>/plan.json`
   - `qc-automation/executions/<EXEC-ID>/progress.json`
   - `qc-automation/executions/<EXEC-ID>/results/*.json`
2. HTML 리포트 산출물 경로를 정한다:
   - `qc-automation/reports/<EXEC-ID>-report.html`
3. JSON 요약 산출물 경로를 정한다:
   - `qc-automation/reports/<EXEC-ID>-summary.json`
4. 리포트에 반드시 포함한다:
   - 전체 요약(총/성공/실패/스킵, 실행 시간)
   - 테스트케이스별 결과(상태, 소요시간, 실패 원인)
   - 실패 케이스는 스크린샷 링크/경로(가능하면 상대경로) 포함
5. (변경사항이 생긴 경우) `git status` 확인 → `git add` → `git commit`
 
## Checklist (선택)
- [ ] 요약에 총/성공/실패/스킵이 일관되게 계산되는가?
- [ ] 실패 케이스에 error 메시지와 스크린샷이 연결되는가?
- [ ] 팀원이 HTML만 봐도 재현/수정 힌트를 얻을 수 있는가?
- [ ] (변경사항이 생긴 경우) 커밋까지 완료했는가?
