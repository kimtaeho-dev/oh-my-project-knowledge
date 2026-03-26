# Claude Code 비즈니스 지식 체계 (Business Knowledge System)

---

## 1. 개요

### 해결하고자 하는 문제

코드에는 비즈니스 규칙이 묻혀 있다. validation 조건, 에러 메시지, 상수, enum, 상태 전이 로직, 주석 — 이것들이 곧 서비스 정책이지만, 코드를 직접 읽지 않으면 알 수 없다.

기획자가 "최소 참가자 수가 몇 명이지?"라고 물으면 개발자가 코드를 뒤져야 하고, 신규 팀원이 "이 에러는 언제 발생하지?"를 파악하려면 소스 코드를 추적해야 한다.

이 문서는 **코드 지식 체계**(project-knowledge-system.md)의 산출물을 입력으로 받아, 코드에 내재된 비즈니스 규칙을 체계적으로 추출하고, **"시스템이 이렇게 동작한다"는 관점**으로 정리하는 체계를 정의한다.

### 코드 지식 체계와의 관계

```
┌─────────────────────────────┐
│  코드 지식 체계 (완성)        │
│  DOMAIN_MAP / FEATURES /     │
│  ENTITIES / SERVICE_FLOW     │
└──────────┬──────────────────┘
           │ 입력 (기능 ID가 연결 고리)
           ▼
┌─────────────────────────────┐
│  비즈니스 지식 체계 (본 문서)  │
│  코드에서 비즈니스 규칙 추출   │
│  → 정책서 / 기획서 / 용어집   │
│  → 정적 웹사이트로 출력       │
└─────────────────────────────┘
```

코드 지식 체계가 "어디에 영향이 가는가"(구조)를 다루고, 비즈니스 지식 체계는 "시스템이 어떻게 동작하는가"(규칙)를 다룬다.

### 핵심 원칙: 최종 문서에 코드 표현을 넣지 않는다

비즈니스 지식 체계의 최종 산출물(POLICIES.md, SPEC.md, GLOSSARY.md, 웹사이트)에는 **파일 경로, 라인 번호, 변수명, 코드 스니펫을 포함하지 않는다.**

- 코드 위치 추적은 코드 지식 체계(FEATURES.md)의 기능 ID를 통해 간접 참조한다
- 추출 과정에서 사용하는 코드 근거는 _temp/ 중간 파일에만 존재하고, 최종 문서 생성 후 삭제한다
- 최종 문서의 서술 관점은 "코드가 이렇게 되어 있다"가 아니라 "시스템이 이렇게 동작한다"이다

### 핵심 목표

- **기획자가** → "이 기능의 정책이 뭐지?"를 코드 없이 확인
- **신규 팀원이** → "이 도메인의 업무 규칙"을 하루 안에 파악
- **개발자가** → 기능 ID로 코드 지식 체계(FEATURES.md)와 비즈니스 지식 체계(POLICIES.md)를 오가며 맥락 파악

### 산출물 우선순위

| 순위 | 산출물 | 핵심 가치 | 주 소비자 |
|---|---|---|---|
| 1 | **서비스 정책서** | "시스템이 이렇게 동작한다"는 규칙 카탈로그 | 기획자, 개발자, 신규 팀원 |
| 2 | **용어집** | 도메인 용어 통일 — 소통 비용 제거 | 전원 |
| 3 | **기획서** | 기능별 What/Why — 신규 팀원 온보딩 | 기획자, 신규 팀원 |
| 4 | **정적 웹사이트** | 위 3개를 검색·탐색 가능한 형태로 통합 | 전원 |

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

### 기능 ID 기반 연결 (코드 지식 ↔ 비즈니스 지식)

코드 근거를 최종 문서에 넣지 않는 대신, **기능 ID가 두 체계의 연결 고리** 역할을 한다.

