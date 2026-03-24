# Claude Code 비즈니스 지식 체계 (Business Knowledge System)

---

## 1. 개요

### 해결하고자 하는 문제

코드에는 비즈니스 규칙이 묻혀 있다. validation 조건, 에러 메시지, 상수, enum, 상태 전이 로직, 주석 — 이것들이 곧 서비스 정책이지만, 코드를 직접 읽지 않으면 알 수 없다.

기획자가 "최소 참가자 수가 몇 명이지?"라고 물으면 개발자가 코드를 뒤져야 하고, 신규 팀원이 "이 에러는 언제 발생하지?"를 파악하려면 소스 코드를 추적해야 한다. CS 대응 시에도 "이 상황에서 시스템이 어떻게 동작하는가?"를 즉시 답하기 어렵다.

이 문서는 **코드 지식 체계**(project-knowledge-system.md)의 산출물을 입력으로 받아, 코드에 내재된 비즈니스 규칙을 체계적으로 추출하고, **비개발자도 탐색 가능한 형태**로 정리하는 체계를 정의한다.

### 코드 지식 체계와의 관계

```
┌─────────────────────────────┐
│  코드 지식 체계 (완성)        │
│  DOMAIN_MAP / FEATURES /     │
│  ENTITIES / SERVICE_FLOW     │
└──────────┬──────────────────┘
           │ 입력
           ▼
┌─────────────────────────────┐
│  비즈니스 지식 체계 (본 문서)  │
│  코드에서 비즈니스 규칙 추출   │
│  → 정책서 / 기획서 / 용어집   │
│  → 정적 웹사이트로 출력       │
└─────────────────────────────┘
```

코드 지식 체계가 "어디에 영향이 가는가"(구조)를 다루고, 비즈니스 지식 체계는 "왜 이렇게 동작하는가"(규칙)를 다룬다.

### 핵심 목표

- **기획자/PO가** → "이 기능의 정책이 뭐지?"를 코드 없이 확인
- **신규 팀원이** → "이 도메인의 업무 규칙"을 하루 안에 파악
- **CS 담당자가** → "이 에러가 왜 발생하는지" 즉시 검색
- **개발자가** → "이 validation을 왜 이렇게 짰는지" 맥락을 코드 옆에서 확인

### 산출물 우선순위

| 순위 | 산출물 | 핵심 가치 | 주 소비자 |
|---|---|---|---|
| 1 | **서비스 정책서** | "시스템이 이렇게 동작한다"는 규칙 카탈로그 | 기획자, CS, 개발자 |
| 2 | **용어집** | 도메인 용어 통일 — 소통 비용 제거 | 전원 |
| 3 | **기획서** | 기능별 What/Why/How — 신규 팀원 온보딩 | 기획자, 신규 팀원 |
| 4 | **정적 웹사이트** | 위 3개를 검색·탐색 가능한 형태로 통합 | 전원 |

정책서가 1순위인 이유: 코드에서 가장 직접적으로 추출 가능하고, 추출 즉시 가치가 발생한다.

### 프로젝트에 배치하는 파일

```
project-root/
├── CLAUDE.md                              # 기존 + 비즈니스 지식 안내 블록 추가
└── .claude/
    ├── BUSINESS_SPEC.md                   # 비즈니스 문서 형식 정의서
    ├── agents/
    │   ├── domain-analyzer.md             # 기존 (코드 지식 체계)
    │   └── policy-extractor.md            # 신규 (비즈니스 지식 체계)
    └── docs/
        ├── (기존 코드 지식 문서들)
        └── business/                      # ↓ 아래는 모두 프롬프트 실행으로 생성
            ├── POLICY_CATALOG.md          # 전체 정책 카탈로그 (최상위 색인)
            ├── GLOSSARY.md                # 전체 용어집
            ├── domains/
            │   └── {domain-name}/
            │       ├── POLICIES.md        # 도메인별 정책서
            │       ├── SPEC.md            # 도메인별 기획서
            │       └── GLOSSARY.md        # 도메인별 용어집
            └── _site/                     # 정적 웹사이트 빌드 결과
                ├── index.html
                ├── search-index.json
                └── ...
```

---

## 2. 핵심 문서 상세

> 각 문서의 상세 형식 정의는 `.claude/BUSINESS_SPEC.md`에 별도 관리한다.
> 여기서는 각 문서의 역할, 추출 원천, 문서 간 관계를 설명한다.

