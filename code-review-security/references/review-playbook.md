# Risk-Oriented Review Probes

Use only the probes that match the touched functionality. The objective is to establish **reachability, violated invariant, and user impact**, not to label every code smell as a bug.

| Workflow / boundary | Questions that reveal real failures | Useful targeted test |
| --- | --- | --- |
| Authentication and object ownership | Is the current actor authorized for *this exact object/action*, not just logged in? Can identifiers be swapped? Does background processing preserve the actor/tenant? | User A requests user B's object ID; direct API and async path must deny. |
| Roles and privilege transitions | Can a user promote themselves, re-enable a disabled account or call an admin path through an alternate endpoint? Do caches contain stale roles? | Downgrade/revoke permission, then retry action with previous session/token. |
| Business rules and state machines | Which transitions must never occur? Are preconditions rechecked at commit time? Can canceled/expired/deleted items be reused? | Attempt disallowed state transition, then verify persistent state remains unchanged. |
| Money, quotas, dates and calculations | Are amounts and units consistent? Is rounding deterministic? Can negative/zero/overflow inputs, timezone/DST or boundaries alter a charge/limit? | Check boundary values, rounding and timezone-dependent dates against domain rules. |
| Writes, deletion and external side effects | Can a failure after one side effect leave partial state? Are destructive actions scoped and recoverable? Are compensating actions sound? | Inject failure between database write and external API, verify recovery and user-visible status. |
| Duplicate/concurrent operations | Can multiple requests pass the same precondition and both mutate state? Is idempotency enforced where retries occur? | Send two parallel or retried requests; verify exactly-once effect or valid deduplication. |
| Validation and serialization | Do API/client/schema layers disagree? Are missing, null, unexpected enum values, Unicode or huge payloads handled without corruption? | Fuzz representative boundary/invalid inputs through real entry point and inspect persisted outputs. |
| Injection and untrusted output | Does user-controlled data reach SQL, shell, template, HTML, path, URL, deserialization or query builders unsafely? | Demonstrate dangerous interpretation with harmless local sentinel input; verify encoding or safe API use. |
| Privacy and logging | Do errors, metrics, caches, audit logs, exports and responses reveal another person's records, tokens or confidential content? | Force an error and inspect sanitized logs/responses with synthetic sensitive fields. |
| Remote requests and files | Could user-controlled URLs reach internal resources? Can paths escape intended storage? Are file type/size/permission checks enforced? | Harmless internal-target/relative-path probe in local isolation; assert rejection. |
| Availability and abuse | Can unauthenticated requests trigger unbounded work, huge memory allocation, excessive retries or unchecked provider costs? | Controlled resource limit test, verify timeout, bounded loops and backpressure. |
| Compatibility and UX correctness | Could changed defaults, API response shape or frontend fallbacks silently show success for failure, incorrect data or stale permissions? | Compare old/new response contracts; simulate timeout/offline and verify accurate user-facing state. |

## Triage rules

1. **Reproducible and reachable:** provide exact path, preconditions, observable wrong behavior and a feasible fix.
2. **Credible but not verified:** list under *Needs verification* with the missing check (e.g., whether a middleware enforces ownership).
3. **Not actionable or already mitigated:** omit from findings. Avoid duplicating one root cause across multiple endpoints unless impacts and fixes differ.
4. **No issues found:** specify the reviewed files and meaningful test coverage, plus significant blind spots. Absence of findings is not a security guarantee.

## Useful outcome-focused invariants

- A user can mutate only resources they are authorized to mutate.
- A failed operation does not claim success or leave an unacknowledged irreversible side effect.
- An operation retried or executed concurrently does not cause an unintended duplicate effect.
- Every externally visible state transition preserves domain constraints.
- Invalid or malicious input cannot reach a sensitive sink without correct validation/escaping.
- Private data does not cross tenant/user boundaries, including errors, caches and exports.
- Existing valid callers continue to receive the documented behavior or an explicit migration path.
