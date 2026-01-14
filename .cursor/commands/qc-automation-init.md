# qc-automation-init
 
## Overview
QA 자동화 작업을 시작하기 위한 **폴더 구조 + 설정 + 자격증명 템플릿**을 준비합니다. (자격증명 파일은 git에 올리지 않습니다)
 
## Steps
1. 아래 폴더 구조를 생성한다:
   - `qc-automation/config/`
   - `qc-automation/testcases/`
   - `qc-automation/scripts/generated/`
   - `qc-automation/executions/`
   - `qc-automation/reports/`
2. `qc-automation/config/settings.json` 생성(프로젝트 설정):
 
```json
{
  "projectName": "My Project",
  "baseUrl": "http://localhost:3000",
  "defaultTimeout": 30000,
  "screenshotOnFailure": true,
  "browser": "chromium",
  "headless": true
}
```
 
3. `qc-automation/config/credentials.env.example` 생성(템플릿):
 
```bash
# 예시 템플릿 (실제 값은 credentials.env에만 저장)
BASE_URL=http://localhost:3000
EMAIL=
PASSWORD=
```
 
4. `qc-automation/config/credentials.env`는 **로컬에서만** 생성하고, `.gitignore`에 아래를 추가한다:
 
```gitignore
qc-automation/config/credentials.env
```
 
5. (변경사항이 생긴 경우) `git status` 확인 → `git add` → `git commit`
 
## Checklist (선택)
- [ ] `settings.json`의 `baseUrl`, `headless`, `defaultTimeout`이 현재 실행 환경과 일치하는가?
- [ ] `credentials.env`가 `.gitignore`에 포함되어 있는가?
- [ ] (변경사항이 생긴 경우) 커밋까지 완료했는가?
