# rules/02-workflow-orchestration.md — WORKFLOW ORCHESTRATION (Plan Mode only)

## APPLICABILITY: mandatory — always active regardless of stack or project

## SUMMARY (READ FIRST)
- Default is Direct Mode. Plan Mode activates only on multi-file changes, API/contract impact, unknown failures, or deploy risk (see R1).
- On problems: Investigation (log/reproduction/hypothesis) → STOP & Re-plan if needed.
- Delegate research/exploration/parallel analysis to roles (sub-agents) to keep main context clean.
- Before done: verification + Evidence. Final check: "Would a staff engineer approve this?"
- If it's a band-aid, seek elegant solution. If it's a simple fix, avoid over-engineering.
- When transforming code patterns: verify behavioral equivalence (happy path + error path + side effects) before applying.

## RULES
### R1) Plan Mode (conditional — not default)
- Default: Direct Mode (proceed without formal plan).
- Enter Plan Mode only when ANY of these conditions apply:
  - 3+ files to modify
  - Public API / contract / DB schema impact
  - Failure cause unknown (investigation needed)
  - Deploy / production / infra risk involved
  - Architecture decision required
- Plan deliverable (keep brief):
  - Goals / non-goals
  - Change scope (files/modules)
  - Step plan (checklist)
  - Verification plan
  - Risks / rollback (org standard)

### R2) Investigation First, then STOP
- Investigation:
  - Failure log summary (masked)
  - Minimal reproduction (test if possible)
  - Cause hypothesis + evidence
- STOP & Re-plan conditions:
  - Cause unknown
  - Approval-required action needed
  - Essential evidence (logs/failing tests/reproduction) insufficient

### R3) Sub-agent Strategy (role separation)
- Purpose: minimize context pollution + parallel thinking.
- Principles:
  - Separate research/exploration/comparison/risk review into roles
  - One task per role
  - Merge results as "conclusion + evidence + recommendation" into execution plan

### R4) Verification Before Done
- No done without proof of operation.
- Compare baseline (main) vs changed version behavior when possible.
- Minimum bar:
  - Test execution (possible scope)
  - Log verification (masked)
  - Accuracy/reproducibility proof
- Self-question: "Would a staff engineer approve this?"
- **Fresh evidence rule**: "통과/완료" 보고 시점이 마지막 verify와 어긋나면 — 파일이 추가 수정됐거나(`<system-reminder>`의 file modification 알림 포함), 시간이 흘렀거나, 의존 파일이 변경됐을 가능성이 있다면 — **재실행 후 보고**. stale evidence(이전에 통과한 결과 그대로 인용) 금지. system-reminder가 알려준 파일이 검증 대상의 의존이면 별도 알림 없는 다른 파일도 함께 변경됐을 수 있으므로, 의심되면 검증 대상 전체를 재실행.

### R5) Behavioral Equivalence Verification (mandatory on pattern transformation)
- When transforming one code pattern into another, **always verify that behavior AND side effects are fully identical**.
- Two "similar-looking" patterns may differ in error propagation/consumption, context scope, execution order, or exception handling.
- Checklist before applying any transformation:
  1. Is the happy path behavior identical?
  2. Is the error/exception path identical? (propagate vs consume vs ignore)
  3. Are side effects identical? (logging, context propagation, resource cleanup)
  4. Are there differences in call timing, thread, or scheduler?
- If differences exist: state them explicitly and add compensating code (e.g., `onErrorComplete`) together with the transformation.
- Applies to ALL code transformations, not just Reactive: callback→Promise, for→stream, try-catch→Result, etc.

### R6) Elegance vs Over-engineering
- For non-trivial changes: consider "is there a more elegant approach?"
- If it's a band-aid: prefer root-cause solution.
- But: for simple, obvious fixes, skip this (no over-engineering).

### R7) Autonomous Bug Fixing (conditional)
- If evidence exists (logs/stack/failing tests/reproduction): diagnose → fix → verify without extra context switching.
- If evidence is lacking: request only 1-3 minimal pieces of evidence.
- Never claim "fixed CI" without equivalent Evidence.

### R8) Context Mode
Task strictness varies by context:
- Local/Experiment: fast feedback first. Partial testing OK (explain impact). Full verification before release.
- Analysis/Review: no code changes = no forced build/test (unless user requests).
- PR/Release: strict mode. Complete standard verification + Evidence.

Default:
- Code changes → PR/Release
- No code changes → Analysis/Review

### R9) Agent Roles (parallel by default)
For feature add/bug fix/refactoring (behavior impact possible), these roles run in parallel:
- planner: scope/steps/verification plan/risk summary
- tdd-guide: test strategy (reproduction/regression/key cases)
- security-reviewer: secrets/permissions/data safety/dependency·license risks
- architect (if needed): structure/boundaries/dependencies/scalability
- build-error-resolver (on failure): failure hypothesis+reproduction+fix path
- code-reviewer (final stage): requirement met/risk/review-friendly diff/evidence sufficiency

| Role | Trigger | Deliverables |
|------|---------|-------------|
| planner | always | goals · scope · steps · verification · risks/rollback |
| tdd-guide | always | reproduction test → happy/edge cases → regression points |
| security-reviewer | always | secret exposure · auth bypass · data safety · dependency risk |
| architect | design decision | 2-3 options → recommendation + boundary/perf impact |
| build-error-resolver | build failure | log summary → reproduction → hypothesis → fix path |
| code-reviewer | final stage | requirement met · violations · diff quality · evidence |

When parallel results conflict: prioritize "evidence (tests/logs/contracts/policies)".

## WHY
- Plan Mode reduces omissions (verification/approval/boundaries).
- Investigation→STOP flow reduces stalls while maintaining safe confidence.
- Role separation keeps main context clean.

## EXAMPLES
- Feature add (3+ steps): checklist plan → small diffs → test/log Evidence summary
- Unknown cause: Investigation → if still unknown → STOP & minimal questions
- BAD: Replacing `subscribe(onError)` with `doOnError` + `subscribe()` without adding `onErrorComplete` — `doOnError` peeks but does not consume, causing `ErrorCallbackNotImplemented`
- GOOD: Verify error path equivalence first, then add `onErrorComplete()` to match the original consume behavior
