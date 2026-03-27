# rules/10-stack-kotlin.md — KOTLIN / CODE STYLE

## APPLICABILITY: conditional — Kotlin projects only. Skip if the project uses a different language.

## SUMMARY (READ FIRST)
- Simplicity first (KISS/YAGNI): no unnecessary abstraction/generalization.
- Review-friendly diffs: do not mix functional changes with formatting/renaming/large moves.
- Follow naming/function size/complexity limits for readability.
- New dependencies only "when truly needed" + license/operational cost/exit plan included.

## RULES
### R1) Simple-first
- Given same functionality, prefer the simpler design.
- Only increase complexity with evidence (profiling/requirements).

### R2) Naming
- Functions/methods: `verb + noun` (e.g., `fetchUser`, `validateToken`)
- Boolean: `is/has/can/should` prefix
- Collections: plural form (`users`, `userIds`)

### R3) Functions / Methods
- Single responsibility (SRP)
- 30-50+ lines → split candidate
- Nesting depth 3+ → simplification candidate
- 5+ branches → structuring candidate (strategy/table-driven)

### R4) Error handling / Logging (least disclosure)
- Empty catch/ignore prohibited (do not swallow failures).
- Token/password/session/cookie/PII logging prohibited.
- No infinite duplicate error output in loops (rate-limit/aggregate).

### R5) Kotlin conventions (baseline)
- Methods: 30 lines recommended max, cyclomatic complexity 10 or below
- Extension functions: use when they improve readability
- Expression body: use when readability is maintained
- Naming: Class PascalCase, Method camelCase, Constant UPPER_SNAKE_CASE

### R6) Library policy + License
- If solvable with standard library, do not add new dependencies.
- New dependency PR must include:
  - Why needed / alternatives / risks / operational cost / exit plan
  - License summary + transitive included
- Allowed: MIT / Apache-2.0 / BSD / ISC
- Conditional (approval needed): MPL / EPL / CDDL (file-level copyleft)
- Prohibited (default): GPL / AGPL (exceptions require explicit approval)

### R7) Reference discovery priority
README → CONTRIBUTING → CI config (`.github/workflows`, `Jenkinsfile`) → build tool task list (`gradle tasks`, `npm scripts`) → if all unclear, ask

## EXAMPLES
- BAD: Adding "might use later" interfaces/generics
- GOOD: Minimal structure satisfying current requirements + tests
