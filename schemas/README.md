# VCP Schemas

Normative JSON Schemas (Draft 2020-12) for every VCP envelope. These ship with each
dated protocol revision and are authoritative alongside the prose in
[`../SPECIFICATION.md`](../SPECIFICATION.md).

| Schema | Purpose | Spec section |
|---|---|---|
| `manifest.schema.json` | Signed capability manifest; identity = `sha256(JCS(contract))` | §4, §5.2 |
| `grant.schema.json` | Single-use, proof-bound authorization grant | §7 |
| `plan.schema.json` | Planner-proposed plan; `plan_hash` is bound to approval | §9 |
| `policy-request.schema.json` | Gateway → Policy Authority decision request | §6 |
| `policy-response.schema.json` | Policy Authority → Gateway decision | §6 |
| `invocation.schema.json` | Gateway → Provider invocation envelope | §8 |
| `attestation.schema.json` | Provider → Gateway result + signed attestation | §9, §10 |
| `audit-event.schema.json` | Signed, OpenTelemetry-compatible audit event | §20 |
| `discovery.schema.json` | Provider + capability discovery documents | §5, §16 |

## Conventions

- All hashing and signing operate on **JCS** ([RFC 8785](https://www.rfc-editor.org/rfc/rfc8785))
  canonicalized bytes, then SHA-256 (`sha256:<hex>`). See §3.
- Content-addressed ids match `^vcp:cap:[A-Za-z0-9._-]+@sha256:[0-9a-f]{64}$`.
- `additionalProperties: false` is required on argument objects (schema-confusion defense).
- The default signature algorithm is `Ed25519`; `alg` is always carried in-band.

Reference implementations and executable conformance vectors that validate against
these schemas live in [`vcp-servers`](https://github.com/hassard0/vcp-servers).
