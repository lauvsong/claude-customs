---
name: github-notification-cleanup
description: GitHub 알림 정리 — 내가 리뷰해야 할 open PR만 남기고 나머지 done 처리
---

DO NOT DO ANY WORK DIRECTLY. ASSIGN THE ENTIRE TASK TO AN AGENT (subagent) using the Agent tool if you call MCP.

GitHub 알림 inbox를 정리해줘. 내가 당장 리뷰해야 하는 PR 알림만 남기고 나머지는 전부 done 처리.
되묻거나 확인하지 말고 즉시 수행하세요.
gh 스킬을 참고하여 `gh` CLI로만 작업을 수행한다.

## 배경 지식 (필독)

- `gh api --paginate "notifications?all=true"` 는 **절대 사용하지 않는다**.
  - `--paginate`는 Link 헤더를 따라가며 archived(done) 페이지까지 긁어온다.
  - archived 알림에 PATCH를 호출하면 해당 알림이 inbox로 되살아날 수 있다.
- done 처리에 `PATCH notifications/threads/{id}` 를 쓰지 않는다.
  - PATCH는 "읽음(read)" 처리만 할 뿐, 실제 아카이브(done)가 아니다.
  - 진짜 done 처리는 GraphQL `markAllNotifications(state: DONE)` mutation을 사용한다.
- **주의**: `markAllNotifications(state: DONE)` 은 **READ/UNREAD 상관없이 inbox의 모든 알림을 아카이브**한다.
  - "UNREAD만 아카이브된다"는 것은 이 GHE 환경에서 사실이 아니다.
  - keep 알림을 PATCH로 READ 처리하는 것만으로는 보호되지 않는다.
- keep 알림은 bulk DONE 처리 후 GraphQL `markAllNotifications(state: UNREAD)`로 복원한다.
  - `query` 파라미터로 keep 알림의 repo를 지정해 범위를 좁힌다.

## 절차

1. 내 GitHub 사용자명 조회 — `gh api user --jq '.login'`

2. 알림 inbox 조회 — `--paginate` 없이 페이지 단위로 수동 조회
   - `gh api "notifications?all=true&per_page=100&page=1"` 부터 시작
   - 빈 배열(`[]`)이 반환되는 페이지가 나오면 중단
   - 최대 10페이지(1000개)까지만 조회

3. 알림 1차 필터링:
   - `subject.type != "PullRequest"` → done 후보
   - PR이면서 `reason != "review_requested"` → done 후보
   - `reason == "review_requested"` PR만 상세 조회 대상으로 넘긴다

4. PR 상세 조회 후 최종 판정:
   - PR 상세 정보에서 아래를 확인한다:
     - PR 상태가 open인지
     - `requested_reviewers`에 내 GitHub username이 포함되는지
     - `requested_teams` 중 내가 속한 팀이 포함되는지
   - **남길 것 (keep)**: subject.type == "PullRequest" AND PR이 open AND (내가 reviewer이거나 내가 속한 팀이 reviewer로 지정됨)
   - **done 처리 대상**: 그 외 전부 (closed/merged PR, 내가 reviewer가 아닌 PR, Issue, CI 등 모든 알림)

5. inbox 전체 done 처리 (GraphQL):
   ```
   gh api graphql -f query='mutation { markAllNotifications(input: {query: "", state: DONE}) { success } }'
   ```
   - READ/UNREAD 구분 없이 inbox의 모든 알림이 아카이브된다

6. keep 알림 복원 (GraphQL):
   - keep 알림이 있는 경우, repo별로 `markAllNotifications(state: UNREAD)` 를 호출해 inbox로 복원한다
   - keep 알림이 여러 repo에 걸쳐 있으면 repo마다 별도 호출한다
   ```
   gh api graphql -f query='mutation { markAllNotifications(input: {query: "repo:{owner}/{repo} is:pr", state: UNREAD}) { success } }'
   ```
   - 복원 후 `gh api "notifications?all=false&per_page=50"` 로 inbox를 확인한다
   - keep 대상이 없으면 이 단계는 건너뛴다

7. 결과 요약 보고:
   - done 처리된 알림 수 및 목록 (repo/title)
   - 남긴 알림 수 및 목록 (repo/title — 리뷰 필요)