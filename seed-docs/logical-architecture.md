# Logical Architecture — Litmers Admin v2

> 문서 목적: 시스템을 “논리적 구성 요소(레이어/모듈/데이터 흐름)”로 설명해 개발/확장/운영 기준을 만든다.  
> 기준: Next.js App Router 구조(`app/**`), UI 컴포넌트 구조(`components/**`), DB 스키마(`convex/schema.js`)

---

## 1) 시스템 개요

Litmers Admin v2는 **단일 웹 어드민**으로, Client–Project–Finance–Workforce–Settings를 하나의 UI/데이터 모델로 묶는다.

- **Frontend/Fullstack 프레임워크**: Next.js(앱 라우터), React
- **UI**: shadcn/ui + Radix + Tailwind 기반 컴포넌트
- **DB/Backend**: Convex (실시간 데이터베이스 + 서버리스 함수)
- **인증/권한**: WorkOS AuthKit
- **현재 데이터 소스 상태**: 일부 페이지는 `lib/dummy-data/**` 기반(프로토타입), 장기적으로 Convex queries/mutations로 대체

---

## 2) 레이어드 아키텍처(논리 레이어)

### 2.1 Presentation Layer (UI)
- **페이지 라우트**: `app/(admin)/**/page.tsx`
- **레이아웃/내비게이션**: `app/(admin)/layout.tsx`, `components/feature/layout/**`
- **기능 단위 컴포넌트**: `components/feature/**` (도메인별)
- **복합 컴포넌트**: `components/composite/**` (테이블, 다이얼로그, 헤더 등)
- **UI primitives**: `components/ui/**` (Button, Table, Dialog 등)

### 2.2 Application Layer (Use-cases / ViewModel)
권장 구조(현재는 페이지 내부 state가 많음):
- **리스트/필터/정렬/페이지네이션 로직**
- **CRUD 오케스트레이션**(모달 submit → mutation → 목록 갱신)
- **권한/가드**(추후)

### 2.3 Domain Layer (Business Rules)
도메인 규칙은 Convex 스키마(`convex/schema.js`) 및 `docs/admin-spec/erd.md`에 정렬되어야 한다.
- 상태 전이 규칙(예: InvoiceStatus, MilestoneStatus)
- 비용/원가 계산 규칙(GradeLevel.hourlyLevelCost 기반)

### 2.4 Data Access Layer (Persistence / Integrations)
권장 구성:
- Convex 함수(`convex/*.js`) — query, mutation, action
- 클라이언트에서 Convex React 훅(`useQuery`, `useMutation`) 사용
- 서버에서 필요 시 Convex HTTP Client 사용
- 외부 통합(서명/메일/스토리지) 어댑터 — Convex actions로 구현

---

## 3) 모듈 구조(도메인별 논리 컴포넌트)

### 3.1 Client
- **UI**: `/clients`, `/clients/[clientId]`
- **핵심 데이터 모델**: Client, Manager
- **주요 흐름**
  - (외부 세일즈 완료 후) Client 생성 → 프로젝트/청구의 탐색 허브

### 3.2 Projects/Delivery
- **UI**: `/projects`, `/projects/[projectId]`, `/projects/[projectId]/requirements`
- **핵심 데이터 모델**: Project, ProjectHistory, WorkItem, DevTasks/DevTaskItem, Specification/Item, Timeline, Review, Requirement
- **주요 흐름**
  - Project stage/timeline 관리 → WorkItem/DevTask 운영 → 리소스 배정/실투입 연결

### 3.3 Finance
- **UI**: `/finance/contracts`, `/finance/invoices`, `/finance/signatures`
- **핵심 데이터 모델**: Contract, Milestone, TaxInvoice, DigitalSignature
- **주요 흐름**
  - 계약 생성 → 마일스톤(청구 단위) → 세금계산서 발행/상태 → 수금 처리 → 전자서명 추적

### 3.4 Workforce
- **UI**: `/workforce/people`, `/workforce/resources`, `/workforce/tasks`
- **핵심 데이터 모델**: Workforce, ResourceAssignment, Task, TaskDailyEntry, MonthlyWorkday, Settings
- **주요 흐름**
  - 인력 마스터(원가 포함) → 배정(assignment) → 태스크/일별 투입 기록 → 가동률/원가 분석
- **관련 문서**: `docs/admin-spec/erd.md` (workforces, tasks, resourceAssignments 컬렉션)

### 3.5 Settings/System
- **Settings 모델**: Settings, MonthlyWorkday
- **System**: 권한/감사로그/운영(추후)

---

## 4) 데이터 흐름(Data Flow) — 현재 vs 목표

### 4.1 현재(프로토타입)
- 페이지 컴포넌트가 `lib/dummy-data/**`를 읽고 React state로 CRUD를 흉내냄
- 영속 저장/동시성/권한/감사로그 부재

### 4.2 목표(프로덕션)

권장 흐름:
1. UI(페이지/컴포넌트) → 2. Convex React 훅(useQuery/useMutation) → 3. Convex 함수(query/mutation/action) → 4. Convex Storage

추가적으로:
- 파일(계약서/녹취/전사/첨부)은 오브젝트 스토리지(Convex File Storage 또는 S3 등) + 메타데이터만 DB 저장
- AI 요약은 Convex action + scheduled function으로 생성 후 summaries 컬렉션에 저장
- Task System: workforces → tasks → taskDailyEntries (일별 투입 기록)
- Resource Management: workforces → resourceAssignments → projects (배정 관리)
- workItems/devTaskItems은 직접 workforces와 연결 (assigneeId)

---

## 5) 논리적 인터페이스(API) 설계 가이드(권장)

> 현재 레포에 API 라우트가 필수로 구현되어 있진 않지만, 확장을 위해 규격을 제안한다.

### 리소스 중심 REST 예시
- `/api/clients`, `/api/clients/:id`, `/api/clients/:id/managers`
- `/api/projects`, `/api/projects/:id`, `/api/projects/:id/work-items`, `/api/projects/:id/dev-tasks`
- `/api/contracts`, `/api/contracts/:id/milestones`
- `/api/invoices`, `/api/signatures`
- `/api/workforce`, `/api/workforce/:id/tasks`, `/api/workforce/:id/assignments`
- `/api/tasks`, `/api/tasks/:id`, `/api/tasks/:id/entries` (Task System)
- `/api/resource-assignments` (프로젝트별 리소스 배정 관리)
- `/api/settings`, `/api/monthly-workdays`

응답 표준:
- `ApiResponse<T>` / `PaginatedResponse<T>` 타입(`types/database.ts`) 기준

---

## 6) 크로스컷(공통) 고려사항

- **권한/인증**
  - 역할 기반 최소 권한 + 모듈별 권한 정책
- **감사로그**
  - createdBy/updatedBy 필드 유지 + 변경 이벤트 로그 테이블(추후)
- **타임존/날짜**
  - DB는 UTC 저장, UI는 `ko-KR`로 표시
- **상태 전이 검증**
  - 서버에서 enum 전이 규칙 강제(클라이언트 단순 변경 금지)
- **관측성**
  - 주요 CRUD/상태변경 이벤트 로깅
  - 대시보드 KPI 계산 배치/캐시 전략(추후)

---

## 7) 관련 문서

- **Physical ERD**: `docs/admin-spec/erd.md`
- **PRD**: `docs/admin-spec/prd.md`
- **Conceptual Model**: `docs/admin-spec/conceptual-model.md`
- **IA**: `docs/admin-spec/information-architecture.md`


