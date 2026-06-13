# Changelog

All notable changes to the VCP specification are documented in this file.

The format is based on [Keep a Changelog](https://keepachangelog.com/en/1.1.0/).
VCP does not use semantic version numbers; instead, the protocol is identified by a
**dated revision** (`YYYY-MM-DD`) carried in the `vcp` field and in the
`SPECIFICATION.md` header. Until the protocol reaches `Stable`, dated revisions MAY
change incompatibly. Each release below corresponds to a protocol revision.

The machine-readable schemas in [`schemas/`](./schemas/) are versioned alongside the
prose and ship in the same revision.

## [Unreleased]

Collecting feedback on the Draft RFC; open RFCs tracked in [`rfcs/`](./rfcs/).

### Clarified (implementation-informed; no behavior change)

Refinements drawn from implementing the spec in four languages — they pin down what
implementers previously had to infer, so independent implementations stay consistent.
No conformance vector changed.

- **§1.3 The Safety Contract** — the ten normative invariants (INV-1…INV-10) every
  conformant implementation must preserve, stating the intent (a model can drive tool
  calls without being trusted to do so safely) as a testable property.
- **§2.1 The Gateway evaluation pipeline** — the normative *ordered* sequence a Gateway
  runs for one call, so two implementations deny the same call with the same
  `reason_code`. Doubles as the build checklist.
- **§3.1 Hash-exclusion table** + number-handling rule — removes the most common cause
  of cross-implementation hash/signature mismatches.
- **§4.1 Exact contract construction** — the identity algorithm pinned to the exact
  eight fields, with the published ground-truth vector as the reference.
- **§7.1 Grant verification order and time semantics** — fixed check order and the
  half-open expiry interval (`now < expires_at`).
- **§26.2** — a widening attenuation is denied with `AUDIENCE_MISMATCH` (pending a
  possible future `SCOPE_WIDENED`).
- **Appendix E — Minimal conformant Gateway** — the smallest L2 loop (8 steps) that
  upholds the invariants; everything else is opt-in and off by default.
- Fixed two stale cross-references in §1.2; moved §27 ahead of the appendices.

## [2026-06-13] — MCP-informed hardening

Protocol revision `2026-06-13`. Additive expansion folding in the good ideas MCP
converged on (per its 2026-07-28 release candidate) and closing gaps critics
identified, without changing any v0.1 conformance vector.

### Added

- **§21 Asynchronous Execution (Tasks)** — long-running work as `task` state handles
  ("call-now, fetch-later"); the originating grant bounds the task's lifetime and
  authority, and **cancellation revokes the grant**. Parallels MCP Tasks
  (SEP-1391/1686) with the added security binding. New `task` capability kind and
  `schemas/task.schema.json`.
- **§22 Interface Capabilities** — signed, sandboxed user-interface surfaces (VCP's
  hardened answer to MCP Apps / SEP-1865): content-addressed and **mandatorily
  signed** UI artifacts, enforced CSP, and **every UI-initiated action re-enters the
  full grant pipeline**. New `interface` capability kind + manifest `interface` block.
- **§23 Reason Code Registry** — a normative, remediable error/denial vocabulary,
  closing MCP's missing-error-standard gap.
- **§24 Extensions and Feature Lifecycle** — reverse-DNS extensions negotiated per
  request, an `Active → Deprecated → Removed` policy with a 12-month minimum window,
  and conformance-gated promotion to Stable.
- **§25 Caching and Distributed Tracing** — `ttl_ms`/`cache_scope` hints valid only
  while the content hash verifies, and W3C Trace Context on invocations and audit
  events.
- **§26 Multi-Provider Composition and On-Behalf-Of Delegation** — a Gateway fanning
  out to many upstream APIs as a first-class, secure, low-friction operation:
  per-provider credential brokering via OAuth Token Exchange (RFC 8693) bound to a
  resource indicator (RFC 8707), an explicit on-behalf-of **delegation chain** in
  every grant and audit event, and **one user approval covering the whole
  cross-service blast radius** (many scoped grants, not many prompts).
- **§27 Environment and Workload Attestation** — optional, low-friction runtime
  integrity: attest *what an actor is* (gateway/provider/agent) distinct from §9's
  *what a call did*. **Off by default**, attest-once / reference-many, and layered — a
  cheap signed `statement` tier (Ed25519 only) for L2/L3 and a hardware `tee` tier
  (RFC 9334 RATS) for L4. New `schemas/environment-attestation.schema.json`,
  `effects.requires_attestation` flag, `grant.attestation_ref`, audit
  `attestation_ref`, reason code `ATTESTATION_REQUIRED`, security test 19, RFC 0008.
- **§28 Command and CLI Capabilities (`VCP-CLI`)** — first-class, safe coverage for
  LLM agents that act through the command line, both operating ordinary existing CLIs
  and using the CLI as a configuration medium. A `command` capability is
  content-addressed and **argv-typed**: the Gateway `exec`s the binary with a resolved
  argv array and **never** uses a shell, so shell injection (CWE-78) is impossible by
  construction; the exact argv is grant-bound; the sandbox (Landlock/seccomp/Seatbelt,
  microVM at L4) is declared in a signed manifest, not host-local settings; command
  output is tainted so attacker-controlled text cannot authorize the next command; and
  a **command bridge** wraps existing CLIs by pinning the binary digest and a signed
  subcommand allowlist. Declarative/dry-run/GitOps bias for configuration changes. New
  `command` capability kind + manifest `command` block, `command` conformance vector,
  security tests 20–22, RFC 0009 (sandbox tiers).
- **§15 hardening** — mandatory `Vcp-Operation` routing header, OAuth `iss` validation
  per RFC 9207 (mix-up defense), issuer-bound credentials.
- New RFCs **0005** (MCP Apps `ui://` interop), **0006** (async task resumability),
  **0007** (on-behalf-of token exchange profile).
- New schema fields: `grant.delegation_chain` / `grant.token_exchange`,
  `audit-event.delegation_chain` / `credential_audience`, `invocation.trace`,
  manifest `capability.interface`, discovery caching hints.
- **Appendix C** — a head-to-head VCP-vs-MCP (2026-07-28 RC) comparison.

## [2026-06-12] — Initial Draft

Protocol revision `2026-06-12`. First public Draft RFC of the Verifiable Capability
Protocol.

### Added

- Zero-trust thesis: the model **plans**, the Gateway **enforces**. A Planner may
  propose a capability call but can never authorize one; authority lives only in the
  Gateway.
- Signed, content-addressed **capability manifests**: identity is the hash of a
  capability's contract, so any change to schema, effects, sandbox, or code digest
  produces a new capability identity and a visible diff (rug-pull defense).
- Canonical JSON pinned to **RFC 8785 (JCS)** + **SHA-256**, with in-band `alg`
  and the `vcp:<type>:<name>@sha256:<hex>` identifier form.
- **Single-use, proof-bound grants**: audience-bound (to `capability_id`),
  argument-bound, plan-bound, time-bound (RECOMMENDED TTL ≤ 300s), scope-bound
  (network/resource/budget), proof-of-possession bound (DPoP-style), and
  attenuable-only (narrow, never widen).
- **Mandatory policy decision interface** (§6): a typed allow/deny/challenge request
  and response shape; engine-agnostic; denials carry structured, remediable
  `reason_code`s.
- **Plan/apply** workflow with dry-run diffs and Provider-signed **attestations**;
  approval binds to the exact `plan_hash`.
- **Effect classes** (`read-only`, `propose-only`, `write-idempotent`,
  `write-reversible`, `write-irreversible`) and **determinism classes** (`pure`,
  `snapshot-read`, `external-read`, `idempotent-write`, `nondeterministic`).
- **Data taint labels** and a non-authoritative-injection model: authority never
  flows from `untrusted_*` data; the Gateway tracks labels across capability
  boundaries and presents them to policy as `data_flows`.
- **Explicit, typed, expiring state handles**: no implicit protocol sessions;
  cross-call state is a scoped `state` capability with an expiry.
- Transport and bridging profiles: **VCP-Local** (signed launcher, sandboxed,
  no inherited environment, filesystem/network allowlists, secret broker),
  **VCP-HTTP** (stateless-by-default production profile), and **VCP-Bridge**
  (wraps unmodified MCP servers, strips raw descriptions, pins observed schemas).
- **Conformance levels VCP-L0..L4**, from MCP-compatible bridge through
  high-assurance enterprise.
- A **12-item normative security test suite** (§18) covering tool poisoning, rug
  pulls, cross-server shadowing, SSRF, token passthrough, session replay, filesystem
  exfiltration, hidden-argument exfiltration, malicious sampling, policy bypass via
  model output, schema confusion, and nondeterministic-replay mismatch.
- Ecosystem cross-references: `cani` (conformant Policy Decision Point),
  `mcp-ledger` (conformant audit + budget substrate), and listing in
  `prosecco-ai-standards`.

[Unreleased]: https://github.com/hassard0/vcp/compare/2026-06-12...HEAD
[2026-06-12]: https://github.com/hassard0/vcp/releases/tag/2026-06-12
