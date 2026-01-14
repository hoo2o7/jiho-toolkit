# Change Log: 2025-12-17 — 급여 관련 정책 & 메뉴별 권한 정책

## 0) 입력 원문

### 요약

1. **급여 정책 변경**: 실제 연봉(Actual Cost)을 시스템에서 완전히 제거하고, "레벨 비용(Level Cost)" 기반으로 모든 원가/수익성 계산을 전환
2. **권한 정책 신규 정의**: 8개 역할(C-level, GA, Manager, PM, Sales, Designer, Developer, QA)에 대한 메뉴/버튼/데이터 접근 권한 정책 수립

### 원문

- 급여 관련 정책: `docs/changes/급여 관련 정책.md`
- 메뉴별 권한 정책: `docs/changes/메뉴별 권한 정책.md`

---

## 1) 변경 유형 분석

### Added

| ID      | 변경 요약                                 | 근거/원문 인용                                                                                               |
| ------- | ----------------------------------------- | ------------------------------------------------------------------------------------------------------------ |
| SAL-03  | 레벨(GradeLevel) 엔티티 신규 추가         | "레벨(GradeLevel)에는: regionGroup, levelName, avgAnnualSalary, workHoursPerYear, hourlyLevelCost, isActive" |
| SAL-04  | Settings에 "급여별 레벨" 설정 탭 추가     | "Settings > 급여별 레벨(레벨 비용) 설정"                                                                     |
| SAL-05  | 직원에 regionGroup(KOR/SEA) 필드 추가     | "직원(Employee)은 regionGroup (KOR/SEA)"                                                                     |
| SAL-06  | 직원에 levelId 필드 추가 (레벨 참조)      | "levelId (해당 regionGroup의 레벨 중 선택)"                                                                  |
| SAL-07  | 직원 테이블에 "시간당 레벨비용" 컬럼 추가 | "Hourly Level Cost (시간당 레벨비용)"                                                                        |
| PERM-01 | 역할(Role) 체계 정의: 8개 역할            | "역할(Role): C-level, GA, Manager, PM, Sales, Designer, Developer, QA"                                       |
| PERM-03 | 메뉴별 접근 권한 정책                     | 각 메뉴별 접근 권한 테이블                                                                                   |
| PERM-02 | 역할 중복 선택 + 합집합 권한 적용         | "역할은 중복 선택 가능, 합집합으로 적용"                                                                     |
| PERM-04 | 버튼별 권한 정책 (작업 추가, CRUD 등)     | "작업 추가 버튼", "CRUD 권한" 테이블                                                                         |
| PERM-05 | 데이터 노출 권한 정책 (컬럼별)            | "employee-table.tsx 컬럼 노출" 테이블                                                                        |
| PERM-07 | Finance 상태 변경 권한 정책               | "GA: 요청됨 → GA검토 → 발행" 등                                                                              |
| PERM-08 | Settings 접근 권한: C-level, GA만         | "C-level, GA만 접근 가능"                                                                                    |

### Removed

| ID     | 변경 요약                                          | 근거/원문 인용                                          |
| ------ | -------------------------------------------------- | ------------------------------------------------------- |
| SAL-01 | 직원의 실제 연봉(annualSalary) 입력/저장/노출 제거 | "실제 연봉(실제비용)은 시스템 어디에도 노출하지 않는다" |
| SAL-02 | 직원의 실제 시간당비용(costPerHour) 컬럼 제거      | "연봉, 시간당비용(실제)을 제거"                         |

### Modified

| ID      | 변경 요약                                      | As-is → To-be                                                       |
| ------- | ---------------------------------------------- | ------------------------------------------------------------------- |
| SAL-08  | 비용 계산 방식 변경                            | 실제비용 기반 → 레벨비용 기반                                       |
| PERM-06 | 직원 테이블 컬럼 노출 정책                     | 전체 노출 → 역할별 차등 노출 (C-level/GA만 레벨/비용 노출)          |
| PERM-09 | 프로젝트 작업관리 화면 "수익성" 노출 권한 수정 | C-level: X → O (단, 노출 대상은 C-level/GA/Manager/PM/Sales만 유지) |

