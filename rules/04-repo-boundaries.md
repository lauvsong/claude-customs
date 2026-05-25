# rules/04-repo-boundaries.md — REPO STRUCTURE & BOUNDARIES

## APPLICABILITY: mandatory — always active regardless of stack or project

## SUMMARY (READ FIRST)
- Respect "modify-with-care / read-only / restricted-access" boundaries for safety.
- Production config / infra / deploy files are not modified by default.
- Secret/credential file contents are never viewed (existence check only, true/false level).
- Worktree 위치는 `<repo>/.worktrees/<task-id>` 형태가 표준. 생성 전 작업 대상 repo를 코드로 검증 (R4).

## RULES
### R1) Project Structure (touch with care)
- Code: `src/main/kotlin/**`
- Tests: `src/test/kotlin/**`
- Resources: `src/main/resources/**`

### R2) Read-only (read only, no modifications)
- `src/main/resources/application-prod.yml`
- `infra/**`
- `k8s/**`
- `build/**`
- `.gradle/**`

### R3) Restricted Access (secrets/credentials)
- File content viewing (`cat/read`) absolutely prohibited.
- Only existence/healthcheck level verification allowed (true/false level).
- No key list/value/raw line output.

### R4) Worktree 생성 (작업 repo 확정 + 위치 컨벤션)
**Step 1: 작업 대상 repo 확정** (위치 결정보다 먼저)
- 티켓 제목의 "BE", "FE" 같은 모호한 단어로 repo를 단정하지 않는다.
- 티켓 본문의 **핵심 키워드**(예: 함수명·툴명·API명·도메인 용어)를 모든 후보 repo에 grep해서 코드로 검증한다.
  - 예: 특정 함수명/툴명 키워드 → `for d in ~/ws/*/; do grep -rl "키워드" "$d" --include="*.kt"; done`
- 매칭이 0건 또는 여러 repo면 **사용자에게 확인** 후 진행.

**Step 2: 위치 컨벤션**
- 표준 위치: **`<repo>/.worktrees/<task-id>`** (예: `~/ws/<repo>/.worktrees/FOO-123`)
- 작업 repo 안에 worktree를 두는 게 원칙. repo와 분리되면 IDE/검색/빌드 도구가 컨텍스트를 잃는다.

**금지 위치**:
- `/tmp/**`, `/var/tmp/**` — 재부팅 시 소실 + 설정 override 안 됨.
- `~/ws/` 직하 (`~/ws/<repo>-<task-id>`) — 워크스페이스 루트 오염.
- `~/ws/.claude/worktrees/` — Claude Code의 자동 격리 worktree 영역. 다른 repo worktree를 섞으면 안 됨.
- `~/ws/<repo>/` (이미 체크아웃된 working tree) — 사용자가 작업 중일 수 있음. 이 경우 worktree 추가 불가 → HANDOFF.md 로 전환.

**Step 3: 컨벤션 불명확 시**: 사용자에게 물어본 후 생성 (잘못된 위치에 만들고 옮기는 비용이 더 큼).

## WHY
- Prod/deploy/secret boundaries have high damage and difficult recovery on mistakes.
- 잘못된 worktree 위치는 (1) 임시 경로의 경우 작업 손실, (2) 워크스페이스 오염, (3) 글로벌 설정 미적용 문제를 야기한다.

## EXAMPLES
- BAD: Suggesting/applying modifications to `application-prod.yml`
- GOOD: Read prod config for context only; if changes needed, request approval/procedure
- BAD: 티켓 제목에 "[BE]"가 있다고 backend-1 repo로 단정 → 실제 코드는 backend-2 repo에 있음
- GOOD: 티켓의 핵심 키워드를 모든 repo에 grep → 매칭된 repo로 확정
- BAD: `git worktree add /tmp/worktree-FOO-123 ...` — 임시 경로, 재부팅 시 소실
- BAD: `git worktree add ~/ws/<repo>-FOO-123 ...` — 워크스페이스 루트 오염
- BAD: `git worktree add ~/ws/.claude/worktrees/FOO-123 ...` — Claude Code 자동 격리 영역 침범
- GOOD: `git worktree add ~/ws/<repo>/.worktrees/FOO-123 -b feat/FOO-123 origin/develop`
