# Conceptual Model — Litmers Admin v2

> 문서 목적: 제품의 **핵심 도메인 엔티티/관계/상태 모델**을 팀 공통 언어로 정리한다.  
> Source of Truth: `convex/schema.js` (DB 스키마), `docs/admin-spec/erd.md` (Physical ERD)  
> 보조 근거: `lib/dummy-data/**`(UI가 현재 다루는 필드), `types/database.ts`(타입 계약)

---

## 1) 도메인 경계(Bounded Context)

- **Client/CRM**: 고객사/담당자(Manager)
- **Delivery/Project**: 프로젝트/이력/스펙/작업/타임라인/리뷰/요구사항(프로젝트)
- **Finance**: 계약/마일스톤/세금계산서/전자서명
- **Workforce**: 인력(Workforce)/배정(Assignment)/태스크/일별투입
- **Settings**: 스킬/계약유형/개발유형/근무일
- **AI Assist**: 요약(프로젝트 히스토리/문서 등 텍스트에 종속)

---

## 2) 핵심 엔티티 요약(Top Entities)

### Customer/CRM (고객 관리)

**설명:** 고객사 관리를 위한 핵심 엔티티

**추가된 기능:** Client Management

#### Client (고객사)

**설명:** 회사와 거래하는 고객사 정보 (법인/개인 구분)

**주요 속성:**
- 고객사명 (필수, 문자열): 고객사 표시명
- 고객사 유형 (필수, 열거형): CORPORATE/INDIVIDUAL 중 하나
- 법인명 (선택, 문자열): 법인 고객사명 (법인인 경우)
- 대표자명 (선택, 문자열): 법인 대표자 (법인인 경우)
- 사업자등록번호 (선택, 문자열): 법인 등록번호 (법인인 경우)
- 회사 주소 (선택, 문자열): 법인 주소 (법인인 경우)
- 등기파일 (선택, 문자열): 사업자등록증 파일 경로 (법인인 경우)
- 개인명 (선택, 문자열): 개인 고객명 (개인인 경우)
- 주민등록증 (선택, 문자열): 신분증 파일 경로 (개인인 경우)
- 생성일/수정일 (필수, 날짜시간): 자동 생성/수정

**관계:**
- 1 Client → N Manager (담당자 정보)
- 1 Client → N Project (진행 프로젝트)
- 1 Client → N TaxInvoice (청구서 발행)

**비즈니스 규칙:**
- 법인/개인에 따라 필수 필드 다름
- 사업자등록번호 중복 불가
- 고객사명으로 검색 가능

**사용 화면:**
- /clients: 고객사 목록
- /clients/new: 고객사 등록
- /clients/[clientId]: 고객사 상세/수정

#### Manager (담당자)

**설명:** 고객사 담당자 연락처 및 역할 정보

**주요 속성:**
- 고객사 ID (필수, 참조): 소속 Client ID
- 이름 (필수, 문자열): 담당자 이름
- 직위 (선택, 문자열): 담당자 직위
- 전화번호 (선택, 문자열): 연락처
- 이메일 (선택, 문자열): 이메일 주소
- 생성일/수정일 (필수, 날짜시간): 자동 생성/수정

**관계:**
- N Managers → 1 Client (고객사 담당자)

**비즈니스 규칙:**
- 고객사당 여러 담당자 가능
- 이메일 형식 검증
- 연락처 중복 가능 (같은 회사 다른 담당자)

**사용 화면:**
- /clients/[clientId]: 담당자 목록/관리

### Project/Delivery (프로젝트 관리)

**설명:** 고객 프로젝트의 전체 라이프사이클 관리

**추가된 기능:** Project Management, Delivery Management

#### Project (프로젝트)

**설명:** 고객사 프로젝트의 기본 정보 및 진행 상태

**주요 속성:**
- 고객사 ID (필수, 참조): 관련 Client ID
- 프로젝트명 (필수, 문자열): 프로젝트 표시명
- 현재 단계 (필수, 열거형): PENDING/PLANNING/DESIGN/DEVELOPMENT/INTERNAL_QA/EXTERNAL_QA/MAINTENANCE/COMPLETED/PAUSED/CANCELLED
- PM ID (필수, 문자열 배열): 프로젝트 매니저 ID 리스트
- 디자이너 ID (선택, 문자열 배열): 디자이너 ID 리스트
- 개발자 ID (선택, 문자열 배열): 개발자 ID 리스트
- QA ID (선택, 문자열 배열): QA 담당자 ID 리스트
- **프로젝트 전체 일정 (Project Schedule):**
  - 예상 시작일 (선택, 날짜): 프로젝트 전체 예상 시작일
  - 예상 종료일 (선택, 날짜): 프로젝트 전체 예상 종료일
  - 실제 시작일 (선택, 날짜): 프로젝트 전체 실제 시작일
  - 실제 종료일 (선택, 날짜): 프로젝트 전체 실제 종료일
- **단계별 일정 (Phase Schedules):**
  - Planning (기획 단계):
    - 예상 시작일 (선택, 날짜)
    - 예상 종료일 (선택, 날짜)
    - 실제 시작일 (선택, 날짜)
    - 실제 종료일 (선택, 날짜)
  - Design (디자인 단계):
    - 예상 시작일 (선택, 날짜)
    - 예상 종료일 (선택, 날짜)
    - 실제 시작일 (선택, 날짜)
    - 실제 종료일 (선택, 날짜)
  - Development (개발 단계):
    - 예상 시작일 (선택, 날짜)
    - 예상 종료일 (선택, 날짜)
    - 실제 시작일 (선택, 날짜)
    - 실제 종료일 (선택, 날짜)
  - Internal QA (내부 QA 단계):
    - 예상 시작일 (선택, 날짜)
    - 예상 종료일 (선택, 날짜)
    - 실제 시작일 (선택, 날짜)
    - 실제 종료일 (선택, 날짜)
  - External QA (외부 QA 단계):
    - 예상 시작일 (선택, 날짜)
    - 예상 종료일 (선택, 날짜)
    - 실제 시작일 (선택, 날짜)
    - 실제 종료일 (선택, 날짜)
  - Maintenance (유지보수 단계):
    - 예상 시작일 (선택, 날짜)
    - 예상 종료일 (선택, 날짜)
    - 실제 시작일 (선택, 날짜)
    - 실제 종료일 (선택, 날짜)
