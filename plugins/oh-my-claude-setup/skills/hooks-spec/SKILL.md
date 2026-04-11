---
name: hooks-spec
description: |
  프로젝트 스택에 맞는 Claude Code hook 파일 생성 규칙 정의.
  oh-my-claude-setup:init 명령의 hooks 구성 Phase에서 참조한다.
  프로젝트 감지 결과와 사용자 선택을 조합하여 스크립트를 생성한다.
---

# Hooks Spec

---

## 1. 생성 파일 목록

| 파일 | 이벤트 | 매처 | 한 줄 역할 |
|------|--------|------|-----------|
| `.claude/hooks/session-init.sh` | SessionStart | `startup\|resume\|clear\|compact` | 세션 시작 시 프로젝트 상태 요약 (git, 런타임, 작업 계획) |
| `.claude/hooks/protect-files.sh` | PreToolUse | `Edit\|Write\|MultiEdit` | 환경변수, 설정, 잠금 파일 등 보호 대상 수정 차단 |
| `.claude/hooks/guard-bash.sh` | PreToolUse | `Bash` | 위험 명령 차단 (합성 명령, git add ., 보호 브랜치 push, 디버그 코드 커밋) |
| `.claude/hooks/post-edit-quality.sh` | PostToolUse | `Edit\|Write\|MultiEdit` | 편집된 파일에 포매터 + 린터 자동 실행 |
| `.claude/hooks/stop-verify.sh` | Stop | (매처 없음) | 작업 종료 전 타입 체크 + 미커밋 변경 검사 |
| `.claude/settings.json` | — | — | 위 5개 hook의 등록 설정 |

---

## 2. 파일별 상세

### 2-1. session-init.sh

#### 역할

Claude는 세션이 시작되거나 compact 후 이전 맥락을 잃는다.
이 hook은 매 세션 시작 시 **현재 프로젝트의 상태를 짧은 텍스트로 요약**하여
Claude의 컨텍스트에 자동 주입한다.

이를 통해:
- Claude가 "지금 어떤 브랜치에서 뭘 하고 있었는지"를 즉시 파악한다
- 사용자가 매번 "지금 feature/login 브랜치야"라고 알려줄 필요가 없다
- compact 후에도 동일한 상태 정보가 복원된다
- 진행 중인 작업 계획이나 과거 교훈이 있으면 자연스럽게 검토를 유도한다

#### 이벤트 / 매처

- 이벤트: `SessionStart`
- 매처: `startup|resume|clear|compact` (모든 세션 시작 유형)
- 출력 방식: stdout 텍스트 → Claude 컨텍스트에 자동 추가

#### 스크립트 구조

스크립트는 아래 순서로 정보를 수집하고, 결과를 stdout으로 출력한다.
각 섹션은 수집 실패 시 건너뛰고, 성공한 항목만 출력한다.

```
1. Git 상태 수집
   - 현재 브랜치 (git branch --show-current)
   - 최근 커밋 해시 + 메시지 (git log -1 --format='%h %s')
   - 미커밋 변경 파일 수 + 목록 상위 15개 (git status --porcelain)

2. 런타임 환경 수집 (프로젝트 스택에 따라 다름 — 아래 "프로젝트별 적응" 참조)
   - 언어 버전
   - 패키지 매니저 + 버전
   - 프레임워크 버전

3. 프로젝트 상태 수집
   - .env 파일 존재 목록 (내용은 노출하지 않음, 파일명만)
   - 작업 계획 파일: .claude/tasks/plans/{브랜치명 슬러그}.md 존재 여부
     - 브랜치명의 '/'를 '-'로 변환하여 파일명 매칭 (예: feature/login → feature-login.md)
   - known-issues.md 항목 수 (grep -c '^\s*-')
   - lessons.md 항목 수 (존재 시 "N개 교훈 — 세션 시작 시 검토" 표시)

4. stdout 출력
   - Markdown 형식으로 구조화
   - 미커밋 변경이 있으면 마지막에 경고 표시
```

#### 프로젝트별 적응 — 런타임 수집 명령

