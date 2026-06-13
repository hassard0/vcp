# RFC 0001: Transparency Registry

- **RFC:** 0001
- **Title:** Transparency Registry for Manifests, Provenance, and Revocations
- **Author:** hassard0
- **Status:** Draft
- **Created:** 2026-06-12
- **Discussion:** (open a Discussion thread and link it here)
- **Affected sections / schemas:** §4, §5.2, §16, conformance L4

## Summary

Define an append-only, publicly auditable **Transparency Registry** that records
signed capability manifests, their provenance, and revocations — so that a Gateway
can verify a capability was published openly (not served selectively to one victim)
and has not been revoked. This is the load-bearing requirement of conformance L4.

## Motivation

Content-addressed identity (§4) already turns a rug pull into a new, unapproved
capability. But two gaps remain:

1. **Selective serving.** A malicious provider can serve a benign manifest to
   reviewers and a different (still validly signed) manifest to a target. Both are
   signed; identity alone does not reveal that two versions exist.
2. **Revocation.** When a key is compromised or a capability is found unsafe, there
   is no standard, tamper-evident way for a Gateway to learn it.

A transparency log — as proven by Certificate Transparency and Sigstore/Rekor —
makes every published manifest publicly discoverable and every revocation
verifiable.

## Detailed design

- **Log entries.** Each entry is `{ manifest_hash, capability_id, issuer,
  published_at, kind: "publish" | "revoke", provenance_ref }`, appended to a
  Merkle-tree-backed log.
- **Inclusion proof.** A provider returns, alongside a manifest, a signed inclusion
  proof (Merkle audit path + signed tree head). A Gateway at L4 **MUST** verify the
  inclusion proof before admitting the capability.
- **Revocation.** A `revoke` entry references a previously published
  `capability_id`. A Gateway **MUST** treat a capability with a logged revocation as
  inadmissible. Revocations are monotonic — a capability, once revoked, stays revoked
  under that id (a fixed version may be republished under its new contract hash).
- **Gateway consultation.** L4 Gateways consult the log named in the provider
  discovery document (`transparency_log`, see `discovery.schema.json`). The log's
  signed tree head is itself verifiable and gossipable to detect a split view.
- **Offline operation.** Gateways **SHOULD** support a cached, periodically refreshed
  signed tree head so that short network partitions do not fail-open. Stale-cache
  policy (max age before fail-closed) is operator-configurable.

## Normative changes

- Add to §4/§5.2: at L4, manifest admission **MUST** include transparency inclusion
  verification.
- Add a `transparency` block to the discovery schema (already stubbed as
  `transparency_log`) specifying log endpoint, public key, and proof format.
- Add an L4 conformance test: *split-view detection* — a Gateway presented with two
  validly signed manifests for one logical capability detects the inconsistency.

## Backward compatibility

Additive. L0–L3 implementations are unaffected; the requirement is scoped to L4.
A dated-revision bump is needed when the discovery schema gains the `transparency`
block.

## Security considerations

Introduces a new trust anchor (the log operator) and a new availability dependency.
Mitigations: signed tree heads + gossip make a misbehaving log detectable; cached
tree heads bound the availability impact. The log itself stores only hashes and
public metadata — never secrets or argument data.

## Alternatives considered

- **No registry, identity only** (status quo): cannot detect selective serving or
  standardize revocation.
- **Per-provider revocation lists** (CRL-style): not tamper-evident; a provider can
  hide its own revocations.

## Open questions

- Single canonical log vs. federation of logs with cross-gossip?
- Should inclusion proofs be required at L3 for write-irreversible effects, not just L4?
