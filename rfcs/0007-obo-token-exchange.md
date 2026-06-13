# RFC 0007: On-Behalf-Of Token Exchange Profile

- **RFC:** 0007
- **Title:** Per-Provider Credential Brokering via OAuth Token Exchange
- **Author:** hassard0
- **Status:** Draft
- **Created:** 2026-06-13
- **Discussion:** (open a Discussion thread and link it here)
- **Affected sections / schemas:** §7 (grants), §15 (HTTP/OAuth), §26 (multi-provider / OBO), `grant.schema.json`, `audit-event.schema.json`

## Summary

Define the concrete OAuth profile by which a VCP Gateway obtains a **per-Provider,
audience-bound, on-behalf-of** credential for each upstream API in a multi-provider
plan (§26), using OAuth 2.0 Token Exchange (RFC 8693) with resource indicators
(RFC 8707), and how the resulting delegation chain is recorded in grants and audit
events.

## Motivation

§26 establishes that a Gateway fans out to many Providers, never passes the user's
token through, and records an on-behalf-of delegation chain. This RFC pins the wire
mechanics so independent implementations interoperate: what is exchanged, what claims
the exchanged token carries, how it is bound to one Provider, and what is (and is not)
recorded.

## Detailed design

- **Exchange.** For each Provider step, the Gateway calls the authorization server's
  token endpoint with `grant_type=urn:ietf:params:oauth:grant-type:token-exchange`
  (RFC 8693), presenting the user's subject token and requesting a token with:
  - `resource` / `audience` = the Provider's resource indicator (RFC 8707), so the
    token is unusable elsewhere;
  - the **`act` (actor) claim** = the agent acting on behalf of the user, preserving
    the OBO relationship;
  - the **minimum scope** the plan step needs.
- **Binding.** The exchanged token is short-lived and held behind the Gateway's
  egress boundary (§7 secret broker). The grant records only its **audience** and a
  **key thumbprint** (`token_exchange.credential_jkt`) — never the token itself.
- **Delegation chain.** The grant and every audit event carry `delegation_chain`
  (authorizer → delegate → enforcer → executor → resource). Sub-delegation extends
  the chain and may only attenuate (§7).
- **Mix-up defense.** The Gateway validates the `iss` response parameter (RFC 9207,
  §15) and binds the credential to its issuing authorization server.

## Normative changes

- Add a §15/§26 subsection specifying the token-exchange request/response profile.
- Schema: `grant.token_exchange` and `grant.delegation_chain`,
  `audit-event.delegation_chain` / `credential_audience` (already added).

## Backward compatibility

Additive. Single-provider deployments that do not exchange tokens are unaffected;
`delegation_chain` degenerates to `[authorizer, delegate, enforcer, executor]`.

## Security considerations

Token exchange is the load-bearing control against confused-deputy and token
passthrough across Providers. Failure to bind `audience` correctly reintroduces those
risks; conformance tests 5 (token passthrough) and 3 (cross-server shadowing) cover
the multi-provider case. Audit events must never log the exchanged token, only its
audience and thumbprint.

## Alternatives considered

- **Per-Provider long-lived API keys**: simpler but reintroduces broad ambient
  authority and breaks per-call auditability.
- **Pass the user token through with narrowed scope**: still a passthrough; rejected.

## Open questions

- Should the Gateway cache exchanged tokens per (subject, provider, scope) for the
  plan's duration, and if so how does that interact with single-use grant semantics?
