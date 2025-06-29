---
layout: post
title:  "체계적인 프로젝트 요구사항 관리"
date:   2025-06-29 11:06:00 +0900
categories: SW Engineering
---

# 체계적인 요구사항 관리를 위한 표준 준수 데이터 아키텍처: PostgreSQL 레퍼런스 구현

## 서론

소프트웨어 요구사항 관리는 프로젝트 성공의 핵심 경로 활동이지만, 모호성, 추적성 부재, 통제되지 않는 변경은 프로젝트 실패의 주요 원인으로 작용합니다. 전통적인 문서 중심의 접근 방식은 규모가 커짐에 따라 한계를 드러내며, 영향 분석이나 규정 준수 검증에 필요한 체계적인 도구를 제공하지 못합니다.
본 보고서는 이러한 문제에 대한 해결책으로, PostgreSQL 기반의 요구사항 관리 시스템을 위한 포괄적인 데이터 아키텍처와 레퍼런스 구현을 제안합니다. 이 설계는 임의로 구성된 것이 아니라, ISO/IEC/IEEE 29148 국제 표준과 SWEBOK(Software Engineering Body of Knowledge) 가이드의 핵심 원칙에 기반하여 체계적으로 도출되었습니다.
보고서의 핵심 결과물은 이론적 모델을 넘어, 즉시 배포 가능한 완전한 PostgreSQL 스키마입니다. 여기에는 모든 테이블, 데이터 타입, 제약 조건, 추적성 분석 함수, 감사(Audit)를 위한 트리거가 포함되어 있어, 사용자가 견고하고 실무에 바로 적용 가능한 시스템의 기반을 즉시 마련할 수 있도록 지원합니다.
이 문서는 기술 리더와 시니어 엔지니어를 대상으로 하며, 엄격한 프로세스를 강제하고, 추적성 및 영향 분석과 같은 강력한 분석 기능을 제공하여 현대의 복잡하고 안전이 중요한(safety-critical) 소프트웨어 개발에 필수적인 시스템 구축을 위한 청사진을 제공하는 것을 목표로 합니다.

## 1. ISO 29148 및 SWEBOK의 기초 원칙
### 1.1. 요구사항 공학에서 표준의 역할
요구사항 관리 시스템 설계를 시작하기에 앞서, 그 기반이 되는 국제 표준과 지식 체계를 이해하는 것은 필수적입니다. ISO/IEC/IEEE 29148과 같은 표준 및 SWEBOK과 같은 가이드는 소프트웨어 산업을 위한 공통된 프레임워크와 용어를 제공합니다. 이는 규범적인 공식이라기보다는, 업계에서 "일반적으로 인정되는 지식(generally accepted knowledge)"을 증류한 것으로, 조직이 흔한 함정을 피하고 "기술적 요구사항 부채(technical requirement debt)"를 줄이는 데 도움을 줍니다.
ISO/IEC/IEEE 29148은 IEEE 830과 같은 오래된 표준을 대체하는 시스템 및 소프트웨어 요구사항 공학의 핵심 국제 표준입니다. 이 표준은 요구사항 공학을 발견, 도출, 분석, 검증, 확인, 소통, 문서화, 관리를 포함하는 전체 생명주기에 걸친 학제간 기능으로 정의합니다. SWEBOK은 이러한 요구사항 공학을 소프트웨어 공학이라는 더 넓은 분야 내에 위치시키며, 설계, 테스팅, 형상 관리와 같은 다른 지식 영역과의 연결고리를 제공합니다.

### 1.2. "좋은" 요구사항의 정의: 특성과 속성
표준의 핵심 교리 중 하나는 잘 작성된 요구사항의 특성을 정의하는 것입니다. 이러한 특성은 단순한 제안이 아니라, 요구사항의 검증 가능성과 명확성의 기반이 됩니다. ISO/IEC/IEEE 29148과 SWEBOK은 요구사항이 반드시 **필수적이고(necessary), 구현과 무관하며(implementation-free), 모호하지 않고(unambiguous), 일관되며(consistent), 완전하고(complete), 단일하며(singular), 실현 가능하고(feasible), 추적 가능하며(traceable), 검증 가능(verifiable)**해야 한다고 강조합니다. "일반적으로", "빠르게"와 같은 모호한 용어는 반드시 정량화 가능한 지표로 대체되어야 합니다.
이러한 특성들은 데이터베이스에서 우리가 수집해야 할 속성으로 직접 변환됩니다. 특히 ISO 표준 기반의 템플릿은 Id, Heading, Text, Owner, Priority, Source, Rationale, Difficulty, Type, Status, Verification Method와 같은 핵심 속성 목록을 제공하며, 이 목록은 우리가 설계할 핵심 requirements 테이블의 컬럼 구조의 기반을 형성합니다.