- 계약 유형 (선택, 설정값): **Settings.계약유형 목록**을 참조 (정적 enum로 고정하지 않음)
- 개발 유형 (선택, 설정값): **Settings.개발유형 목록**을 참조 (정적 enum로 고정하지 않음)
- 총 가격 (선택, 숫자): 프로젝트 총 금액
- 메모 (선택, 문자열): 프로젝트 관련 메모
- 생성자 ID (필수, 문자열): 프로젝트 생성자
- 생성일/수정일 (필수, 날짜시간): 자동 생성/수정

**일정 관리 UI:**
- 프로젝트 생성/수정 모달: 프로젝트 전체 및 단계별 **예상** 시작일/종료일 입력
- 프로젝트 상태 변경 모달: 프로젝트 전체 및 단계별 **실제** 시작일/종료일 입력
- 모든 일정 필드는 선택 입력 (필수 아님)

**관계:**
- N Projects → 1 Client (고객사 프로젝트)
- 1 Project → N ProjectHistory (프로젝트 이벤트)
- 1 Project → 0..1 Specification (견적 정보)
- 1 Project → N WorkItem (작업 단위)
- 1 Project → 0..1 DevTasks (개발 태스크 그룹)
- 1 Project → N ProjectTimeline (단계별 일정)
- 1 Project → 0..1 ProjectReview (완료 후 회고)
- 1 Project → N Requirement (프로젝트 요구사항)
- 1 Project → 0..1 Contract (계약 정보)
- 1 Project → N ResourceAssignment (리소스 배정)

**비즈니스 규칙:**
- 단계 변경 시 타임라인 자동 업데이트
- 완료 시 ProjectReview 필수 작성
- 계약 연결 후 가격 변경 불가

**사용 화면:**
- /projects: 프로젝트 목록
- /projects/new: 프로젝트 생성
- /projects/[projectId]: 프로젝트 상세/관리

#### ProjectHistory (프로젝트 히스토리)

**설명:** 프로젝트 진행 과정의 이벤트 기록

**주요 속성:**
- 프로젝트 ID (필수, 참조): 관련 Project ID
- 단계 (필수, 열거형): 기록 시점 단계
- 이벤트 유형 (필수, 열거형): CALL/EMAIL/NOTE/FILE/STATUS_CHANGE
- 스크립트 (선택, 문자열): 이벤트 관련 텍스트(노트/메모 등)
- 참여자 (필수, 문자열 배열): 참여자 리스트
- 제목 (선택, 문자열): 이벤트 제목
- 내용 (선택, 문자열): 상세 내용
- 파일 (선택, 문자열): 첨부 파일 경로
- 생성자 ID (필수, 문자열): 기록자
- 생성일 (필수, 날짜시간): 이벤트 발생 시각

**관계:**
- N ProjectHistories → 1 Project (프로젝트 이벤트)
- 1 ProjectHistory → N Summary (AI 요약)

**비즈니스 규칙:**
- 단계 변경 시 자동 기록
- 파일 첨부 시 저장 경로 관리

**사용 화면:**
- /projects/[projectId]/history: 히스토리 타임라인

#### Specification / SpecificationItem (스펙 명세)

**설명:** 프로젝트 견적 및 상세 명세서 관리

**주요 속성 (Specification):**
- 프로젝트 ID (필수, 참조, 고유): 연결된 Project ID
- 총 비용 (선택, 숫자): 전체 견적 금액
- 생성일/수정일 (필수, 날짜시간): 자동 생성/수정

**주요 속성 (SpecificationItem):**
- 스펙 ID (필수, 참조): 상위 Specification ID
- 항목명 (필수, 문자열): 명세 항목명
- 목적 (선택, 문자열): 항목 목적
- 기능 (선택, 문자열): 제공 기능
- 페이지 (선택, 문자열): 관련 페이지/화면
- 비용 (선택, 숫자): 항목별 금액

**관계:**
- 1 Specification → N SpecificationItems (명세 항목)
- 1 Specification → 1 Project (프로젝트 견적)

**비즈니스 규칙:**
- 프로젝트당 하나의 Specification
- 비용 합계가 총 비용과 일치해야 함
- 변경 시 승인 프로세스 필요

**사용 화면:**
- /projects/[projectId]/specification: 명세서 관리
- /proposals/new: 견적 생성 시 사용

#### WorkItem (작업 항목)

**설명:** 프로젝트의 구체적인 작업 단위 및 진행 관리

**주요 속성:**
- 프로젝트 ID (필수, 참조): 관련 Project ID
- 담당자 ID (선택, 참조): 할당된 Workforce ID
- 상태 (필수, 열거형): TODO/IN_PROGRESS/REVIEW/DONE/CANCELLED
- 제목 (필수, 문자열): 작업 내용
- 설명 (선택, 문자열): 상세 설명
- 시작일 (선택, 날짜): 작업 시작일
- 종료일 (선택, 날짜): 작업 종료일
- 작업량 퍼센트 (선택, 숫자): 전체 작업 중 비율
- 예상 시간 (선택, 숫자): 예상 소요 시간
- 예상 비용 (선택, 숫자): 예상 비용
- 실제 시간 (선택, 숫자): 실제 소요 시간
- 실제 비용 (선택, 숫자): 실제 비용
- 생성자 ID (필수, 문자열): 작업 생성자
- 생성일/수정일 (필수, 날짜시간): 자동 생성/수정

**관계:**
- N WorkItems → 1 Project (프로젝트 작업)
- N WorkItems → 0..1 Workforce (담당자 배정)

**비즈니스 규칙:**
- 예상/실제 시간 자동 계산
- 상태 변경 시 담당자 알림
- 완료 시 실제 시간 필수 입력

**사용 화면:**
- /projects/[projectId]/work-items: 작업 관리
- /projects/[projectId]/overview: 진행률 표시

#### DevTasks / DevTaskItem (개발 태스크)

**설명:** 개발 작업의 그룹화 및 세부 태스크 관리

**주요 속성 (DevTasks):**
- 프로젝트 ID (필수, 참조, 고유): 연결된 Project ID
- 생성일/수정일 (필수, 날짜시간): 자동 생성/수정

