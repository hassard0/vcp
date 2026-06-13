# RFC 0003: Formal Policy Verification

- **RFC:** 0003
- **Title:** Formal Verification of Policy Properties
- **Author:** hassard0
- **Status:** Draft
- **Created:** 2026-06-12
- **Discussion:** (open a Discussion thread and link it here)
- **Affected sections / schemas:** §6 (policy decision interface), conformance L4

## Summary

Define how an operator can **formally verify** global properties of their policy set
over the §6 decision interface — for example, "no decision can ever allow
confidential data to reach an external sink" — rather than relying on testing alone.

## Motivation

The §6 interface standardizes the *shape* of a policy decision but not its
*guarantees*. As policy sets grow, operators need assurance that no combination of
inputs yields an unsafe `allow`. Authorization languages such as Cedar are designed
to be analyzable; OPA has companion tooling; a local PDP such as `cani` can expose
its rules for analysis. VCP should specify the property language and the obligations
on a policy engine that claims verifiability, without mandating a specific engine.

## Detailed design

- **Property language.** A small, declarative language over the policy-request fields
  (`subject`, `capability`, `effect`, `data_flows[].classification`, `risk`) and the
  decision (`allow`/`deny`/`challenge`). Properties are universally quantified
  invariants, e.g. `forall r: r.data_flows has {classification: "confidential",
  to: external} ⇒ decision != allow`.
- **Engine obligation.** An engine claiming L4 verifiability **MUST** be able to
  either prove a stated property holds for all inputs or produce a counterexample
  decision request that violates it.
- **Artifacts.** The proof or counterexample is emitted as a signed artifact and
  **MAY** be logged to the transparency registry (RFC 0001) for audit.

## Normative changes

- Add §6.x defining the property language grammar and the verifiability obligation.
- Add an L4 conformance test: given a deliberately flawed policy set and a stated
  property, the engine returns a valid counterexample.

## Backward compatibility

Additive, L4-scoped. Engines that do not claim verifiability are unaffected; they
simply do not advertise the L4 verifiability capability.

## Security considerations

Formal verification raises assurance but is only as good as the property set: an
operator who never states "confidential ↛ external" gains nothing. Guidance will
ship a baseline property library (the §12 taint invariants expressed as properties).

## Alternatives considered

- **Testing only** (status quo): cannot cover the input space for high-assurance
  deployments.
- **Mandate Cedar/OPA**: VCP stays engine-neutral; mandating one engine fragments
  adoption.

## Open questions

- Should the property language be a profile of an existing language (e.g. Cedar's
  analysis fragment) rather than new?
- How to verify properties that depend on external state (data classification) that
  is not known statically?