### 1.3. 요구사항의 분류 체계
모든 요구사항이 동일한 성격을 갖지는 않습니다. 표준은 적절한 구성, 할당 및 분석을 위해 필수적인 명확한 분류 시스템을 제공합니다. 가장 기본적인 분류는 시스템이 "무엇을(what)" 하는지를 정의하는 **기능 요구사항(Functional Requirements)**과 "어떻게(how)" 동작해야 하는지를 정의하는 비기능 요구사항(Non-Functional Requirements) 사이에서 이루어집니다. SWEBOK v4는 비기능 요구사항을 "기술적 제약(Technical Constraints)"과 "서비스 품질 제약(Quality of Service constraints)"으로 더욱 세분화합니다.
이 외에도 표준은 다음과 같은 중요한 요구사항 유형을 식별합니다.
 * 시스템 요구사항 vs. 소프트웨어 요구사항: 시스템 요구사항은 하드웨어, 소프트웨어, 사람을 포함한 전체 솔루션을 포괄하며, 소프트웨어 요구사항은 이로부터 파생됩니다.
 * 비즈니스 또는 미션 요구사항: 프로젝트를 추진하는 높은 수준의 목표입니다.
 * 이해관계자 요구사항: 사용자와 기타 이해관계자의 필요사항으로, 이는 시스템 요구사항으로 변환됩니다.
 * 도메인 요구사항: 항공, 금융, 의료와 같이 특정 산업 분야의 규정이나 표준에서 비롯된 제약 조건입니다.
 * 인터페이스 요구사항: 시스템이 외부 컴포넌트(다른 시스템, 하드웨어, API 등)와 상호작용하는 방식을 정의합니다.
이러한 분류 체계는 데이터 모델에 직접적인 영향을 미칩니다. 각 요구사항을 이 분류에 따라 범주화할 수 있어야 하며, 이는 데이터 무결성을 위해 열거형(enumerated type)으로 구현될 requirement_type 필드를 통해 관리될 것입니다.

### 1.4. 요구사항 생명주기 프로세스
표준은 요구사항을 일회성의 "수집" 단계가 아닌, 전체 생명주기에 걸쳐 반복되는 프로세스로 설명합니다. 우리 시스템은 이 프로세스를 지원하도록 설계되어야 합니다. 이 프로세스는 일반적으로 **도출(Elicitation) → 분석(Analysis) → 명세(Specification) → 확인(Validation) → 관리(Management)**의 단계를 따릅니다.
 * 도출: 요구사항의 출처(이해관계자, 도메인 지식, 비즈니스 규칙)를 식별하고 기법(인터뷰, 워크숍, 프로토타입)을 사용합니다.
 * 분석: 충돌을 해결하고, 개념을 모델링하며, 절충안을 협상합니다.
 * 명세: 요구사항을 명확하고 구조화된 형식으로 문서화합니다. SWEBOK v4는 비정형 자연어부터 모델 기반 명세까지 다양한 접근 방식을 강조합니다.
 * 확인: 명세된 요구사항이 올바른지 검토, 프로토타이핑, 인수 테스트를 통해 보장합니다.
 * 관리: 변경 사항 처리, 요구사항 추적, 버전/베이스라인 관리를 포함하는 지속적인 활동입니다.
이러한 프로세스 중심적 관점은 데이터 모델이 단순한 정적 저장소가 아니라, 분석, 확인, 관리와 같은 생명주기 활동을 능동적으로 지원해야 함을 시사합니다. '제안됨', '승인됨', '구현됨', '검증됨'과 같은 상태를 추적하는 status 속성은 요구사항이 이 단계들을 거치는 과정을 추적하는 데 중심적인 역할을 합니다. 마찬가지로 verification_method와 같은 속성도 선택적 메타데이터가 아니라 시스템 기능의 핵심입니다.
더 나아가, SWEBOK v3에서 v4로의 발전과 ISO 29148의 포괄적인 성격은 요구사항이 더 이상 단일한 자연어 명세서에만 국한되지 않음을 보여줍니다. 시스템은 "다국어(polyglot)" 요구사항 환경을 지원하도록 설계되어야 합니다. 즉, 하나의 요구사항이 전통적인 텍스트 문장일 수도, 인수 조건이 포함된 사용자 스토리일 수도, 또는 공식 모델(예: UML, SysML)에 대한 포인터일 수도 있습니다. 따라서 단일 text 필드만으로는 부족하며, 구조화된 데이터(예: BDD를 위한 Gherkin 구문)를 저장하기 위한 JSONB 필드나 외부 모델링 아티팩트와 연결하기 위한 전용 테이블 같은 유연한 구조를 채택하여 시스템이 다양한 개발 방법론에 적응할 수 있도록 해야 합니다.