**주요 속성 (DevTaskItem):**
- 개발태스크 ID (필수, 참조): 상위 DevTasks ID
- 항목명 (필수, 문자열): 태스크 제목
- 상태 (필수, 열거형): TODO/IN_PROGRESS/REVIEW/DONE/BLOCKED
- 내용 (선택, 문자열): 상세 내용
- 담당자 ID (선택, 참조): 할당된 Workforce ID
- 시작일 (필수, 날짜): 작업 시작일
- 종료일 (필수, 날짜): 작업 종료일
- 작업일수 (필수, 숫자): 시작일과 종료일 사이의 실제 작업일 수 (Settings의 MonthlyWorkday 기준으로 자동 계산)
- 투입% (필수, 숫자): 전체 작업 시간 중 투입 비율 (0-100)
- 예상 투입 시간 (필수, 숫자): 예상 소요 시간 (시간 단위)
  - 투입 방식이 "%로 입력"인 경우: 작업일수 × (투입% ÷ 100) × 8시간
  - 투입 방식이 "시간으로 입력"인 경우: 사용자 입력값
- 실제 투입 시간 (필수, 숫자, 기본값 0): 실제 소요 시간 (일별 투입 기록의 합계로 자동 계산)
- 예상 투입 비용 (필수, 숫자): 예상 비용 (예상 투입 시간 × 담당자의 시간당 비용)
- 실제 투입 비용 (필수, 숫자, 기본값 0): 실제 비용 (실제 투입 시간 × 담당자의 시간당 비용, 자동 계산)
- 메모 (선택, 문자열): 추가 메모
- 생성일/수정일 (필수, 날짜시간): 자동 생성/수정

**관계:**
- 1 DevTasks → N DevTaskItems (개발 태스크 항목)
- 1 DevTasks → 1 Project (프로젝트 개발 작업)
- N DevTaskItems → 0..1 Workforce (담당자 배정)

**비즈니스 규칙:**
- 프로젝트당 하나의 DevTasks 그룹
- 작업일수는 Settings의 MonthlyWorkday 기준으로 자동 계산 (주말/공휴일 제외)
- 투입 방식 선택에 따라 예상 투입 시간 또는 투입%가 자동 계산됨
- 실제 투입 시간은 일별 투입 기록의 합계로 자동 계산
- 예상/실제 투입 비용은 담당 Workforce의 시간당 비용을 기준으로 자동 계산
- 태스크 완료율로 프로젝트 진행률 계산
- 담당자 변경 시 기존 기록 유지

**사용 화면:**
- /projects/[projectId]/dev-tasks: 개발 태스크 관리 (작업일수 계산, 투입 방식 선택, 비용 계산)
- /workforce/tasks: 개인 태스크 목록

#### ProjectTimeline (프로젝트 타임라인)

**설명:** 프로젝트 단계별 일정 관리 및 지연 모니터링

**주요 속성:**
- 프로젝트 ID (필수, 참조): 관련 Project ID
- 단계 (필수, 열거형): 프로젝트 단계
- 예정일 (선택, 날짜): 계획된 완료일
- 실제일 (선택, 날짜): 실제 완료일
- 지연 여부 (필수, 숫자): -1(단축)/0(정시)/1(지연)
- 메모 (선택, 문자열): 일정 관련 메모
- 생성일/수정일 (필수, 날짜시간): 자동 생성/수정

**관계:**
- N ProjectTimelines → 1 Project (프로젝트 단계 일정)

**비즈니스 규칙:**
- 단계 변경 시 타임라인 자동 업데이트
- 지연 시 담당자 알림
- 실제일 입력 시 지연 여부 자동 계산

**사용 화면:**
- /projects/[projectId]/timeline: 타임라인 관리
- /projects/[projectId]/overview: 지연 현황 표시

#### ProjectReview (프로젝트 회고)

**설명:** 프로젝트 완료 후 개선점 및 학습 내용 기록

**주요 속성:**
- 프로젝트 ID (필수, 참조, 고유): 관련 Project ID
- 지연 여부 (필수, 불린): 프로젝트 지연 발생 여부
- PM ID (필수, 문자열 배열): 프로젝트 매니저 ID 리스트
- 디자이너 ID (선택, 문자열 배열): 디자이너 ID 리스트
- 개발자 ID (선택, 문자열 배열): 개발자 ID 리스트
- QA ID (선택, 문자열 배열): QA 담당자 ID 리스트
- 가장 어려웠던 작업 (선택, 문자열): 프로젝트 하이라이트
- 가장 쉬웠던 작업 (선택, 문자열): 성공 요인
- 핵심 학습 내용 (선택, 문자열): 배운 점
- 고객 페인포인트 (선택, 문자열): 고객 불만 사항
- 개선사항 (선택, 문자열): 다음 프로젝트 개선점
- 생성자 ID (필수, 문자열): 회고 작성자
- 생성일/수정일 (필수, 날짜시간): 자동 생성/수정

**관계:**
- 1 ProjectReview → 1 Project (프로젝트 회고)

**비즈니스 규칙:**
- 프로젝트 완료 시 필수 작성
- 팀원별 피드백 수집
- 향후 프로젝트 개선 가이드로 활용

**사용 화면:**
- /projects/[projectId]/review: 회고 작성/수정
- /projects/completed: 완료 프로젝트 회고 조회

### Finance (재무 관리)

**설명:** 프로젝트 계약 및 청구/수금 관리

**추가된 기능:** Contract Management, Billing Management

#### Contract (계약)

**설명:** 프로젝트 계약 정보 및 기본 조건 관리

**주요 속성:**
- 프로젝트 ID (필수, 참조, 고유): 연결된 Project ID
- 계약명 (필수, 문자열): 계약서 제목
- 계약서 URL (선택, 문자열): 계약서 파일 경로
- 총 금액 (필수, 숫자): 계약 총 금액
- 총 세액 (선택, 숫자): 부가세 포함 총액
- 계약 시작일 (선택, 날짜): 계약 시작일
- 계약 종료일 (선택, 날짜): 계약 종료일
- 계약 유형 (선택, 설정값): **Settings.계약유형 목록**을 참조 (정적 enum로 고정하지 않음)
- 메모 (선택, 문자열): 계약 관련 메모
- 생성자 ID (필수, 문자열): 계약 생성자
- 생성일/수정일 (필수, 날짜시간): 자동 생성/수정

**관계:**
- 1 Contract → 1 Project (프로젝트당 하나의 계약, projectId로 고유 연결)
- 1 Contract → N Milestone (계약 마일스톤)
- 1 Contract → N DigitalSignature (전자서명 요청)

**비즈니스 규칙:**
- 프로젝트당 하나의 활성 계약
- 총 금액 변경 시 마일스톤 재계산 필요
- 계약서 파일 필수 첨부