### 추출 원천 매핑

코드의 어떤 요소에서 어떤 비즈니스 정보가 추출되는가:

| 코드 요소 | 추출 대상 | 비즈니스 문서 |
|---|---|---|
| validation 조건 (if/guard/assert) | 비즈니스 규칙, 제약 조건 | POLICIES.md |
| 에러 메시지 (throw/toast/alert) | 에러 시나리오, 사용자 안내 문구 | POLICIES.md |
| enum / const / 상수 정의 | 허용값 목록, 상태 코드 | POLICIES.md, GLOSSARY.md |
| 주석 (TODO, FIXME, JSDoc, /** */) | 의도, 배경, 제약 사유 | SPEC.md |
| 상태 전이 로직 (ENTITIES.md 참조) | 상태 머신, 전이 조건 | POLICIES.md |
| 설정값 (timeout, limit, threshold) | 시스템 설정 정책 | POLICIES.md |
| API 응답 코드/포맷 | 인터페이스 계약 | SPEC.md |
| 도메인 용어 (변수명, 타입명, 파일명) | 용어 정의 | GLOSSARY.md |

### 문서 간 관계

| 문서 | 역할 | 참조 방향 |
|---|---|---|
| POLICY_CATALOG.md | 전체 정책 색인 — "이 규칙 어디에 있지?" | → 도메인별 POLICIES.md로 이동 |
| POLICIES.md | 도메인별 비즈니스 규칙 카탈로그 | ← FEATURES.md의 기능 ID를 키로 매핑 |
| SPEC.md | 도메인별 기획서 — "이 기능은 왜 이렇게?" | ← POLICIES.md + ENTITIES.md를 입력으로 |
| GLOSSARY.md (전체) | 전체 프로젝트 용어 통합 | ← 도메인별 GLOSSARY.md를 취합 |
| GLOSSARY.md (도메인) | 도메인 내 용어 정의 | ← 코드에서 추출 |
| _site/ | 정적 웹사이트 | ← 모든 business/ 문서를 HTML로 변환 |

### 코드 지식 체계 문서와의 크로스 레퍼런스

| 코드 지식 문서 | 비즈니스 지식 문서 | 연결 방식 |
|---|---|---|
| FEATURES.md의 기능 ID | POLICIES.md의 섹션 헤딩 | 동일 기능 ID 사용 |
| ENTITIES.md의 상태 전이 | POLICIES.md의 상태 정책 | 상태 전이 조건을 비즈니스 언어로 번역 |
| ENTITIES.md의 필드 정의 | GLOSSARY.md의 용어 정의 | 기술 필드명 → 비즈니스 용어 매핑 |
| SERVICE_FLOW.md의 흐름 | SPEC.md의 유저 시나리오 | 기술 흐름 → 사용자 관점 시나리오로 재구성 |

---

## 3. Subagent 설정 및 추천 실행 세팅

### 추천 세팅

| 구분 | 모델 | 이유 |
|---|---|---|
| 메인 에이전트 | Opus / 1M context / effort: high | Phase 3에서 도메인 간 정책 충돌 감지 및 기획서 작성에 추론 품질 필요 |
| subagent | Sonnet | Phase 2는 "코드를 읽고 정해진 형식으로 규칙을 추출하는" 구조화된 작업 |

### policy-extractor subagent 정의

아래 파일을 `.claude/agents/policy-extractor.md`에 저장한다.

