# rules/11-stack-reactive.md — WEBFLUX / REACTOR / CONCURRENCY

## SUMMARY (READ FIRST)
- Production blocking calls (`block*`) are prohibited in principle (exception: tests only).
- Blocking I/O must be isolated to `Schedulers.boundedElastic()` or equivalent.
- `onErrorResume` with fallback values requires WARN+ logging + explicit policy (when/what/impact).
- Event loop / request thread occupying work is prohibited (if unavoidable: isolate + justify + test).
- traceId/MDC context는 Coroutine 혼용 및 fire-and-forget 시 반드시 명시적으로 전파해야 한다.

## RULES
### R1) No blocking in prod
- Prohibited: `block()`, `blockFirst()`, `blockLast()` (production paths)
- Test code: `block()` allowed when needed

### R2) Scheduler policy
- Blocking-capable I/O: isolate to `Schedulers.boundedElastic()`.
- Separate CPU work and I/O work execution resources.

### R3) Error handling policy
- When using `onErrorResume` for "fallback value return":
  - WARN+ level logging
  - Explicit fallback policy (condition/fallback value/impact)

### R4) Async / Concurrency
- Event loop / request-handling thread occupying blocking I/O prohibited
- If unavoidable: dedicated thread pool/work queue isolation + reason/impact + test evidence

### R5) TraceId / MDC Context 전파
- Reactor의 `Mono`/`Flux` 체인에서 traceId(MDC)는 자동 전파되지 않는 경우가 있으므로 주의.
- **Coroutine 혼용 시**: `ReactorContext`와 Coroutine `Context`는 자동 연결되지 않는다. Coroutine으로 전환할 때 반드시 `ReactorContext`에서 traceId를 꺼내 Coroutine `Context`에 명시적으로 전달해야 한다.
  - `mono { ... }.contextWrite(ctx)` 패턴으로 Reactor → Coroutine 방향 전파
  - Coroutine → Reactor 방향은 `Mono.deferContextual`로 context를 받아 전달
- **Fire-and-forget (`subscribe()`)**: `subscribe()`는 새 subscription을 생성하므로 상위 Reactor Context(traceId 포함)가 끊긴다.
  - `subscribe()` 호출 전 `.contextWrite()`로 현재 context를 명시적으로 주입
  - 가능하면 `subscribe()` 대신 `.then()`/`flatMap()`으로 체인에 포함시켜 context가 자연 전파되도록 한다
- traceId 전파가 누락되면 로그 추적이 불가능해지므로, 신규 비동기 분기점마다 context 전파 여부를 반드시 확인한다.

## EXAMPLES
- BAD: `.block()` call in request handling flow
- GOOD: Move blocking I/O to boundedElastic, keep caller non-blocking
- BAD: `someMono.subscribe()` — traceId 유실
- GOOD: `someMono.contextWrite(ctx).subscribe()` 또는 체인 내 `.flatMap()`으로 포함

## SELF-CHECK (before final response)
- No event-loop blocking in request path?
- Blocking I/O isolated to boundedElastic?
- traceId/MDC context propagated at every async boundary?
