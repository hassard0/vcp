# RFC 0006: Async Task Resumability and Replay

- **RFC:** 0006
- **Title:** Resumability, Replay, and Result Durability for Tasks
- **Author:** hassard0
- **Status:** Draft
- **Created:** 2026-06-13
- **Discussion:** (open a Discussion thread and link it here)
- **Affected sections / schemas:** §10 (determinism), §21 (tasks), `task.schema.json`

## Summary

Specify how a `task` (§21) survives connection faults and Gateway restarts: how
`tasks/get` is made idempotent and resumable, how a result remains fetchable
("fetch-later") with a stable `result_hash`, and how a task's determinism evidence
(§10) is recorded so the task is replayable or explicitly marked non-deterministic.

## Motivation

§21 establishes the stateless task state machine and binds a task to its grant. It
does not yet pin the durability guarantees: how long a completed result stays
fetchable, what happens to an in-flight task when the Gateway instance that started
it goes away, and how an `external-read`/`nondeterministic` task carries replay
evidence. These are the details that make "call-now, fetch-later" reliable in
production behind a load balancer.

## Detailed design

- **Resumable status.** `tasks/get` is a pure, idempotent read keyed by `task_id`;
  any Gateway instance can answer it from shared task storage. There is no implicit
  session (§5.1).
- **Result durability.** A `succeeded` task's result + attestation **MUST** remain
  fetchable until at least the task's `expires_at`. `result_hash` **MUST** be stable
  across fetches; a Gateway **MUST NOT** recompute a different result on a second
  fetch.
- **Restart semantics.** If the executing instance fails, the task **MUST** either
  resume from durable state or transition to `failed` with `GRANT_EXPIRED`/a specific
  reason; it **MUST NOT** silently drop. A `write-idempotent` task **MAY** be retried
  under the same `idempotency_key`.
- **Replay evidence.** A task whose determinism class is `external-read` **MUST**
  record `observed_external_refs`; a `nondeterministic` task **MUST** record
  record/replay evidence (§10) at completion, else it is non-conformant
  (`REPLAY_EVIDENCE_MISSING`).

## Normative changes

- Add §21 subsections for resumability, durability windows, and restart semantics.
- Possibly add a `durable_until` field to `task.schema.json` distinct from
  `expires_at` (result retention vs. authorization lifetime).

## Backward compatibility

Additive within the task model. Synchronous capabilities are unaffected.

## Security considerations

Durable results are a data-at-rest surface: stored results **MUST** follow the same
no-secrets and taint rules as inline results, and access to `tasks/get` **MUST** be
subject-scoped (`SUBJECT_MISMATCH`). A resumed task **MUST NOT** outlive its grant's
authorization window without re-authorization.

## Alternatives considered

- **Tie result retention to the grant TTL only**: too short for genuine long-running
  work; hence the separate `durable_until`.

## Open questions

- Should task storage be a normative interface (like §6 policy) or implementation-
  defined?