```
FEATURES.md                          POLICIES.md
┌────────────────────────┐          ┌────────────────────────┐
│ ## poker.투표           │ ◄──────► │ ## poker.투표           │
│ - 코드 위치: ...        │  기능 ID  │ - 규칙: 2명 이상 ...    │
│ - 트리거: ...           │  공유     │ - 위반 시: ...          │
│ - 영향 범위: ...        │          │ - 에러: ...             │
└────────────────────────┘          └────────────────────────┘
     (개발자용)                          (전원 공유)
```

갱신 시 흐름:
1. 코드 변경 → 코드 지식 체계의 FEATURES.md 갱신 (기능 ID 단위)
2. 변경된 기능 ID를 기준으로 → 비즈니스 지식 체계의 POLICIES.md 해당 섹션만 재추출
3. 기능 ID가 바뀌지 않은 섹션은 건드리지 않음 (수동 보강 내용 보호)

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

### 핵심 원칙

- **최종 산출물(POLICIES.md, GLOSSARY.md)에 코드 경로, 변수명, 라인 번호를 넣지 마라**
- 코드 근거는 _temp/{domain}_policies_raw.md에만 기록한다
- 최종 문서의 서술은 "시스템이 이렇게 동작한다"는 관점으로 작성하라
- 기능 ID(`{app}.{기능명_스네이크}`)는 유일한 연결 고리이다. FEATURES.md와 동일한 기능 ID를 사용하라

### 사전 작업: 코드 지식 문서 로드

먼저 해당 도메인의 FEATURES.md와 ENTITIES.md를 읽어라.
여기서 다음을 파악한다:
- 기능 ID 목록 (POLICIES.md의 섹션 구조)
- 코드 위치 (탐색 대상 파일 — 추출에만 사용, 최종 문서에 넣지 않음)
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

_raw.md 기록: 코드 조건 원문 + 비즈니스 번역 + 코드 근거
최종 문서: 비즈니스 번역만 사용, 코드 근거 제거

#### 2. 에러 메시지 추출

탐색 패턴:
  - TypeScript/JavaScript:
    grep -rn "throw new\|new Error\|toast\.\|alert(\|console\.error\|setError\|setErrorMessage\|showError" {경로}
    grep -rn "\".*찾을 수 없\|\".*유효하지\|\".*실패\|\".*불가\|\".*없습니다\|\".*않습니다\|not found\|invalid\|failed\|unauthorized\|forbidden" {경로}
  - Spring Boot/Kotlin:
    grep -rn "throw\|ResponseStatusException\|@ResponseStatus\|ErrorCode\.\|ErrorResponse\|message\s*=" {경로}

#### 3. 상수/Enum 추출

탐색 패턴:
  - TypeScript/JavaScript:
    grep -rn "const.*=\s*\[\|const.*=\s*{\|enum\s\|as\s*const\|TIMEOUT\|LIMIT\|MAX_\|MIN_\|DEFAULT_" {경로}
  - Spring Boot/Kotlin:
    grep -rn "enum class\|companion object\|const val\|object.*{\|@Value(" {경로}

#### 4. 설정값/임계치 추출

탐색 패턴:
  grep -rn "setTimeout\|setInterval\|TIMEOUT\|DELAY\|INTERVAL\|THRESHOLD\|LIMIT\|MAX_\|MIN_\|RETRY\|FALLBACK\|ms\|seconds" {경로}

#### 5. 주석에서 의도/배경 추출

탐색 패턴:
  grep -rn "// TODO\|// FIXME\|// HACK\|// NOTE\|// WHY\|// REASON\|/\*\*\|/// " {경로}
  grep -rn "// .*때문\|// .*이유\|// .*위해\|// .*방지\|// .*workaround\|// .*fallback" {경로}

#### 6. 도메인 용어 추출

ENTITIES.md의 필드명, 타입명, 상태값에서 추출한다.
최종 문서에는 비즈니스 용어와 정의만 기재. 코드 변수명/필드명은 넣지 않는다.