```markdown
---
name: policy-extractor
description: |
  도메인 단위 비즈니스 규칙 추출 전용 에이전트.
  메인 에이전트가 도메인 경로와 코드 지식 문서를 전달하면,
  해당 도메인의 비즈니스 규칙/용어를 추출하여
  _temp/{domain}_policies_raw.md 파일로 출력한다.
  "정책 추출", "policy extract", "비즈니스 규칙" 키워드가 포함된 작업에 사용하라.
tools:
  - Read
  - Grep
  - Glob
  - Write
  - Bash(grep:*,find:*,wc:*,head:*,tail:*,cat:*,sed:*)
model: sonnet
---

당신은 비즈니스 규칙 추출 전문 에이전트이다.
메인 에이전트로부터 아래 정보를 전달받는다:

- 분석 대상 도메인명
- 도메인 관련 디렉토리 경로 목록
- 스택 유형 (A: BE 중심 / B: FE 중심 / C: 모노레포)
- BUSINESS_SPEC.md 경로
- 코드 지식 문서 경로:
  - .claude/docs/domains/{domain}/FEATURES.md
  - .claude/docs/domains/{domain}/ENTITIES.md
  - .claude/docs/domains/{domain}/OVERVIEW.md

## 임무

전달받은 도메인의 코드를 탐색하여 비즈니스 규칙을 추출하라.

### 사전 작업: 코드 지식 문서 로드

먼저 해당 도메인의 FEATURES.md와 ENTITIES.md를 읽어라.
여기서 다음을 파악한다:
- 기능 ID 목록 (POLICIES.md의 섹션 구조)
- 코드 위치 (탐색 대상 파일)
- 상태 전이 (상태 정책의 뼈대)
- 엔티티 필드명 (용어집의 후보)

### 추출 대상 및 탐색 규칙

#### 1. Validation 규칙 추출

탐색 패턴 (스택별):
  - TypeScript/JavaScript:
    grep -rn "if.*throw\|if.*return.*null\|if.*return.*false\|\.length\s*[<>=]\|\.every\(\|\.some\(\|\.filter\(\|\.includes\(" {경로}
    grep -rn "z\.string\|z\.number\|z\.enum\|z\.object\|yup\.\|Joi\.\|validate\|isValid" {경로}
  - Spring Boot/Kotlin:
    grep -rn "@Valid\|@NotNull\|@NotBlank\|@Size\|@Min\|@Max\|@Pattern\|require(\|check(\|assert" {경로}

추출 형식:
  - 규칙 ID: POL-{NNN}
  - 조건: {코드에서 확인한 조건}
  - 위반 시 동작: {에러 메시지, throw, UI 변화 등}
  - 코드 근거: {파일:라인}

#### 2. 에러 메시지 추출

탐색 패턴:
  - TypeScript/JavaScript:
    grep -rn "throw new\|new Error\|toast\.\|alert(\|console\.error\|setError\|setErrorMessage\|showError" {경로}
    grep -rn "\".*찾을 수 없\|\".*유효하지\|\".*실패\|\".*불가\|\".*없습니다\|\".*않습니다\|not found\|invalid\|failed\|unauthorized\|forbidden" {경로}
  - Spring Boot/Kotlin:
    grep -rn "throw\|ResponseStatusException\|@ResponseStatus\|ErrorCode\.\|ErrorResponse\|message\s*=" {경로}

추출 형식:
  - 에러 ID: ERR-{NNN}
  - 메시지: {실제 에러 메시지 텍스트}
  - 발생 조건: {어떤 상황에서 이 에러가 나는가}
  - 사용자 영향: {사용자에게 어떻게 보이는가}
  - 코드 근거: {파일:라인}

#### 3. 상수/Enum 추출

탐색 패턴:
  - TypeScript/JavaScript:
    grep -rn "const.*=\s*\[\|const.*=\s*{\|enum\s\|as\s*const\|CARD_VALUES\|TIMEOUT\|LIMIT\|MAX_\|MIN_\|DEFAULT_" {경로}
  - Spring Boot/Kotlin:
    grep -rn "enum class\|companion object\|const val\|object.*{\|@Value(" {경로}

추출 형식:
  - 상수명: {이름}
  - 값: {실제 값}
  - 비즈니스 의미: {이 상수가 서비스에서 어떤 의미를 갖는가}
  - 코드 근거: {파일:라인}

#### 4. 설정값/임계치 추출

탐색 패턴:
  grep -rn "setTimeout\|setInterval\|TIMEOUT\|DELAY\|INTERVAL\|THRESHOLD\|LIMIT\|MAX_\|MIN_\|RETRY\|FALLBACK\|ms\|seconds" {경로}

추출 형식:
  - 설정명: {이름}
  - 값: {숫자 + 단위}
  - 비즈니스 의미: {이 값이 사용자 경험에 미치는 영향}
  - 코드 근거: {파일:라인}

#### 5. 주석에서 의도/배경 추출

탐색 패턴:
  grep -rn "// TODO\|// FIXME\|// HACK\|// NOTE\|// WHY\|// REASON\|/\*\*\|/// " {경로}
  grep -rn "// .*때문\|// .*이유\|// .*위해\|// .*방지\|// .*workaround\|// .*fallback" {경로}

추출 형식:
  - 위치: {파일:라인}
  - 내용: {주석 텍스트}
  - 분류: {설계 의도 / 기술 부채 / 임시 조치 / 비즈니스 배경}

#### 6. 도메인 용어 추출

ENTITIES.md의 필드명, 타입명, 상태값에서 추출한다.
추가로 코드의 함수명/변수명에서 도메인 특화 용어를 식별한다.

추출 형식:
  - 용어: {영문} ({한글})
  - 정의: {이 프로젝트에서의 의미}
  - 유사어/혼동 주의: {다른 용어와의 차이}
  - 사용 위치: {어떤 엔티티/기능에서 등장하는가}

### 산출물 1: _temp/{domain_name}_policies_raw.md (필수)

아래 형식을 정확히 따라라. 이 파일이 메인 에이전트의 Phase 3 입력이 된다.

--- 형식 시작 ---

## {domain_name} 비즈니스 규칙 추출 결과

### 정책 (Validation + 상태 전이 + 설정값)

#### {기능_ID} 관련 정책
- POL-{NNN}: {규칙 요약}
  - 조건: {조건}
  - 위반 시: {동작}
  - 코드 근거: {파일:라인}

### 에러 시나리오
- ERR-{NNN}: "{에러 메시지}"
  - 발생 조건: {조건}
  - 사용자 영향: {설명}
  - 코드 근거: {파일:라인}

### 상수/Enum
- {상수명} = {값}: {비즈니스 의미}
  - 코드 근거: {파일:라인}

### 설정값/임계치
- {설정명} = {값}: {비즈니스 의미}
  - 코드 근거: {파일:라인}

### 주석에서 추출한 의도/배경
- [{분류}] {파일:라인}: {주석 내용 요약}

### 용어
- {영문} ({한글}): {정의}

--- 형식 끝 ---

### 산출물 2: domains/{domain_name}/POLICIES.md
BUSINESS_SPEC.md의 POLICIES 형식을 따라 작성하라.

### 산출물 3: domains/{domain_name}/GLOSSARY.md
BUSINESS_SPEC.md의 GLOSSARY 형식을 따라 작성하라.

## 규칙

- 코드에서 확인되지 않는 정책은 작성하지 마라. 추측 금지.
- 정책 ID(POL-{NNN}), 에러 ID(ERR-{NNN})는 도메인 내에서 고유하게 부여하라.
- 같은 파일을 두 번 읽지 마라. grep 결과를 먼저 수집하고, 필요한 파일만 상세 읽기하라.
- 코드 지식 문서(FEATURES.md)의 기능 ID를 POLICIES.md의 섹션 구조로 사용하라.
- 완료 시 메인 에이전트에게 "✅ {domain_name} 정책 추출 완료. 정책 {N}건, 에러 {M}건, 용어 {K}건" 형태로 보고하라.
```

