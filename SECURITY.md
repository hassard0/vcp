# Security Policy

This document governs the **VCP specification repository**. Vulnerabilities in
reference implementations belong in
[`vcp-servers`](https://github.com/hassard0/vcp-servers), not here.

## Reporting a vulnerability

If you believe you have found a security-relevant flaw **in the protocol design** —
a way that a conformant VCP implementation could still be coerced into an
unauthorized action, data movement, or authority escalation — please report it
privately.

- Use **GitHub private vulnerability reporting** ("Report a vulnerability") on this
  repository, or contact the maintainers via GitHub.
- **Do not** open a public issue for a suspected vulnerability.
- Please include: the section(s) of `SPECIFICATION.md` involved, the attacker model
  you assume, and a concrete sequence that achieves the unauthorized outcome.

We aim to acknowledge reports within **5 business days** and to work with you on
coordinated disclosure. Design-level findings that survive review become either a
spec fix (a new dated revision) or a tracked RFC.

## Threat model charter

VCP exists to neutralize the documented MCP failure modes. Each is mapped to the
control that mitigates it; the controls are normative requirements in
`SPECIFICATION.md`.

| Threat | Primary VCP control | Spec |
|---|---|---|
| **Tool poisoning** (hidden instructions in descriptions) | Descriptions are never authority; the Planner receives a Gateway-compiled affordance from a signed manifest | §5, §13 |
| **Rug pulls** (definitions mutate after approval) | Content-addressed identity = contract hash; any change is a new, unapproved capability | §4 |
| **Over-trusted local servers** | VCP-Local sandbox: signed launcher, no inherited env, fs/network allowlists, secret broker | §14 |
| **Token passthrough / confused deputy** | Single-use, proof-bound grants bound to capability + arguments + plan + scope | §7 |
| **SSRF (metadata discovery)** | Guarded discovery fetches; internal addresses refused | §15, §17 (test 4) |
| **Session hijacking / replay** | Stateless VCP-HTTP; single-use grants; explicit state handles | §7, §15 |
| **Stateful-session ambiguity** | No implicit sessions; typed, expiring state handles | §5.1 |
| **Prompt injection via resource data** | Taint labels; authority never flows from `untrusted_*` data | §12 |
| **Covert sampling** | Delegated inference under policy, with redaction and budgets | §13.3 |

## Conformance security suite

A conformant implementation **MUST** pass the 12-test normative security suite in
[`SPECIFICATION.md` §18](./SPECIFICATION.md#18-normative-security-test-suite):
tool poisoning, description rug pull, cross-server shadowing, SSRF in metadata
discovery, token passthrough, session replay, filesystem exfiltration, hidden
argument exfiltration, malicious sampling request, policy bypass through model
output, schema confusion, and nondeterministic replay mismatch. Each test asserts
the attack is **rejected, contained, or made auditable**. Executable conformance
vectors are published under `vcp-servers/conformance`.

## Scope and non-goals

VCP **reduces** but does not **eliminate** prompt injection: it removes injection's
*authority*, not its *existence*. A Planner can still be tricked into proposing a bad
plan; VCP's guarantee is that the Gateway rejects any plan whose authority derives
from tainted data, and that no side effect occurs without a bounded, approved grant.
Operators remain responsible for classifying data, minimizing secrets that reach the
Planner, and reviewing audit trails.
