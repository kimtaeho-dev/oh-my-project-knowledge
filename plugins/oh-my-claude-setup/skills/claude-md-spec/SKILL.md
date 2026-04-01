---
name: claude-md-spec
description: |
  CLAUDE.md 파일의 형식 정의. 프로젝트 유형(단일 앱/모노레포/MSA)별 구조와 필수 섹션을 정의한다.
  oh-my-claude-setup:init 명령 실행 시 반드시 이 형식을 따른다.
---

# CLAUDE.md 형식 정의

## 프로젝트 유형 판단 기준

### 모노레포 (하나 이상 해당 시)

- `pnpm-workspace.yaml`, `turbo.json`, `nx.json`, `lerna.json` 존재
- `apps/`, `packages/` 디렉토리 존재 + 각 하위에 독립 빌드 파일 (package.json 등)

### MSA (하나 이상 해당 시)

- `docker-compose.yml` / `docker-compose.yaml` 내 복수의 service 정의
- `services/`, `microservices/` 디렉토리 존재 + 각 하위에 독립 빌드 파일 (pom.xml, build.gradle, requirements.txt, go.mod, package.json 등)
- `k8s/`, `kubernetes/`, `helm/` 디렉토리 존재 + 복수의 Deployment 정의

### 단일 앱

위 기준에 해당하지 않는 경우

---

## 모노레포인 경우

### 루트 CLAUDE.md (최소 구성)

```markdown
# CLAUDE.md

{프로젝트 한 줄 설명} (Turborepo/Nx/Lerna + 패키지매니저 workspace)

## 앱 구성

| 앱 | 포트 | 프레임워크 | UI 라이브러리 | 라우터 |
| -- | ---- | ---------- | ------------- | ------ |
| {앱명} | {포트} | {프레임워크} | {UI 라이브러리} | {라우터} |

앱별 상세는 `apps/[app-name]/CLAUDE.md` 참고.
```

### 앱별 CLAUDE.md (`apps/{app-name}/CLAUDE.md`)

필수 섹션:

1. **앱 설명** — 이 앱이 무엇을 하는 곳인지 한 줄 설명
2. **커맨드** — lint, type-check, test, codegen 등 앱 실행에 필요한 명령어
3. **아키텍처** — API 호출 방식, 상태 관리, 스타일링, 인증, 테스트 등 앱별 기술 결정사항
4. **도메인 → 파일 위치 테이블** — 도메인명과 pages/, components/, queries/ 등 주요 경로를 매핑한 표
5. **주의사항** — 이 앱에서 반드시 알아야 할 특수 규칙, 비활성화된 기능, 알려진 제약

형식:

````markdown
# {app-name}

{앱 한 줄 설명}

## 커맨드

```bash
pnpm lint
pnpm type-check
pnpm gen:openapi
```

## 아키텍처

### API

...

### 상태 관리

...

## 도메인 → 파일 위치

| 도메인 | pages/ | components/pages/ | queries/ |
| ------ | ------ | ----------------- | -------- |
| {기능명} | {경로/} | {경로/} | {useXxx.ts} |

## 주의사항

- 항목1
- 항목2
````

---

## MSA인 경우

### 루트 CLAUDE.md (최소 구성)

```markdown
# CLAUDE.md

{프로젝트 한 줄 설명}

## 서비스 구성

| 서비스 | 포트 | 언어/프레임워크 | 역할 |
| ------ | ---- | --------------- | ---- |
| {서비스명} | {포트} | {언어/프레임워크} | {역할} |

서비스별 상세는 `services/[service-name]/CLAUDE.md` 참고.
```

### 서비스별 CLAUDE.md (`services/{service-name}/CLAUDE.md`)

필수 섹션:

1. **서비스 설명** — 이 서비스가 담당하는 역할 한 줄 설명
2. **커맨드** — 빌드, 테스트, 실행, codegen 등 서비스 운용에 필요한 명령어
3. **아키텍처** — 레이어 구조, DB 연결 방식, 외부 서비스 연동, 인증, 이벤트/메시징 등 기술 결정사항
4. **도메인 → 파일 위치 테이블** — 도메인명과 controller/, service/, repository/ 등 주요 경로를 매핑한 표
5. **주의사항** — 반드시 알아야 할 특수 규칙, 비활성화된 기능, 알려진 제약

형식:

````markdown
# {service-name}

{서비스 한 줄 설명}

## 커맨드

```bash
./gradlew build
./gradlew test
```

## 아키텍처

### 레이어 구조

...

### DB 연결

...

### 외부 연동

...

## 도메인 → 파일 위치

| 도메인 | controller/ | service/ | repository/ |
| ------ | ----------- | -------- | ----------- |
| {기능명} | {경로/} | {경로/} | {경로/} |

## 주의사항

- 항목1
- 항목2
````

---

## 단일 앱인 경우

루트 CLAUDE.md 하나에 전체 내용을 포함한다.

필수 섹션:

1. **프로젝트 설명** — 한 줄 설명
2. **커맨드** — lint, type-check, test, dev, build 등 주요 명령어
3. **아키텍처** — API 호출 방식, 상태 관리, 스타일링, 인증, 테스트 등 기술 결정사항
4. **도메인 → 파일 위치 테이블** — 도메인명과 주요 경로를 매핑한 표
5. **주의사항** — 반드시 알아야 할 특수 규칙, 비활성화된 기능, 알려진 제약

형식:

````markdown
# CLAUDE.md

{프로젝트 한 줄 설명}

## 커맨드

```bash
npm run dev
npm run lint
npm run test
```

## 아키텍처

### API

...

### 상태 관리

...

## 도메인 → 파일 위치

| 도메인 | 경로 | 설명 |
| ------ | ---- | ---- |
| {기능명} | {경로/} | {설명} |

## 주의사항

- 항목1
- 항목2
````

---

## 작성 규칙

- CLAUDE.md는 한국어로 작성한다
- 세부 원칙 및 규칙은 CLAUDE.md에 포함하지 않는다 (.claude/rules/principles/ 경로에 별도 파일로 분리)
- 테이블의 각 열은 코드에서 실제 확인된 값만 기재한다. 추측하지 않는다
- 도메인 → 파일 위치 테이블은 Claude가 기능 수정 시 파일을 바로 찾을 수 있도록 구체적으로 작성한다
- 커맨드 섹션은 package.json, Makefile, build.gradle 등에서 실제 확인된 명령어만 기재한다