| 스택 | 언어 버전 | 패키지 매니저 | 프레임워크 버전 |
|------|-----------|---------------|----------------|
| JS/TS (Node) | `node --version` | pnpm-lock.yaml → `pnpm --version` / yarn.lock → `yarn --version` / 그 외 → `npm --version` | package.json에서 next, react, vue, angular 등의 버전 추출 |
| Go | `go version` | (없음) | go.mod에서 모듈 경로 추출 |
| Rust | `rustc --version` | `cargo --version` | Cargo.toml에서 패키지 버전 추출 |
| Python | `python3 --version` | pip / poetry / pipenv 감지 | pyproject.toml에서 프레임워크 버전 추출 |
| Kotlin/Java | `java --version` | `gradle --version` 또는 `mvn --version` | build.gradle.kts에서 Spring Boot 버전 추출 |

프레임워크 버전 추출 시 jq가 있으면 jq, 없으면 grep+sed로 파싱한다.

#### 주의사항

- timeout을 10초로 설정. 대규모 repo에서 git status가 느릴 수 있음
- .env 파일의 **내용은 절대 출력하지 않는다**. 파일명만 나열
- 출력 총량이 10,000자를 넘으면 Claude Code가 파일로 저장하므로, 미커밋 파일 목록을 15개로 제한

---

### 2-2. protect-files.sh

#### 역할

환경변수, 인증서, 빌드 설정, 잠금 파일 등 **수동 관리가 필요한 파일**을
Claude가 임의로 수정하는 것을 차단한다.

차단 시 Claude에게 "이 파일은 보호 목록에 있으니 사용자에게 확인을 요청하세요"라는
피드백을 전달하여, Claude가 사용자에게 자연스럽게 물어보도록 유도한다.

#### 이벤트 / 매처

- 이벤트: `PreToolUse`
- 매처: `Edit|Write|MultiEdit`
- 차단 방식: exit 2 + stderr 메시지 → Claude에게 에러 피드백

#### 스크립트 구조

```
1. stdin에서 JSON 읽기
2. tool_input.file_path 추출 (jq 우선, grep+sed 폴백)
3. file_path가 없으면 exit 0 (통과)
4. 카테고리별 패턴 배열과 대조:
   - CRITICAL: 환경변수, 인증서
   - CONFIG: 빌드/도구 설정
   - INFRA: CI/CD, 배포
   - LOCK: 의존성 잠금
5. 매칭 시: stderr에 "차단 이유 + 카테고리" 출력 → exit 2
6. 미매칭 시: exit 0 (허용)
```

#### 프로젝트별 적응 — 보호 패턴

**공통 (모든 스택):**
```
CRITICAL: \.env($|\..*)  \.(pem|key|cert|crt)$
INFRA:    \.github/  \.gitlab-ci\.yml$  Dockerfile  docker-compose  \.dockerignore$
          vercel\.json$  netlify\.toml$  fly\.toml$  railway\.json$
```

**스택별 CONFIG 패턴:**

| 스택 | CONFIG 패턴 |
|------|-------------|
| Next.js / React | `next\.config\.*`, `vite\.config\.*`, `tsconfig.*\.json`, eslint/prettier/postcss/tailwind 설정 |
| Go | (CONFIG 해당 없음 — go.sum은 LOCK) |
| Rust | (CONFIG 해당 없음 — Cargo.lock은 LOCK) |
| Spring Boot/Kotlin | `application.*\.yml`, `application.*\.properties`, `build\.gradle.*` |
| Python | `pyproject\.toml`, `setup\.cfg` |

**스택별 LOCK 패턴:**

| 스택 | LOCK 패턴 |
|------|-----------|
| JS/TS (pnpm) | `pnpm-lock\.yaml` |
| JS/TS (yarn) | `yarn\.lock` |
| JS/TS (npm) | `package-lock\.json` |
| Go | `go\.sum` |
| Rust | `Cargo\.lock` |
| Python (poetry) | `poetry\.lock` |
| Python (pipenv) | `Pipfile\.lock` |
| Kotlin/Java (Gradle) | `gradle\.lockfile`, `gradlew`, `gradlew\.bat` |

#### 주의사항

- `.claude/settings.json`은 보호 목록에 **포함하지 않는다**. hook 설정 자체를 수정할 수 없게 되는 역설이 발생함 (실사용에서 발견된 문제)
- 카테고리명을 stderr에 포함하면 Claude가 사용자에게 "이 파일은 환경변수 파일이라 보호되어 있습니다"처럼 맥락을 전달할 수 있음
- timeout 5초 (파일 경로 매칭만 하므로 빠름)