## 2. 핵심 데이터 모델: 요구사항을 위한 관계형 청사진
### 2.1. 핵심 스키마의 엔티티-관계 다이어그램 (ERD)
데이터베이스의 전체 구조를 한눈에 파악할 수 있도록 핵심 엔티티와 그 관계를 시각적으로 표현하는 것은 매우 중요합니다. 아래 표는 제안된 데이터 모델의 주요 엔티티와 그 관계를 요약한 ERD입니다. 이는 projects, requirements, stakeholders, artifacts와 같은 핵심 테이블과 이들 간의 일대다(one-to-many) 및 다대다(many-to-many) 관계를 보여주며, 아키텍처에 대한 높은 수준의 이해를 돕습니다.

| 소스 테이블 | 관계 | 대상 테이블 | 설명 |
|---|---|---|---|
| projects | One-to-Many | requirements | 하나의 프로젝트는 여러 요구사항을 포함합니다. |
| stakeholders | One-to-Many | requirements | 하나의 이해관계자는 여러 요구사항의 출처 또는 소유자가 될 수 있습니다. |
| projects | One-to-Many | artifacts | 하나의 프로젝트는 여러 관련 아티팩트를 가집니다. |
| requirements | Many-to-Many | artifacts | 요구사항과 아티팩트는 traceability_links 테이블을 통해 다대다 관계를 맺습니다. |
| users | Many-to-Many | roles | 사용자와 역할은 user_roles 테이블을 통해 다대다 관계를 맺습니다. |
| roles | Many-to-Many | permissions | 역할과 권한은 role_permissions 테이블을 통해 다대다 관계를 맺습니다. |

### 2.2. 핵심 테이블 정의 (DDL)
이 섹션은 시스템의 기반이 되는 테이블에 대한 완전하고 주석이 달린 PostgreSQL DDL(Data Definition Language)을 제공합니다.
projects 테이블
요구사항을 담는 컨테이너 역할을 합니다.

```sql
CREATE TABLE projects (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    name VARCHAR(255) NOT NULL UNIQUE,
    description TEXT,
    created_at TIMESTAMPTZ NOT NULL DEFAULT NOW()
);
```

stakeholders 테이블
요구사항의 출처가 되는 사람이나 역할을 기록합니다.

```sql
CREATE TABLE stakeholders (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    name VARCHAR(255) NOT NULL,
    role VARCHAR(100),
    contact_info TEXT
);
```

artifacts 테이블
요구사항에 연결될 수 있는 모든 개발 산출물(예: 설계 문서, 테스트 계획, 코드 모듈, Jira 티켓)을 나타내는 일반 테이블입니다. 이는 유연한 추적성을 위한 핵심 요소입니다. 이러한 일반적인 아티팩트 테이블의 생성은 요구사항 시스템을 Jira나 Git과 같은 특정 도구 체인에서 분리시키는 중요한 설계 결정입니다. 이를 통해 시스템은 스키마 변경 없이 어떤 외부 엔티티와도 연결될 수 있어 확장성과 유지보수성이 크게 향상됩니다.

```sql
CREATE TABLE artifacts (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    project_id UUID NOT NULL REFERENCES projects(id) ON DELETE CASCADE,
    artifact_type VARCHAR(50) NOT NULL, -- 예: 'Jira Ticket', 'Git Commit', 'Design Document'
    name VARCHAR(255) NOT NULL,
    description TEXT,
    external_url TEXT, -- 외부 시스템의 아티팩트를 가리키는 URL
    created_at TIMESTAMPTZ NOT NULL DEFAULT NOW()
);
```

requirements 테이블
시스템의 중심 엔티티로, 그 구조는 1장에서 논의된 표준에서 직접 파생됩니다. UUID를 기본 키로 사용하여 안정적이고 고유한 식별자를 보장하고, ENUM 타입을 사용하여 데이터 품질을 강제하며, JSONB 필드를 통해 미래의 요구사항 형식에 대한 유연성을 확보합니다. 이는 기술적 선택을 넘어 데이터 품질, 무결성, 미래 확장성을 보장하는 아키텍처적 결정입니다.
-- ENUM 타입들을 먼저 정의합니다.