---

## 4. 주요 프롬프트

### 4-1. 초기 생성 프롬프트

> 전제 조건:
> - 코드 지식 체계가 이미 생성되어 있어야 한다 (.claude/docs/ 하위 문서 존재)
> - `.claude/BUSINESS_SPEC.md`와 `.claude/agents/policy-extractor.md`가 존재해야 한다

```
코드베이스에서 비즈니스 규칙을 추출하여 .claude/docs/business/ 아래에 비즈니스 지식 문서를 생성하라.

## 사전 작업
1. .claude/BUSINESS_SPEC.md 파일을 읽어라.
   이 파일에 각 문서의 정확한 형식, 필수 섹션, 작성 규칙이 정의되어 있다.
2. .claude/agents/policy-extractor.md 가 존재하는지 확인하라.
   없으면 비즈니스 규칙 추출을 직접 수행한다 (subagent 없이 순차 진행).
3. .claude/docs/DOMAIN_MAP.md를 읽어 도메인 목록을 확인하라.
4. 각 도메인의 FEATURES.md와 ENTITIES.md가 존재하는지 확인하라.
   누락된 도메인이 있으면 해당 도메인은 비즈니스 분석에서 제외하고 경고를 출력하라.

## 핵심 원칙: 컨텍스트 절약

- 같은 파일/디렉토리를 두 번 읽지 마라
- 분석 중간 결과는 반드시 파일(.claude/docs/business/_temp/)에 기록하라
- grep 결과를 먼저 수집하고, 필요한 파일만 상세 읽기하라

## 분석 절차

### Phase 1: 입력 검증 및 분석 대상 확정

이 단계는 직접 수행한다 (subagent 사용하지 않음).

1. `.claude/docs/DOMAIN_MAP.md`를 읽어 도메인 목록을 확인하라
2. 각 도메인에 대해 FEATURES.md, ENTITIES.md 존재 여부를 확인하라
3. `.claude/docs/CODE_STRUCTURE.md`에서 각 도메인의 코드 경로를 확인하라
4. 분석 대상을 `.claude/docs/business/_temp/analysis_targets.md`에 기록하라:

--- analysis_targets.md 형식 ---

## 분석 대상 도메인
- {domain_name}:
  - 코드 경로: {경로 목록}
  - FEATURES.md: {존재/누락}
  - ENTITIES.md: {존재/누락}
  - 기능 수: {N}개
  - 엔티티 수: {M}개

## 스택 유형: {A 또는 B 또는 C}

## 제외 도메인 (코드 지식 문서 누락)
- {domain_name}: {누락 사유}

--- 형식 끝 ---

### Phase 2: 도메인별 비즈니스 규칙 추출 (policy-extractor subagent 활용)

`.claude/docs/business/_temp/analysis_targets.md`를 읽고, 각 도메인을 policy-extractor subagent에 위임하라.

각 도메인에 대해 policy-extractor subagent를 호출하라.
호출 시 아래 정보를 프롬프트에 포함하라:

--- subagent 호출 프롬프트 형식 ---

도메인명: {domain_name}
관련 디렉토리:
  - {경로1}
  - {경로2}
스택 유형: {A 또는 B 또는 C}
BUSINESS_SPEC 경로: .claude/BUSINESS_SPEC.md
코드 지식 문서 경로:
  - .claude/docs/domains/{domain_name}/FEATURES.md
  - .claude/docs/domains/{domain_name}/ENTITIES.md
  - .claude/docs/domains/{domain_name}/OVERVIEW.md
출력 경로:
  - .claude/docs/business/_temp/{domain_name}_policies_raw.md
  - .claude/docs/business/domains/{domain_name}/POLICIES.md
  - .claude/docs/business/domains/{domain_name}/GLOSSARY.md

--- 형식 끝 ---

도메인 간 추출은 독립적이므로, 가능한 한 여러 subagent를 동시에 실행하라.
단, 동시 실행 수는 3~4개를 넘지 마라.

subagent가 없으면 각 도메인에 대해 직접 순차 수행하라:
코드 지식 문서 로드 → grep 수집 → _policies_raw.md 기록 → 문서 작성 → 다음 도메인.

모든 도메인의 _policies_raw.md 파일이 생성되었는지 확인하라.
누락된 도메인이 있으면 해당 subagent를 재실행하라.

### Phase 3: 교차 분석 및 통합 문서 생성

이 단계는 직접 수행한다 (subagent 사용하지 않음).
`.claude/docs/business/_temp/` 의 중간 결과 파일들만 읽고 진행하라.

#### Step A: POLICY_CATALOG.md 조립
1. 각 `{domain}_policies_raw.md`의 정책을 취합하라
2. 도메인 간 정책 충돌/중복을 감지하라:
   - 동일한 조건에 대해 다른 도메인에서 다른 규칙이 있는 경우 → [충돌] 마킹
   - 동일한 규칙이 여러 도메인에 중복 존재하는 경우 → [공통 정책]으로 승격
3. `.claude/docs/business/POLICY_CATALOG.md` 작성

#### Step B: GLOSSARY.md 통합
1. 각 도메인의 GLOSSARY.md에서 용어를 취합하라
2. 동일 용어의 정의 불일치를 감지하라 → [정의 불일치] 마킹
3. `.claude/docs/business/GLOSSARY.md` 작성 (전체 프로젝트 용어집)

#### Step C: SPEC.md 생성 (도메인별 기획서)
1. 각 도메인의 POLICIES.md + ENTITIES.md + SERVICE_FLOW.md를 입력으로
2. 기능별 What(무엇을 하는가) / Why(왜 이렇게 동작하는가) / How(어떤 조건에서 어떻게) 를 기술하라
3. 비개발자가 읽을 수 있는 언어로 작성하라 (코드 경로, 기술 용어 최소화)
4. `.claude/docs/business/domains/{domain}/SPEC.md` 작성

#### Step D: 정적 웹사이트 생성
1. 모든 business/ 문서를 HTML로 변환하라
2. 검색 인덱스(search-index.json)를 생성하라
3. `.claude/docs/business/_site/` 에 출력하라
4. 상세 요구사항은 5장 웹사이트 스펙 참조

### Phase 4: 검증 및 정리

1. 교차 검증:
   - [ ] FEATURES.md의 모든 기능 ID가 POLICIES.md에 섹션으로 존재하는가
   - [ ] POLICIES.md의 모든 정책에 코드 근거(파일:라인)가 있는가
   - [ ] ENTITIES.md의 상태 전이가 POLICIES.md의 상태 정책과 일치하는가
   - [ ] GLOSSARY.md에 정의 불일치([정의 불일치] 마킹)가 남아있는가 → 해소하라
   - [ ] SPEC.md에서 참조하는 정책 ID가 POLICIES.md에 존재하는가
   - [ ] 불일치가 있으면 수정하라
2. `.claude/docs/business/_temp/` 디렉토리를 삭제하라
```