### 산출물 1: _temp/{domain_name}_policies_raw.md (임시)

코드 근거를 포함하는 중간 결과물. 최종 문서 생성 후 Phase 4에서 삭제된다.

--- 형식 시작 ---

## {domain_name} 비즈니스 규칙 추출 결과

### 정책

#### {기능_ID} 관련 정책
- POL-{NNN}: {규칙 요약}
  - 조건 (코드 원문): {코드 조건식}
  - 조건 (비즈니스 번역): {비즈니스 언어 번역}
  - 위반 시: {동작}
  - 코드 근거: {파일:라인}

### 에러 시나리오
- ERR-{NNN}: "{에러 메시지}"
  - 발생 조건: {비즈니스 언어}
  - 사용자 영향: {설명}
  - 대응 방법: {사용자 행동 가이드}
  - 코드 근거: {파일:라인}

### 상수/설정값
- {비즈니스 의미}: {값}
  - 코드 근거: {파일:라인}

### 용어
- {한글 용어}: {정의}

--- 형식 끝 ---

### 산출물 2: domains/{domain_name}/POLICIES.md (최종 — 코드 표현 없음)
BUSINESS_SPEC.md의 POLICIES 형식을 따라 작성하라.
_raw.md의 "조건 (비즈니스 번역)"을 사용하고, 코드 근거는 포함하지 마라.

### 산출물 3: domains/{domain_name}/GLOSSARY.md (최종 — 코드 표현 없음)
BUSINESS_SPEC.md의 GLOSSARY 형식을 따라 작성하라.

## 규칙

- 코드에서 확인되지 않는 정책은 작성하지 마라. 추측 금지.
- 정책 ID(POL-{NNN}), 에러 ID(ERR-{NNN})는 도메인 내에서 고유하게 부여하라.
- 같은 파일을 두 번 읽지 마라. grep 결과를 먼저 수집하고, 필요한 파일만 상세 읽기하라.
- 코드 지식 문서(FEATURES.md)의 기능 ID를 POLICIES.md의 섹션 구조로 사용하라.
- **최종 문서(POLICIES.md, GLOSSARY.md)에 파일 경로, 라인 번호, 변수명을 포함하지 마라.**
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
2. .claude/agents/policy-extractor.md 가 존재하는지 확인하라.
   없으면 비즈니스 규칙 추출을 직접 수행한다 (subagent 없이 순차 진행).
3. .claude/docs/DOMAIN_MAP.md를 읽어 도메인 목록을 확인하라.
4. 각 도메인의 FEATURES.md와 ENTITIES.md가 존재하는지 확인하라.
   누락된 도메인이 있으면 해당 도메인은 비즈니스 분석에서 제외하고 경고를 출력하라.

## 핵심 원칙

- 같은 파일/디렉토리를 두 번 읽지 마라
- 분석 중간 결과는 반드시 파일(.claude/docs/business/_temp/)에 기록하라
- **최종 문서에 코드 경로, 변수명, 라인 번호를 넣지 마라**

## 분석 절차

### Phase 1: 입력 검증 및 분석 대상 확정

이 단계는 직접 수행한다 (subagent 사용하지 않음).

1. `.claude/docs/DOMAIN_MAP.md`를 읽어 도메인 목록을 확인하라
2. 각 도메인에 대해 FEATURES.md, ENTITIES.md 존재 여부를 확인하라
3. `.claude/docs/CODE_STRUCTURE.md`에서 각 도메인의 코드 경로를 확인하라
4. 분석 대상을 `.claude/docs/business/_temp/analysis_targets.md`에 기록하라

### Phase 2: 도메인별 비즈니스 규칙 추출 (policy-extractor subagent 활용)

각 도메인을 policy-extractor subagent에 위임하라.
동시 실행 수는 3~4개를 넘지 마라.
subagent가 없으면 직접 순차 수행하라.