```sql
CREATE TYPE requirement_type_enum AS ENUM ('Functional', 'NonFunctional', 'Interface', 'Domain', 'System', 'Business');
CREATE TYPE requirement_status_enum AS ENUM ('Proposed', 'Approved', 'Implemented', 'Verified', 'Rejected', 'Obsolete');
CREATE TYPE priority_enum AS ENUM ('Critical', 'High', 'Medium', 'Low');
CREATE TYPE difficulty_enum AS ENUM ('High', 'Medium', 'Low');
CREATE TYPE verification_method_enum AS ENUM ('Test', 'Demonstration', 'Inspection', 'Analysis');

CREATE TABLE requirements (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    req_id VARCHAR(50) NOT NULL, -- 사람이 읽을 수 있는 프로젝트별 식별자 (예: 'PROJ-FUN-001')
    project_id UUID NOT NULL REFERENCES projects(id) ON DELETE CASCADE,
    heading TEXT NOT NULL, -- 요구사항의 짧은 제목
    text TEXT NOT NULL, -- 요구사항의 전체 내용
    rationale TEXT, -- 요구사항이 필요한 이유 설명
    source_id UUID REFERENCES stakeholders(id), -- 요구사항 출처
    owner_id UUID REFERENCES stakeholders(id), -- 요구사항 유지보수 책임자
    type requirement_type_enum NOT NULL, -- 요구사항 유형
    status requirement_status_enum NOT NULL DEFAULT 'Proposed', -- 요구사항 상태
    priority priority_enum, -- 우선순위
    difficulty difficulty_enum, -- 구현 난이도
    verification_method verification_method_enum, -- 검증 방법
    created_at TIMESTAMPTZ NOT NULL DEFAULT NOW(),
    updated_at TIMESTAMPTZ NOT NULL DEFAULT NOW(),
    version INTEGER NOT NULL DEFAULT 1, -- 낙관적 잠금 및 버전 관리를 위한 버전 번호
    custom_attributes JSONB, -- 사용자 스토리 포인트, Gherkin 구문 등 추가 속성을 위한 필드
    CONSTRAINT uq_project_req_id UNIQUE (project_id, req_id)
);
```

### 2.3. PostgreSQL 타입과 제약조건을 통한 데이터 무결성 강제
데이터 품질은 시스템의 신뢰성에 있어 가장 중요합니다. 애플리케이션 레벨의 로직에 의존하기보다 데이터베이스 자체의 기능을 사용하여 규칙을 강제하는 것이 훨씬 견고합니다. 예를 들어, status나 priority 같은 필드에 단순 TEXT 타입을 사용하는 대신, PostgreSQL의 CREATE TYPE... AS ENUM을 사용합니다. 이 접근법은 '완료', '처리됨'과 같이 모호하거나 비표준적인 값이 입력되는 것을 원천적으로 차단하고, 사전에 정의된 유효한 값('Proposed', 'Approved' 등)만 허용합니다. 이는 데이터베이스 계층에서부터 표준에서 요구하는 명확성과 일관성을 강제하여 데이터 손상 및 불일치를 방지하는 효과적인 방법입니다. 또한 FOREIGN KEY 제약 조건은 참조 무결성을 보장하고 NOT NULL 제약 조건은 핵심 필드의 완전성을 보장합니다.

## 3. 종단간 추적성 및 영향 분석을 위한 아키텍처
### 3.1. 추적성의 과제: 단순한 연결을 넘어서
추적성은 요구사항의 생명주기를 순방향(구현 및 테스트로)과 역방향(근원으로) 모두 따라갈 수 있는 능력으로, ISO 26262와 같은 안전-필수 표준에서 의무적으로 요구됩니다. 전통적으로 요구사항 추적성 매트릭스(RTM)가 이 결과물로 사용되지만, 이를 수동으로 생성하고 유지하는 것은 오류가 발생하기 쉽고 확장성이 떨어집니다. 따라서 시스템은 RTM 생성을 자동화하고 더 동적인 분석을 수행할 수 있어야 합니다.

### 3.2. 추적성을 그래프로 모델링하기
요구사항, 설계, 코드, 테스트 간의 관계는 단순한 선형 순서가 아니라 복잡한 웹, 즉 그래프를 형성합니다. 이러한 관계를 명시적으로 모델링하기 위해 전용 링크 테이블을 사용하는 것은 requirements 테이블에 여러 외래 키를 추가하는 것보다 훨씬 유연한 접근 방식입니다. 이 방식은 단순히 요구사항과 테스트를 연결하는 것을 넘어, 요구사항을 다른 요구사항(분해), 리스크(위험 관리), 결함 등 다양한 아티팩트와 연결할 수 있게 해줍니다. 이는 본질적으로 PostgreSQL 내에서 그래프 데이터베이스의 핵심 개념을 에뮬레이션하는 것으로, 훨씬 더 강력한 아키텍처입니다.
traceability_links 테이블 구현
이 테이블은 그래프의 '엣지(edge)' 역할을 하며, 관계의 속성을 정의합니다.

