# Information Architecture (IA) — Litmers Admin v2

> 문서 목적: 사용자가 “어디서 무엇을 하는지”를 라우트/네비게이션 기준으로 정의한다.  
> 기준: `components/feature/layout/admin-sidebar.tsx` 메뉴, `app/(admin)/**` 라우트 구조

---

## 1) 글로벌 내비게이션(사이드바)

사이드바 메뉴(요약):

- **대시보드**: `/dashboard`
- **Client**
  - 고객사 관리: `/clients`
- **프로젝트**: `/projects`
- **Workforce**
  - 캘린더: `/workforce`
  - 내 태스크: `/workforce/tasks`
  - 리소스 관리: `/workforce/resources`
  - 직원 관리: `/workforce/people`
- **Finance**
  - 대시보드: `/finance`
  - 계약: `/finance/contracts`
  - 세금계산서: `/finance/invoices`
  - 전자서명: `/finance/signatures`
- **설정**: `/settings`
- **시스템**: `/system`

> 참고: 세일즈 단계/파이프라인 관리는 외부 프로그램에서 진행한다.  
> Litmers Admin v2의 운영 플로우는 **(세일즈 완료) → 고객사(Client) 생성 → 프로젝트(Project) 생성 → 재무/리소스 운영**을 중심으로 한다.

---

## 2) 사이트맵(Sitemap) — 라우트 트리

```
/(admin)
  /dashboard
  /clients
    /[clientId]
  /projects
    /[projectId]
      /requirements
  /proposals
    /[proposalId]
  /finance
    /contracts
      /[projectId]
    /invoices
      /[invoiceId]
    /signatures
      /[signatureId]
  /workforce
    /people
    /resources
    /tasks
      /[taskId]
  /settings
  /system
```

---

## 3) 화면(정보) 단위 정의

각 화면은 “목적 → 사용자가 하는 핵심 액션 → 주요 정보 구성”을 명확히 한다.

### 3.1 대시보드 `/dashboard`
- **목적**: 전체 운영 현황을 1분 내 파악(프로젝트/리소스/재무)
- **핵심 컴포넌트(현재 구현)**:
  - KPI Cards, Project Status, Workforce Utilization, Today Tasks, Finance Overview, Recent Activities
- **핵심 액션**:
  - 위험/지연 영역 클릭 → 해당 모듈 목록으로 이동(딥링크)

### 3.2 Client

#### 고객사 목록 `/clients`
- **목적**: 고객사 마스터 데이터 관리 + 고객 단위 탐색 시작점
- **핵심 액션**:
  - 법인/개인 유형별 필드 관리
  - 담당자(Manager) 관리 진입
  - 고객 기준 프로젝트/재무(계약/청구) 탐색

#### 고객사 상세 `/clients/[clientId]`
- **목적**: 고객사 운영 컨텍스트 집약(담당자/프로젝트/계약/청구)

### 3.3 Projects `/projects`, `/projects/[projectId]`, `/projects/[projectId]/requirements`
- **목적**: 고객사 생성 이후 프로젝트(Delivery) 운영
- **핵심 액션**:
  - 프로젝트 생성/상세
  - 단계/타임라인/히스토리 관리
  - 요구사항, 작업(WorkItem), 개발태스크(DevTask) 관리
  - 리소스 배정/실투입 추적(Workforce와 연결)

### 3.4 Proposals `/proposals`, `/proposals/[proposalId]`
- **목적**: 제안서 문서/산출물 관리(현재 라우트 존재, 상세는 구현 상태 확인 필요)

### 3.5 Finance

#### Finance 대시보드 `/finance`
- **목적**: 청구/수금/미수 현황 요약

#### 계약 `/finance/contracts` (+ `/finance/contracts/[projectId]`)
- **목적**: 프로젝트 기준 계약/마일스톤 운영
- **핵심 액션**:
  - 계약 생성/수정, 계약서 첨부/링크
  - 마일스톤 생성 및 상태(청구/수금/연체) 관리
  - 전자서명 요청(연결)

#### 세금계산서 `/finance/invoices` (+ `/finance/invoices/[invoiceId]`)
- **목적**: 발행/전송/수금 상태 관리

#### 전자서명 `/finance/signatures` (+ `/finance/signatures/[signatureId]`)
- **목적**: 서명 요청/진행/완료 추적

### 3.6 Workforce

#### 캘린더 `/workforce`
- **목적**: 통합 일정 시각화 및 개인 투입 관리
- **핵심 액션**:
  - 오늘 표시 및 날짜별 task 개수 표시
  - 내가 배정된 모든 Task(시작~종료 기간) 표시
  - 참여 프로젝트의 주요 일정(시작/완료 예정) 표시
  - 날짜 클릭 시 Task 관리 화면으로 이동

#### 내 태스크 `/workforce/tasks` (+ `/workforce/tasks/[taskId]`)
- **목적**: 개인 업무 관리 및 일별 투입 기록 (Task System 핵심)
- **핵심 액션**:
  - Task 생성/수정/삭제
  - 일별 투입시간 기록 (퇴근 전 필수)
  - 8시간 규칙 준수 체크 및 경고
  - Today/All Task 목록 필터링
  - 완료 시 자동 상태 업데이트

#### 리소스 `/workforce/resources`
- **목적**: 프로젝트별 리소스 배정 및 상태 추적
- **핵심 액션**:
  - 리소스 배정 기간(start~end) 및 예상 투입량 설정
  - 배정 상태 관리(in_progress/completed/delayed)
  - 예상 vs 실제 투입량 비교 및 경고
  - 리소스별 Task 목록 조회

#### 직원 `/workforce/people`
- **목적**: 인력 마스터(역할/스킬/고용형태/원가) CRUD
- **관련 문서**: `docs/admin-spec/erd.md` (workforces 컬렉션)

### 3.7 Settings `/settings`
- **목적**: 기준정보(스킬/계약유형/개발유형/근무일) 관리

### 3.8 System `/system`
- **목적**: 운영/관리자 영역(권한/감사로그/환경설정 등) — 단계적 확장

---

## 4) IA 원칙(권장)

- **목록 → 상세 → 편집** 패턴을 모든 모듈에 일관 적용
- 필터는 **URL 쿼리**로 유지(새로고침/공유 가능)
- “기록(History) + 다음 행동(Action)”을 함께 배치해 실행력을 강화
- Client를 **최상위 탐색 축**으로 유지하되, Project/Finance/Workforce가 서로 딥링크 가능해야 함

---

## 5) 관련 문서

- **Physical ERD**: `docs/admin-spec/erd.md`
- **PRD**: `docs/admin-spec/prd.md`
- **Conceptual Model**: `docs/admin-spec/conceptual-model.md`
- **Logical Architecture**: `docs/admin-spec/logical-architecture.md`


