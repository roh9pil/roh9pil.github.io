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