```sql
CREATE TYPE trace_link_type_enum AS ENUM (
    'satisfies',       -- 설계/코드가 요구사항을 만족시킴
    'verifies',        -- 테스트가 요구사항을 검증함
    'is_derived_from', -- 하위 요구사항이 상위 요구사항에서 파생됨
    'conflicts_with',  -- 다른 요구사항과 충돌함
    'relates_to'       -- 일반적인 관련성
);

CREATE TABLE traceability_links (
    id BIGSERIAL PRIMARY KEY,
    -- 요구사항과 아티팩트를 모두 참조하기 위해 두 개의 컬럼을 사용합니다.
    -- 실제 구현에서는 하나의 source_id와 target_id로 통합하고, 타입을 저장하는 컬럼을 추가할 수 있습니다.
    -- 여기서는 명확성을 위해 분리된 컬럼을 예시로 사용합니다.
    source_requirement_id UUID REFERENCES requirements(id) ON DELETE CASCADE,
    source_artifact_id UUID REFERENCES artifacts(id) ON DELETE CASCADE,
    target_requirement_id UUID REFERENCES requirements(id) ON DELETE CASCADE,
    target_artifact_id UUID REFERENCES artifacts(id) ON DELETE CASCADE,
    link_type trace_link_type_enum NOT NULL,
    created_at TIMESTAMPTZ NOT NULL DEFAULT NOW(),
    created_by TEXT, -- 변경을 수행한 사용자 ID
    CONSTRAINT check_source_present CHECK (source_requirement_id IS NOT NULL OR source_artifact_id IS NOT NULL),
    CONSTRAINT check_target_present CHECK (target_requirement_id IS NOT NULL OR target_artifact_id IS NOT NULL),
    CONSTRAINT check_no_self_link CHECK (
        (source_requirement_id IS NULL OR source_requirement_id <> target_requirement_id) AND
        (source_artifact_id IS NULL OR source_artifact_id <> target_artifact_id)
    )
);
```

### 3.3. PostgreSQL 재귀 쿼리를 이용한 영향 분석
추적성 그래프의 가장 큰 가치는 "만약 이 요구사항을 변경하면 어떤 테스트, 코드, 다른 요구사항이 영향을 받는가?"와 같은 "what-if" 질문, 즉 영향 분석을 수행하는 데 있습니다. 이는 전형적인 그래프 순회 문제이며, PostgreSQL의 WITH RECURSIVE 공통 테이블 표현식(CTE)은 이를 위한 완벽하고 고성능의 도구입니다.
이 기능을 활용하면 여러 번의 복잡한 애플리케이션 레벨 루프 없이도 traceability_links 그래프를 임의의 깊이까지 탐색할 수 있습니다. 복잡한 순회 로직을 애플리케이션에서 데이터베이스 계층으로 옮기는 것은 중요한 아키텍처 결정입니다. 애플리케이션에서 재귀 함수를 구현하면 데이터베이스에 여러 번 왕복하며 성능 저하를 유발하는 반면(N+1 쿼리 문제), 단일 WITH RECURSIVE 쿼리는 데이터베이스의 쿼리 최적화기를 활용하여 전체 순회를 한 번의 작업으로 효율적으로 수행하므로 훨씬 뛰어난 성능과 확장성을 제공합니다.
구현 예시: 하위 영향 분석 쿼리
주어진 requirement_id에서 시작하여 'is_derived_from', 'satisfies' 등의 링크를 따라 모든 영향을 받는 하위 요구사항, 설계, 코드 등을 찾는 재귀 쿼리 예시입니다.

```sql
-- 특정 요구사항('target_req_id'에 UUID 입력) 변경 시 영향을 받는 모든 하위 아티팩트를 찾는 쿼리
WITH RECURSIVE impact_analysis (source_id, target_id, target_type, link_type, depth, path) AS (
    -- Anchor member: 시작점 설정
    SELECT
        r.id AS source_id,
        tl.target_requirement_id AS target_id,
        'requirement' AS target_type,
        tl.link_type,
        1 AS depth,
        ARRAY[r.id] AS path
    FROM requirements r
    JOIN traceability_links tl ON r.id = tl.source_requirement_id
    WHERE r.id = 'your-requirement-uuid-here' -- <-- 여기에 분석할 요구사항의 UUID를 입력하세요.
    UNION ALL
    SELECT
        r.id AS source_id,
        tl.target_artifact_id AS target_id,
        'artifact' AS target_type,
        tl.link_type,
        1 AS depth,
        ARRAY[r.id] AS path
    FROM requirements r
    JOIN traceability_links tl ON r.id = tl.source_requirement_id
    WHERE r.id = 'your-requirement-uuid-here'

    UNION ALL

    -- Recursive member: 이전 단계의 결과를 기반으로 다음 단계 탐색
    SELECT
        ia.target_id,
        tl.target_requirement_id,
        'requirement',
        tl.link_type,
        ia.depth + 1,
        ia.path |
| ia.target_id
    FROM traceability_links tl
    JOIN impact_analysis ia ON tl.source_requirement_id = ia.target_id
    WHERE NOT (ia.target_id = ANY(ia.path)) -- 순환 참조 방지
    AND ia.target_type = 'requirement'

    UNION ALL

    SELECT
        ia.target_id,
        tl.target_artifact_id,
        'artifact',
        tl.link_type,
        ia.depth + 1,
        ia.path |
| ia.target_id
    FROM traceability_links tl
    JOIN impact_analysis ia ON tl.source_requirement_id = ia.target_id
    WHERE NOT (ia.target_id = ANY(ia.path)) -- 순환 참조 방지
    AND ia.target_type = 'requirement'
)
-- 결과 조회: 영향을 받는 요구사항과 아티팩트의 상세 정보
SELECT
    ia.depth,
    ia.link_type,
    ia.target_type,
    COALESCE(r.req_id, a.name) AS name,
    COALESCE(r.heading, a.description) AS description
FROM impact_analysis ia
LEFT JOIN requirements r ON ia.target_id = r.id AND ia.target_type = 'requirement'
LEFT JOIN artifacts a ON ia.target_id = a.id AND ia.target_type = 'artifact';
```

