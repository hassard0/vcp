# Contributing to VCP

Thanks for your interest in the **Verifiable Capability Protocol**. VCP is a spec-first
project: this repository holds the normative specification, its schemas, and the RFC
backlog. Reference implementations live in a separate repository,
[`vcp-servers`](https://github.com/hassard0/vcp-servers).

Please read the [Governance](./GOVERNANCE.md) document for how decisions are made and
how the RFC process works, and the [Code of Conduct](./CODE_OF_CONDUCT.md) for
expected behavior. Security issues follow the separate process in
[SECURITY.md](./SECURITY.md).

## Where things go

- **Specification prose** — `SPECIFICATION.md`, `docs/`
- **Normative schemas** — `schemas/` (versioned with the prose)
- **RFCs** — `rfcs/` (see [`rfcs/README.md`](./rfcs/README.md))
- **Reference implementations, SDKs, gateways** —
  [`vcp-servers`](https://github.com/hassard0/vcp-servers) (a separate repo)
- **Conformance test vectors** —
  [`vcp-servers/conformance`](https://github.com/hassard0/vcp-servers) (language-agnostic JSON)

## Types of contributions

- **Spec feedback and open-ended questions** → open a **GitHub Discussion**. Use this
  for "have you considered…", design debates, and anything not yet a concrete defect.
- **Spec bugs** (ambiguity, inconsistency, contradiction, an error in an example or a
  hash) → open an **Issue** using the `spec-bug` template.
- **Conformance gaps** (the spec is unimplementable, or a security test in §18 is
  missing, wrong, or untestable) → open an **Issue** using the `conformance-gap`
  template.
- **Normative changes** (anything that changes a MUST/SHOULD/REQUIRED requirement, an
  actor's responsibilities, an envelope's wire shape, or a schema) → these **require
  an RFC**. See below.
- **Reference implementations** → contribute to
  [`vcp-servers`](https://github.com/hassard0/vcp-servers), not this repo.
- **Conformance vectors** → new or improved language-agnostic JSON vectors are very
  welcome and land in `vcp-servers/conformance`.

## How to open a good spec issue

A useful spec issue is precise enough that a maintainer can act on it without
guessing:

1. **Section reference** — cite the exact section (e.g. "§7 Grants" or "§5.2
   Manifest") and, where helpful, the line or the JSON field.
2. **What's wrong** — state the inconsistency, ambiguity, or error concretely. If two
   parts of the spec disagree, cite both.
3. **Why it matters** — does it create an interoperability hazard, a security gap, or
   just confusion? This helps prioritize.
4. **Suggested fix** — even a rough proposed wording change accelerates resolution. If
   the fix changes a normative requirement, expect it to move to the RFC process.

The `spec-bug` and `conformance-gap` issue templates prompt for these fields.

## Normative changes need an RFC

This is the project's central rule. If your change would alter what an implementation
**MUST**, **MUST NOT**, **SHOULD**, **SHOULD NOT**, or is **REQUIRED** to do — or
would change a schema in `schemas/` — it cannot be merged as a plain pull request. It
needs an accepted RFC. The full lifecycle (Draft → Discussion → Last Call →
Accepted/Rejected → Final) is in [GOVERNANCE.md](./GOVERNANCE.md) and
[`rfcs/README.md`](./rfcs/README.md). To start, copy
[`rfcs/0000-template.md`](./rfcs/0000-template.md), open a Discussion, and submit a PR.

Editorial fixes, clarifications that do not change meaning, examples, and tooling can
go straight to a pull request and are merged by lazy consensus.

## Style guide

- **RFC 2119 keywords only in normative text.** Use MUST/MUST NOT/SHOULD/SHOULD
  NOT/REQUIRED/RECOMMENDED/MAY/OPTIONAL only where you intend a real conformance
  requirement, and only in all-capitals, per
  [RFC 2119](https://www.rfc-editor.org/rfc/rfc2119) and
  [RFC 8174](https://www.rfc-editor.org/rfc/rfc8174). Do not use them casually in
  prose, examples, or this kind of process doc.
- **Canonicalize hashed examples with JCS.** Any example that shows a hash, signature,
  `contract_hash`, `argument_hash`, or `plan_hash` must be consistent with
  [RFC 8785 (JCS)](https://www.rfc-editor.org/rfc/rfc8785) + SHA-256 as defined in §3.
  Do not hand-wave a hash that contradicts the canonicalization rules.
- **Keep `schemas/` in sync with the prose.** If a prose change touches an envelope or
  a manifest field, update the corresponding schema in the same change set. A schema
  and the text that describes it must never disagree.
- **Use the exact actor and section names.** Host, Planner, Gateway, Capability
  Provider, Policy Authority, Transparency Registry — and the existing section numbers.
  Consistency is a security property here, not just a cosmetic one.

## Developer Certificate of Origin (sign-off)

All commits must be signed off under the
[Developer Certificate of Origin](https://developercertificate.org/). This is a
lightweight attestation that you have the right to submit the contribution. Add a
sign-off line to each commit:

```
Signed-off-by: Your Name <you@example.com>
```

You can add it automatically with `git commit -s`. Pull requests whose commits are not
signed off will be asked to amend before merge.

## Pull request checklist

- [ ] The change is in scope for this repo (spec/schemas/RFCs, not implementation).
- [ ] Normative changes are backed by an accepted (or in-flight) RFC.
- [ ] Any touched schema in `schemas/` is updated and consistent with the prose.
- [ ] Hashed examples are JCS-consistent (§3).
- [ ] RFC 2119 keywords appear only in normative text.
- [ ] All commits are `Signed-off-by` (DCO).

## See also

- [GOVERNANCE.md](./GOVERNANCE.md) — roles, decision-making, RFC lifecycle, versioning
- [CODE_OF_CONDUCT.md](./CODE_OF_CONDUCT.md) — expected behavior and enforcement
- [SECURITY.md](./SECURITY.md) — vulnerability reporting and the threat-model charter