---

### 2-3. guard-bash.sh

#### 역할

principles에서 "~하지 마라"로 안내하던 규칙 중
**셸 명령 레벨에서 결정론적으로 판별 가능한 것들**을 자동 차단한다.

이 hook이 존재하면 principles의 `command-principles.md`가 불필요해지고,
`git-workflow.md`와 `code-modification.md`에서도 해당 규칙을 제거할 수 있다.

#### 이벤트 / 매처

- 이벤트: `PreToolUse`
- 매처: `Bash`
- 차단 방식: exit 2 + stderr 메시지 → Claude에게 에러 피드백

#### 스크립트 구조 — 5개 규칙을 순서대로 검사

```
1. stdin에서 JSON 읽기 → tool_input.command 추출

2. 규칙 1: 셸 연산자 합성 명령 차단
   - git commit 메시지 내부는 검사에서 제외 (sed로 git commit 이후 제거)
   - 나머지 부분에서 &&, ||, ; 탐지 시 차단
   - 파이프(|)와 리다이렉션(>)은 차단 대상에서 제외한다 (grep | head 등 단일 논리 연산에서 필수적이며, 차단 시 오탐이 과다)
   - 차단 메시지에 "각 명령어를 개별 실행하세요" 안내 포함

3. 규칙 2: 커밋 메시지 내 $() / 백틱 차단
   - "git commit"이 포함된 명령에서만 검사
   - 차단 메시지에 "-m 옵션 여러 번 사용" 안내 포함

4. 규칙 3: git add . / -A / --all 차단
   - 정확한 패턴 매칭 (git add .으로 시작, git add -A 등)
   - 차단 메시지에 "파일을 명시적으로 지정하세요" 안내 포함

5. 규칙 4: 보호 브랜치 직접 push 차단
   - 패턴은 Git 전략에 따라 다름 (아래 "프로젝트별 적응" 참조)
   - 차단 메시지에 "작업 브랜치에서 push 후 PR 생성" 안내 포함

6. 규칙 5: 디버그 코드 포함 커밋 차단
   - "git commit"이 포함된 명령에서만 검사
   - staged된 코드 파일에서 디버그 패턴 탐지 (git diff --cached)
   - 추가된 줄(+)만 검사하여 기존 코드의 디버그 코드를 오탐하지 않음
   - 탐지 시 발견 건수 + 해당 줄 상위 10개 표시

7. 모든 규칙 통과: exit 0
```

#### 프로젝트별 적응 — 규칙 4 (보호 브랜치)

| Git 전략 | 보호 브랜치 | push 차단 패턴 |
|----------|-------------|----------------|
| Git Flow | main, develop | `git push.*(origin\|upstream)\s+(main\|develop)(\s\|$)` |
| GitHub Flow | main | `git push.*(origin\|upstream)\s+main(\s\|$)` |
| Trunk-based | main (--force만) | `git push.*--force.*(origin\|upstream)\s+main` 또는 `git push.*(origin\|upstream)\s+main.*--force` |

#### 프로젝트별 적응 — 규칙 5 (디버그 코드 패턴)

| 스택 | 차단 패턴 | 허용 예외 |
|------|-----------|-----------|
| JS/TS | `console\.(log\|debug\|dir\|trace)\b`, `\bdebugger\b` | `console.error`, `console.warn`, `console.info` |
| Go | `fmt\.Print(ln\|f)?\b` | `log.*` 패키지 사용 |
| Python | `print\(`, `breakpoint\(\)`, `pdb\.set_trace` | `logging.*` 사용 |
| Kotlin/Java | `println\(`, `System\.out\.print` | `logger.*` 사용 |
| Rust | `dbg!\(`, `println!\(` | `log::`, `tracing::` 사용 |

#### 주의사항

- 규칙 1에서 git commit 메시지 내부를 제외하지 않으면, 커밋 메시지에 "A && B" 같은 텍스트가 있을 때 오탐 발생
- 규칙 5는 staged 파일의 **추가된 줄(+)**만 검사. 기존 코드에 console.log가 있어도 이번 커밋에서 추가한 게 아니면 통과
- timeout 10초 (규칙 5에서 git diff --cached가 약간 걸릴 수 있음)

