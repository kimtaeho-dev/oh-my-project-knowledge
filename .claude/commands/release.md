마켓플레이스 플러그인의 새 버전을 릴리스하라.

## 인자 형식

`{플러그인명} {bump-type}`

- `{플러그인명}`: 릴리스할 플러그인 이름 (예: `oh-my-project-knowledge`)
- `{bump-type}`: 버전 범프 유형 — `patch` | `minor` | `major`

예시: `oh-my-project-knowledge minor`

인자가 없으면 사용법을 안내하고 중단하라.

## 사전 검증

1. `plugins/{플러그인명}/.claude-plugin/plugin.json`이 존재하는지 확인하라. 없으면 오류 출력 후 중단.
2. `.claude-plugin/marketplace.json`에 해당 플러그인이 등록되어 있는지 확인하라. 없으면 오류 출력 후 중단.
3. 작업 디렉토리에 커밋되지 않은 변경이 있는지 확인하라 (`git status --porcelain`). 있으면 경고하고 먼저 커밋할지 사용자에게 확인하라.
4. `gh` CLI가 설치되어 있는지 확인하라. 없으면 GitHub Release 생성을 건너뛸 것임을 안내하라.

## 릴리스 절차

### Step 1: 버전 계산

1. `plugins/{플러그인명}/.claude-plugin/plugin.json`에서 현재 버전을 읽어라
2. SemVer 규칙에 따라 새 버전을 계산하라:
   - `patch`: 0.2.0 → 0.2.1
   - `minor`: 0.2.0 → 0.3.0
   - `major`: 0.2.0 → 1.0.0
3. 계산된 새 버전을 사용자에게 표시하고 확인을 받아라

### Step 2: 버전 업데이트

두 파일의 버전을 동시에 업데이트하라:

1. `plugins/{플러그인명}/.claude-plugin/plugin.json`의 `version` 필드
2. `.claude-plugin/marketplace.json`의 해당 플러그인 `version` 필드

### Step 3: 변경 이력 수집

1. 이전 릴리스 태그를 찾아라: `git tag --list '{플러그인명}-v*' --sort=-version:refname | head -1`
2. 이전 태그가 있으면 해당 태그 이후의 커밋 목록을 수집하라:
   `git log {이전태그}..HEAD --oneline -- plugins/{플러그인명}/`
3. 이전 태그가 없으면 전체 커밋에서 해당 플러그인 관련 커밋을 수집하라:
   `git log --oneline -- plugins/{플러그인명}/`
4. 수집된 커밋 목록을 릴리스 노트로 정리하라 (한글, 글머리 기호 목록)

### Step 4: 커밋 및 태그

1. 변경된 두 파일을 스테이징하라
2. 커밋 메시지 형식:
   ```
   Release {플러그인명} v{새버전}
   ```
3. 태그 생성:
   ```
   git tag {플러그인명}-v{새버전}
   ```

### Step 5: 푸시

1. 커밋과 태그를 함께 푸시하라:
   ```
   git push && git push --tags
   ```

### Step 6: GitHub Release 생성

`gh` CLI로 GitHub Release를 생성하라:

```
gh release create {플러그인명}-v{새버전} \
  --title "{플러그인명} v{새버전}" \
  --notes "{릴리스 노트}"
```

릴리스 노트 형식:

```markdown
## {플러그인명} v{새버전}

### 변경 사항
- {커밋 요약 1}
- {커밋 요약 2}
- ...

### 설치 / 업데이트
\`\`\`bash
# 신규 설치
claude plugin install {플러그인명}@oh-my-claude-plugins

# 업데이트
claude plugin update {플러그인명}@oh-my-claude-plugins
\`\`\`
```

## 완료 출력

릴리스가 완료되면 다음 정보를 출력하라:

- 플러그인명
- 이전 버전 → 새 버전
- 태그명
- GitHub Release URL
