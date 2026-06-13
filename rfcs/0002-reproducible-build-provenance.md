# RFC 0002: Reproducible-Build Provenance

- **RFC:** 0002
- **Title:** Reproducible-Build Provenance for Capability Code
- **Author:** hassard0
- **Status:** Draft
- **Created:** 2026-06-12
- **Discussion:** (open a Discussion thread and link it here)
- **Affected sections / schemas:** §5.2 (`provenance`), conformance L4

## Summary

Specify how the manifest `provenance` block (`source_repo`, `build_digest`,
`container_digest`) is produced and verified, so that a Gateway at L4 can confirm a
capability's running code corresponds to auditable source — applying SLSA-style
supply-chain assurance to agent capabilities.

## Motivation

A signed manifest proves *who* published a capability and pins *what* its contract
is. It does not, by itself, prove that the code actually executing matches reviewable
source. Without provenance, a provider can ship a benign-looking contract backed by
arbitrary server-side behavior. SLSA frames exactly this gap for software artifacts;
VCP should adopt the same levels for capability code.

## Detailed design

- **Provenance attestation.** Providers attach an in-toto / SLSA provenance
  statement binding `source_repo` + commit, the builder identity, and the resulting
  `build_digest` / `container_digest`.
- **Verification.** An L4 Gateway verifies (a) the provenance signature chains to a
  trusted builder, (b) the `container_digest` in the manifest matches the deployed
  image it is invoking (for VCP-Local) or is asserted by the provider's discovery
  metadata (for VCP-HTTP), and (c) the provenance references the same `source_repo`
  the manifest declares.
- **SLSA levels.** Map VCP provenance requirements to SLSA build levels; L4 requires
  at least SLSA Build L2 (hosted, signed provenance), with L3 (hardened, isolated
  builder) RECOMMENDED for `write-irreversible` capabilities.

## Normative changes

- Promote the `provenance` block from informational to verifiable at L4: define
  required fields and the attestation format.
- Add an L4 conformance test: a Gateway rejects a capability whose deployed image
  digest does not match its provenance.

## Backward compatibility

Additive and scoped to L4. The `provenance` block already exists in the manifest
schema as optional metadata; this RFC defines its verified semantics.

## Security considerations

Raises the bar from "trusted publisher" to "auditable code". Remaining gap: VCP-HTTP
providers self-host, so digest binding is an assertion unless paired with remote
attestation (TEE) — noted as future work. Does not weaken any existing control.

## Alternatives considered

- **Trust the publisher key only** (status quo): no link from contract to source.
- **Mandate TEEs / remote attestation now**: too heavy for v0.x adoption; revisit.

## Open questions

- Exact provenance format: in-toto attestation vs. SLSA provenance v1 predicate?
- How to bind digests for purely remote (VCP-HTTP) providers without a TEE?