**사용 화면:**
- /finance/contracts: 계약 목록
- /finance/contracts/new: 계약 생성
- /finance/contracts/[projectId]: 프로젝트별 계약 관리

#### Milestone (마일스톤)

**설명:** 계약의 청구/수금 단위 및 진행 관리

**주요 속성:**
- 계약 ID (필수, 참조): 상위 Contract ID
- 세금계산서 ID (선택, 참조): 연결된 TaxInvoice ID
- 마일스톤 유형 (필수, 설정값): **Settings.마일스톤유형 목록**을 참조 (정적 enum로 고정하지 않음)
- 상태 (필수, 열거형): PENDING/INVOICED/PAID/OVERDUE
- 예정 금액 (필수, 숫자): 청구 예정 금액
- 예정 세액 (선택, 숫자): 부가세 예정 금액
- 예정일 (필수, 날짜): 청구 예정일
- 지급 금액 (선택, 숫자): 실제 지급 금액
- 지급 세액 (선택, 숫자): 실제 지급 세액
- 지급일 (선택, 날짜): 실제 지급일
- 메모 (선택, 문자열): 마일스톤 관련 메모
- 생성자 ID (필수, 문자열): 마일스톤 생성자
- 생성일/수정일 (필수, 날짜시간): 자동 생성/수정

**관계:**
- N Milestones → 1 Contract (계약 마일스톤)
- N Milestones → 0..1 TaxInvoice (세금계산서 연결)

**비즈니스 규칙:**
- 예정일 초과 시 OVERDUE로 자동 변경
- 지급 시 PAID 상태로 변경
- 금액 합계가 계약 총액을 초과하지 않도록 검증

**사용 화면:**
- /finance/contracts/[projectId]: 마일스톤 목록/관리
- /finance/invoices: 청구 상태별 조회

#### TaxInvoice (세금계산서)

**설명:** 세금계산서 발행 및 관리

**주요 속성:**
- 고객사 ID (필수, 참조): 청구 대상 Client ID
- 상태 (필수, 열거형): REQUESTED/GA_REVIEW/INVOICE_SENT/PAYED
- 고객사 유형 (필수, 열거형): CORPORATE/INDIVIDUAL
- 계산서 유형 (필수, 열거형): CASH_RECEIPT/TAX_INVOICE
- 용도 유형 (필수, 열거형): RECEIPT/CLAIM (영수/청구)
- 이메일 수신 (선택, 문자열): 계산서 수신 이메일
- 작성일 (선택, 날짜): 계산서 작성일
- 총 금액 (필수, 숫자): 공급가액
- 총 세액 (선택, 숫자): 부가세
- 품목 (선택, 문자열): 공급 품목명
- 메모 (선택, 문자열): 계산서 관련 메모
- 생성자 ID (필수, 문자열): 계산서 생성자
- 생성일/수정일 (필수, 날짜시간): 자동 생성/수정

**관계:**
- N TaxInvoices → 1 Client (고객사 청구서)
- N TaxInvoices → N Milestone (마일스톤 청구서)

**비즈니스 규칙:**
- 상태 변경 시 고객사 이메일 자동 발송
- 작성일 기준 세금계산서 번호 자동 생성
- 취소 시 재발행 불가

**사용 화면:**
- /finance/invoices: 계산서 목록/관리
- /finance/invoices/new: 계산서 생성
- /finance/invoices/[invoiceId]: 계산서 상세/수정

#### DigitalSignature (전자서명)

**설명:** 전자서명 요청 및 진행 상태 관리

**주요 속성:**
- 계약 ID (필수, 참조): 서명 대상 Contract ID
- 상태 (필수, 열거형): **요청 전송 → GA 검토 → 전자서명 발송 → (이후 모두싸인 API 기반 상태 동기화)**  
  - 내부 프로세스 상태: `REQUESTED | GA_REVIEW | MODUSIGN_SENT`  
  - 모두싸인(API) 이력 이벤트: `document histories` 기반으로 수신/저장/표시 (아래 “모두싸인 API 연동” 참고)
- 이메일 수신 (선택, 문자열): 서명 요청 이메일
- 담당자 ID (필수, 문자열 배열): 서명 담당자 ID 리스트
- 메모 (선택, 문자열): 서명 요청 관련 메모
- 생성자 ID (필수, 문자열): 서명 요청자
- 생성일/수정일 (필수, 날짜시간): 자동 생성/수정
- 외부 문서 ID (선택, 문자열): 모두싸인 `documentId` (전자서명 발송 이후에만 존재)

**관계:**
- N DigitalSignatures → 1 Contract (계약 서명 요청)

**비즈니스 규칙:**
- 담당자별 서명 상태 개별 관리
- 만료 기한 30일
- 모든 담당자 서명 완료 시 계약 활성화

**모두싸인 API 연동 (상태/이력 수신):**
- 전자서명 “발송(MODUSIGN_SENT)” 이후에는, **모두싸인 API의 문서 이력 조회**를 통해 상태를 수신한다.  
  - API: `GET /documents/{documentId}/histories` (문서 이력 조회) — 참고: `https://developers.modusign.co.kr/reference/documentcontroller_getalldocumenthistories`
- 수신 가능한 이벤트(예시, 모두싸인 제공 값):
  - `START_SIGNING_REQUEST` (서명 요청 시작)
  - `SIGNING_REQUEST` (서명 요청 됨)
  - `FIRST_READING_DOCUMENT_IN_SIGNING_TURN` (서명 차례에 첫 문서 조회)
  - `READING_DOCUMENT_IN_SIGNING_TURN` (서명 차례에 문서 조회)
  - `VERIFICATION_FAILURE` (추가 인증 실패)
  - `VERIFICATION_EXCEEDED` (추가 인증 5회 실패)
  - `VERIFICATION_COMPLETED` (인증 성공)
  - `SIGNING_COMPLETED` (서명 입력)
  - `SIGNING_COMPLETED_ALL` (모든 서명 입력 완료)
  - `REJECT_SIGNING` (서명 거절)
  - `CANCEL_SIGNING` (서명 취소)
  - `CANCEL_SIGNING_REQUEST` (서명 요청 취소)
  - `REMIND_SIGNING` (서명 알림 재전송)
  - `CHANGE_SIGNING_EXPIRY_DATE` (서명 유효 기간 수정)
  - `CORRECT_SIGNING` (서명 수정)
  - `SIGNING_REQUEST_TO_HOST` (호스트에게 서명 요청)
  - `START_SESSION` (대면 서명 세션 시작)
  - `START_VIEW_ONLY_REQUEST` (열람 요청 시작)
  - `VIEW_ONLY_REQUEST` (열람 요청)
  - `VIEW_ONLY_COMPLETED` (열람 완료)
  - `VIEW_ONLY_COMPLETED_ALL` (모든 열람 완료)
  - `CANCEL_VIEW_ONLY_REQUEST` (열람 요청 취소)
  - `FIRST_READING_VIEW_ONLY_IN_OWN_TURN` (열람 요청자가 문서 처음 열람)
  - `READING_VIEW_ONLY_IN_OWN_TURN` (열람 요청자가 문서 열람)
  - `REMIND_VIEW_ONLY` (열람 재요청)
  - `CHANGE_VIEWING_EXPIRY_DATE` (열람 만료일 변경)

