# Physical ERD — Litmers Admin v2 (Convex Collections)

> 문서 목적: Convex 기반 데이터베이스 스키마(컬렉션/테이블)를 정의하고, 엔티티 간 관계/제약/인덱스를 명세한다.  
> Source of Truth: 본 문서 + `convex/schema.js`  
> 근거: `docs/admin-spec/conceptual-model.md`, `docs/admin-spec/prd.md`  
> 최종 업데이트: 2025-12-19 (스키마 감사 기반 업데이트)

---

## 1) ERD 다이어그램 (Mermaid)

```mermaid
erDiagram
    %% ===== Customer/CRM =====
    clients ||--o{ managers : "has"
    clients ||--o{ projects : "has"
    clients ||--o{ taxInvoices : "billed to"

    %% ===== Project/Delivery =====
    projects ||--o{ projectHistories : "logs"
    projects ||--o| specifications : "has"
    projects ||--o{ workItems : "contains"
    projects ||--o| devTasks : "has"
    projects ||--o{ projectTimelines : "tracks"
    projects ||--o| projectReviews : "reviewed by"
    projects ||--o{ requirements : "defines"
    projects ||--o| contracts : "has"
    projects ||--o{ resourceAssignments : "assigns"
    projects ||--o{ tasks : "linked to"

    specifications ||--o{ specificationItems : "contains"
    devTasks ||--o{ devTaskItems : "contains"
    projectHistories ||--o{ summaries : "summarized by"
    projectHistories }o--o{ requirements : "extracted from"

    %% ===== Finance =====
    contracts ||--o{ milestones : "has"
    contracts ||--o{ digitalSignatures : "requires"
    milestones }o--o| taxInvoices : "invoiced by"
    milestones ||--o{ paymentRecords : "paid by"

    %% ===== Workforce =====
    workforces ||--o{ tasks : "owns"
    workforces ||--o{ resourceAssignments : "assigned to"
    workforces }o--o{ workItems : "assigned to"
    workforces }o--o{ devTaskItems : "assigned to"
    workforces }o--|| gradeLevels : "belongs to"

    tasks ||--o{ taskDailyEntries : "logs"

    %% ===== Settings =====
    gradeLevels ||--o{ workforces : "includes"
    monthlyWorkdays ||--o{ tasks : "used by"
    settings ||--o{ settingsAuditLog : "audit logs"

    %% ===== System/Security =====
    userPermissions ||--o{ auditLogs : "tracks"
    integrationConfigs ||--o{ auditLogs : "logs changes"

    %% ===== Entity Definitions =====
    clients {
        id string PK
        name string
        type string "CORPORATE|INDIVIDUAL"
        corporateName string
        representativeName string
        businessNumber string UK
        address string
        registrationFile string
        individualName string
        idCardFile string
        createdAt number
        updatedAt number
    }

    managers {
        id string PK
        clientId string FK
        name string
        position string
        phone string
        email string
        createdAt number
        updatedAt number
    }

    projects {
        id string PK
        clientId string FK
        name string
        stage string "ProjectStage enum"
        pmIds array
        designerIds array
        developerIds array
        qaIds array
        startDate number
        endDate number
        contractType string "Settings ref"
        devType string "Settings ref"
        totalPrice number
        memo string
        createdById string
        createdAt number
        updatedAt number
    }

    projectHistories {
        id string PK
        projectId string FK
        stage string
        eventType string "CALL|EMAIL|NOTE|FILE|STATUS_CHANGE"
        script string
        participants array
        title string
        content string
        file string
        createdById string
        createdAt number
    }

    specifications {
        id string PK
        projectId string FK_UK
        totalCost number
        createdAt number
        updatedAt number
    }

    specificationItems {
        id string PK
        specificationId string FK
        itemName string
        purpose string
        feature string
        page string
        cost number
    }

    workItems {
        id string PK
        projectId string FK
        assigneeId string FK_nullable
        status string "WorkItemStatus"
        title string
        description string
        startDate number
        endDate number
        workPercent number
        estimatedHours number
        estimatedCost number
        actualHours number
        actualCost number
        createdById string
        createdAt number
        updatedAt number
    }

    devTasks {
        id string PK
        projectId string FK_UK
        createdAt number
        updatedAt number
    }

    devTaskItems {
        id string PK
        devTasksId string FK
        itemName string
        status string "DevTaskStatus"
        content string
        assigneeId string FK_nullable
        startDate number
        endDate number
        workDays number
        inputPercent number
        estimatedHours number
        actualHours number
        estimatedCost number
        actualCost number
        memo string
        createdAt number
        updatedAt number
    }

    projectTimelines {
        id string PK
        projectId string FK
        stage string "ProjectStage"
        plannedDate number
        actualDate number
        delayFlag number "-1|0|1"
        memo string
        createdAt number
        updatedAt number
    }

    projectReviews {
        id string PK
        projectId string FK_UK
        isDelayed boolean
        pmIds array
        designerIds array
        developerIds array
        qaIds array
        hardestWork string
        easiestWork string
        keyLearnings string
        customerPainPoints string
        improvements string
        createdById string
        createdAt number
        updatedAt number
    }

    requirements {
        id string PK
        type string "PROJECT"
        projectId string FK_nullable
        title string
        content string
        script string
        status string "RequirementStatus"
        priority string "RequirementPriority"
        createdAt number
        updatedAt number
    }

    requirementHistoryLinks {
        id string PK
        requirementId string FK
        projectHistoryId string FK
    }

    contracts {
        id string PK
        projectId string FK_UK
        name string
        documentUrl string
        totalAmount number
        totalTax number
        startDate number
        endDate number
        contractType string "Settings ref"
        signatureStatus string "denormalized"
        memo string
        createdById string
        createdAt number
        updatedAt number
    }

    milestones {
        id string PK
        contractId string FK
        taxInvoiceId string FK_nullable
        milestoneType string "Settings ref"
        status string "MilestoneStatus"
        expectedAmount number
        expectedTax number
        dueDate number
        paidAmount number
        paidTax number
        paidDate number
        memo string
        createdById string
        createdAt number
        updatedAt number
    }

    taxInvoices {
        id string PK
        clientId string FK
        status string "InvoiceStatus"
        clientType string "CORPORATE|INDIVIDUAL"
        invoiceType string "CASH_RECEIPT|TAX_INVOICE"
        usageType string "RECEIPT|CLAIM"
        recipientEmail string
        issuedDate number
        totalAmount number
        totalTax number
        itemName string
        memo string
        createdById string
        createdAt number
        updatedAt number
    }

    digitalSignatures {
        id string PK
        contractId string FK
        status string "SignatureStatus"
        recipientEmail string
        assigneeIds array
        memo string
        externalDocumentId string
        createdById string
        createdAt number
        updatedAt number
    }

    paymentRecords {
        id string PK
        milestoneId string FK
        amount number
        paymentMethod string "PaymentMethod enum"
        paidAt number
        confirmedBy string
        memo string
        createdAt number
    }

    workforces {
        id string PK
        name string
        email string UK
        roles array "WorkforceRole[]"
        employmentType string "WorkforceType"
        phone string
        companyPhone string
        skillStack array
        joinDate number
        regionGroup string "KOR|SEA"
        levelId string FK
        isActive boolean "default true"
        avatarUrl string
        memo string
        createdAt number
        updatedAt number
    }

    resourceAssignments {
        id string PK
        workforceId string FK
        projectId string FK
        startDate number
        endDate number
        expectedPercent number
        expectedHours number
        memo string
        status string "ResourceStatus"
        createdAt number
        updatedAt number
    }

    tasks {
        id string PK
        workforceId string FK
        projectId string FK_nullable
        name string
        startDate number
        expectedEndDate number
        actualEndDate number
        status string "TaskStatus"
        memo string
        workDays number
        inputPercent number
        estimatedHours number
        actualHours number
        estimatedCost number
        actualCost number
        createdById string
        createdAt number
        updatedAt number
    }

    taskDailyEntries {
        id string PK
        taskId string FK
        date number UK_with_taskId
        hours number "0-24 validation"
        memo string
        createdAt number
        updatedAt number
    }

    gradeLevels {
        id string PK
        regionGroup string "KOR|SEA"
        levelName string UK_with_regionGroup
        avgAnnualSalary number "positive validation"
        workHoursPerYear number
        hourlyLevelCost number "computed"
        isActive boolean
        createdAt number
        updatedAt number
    }

    settings {
        id string PK
        skillStackList array
        contractTypeList array
        devTypeList array
        milestoneTypeList array
        version number "auto increment"
        createdById string
        createdAt number
        updatedAt number
    }

    monthlyWorkdays {
        id string PK
        year number UK_with_month
        month number UK_with_year
        workdayList array
        createdById string
        createdAt number
        updatedAt number
    }

    summaries {
        id string PK
        projectHistoryId string FK_nullable
        content string
        createdAt number
    }

    auditLogs {
        id string PK
        userId string
        action string "CREATE|UPDATE|DELETE|LOGIN"
        entityType string
        entityId string
        changes any
        ipAddress string
        userAgent string
        createdAt number
    }

    userPermissions {
        id string PK
        userId string UK
        role string "ADMIN|PM|DEVELOPER|VIEWER"
        permissions array
        isActive boolean
        createdAt number
        updatedAt number
    }

    integrationConfigs {
        id string PK
        service string UK
        isEnabled boolean
        config any
        lastSyncAt number
        createdAt number
        updatedAt number
    }

    settingsAuditLog {
        id string PK
        settingType string
        previousValue any
        newValue any
        changedBy string
        changedAt number
    }
```