### Phase 3: 교차 분석 및 통합 문서 생성

이 단계는 직접 수행한다 (subagent 사용하지 않음).

#### Step A: POLICY_CATALOG.md 조립
1. 각 도메인 POLICIES.md의 정책을 취합하라
2. 도메인 간 정책 충돌/중복을 감지하라
3. `.claude/docs/business/POLICY_CATALOG.md` 작성

#### Step B: GLOSSARY.md 통합
1. 각 도메인의 GLOSSARY.md에서 용어를 취합하라
2. 동일 용어의 정의 불일치를 감지하라
3. `.claude/docs/business/GLOSSARY.md` 작성

#### Step C: SPEC.md 생성 (도메인별 기획서)
1. 각 도메인의 POLICIES.md + OVERVIEW.md + SERVICE_FLOW.md를 입력으로
2. 기능별 What / Why 를 비개발자 관점으로 기술하라
3. `.claude/docs/business/domains/{domain}/SPEC.md` 작성

#### Step D: 정적 웹사이트 생성
1. 모든 business/ 문서를 HTML로 변환하라
2. 검색 인덱스(search-index.json)를 생성하라
3. `.claude/docs/business/_site/` 에 출력하라

### Phase 4: 검증 및 정리

1. 교차 검증:
   - [ ] FEATURES.md의 모든 기능 ID가 POLICIES.md에 섹션으로 존재하는가
   - [ ] ENTITIES.md의 상태 전이가 POLICIES.md의 상태 정책과 일치하는가
   - [ ] GLOSSARY.md에 정의 불일치가 남아있는가 → 해소하라
   - [ ] SPEC.md에서 참조하는 정책 ID가 POLICIES.md에 존재하는가
   - [ ] **최종 문서에 코드 경로, 변수명, 라인 번호가 포함되어 있지 않은가**
2. `.claude/docs/business/_temp/` 디렉토리를 삭제하라
```

### 4-2. 갱신 프롬프트 (기능 추가/수정 후)

```
{도메인명} 도메인에 변경이 발생했다.
코드 변경 사항을 분석하여 비즈니스 문서를 갱신하라.

## 전제 조건 확인
코드 지식 체계의 갱신이 이미 완료되었는지 확인하라.
.claude/docs/domains/{domain-name}/FEATURES.md가 최신 상태여야 한다.

## 갱신 범위 판단
1. FEATURES.md에서 변경된 기능 ID를 식별하라
2. 해당 기능 ID에 대응하는 POLICIES.md 섹션만 재추출 대상으로 삼아라
3. 해당 기능 ID의 코드 위치(FEATURES.md에 기재된 경로)를 탐색하여 정책을 재추출하라

## 수동 보강 보호 규칙
- POLICIES.md 또는 SPEC.md에 `[수동 추가]` 태그가 붙은 내용은 덮어쓰지 마라
- 재추출 대상 섹션에 `[수동 추가]` 내용이 있으면 보존한 채로 자동 추출 부분만 갱신하라
- 자동 추출 내용과 수동 추가 내용이 충돌하면 [충돌 — 확인 필요] 마킹 후 양쪽을 모두 남겨라

## 갱신 대상 문서
1. .claude/docs/business/domains/{domain-name}/POLICIES.md — 변경된 기능 ID 섹션 재추출
2. .claude/docs/business/POLICY_CATALOG.md — 해당 도메인 섹션 갱신
3. .claude/docs/business/domains/{domain-name}/GLOSSARY.md — 새 용어 반영
4. .claude/docs/business/GLOSSARY.md — 전체 용어집 갱신
5. .claude/docs/business/domains/{domain-name}/SPEC.md — 기획서 갱신 (해당 시)
6. .claude/docs/business/_site/ — 웹사이트 재빌드

