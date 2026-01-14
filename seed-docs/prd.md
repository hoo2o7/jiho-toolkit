# PRD (Product Requirements Document) — Litmers Admin v2

> 문서 목적: **Litmers 운영팀(PM/Delivery/재무/리소스/운영/관리자)**이 사용하는 통합 어드민의 제품 요구사항을 정의한다.  
> 기준 리포지토리: `litmers-admin-v2` (Next.js App Router)  
> 근거 소스: `app/(admin)/**` 라우트, `components/feature/**`, `convex/schema.js`, `lib/dummy-data/**`

---

## 1) 배경 / 문제 정의

Litmers는 고객사(Client) 단위로 **(외부 프로그램에서 세일즈 진행/완료)** → 프로젝트(Project) → 계약/청구(Finance) → 리소스 배치(Workforce)까지 이어지는 운영 프로세스를 갖는다.  
기존에는 정보가 분산(스프레드시트/메신저/문서/개별 툴)되어 다음 문제가 발생한다.

- **파편화된 정보**: 프로젝트 진행상태, 청구/미수금, 리소스 투입이 단절
- **상태 가시성 부족**: “지금 어디서 막혔는지(딜/프로젝트/청구/리소스)”를 한 화면에서 파악하기 어려움
- **의사결정 지연**: 결정 기록/요구사항이 구조화되지 않아 회고/리스크 관리가 어려움

---

## 2) 제품 목표(Goals) / 성공 지표(Success Metrics)

### 제품 목표
- **G1. End-to-end 운영 흐름 통합**: Client–Project–Finance–Workforce가 같은 데이터 모델로 연결
- **G2. 실행(Execution) 중심 UI**: 목록/필터/상세/편집/이력/첨부 중심으로 “업무 처리”가 빠름
- **G3. 의사결정 가속**: 대시보드에서 KPI·리스크·다음 액션을 즉시 확인

### 성공 지표(예시)
- **온보딩 리드타임**: Client 생성 → Project 생성까지 평균 기간
- **계약 준비 리드타임**: Project 생성 → Contract 생성까지 평균 기간
- **미수금(Due/Overdue) 감소**: `InvoiceStatus=OVERDUE` 건수/금액
- **리소스 가동률 정확도**: 배정(Assignment) 대비 실제 투입(Tasks/DailyEntry) 편차
- **원가/수익성 신뢰도**: 레벨 비용 기반 계산(Actual salary 미노출)로 인한 리포트 일관성

---

## 3) 사용자 / 페르소나

- **PM/Delivery**: 프로젝트 단계/일정/요구사항/작업(WorkItem, DevTask) 관리, 회의/결정 기록
- **재무/운영(Finance/Ops)**: 계약(Contract) 및 마일스톤(Milestone) 기반 청구/수금 관리, 전자서명
- **리소스 매니저(Workforce Manager)**: 인력(Workforce) 정보, 투입/배치(ResourceAssignment), 태스크/실투입 추적
- **관리자(Admin)**: 기준정보(Settings), 접근권한/시스템 운영(System)

> 참고: **세일즈 단계/리드 관리는 외부 프로그램**에서 수행한다.  
> 단, 사내 역할(Role) 정의에는 `Sales`가 포함될 수 있으며(권한 정책), 이는 프로젝트/재무 등 내부 운영에서 필요한 범위로 제한한다.

---

## 4) 범위(Scope)

### In Scope (현재 라우트 기준)
- **대시보드**: KPI/프로젝트 현황/가동률/최근활동
- **Client**: 고객사 목록/상세, 담당자(Manager) 관리
- **Projects**: 프로젝트 목록/상세, 요구사항, 히스토리/타임라인/리뷰, 작업/개발태스크(WorkItem/DevTasks)
- **Finance**: 계약/마일스톤, 세금계산서(TaxInvoice), 전자서명(DigitalSignature)
- **Workforce**: 캘린더/내 태스크/리소스/직원(people)/투입(assignment) 및 일별 투입 기록
- **Settings**: 스킬스택/계약유형/개발유형 리스트, 월별 근무일, **급여별 레벨(레벨 비용) 설정**
- **System**: 운영/관리 기능(현재 placeholder 포함)