---

## 2) 컬렉션(테이블) 상세 정의

### 2.1 Customer/CRM

#### clients
| 필드 | 타입 | 필수 | 제약/설명 |
|------|------|:----:|-----------|
| id | `Id<"clients">` | O | PK, 자동 생성 |
| name | string | O | 고객사 표시명 |
| type | string | O | `CORPORATE` \| `INDIVIDUAL` |
| corporateName | string | - | 법인명 (CORPORATE) |
| representativeName | string | - | 대표자명 (CORPORATE) |
| businessNumber | string | - | 사업자등록번호 (UK, CORPORATE), 형식: `/^\d{3}-\d{2}-\d{5}$/` |
| address | string | - | 회사 주소 (CORPORATE) |
| registrationFile | string | - | 사업자등록증 파일 경로 |
| individualName | string | - | 개인명 (INDIVIDUAL) |
| idCardFile | string | - | 신분증 파일 경로 |
| createdAt | number | O | 생성 타임스탬프 |
| updatedAt | number | O | 수정 타임스탬프 |

**인덱스:**
- `by_businessNumber`: 사업자등록번호 고유성 검증
- `by_name`: 고객사명 검색
- `by_type`: 유형별 필터링 최적화 *(신규)*

---

#### managers
| 필드 | 타입 | 필수 | 제약/설명 |
|------|------|:----:|-----------|
| id | `Id<"managers">` | O | PK |
| clientId | `Id<"clients">` | O | FK → clients |
| name | string | O | 담당자명 |
| position | string | - | 직위 |
| phone | string | - | 연락처 |
| email | string | - | 이메일 (형식 검증) |
| createdAt | number | O | |
| updatedAt | number | O | |

**인덱스:**
- `by_clientId`: 고객사별 담당자 조회

---

### 2.2 Project/Delivery

#### projects
| 필드 | 타입 | 필수 | 제약/설명 |
|------|------|:----:|-----------|
| id | `Id<"projects">` | O | PK |
| clientId | `Id<"clients">` | O | FK → clients |
| name | string | O | 프로젝트명 |
| stage | string | O | `ProjectStage` enum |
| pmIds | string[] | O | PM 배열 (WorkOS userId 또는 workforceId) |
| designerIds | string[] | - | 디자이너 배열 |
| developerIds | string[] | - | 개발자 배열 |
| qaIds | string[] | - | QA 배열 |
| startDate | number | - | 프로젝트 시작일 |
| endDate | number | - | 프로젝트 종료일 |
| contractType | string | - | Settings 참조 |
| devType | string | - | Settings 참조 |
| totalPrice | number | - | 총 금액 |
| memo | string | - | 메모 |
| createdById | string | O | WorkOS userId |
| createdAt | number | O | |
| updatedAt | number | O | |

**인덱스:**
- `by_clientId`: 고객사별 프로젝트 목록
- `by_stage`: 단계별 필터링
- `by_createdAt`: 최신순 정렬
- `by_clientId_stage`: 고객사+단계별 복합 필터링 최적화 *(신규)*

**ProjectStage enum:**
`PENDING` | `PLANNING` | `DESIGN` | `DEVELOPMENT` | `INTERNAL_QA` | `EXTERNAL_QA` | `MAINTENANCE` | `COMPLETED` | `PAUSED` | `CANCELLED`

> ⚠️ **Validation 강화**: `stage` 값은 반드시 ProjectStage enum 값만 허용

---

#### projectHistories
| 필드 | 타입 | 필수 | 제약/설명 |
|------|------|:----:|-----------|
| id | `Id<"projectHistories">` | O | PK |
| projectId | `Id<"projects">` | O | FK → projects |
| stage | string | O | 기록 시점 단계 |
| eventType | string | O | `CALL` \| `EMAIL` \| `NOTE` \| `FILE` \| `STATUS_CHANGE` |
| script | string | - | 이벤트 관련 텍스트 |
| participants | string[] | O | 참여자 리스트 |
| title | string | - | 이벤트 제목 |
| content | string | - | 상세 내용 |
| file | string | - | 첨부 파일 경로 |
| createdById | string | O | WorkOS userId |
| createdAt | number | O | |

**인덱스:**
- `by_projectId`: 프로젝트별 히스토리
- `by_projectId_createdAt`: 프로젝트별 시간순 조회

---

#### specifications
| 필드 | 타입 | 필수 | 제약/설명 |
|------|------|:----:|-----------|
| id | `Id<"specifications">` | O | PK |
| projectId | `Id<"projects">` | O | FK → projects, UK (1:1) |
| totalCost | number | - | 전체 견적 금액 |
| createdAt | number | O | |
| updatedAt | number | O | |