**사용 화면:**
- /finance/signatures: 서명 요청 목록
- /finance/signatures/new: 서명 요청 생성
- /finance/signatures/[signatureId]: 서명 진행 상태

### Workforce (인력 관리)

**설명:** 프로젝트에 투입되는 인력 리소스 정보 및 배정/태스크 관리를 위한 핵심 엔티티

**추가된 기능:** Workforce Management, Task System

#### Workforce (인력)

**설명:** 회사 소속 인력의 기본 정보 및 역량 데이터

**주요 속성:**
- 이름 (필수, 문자열): 인력의 실명
- 이메일 (필수, 문자열, 고유): 회사 이메일 주소
- 역할 (필수, 열거형 배열): PM/SALES/DESIGNER/DEVELOPER/QA/MANAGER/CLEVEL 중 복수 선택 가능
- 고용형태 (필수, 열거형): FULL_TIME/CONTRACT/FREELANCE 중 하나
- 전화번호 (선택, 문자열): 연락처
- 전화번호(회사) (선택, 문자열): 회사 전화번호
- 스킬스택 (선택, 문자열 배열): 보유 기술 스택
- 입사일 (선택, 날짜): 입사 일자
- **지역그룹 (필수, 열거형): KOR/SEA 중 하나** *(2024-12-17 추가)*
- **레벨 ID (필수, 참조): 연결된 GradeLevel ID** *(2024-12-17 추가)*
- 메모 (선택, 문자열): 추가 정보
- 생성일/수정일 (필수, 날짜시간): 자동 생성/수정

> ⚠️ **2024-12-17 변경**: 실제 연봉(annualSalary), 시간당 비용(costPerHour) 속성은 제거됨. 모든 비용 계산은 GradeLevel의 hourlyLevelCost를 참조.

**관계:**
- 1 Workforce → N Task (인력이 수행하는 태스크)
- 1 Workforce → N ResourceAssignment (프로젝트 배정)
- 1 Workforce → 0..1 WorkItem (프로젝트 작업 담당)
- 1 Workforce → 0..1 DevTaskItem (개발 태스크 담당)
- **N Workforce → 1 GradeLevel (인력이 속한 급여 레벨)** *(2024-12-17 추가)*

**비즈니스 규칙:**
- 이메일은 회사 도메인으로 제한 (@company.com)
- **시간당 비용은 GradeLevel.hourlyLevelCost를 참조 (직접 계산하지 않음)** *(2024-12-17 변경)*
- **regionGroup 선택 시 해당 지역의 GradeLevel만 선택 가능** *(2024-12-17 추가)*
- 퇴사 시 기존 태스크는 다른 인력으로 재배정 필요

**사용 화면:**
- /workforce/people: 인력 목록/검색/필터
- /workforce/people/new: 인력 등록
- /workforce/people/[id]: 인력 상세/수정
- /projects/[id]/overview: 프로젝트 배정 인력 표시

#### ResourceAssignment (리소스 배정)

**설명:** 프로젝트에 인력을 배정하고 투입량을 관리하는 연결 엔티티

**주요 속성:**
- 인력 ID (필수, 참조): 배정되는 Workforce ID
- 프로젝트 ID (필수, 참조): 배정되는 Project ID
- 시작일 (필수, 날짜): 배정 시작일
- 종료일 (필수, 날짜): 배정 종료일
- 예상 투입 퍼센트 (선택, 숫자): 전체 시간 중 투입 비율 (0-100)
- 예상 투입 시간 (선택, 숫자): 월간 예상 투입 시간
- 메모 (선택, 문자열): 배정 관련 메모
- 상태 (필수, 열거형): IN_PROGRESS/COMPLETED/DELAYED 중 하나
- 생성일/수정일 (필수, 날짜시간): 자동 생성/수정

**관계:**
- N ResourceAssignments → 1 Workforce (인력이 여러 프로젝트에 배정)
- N ResourceAssignments → 1 Project (프로젝트에 여러 인력 배정)

**비즈니스 규칙:**
- 시작일 ≤ 종료일
- 예상 투입 시간 = (종료일-시작일) × 일 근무시간 × (투입%/100)
- 배정 기간 중 태스크 투입량 모니터링 필요

**사용 화면:**
- /workforce/resources: 리소스 배정 관리
- /projects/[id]/overview: 프로젝트 리소스 현황
- /workforce/tasks: 태스크 생성 시 리소스 선택

#### Task (태스크)

**설명:** 인력의 구체적인 업무 단위 및 일별 투입 기록 관리

**주요 속성:**
- 인력 ID (필수, 참조): 담당 Workforce ID
- 프로젝트 ID (선택, 참조): 연결된 Project ID (optional)
- 태스크명 (필수, 문자열): 업무 내용
- 시작일 (필수, 날짜): 태스크 시작 예정일
- 종료 예정일 (필수, 날짜): 태스크 완료 예정일
- 실제 종료일 (선택, 날짜): 실제 완료일
- 상태 (필수, 열거형): IN_PROGRESS/COMPLETED 중 하나
- 메모 (선택, 문자열): 태스크 관련 메모
- 작업일수 (필수, 숫자): 시작일과 종료 예정일 사이의 실제 작업일 수 (Settings의 MonthlyWorkday 기준으로 자동 계산)
- 투입% (필수, 숫자): 전체 작업 시간 중 투입 비율 (0-100)
- 예상 투입 시간 (필수, 숫자): 예상 소요 시간 (시간 단위)
  - 투입 방식이 "%로 입력"인 경우: 작업일수 × (투입% ÷ 100) × 8시간
  - 투입 방식이 "시간으로 입력"인 경우: 사용자 입력값