### Out of Scope (초기)
- **Sales/Lead 파이프라인 관리**(리드 생성/단계/히스토리/팔로업 등): 외부 프로그램에서 처리
- 외부 결제/회계 시스템과의 실시간 정산 자동연동(추후 단계)
- 복잡한 권한 체계(조직/부서/프로젝트 권한) — 최소 RBAC부터 시작
- 고객 포털(고객이 직접 로그인하는 화면)

---

## 5) 핵심 사용자 시나리오(User Journeys)

### J1. “세일즈 완료 → 고객사/프로젝트 온보딩”
1. 외부 프로그램에서 세일즈가 완료된다(딜 확정)
2. Admin(또는 운영자)이 Litmers Admin v2에서 Client(고객사)를 생성/정리한다
3. PM이 Project를 생성하고, 요구사항/작업 단위를 구성한다
4. 필요 시 Contract/Milestone을 생성하여 청구 운영으로 연결한다

### J2. “프로젝트 운영”
1. PM이 Project 상세에서 단계(ProjectStage)와 일정(Timeline) 관리
2. 회의/결정(ProjectHistory) 기록 및 AI 요약(선택)
3. WorkItem/DevTaskItem을 통해 작업 단위 관리 및 담당자 지정
4. 리소스 배정(ResourceAssignment)과 실제 투입(Task/DailyEntry)을 비교

### J3. “청구/수금 운영”
1. Contract에 Milestone(금액/예정일) 생성
2. Milestone에 TaxInvoice 연결 및 발행/전송 상태 관리
3. 수금 시 Milestone/Invoice 상태 업데이트(미수/연체 추적)
4. 전자서명(DigitalSignature) 요청 및 상태 추적

---

## 6) 기능 요구사항(Functional Requirements)

### 6.1 대시보드 (`/dashboard`)
- **요구사항**
  - 프로젝트 진행 현황(stage별/지연 여부)
  - 인력 가동률(배정/실투입 기반)
  - 재무 요약(청구/수금/미수)
  - 최근 활동(프로젝트/재무/리소스 이벤트)

### 6.2 Client (`/clients`, `/clients/[clientId]`)
- 고객사 유형(법인/개인)별 필드 관리
- 고객 담당자(Manager) 정보 CRUD
- 고객별 프로젝트/재무(계약/청구) 연결(탐색/집계)

### 6.3 Projects (`/projects`, `/projects/[projectId]`, `/projects/[projectId]/requirements`)
- 프로젝트 생성/목록/필터
- 단계(ProjectStage) 및 타임라인(ProjectTimeline) 관리
- 작업(WorkItem)과 개발 태스크(DevTaskItem) 관리(상태/담당/기간/예상/실적)
- 프로젝트 히스토리(ProjectHistory) 기록 및 요약
- 프로젝트 회고/리뷰(ProjectReview)
- 계약(Contract) 및 리소스 배정(ResourceAssignment) 연결

### 6.4 Finance (`/finance/contracts`, `/finance/invoices`, `/finance/signatures`)
- Contract 생성/상세/첨부(계약서 URL)
- Milestone 생성 및 상태(예정/청구/수금/연체) 관리
- TaxInvoice 발행/전송/수금 상태(초안/발송/수금/취소/연체)
- DigitalSignature 요청/상태 추적

### 6.5 Workforce (`/workforce/people`, `/workforce/resources`, `/workforce/tasks`)
- **직원 관리**
  - 인력 마스터(Workforce) 목록/상세/CRUD
  - 스킬스택/역할/고용형태 관리
  - 직원에 `regionGroup(KOR/SEA)` 및 `levelId`(레벨 참조) 관리
  - **원가 계산은 "레벨 비용(Level Cost)" 기반**으로 수행(실제 연봉/실제 시급 저장·노출 금지)
  - 관련: `docs/admin-spec/erd.md` (workforces 컬렉션 참고)