**인덱스:**
- `by_projectId`: 프로젝트별 스펙 조회 (고유)

---

#### specificationItems
| 필드 | 타입 | 필수 | 제약/설명 |
|------|------|:----:|-----------|
| id | `Id<"specificationItems">` | O | PK |
| specificationId | `Id<"specifications">` | O | FK → specifications |
| itemName | string | O | 항목명 |
| purpose | string | - | 목적 |
| feature | string | - | 기능 |
| page | string | - | 관련 페이지 |
| cost | number | - | 항목별 금액 |

**인덱스:**
- `by_specificationId`: 스펙별 항목 목록

---

#### workItems
| 필드 | 타입 | 필수 | 제약/설명 |
|------|------|:----:|-----------|
| id | `Id<"workItems">` | O | PK |
| projectId | `Id<"projects">` | O | FK → projects |
| assigneeId | `Id<"workforces">` | - | FK → workforces (nullable) |
| status | string | O | `WorkItemStatus` enum |
| title | string | O | 작업 내용 |
| description | string | - | 상세 설명 |
| startDate | number | - | 작업 시작일 |
| endDate | number | - | 작업 종료일 |
| workPercent | number | - | 작업량 퍼센트 |
| estimatedHours | number | - | 예상 시간 |
| estimatedCost | number | - | 예상 비용 |
| actualHours | number | - | 실제 시간 |
| actualCost | number | - | 실제 비용 |
| createdById | string | O | WorkOS userId |
| createdAt | number | O | |
| updatedAt | number | O | |

**인덱스:**
- `by_projectId`: 프로젝트별 작업 목록
- `by_assigneeId`: 담당자별 작업 목록
- `by_status`: 상태별 필터링

**WorkItemStatus enum:**
`TODO` | `IN_PROGRESS` | `REVIEW` | `DONE` | `CANCELLED`

---

#### devTasks
| 필드 | 타입 | 필수 | 제약/설명 |
|------|------|:----:|-----------|
| id | `Id<"devTasks">` | O | PK |
| projectId | `Id<"projects">` | O | FK → projects, UK (1:1) |
| createdAt | number | O | |
| updatedAt | number | O | |

**인덱스:**
- `by_projectId`: 프로젝트별 DevTasks 조회 (고유)

---

#### devTaskItems
| 필드 | 타입 | 필수 | 제약/설명 |
|------|------|:----:|-----------|
| id | `Id<"devTaskItems">` | O | PK |
| devTasksId | `Id<"devTasks">` | O | FK → devTasks |
| itemName | string | O | 태스크 제목 |
| status | string | O | `DevTaskStatus` enum |
| content | string | - | 상세 내용 |
| assigneeId | `Id<"workforces">` | - | FK → workforces (nullable) |
| startDate | number | O | 작업 시작일 |
| endDate | number | O | 작업 종료일 |
| workDays | number | O | 작업일수 (자동 계산) |
| inputPercent | number | O | 투입% (0-100) |
| estimatedHours | number | O | 예상 투입 시간 |
| actualHours | number | O | 실제 투입 시간 (기본값 0) |
| estimatedCost | number | O | 예상 투입 비용 |
| actualCost | number | O | 실제 투입 비용 (기본값 0) |
| memo | string | - | 메모 |
| createdAt | number | O | |
| updatedAt | number | O | |

**인덱스:**
- `by_devTasksId`: DevTasks별 항목 목록
- `by_assigneeId`: 담당자별 태스크 목록
- `by_status`: 상태별 필터링

**DevTaskStatus enum:**
`TODO` | `IN_PROGRESS` | `REVIEW` | `DONE` | `BLOCKED`

---

#### projectTimelines
| 필드 | 타입 | 필수 | 제약/설명 |
|------|------|:----:|-----------|
| id | `Id<"projectTimelines">` | O | PK |
| projectId | `Id<"projects">` | O | FK → projects |
| stage | string | O | `ProjectStage` enum |
| plannedDate | number | - | 계획 완료일 |
| actualDate | number | - | 실제 완료일 |
| delayFlag | number | O | -1(단축) / 0(정시) / 1(지연) |
| memo | string | - | 메모 |
| createdAt | number | O | |
| updatedAt | number | O | |

**인덱스:**
- `by_projectId`: 프로젝트별 타임라인
- `by_projectId_stage`: 프로젝트+단계별 조회

---

#### projectReviews
| 필드 | 타입 | 필수 | 제약/설명 |
|------|------|:----:|-----------|
| id | `Id<"projectReviews">` | O | PK |
| projectId | `Id<"projects">` | O | FK → projects, UK (1:1) |
| isDelayed | boolean | O | 지연 여부 |
| pmIds | string[] | O | PM 리스트 |
| designerIds | string[] | - | 디자이너 리스트 |
| developerIds | string[] | - | 개발자 리스트 |
| qaIds | string[] | - | QA 리스트 |
| hardestWork | string | - | 가장 어려웠던 작업 |
| easiestWork | string | - | 가장 쉬웠던 작업 |
| keyLearnings | string | - | 핵심 학습 내용 |
| customerPainPoints | string | - | 고객 페인포인트 |
| improvements | string | - | 개선사항 |
| createdById | string | O | WorkOS userId |
| createdAt | number | O | |
| updatedAt | number | O | |

**인덱스:**
- `by_projectId`: 프로젝트별 회고 조회 (고유)

---

#### requirements
| 필드 | 타입 | 필수 | 제약/설명 |
|------|------|:----:|-----------|
| id | `Id<"requirements">` | O | PK |
| type | string | O | `PROJECT` |
| projectId | `Id<"projects">` | - | FK → projects (nullable) |
| title | string | O | 요구사항 제목 |
| content | string | O | 상세 요구사항 |
| script | string | - | 관련 스크립트 |
| status | string | O | `RequirementStatus` enum |
| priority | string | O | `RequirementPriority` enum |
| createdAt | number | O | |
| updatedAt | number | O | |

**인덱스:**
- `by_projectId`: 프로젝트별 요구사항
- `by_status`: 상태별 필터링
- `by_priority`: 우선순위별 필터링

**RequirementStatus enum:** `PENDING` | `CONFIRMED` | `REJECTED`  
**RequirementPriority enum:** `HIGH` | `MEDIUM` | `LOW`

---

#### requirementHistoryLinks (N:N 연결 테이블)
| 필드 | 타입 | 필수 | 제약/설명 |
|------|------|:----:|-----------|
| id | `Id<"requirementHistoryLinks">` | O | PK |
| requirementId | `Id<"requirements">` | O | FK → requirements |
| projectHistoryId | `Id<"projectHistories">` | O | FK → projectHistories |

**인덱스:**
- `by_requirementId`: 요구사항별 연결된 히스토리
- `by_projectHistoryId`: 히스토리별 연결된 요구사항

---

### 2.3 Finance