### Impact-shift

- 없음

---

## 2) 영향 범위 분석 (UI Only)

| 영역             | 필요 여부 | 변경 내용(요약)                     | 영향 모듈 | 산출물/비고                |
| ---------------- | :-------: | ----------------------------------- | --------- | -------------------------- |
| 화면(Route)      |    ✅     | Settings에 "급여별 레벨" 탭 추가    | M02       | 탭 + 컴포넌트              |
| 컴포넌트         |    ✅     | 레벨 관리 컴포넌트 신규 생성        | M02       | `grade-level-list.tsx`     |
| 컴포넌트         |    ✅     | 직원 테이블 컬럼 변경               | M03       | `employee-table.tsx`       |
| 컴포넌트         |    ✅     | 직원 폼 필드 변경                   | M03       | `employee-form-dialog.tsx` |
| 상태/타입        |    ✅     | Employee, GradeLevel 타입 정의      | M03       | 타입 수정/추가             |
| 상태/훅          |    ✅     | 중앙화된 권한 시스템 구축           | M01       | `use-permissions.ts`       |
| 상태/Interaction |    ✅     | 권한 기반 조건부 렌더링             | M04, M05  | 훅 활용                    |
| 스타일/UX        |    ❌     | -                                   | -         | -                          |
| 더미데이터       |    ✅     | workforce.ts, settings.ts 수정      | M02, M03  | 데이터 구조 변경           |
| Conceptual Model |    ✅     | GradeLevel 엔티티, 권한 시스템 추가 | M06       | 문서 업데이트              |

---

## 3) 영향 모듈 리스트 & 우선순위

| 모듈 ID | 설명                                                     | 우선순위  | 상태                 |
| ------- | -------------------------------------------------------- | --------- | -------------------- |
| **M01** | 권한 시스템 중앙화 (use-permissions hook, UserRole 확장) | 🔴 High   | ✅ Done              |
| **M02** | Settings > 급여별 레벨 탭 (GradeLevel CRUD)              | 🔴 High   | ✅ Done              |
| **M03** | 직원 테이블/폼 수정 (레벨 기반으로 전환)                 | 🔴 High   | ✅ Done              |
| **M04** | 직원 테이블 컬럼 노출 권한 적용                          | 🟡 Medium | ✅ Done (M03에 포함) |
| **M05** | 직원 추가 버튼 GA 권한 적용                              | 🟡 Medium | ✅ Done              |
| **M06** | conceptual-model.md 업데이트                             | 🟡 Medium | ✅ Done              |

### 의존성(요약)

| From | To  | 이유                                    |
| ---- | --- | --------------------------------------- |
| M01  | M03 | 권한 훅이 있어야 컬럼 조건부 노출 가능  |
| M01  | M04 | 권한 훅 필요                            |
| M01  | M05 | 권한 훅 필요                            |
| M02  | M03 | 레벨 데이터가 있어야 직원에서 선택 가능 |

### 실행 순서

1. **M01** (권한 시스템) — 선행 필수
2. **M02** (레벨 설정) — M01과 병렬 가능하지만 M03 전에 완료 필요
3. **M03** (직원 테이블/폼) — M01, M02 완료 후
4. **M04, M05** (권한 적용) — M01, M03 완료 후
5. **M06** (문서 업데이트) — 개발 완료 후

---

## 4) 실행 계획 (Step 8 적용)

### Phase 1: 기반 시스템 (M01, M02)

- **M01**: 권한 시스템 중앙화
  - Step 8-1: UserRole 확장, usePermissions 훅 스펙 정의
  - Step 8-4: `lib/dummy-data/user.ts`, `hooks/use-permissions.ts` 구현
  - Step 8-5: 권한 체크 동작 확인