- **Task System (핵심 업무 관리)**
  - 개인별 Task 생성/관리 (프로젝트 연결 가능)
  - 일별 투입시간 기록 (TaskDailyEntry) - 퇴근 전 필수 입력
  - 8시간 근무 규칙 준수 체크 및 경고
  - Today/All Task 목록 관리
  - 완료 상태 자동 업데이트 및 실적 집계

- **Resource Management**
  - 프로젝트별 리소스 배정(ResourceAssignment)
  - 배정 기간/예상 투입량 설정
  - 배정 상태 추적 (in_progress/completed/delayed)
  - 예상 vs 실제 투입량 비교 분석

### 6.6 Settings (`/settings`)
- 스킬스택/계약유형/개발유형 리스트 관리
- 월별 근무일 관리
- **급여별 레벨(GradeLevel) 관리**
  - regionGroup별 레벨 리스트 CRUD
  - 시간당 레벨비용(hourlyLevelCost) 계산/저장(정책에 따른 산식)

### 6.7 System (`/system`)
- 초기: 시스템 상태/운영 메뉴 placeholder
- 추후: 권한/감사로그/환경설정/통합(SSO, 웹훅 등)
 - **권한 정책(RBAC)**
   - 역할(Role): `C-level`, `GA`, `Manager`, `PM`, `Sales`, `Designer`, `Developer`, `QA`, `ADMIN`, `TECH_LEAD`
   - 역할은 **복수 선택 가능**하며 권한은 **합집합(Union)** 으로 적용
   - Settings 접근 권한: **C-level, GA만**
   - Workforce > 직원(people)
     - **직원 추가 버튼**: GA만 노출
     - **원가/레벨 관련 컬럼**(예: level, hourlyLevelCost)은 C-level/GA만 노출(그 외 역할은 비노출/마스킹)
   - Projects > 수익성(Profitability) 관련 노출: C-level/GA/Manager/PM/Sales만(권한 정책에 따름)

---

## 7) 비기능 요구사항(Non-functional Requirements)

- **보안/접근제어**
  - 최소 RBAC(역할 기반) 및 민감정보 마스킹(원가 등)
  - 감사 로그(누가/언제/무엇을 변경했는지)
- **데이터 무결성**
  - 관계 삭제 정책(onDelete Cascade/SetNull) 명확화
  - 상태 전이 규칙(예: MilestoneStatus, InvoiceStatus) 검증
- **성능**
  - 목록 페이지: pagination/filter 서버 처리(추후), 초기에는 최소 1초 내 응답 목표
- **사용성**
  - 표 기반 작업 흐름, 단축 액션(상태 변경/편집/삭제), 필터 유지
- **국제화/표준**
  - 통화/날짜 포맷(KRW/ko-KR), 타임존 정책(UTC 저장 + 표시 변환)
- **확장성**
  - 모듈별 API 분리, 도메인 타입(`types/database.ts`) 중심의 계약 유지

---

## 8) 데이터/통합 가정(Assumptions)

- DB는 **Convex**(`convex/schema.js`)가 Source of Truth
- 현재 UI는 더미데이터 기반(`lib/dummy-data/**`)이므로,
  - 향후 Convex queries/mutations로 CRUD를 이전
  - 화면 컴포넌트는 최대한 유지하며 데이터 소스만 Convex로 교체
- 권한/인증은 **WorkOS AuthKit** 사용

---

## 9) 리스크 / 오픈 이슈

- **모델 불일치**: 더미데이터의 enum/필드가 Convex 스키마와 완전히 일치하지 않는 부분 존재(예: ProjectStage).
  → 마이그레이션 시 “표준 enum”을 `docs/admin-spec/erd.md` 기준으로 통일 필요.

---

## 10) 관련 문서

- **Physical ERD**: `docs/admin-spec/erd.md`
- **Conceptual Model**: `docs/admin-spec/conceptual-model.md`
- **Information Architecture (IA)**: `docs/admin-spec/information-architecture.md`
- **Logical Architecture**: `docs/admin-spec/logical-architecture.md`