### 3.4. 요구사항 추적성 매트릭스(RTM) 생성
RTM은 프로젝트 관리자와 QA팀에게 핵심적인 산출물입니다. 아래 SQL 쿼리는 traceability_links 테이블에 대한 JOIN을 사용하여 요구사항이 해당 테스트 케이스 및 최신 테스트 실행 상태에 어떻게 매핑되는지를 보여주는 고전적인 RTM을 구성합니다. 이는 기반이 되는 그래프 모델이 어떻게 전통적인 표 형식의 보고서를 생성할 수 있는지 보여줍니다.

예시: 요구사항 추적성 매트릭스(RTM)

| Requirement ID | Requirement Description | Test Case ID | Test Status |
|---|---|---|---|
| PROJ-FUN-001 | 사용자는 이메일과 비밀번호로 로그인할 수 있어야 한다. | TC-AUTH-001 | PASSED |
| PROJ-FUN-001 | 사용자는 이메일과 비밀번호로 로그인할 수 있어야 한다. | TC-AUTH-002 | FAILED |
| PROJ-FUN-002 | 사용자는 비밀번호를 재설정할 수 있어야 한다. | TC-AUTH-003 | PASSED |
| PROJ-NFR-001 | 로그인 응답 시간은 1초 미만이어야 한다. | TC-PERF-001 | PASSED |
| PROJ-FUN-003 | 관리자는 사용자 계정을 비활성화할 수 있어야 한다. | (uncovered) | (uncovered) |
이 매트릭스는 어떤 요구사항이 테스트되지 않았는지(커버리지 갭), 또는 어떤 요구사항에 실패한 테스트가 있는지를 명확하게 보여주어 커버리지 분석의 기초를 제공합니다.

## 4. 불변의 역사 구현: 변경 관리 및 버전 관리
### 4.1. 변경의 불가피성
요구사항은 정적이지 않고 프로젝트 생명주기 동안 계속 진화합니다. 이러한 변경을 관리하는 것은 요구사항 관리의 핵심 기능입니다. 통제되지 않는 변경은 주요 프로젝트 리스크이며, 공식적인 변경 관리 프로세스는 영향 분석(3장 참조)과 감사 및 잠재적 롤백을 위한 모든 수정 내역의 명확한 이력 유지를 요구합니다.

### 4.2. 버전 관리 전략: 스냅샷 기반 감사
데이터 버전 관리에는 주로 스냅샷과 델타 저장 방식이 있습니다. 델타 저장(변경된 부분만 저장)은 공간 효율적일 수 있지만, 특정 시점의 버전을 재구성하려면 처음부터 모든 델타를 재생해야 하므로 복잡합니다. 반면, 스냅샷 저장(특정 시점의 레코드 전체 상태 저장)은 더 간단하며 어떤 과거 버전이든 쉽게 검색할 수 있습니다. 명확성과 재구성의 용이성이 가장 중요한 감사 추적의 목적상, 스냅샷 접근 방식이 더 우수합니다.

### 4.3. PostgreSQL 트리거를 이용한 구현
데이터가 변경될 때마다 이러한 스냅샷을 자동으로 캡처하는 메커니즘이 필요하며, PostgreSQL 트리거는 이를 위한 이상적인 도구입니다. requirements, traceability_links와 같이 감사하고자 하는 모든 테이블에 적용할 수 있는 제네릭 트리거 함수를 생성할 것입니다. 이 함수는 INSERT, UPDATE, DELETE 작업이 발생한 AFTER에 실행됩니다.
이러한 제네릭, 트리거 기반, 스냅샷 스타일의 감사 시스템을 구축하는 것은 강력하고 재사용 가능한 패턴입니다. OLD 및 NEW 레코드를 JSONB로 저장하는 것이 핵심 결정 사항입니다. 이는 requirements와 같은 감사 대상 테이블의 스키마가 변경되더라도 audit_log 테이블의 스키마는 변경할 필요가 없음을 의미합니다. 이로 인해 감사 시스템 자체가 매우 견고하고 유지보수가 용이해집니다. to_jsonb(NEW) 함수는 컬럼을 알 필요 없이 새로운 행 구조 전체를 자동으로 캡처하여 감사 메커니즘을 감사 대상 스키마로부터 분리시킵니다. 이는 잘 설계된 확장 가능한 시스템의 특징입니다.
audit_log 테이블 구현

