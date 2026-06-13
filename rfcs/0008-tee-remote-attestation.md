# RFC 0008: TEE Remote Attestation Tier

- **RFC:** 0008
- **Title:** Hardware Remote-Attestation (`tee`) Tier for Environment Attestation
- **Author:** hassard0
- **Status:** Draft
- **Created:** 2026-06-13
- **Discussion:** (open a Discussion thread and link it here)
- **Affected sections / schemas:** §27 (environment attestation), conformance L4, `environment-attestation.schema.json`

## Summary

Specify the `tee` tier of environment attestation (§27.3): how a Gateway, Provider,
or Agent presents **hardware remote-attestation Evidence** following the RATS
architecture (RFC 9334), and how a Gateway (the Verifier) appraises it to an
Attestation Result before minting grants. This is the L4 strengthening of the
default `statement` tier.

## Motivation

The `statement` tier (§27.3) proves key continuity and a *claimed* build digest with
nothing but the actor's signing key. It does not prove the code is *actually* running
unmodified in a genuine environment — a compromised host can sign an honest-looking
statement. For high-assurance (L4) deployments — regulated data, confidential
computing — the relying party needs hardware-rooted evidence. RFC 9334 (RATS) is the
standard architecture for exactly this.

## Detailed design

- **RATS roles.** Attester = the actor (gateway/provider/agent); Verifier = the
  Gateway; Relying Party = policy. The Attester produces **Evidence** (e.g. an
  Intel TDX / AMD SEV-SNP quote, an AWS Nitro attestation document, or a TPM quote);
  the Verifier appraises it against reference values and an endorsement, yielding an
  **Attestation Result**.
- **Freshness.** The Gateway's `nonce` (§27.4) is included in the quoted measurement,
  binding the Evidence to this challenge.
- **Reference values.** The expected measurements/`build_digest` come from the
  manifest provenance (RFC 0002) and/or the transparency registry (RFC 0001).
- **Carriage.** `environment-attestation` with `tier: "tee"` carries a `measurement`
  and an `evidence_ref`; the full Evidence is appraised out-of-band (it can be large),
  and only the Attestation Result is cached and referenced per call (§27.2).
- **Verifier delegation.** A Gateway MAY delegate appraisal to a dedicated Verifier
  service (e.g. a confidential-computing attestation service) and treat its signed
  Attestation Result as authoritative.

## Normative changes

- Add §27.3/§27.4 detail for `tier: tee`: Evidence formats accepted, nonce binding,
  reference-value sourcing, and Attestation Result caching.
- Add an L4 conformance test: a forged/replayed TEE quote is rejected
  (`ATTESTATION_INVALID`).

## Backward compatibility

Additive and L4-scoped. The default `statement` tier and all of L0–L3 are unaffected;
environment attestation remains off unless required.

## Security considerations

TEE attestation raises assurance to hardware-rooted but adds a Verifier trust anchor
and an availability dependency; the result is cached per boot epoch to bound cost.
Evidence and measurements are not secrets but MUST be bound to the challenge nonce to
prevent replay. A compromised Verifier service is a single point of failure — its
Attestation Results SHOULD themselves be signed and auditable.

## Alternatives considered

- **Statement tier only** (status quo): adequate for most; insufficient for L4.
- **Mandate one TEE vendor**: fragments deployment; VCP stays evidence-format-neutral.

## Open questions

- Should VCP define a normative Attestation Result format, or defer to existing ones
  (e.g. EAT / CWT)?
- How do `boot_epoch` caching windows interact with TEE measurement freshness
  requirements?
