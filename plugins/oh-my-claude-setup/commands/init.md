프로젝트에 이상적인 Claude Code 초기 환경을 구성한다. CLAUDE.md, 원칙 파일, 이슈 트래킹 파일, hooks를 자동 생성한다.

## 사전 작업

1. 이 플러그인의 스킬을 확인한다:
   - **claude-md-spec**: CLAUDE.md 형식 정의 (프로젝트 유형별)
   - **principles-spec**: 원칙 파일 내용 정의
   - **hooks-spec**: hook 스크립트 템플릿 및 스택별 카탈로그
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

**4단계(hooks)에서 재사용하기 위해 아래 정보를 추가로 수집한다:**
6. 패키지 매니저 종류 (pnpm-lock.yaml / yarn.lock / package-lock.json / go.mod / Cargo.toml / build.gradle.kts)
7. 포매터 설정 파일 존재 여부 (.prettierrc, biome.json, gofmt, rustfmt.toml 등)
8. 린터 설정 파일 존재 여부 (.eslintrc*, eslint.config.*, biome.json, ruff.toml 등)
9. 타입 체커 설정 파일 존재 여부 (tsconfig.json, pyrightconfig.json, mypy 설정 등)
10. Git 브랜치 구조 (main, develop 존재 여부로 Git 전략 추론)

### Step 3: CLAUDE.md 생성

**claude-md-spec** 스킬의 형식을 따라 프로젝트 유형에 맞는 CLAUDE.md를 생성한다.

- **모노레포**: 루트 CLAUDE.md (최소 구성) + 각 `apps/{app-name}/CLAUDE.md`
- **MSA**: 루트 CLAUDE.md (최소 구성) + 각 `services/{service-name}/CLAUDE.md`
- **단일 앱**: 루트 CLAUDE.md에 전체 내용 포함

도메인 → 파일 위치 테이블은 CLAUDE.md에 직접 포함하지 않고 별도 파일 `.claude/DOMAIN_MAP.md`로 분리한다. CLAUDE.md에는 "프로젝트 도메인 맵: `.claude/DOMAIN_MAP.md` 참조" 한 줄만 둔다. DOMAIN_MAP.md의 작성 규칙은 "1.5단계: 도메인 맵 생성"을 따른다.

---

## 1.5단계: 도메인 맵 생성

`.claude/DOMAIN_MAP.md` 파일을 생성한다. 이 파일의 목적:

- Superpowers의 brainstorming 스킬이 "어디를 건드릴지" 빠르게 좁힐 수 있도록 함
- VOC/문의 대응 시 자연어 증상에서 의심 코드 영역으로 역추적할 수 있도록 함

### 파일 형식

```markdown
# 도메인 맵

| 기능 | 파일 위치 | 증상 키워드 |
|---|---|---|
| 적립금 조회 (백오피스) | src/payment/point/ | 적립금, 포인트 안 보임 |
| 프로필 수정 | src/mypage/profile/ | 프로필, 닉네임 변경 |
| 결제 완료 처리 | src/payment/checkout/ | 결제 안 됨, 주문 실패 |
```

### 작성 규칙

- **기능 열**: 자유 한국어. 모호하면 괄호로 앱/맥락 보충. 표준 식별자 컨벤션은 사용하지 않는다.
- **파일 위치 열**: 디렉토리 경로 또는 핵심 파일 1-2개. 디렉토리 끝에 `/` 표기.
- **증상 키워드 열**: 사용자가 VOC/문의에서 실제로 쓸 만한 단어 5개 이내. 코드 식별자가 아니라 사용자 언어로 작성.

### 작성 절차