---

### 2-4. post-edit-quality.sh

#### 역할

Claude가 파일을 편집할 때마다 **포매터와 린터를 자동 실행**하여
코드 스타일과 품질을 일관되게 유지한다.

PostToolUse는 이미 실행된 후이므로 차단은 불가능하다.
대신 린터 에러가 발견되면 JSON 피드백으로 Claude에게 전달하여
다음 편집에서 수정하도록 유도한다.

#### 이벤트 / 매처

- 이벤트: `PostToolUse`
- 매처: `Edit|Write|MultiEdit`
- 피드백 방식: JSON stdout의 `additionalContext` → Claude가 다음 액션에서 인지

#### 스크립트 구조

```
1. stdin에서 JSON 읽기 → tool_input.file_path 추출
2. 파일 존재 확인 (삭제된 파일이면 exit 0)
3. 확장자 필터:
   - 코드 파일 (스택별 대상 확장자): 포매터 + 린터 실행
   - 보조 파일 (.json, .css, .md 등): 포매터만 실행
   - 그 외 (.png, .woff 등): exit 0 (스킵)
4. 패키지 매니저 감지 (pnpm-lock → pnpm exec / yarn.lock → yarn / 그 외 → npx)
5. 포매터 실행
   - 프로젝트에서 포매터가 감지된 경우: 해당 도구로 단일 파일에 --write
   - 포매터가 감지되지 않은 경우: 이 단계를 건너뛴다
6. 린터 실행
   - 프로젝트에서 린터가 감지된 경우: 해당 도구로 단일 파일에 --fix --quiet
   - 린터가 감지되지 않은 경우: 이 단계를 건너뛴다
7. 린터 에러가 남아있으면 JSON 피드백 출력
8. exit 0 (PostToolUse는 항상 exit 0)
```

#### 프로젝트별 적응 — 포매터

| 감지 조건 | 포맷 명령 | 대상 확장자 |
|-----------|-----------|-------------|
| `.prettierrc*` 또는 `prettier.config.*` 또는 package.json에 prettier | `{PM} prettier --write "{FILE}"` | `.ts,.tsx,.js,.jsx,.json,.css,.scss,.md,.html,.yml,.yaml` |
| `biome.json` 또는 `biome.jsonc` | `{PM} biome format --write "{FILE}"` | `.ts,.tsx,.js,.jsx,.json,.css` |
| `go.mod` | `gofmt -w "{FILE}"` | `.go` |
| `Cargo.toml` | `rustfmt "{FILE}"` | `.rs` |
| `build.gradle.kts` + ktlint | `ktlint --format "{FILE}"` | `.kt,.kts` |
| `pyproject.toml` + black | `black "{FILE}"` | `.py` |
| `ruff.toml` 또는 pyproject.toml + ruff | `ruff format "{FILE}"` | `.py` |
| 아무것도 감지 안 됨 | (포매터 단계 스킵) | — |

#### 프로젝트별 적응 — 린터

| 감지 조건 | 린트 명령 | 대상 확장자 |
|-----------|-----------|-------------|
| `.eslintrc*` 또는 `eslint.config.*` | `{PM} eslint --fix --quiet "{FILE}"` | `.ts,.tsx,.js,.jsx` |
| `biome.json` | `{PM} biome check --fix "{FILE}"` | `.ts,.tsx,.js,.jsx` |
| `go.mod` | (단일 파일 모드 미지원 — stop-verify.sh에서 실행) | `.go` |
| `Cargo.toml` | (단일 파일 모드 미지원 — stop-verify.sh에서 실행) | `.rs` |
| `ruff.toml` 또는 pyproject.toml + ruff | `ruff check --fix "{FILE}"` | `.py` |
| 아무것도 감지 안 됨 | (린터 단계 스킵) | — |

#### 주의사항

- **매 편집마다 실행되므로 속도가 핵심**. 단일 파일만 대상으로 하여 200-500ms 이내에 끝나야 함
- 전체 프로젝트 검사(tsc, go build 등)는 여기서 하지 않는다. 그건 stop-verify.sh의 역할
- `2>/dev/null || true` 패턴으로 포매터/린터가 해당 파일 형식을 지원하지 않을 때의 에러를 무시
- timeout 15초

