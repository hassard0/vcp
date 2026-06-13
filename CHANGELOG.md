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