1. 1단계 Step 2에서 파악한 디렉토리 구조와 도메인 경계를 기반으로 후보 기능 목록을 만든다
2. 각 기능별로:
   - 핵심 파일 또는 디렉토리를 코드베이스에서 직접 확인하여 경로를 적는다 (추측 금지)
   - 증상 키워드는 다음 단서에서 추출:
     - 라우트 경로의 한글 주석 (예: `// 마이페이지 적립금 조회`)
     - PRD/기획 문서가 코드 근처에 있다면 거기서 사용된 용어
     - 컴포넌트/페이지 제목 텍스트
   - 코드에서 추출 불가능하면 빈 칸으로 두고, 작성 후 사용자에게 보완 요청
3. 도메인이 7개를 넘으면 도메인별로 묶어 소제목(`### {도메인명}`)을 추가
4. 작성 완료 후 사용자에게 안내:
   > DOMAIN_MAP.md를 생성했습니다. 증상 키워드는 코드에서 추출 가능한 만큼만 채웠습니다.
   > 실제 VOC/문의에서 자주 쓰이는 표현을 추가로 보완하시면 검색 정확도가 올라갑니다.

### 금지 사항

- 영향 관계(impacts/affected_by) 같은 정적 의존성 매트릭스는 작성하지 않는다. Superpowers의 brainstorming이 동적으로 추론하는 것이 더 정확하다.
- ENTITIES, SERVICE_FLOW, CHANGE_LOG 같은 하위 문서는 생성하지 않는다.

---

## 2단계: 원칙 파일 구성

`.claude/rules/principles/` 디렉토리를 생성하고, **principles-spec** 스킬에 정의된 각 파일을 생성한다.

**4단계에서 hooks를 구성하는 경우**, principles-spec의 hooks 연동 원칙에 따라:
- `command-principles.md`는 생성하지 않는다 (guard-bash.sh가 대체)
- `git-workflow.md`에서 hook이 강제하는 규칙을 제거한다
- `code-modification.md`에서 hook이 강제하는 규칙을 제거한다
- `hooks-overview.md`를 생성한다

**hooks를 구성하지 않는 경우**, 기존과 동일하게 7개 파일을 모두 생성한다:
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

## 4단계: Hooks 구성

### Step 1: 스택 기반 hook 변수 확정

1단계 Step 2에서 수집한 프로젝트 정보를 바탕으로, **hooks-spec** 스킬의 각 파일별 상세(§2)에 정의된 '프로젝트별 적응' 테이블을 참조하여 아래 변수를 확정한다.

확정할 변수 (→ 참조 위치):
- `FORMATTER_CMD`, `FORMATTER_EXTENSIONS` → hooks-spec §2-4 포매터 테이블
- `LINTER_CMD`, `LINTER_EXTENSIONS` → hooks-spec §2-4 린터 테이블
- `TYPE_CHECK_CMD`, `TYPE_CHECK_EXTENSIONS` → hooks-spec §2-5 타입 체커 테이블
- `PROTECTED_PATTERNS` → hooks-spec §2-2 보호 패턴 (공통 + 스택별)
- `GIT_STRATEGY`, `PROTECTED_BRANCHES` → hooks-spec §2-3 규칙 4 테이블
- `DEBUG_PATTERNS` → hooks-spec §2-3 규칙 5 테이블
- `PM` → hooks-spec §4 공통 설계 원칙
- `COMMIT_LANG` → hooks-spec §5 질의응답 6번

자동 감지로 확정 가능한 항목은 질문 없이 확정한다.

### Step 2: 사용자 질의응답

hooks는 기본적으로 활성화한다. 감지 결과를 요약하고, 변경 원하는 항목만 말해달라고 한다.
한 번에 모든 질문을 나열하지 않고, 요약 → 확인 → (필요시) 세부 조정 순서로 진행한다.

```
예시 출력:

📋 Hook 설정을 구성합니다. 프로젝트 분석 결과:

  포매터: Prettier (감지됨)
  린터: ESLint (감지됨) 
  타입 체크: tsc --noEmit (tsconfig.json 감지)
  Git 전략: Git Flow (develop 브랜치 감지)
  보호 파일: .env*, next.config.*, pnpm-lock.yaml, tsconfig.json, CI/CD 설정
  커밋 메시지: 한국어
  
이대로 진행할까요? 변경하고 싶은 항목이 있으면 말씀해주세요.
```