#### contracts
| 필드 | 타입 | 필수 | 제약/설명 |
|------|------|:----:|-----------|
| id | `Id<"contracts">` | O | PK |
| projectId | `Id<"projects">` | O | FK → projects, UK (1:1) |
| name | string | O | 계약명 |
| documentUrl | string | - | 계약서 파일 경로 |
| totalAmount | number | O | 계약 총 금액 (양수 검증 필요) |
| totalTax | number | - | 부가세 포함 총액 |
| startDate | number | - | 계약 시작일 |
| endDate | number | - | 계약 종료일 |
| contractType | string | - | Settings 참조 |
| signatureStatus | string | - | **서명 상태 (denormalized)** - digitalSignatures에서 동기화 *(신규)* |
| memo | string | - | 메모 |
| createdById | string | O | WorkOS userId |
| createdAt | number | O | |
| updatedAt | number | O | |

**인덱스:**
- `by_projectId`: 프로젝트별 계약 조회 (고유)

> ⚠️ **Validation**: `totalAmount`는 양수(> 0)만 허용

---

#### milestones
| 필드 | 타입 | 필수 | 제약/설명 |
|------|------|:----:|-----------|
| id | `Id<"milestones">` | O | PK |
| contractId | `Id<"contracts">` | O | FK → contracts |
| taxInvoiceId | `Id<"taxInvoices">` | - | FK → taxInvoices (nullable) |
| milestoneType | string | O | Settings 참조 |
| status | string | O | `MilestoneStatus` enum |
| expectedAmount | number | O | 청구 예정 금액 |
| expectedTax | number | - | 부가세 예정 금액 |
| dueDate | number | O | 청구 예정일 |
| paidAmount | number | - | 실제 지급 금액 |
| paidTax | number | - | 실제 지급 세액 |
| paidDate | number | - | 실제 지급일 |
| memo | string | - | 메모 |
| createdById | string | O | WorkOS userId |
| createdAt | number | O | |
| updatedAt | number | O | |

**인덱스:**
- `by_contractId`: 계약별 마일스톤 목록
- `by_status`: 상태별 필터링
- `by_dueDate`: 예정일 기준 정렬/필터
- `by_status_dueDate`: 상태+예정일 복합 (대시보드용)
- `by_dueDate_status`: 기간별 마일스톤 조회 최적화 *(신규)*

**MilestoneStatus enum:**
`PENDING` | `INVOICED` | `PAID` | `OVERDUE`

> ⚠️ **Validation**: `expectedAmount`, `paidAmount`는 양수(> 0)만 허용

---

#### taxInvoices
| 필드 | 타입 | 필수 | 제약/설명 |
|------|------|:----:|-----------|
| id | `Id<"taxInvoices">` | O | PK |
| clientId | `Id<"clients">` | O | FK → clients |
| status | string | O | `InvoiceStatus` enum |
| clientType | string | O | `CORPORATE` \| `INDIVIDUAL` |
| invoiceType | string | O | `CASH_RECEIPT` \| `TAX_INVOICE` |
| usageType | string | O | `RECEIPT` \| `CLAIM` |
| recipientEmail | string | - | 수신 이메일 |
| issuedDate | number | - | 작성일 |
| totalAmount | number | O | 공급가액 |
| totalTax | number | - | 부가세 |
| itemName | string | - | 품목명 |
| memo | string | - | 메모 |
| createdById | string | O | WorkOS userId |
| createdAt | number | O | |
| updatedAt | number | O | |

**인덱스:**
- `by_clientId`: 고객사별 계산서 목록
- `by_status`: 상태별 필터링
- `by_issuedDate`: 작성일 기준 정렬

**InvoiceStatus enum:**
`REQUESTED` | `GA_REVIEW` | `INVOICE_SENT` | `PAYED`

---

#### digitalSignatures
| 필드 | 타입 | 필수 | 제약/설명 |
|------|------|:----:|-----------|
| id | `Id<"digitalSignatures">` | O | PK |
| contractId | `Id<"contracts">` | O | FK → contracts |
| status | string | O | `SignatureStatus` enum |
| recipientEmail | string | - | 서명 요청 이메일 |
| assigneeIds | string[] | O | 서명 담당자 ID 리스트 |
| memo | string | - | 메모 |
| externalDocumentId | string | - | 모두싸인 documentId |
| createdById | string | O | WorkOS userId |
| createdAt | number | O | |
| updatedAt | number | O | |

**인덱스:**
- `by_contractId`: 계약별 서명 요청 목록
- `by_status`: 상태별 필터링

**SignatureStatus enum (내부 프로세스):**
`REQUESTED` | `GA_REVIEW` | `MODUSIGN_SENT`

---

### 2.4 Workforce

#### workforces
| 필드 | 타입 | 필수 | 제약/설명 |
|------|------|:----:|-----------|
| id | `Id<"workforces">` | O | PK |
| name | string | O | 이름 |
| email | string | O | 이메일 (UK) |
| roles | string[] | O | `WorkforceRole[]` |
| employmentType | string | O | `WorkforceType` enum |
| phone | string | - | 연락처 |
| companyPhone | string | - | 회사 전화번호 |
| skillStack | string[] | - | 보유 기술 스택 |
| joinDate | number | - | 입사일 |
| regionGroup | string | O | `KOR` \| `SEA` |
| levelId | `Id<"gradeLevels">` | O | FK → gradeLevels |
| isActive | boolean | O | **활성 상태** (기본값 true, 퇴사자 관리용) *(신규)* |
| avatarUrl | string | - | **프로필 이미지 URL** *(신규)* |
| memo | string | - | 메모 |
| createdAt | number | O | |
| updatedAt | number | O | |

**인덱스:**
- `by_email`: 이메일 고유성 검증/조회
- `by_levelId`: 레벨별 인력 목록
- `by_regionGroup`: 지역그룹별 필터링
- `by_isActive`: 활성 인력만 필터링 *(신규)*

**WorkforceRole enum:**
`PM` | `SALES` | `DESIGNER` | `DEVELOPER` | `QA` | `MANAGER` | `CLEVEL` | `GA`

**WorkforceType enum:**
`FULL_TIME` | `CONTRACT` | `FREELANCE`

> ⚠️ **정책**: 실제 연봉(annualSalary), 시간당비용(costPerHour) 필드는 저장하지 않음. 비용 계산은 `gradeLevels.hourlyLevelCost` 참조.

---

#### resourceAssignments
| 필드 | 타입 | 필수 | 제약/설명 |
|------|------|:----:|-----------|
| id | `Id<"resourceAssignments">` | O | PK |
| workforceId | `Id<"workforces">` | O | FK → workforces |
| projectId | `Id<"projects">` | O | FK → projects |
| startDate | number | O | 배정 시작일 |
| endDate | number | O | 배정 종료일 |
| expectedPercent | number | - | 예상 투입% (0-100) |
| expectedHours | number | - | 예상 투입 시간 |
| memo | string | - | 메모 |
| status | string | O | `ResourceStatus` enum |
| createdAt | number | O | |
| updatedAt | number | O | |

