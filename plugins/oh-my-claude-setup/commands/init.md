프로젝트에 이상적인 Claude Code 초기 환경을 구성한다. CLAUDE.md, 원칙 파일, 이슈 트래킹 파일을 자동 생성한다.

## 사전 작업

1. 이 플러그인의 스킬을 확인한다:
   - **claude-md-spec**: CLAUDE.md 형식 정의 (프로젝트 유형별)
   - **principles-spec**: 원칙 파일 내용 정의
2. 대상 프로젝트의 루트 디렉토리에서 실행되고 있는지 확인한다.

## 핵심 원칙

- CLAUDE.md는 한국어로 작성한다
- 코드에서 실제 확인된 정보만 기재한다. 추측하지 않는다
- 기존 CLAUDE.md, .claude/rules/ 파일이 있으면 내용을 통합하여 중복 없이 정리한다

---

## 1단계: 프로젝트 분석 및 CLAUDE.md 생성

### Step 1: 프로젝트 유형 판단

아래 파일/디렉토리 존재 여부를 확인하여 프로젝트 유형을 판단한다.

**모노레포 판단** (하나 이상 해당 시):
- `pnpm-workspace.yaml`, `turbo.json`, `nx.json`, `lerna.json` 존재
- `apps/`, `packages/` 디렉토리 존재 + 각 하위에 독립 빌드 파일

**MSA 판단** (하나 이상 해당 시):
- `docker-compose.yml` / `docker-compose.yaml` 내 복수의 service 정의
- `services/`, `microservices/` 디렉토리 존재 + 각 하위에 독립 빌드 파일
- `k8s/`, `kubernetes/`, `helm/` 디렉토리 존재 + 복수의 Deployment 정의

**단일 앱**: 위 기준에 해당하지 않는 경우

### Step 2: 프로젝트 정보 수집

1. `tree -L 3 -d --charset=ascii`로 디렉토리 구조를 파악한다
2. 빌드 파일(package.json, build.gradle, pom.xml, requirements.txt, go.mod 등)에서 기술 스택과 버전을 확인한다
3. 실행 스크립트(scripts 섹션, Makefile 등)에서 커맨드를 수집한다
4. 소스 코드 디렉토리 구조에서 도메인 경계를 식별한다
5. .env.example 또는 .env.sample에서 환경변수를 수집한다

### Step 3: CLAUDE.md 생성

**claude-md-spec** 스킬의 형식을 따라 프로젝트 유형에 맞는 CLAUDE.md를 생성한다.

- **모노레포**: 루트 CLAUDE.md (최소 구성) + 각 `apps/{app-name}/CLAUDE.md`
- **MSA**: 루트 CLAUDE.md (최소 구성) + 각 `services/{service-name}/CLAUDE.md`
- **단일 앱**: 루트 CLAUDE.md에 전체 내용 포함

도메인 → 파일 위치 테이블은 실제 디렉토리를 탐색하여 구체적으로 작성한다.

---

## 2단계: 원칙 파일 구성

`.claude/rules/principles/` 디렉토리를 생성하고, **principles-spec** 스킬에 정의된 각 파일을 생성한다.

생성할 파일 목록:
1. `workflow-start.md` — 작업 시작 원칙
2. `code-modification.md` — 코드 수정 원칙
3. `git-workflow.md` — Git Workflow
4. `work-history.md` — 작업 히스토리
5. `self-improvement.md` — 자기개선
6. `lessons.md` — Lessons Learned
7. `command-principles.md` — 명령어 작성 원칙

기존 파일이 존재하면 내용이 중복되는 항목은 통합하여 중복 없이 정리한다.

---

## 3단계: known-issues.md 생성

1. `.claude/tasks/` 디렉토리가 없으면 생성한다
2. 프로젝트 전체에서 TODO, FIXME 주석을 검색한다
3. 수집 결과를 아래 형식으로 `.claude/tasks/known-issues.md`에 기록한다:

```markdown
# Known Issues

## TODO

[TODO] {제목}
- 파일: {파일경로}:{라인번호}
- 영향 범위: {영향을 받는 기능 또는 모듈}
- 상태: 미착수

## FIXME

[FIXME] {제목}
- 파일: {파일경로}:{라인번호}
- 영향 범위: {영향을 받는 기능 또는 모듈}
- 상태: 미착수
```

TODO/FIXME가 없으면 빈 템플릿만 생성한다.

---

## 4단계: suspected-bugs.md 생성

1. 프로젝트 전체에서 에러로 의심되는 패턴을 검색한다:
   - catch 블록에서 에러를 무시하는 코드 (빈 catch 블록)
   - 사용되지 않는 변수/import (린터가 잡지 못한 것)
   - 하드코딩된 값 (매직 넘버, 하드코딩 URL 등)
   - 잠재적 null/undefined 접근
2. 발견된 항목을 아래 형식으로 `.claude/tasks/suspected-bugs.md`에 기록한다:

```markdown
# Suspected Bugs

[버그 의심] {제목}
- 파일: {파일경로}:{라인번호}
- 의심 이유: {간략한 설명}
- 영향 범위: {영향을 받는 기능 또는 모듈}
- 상태: 미착수
```

의심 항목이 없으면 빈 템플릿만 생성한다.

---

## 5단계: 완료 보고

모든 구성이 완료되면 아래 체크리스트로 결과를 보고한다:

```
## 구성 완료 보고

### CLAUDE.md
- [ ] 프로젝트 유형: {단일 앱 / 모노레포 / MSA}
- [ ] 루트 CLAUDE.md 생성 완료
- [ ] (모노레포) 앱별 CLAUDE.md 생성: {생성된 앱 목록}
- [ ] (MSA) 서비스별 CLAUDE.md 생성: {생성된 서비스 목록}

### 원칙 파일 (.claude/rules/principles/)
- [ ] workflow-start.md
- [ ] code-modification.md
- [ ] git-workflow.md
- [ ] work-history.md
- [ ] self-improvement.md
- [ ] lessons.md
- [ ] command-principles.md

### 이슈 트래킹
- [ ] .claude/tasks/known-issues.md (TODO: {n}건, FIXME: {n}건)
- [ ] .claude/tasks/suspected-bugs.md ({n}건)

### 특이사항
- {분석 중 발견된 주의가 필요한 영역}
```