### 4-2. 갱신 프롬프트 (기능 추가/수정 후)

```
{도메인명} 도메인에 변경이 발생했다.
코드 변경 사항을 분석하여 아래 비즈니스 문서를 갱신하라:

1. .claude/docs/business/domains/{domain-name}/POLICIES.md — 변경된 기능의 정책 업데이트
2. .claude/docs/business/POLICY_CATALOG.md — 해당 도메인 섹션 갱신
3. .claude/docs/business/domains/{domain-name}/GLOSSARY.md — 새 용어가 추가되었다면 반영
4. .claude/docs/business/GLOSSARY.md — 전체 용어집 갱신
5. .claude/docs/business/domains/{domain-name}/SPEC.md — 기획서 갱신 (해당 시)
6. .claude/docs/business/_site/ — 웹사이트 재빌드

변경 사항: {간략한 변경 설명}

참고: 코드 지식 체계의 갱신이 이미 완료되었는지 확인하라.
코드 지식 문서(FEATURES.md 등)가 먼저 갱신되어야 비즈니스 문서의 정합성이 보장된다.
```

### 4-3. 정책 조회 프롬프트 (개발 중)

```
{도메인}.{기능} 의 비즈니스 규칙을 확인하라.

1. .claude/docs/business/domains/{domain}/POLICIES.md 에서 해당 기능 섹션을 읽어라
2. 관련 정책(POL-xxx)과 에러 시나리오(ERR-xxx)를 나열하라
3. 현재 코드와 정책 문서가 일치하는지 확인하라 (코드 근거 라인 대조)

불일치가 있으면 어느 쪽이 최신인지 판단하고 갱신 방향을 제안하라.
```

