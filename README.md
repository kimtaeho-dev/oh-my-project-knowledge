# oh-my-claude-plugins

Claude Code 플러그인 마켓플레이스입니다.

## 설치

```bash
# 1. 마켓플레이스 등록
/plugin marketplace add kimtaeho-dev/oh-my-claude-plugins

# 2. 플러그인 설치
/plugin install oh-my-claude-setup@oh-my-claude-plugins

# 3. 플러그인 로드
/reload-plugins
```

로컬 테스트:
```bash
claude --plugin-dir ./plugins/oh-my-claude-plugins
```

## 플러그인 목록

| 플러그인 | 버전 | 설명 |
|---|---|---|
| [oh-my-claude-setup](plugins/oh-my-claude-setup/) | 2.0.0 | 한국 팀 Claude Code 프로젝트 부트스트래퍼 (Superpowers와 함께 사용 권장) |

### oh-my-claude-setup

`/oh-my-claude-setup:init` 한 번으로 프로젝트 컨텍스트와 가드레일을 자동 구성합니다. 단일 앱 / 모노레포 / MSA 프로젝트 유형을 자동 판별하여 대응합니다.

[Superpowers](https://github.com/obra/superpowers)를 메인 워크플로우 엔진으로 사용하는 것을 권장합니다. Superpowers가 brainstorm → plan → execute의 개발 워크플로우를 담당하고, `oh-my-claude-setup`은 그 토양(프로젝트 컨텍스트, 원칙, hooks)을 만듭니다.

| 명령 | 설명 |
|---|---|
| `/oh-my-claude-setup:init` | Claude Code 초기 환경 구성 |

생성되는 파일:
- `CLAUDE.md` — 프로젝트 유형별 구조화된 진입점
- `.claude/DOMAIN_MAP.md` — 도메인 → 파일 위치 + 증상 키워드 매핑
- `.claude/rules/principles/` — 작업 원칙 파일들 (hooks 사용 시 일부 통합/생략)
- `.claude/tasks/known-issues.md` — TODO/FIXME 자동 수집
- `.claude/hooks/` — 5개 hook 스크립트 (session-init, protect-files, guard-bash, post-edit-quality, stop-verify)
- `.claude/settings.json` — hooks 등록

### oh-my-project-knowledge (deprecated)

[`plugins/oh-my-project-knowledge/`](plugins/oh-my-project-knowledge/)는 v2.0.0 기준으로 `oh-my-claude-setup`의 슬림 도메인 맵으로 흡수되었습니다. marketplace에서는 더 이상 노출되지 않으며, 코드는 학습 자산으로 보존됩니다. 자세한 내용은 [oh-my-project-knowledge/README.md](plugins/oh-my-project-knowledge/README.md)를 참조하세요.

## 레포 구조

```
oh-my-claude-plugins/
├── .claude-plugin/
│   └── marketplace.json               # 마켓플레이스 카탈로그
├── plugins/
│   ├── oh-my-claude-setup/            # Claude Code 초기 환경 구성 플러그인
│   │   ├── .claude-plugin/
│   │   │   └── plugin.json
│   │   ├── commands/
│   │   │   └── init.md
│   │   └── skills/
│   │       ├── claude-md-spec/SKILL.md
│   │       ├── principles-spec/SKILL.md
│   │       └── hooks-spec/SKILL.md
│   └── oh-my-project-knowledge/       # [DEPRECATED] 프로젝트 지식 문서화 플러그인 (학습 자산으로 보존)
└── README.md
```

향후 새 플러그인은 `plugins/` 아래에 추가됩니다.