- 실제 투입 시간 (필수, 숫자, 기본값 0): 실제 소요 시간 (TaskDailyEntry의 투입 시간 합계로 자동 계산)
- 예상 투입 비용 (필수, 숫자): 예상 비용 (예상 투입 시간 × 담당자의 시간당 비용)
- 실제 투입 비용 (필수, 숫자, 기본값 0): 실제 비용 (실제 투입 시간 × 담당자의 시간당 비용, 자동 계산)
- 생성자 ID (필수, 문자열): 태스크 생성자
- 생성일/수정일 (필수, 날짜시간): 자동 생성/수정

**관계:**
- N Tasks → 1 Workforce (인력이 수행하는 태스크)
- N Tasks → 0..1 Project (프로젝트 연결 태스크)
- 1 Task → N TaskDailyEntry (일별 투입 기록)

**비즈니스 규칙:**
- 시작일 ≤ 종료 예정일 ≤ 실제 종료일
- 상태가 COMPLETED일 때 실제 종료일 필수 입력
- 작업일수는 Settings의 MonthlyWorkday 기준으로 자동 계산 (주말/공휴일 제외)
- 투입 방식 선택에 따라 예상 투입 시간 또는 투입%가 자동 계산됨
- 실제 투입 시간은 TaskDailyEntry들의 투입 시간 합계로 자동 계산
- 예상/실제 투입 비용은 담당 Workforce의 시간당 비용을 기준으로 자동 계산
- 투입%는 0-100 사이의 값만 허용

**사용 화면:**
- /workforce/tasks: 개인 태스크 목록
- /workforce/tasks/today: 오늘의 태스크
- /workforce/tasks/new: 태스크 생성 (작업일수 계산, 투입 방식 선택, 비용 계산)
- /workforce/tasks/[id]: 태스크 상세/수정

#### TaskDailyEntry (일별 투입 기록)

**설명:** 태스크의 일별 실제 투입 시간을 기록하는 상세 엔티티

**주요 속성:**
- 태스크 ID (필수, 참조): 연결된 Task ID
- 날짜 (필수, 날짜, 고유): 기록 날짜
- 투입 시간 (필수, 숫자): 해당 날짜 투입 시간 (시간 단위)
- 메모 (선택, 문자열): 일별 업무 내용
- 생성일/수정일 (필수, 날짜시간): 자동 생성/수정

**관계:**
- N TaskDailyEntries → 1 Task (태스크의 일별 기록)

**비즈니스 규칙:**
- 같은 태스크의 같은 날짜에 중복 기록 불가
- 투입 시간은 0.5시간 단위로 기록 (30분 단위)
- 퇴근 전까지 기록 필수 (다음 날 기록 제한)

**사용 화면:**
- /workforce/tasks/today: 오늘 투입 시간 기록
- /workforce/tasks/[id]: 태스크별 일별 기록 조회

### Settings (시스템 설정)

**설명:** 시스템 운영에 필요한 기준 정보 및 설정 데이터

**추가된 기능:** Settings Management

#### GradeLevel (급여 레벨) *(2024-12-17 추가)*

**설명:** 지역 그룹별 급여 레벨 및 시간당 비용 관리. 실제 연봉 대신 레벨 기반 비용 산정에 사용.

**주요 속성:**
- 지역그룹 (필수, 열거형): KOR/SEA (추후 확장 가능)
- 레벨명 (필수, 문자열): 예) KOR-L1, SEA-Mid 등 (자유 입력)
- 평균연봉 (필수, 숫자): 해당 레벨의 평균 연봉 (KRW)
- 연간근무시간 (필수, 숫자, 기본값 2080): 시간당 비용 계산 기준
- 시간당레벨비용 (필수, 숫자, 읽기전용): 자동 계산 (avgAnnualSalary ÷ workHoursPerYear)
- 활성여부 (필수, 불린, 기본값 true): 비활성 시 신규 배정 불가
- 생성일/수정일 (필수, 날짜시간): 자동 생성/수정

**관계:**
- 1 GradeLevel → N Workforce (레벨에 속한 인력)

**비즈니스 규칙:**
- regionGroup + levelName 조합은 고유해야 함
- avgAnnualSalary는 0보다 커야 함
- 레벨 삭제는 금지 (비활성화로 처리) — 과거 데이터 참조 가능성
- hourlyLevelCost는 avgAnnualSalary ÷ workHoursPerYear로 자동 계산
- workHoursPerYear 기본값: 2080 (주 40시간 × 52주)

**사용 화면:**
- /settings (급여별 레벨 탭): 레벨 목록/생성/수정
- /workforce/people: 직원 추가/수정 시 레벨 선택

---

#### Settings (설정)

**설명:** 시스템 전역 설정 및 기준 데이터 관리

**주요 속성:**
- 스킬스택 목록 (필수, 문자열 배열): 사용 가능한 기술 스택 리스트
- 계약유형 목록 (필수, 문자열 배열): 사용 가능한 계약 유형 리스트 (설정값)
- 개발유형 목록 (필수, 문자열 배열): 사용 가능한 개발 유형 리스트
- 마일스톤유형 목록 (필수, 문자열 배열): 사용 가능한 마일스톤 유형 리스트 (설정값)
- 생성자 ID (필수, 문자열): 설정 생성자
- 생성일/수정일 (필수, 날짜시간): 자동 생성/수정

**관계:**
- 1 Settings → N Workforce (스킬스택 참조)
- 1 Settings → N Project (계약유형/개발유형 참조)
- 1 Settings → N Finance (계약유형/마일스톤유형 참조)

**비즈니스 규칙:**
- 설정은 시스템 전체에 적용되는 글로벌 데이터
- 변경 시 기존 데이터와의 호환성 검토 필요
- 최소 1개 이상의 옵션이 각 목록에 존재해야 함

**사용 화면:**
- /settings: 설정 관리 화면
- /workforce/people/new: 스킬스택 선택
- /projects/new: 계약유형/개발유형 선택

#### MonthlyWorkday (월별 근무일)

**설명:** 월별 근무일 수를 정의하여 인력 원가 계산의 기준으로 사용