---

## 5. 정적 웹사이트 스펙

### 목적

Markdown 문서의 한계(검색 불편, 탐색 구조 부재)를 보완하여, 비개발자도 브라우저에서 비즈니스 규칙을 직접 탐색할 수 있게 한다.

### 기술 스택

- **빌드**: 프로젝트 의존성 없이 독립 실행 가능해야 함
  - 추천: Node.js 스크립트 (Markdown → HTML 변환)
  - 대안: Python (markdown2/jinja2), 또는 Claude Code가 직접 HTML 생성
- **출력**: 순수 정적 파일 (HTML + CSS + JS). 서버 불필요
- **호스팅**: 로컬 file:// 또는 간단한 정적 서버 (npx serve, python -m http.server)

### 페이지 구조

```
index.html                    # 홈 — 도메인 목록 + 전체 검색
├── policy-catalog.html       # 정책 카탈로그 (전체 색인)
├── glossary.html             # 전체 용어집
├── {domain}/
│   ├── index.html            # 도메인 홈 (정책 요약 + 기획서 + 용어)
│   ├── policies.html         # 도메인 정책서
│   ├── spec.html             # 도메인 기획서
│   └── glossary.html         # 도메인 용어집
└── search.html               # 전문 검색 페이지
```

### 검색 요구사항