```sql
CREATE TABLE audit_log (
    id BIGSERIAL PRIMARY KEY,
    table_name TEXT NOT NULL,
    record_pk TEXT NOT NULL, -- 다양한 PK 타입을 위해 TEXT로 저장
    operation_type CHAR(1) NOT NULL, -- 'I': Insert, 'U': Update, 'D': Delete
    changed_by TEXT NOT NULL, -- current_user 또는 애플리케이션 사용자 ID
    changed_at TIMESTAMPTZ NOT NULL DEFAULT NOW(),
    old_data JSONB, -- UPDATE, DELETE 시 이전 레코드 전체 저장
    new_data JSONB, -- INSERT, UPDATE 시 신규 레코드 전체 저장
    query TEXT -- (선택 사항) 실행된 쿼리 저장
);
```

PL/pgSQL 트리거 함수 구현
재사용 가능한 log_changes() 함수는 TG_OP, TG_TABLE_NAME, OLD, NEW와 같은 특수 변수를 사용하여 audit_log 테이블을 동적으로 채웁니다.

```sql
CREATE OR REPLACE FUNCTION log_changes()
RETURNS TRIGGER AS $$
DECLARE
    audit_row audit_log;
    pk_column_name TEXT;
BEGIN
    -- 기본 키 컬럼 이름을 동적으로 찾기 (간단한 예시로 'id'를 가정)
    -- 실제 프로덕션에서는 information_schema를 쿼리하여 PK를 찾아야 함
    pk_column_name := 'id';

    audit_row = ROW(
        NEXTVAL('audit_log_id_seq'),
        TG_TABLE_NAME,
        NULL, -- record_pk는 아래에서 설정
        LEFT(TG_OP, 1),
        session_user::TEXT,
        NOW(),
        NULL, -- old_data
        NULL, -- new_data
        current_query()
    );

    IF (TG_OP = 'UPDATE') THEN
        audit_row.old_data = to_jsonb(OLD);
        audit_row.new_data = to_jsonb(NEW);
        -- JSONB에서 PK 값 추출
        audit_row.record_pk = (NEW::jsonb ->> pk_column_name);
    ELSIF (TG_OP = 'DELETE') THEN
        audit_row.old_data = to_jsonb(OLD);
        -- JSONB에서 PK 값 추출
        audit_row.record_pk = (OLD::jsonb ->> pk_column_name);
    ELSIF (TG_OP = 'INSERT') THEN
        audit_row.new_data = to_jsonb(NEW);
        -- JSONB에서 PK 값 추출
        audit_row.record_pk = (NEW::jsonb ->> pk_column_name);
    END IF;

    INSERT INTO audit_log VALUES (audit_row.*);

    RETURN NULL; -- AFTER 트리거이므로 결과는 무시됨
END;
$$ LANGUAGE plpgsql;
```

### 4.4. 이력 재구성
감사 로그는 효과적으로 쿼리할 수 없을 때 쓸모가 없습니다. 이 아키텍처는 모든 변경 사항에 대한 불변의, 추가 전용(append-only) 원장을 생성합니다. 이는 단순한 버전 관리를 넘어, 규정 준수 및 감사(예: FDA, ISO 26262)를 위한 부인 방지(non-repudiable) 소스 역할을 합니다. 감사관이 "변경 요청이 승인되기 전후의 이 안전 요구사항 상태와 누가 승인했는지 보여주시오"라고 질문할 때, audit_log는 old_data, new_data JSONB 객체와 함께 changed_by 사용자 및 정확한 changed_at 타임스탬프를 제공하여 명확한 증거를 제시할 수 있습니다.
구현 예시: 특정 요구사항의 전체 버전 이력 조회
SELECT
    changed_at,
    operation_type,
    changed_by,
    new_data ->> 'status' AS status,
    new_data ->> 'version' AS version,
    new_data ->> 'text' AS text
FROM audit_log
WHERE table_name = 'requirements'
  AND record_pk = 'your-requirement-uuid-here' -- <-- 조회할 요구사항의 UUID
ORDER BY changed_at ASC;

## 5. 안전한 기반: 역할 기반 접근 제어 (RBAC)
### 5.1. 최소 권한의 원칙
모든 사용자가 동일한 수준의 접근 권한을 가져서는 안 됩니다. 역할 기반 접근 제어(RBAC)는 이를 강제하는 표준 패턴으로, 사용자에게 직무 수행에 필요한 최소한의 권한만을 부여하여 우발적이거나 악의적인 손상 위험을 줄이는 핵심 보안 원칙입니다.