**인덱스:**
- `by_workforceId`: 인력별 배정 목록
- `by_projectId`: 프로젝트별 배정 목록
- `by_status`: 상태별 필터링
- `by_projectId_dateRange`: 프로젝트+기간 복합 조회
- `by_workforceId_startDate`: 캘린더뷰 성능 향상 *(신규)*

**ResourceStatus enum:**
`IN_PROGRESS` | `COMPLETED` | `DELAYED`

---

#### tasks
| 필드 | 타입 | 필수 | 제약/설명 |
|------|------|:----:|-----------|
| id | `Id<"tasks">` | O | PK |
| workforceId | `Id<"workforces">` | O | FK → workforces |
| projectId | `Id<"projects">` | - | FK → projects (nullable) |
| name | string | O | 태스크명 |
| startDate | number | O | 시작 예정일 |
| expectedEndDate | number | O | 종료 예정일 |
| actualEndDate | number | - | 실제 종료일 |
| status | string | O | `TaskStatus` enum |
| memo | string | - | 메모 |
| workDays | number | O | 작업일수 (자동 계산) |
| inputPercent | number | O | 투입% (0-100) |
| estimatedHours | number | O | 예상 투입 시간 |
| actualHours | number | O | 실제 투입 시간 (기본값 0) |
| estimatedCost | number | O | 예상 비용 |
| actualCost | number | O | 실제 비용 (기본값 0) |
| createdById | string | O | WorkOS userId |
| createdAt | number | O | |
| updatedAt | number | O | |

**인덱스:**
- `by_workforceId`: 인력별 태스크 목록
- `by_workforceId_status`: 인력+상태별 필터
- `by_projectId`: 프로젝트별 태스크 목록
- `by_status`: 상태별 필터링
- `by_startDate`: 날짜 기준 정렬
- `by_startDate_expectedEndDate`: 범위 쿼리 최적화 *(신규)*

**TaskStatus enum:**
`IN_PROGRESS` | `COMPLETED`

---

#### taskDailyEntries
| 필드 | 타입 | 필수 | 제약/설명 |
|------|------|:----:|-----------|
| id | `Id<"taskDailyEntries">` | O | PK |
| taskId | `Id<"tasks">` | O | FK → tasks |
| date | number | O | 기록 날짜 (UK with taskId) |
| hours | number | O | 투입 시간 (0.5 단위, **0 < hours ≤ 24 검증**) |
| memo | string | - | 일별 업무 내용 |
| createdAt | number | O | |
| updatedAt | number | O | |

**인덱스:**
- `by_taskId`: 태스크별 일별 기록
- `by_taskId_date`: 태스크+날짜 복합 (고유성 검증)

> ⚠️ **Validation**: `hours`는 0 초과 24 이하만 허용

---

### 2.5 Settings

#### gradeLevels
| 필드 | 타입 | 필수 | 제약/설명 |
|------|------|:----:|-----------|
| id | `Id<"gradeLevels">` | O | PK |
| regionGroup | string | O | `KOR` \| `SEA` |
| levelName | string | O | 레벨명 (UK with regionGroup) |
| avgAnnualSalary | number | O | 평균 연봉 (**양수 검증 필요, > 0**) |
| workHoursPerYear | number | O | 연간 근무시간 (기본 2080) |
| hourlyLevelCost | number | O | 시간당 레벨비용 (계산: avgAnnualSalary ÷ workHoursPerYear) |
| isActive | boolean | O | 활성 여부 (기본 true) |
| createdAt | number | O | |
| updatedAt | number | O | |

**인덱스:**
- `by_regionGroup`: 지역그룹별 레벨 목록
- `by_regionGroup_levelName`: 고유성 검증
- `by_isActive`: 활성 레벨만 필터링

> ⚠️ **Validation**: `avgAnnualSalary`는 양수(> 0)만 허용

---

#### settings (adminSettings)
| 필드 | 타입 | 필수 | 제약/설명 |
|------|------|:----:|-----------|
| id | `Id<"adminSettings">` | O | PK |
| skillStackList | string[] | O | 스킬스택 목록 |
| contractTypeList | string[] | O | 계약유형 목록 |
| devTypeList | string[] | O | 개발유형 목록 |
| milestoneTypeList | string[] | O | 마일스톤유형 목록 |
| version | number | O | **설정 버전** (변경 시 자동 증가) *(신규)* |
| createdById | string | O | WorkOS userId |
| createdAt | number | O | |
| updatedAt | number | O | |

> 시스템에 하나의 settings 레코드만 존재 (싱글톤)  
> ⚠️ **패턴 강화**: `getOrCreate` 패턴으로 upsert 구현 권장

---

#### monthlyWorkdays
| 필드 | 타입 | 필수 | 제약/설명 |
|------|------|:----:|-----------|
| id | `Id<"monthlyWorkdays">` | O | PK |
| year | number | O | 연도 (UK with month) |
| month | number | O | 월 1-12 (UK with year) |
| workdayList | number[] | O | 근무일 리스트 (1-31) |
| createdById | string | O | WorkOS userId |
| createdAt | number | O | |
| updatedAt | number | O | |

**인덱스:**
- `by_year_month`: 연도+월 복합 (고유성 검증)
- `by_year`: 연도별 조회

---

### 2.6 AI Assist

#### summaries
| 필드 | 타입 | 필수 | 제약/설명 |
|------|------|:----:|-----------|
| id | `Id<"summaries">` | O | PK |
| projectHistoryId | `Id<"projectHistories">` | - | FK → projectHistories (nullable) |
| content | string | O | AI 요약 내용 (최대 1000자) |
| createdAt | number | O | |

**인덱스:**
- `by_projectHistoryId`: 히스토리별 요약 조회

---

### 2.7 System/Security *(신규 섹션)*

#### auditLogs *(신규)*
| 필드 | 타입 | 필수 | 제약/설명 |
|------|------|:----:|-----------|
| id | `Id<"auditLogs">` | O | PK |
| userId | string | O | 활동 수행자 ID (WorkOS userId) |
| action | string | O | `CREATE` \| `UPDATE` \| `DELETE` \| `LOGIN` \| `LOGOUT` |
| entityType | string | O | 대상 엔티티 타입 (예: `projects`, `clients`) |
| entityId | string | O | 대상 엔티티 ID |
| changes | any | - | 변경 내용 (이전/이후 값 기록) |
| ipAddress | string | - | 클라이언트 IP 주소 |
| userAgent | string | - | 브라우저/클라이언트 정보 |
| createdAt | number | O | 활동 발생 시간 |

**인덱스:**
- `by_userId`: 사용자별 활동 조회
- `by_entityType_entityId`: 특정 엔티티 변경 이력 조회
- `by_createdAt`: 시간순 정렬
- `by_action`: 활동 유형별 필터링
- `by_userId_createdAt`: 사용자별 시간순 활동 조회

**AuditAction enum:**
`CREATE` | `UPDATE` | `DELETE` | `LOGIN` | `LOGOUT`

> 📋 **용도**: 시스템 감사, 보안 규정 준수, 변경 이력 추적

---