---

### 2-5. stop-verify.sh

#### 역할

Claude가 응답을 종료하려 할 때, **타입 에러가 없는지** + **코드 변경을 커밋했는지**를
순서대로 검증한다.

검증 실패 시 exit 2를 반환하면 Claude가 "아직 끝내면 안 되는구나"라고 판단하고
에러를 수정하거나 커밋을 실행한 뒤 다시 종료를 시도한다.

타입 체크를 PostToolUse가 아닌 Stop에 배치하는 이유:
tsc, go build 등은 **전체 프로젝트**를 검사하므로 10-30초가 걸린다.
매 편집마다 실행하면 10개 파일 수정 시 5분 이상 대기하게 되므로,
작업 종료 시 1회만 실행하는 것이 효율적이다.

#### 이벤트 / 매처

- 이벤트: `Stop`
- 매처: 없음 (매 응답 종료 시 실행)
- 차단 방식: exit 2 + stderr → Claude가 수정 후 재시도

#### 스크립트 구조

```
1. stdin에서 JSON 읽기

2. 무한루프 방어 (최우선)
   - stop_hook_active 필드 확인
   - true이면 즉시 exit 0 (이전 Stop hook이 이미 차단한 적 있음 = 두 번째 시도)
   - 이 체크가 없으면: exit 2 → Claude 계속 → 또 Stop → 또 exit 2 → 무한반복

3. 공통: git 변경 파일 수집
   - git diff --name-only (unstaged)
   - git diff --cached --name-only (staged)
   - git ls-files --others --exclude-standard (untracked)
   - 합산 후 중복 제거

4. 검증 1: 타입 체크 (타입 체커 설정 파일 존재 시에만)
   - 변경 파일 중 대상 확장자가 없으면 스킵
   - 타입 체크 명령 실행
   - 에러 발견 시: stderr에 에러 내용(상위 25줄) + "수정 요청" → exit 2
   - 에러 없으면: 다음 검증으로 진행

5. 검증 1.5: 프로젝트 단위 린터 (go vet, cargo clippy 등 — 해당 스택인 경우에만)
   - 변경 파일 중 대상 확장자가 없으면 스킵
   - 린트 명령 실행
   - 에러 발견 시: stderr에 에러 내용(상위 25줄) + "수정 요청" → exit 2
   - 에러 없으면: 다음 검증으로 진행

6. 검증 2: 미커밋 코드 변경 검사
   - 변경 파일이 없으면 exit 0
   - .claude/, .git/ 변경만 있으면 exit 0 (코드 변경이 아님)
   - 코드 변경이 있으면: stderr에 파일 목록 + 커밋 메시지 형식 안내 → exit 2
```

#### 프로젝트별 적응 — 타입 체커

| 감지 조건 | 체크 명령 | 대상 확장자 |
|-----------|-----------|-------------|
| `tsconfig.json` | `{PM} tsc --noEmit` | `.ts,.tsx` |
| `go.mod` | `go build ./...` | `.go` |
| `Cargo.toml` | `cargo check` | `.rs` |
| `build.gradle.kts` | `./gradlew compileKotlin` | `.kt,.kts` |
| `pyrightconfig.json` | `pyright` | `.py` |
| pyproject.toml에 mypy | `mypy .` | `.py` |
| 위 조건 모두 미충족 | (타입 체크 단계 스킵) | — |

#### 프로젝트별 적응 — 린터 (post-edit에서 이동된 항목)

단일 파일 모드를 지원하지 않는 린터는 post-edit-quality.sh 대신 여기서 실행한다.
타입 체크 후, 커밋 검사 전에 실행한다.

| 감지 조건 | 린트 명령 | 대상 확장자 |
|-----------|-----------|-------------|
| `go.mod` | `go vet ./...` | `.go` |
| `Cargo.toml` | `cargo clippy --fix --allow-dirty` | `.rs` |

#### 주의사항