사용자가 "이대로 진행"하면 즉시 파일 생성 단계로 넘어간다.
변경 요청이 있으면 해당 항목만 조정한 후 진행한다.

환경 제약 등으로 hooks 구성 자체가 곤란하다는 사용자 요청이 명시적으로 있는 경우에만 4단계 전체를 스킵한다. 이 경우 2단계에서 `command-principles.md`를 생성하지 않은 상태였다면 다시 생성한다.

### Step 3: Hook 스크립트 생성

`.claude/hooks/` 디렉토리를 생성하고, hooks-spec 스킬의 각 파일별 상세(§2)를 따라 5개 스크립트를 생성한다. 각 스크립트는 해당 섹션의 '역할', '스크립트 구조', '프로젝트별 적응', '주의사항'을 빠짐없이 반영한다:

1. **session-init.sh** — 감지한 스택에 맞춰 런타임 버전 수집 명령 조정
2. **protect-files.sh** — 확정된 PROTECTED_PATTERNS를 카테고리별 배열에 배치. `.claude/settings.json`은 보호 목록에서 제외
3. **guard-bash.sh** — 5개 규칙: 셸 연산자 차단, $() 백틱 차단, git add . 차단, 보호 브랜치 push 차단 (GIT_STRATEGY 기반), 디버그 코드 커밋 차단 (DEBUG_PATTERNS 기반)
4. **post-edit-quality.sh** — 감지된 FORMATTER_CMD, LINTER_CMD 사용. 미감지 도구는 스킵 로직 포함
5. **stop-verify.sh** — 무한루프 방어(stop_hook_active) 최상단 배치. 타입 체커 미감지 시 스킵. 미커밋 검사에서 .claude/ 변경 예외 처리

### Step 4: settings.json 머지

기존 `.claude/settings.json`이 있는 경우:
- 기존 `hooks` 키가 없으면 추가한다
- 기존 `hooks` 키가 있으면 사용자에게 덮어쓸지 확인한다
- `hooks` 외의 다른 키(permissions, env 등)는 보존한다

기존 파일이 없으면 hooks-spec 스킬의 settings.json 생성 규칙에 따라 새로 생성한다.
모든 command에 `bash "$CLAUDE_PROJECT_DIR"/.claude/hooks/xxx.sh` 형태를 사용한다 (chmod +x 불필요).

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

### 도메인 맵
- [ ] .claude/DOMAIN_MAP.md 생성 완료 ({행 수}개 기능)
- [ ] 모든 행의 파일 위치가 실제 존재함을 검증

### 원칙 파일 (.claude/rules/principles/)
- [ ] workflow-start.md
- [ ] code-modification.md
- [ ] git-workflow.md
- [ ] work-history.md
- [ ] self-improvement.md
- [ ] lessons.md
- [ ] command-principles.md (hooks 미구성 시에만)
- [ ] hooks-overview.md (hooks 구성 시에만)

### Hooks (.claude/hooks/) — hooks 구성 시
- [ ] session-init.sh — SessionStart: 컨텍스트 주입
- [ ] protect-files.sh — PreToolUse: 보호 파일 차단
- [ ] guard-bash.sh — PreToolUse: 명령어 가드레일
- [ ] post-edit-quality.sh — PostToolUse: {포매터} + {린터}
- [ ] stop-verify.sh — Stop: {타입 체커} + 커밋 검사
- [ ] .claude/settings.json — hooks 설정

### 이슈 트래킹
- [ ] .claude/tasks/known-issues.md (TODO: {n}건, FIXME: {n}건)

### 특이사항
- {분석 중 발견된 주의가 필요한 영역}
```
