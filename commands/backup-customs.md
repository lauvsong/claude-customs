# /backup-customs

전역 Claude 설정(`~/.claude/`)을 이 레포에 동기화한다.

## Procedure

1. 아래 대상 파일/디렉토리를 `~/.claude/`에서 이 레포의 루트로 복사한다:
   - `CLAUDE.md`
   - `settings.json` → **env 섹션(토큰/시크릿)은 반드시 제거** 후 복사
   - `rules/`
   - `commands/`
   - `hooks/`
   - `skills/` → 직접 만든 스킬만 (symlink, LICENSE.txt 포함 스킬은 제외)
   - `scheduled-tasks/`

2. 사내 정보가 포함된 파일은 제외하거나 sanitize한다:
   - 토큰, 시크릿, API 키
   - 사내 서비스명, 내부 URL, 프로젝트 코드명
   - 사내 전용 스킬 (jira, wiki 등)

## Excluded (동기화 대상 아님)

- `settings.json`의 `env` 섹션
- `settings.local.json`
- `history.jsonl`, `sessions/`, `projects/`, `cache/`, `debug/`
- `telemetry/`, `plans/`, `tasks/`, `todos/`, `transcripts/`
- `backups/`, `paste-cache/`, `shell-snapshots/`, `file-history/`
- `statsig/`, `stats-cache.json`, `session-env/`, `ide/`, `plugins/`
- symlink 스킬, marketplace 스킬 (LICENSE.txt 포함)