#### userPermissions *(신규)*
| 필드 | 타입 | 필수 | 제약/설명 |
|------|------|:----:|-----------|
| id | `Id<"userPermissions">` | O | PK |
| userId | string | O | WorkOS userId (UK) |
| role | string | O | 사용자 역할 |
| permissions | string[] | O | 세부 권한 목록 |
| isActive | boolean | O | 권한 활성 여부 |
| createdAt | number | O | |
| updatedAt | number | O | |

**인덱스:**
- `by_userId`: 사용자별 권한 조회 (고유)
- `by_role`: 역할별 사용자 목록

**UserRole enum:**
`ADMIN` | `PM` | `DEVELOPER` | `DESIGNER` | `QA` | `VIEWER` | `GA` | `CLEVEL`

**Permission examples:**
- `clients.read`, `clients.write`, `clients.delete`
- `projects.read`, `projects.write`
- `finance.read`, `finance.write` (민감 데이터)
- `workforce.salary.read` (급여 정보 조회)
- `system.settings.write`

> 📋 **RBAC 구현**: 역할(role) 기반 + 세부 권한(permissions) 조합

---

#### integrationConfigs *(신규)*
| 필드 | 타입 | 필수 | 제약/설명 |
|------|------|:----:|-----------|
| id | `Id<"integrationConfigs">` | O | PK |
| service | string | O | 서비스명 (UK) - `github`, `clickup`, `slack`, `modusign` 등 |
| isEnabled | boolean | O | 통합 활성화 여부 |
| config | any | O | 서비스별 설정 (API 키, 웹훅 URL 등) |
| lastSyncAt | number | - | 마지막 동기화 시간 |
| createdAt | number | O | |
| updatedAt | number | O | |

**인덱스:**
- `by_service`: 서비스명으로 조회 (고유)
- `by_isEnabled`: 활성화된 통합만 필터링

**지원 서비스:**
- `github`: GitHub 웹훅 및 API 연동
- `clickup`: ClickUp 웹훅 및 태스크 동기화
- `modusign`: 모두싸인 전자서명 연동
- `slack`: Slack 알림
- `teams`: Microsoft Teams 연동
- `notion`: Notion 연동

---

### 2.8 Finance 확장 *(신규)*

#### paymentRecords *(신규)*
| 필드 | 타입 | 필수 | 제약/설명 |
|------|------|:----:|-----------|
| id | `Id<"paymentRecords">` | O | PK |
| milestoneId | `Id<"milestones">` | O | FK → milestones |
| amount | number | O | 실제 결제 금액 (양수 검증) |
| paymentMethod | string | O | 결제 방식 |
| paidAt | number | O | 실제 결제 시간 |
| confirmedBy | string | O | 확인자 WorkOS userId |
| memo | string | - | 결제 메모 |
| createdAt | number | O | |

**인덱스:**
- `by_milestoneId`: 마일스톤별 결제 기록
- `by_paidAt`: 결제일 기준 정렬

**PaymentMethod enum:**
`BANK_TRANSFER` | `CARD` | `CASH` | `OTHER`

> 📋 **용도**: 하나의 마일스톤에 대해 분할 결제 기록 관리

---

### 2.9 Settings 확장 *(신규)*

#### settingsAuditLog *(신규)*
| 필드 | 타입 | 필수 | 제약/설명 |
|------|------|:----:|-----------|
| id | `Id<"settingsAuditLog">` | O | PK |
| settingType | string | O | 변경된 설정 유형 (`adminSettings`, `gradeLevels` 등) |
| previousValue | any | - | 변경 전 값 |
| newValue | any | O | 변경 후 값 |
| changedBy | string | O | 변경자 WorkOS userId |
| changedAt | number | O | 변경 시간 |

**인덱스:**
- `by_settingType`: 설정 유형별 변경 이력
- `by_changedBy`: 변경자별 이력
- `by_changedAt`: 시간순 정렬

> 📋 **용도**: 시스템 설정 변경 감사, 롤백 지원

---

## 3) 관계 요약 (카디널리티)

| From | To | 관계 | 설명 |
|------|----|:----:|------|
| clients | managers | 1:N | 고객사 → 담당자 |
| clients | projects | 1:N | 고객사 → 프로젝트 |
| clients | taxInvoices | 1:N | 고객사 → 세금계산서 |
| projects | projectHistories | 1:N | 프로젝트 → 이력 |
| projects | specifications | 1:0..1 | 프로젝트 → 스펙 |
| projects | workItems | 1:N | 프로젝트 → 작업 |
| projects | devTasks | 1:0..1 | 프로젝트 → 개발태스크그룹 |
| projects | projectTimelines | 1:N | 프로젝트 → 타임라인 |
| projects | projectReviews | 1:0..1 | 프로젝트 → 회고 |
| projects | requirements | 1:N | 프로젝트 → 요구사항 |
| projects | contracts | 1:0..1 | 프로젝트 → 계약 |
| projects | resourceAssignments | 1:N | 프로젝트 → 리소스배정 |
| projects | tasks | 1:N | 프로젝트 → 태스크 |
| specifications | specificationItems | 1:N | 스펙 → 항목 |
| devTasks | devTaskItems | 1:N | 개발태스크그룹 → 항목 |
| contracts | milestones | 1:N | 계약 → 마일스톤 |
| contracts | digitalSignatures | 1:N | 계약 → 전자서명 |
| milestones | taxInvoices | N:0..1 | 마일스톤 → 세금계산서 |
| milestones | paymentRecords | 1:N | 마일스톤 → 결제기록 *(신규)* |
| projectHistories | summaries | 1:N | 이력 → 요약 |
| requirements | projectHistories | N:N | 요구사항 ↔ 이력 (링크테이블) |
| workforces | tasks | 1:N | 인력 → 태스크 |
| workforces | resourceAssignments | 1:N | 인력 → 배정 |
| workforces | workItems | 1:N | 인력 → 작업 |
| workforces | devTaskItems | 1:N | 인력 → 개발태스크 |
| gradeLevels | workforces | 1:N | 레벨 → 인력 |
| tasks | taskDailyEntries | 1:N | 태스크 → 일별기록 |
| adminSettings | settingsAuditLog | 1:N | 설정 → 감사로그 *(신규)* |

---

## 4) 삭제 정책 (논리적 규칙)

