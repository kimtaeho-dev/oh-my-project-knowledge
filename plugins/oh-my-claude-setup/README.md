# oh-my-claude-setup

한국 팀을 위한 Claude Code 프로젝트 부트스트래퍼. `/oh-my-claude-setup:init` 한 번으로 CLAUDE.md, 작업 원칙, 도메인 맵, hooks를 자동 구성합니다.

> **권장 사용법**
>
> 이 플러그인은 [Superpowers](https://github.com/obra/superpowers)를 메인 워크플로우 엔진으로 사용하는 것을 가정합니다.
> Superpowers가 brainstorm → plan → execute의 개발 워크플로우를 담당하고, `oh-my-claude-setup`은 그 토양(프로젝트 컨텍스트, 원칙, 가드레일)을 만듭니다.
>
> 설치 권장 순서:
>
> 1. Superpowers 설치: `/plugin marketplace add obra/superpowers-marketplace` → `/plugin install superpowers@superpowers-marketplace`
> 2. `oh-my-claude-setup` 설치 후 `/oh-my-claude-setup:init`으로 프로젝트 부트스트래핑

## 이 플러그인이 만드는 것

`/oh-my-claude-setup:init` 실행 시 다음 파일들을 생성합니다:

- `CLAUDE.md` — 프로젝트 진입점 (한국어, 단일 앱 / 모노레포 / MSA 자동 판별)
- `.claude/DOMAIN_MAP.md` — 도메인 → 파일 위치 + 증상 키워드 매핑 (단일 파일)
- `.claude/rules/principles/` — 작업 원칙 파일들 (hooks 사용 시 일부 통합/생략)
- `.claude/tasks/known-issues.md` — TODO/FIXME 자동 수집
- `.claude/hooks/` — 5개 hook 스크립트 (session-init, protect-files, guard-bash, post-edit-quality, stop-verify)
- `.claude/settings.json` — hooks 등록

## 사용

```bash
# 1. 마켓플레이스 등록
/plugin marketplace add kimtaeho-dev/oh-my-claude-plugins

# 2. 설치
/plugin install oh-my-claude-setup@oh-my-claude-plugins

# 3. 프로젝트 루트에서 실행
/oh-my-claude-setup:init
```

## 설계 원칙

- **정적 문서는 최소한으로**: Superpowers의 brainstorming이 매번 컨텍스트를 동적으로 잡으므로, 갱신 부담이 큰 영향 관계 매트릭스/시퀀스 다이어그램은 만들지 않습니다. `DOMAIN_MAP.md` 한 파일이 "어디를 건드릴지" 좁히는 데 충분합니다.
- **자유 한국어 기능명**: 표준 식별자 컨벤션을 강제하지 않습니다. VOC/문의에서 실제 쓰이는 표현(증상 키워드)을 그대로 매핑합니다.
- **hooks가 강제하는 규칙은 principles에서 제거**: 같은 규칙을 두 군데서 지시하지 않습니다. hook이 환경에서 강제하면 그 자체로 충분하고, principles에는 Claude의 판단이 필요한 규칙만 남깁니다.

## 관련 플러그인

- [Superpowers](https://github.com/obra/superpowers) — 메인 워크플로우 엔진. brainstorm → plan → execute, TDD, 코드 리뷰, 디버깅 방법론. 함께 사용 권장.