### 5.2. 유연한 RBAC 데이터 모델
단순히 사용자 테이블에 역할 플래그를 두는 것은 세분화된 제어에 불충분합니다. 역할과 권한을 별도의 테이블로 정규화하는 것이 훨씬 견고한 모델입니다. 이 정규화된 스키마는 사용자가 여러 역할을 가질 수 있게 하고(예: 팀 리더는 '관리자'이자 '개발자'), 관리자가 코드 배포 없이 데이터 변경만으로 새로운 역할을 정의하고 권한을 할당할 수 있게 하여 시스템을 훨씬 유연하고 확장 가능하게 만듭니다.
RBAC DDL 구현
```sql
CREATE TABLE users (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    username VARCHAR(100) NOT NULL UNIQUE,
    hashed_password TEXT NOT NULL,
    email VARCHAR(255) UNIQUE,
    is_active BOOLEAN DEFAULT TRUE
);

CREATE TABLE roles (
    id SERIAL PRIMARY KEY,
    role_name VARCHAR(50) NOT NULL UNIQUE,
    description TEXT
);

-- 권한은 데이터베이스 작업이 아닌 비즈니스 프로세스 작업에 기반해야 합니다.
-- 'UPDATE'가 아닌 'APPROVE_REQUIREMENT'와 같이 의미론적 권한을 정의하면
-- 보안 모델이 기본 스키마에서 분리되어 비즈니스 로직 변경에 더 탄력적으로 대응할 수 있습니다.
CREATE TABLE permissions (
    id SERIAL PRIMARY KEY,
    permission_name VARCHAR(100) NOT NULL UNIQUE, -- 예: 'CREATE_REQUIREMENT', 'APPROVE_REQUIREMENT'
    description TEXT
);

CREATE TABLE user_roles (
    user_id UUID NOT NULL REFERENCES users(id) ON DELETE CASCADE,
    role_id INTEGER NOT NULL REFERENCES roles(id) ON DELETE CASCADE,
    PRIMARY KEY (user_id, role_id)
);

CREATE TABLE role_permissions (
    role_id INTEGER NOT NULL REFERENCES roles(id) ON DELETE CASCADE,
    permission_id INTEGER NOT NULL REFERENCES permissions(id) ON DELETE CASCADE,
    PRIMARY KEY (role_id, permission_id)
);
```

### 5.3. 접근 제어 강제
데이터 모델만으로는 보안이 완성되지 않습니다. 애플리케이션 또는 데이터베이스 로직이 이를 사용하여 보안을 강제해야 합니다. 이는 애플리케이션 계층의 미들웨어를 통해 수행되거나, PostgreSQL의 행 수준 보안(Row-Level Security) 정책을 사용할 수 있습니다. 여기서는 권한 확인을 위한 데이터베이스 함수 예제를 제공합니다.
구현 예시: 권한 확인 함수

```sql
CREATE OR REPLACE FUNCTION has_permission(p_user_id UUID, p_permission_name VARCHAR)
RETURNS BOOLEAN AS $$
DECLARE
    has_perm BOOLEAN;
BEGIN
    SELECT EXISTS (
        SELECT 1
        FROM user_roles ur
        JOIN role_permissions rp ON ur.role_id = rp.role_id
        JOIN permissions p ON rp.permission_id = p.id
        WHERE ur.user_id = p_user_id
          AND p.permission_name = p_permission_name
    ) INTO has_perm;
    RETURN has_perm;
END;
$$ LANGUAGE plpgsql;
```

## 결론 
본 보고서는 소프트웨어 요구사항의 체계적인 관리를 위해 ISO/IEC 표준과 SWEBOK 가이드에 기반한 포괄적인 데이터 아키텍처를 제안하고, 이를 PostgreSQL에 즉시 배포할 수 있는 코드로 구체화했습니다.
제안된 아키텍처의 핵심 설계 결정은 다음과 같습니다.
 * 표준 준수 관계형 코어: ISO 29148에서 정의한 속성과 분류 체계를 직접 반영한 테이블 구조를 통해 데이터의 일관성과 명확성을 보장합니다.
 * 유연한 그래프 기반 추적성 모델: 요구사항과 관련 산출물 간의 복잡한 관계를 그래프로 모델링하고, PostgreSQL의 재귀 쿼리를 활용하여 강력한 동적 영향 분석을 가능하게 합니다.
 * 견고한 트리거 기반 감사 시스템: 모든 데이터 변경 사항을 스냅샷 형태로 JSONB에 기록하여, 스키마 변경에 유연하게 대응하고 규정 준수를 위한 불변의 감사 추적을 제공합니다.
 * 세분화된 RBAC 보안 계층: 역할과 권한을 정규화하여 최소 권한 원칙을 적용하고, 비즈니스 프로세스 중심의 권한 관리를 통해 시스템을 안전하게 보호합니다.
이 아키텍처는 사용자의 질의에 직접적으로 응답하여, 체계적이고 표준화되었으며 즉시 배포 가능한 솔루션을 제공합니다. 이는 단순한 데이터베이스 스키마를 넘어, 모범 사례를 강제하고, 추적성을 통해 깊이 있는 분석적 통찰력을 제공하며, 규정 준수와 보안을 보장함으로써 궁극적으로 프로젝트 리스크를 줄이고 소프트웨어 품질을 향상시키는 강력한 기반이 될 것입니다.