- **클라이언트 사이드 전문 검색** (서버 없음)
  - 빌드 시 `search-index.json` 생성 (정책 ID, 규칙 텍스트, 에러 메시지, 용어 등 인덱싱)
  - 런타임에 JSON 로드 후 JS로 필터링
  - 추천 라이브러리: lunr.js, fuse.js, 또는 자체 구현 (단순 키워드 매칭이면 충분)
- **검색 대상**: 정책 ID, 규칙 텍스트, 에러 메시지, 용어, 기획서 본문
- **검색 결과**: 해당 항목의 도메인, 기능 ID, 정책/에러 ID, 본문 미리보기, 상세 페이지 링크

### 네비게이션 요구사항

- **좌측 사이드바**: 도메인 목록 → 정책/기획서/용어집 하위 링크
- **브레드크럼**: 현재 위치 표시 (홈 > 도메인 > 정책서 > 기능 ID)
- **앵커 링크**: 각 정책(POL-xxx), 에러(ERR-xxx)에 직접 링크 가능 (URL hash)
- **크로스 레퍼런스**: 정책서에서 용어집으로, 기획서에서 정책서로 하이퍼링크

### 디자인 원칙

- **텍스트 중심**: 장식 최소화, 가독성 우선
- **인쇄 가능**: 각 페이지 print CSS 포함 (필요 시 PDF 출력용)
- **다크모드 지원**: prefers-color-scheme 반응
- **모바일 대응**: 반응형 (단, 데스크톱 우선)

### 빌드 트리거

- 초기 생성 프롬프트(4-1)의 Phase 3 Step D에서 자동 실행
- 갱신 프롬프트(4-2) 실행 시 자동 재빌드
- 수동 빌드 명령: `node .claude/docs/business/_site/build.js` (빌드 스크립트 포함)

---

## 6. 운영 가이드

### 문서 신뢰도 유지 원칙

1. **코드가 진실의 원천**
   - 비즈니스 문서는 코드에서 추출한 파생물. 코드와 문서가 충돌하면 코드가 맞다
   - 단, 코드에 없는 비즈니스 배경(기획 의도, 정책 결정 사유)은 SPEC.md에 수동 보강 가능 — 이 경우 `[수동 추가]` 태그를 붙여라

2. **코드 지식 체계 선행 갱신**
   - 비즈니스 문서 갱신 전에 코드 지식 체계(FEATURES.md 등)가 최신인지 반드시 확인
   - 순서: 코드 변경 → 코드 지식 갱신(4-2) → 비즈니스 지식 갱신(4-2)

3. **점진적 구축**
   - 처음부터 완벽한 문서를 만들려 하지 않는다
   - 1단계: 핵심 도메인의 POLICIES.md (가장 높은 ROI)
   - 2단계: GLOSSARY.md (소통 비용 즉시 감소)
   - 3단계: SPEC.md (온보딩 가치)
   - 4단계: 정적 웹사이트 (탐색 가치)

4. **갱신 습관**
   - 코드 지식 체계 갱신과 함께 비즈니스 지식 갱신을 팀 프로세스에 포함
   - 또는 CLAUDE.md에 "코드 변경 후 비즈니스 문서도 갱신 여부 확인" 규칙 추가

### 기존 작업 원칙 md에 삽입할 블록

아래 코드블록 전체를 기존 작업 원칙 md (또는 CLAUDE.md)에 삽입한다.
코드 지식 체계 블록 아래에 추가하는 것을 권장한다.

```markdown
## 비즈니스 지식 문서

이 프로젝트의 비즈니스 규칙, 서비스 정책, 도메인 용어는 아래 문서에 정리되어 있다.

- 정책 카탈로그: .claude/docs/business/POLICY_CATALOG.md
- 전체 용어집: .claude/docs/business/GLOSSARY.md
- 도메인별 상세: .claude/docs/business/domains/{domain-name}/
  - POLICIES.md — 비즈니스 규칙
  - SPEC.md — 기획서
  - GLOSSARY.md — 도메인 용어집
- 웹사이트: .claude/docs/business/_site/index.html

## 비즈니스 규칙 확인 원칙

### 신규 validation 추가 시

새로운 validation, 에러 처리, 상수, 상태 전이를 추가할 때:
1. 해당 도메인의 POLICIES.md를 확인하여 기존 정책과 충돌하지 않는지 검토하라
2. 추가한 규칙을 POLICIES.md에 반영하라 (POL-{NNN} 형식)
3. 새 에러 메시지를 추가했다면 ERR-{NNN}으로 등록하라

### 금지 사항

- 비즈니스 규칙의 코드 근거 없이 정책 문서에 기록하는 것을 금지한다
- POLICIES.md에 등록하지 않은 validation 로직을 코드에 추가하는 것을 권장하지 않는다 (추가 후 반드시 문서 반영)
```

