# rules/04-repo-boundaries.md — REPO STRUCTURE & BOUNDARIES

## APPLICABILITY: mandatory — always active regardless of stack or project

## SUMMARY (READ FIRST)
- Respect "modify-with-care / read-only / restricted-access" boundaries for safety.
- Production config / infra / deploy files are not modified by default.
- Secret/credential file contents are never viewed (existence check only, true/false level).

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

## WHY
- Prod/deploy/secret boundaries have high damage and difficult recovery on mistakes.

## EXAMPLES
- BAD: Suggesting/applying modifications to `application-prod.yml`
- GOOD: Read prod config for context only; if changes needed, request approval/procedure