**주요 속성:**
- 연도 (필수, 숫자): 해당 연도
- 월 (필수, 숫자): 해당 월 (1-12)
- 근무일 리스트 (필수, 숫자 배열): 해당 월의 근무일 (1-31)
- 생성자 ID (필수, 문자열): 근무일 설정자
- 생성일/수정일 (필수, 날짜시간): 자동 생성/수정

**관계:**
- 1 MonthlyWorkday → N Workforce (원가 계산에 사용)

**비즈니스 규칙:**
- 연도+월 조합으로 고유 (중복 불가)
- 근무일 리스트는 해당 월의 유효한 날짜만 포함
- 시간당 비용 계산 시 사용: **GradeLevel.hourlyLevelCost**를 기준으로 사용 (실제 연봉 기반 계산 금지)

**사용 화면:**
- /settings: 월별 근무일 관리
- /workforce/people/[id]: 원가 계산 표시

### AI Assist (AI 지원 기능)

**설명:** 프로젝트 관리에서 AI를 활용한 자동화 지원 기능

**추가된 기능:** AI-Powered Project Management

#### Summary (요약)

**설명:** 프로젝트/업무 기록(히스토리, 메모, 콜 로그 등)의 텍스트를 AI가 자동으로 요약

**주요 속성:**
- 프로젝트히스토리 ID (선택, 참조): 연결된 ProjectHistory ID
- 요약 내용 (필수, 문자열): AI 생성 요약 텍스트
- 생성일 (필수, 날짜시간): 자동 생성

**관계:**
- N Summaries → 0..1 ProjectHistory (프로젝트 관련 요약)

**비즈니스 규칙:**
- 원본 텍스트가 있으면 자동 생성 가능
- 요약은 읽기 전용 (수정 불가)
- 최대 1000자 제한

**사용 화면:**
- /projects/[projectId]/history: 프로젝트 히스토리 요약 표시

#### Requirement (요구사항 추출)

**설명:** 프로젝트 관련 기록/대화 내용에서 고객 요구사항을 AI가 자동 추출

**주요 속성:**
- 유형 (필수, 열거형): PROJECT
- 프로젝트 ID (선택, 참조): 연결된 Project ID
- 제목 (필수, 문자열): 요구사항 제목
- 요구사항 내용 (필수, 문자열): 상세 요구사항
- 스크립트 (선택, 문자열): 관련 스크립트 내용
- 상태 (필수, 열거형): PENDING/CONFIRMED/REJECTED 중 하나
- 우선순위 (필수, 열거형): HIGH/MEDIUM/LOW 중 하나
- 생성일/수정일 (필수, 날짜시간): 자동 생성/수정

**관계:**
- N Requirements → 0..1 Project (프로젝트 요구사항)
- N Requirements → N ProjectHistory (프로젝트 히스토리 연결)

**비즈니스 규칙:**
- 상태 변경 시 담당자 알림
- 우선순위에 따라 업무 우선순위 결정
- 중복 요구사항 방지

**사용 화면:**
- /projects/[projectId]/requirements: 프로젝트 요구사항 관리

---

## 3) 관계(Relationships) — 핵심 연결 구조

아래는 “운영 흐름”이 끊기지 않도록 설계된 주 관계들이다.

### 3.1 Client 중심
- **Client 1 — N Manager**
- **Client 1 — N Project**
- **Client 1 — N TaxInvoice**

### 3.2 Project 중심
- **Project N — 1 Client**
- **Project 1 — N ProjectHistory**
- **Project 1 — 0..1 Specification**, **Specification 1 — N SpecificationItem**
- **Project 1 — N WorkItem** (담당자 Workforce optional)
- **Project 1 — 0..1 DevTasks**, **DevTasks 1 — N DevTaskItem** (담당자 Workforce optional)
- **Project 1 — N ProjectTimeline**
- **Project 1 — 0..1 ProjectReview**
- **Project 1 — N Requirement**
- **Project 1 — 0..1 Contract**
- **Project 1 — N ResourceAssignment**
- **Project 1 — N Task** (Workforce task가 프로젝트에 연결될 수 있음)

### 3.3 Finance 중심
- **Contract 1 — 1 Project**
- **Contract 1 — N Milestone**
- **Contract 1 — N DigitalSignature**
- **Milestone N — 0..1 TaxInvoice** (하나의 세금계산서가 여러 마일스톤과 연결 가능)
- **TaxInvoice N — 1 Client**

### 3.4 Workforce 중심 (Task System 포함)
- **Workforce 1 — N Task** (개인별 태스크 관리, 일별 투입 기록)
- **Task 1 — N TaskDailyEntry** (일별 실제 투입시간/메모 기록)
- **Task N — 0..1 Project** (태스크가 프로젝트와 연결될 수 있음)
- **Workforce 1 — N ResourceAssignment** (프로젝트별 리소스 배정)
- **ResourceAssignment N — 1 Project** (배정 기간/예상 투입량 관리)
- **WorkItem N — 0..1 Workforce** (프로젝트 작업 단위 담당자 배정)
- **DevTaskItem N — 0..1 Workforce** (개발 태스크 담당자 배정)

---

## 4) 상태(State) 모델 — 주요 Enum

> 실제 enum 정의는 `docs/admin-spec/erd.md` 참고. 단, **ContractType/MilestoneType/DevType 등 일부 “type” 값은 Settings(설정값) 기반**으로 관리될 수 있다.

### 4.1 Requirements
- **RequirementType**: `PROJECT`
- **RequirementStatus**: `PENDING | CONFIRMED | REJECTED`
- **RequirementPriority**: `HIGH | MEDIUM | LOW`

### 4.2 Project
- **ProjectStage**: `PENDING | PLANNING | DESIGN | DEVELOPMENT | INTERNAL_QA | EXTERNAL_QA | MAINTENANCE | COMPLETED | PAUSED | CANCELLED`
- **WorkItemStatus**: `TODO | IN_PROGRESS | REVIEW | DONE | CANCELLED`
- **DevTaskStatus**: `TODO | IN_PROGRESS | REVIEW | DONE | BLOCKED`