- **무한루프 방어가 가장 중요**. `stop_hook_active` 체크를 스크립트 최상단에 배치
- 타입 에러 출력을 25줄로 제한. 대규모 프로젝트에서 수백 줄이 나올 수 있음
- `.claude/` 내부 변경(known-issues.md, lessons.md 등)은 코드 변경으로 간주하지 않음
- 커밋 요청 시 메시지에 프로젝트의 커밋 메시지 형식 안내를 포함 (COMMIT_LANG에 맞는 언어로)
- timeout 60초 (tsc가 대규모 프로젝트에서 30초 이상 걸릴 수 있음)

---

## 3. settings.json 생성 규칙

`.claude/settings.json`에 hooks 키를 추가한다.
기존 settings.json이 있으면 hooks 키만 머지한다 (다른 키는 보존).

모든 command에 `bash "$CLAUDE_PROJECT_DIR"/.claude/hooks/xxx.sh` 형태를 사용한다.
이렇게 하면 chmod +x 없이도 동작한다 (파일 다운로드 시 실행 권한 소실 방지).

```json
{
  "hooks": {
    "SessionStart": [
      {
        "matcher": "startup|resume|clear|compact",
        "hooks": [
          {
            "type": "command",
            "command": "bash \"$CLAUDE_PROJECT_DIR\"/.claude/hooks/session-init.sh",
            "statusMessage": "프로젝트 상태 수집 중...",
            "timeout": 10
          }
        ]
      }
    ],
    "PreToolUse": [
      {
        "matcher": "Edit|Write|MultiEdit",
        "hooks": [
          {
            "type": "command",
            "command": "bash \"$CLAUDE_PROJECT_DIR\"/.claude/hooks/protect-files.sh",
            "statusMessage": "보호 파일 검사 중...",
            "timeout": 5
          }
        ]
      },
      {
        "matcher": "Bash",
        "hooks": [
          {
            "type": "command",
            "command": "bash \"$CLAUDE_PROJECT_DIR\"/.claude/hooks/guard-bash.sh",
            "statusMessage": "명령어 검사 중...",
            "timeout": 10
          }
        ]
      }
    ],
    "PostToolUse": [
      {
        "matcher": "Edit|Write|MultiEdit",
        "hooks": [
          {
            "type": "command",
            "command": "bash \"$CLAUDE_PROJECT_DIR\"/.claude/hooks/post-edit-quality.sh",
            "statusMessage": "코드 품질 검사 중...",
            "timeout": 15
          }
        ]
      }
    ],
    "Stop": [
      {
        "hooks": [
          {
            "type": "command",
            "command": "bash \"$CLAUDE_PROJECT_DIR\"/.claude/hooks/stop-verify.sh",
            "statusMessage": "최종 검증 중...",
            "timeout": 60
          }
        ]
      }
    ]
  }
}
```

---

## 4. 공통 설계 원칙

모든 스크립트에 공통으로 적용:

1. **jq 폴백**: jq 있으면 jq, 없으면 grep+sed로 stdin JSON 파싱
2. **패키지 매니저 감지**: pnpm-lock.yaml → `pnpm exec` / yarn.lock → `yarn` / 그 외 → `npx` (JS/TS 스택)
3. **확장자 필터**: 대상이 아닌 파일은 최대한 빨리 exit 0으로 빠져나감
4. **에러 메시지 한국어**: stderr의 안내 메시지는 한국어로 작성하여 Claude가 한국어로 사용자에게 전달

---

## 5. 질의응답 항목

init.md 4단계 Step 2에서 사용자에게 확인한다.
자동 감지 결과를 기본값으로 제안하고, 사용자가 수정할 수 있게 한다.

1. **보호 파일**: 감지된 config 파일 목록을 보여주고, 추가/제거할 파일 확인
2. **Git 전략**: Git Flow / GitHub Flow / Trunk-based 중 선택 (기본값: 브랜치 패턴 감지 기반)
3. **PostToolUse 범위**: 포매터 on/off, 린터 on/off (기본값: 감지된 도구 기준 모두 on)
4. **타입 체크 위치**: Stop에서 실행 / 비활성화 (기본값: 타입 체커 감지 시 on)
5. **커밋 강제**: Stop에서 미커밋 검사 on/off (기본값: on)
6. **커밋 메시지 언어**: 한국어 / 영어 (기본값: 한국어)

감지 결과가 명확한 경우(예: prettier 설정 파일 존재 → 포매터=Prettier)는
질문 없이 자동 확정하고, 결과 요약에서 보여준다.