| Parent | Child | 정책 | 비고 |
|--------|-------|:----:|------|
| clients | managers | CASCADE | 고객사 삭제 시 담당자도 삭제 |
| clients | projects | RESTRICT | 프로젝트 존재 시 고객사 삭제 금지 |
| projects | projectHistories | CASCADE | 프로젝트 삭제 시 이력도 삭제 |
| projects | specifications | CASCADE | |
| projects | workItems | CASCADE | |
| projects | devTasks | CASCADE | |
| projects | projectTimelines | CASCADE | |
| projects | projectReviews | CASCADE | |
| projects | requirements | SET_NULL | 요구사항은 유지, projectId만 null |
| projects | contracts | CASCADE | |
| projects | resourceAssignments | CASCADE | |
| projects | tasks | SET_NULL | 태스크 유지, projectId만 null |
| contracts | milestones | CASCADE | 계약 삭제 시 마일스톤도 삭제 |
| contracts | digitalSignatures | CASCADE | |
| milestones | paymentRecords | CASCADE | 마일스톤 삭제 시 결제기록도 삭제 *(신규)* |
| workforces | tasks | RESTRICT | 태스크 존재 시 인력 삭제 금지 |
| workforces | resourceAssignments | RESTRICT | 배정 존재 시 인력 삭제 금지 |
| gradeLevels | workforces | RESTRICT | 인력 존재 시 레벨 삭제 금지 (비활성화로 처리) |
| tasks | taskDailyEntries | CASCADE | 태스크 삭제 시 일별기록도 삭제 |
| adminSettings | settingsAuditLog | CASCADE | 설정 삭제 시 감사로그도 삭제 *(신규)* |
| userPermissions | auditLogs | NONE | 감사로그는 독립적 보존 *(신규)* |
| integrationConfigs | - | - | 삭제 시 관련 웹훅 비활성화 *(신규)* |

> **Convex 참고**: Convex는 DB 레벨 FK/CASCADE를 지원하지 않으므로, mutation 레이어에서 논리적으로 구현해야 함.
> **auditLogs 보존 정책**: 감사 로그는 규정 준수를 위해 삭제하지 않고 영구 보존

---

## 5) 계산 필드 규칙

| 컬렉션 | 필드 | 계산 규칙 | 저장 여부 |
|--------|------|-----------|:--------:|
| gradeLevels | hourlyLevelCost | avgAnnualSalary ÷ workHoursPerYear | O (캐시) |
| devTaskItems | workDays | 시작일~종료일 사이 근무일 수 (monthlyWorkdays 참조) | O |
| devTaskItems | estimatedHours | workDays × (inputPercent ÷ 100) × 8 (또는 직접 입력) | O |
| devTaskItems | actualHours | 일별 투입 기록 합계 | O (갱신) |
| devTaskItems | estimatedCost | estimatedHours × workforce.gradeLevel.hourlyLevelCost | O |
| devTaskItems | actualCost | actualHours × workforce.gradeLevel.hourlyLevelCost | O (갱신) |
| tasks | workDays | 동일 규칙 | O |
| tasks | estimatedHours | 동일 규칙 | O |
| tasks | actualHours | TaskDailyEntry.hours 합계 | O (갱신) |
| tasks | estimatedCost | estimatedHours × workforce.gradeLevel.hourlyLevelCost | O |
| tasks | actualCost | actualHours × workforce.gradeLevel.hourlyLevelCost | O (갱신) |

---

## 6) WorkOS 통합 명세

### 6.1 인증/권한 구조
- **인증**: WorkOS AuthKit 사용 (세션/토큰)
- **역할 관리**: WorkOS Organizations/Roles 또는 본 시스템 `workforces.roles[]`로 이중 관리 가능
- **권한 체크**: 프론트엔드 `usePermissions` 훅 + 백엔드 mutation 검증

### 6.2 사용자 ID 참조
- `createdById`, `createdBy` 등 필드는 **WorkOS userId(문자열)** 저장
- 화면에서 사용자명 표시가 필요한 경우:
  - WorkOS API로 조회
  - 또는 `workforces` 테이블의 이메일로 매칭

### 6.3 별도 권한 테이블 없음
- RBAC용 별도 테이블(`roles`, `permissions` 등)은 이번 ERD에서 생성하지 않음
- `workforces.roles[]`와 `docs/admin-spec/2025-12-17-salary-permission-policy.md` 정책 문서 참조

---

## 7) ERD 커버리지 검증 체크리스트

### 7.1 사용자 시나리오(J1~J3) 검증

#### J1: 세일즈 완료 → 고객사/프로젝트 온보딩
| 단계 | 필요 엔티티 | 커버 여부 | 비고 |
|------|-------------|:--------:|------|
| 1. 고객사 생성 | clients, managers | ✅ | |
| 2. 프로젝트 생성 | projects | ✅ | clientId FK |
| 3. 요구사항/작업 구성 | requirements, workItems | ✅ | projectId FK |
| 4. 계약/마일스톤 생성 | contracts, milestones | ✅ | projectId FK (1:1) |

#### J2: 프로젝트 운영
| 단계 | 필요 엔티티 | 커버 여부 | 비고 |
|------|-------------|:--------:|------|
| 1. 단계/일정 관리 | projects.stage, projectTimelines | ✅ | |
| 2. 회의/결정 기록 | projectHistories, summaries | ✅ | AI 요약 연동 |
| 3. 작업/태스크 관리 | workItems, devTasks, devTaskItems | ✅ | assigneeId FK |
| 4. 리소스 배정/실투입 | resourceAssignments, tasks, taskDailyEntries | ✅ | |

#### J3: 청구/수금 운영
| 단계 | 필요 엔티티 | 커버 여부 | 비고 |
|------|-------------|:--------:|------|
| 1. 계약에 마일스톤 생성 | milestones | ✅ | contractId FK |
| 2. 세금계산서 연결/발행 | taxInvoices, milestones.taxInvoiceId | ✅ | |
| 3. 상태 관리 (미수/연체) | milestones.status, taxInvoices.status | ✅ | OVERDUE enum |
| 4. 전자서명 추적 | digitalSignatures, externalDocumentId | ✅ | 모두싸인 연동 |

### 7.2 기능 요구사항(FR 6.x) 검증

| FR | 기능 | 필요 엔티티/필드 | 커버 여부 |
|----|------|------------------|:--------:|
| 6.1 | 대시보드 KPI | projects.stage, milestones.status, tasks.status | ✅ |
| 6.2 | Client CRUD | clients, managers | ✅ |
| 6.3 | Projects CRUD | projects, projectHistories, workItems, devTasks, requirements, projectTimelines, projectReviews | ✅ |
| 6.4 | Finance CRUD | contracts, milestones, taxInvoices, digitalSignatures | ✅ |
| 6.5 | Workforce Task System | tasks, taskDailyEntries, resourceAssignments | ✅ |
| 6.5 | 직원 관리 | workforces, gradeLevels | ✅ |
| 6.6 | Settings | settings, monthlyWorkdays, gradeLevels | ✅ |
| 6.7 | System (권한) | workforces.roles, WorkOS 연동 | ✅ |

### 7.3 정책/보안/무결성 검증

| 항목 | 요구사항 | 커버 여부 | 비고 |
|------|----------|:--------:|------|
| 실제 연봉 미저장 | workforces에 annualSalary 없음 | ✅ | gradeLevels만 참조 |
| 실제 시급 미저장 | workforces에 costPerHour 없음 | ✅ | hourlyLevelCost만 참조 |
| 사업자번호 고유 | clients.businessNumber UK | ✅ | 인덱스 정의 |
| 이메일 고유 | workforces.email UK | ✅ | 인덱스 정의 |
| 레벨명 고유 | gradeLevels regionGroup+levelName UK | ✅ | 복합 인덱스 |
| 일별기록 고유 | taskDailyEntries taskId+date UK | ✅ | 복합 인덱스 |
| 삭제 정책 | CASCADE/RESTRICT/SET_NULL 정의 | ✅ | 섹션 4 참고 |

