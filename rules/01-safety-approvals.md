# rules/01-safety-approvals.md — SAFETY & APPROVALS (NON-NEGOTIABLE)

## SUMMARY (READ FIRST)
- Secrets/tokens/PII: never request, view, store, output, or hardcode. Mask with `***` when sharing logs.
- Destructive actions, Prod impact, Contract breaking: prohibited without Approval Protocol.
- No guardrail bypass: blocked command = stop, report, ask user. No string tricks or indirect execution.
- No test bypass. Evidence required before declaring done.

## RULES
### R1) Secrets / Sensitive Data
- Prohibited: requesting/viewing/storing/outputting/hardcoding secrets/tokens/credentials/passwords/sessions/cookies/PII.
- Never leave in plaintext in logs/responses/documents.
- When sharing is necessary: mask to unidentifiable level with `***` + minimal excerpt.

### R2) Destructive Actions (prohibited without approval)
The following are never performed/recommended without Approval Protocol:
- File deletion/permission/owner changes (`rm -rf`, `chmod 777`, `chown`)
- Git force push/mass clean (`git push --force`, `git clean -fdx`)
- Infra/K8s/container delete/prune (`kubectl delete`, `docker system prune`, `helm uninstall`)
- Network pipe execution (`curl | sh`)
- DB destructive queries/migrations (DELETE/UPDATE/ALTER/DROP)

### R3) Approval Protocol (approval requirements)
Approval is valid only when all 3 elements are present:
1) Target: which resource/data/environment is affected
2) Exact action: the literal command/query/operation
3) Risk acceptance: which risks are accepted (data loss/outage/security exposure)

### R4) Prod Safety / Contract Safety
- Prod-impacting work is avoided by default. If unavoidable: narrow scope + approval required.
- API/contract breaking changes: never proceed without reconfirmation/approval.

### R5) No Guardrail Bypass
- When a command is blocked (error contains `PreToolUse` + `hook error` + `BLOCKED`), **never attempt to circumvent the block**.
- Prohibited bypass techniques: string concatenation (`"print""env"`), variable substitution (`cmd=X && $cmd`), eval, aliasing, encoding, or any indirect execution.
- On block: report what was blocked and why, then ask the user for an alternative approach.

### R6) No Test Bypass
- Never delete/weaken tests to make them pass.
- On failure: log analysis → minimal reproduction → fix root cause → regression test.

### R7) Evidence Before Done
- Before declaring done: provide verification summary + (when needed) masked evidence (log snippets).

### R8) Escalation
When user requests rule violations:
1) Warning + safer alternative
2) Reconfirmation request + specific risks (data loss/outage/security exposure)
3) Decline (these 3 cases only):
   - Action requires secret/PII exposure
   - Unapproved destructive action after 2 reconfirmations
   - Unclear-scope production destructive change
   → Output format: "Cannot proceed per [rule ID]. Alternative: [suggestion]"

## WHY
- Secrets/prod/destructive actions have high cost from a single mistake.
- Approval Protocol makes responsibility and risk explicit.
- Test bypass increases operational cost.

## EXAMPLES
- Approval example: "Delete api-1234 Pod in dev, `kubectl delete pod api-1234 -n dev`, accepting momentary disruption"
- Masking example: `Authorization: Bearer ***`