- **M02**: Settings > 급여별 레벨 탭
  - Step 8-1: GradeLevel CRUD 스펙 정의
  - Step 8-3: GradeLevel 타입/더미데이터 정의
  - Step 8-4: `grade-level-list.tsx` 구현, settings/page.tsx에 탭 추가
  - Step 8-5: 레벨 CRUD 동작 확인

### Phase 2: 직원 관리 변경 (M03)

- **M03**: 직원 테이블/폼 레벨 기반 전환
  - Step 8-1: Employee 타입 변경 스펙 정의 (연봉 제거, 레벨 추가)
  - Step 8-3: Employee 타입/더미데이터 수정
  - Step 8-4: `employee-table.tsx`, `employee-form-dialog.tsx` 수정
  - Step 8-5: 직원 추가/수정/목록 동작 확인

### Phase 3: 권한 적용 (M04, M05)

- **M04**: 직원 테이블 컬럼 노출 권한
  - Step 8-4: employee-table.tsx에 usePermissions 적용
  - Step 8-5: 역할별 컬럼 노출 확인

- **M05**: 직원 추가 버튼 GA 권한
  - Step 8-4: people/page.tsx에 usePermissions 적용
  - Step 8-5: GA 외 역할에서 버튼 숨김 확인

### Phase 4: 문서화 (M06)

- **M06**: conceptual-model.md 업데이트
  - GradeLevel 엔티티 추가
  - Workforce 엔티티 속성 변경 (regionGroup, levelId)
  - 권한 시스템 섹션 추가

---

## 5) 리스크 / TBD

| ID       | 내용                                                       | 상태           | 비고                         |
| -------- | ---------------------------------------------------------- | -------------- | ---------------------------- |
| TBD-001  | Finance 관련 권한 적용 범위                                | 이번 스코프 외 | 추후 별도 변경 요청으로 처리 |
| TBD-002  | 사이드바 메뉴 접근 제한 적용                               | 이번 스코프 외 | M01 완료 후 점진적 적용 가능 |
| RISK-001 | 기존 더미데이터 annualSalary/costPerHour 제거 시 타입 에러 | 저위험         | M03에서 일괄 처리            |

---

## 6) 완료 정의 (Definition of Done)

- [x] M01: `usePermissions` 훅이 역할별 권한을 올바르게 반환
- [x] M02: Settings에서 GradeLevel CRUD 동작 (KOR/SEA 구분, 시간당 레벨비용 자동 계산)
- [x] M03: 직원 테이블에서 연봉/시간당비용 대신 레벨/시간당레벨비용 표시
- [x] M03: 직원 폼에서 regionGroup 선택 → 해당 레벨 리스트 표시 → 선택
- [x] M04: C-level/GA만 레벨/시간당레벨비용 컬럼 노출, 나머지는 이름/역할/스킬만
- [x] M05: GA만 "직원 추가" 버튼 노출
- [x] M06: conceptual-model.md에 GradeLevel, 권한 시스템 반영
- [x] 린트/타입 에러 없음
- [ ] 기존 페이지 회귀 테스트 (직원 목록, Settings)

---

## 7) 정책 충돌 점검 결과 (Step 0-5)

### 점검 대상 문서

- `docs/admin-spec/conceptual-model.md`: ✅ 업데이트 완료
- 라우팅/폴더 구조: ✅ 충돌 없음 (기존 구조 유지)
- 권한/validation 정책: ✅ `usePermissions` 훅으로 중앙화

### TBD 상태

| ID      | 내용                    | 상태                                         |
| ------- | ----------------------- | -------------------------------------------- |
| TBD-001 | Finance 관련 권한 적용  | 이번 스코프 외 (추후 처리)                   |
| TBD-002 | 사이드바 메뉴 접근 제한 | 이번 스코프 외 (M01 기반으로 추후 적용 가능) |

### 결론

- **충돌 없음**
- **BLOCKED 항목 없음**
