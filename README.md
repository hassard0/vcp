# VCP — Verifiable Capability Protocol

> **Zero-trust capability execution for AI agents.** A model may *propose* a tool
> call, but it can never *authorize* one. Authorization comes from a signed,
> content-addressed manifest, a mandatory policy decision, explicit consent, and a
> single-use, proof-bound grant minted by an enforcing **Gateway**.

[![Spec status: Draft RFC](https://img.shields.io/badge/spec-Draft%20RFC-orange)](./SPECIFICATION.md)
[![Protocol revision](https://img.shields.io/badge/revision-2026--06--13-blue)](./CHANGELOG.md)
[![Spec license: CC BY 4.0](https://img.shields.io/badge/spec-CC%20BY%204.0-green)](./LICENSE-SPEC)
[![Code license: Apache 2.0](https://img.shields.io/badge/code-Apache%202.0-green)](./LICENSE)

VCP is a stricter sibling of the [Model Context Protocol](https://modelcontextprotocol.io)
(MCP). MCP's breakthrough is ecosystem simplicity — and that same easy
composability is its security weakness. The MCP spec itself states it cannot enforce
many security principles at the protocol level, and that authorization is optional.

**VCP flips that:** security, provenance, policy, and determinism are *protocol
requirements*, not implementation advice.

```
Model proposes plan
  → Gateway validates manifests & plan
    → Policy authorizes a bounded grant
      → Provider executes within the grant (sandboxed)
        → Gateway validates the signed attestation
          → Model receives a tainted result (never authority)
```

## Why VCP exists — the MCP failure modes it eliminates

| MCP failure mode | VCP control |
|---|---|
| **Tool poisoning** — hidden instructions in tool descriptions | Descriptions are never authority. The Planner gets a Gateway-compiled affordance from a **signed manifest**, never raw Provider text. |
| **Rug pulls** — tool definitions mutate after approval | Identity is the **contract hash**. Any change ⇒ a new capability id ⇒ rejected until re-approved. A silent mutation becomes a visible diff. |
| **Over-trusted local servers** — STDIO runs with the host's privileges | **VCP-Local** sandbox: signed launcher, no inherited env, filesystem/network allowlists, secret broker. Ambient authority only in `dev`. |
| **Token passthrough / confused deputy** | The unit of authority is a **single-use, proof-bound grant** bound to capability + arguments + plan + scope + budget + a holder key. No reusable bearer token to pass through. |
| **SSRF, session hijacking, replay** | **Stateless VCP-HTTP**: one request = one decision, guarded metadata discovery, single-use grants, no implicit sessions. |
| **Stateful-session ambiguity** | No implicit protocol sessions. State is an **explicit, typed, expiring handle**. |
| **Prompt injection via resources** | **Taint labels**: authority never flows from `untrusted_*` data, even when the model is tricked into proposing a bad plan. |
| **Cross-server shadowing / confused deputy** (multi-provider) | **Per-provider scoped credentials** via token exchange + an **on-behalf-of delegation chain** in every grant and audit event; one user approval covers the whole cross-service action (§26). |

## The shape of the idea

```
Capabilities are signed.
Descriptions are not authority.
Models plan; gateways enforce.
Every side effect needs a bounded grant.
Every write has plan/apply semantics.
Every output is tainted until policy says otherwise.
Every state handle is explicit.
Every sensitive call is replayable or explicitly non-deterministic.
Every manifest change is a new identity.
Every production provider is sandboxed, authenticated, and auditable.
```

## Repository layout

```
SPECIFICATION.md   The normative v0.1 spec (RFC-2119). Start here.
schemas/           Normative JSON Schemas for every envelope (manifest, grant, plan,
                   policy request/response, invocation, attestation, audit, discovery).
rfcs/              Open RFCs — the deferred/large ideas, open for discussion.
docs/design/       Design rationale behind v0.1.
CHANGELOG.md       Dated protocol revisions (Keep a Changelog).
GOVERNANCE.md      How VCP is governed and how the RFC process works.
SECURITY.md        Threat model + responsible disclosure.
```

## Conformance ladder

| Level | What it adds |
|---|---|
| **VCP-L0** | MCP-compatible bridge: wraps MCP servers, signs observed schemas, adds policy + audit, marks trust `legacy`. |
| **VCP-L1** | Signed, content-addressed manifests; strict schema validation; no hidden metadata changes. |
| **VCP-L2** | Mandatory auth; per-call proof-bound grants; sandboxing; network/file/secret isolation; policy interface. |
| **VCP-L3** | Plan/apply; dry-run for writes; idempotency keys; replay logs; result attestations; snapshot refs. |
| **VCP-L4** | Transparency registry; reproducible-build provenance; formal policy verification; DLP/data-flow proofs. |

Each level has a **normative security test suite** (12 attack scenarios; see
[SPECIFICATION.md §18](./SPECIFICATION.md#18-normative-security-test-suite)).

## Reference implementations

Reference SDKs and gateways live in
**[`vcp-servers`](https://github.com/hassard0/vcp-servers)** — a **lightweight
client/SDK + MCP bridge** and a **heavy enforcing gateway**, in TypeScript, Python,
Go, and Rust, all driven by shared, language-agnostic conformance vectors.

## Ecosystem

VCP is designed to compose with existing building blocks rather than reinvent them:

- **[`cani`](https://github.com/hassard0/cani)** — a local-first Policy Decision
  Point; a conformant **Policy Authority** for the §6 decision interface.
- **[`mcp-ledger`](https://github.com/hassard0/mcp-ledger)** — append-only audit +
  budget enforcement; a conformant **audit and budget substrate** (grants carry a
  budget; the ledger enforces it).
- **[`prosecco-ai-standards`](https://github.com/hassard0/prosecco-ai-standards)** —
  the AI-interoperability standards directory where VCP is listed.
- **Sigstore / SLSA** inform manifest signing and the L4 transparency registry.
- **OPA / Cedar** satisfy the §6 policy decision shape.
- **OpenTelemetry** is the observability substrate for §20.

## Status & contributing

VCP is a **Draft RFC** — it may change incompatibly until it reaches `Stable`.
Discussion happens in this repo's **Discussions**; normative changes go through the
**[RFC process](./rfcs/README.md)**. See [CONTRIBUTING.md](./CONTRIBUTING.md) and
[GOVERNANCE.md](./GOVERNANCE.md). Found a security-relevant gap? See
[SECURITY.md](./SECURITY.md).

## License

The prose specification (`SPECIFICATION.md`, `docs/`, `rfcs/`) is licensed
**CC BY 4.0**; the `schemas/` directory and any code are **Apache-2.0**. See
[LICENSE-SPEC](./LICENSE-SPEC) and [LICENSE](./LICENSE).