---

## 7. 실제 추출 예시 (Planning Poker 기준)

코드 지식 체계 실행 결과에서 비즈니스 지식으로 변환되는 예시:

### POLICIES.md 예시 (poker 도메인)

```markdown
## poker.투표

### POL-001: 최소 참가자 수
- **규칙**: 참가자가 2명 이상이어야 투표 공개(reveal)가 가능하다
- **조건**: `participants.length >= 2 && participants.every(p => p.hasVoted)`
- **위반 시**: 대기 화면 유지 (투표 진행 불가)
- **코드 근거**: `app/room/[roomId]/page.tsx` (allVoted 계산 로직)

### POL-002: 투표 카드 옵션
- **규칙**: 피보나치 수열 기반 카드(1, 2, 3, 5, 8, 13, 21) + 불확실(?) + 휴식(☕)
- **조건**: `CARD_VALUES = ['1','2','3','5','8','13','21','?','☕']`
- **비즈니스 배경**: 피보나치 수열은 추정치의 불확실성 증가를 반영. ?와 ☕는 비숫자 옵션으로 Mode/Average 계산에서 제외됨
- **코드 근거**: `components/poker/CardDeck.tsx`

### POL-003: 자동 공개 카운트다운
- **규칙**: 전원 투표 완료 시 2초 카운트다운 후 자동으로 결과를 공개한다
- **설정값**: 2초 (하드코딩)
- **비즈니스 배경**: 즉시 공개하면 마지막 투표자의 영향이 없고, 너무 길면 UX 지연
- **코드 근거**: `app/room/[roomId]/page.tsx` (countdown Effect)
```

### GLOSSARY.md 예시

```markdown
## P

### Participant (참가자)
- **정의**: Planning Poker 세션에 참여 중인 개인. 호스트와 일반 참가자 모두 포함
- **필드**: id(UUID), name(닉네임), hasVoted(투표 여부), vote(공개된 카드값)
- **혼동 주의**: Peer(P2P 연결 단위)와 구분. Participant는 게임 논리, Peer는 통신 논리

### Phase (단계)
- **정의**: 현재 투표 라운드의 상태. `voting`(투표 중) 또는 `revealed`(결과 공개됨)
- **전이**: voting → revealed (전원 투표 완료 + 카운트다운), revealed → voting (재투표 또는 다음 티켓)
```

### SPEC.md 예시

```markdown
## poker.방_생성_위저드

### 이 기능은 무엇인가
Planning Poker 세션을 시작하기 위해 Jira 인증 → 닉네임 설정 → Epic 선택의 3단계를 안내하는 마법사 화면이다.

### 왜 이렇게 동작하는가
- **3단계 분리 이유**: Jira 인증 실패를 즉시 피드백하고(Step 1), 닉네임을 중간에 설정하며(Step 2), 무거운 이슈 목록 로드를 마지막에 수행(Step 3)하여 사용자가 각 단계의 성공/실패를 명확히 인지할 수 있게 한다.
- **Cloud/Server 자동 분기**: email 필드 존재 여부로 Jira Cloud(Basic auth)과 Server·DC(Bearer PAT)를 자동 판별한다. 사용자가 인증 타입을 직접 선택하지 않아도 된다.
- **인증 정보 캐싱**: localStorage에 저장하여 재방문 시 입력을 생략한다. 보안과 편의의 트레이드오프로, 공용 PC에서는 수동 삭제가 필요하다.

### 주요 정책
- POL-004: Jira 인증 필수 — 인증 검증 실패 시 다음 단계 진행 불가
- POL-005: Epic 유효성 검증 — issuetype이 Epic이 아니면 이슈 목록 조회를 차단
- POL-006: 이슈 최대 100건 — maxResults=100으로 하드코딩. 대형 Epic에서 성능 보호
```
