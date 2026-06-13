# RFC 0004: DLP / Data-Flow Proofs

- **RFC:** 0004
- **Title:** Provable Data-Flow Labeling (DLP Proofs)
- **Author:** hassard0
- **Status:** Draft
- **Created:** 2026-06-12
- **Discussion:** (open a Discussion thread and link it here)
- **Affected sections / schemas:** §12 (taint), §15 (data-flow-aware policy), conformance L4

## Summary

Extend the §12 taint model so that a Gateway can **prove** — not merely assert — that
a given result obeyed the declared data-flow policy. Today, taint labels are tracked
by the Gateway and trusted by downstream consumers. This RFC adds verifiable evidence
that the labels were honored end to end.

## Motivation

§12 makes injection non-authoritative by tracking labels and rejecting plans whose
authority derives from `untrusted_*` data. §15's data-flow-aware policy can block a
`confidential → external` movement. But the *evidence* that a flow was honored lives
only in the Gateway's own bookkeeping. For regulated deployments (DLP, data
residency), an auditor needs cryptographic evidence, independent of trusting the
Gateway's word.

## Detailed design

- **Labeled lineage.** Each datum carries a signed lineage record: its label, its
  source capability, and the transform that produced it. Derived data inherits the
  most restrictive source label (already required by §12); this RFC makes the
  inheritance step attestable.
- **Flow attestation.** When a capability emits a result, its attestation (§9) is
  extended with a `data_flow_proof`: the set of input lineage hashes consumed and the
  output label, signed by the Provider and counter-checked by the Gateway against the
  policy decision.
- **Auditor verification.** An independent auditor, given the audit log (§20) and the
  flow attestations, can replay the label algebra and confirm no policy-forbidden
  flow occurred — without trusting the Gateway's assertion.
- **Redaction proofs.** For delegated inference (§13.3) where data is redacted before
  leaving, the proof attests that the redaction transform was applied to the declared
  fields.

## Normative changes

- Extend the attestation schema with an optional `data_flow_proof` block.
- Add §12.x defining the label algebra precisely enough to be independently
  replayable.
- Add an L4 conformance test: an auditor detects a Gateway that allowed a
  `confidential → external` flow despite a policy forbidding it.

## Backward compatibility

Additive, L4-scoped. The base §12 taint model is unchanged for L0–L3.

## Security considerations

Strengthens auditability and moves trust from "the Gateway says so" to "the evidence
shows so". Does not replace runtime enforcement — it makes enforcement *checkable
after the fact*. Lineage records must themselves avoid leaking the sensitive payload;
they carry hashes and labels, not content.

## Alternatives considered

- **Gateway-asserted flows** (status quo): adequate for many deployments, insufficient
  for regulated/audited ones.
- **Information-flow-typed runtimes**: stronger but require a typed execution
  substrate VCP does not assume.

## Open questions

- Granularity: field-level vs. record-level lineage — what is the right default?
- How to express residency/jurisdiction constraints in the label algebra?
