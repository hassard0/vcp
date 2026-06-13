# RFC 0005: MCP Apps Interoperability

- **RFC:** 0005
- **Title:** Bridging MCP Apps (`ui://`) to VCP Interface Capabilities
- **Author:** hassard0
- **Status:** Draft
- **Created:** 2026-06-13
- **Discussion:** (open a Discussion thread and link it here)
- **Affected sections / schemas:** §16 (bridge), §22 (interface capabilities), `manifest.schema.json`

## Summary

Define how the `VCP-Bridge` profile (§16) maps an **MCP Apps** UI resource (the
`ui://` scheme, SEP-1865) into a VCP **interface capability** (§22), so that
server-rendered UIs from the MCP ecosystem can be consumed under VCP's stronger
trust guarantees.

## Motivation

MCP Apps (officially released 2026-01-26) lets MCP servers ship interactive HTML that
hosts render in sandboxed iframes, with tools declaring UI templates ahead of time.
It is a genuinely good idea and is already shipping in major hosts. VCP's §22
interface capability covers the same ground but makes signing and content-address
verification mandatory and routes UI-initiated actions through the grant pipeline.
A bridge lets the large body of existing MCP Apps work without weakening VCP.

## Detailed design

- **Resource mapping.** An MCP `ui://server/template` resource becomes a VCP
  `interface` capability whose `interface.surface` is `vcp:ui:<name>@sha256:<hash>`,
  with `content_hash` computed over the **observed** HTML bytes (the bridge pins what
  it saw, §4 rug-pull defense). `render` = `html-sandboxed`.
- **CSP / permissions.** The MCP App's declared CSP and permission metadata map onto
  `interface.csp` / `interface.permissions`. Absent CSP ⇒ deny-all default (§22).
- **Visibility.** MCP Apps' `visibility: ["model","app"]` maps onto `model_visible`
  and the `host_actions` allowlist: app-only tools become non-`model_visible`
  controls; model-visible tools must each be a separately verified VCP capability in
  `host_actions`.
- **Actions.** MCP Apps route View→host `tools/call` over `postMessage`. The bridge
  intercepts these and re-issues them as VCP plans, so each UI action gets policy,
  taint, and a single-use grant. A UI call to a capability outside `host_actions`
  is rejected (`INTERFACE_HASH_MISMATCH` / policy deny).
- **Trust marking.** Bridged interfaces carry `provenance: legacy_mcp` and are at
  most VCP-L0 unless the upstream server adopts native signing.

## Normative changes

- Add a §16 subsection specifying the `ui://` → `interface` mapping.
- No schema change beyond §22's `interface` block (already added).

## Backward compatibility

Additive; lives entirely in the bridge profile. Native VCP interfaces (§22) are
unaffected.

## Security considerations

The bridge cannot make an unsigned MCP App as trustworthy as a natively signed VCP
interface — it can only pin what it observed and route actions through the gateway.
That already neutralizes UI-swap and unauthorized-action-from-UI risks; it does not
prove provenance of the UI code (see RFC 0002).

## Alternatives considered

- **Reject MCP Apps entirely**: loses real ecosystem value.
- **Render MCP Apps unmediated**: reintroduces the trust assumptions VCP removes.

## Open questions

- How to surface to the user that a bridged UI is `legacy_mcp` (lower trust) vs. a
  natively signed interface?
