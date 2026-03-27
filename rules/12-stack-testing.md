# rules/12-stack-testing.md — TESTING / BUILD / VERIFICATION

## APPLICABILITY: conditional — Kotlin + Gradle projects only. Testing principles (R1, R6) are universal; commands and conventions (R3~R5, R7) are stack-specific.

## SUMMARY (READ FIRST)
- Behavior changes require test updates/additions (especially business logic/data/permissions/contracts).
- On failure: log analysis → minimal reproduction test → fix → re-verify. No test bypass.
- PR/Release level: `./gradlew clean build` is the baseline verification.
- Lint/format: use only "existing tasks" (no forced introduction).

## RULES
### R1) When tests are required
- Required:
  - Business logic/flow changes
  - Data model/schema changes
  - Permission/policy logic changes
  - External API/contract semantic changes
- Recommended:
  - Config/logging/metrics/docs changes (promote to Required if behavior/operational impact)
- Minimum:
  - Behavior-unchanged refactoring → existing tests pass + justification

### R2) Promotion examples
- Logging level/filter change may miss failure detection (ERROR/WARN) → Required
- Log format change may break parsers/tools/alerts → Required
- Config change affects timeout/retry/thread pool/queue size runtime behavior → Required
- High-cardinality labels/fields (e.g., userId) in metrics/logs → Required (performance/cost impact)

### R3) Verification commands (Gradle)
- Full verify (PR/Release baseline): `./gradlew clean build`
- Unit tests: `./gradlew test`
- Run locally: `./gradlew bootRun`
- Profile run: `./gradlew bootRun --args='--spring.profiles.active=dev'`
- Lint/Format:
  - Check with `./gradlew tasks` first
  - Use `ktlintCheck` or `spotlessCheck` if available
  - If neither exists: report "not configured" and skip

### R4) Testing conventions
- Test naming:
  - Class: `UserServiceTest`
  - Method: `` `backtick style allowed for test names` ``
- Coroutines testing: use `runTest` (avoid `runBlocking`)
- Assertions: `org.junit.jupiter.api.Assertions` or `kotlin.test` (`assertEquals`, `assertTrue`, `assertNotNull` 등)
  - `assert()` (Kotlin stdlib) 사용 금지 — `-ea` JVM 플래그 없으면 실행되지 않아 테스트가 항상 통과할 수 있음
  - 반드시 `assertEquals`, `assertTrue`, `assertNotNull`, `assertNull` 등 명시적 assertion 함수 사용
- Parameterized-style testing:
  - 동일 로직에 대해 입력만 다른 반복 테스트는 개별 `@Test`로 분리하지 않고, 하나의 테스트에서 `forEach`로 순회한다
  - expected map/list를 정의하고 `forEach`로 검증 — Java의 `@ParameterizedTest`와 동일한 목적

### R5) Multi-module testing
- Affected module test: `./gradlew :module-name:test`
- CI full verification: `./gradlew clean build` (required before PR merge)

### R6) Failure handling
- On failure: log analysis → minimal reproduction test (if possible) → fix → regression test
- Never delete/weaken tests to make them pass.

### R7) Evidence format
- Build/Test results summary (success/failure, key numbers)
- Log snippets shared with `***` masking

## EXAMPLES
- BAD: Deleting failing tests to pass build
- GOOD: Pin failure cause with reproduction test, then fix and verify regression
- BAD: `assert(result.isNotEmpty()) { "should not be empty" }` — `-ea` 없으면 무시됨
- GOOD: `assertTrue(result.isNotEmpty())` / `assertNotNull(result.temperature)`
- BAD: 입력만 다른 동일 검증을 `@Test` 3개로 분리
- GOOD: expected map + `forEach`로 하나의 `@Test`에서 전체 케이스 순회 검증