### 4.3 Finance
- **ContractType**: **Settings의 계약유형 목록(설정값)**을 참조 (정적 enum로 고정하지 않음) — `Settings.계약유형 목록` 참고
- **MilestoneType**: **Settings의 마일스톤유형 목록(설정값)**을 참조 — `Settings.마일스톤유형 목록` 참고
- **MilestoneStatus**: `PENDING | INVOICED | PAID | OVERDUE`
- **InvoiceStatus**: `REQUESTED | GA_REVIEW | INVOICE_SENT | PAYED`
- **SignatureStatus (내부 프로세스)**: `REQUESTED | GA_REVIEW | MODUSIGN_SENT`
- **SignatureHistoryEvent (모두싸인 API 수신값)**: `GET /documents/{documentId}/histories`의 이벤트를 그대로 저장/표시 (문서 이력 기반)

### 4.4 Workforce
- **WorkforceType**: `FULL_TIME | CONTRACT | FREELANCE`
- **WorkforceRole**: `PM | SALES | DESIGNER | DEVELOPER | QA | MANAGER | ADMIN | TECH_LEAD`
- **TaskStatus**: `IN_PROGRESS | COMPLETED`
- **ResourceStatus**: `IN_PROGRESS | COMPLETED | DELAYED`

---

## 5) 모델 불일치 및 정리 포인트(현재 코드 기준)

현재 UI는 `lib/dummy-data/**`에 정의된 타입/enum을 일부 사용한다. 이 값들은 Convex 스키마/ERD와 1:1로 일치하지 않을 수 있다.

**주요 불일치 사항:**
- **Finance 더미 타입**: `finance.ts`의 `MilestoneMock`, `DigitalSignatureMock` 등이 ERD 모델과 다름
- **Task Status**: 더미데이터는 COMPLETED만 존재하나 UI에서 IN_PROGRESS도 사용

**권장 원칙:**
- 도메인 표준은 **`docs/admin-spec/erd.md`를 기준**으로 정하고
- UI는 매핑 레이어(예: adapter/mapper)에서 변환하거나, 더미데이터를 ERD 기준 enum으로 교체

---

## 6) 권한 시스템 (Permission System) *(2024-12-17 추가)*

### 6.1 역할(Role) 체계

**사용 가능한 역할:**
- **C-level**: 경영진 (최고 의사결정권자)
- **GA**: General Affairs (총무/관리)
- **Manager**: 관리자
- **PM**: 프로젝트 매니저
- **Sales**: 영업 담당
- **Designer**: 디자이너
- **Developer**: 개발자
- **QA**: QA 담당

**규칙:**
- 역할은 **복수 선택 가능** (한 사용자에게 여러 역할 부여)
- 권한은 **합집합**으로 적용 (하나의 역할이라도 허용하면 최종 허용)

### 6.2 핵심 보안 정책

```
- actual_cost: always hidden (실제 비용은 전 역할에서 노출 금지)
- cost_calculation: level_cost only (모든 비용 계산은 레벨 비용 기준)
- settings_access: C-level, GA only (설정 메뉴는 C-level, GA만 접근)
- employee_add: GA only (직원 추가는 GA만 가능)
```

### 6.3 직원 관리 테이블 컬럼 노출 정책

| 역할 | 노출 컬럼 |
|---|---|
| C-level, GA | 이름, 역할, 스킬, 고용형태, **레벨**, **시간당 레벨 비용** |
| Manager, PM, Sales, Designer, Developer, QA | 이름, 역할, 스킬 |

### 6.4 메뉴 접근 권한 (요약)

| 메뉴 | 접근 가능 역할 |
|---|---|
| Client | 전 역할 |
| Project | 전 역할 |
| Workforce > Calendar | 전 역할 |
| Workforce > My Tasks | 전 역할 (본인 화면만) |
| Workforce > Resource Management | 전 역할 |
| Workforce > Employee Management | 전 역할 (단, 추가는 GA만) |
| Finance > Dashboard | C-level, GA, Manager, PM, Sales |
| Finance > Contract | C-level, GA, Manager, PM, Sales |
| Finance > Invoice | C-level, GA, Manager, PM, Sales |
| Finance > E-sign | C-level, GA, Manager, PM, Sales |
| Settings | C-level, GA |

> 상세 권한 정책은 `docs/changes/메뉴별 권한 정책.md` 참고

---

## 7) UI Navigation Objects (Frontend)

### NavigationConfig / NavigationItem / LayoutPreference

**설명:** 프론트엔드에서 사용하는 네비게이션 및 레이아웃 설정 데이터 (서버 상태 없음)

**추가된 기능:** F04 (공통 레이아웃)

**주요 속성 (NavigationItem):**
- 라벨 (필수, 문자열): 메뉴 표시명
- 경로 (필수, 문자열): 라우팅 경로
- 아이콘 (선택, 문자열): 아이콘 이름
- 권한 (선택, 문자열): 접근 권한 범위
- 하위메뉴 (선택, NavigationItem 배열): 서브 메뉴
- 비활성화 (선택, 불린): 준비중 표시

**주요 속성 (NavigationConfig):**
- 권한범위 (필수, 열거형): ADMIN/PM/DEVELOPER 등
- 메뉴리스트 (필수, NavigationItem 배열): 권한별 메뉴 구성

**주요 속성 (LayoutPreference):**
- 사이드바 접힘 (선택, 불린): 사이드바 접기 상태
- 테마 (선택, 열거형): LIGHT/DARK/AUTO
- 언어 (선택, 열거형): KO/EN

**관계:**
- 1 NavigationConfig → N NavigationItems (권한별 메뉴 구성)
- 1 User → 0..1 LayoutPreference (사용자별 레이아웃 설정)

**비즈니스 규칙:**
- 권한별 메뉴 동적 표시
- LayoutPreference는 localStorage로 유지
- NavigationConfig는 런타임 고정 데이터

**사용 화면:**
- 모든 페이지: 사이드바 네비게이션
- /settings: 레이아웃 설정

---

## 8) 개발 가이드라인 (Step 8-3 준수)

**새로운 기능 개발 시 필수 준수 사항:**

1. **Entity 추가 시**: 새로운 개념이 등장하면 본 문서에 상세 정의
2. **속성 확장 시**: 필수/선택 여부, 타입, 비즈니스 규칙 명시
3. **관계 정의 시**: UI 플로우를 따라 자연어로 서술
4. **제약사항**: 검증 규칙과 조건을 상세히 기록
5. **사용 화면**: 실제 UI 경로와 목적을 기록

---

## 9) 관련 문서

- **Physical ERD**: `docs/admin-spec/erd.md`
- **PRD**: `docs/admin-spec/prd.md`
- **IA**: `docs/admin-spec/information-architecture.md`
- **Logical Architecture**: `docs/admin-spec/logical-architecture.md`