---

## 8) 후속 작업 항목

### 8.1 완료된 작업
- [x] `convex/schema.js`에 위 ERD 기반 스키마 정의 ✅ (2024-12-18 완료)
- [x] 스키마 감사 및 요구사항 분석 ✅ (2025-12-19 완료)
- [x] ERD 문서 업데이트 ✅ (2025-12-19 완료)

### 8.2 🔴 HIGH 우선순위 (즉시 처리 필요)

| # | 항목 | 모듈 | 예상 시간 |
|---|------|------|-----------|
| 1 | auditLogs 테이블 생성 | System | 4h |
| 2 | userPermissions 테이블 생성 | System | 4h |
| 3 | Finance 모듈 DB 연동 | Finance | 8h |
| 4 | milestones.by_dueDate_status 인덱스 | Finance | 30m |
| 5 | resourceAssignments.by_workforceId_startDate 인덱스 | Workforce | 30m |
| 6 | adminTasks.by_startDate_expectedEndDate 인덱스 | Workforce | 30m |
| 7 | Dashboard 집계 쿼리 생성 | Dashboard | 2h |
| 8 | 금액 필드 양수 검증 | Finance | 1h |

### 8.3 🟡 MEDIUM 우선순위 (단기 계획)

| # | 항목 | 모듈 | 예상 시간 |
|---|------|------|-----------|
| 9 | projects.by_clientId_stage 복합 인덱스 | Projects | 30m |
| 10 | stage enum validator 강화 | Projects | 1h |
| 11 | contracts.signatureStatus 필드 추가 | Finance | 1h |
| 12 | integrationConfigs 테이블 생성 | System | 2h |
| 13 | workforces.isActive 필드 추가 | Workforce | 1h |
| 14 | adminSettings 싱글톤 패턴 강화 | Settings | 1h |
| 15 | gradeLevels.avgAnnualSalary 양수 검증 | Settings | 30m |
| 16 | 시스템 대시보드 집계 쿼리 | System | 3h |

### 8.4 🟢 LOW 우선순위 (장기 개선)

| # | 항목 | 모듈 | 예상 시간 |
|---|------|------|-----------|
| 17 | clients.by_type 인덱스 | Clients | 30m |
| 18 | 사업자등록번호 형식 검증 | Clients | 1h |
| 19 | 프로젝트 목록 denormalization | Projects | 3h |
| 20 | paymentRecords 테이블 생성 | Finance | 4h |
| 21 | workforces.avatarUrl 필드 추가 | Workforce | 30m |
| 22 | taskDailyEntries.hours 범위 검증 | Workforce | 30m |
| 23 | adminSettings.version 버전 관리 | Settings | 1h |
| 24 | settingsAuditLog 테이블 생성 | Settings | 2h |

### 8.5 기존 작업 항목
- [ ] Convex mutations에서 논리적 FK/고유성/삭제정책 검증 로직 구현
- [ ] 기존 더미데이터 → Convex 마이그레이션 스크립트 작성
- [ ] 화면 컴포넌트 데이터 소스를 Convex queries로 전환

### 스키마 명명 규칙 (ERD ↔ Convex 매핑)

기존 Platform 테이블과 충돌을 피하기 위해 일부 컬렉션명이 다릅니다:

| ERD 엔티티명 | Convex 컬렉션명 | 비고 |
|-------------|----------------|------|
| tasks | `adminTasks` | Platform의 기타 tasks와 구분 |
| settings | `adminSettings` | Platform 설정과 구분 |
| auditLogs | `auditLogs` | *(신규)* System 감사 로그 |
| userPermissions | `userPermissions` | *(신규)* 사용자 권한 |
| integrationConfigs | `integrationConfigs` | *(신규)* 통합 설정 |
| paymentRecords | `paymentRecords` | *(신규)* 결제 기록 |
| settingsAuditLog | `settingsAuditLog` | *(신규)* 설정 감사 로그 |
| 기타 모든 엔티티 | ERD와 동일 | |

---

## 9) 관련 문서

- [conceptual-model.md](conceptual-model.md) — 도메인 개념 모델
- [prd.md](prd.md) — 제품 요구사항
- [logical-architecture.md](logical-architecture.md) — 논리 아키텍처
- [information-architecture.md](information-architecture.md) — 정보 아키텍처
- [2025-12-17-salary-permission-policy.md](2025-12-17-salary-permission-policy.md) — 급여/권한 정책
- [SCHEMA-AUDIT-REPORT.md](../schema-audit/SCHEMA-AUDIT-REPORT.md) — 스키마 감사 보고서 *(신규)*
- [IMPLEMENTATION-CHECKLIST.md](../schema-audit/IMPLEMENTATION-CHECKLIST.md) — 구현 체크리스트 *(신규)*

---

## 10) 변경 이력

| 날짜 | 버전 | 변경 내용 | 작성자 |
|------|------|----------|--------|
| 2024-12-18 | 1.0.0 | 최초 작성 | - |
| 2025-12-19 | 2.0.0 | 스키마 감사 결과 반영 | AI Agent |

### v2.0.0 주요 변경사항 (2025-12-19)

**신규 테이블 (5개)**
- `auditLogs`: 시스템 감사 로그
- `userPermissions`: 사용자별 권한 관리 (RBAC)
- `integrationConfigs`: 외부 서비스 통합 설정
- `paymentRecords`: 마일스톤별 결제 기록
- `settingsAuditLog`: 설정 변경 감사 로그

**신규 필드 (4개)**
- `contracts.signatureStatus`: 서명 상태 denormalization
- `workforces.isActive`: 퇴사자 관리 (활성 상태)
- `workforces.avatarUrl`: 프로필 이미지 URL
- `adminSettings.version`: 설정 버전 관리

**신규 인덱스 (5개)**
- `clients.by_type`: 유형별 필터링
- `projects.by_clientId_stage`: 고객사+단계 복합 인덱스
- `milestones.by_dueDate_status`: 기간별 마일스톤 조회
- `resourceAssignments.by_workforceId_startDate`: 캘린더뷰 최적화
- `adminTasks.by_startDate_expectedEndDate`: 범위 쿼리 최적화

**Validation 강화**
- `contracts.totalAmount`, `milestones.expectedAmount`: 양수 검증
- `gradeLevels.avgAnnualSalary`: 양수 검증
- `taskDailyEntries.hours`: 0-24 범위 검증
- `clients.businessNumber`: 형식 검증 (정규식)
- `projects.stage`: enum 값 검증

**Enum 추가**
- `WorkforceRole`에 `GA` 추가
- `AuditAction`: `CREATE` | `UPDATE` | `DELETE` | `LOGIN` | `LOGOUT`
- `UserRole`: `ADMIN` | `PM` | `DEVELOPER` | `DESIGNER` | `QA` | `VIEWER` | `GA` | `CLEVEL`
- `PaymentMethod`: `BANK_TRANSFER` | `CARD` | `CASH` | `OTHER`