변경 사항: {간략한 변경 설명}
```

### 4-3. 정책 조회 프롬프트 (개발 중)

```
{도메인}.{기능} 의 비즈니스 규칙을 확인하라.

1. .claude/docs/business/domains/{domain}/POLICIES.md 에서 해당 기능 ID 섹션을 읽어라
2. 관련 정책(POL-xxx)과 에러 시나리오(ERR-xxx)를 나열하라
3. 해당 기능 ID의 코드(FEATURES.md의 코드 위치 참조)를 현재 코드와 비교하여
   정책 문서의 내용이 여전히 유효한지 확인하라

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
  - 추천 라이브러리: lunr.js, fuse.js, 또는 자체 구현
- **검색 대상**: 정책 ID, 규칙 텍스트, 에러 메시지, 용어, 기획서 본문
- **검색 결과**: 도메인, 기능, 정책/에러 ID, 본문 미리보기, 상세 페이지 링크

### 네비게이션 요구사항

- **좌측 사이드바**: 도메인 목록 → 정책/기획서/용어집 하위 링크
- **브레드크럼**: 현재 위치 표시 (홈 > 도메인 > 정책서 > 기능명)
- **앵커 링크**: 각 정책(POL-xxx), 에러(ERR-xxx)에 직접 링크 가능 (URL hash)
- **크로스 레퍼런스**: 정책서에서 용어집으로, 기획서에서 정책서로 하이퍼링크

### 디자인 원칙

- **텍스트 중심**: 장식 최소화, 가독성 우선
- **인쇄 가능**: 각 페이지 print CSS 포함
- **다크모드 지원**: prefers-color-scheme 반응
- **모바일 대응**: 반응형 (데스크톱 우선)

### 빌드 트리거

- 초기 생성 프롬프트(4-1)의 Phase 3 Step D에서 자동 실행
- 갱신 프롬프트(4-2) 실행 시 자동 재빌드
- 수동 빌드: `node .claude/docs/business/_site/build.js`

---

## 6. 운영 가이드

### 문서 신뢰도 유지 원칙

1. **코드가 진실의 원천**
   - 비즈니스 문서는 코드에서 추출한 파생물. 코드와 문서가 충돌하면 코드가 맞다
   - 단, `[수동 추가]` 태그가 붙은 내용은 사람이 의도적으로 추가한 것이므로 자동 갱신으로 삭제하지 않는다

2. **코드 지식 체계 선행 갱신**
   - 순서: 코드 변경 → 코드 지식 갱신 → 비즈니스 지식 갱신
   - 기능 ID가 이 두 단계를 연결하는 고리이다

3. **수동 보강의 원칙**
   - 코드에서 추출 불가능한 비즈니스 배경은 `[수동 추가]` 태그를 붙여 직접 기록할 수 있다
   - `[수동 추가]` 내용은 자동 갱신 시 보존된다

4. **점진적 구축**
   - 1단계: 핵심 도메인의 POLICIES.md (가장 높은 ROI)
   - 2단계: GLOSSARY.md (소통 비용 즉시 감소)
   - 3단계: SPEC.md (온보딩 가치)
   - 4단계: 정적 웹사이트 (탐색 가치)

### 기존 작업 원칙 md에 삽입할 블록

아래 코드블록 전체를 기존 작업 원칙 md (또는 CLAUDE.md)에 삽입한다.

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

### 수동 보강

- 코드에서 추출 불가능한 비즈니스 배경은 `[수동 추가]` 태그를 붙여 직접 기록할 수 있다
- `[수동 추가]` 내용은 자동 갱신 시 보존된다
```

---

## 7. 실제 추출 예시 (Planning Poker 기준)

코드 지식 체계 실행 결과에서 비즈니스 지식으로 변환되는 예시:

### POLICIES.md 예시 (poker 도메인)

```markdown
## poker.투표

### POL-001: 최소 참가자 수
- **규칙**: 참가자가 2명 이상 모이고 전원이 카드를 선택해야 결과가 공개된다
- **위반 시**: 대기 화면이 유지되며 투표를 진행할 수 없다

### POL-002: 투표 카드 옵션
- **규칙**: 피보나치 수열 기반 카드(1, 2, 3, 5, 8, 13, 21)와 특수 옵션 불확실(?), 휴식(☕)을 선택할 수 있다
- **비즈니스 배경**: 피보나치 수열은 작업 규모가 커질수록 추정 불확실성이 증가하는 특성을 반영한다. ?와 ☕는 숫자가 아니므로 평균/최빈값 계산에서 제외된다

### POL-003: 자동 공개 카운트다운
- **규칙**: 전원 투표 완료 시 2초 카운트다운 후 자동으로 결과가 공개된다
- **비즈니스 배경**: 즉시 공개하면 마지막 투표자에게 부담이 없고, 너무 길면 흐름이 끊긴다

### ERR-001: 방을 찾을 수 없음
- **메시지**: "방을 찾을 수 없습니다"
- **발생 상황**: 초대 링크를 클릭했지만 해당 방이 이미 종료되었거나 존재하지 않을 때
- **대응 방법**: 홈 화면으로 이동하여 새 방을 생성하거나 호스트에게 새 링크를 요청한다
```

### GLOSSARY.md 예시

```markdown
## ㅊ

### 참가자
- **정의**: Planning Poker 세션에 참여 중인 사람. 방을 만든 호스트와 초대받아 들어온 일반 참가자 모두 포함한다
- **혼동 주의**: "피어(Peer)"는 기술적 통신 연결 단위를 가리키는 용어로, 참가자와 1:1 대응하지만 의미가 다르다

## ㄷ

### 단계 (Phase)
- **정의**: 현재 투표 라운드의 상태. "투표 중"과 "결과 공개됨" 두 가지가 있다
- **전이**: 전원 투표 완료 후 카운트다운이 끝나면 "투표 중" → "결과 공개됨"으로 전환된다. 재투표 또는 다음 티켓으로 넘어가면 "투표 중"으로 복귀한다
```

### SPEC.md 예시

```markdown
## 방 생성 마법사

### 이 기능은 무엇인가
Planning Poker 세션을 시작하기 위한 3단계 설정 화면이다. Jira 인증 → 닉네임 설정 → Epic 선택 순서로 진행한다.

### 사용자 시나리오
1. 사용자가 Jira 접속 정보(도메인, 토큰)를 입력한다
2. 시스템이 Jira에 접속하여 인증을 확인한다
3. 사용자가 닉네임을 설정한다
4. 사용자가 추정할 Epic의 키를 입력한다
5. 시스템이 해당 Epic의 하위 이슈 목록을 불러온다
6. 사용자가 "방 만들기"를 누르면 세션이 시작되고 초대 링크를 공유할 수 있는 대기 화면으로 이동한다

### 왜 이렇게 동작하는가
- **3단계 분리 이유**: Jira 인증 실패를 첫 단계에서 즉시 알려주고, 무거운 이슈 목록 로드는 마지막에 수행하여 각 단계의 성공/실패를 명확히 알 수 있게 한다
- **Jira Cloud/Server 자동 판별**: 이메일 입력 여부로 Cloud와 Server를 구분하므로, 사용자가 인증 타입을 직접 선택할 필요가 없다
- **인증 정보 저장**: 브라우저에 저장하여 재방문 시 입력을 생략한다. 공용 PC에서는 수동 삭제가 필요하다 [수동 추가]

### 주요 정책
- POL-004: Jira 인증 필수 — 인증 실패 시 다음 단계로 진행할 수 없다
- POL-005: Epic 유효성 검증 — Epic이 아닌 이슈를 입력하면 목록 조회가 차단된다
- POL-006: 이슈 최대 100건 — 한 번에 최대 100개의 이슈만 불러온다
```