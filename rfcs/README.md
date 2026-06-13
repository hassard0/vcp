# VCP RFCs

Substantive changes to VCP — anything that alters a normative MUST/SHOULD
requirement, changes a schema, or adds a new protocol surface — go through the RFC
process. Small fixes (typos, clarifications that don't change meaning) can be plain
PRs or issues.

## Lifecycle

```
Draft → Discussion → Last Call → Accepted / Rejected → Final
```

- **Draft** — opened as a PR adding `rfcs/NNNN-title.md` from the template, plus a
  Discussion thread.
- **Discussion** — open feedback in the linked Discussion thread.
- **Last Call** — maintainers signal intent to accept/reject; a final comment window.
- **Accepted** — merged; the change is scheduled into a dated protocol revision.
- **Rejected** — kept in the tree for the historical record, marked Rejected.
- **Final** — the change has shipped in a revision of `SPECIFICATION.md`.

See [GOVERNANCE.md](../GOVERNANCE.md) for how decisions are made.

## Numbering

RFCs are numbered sequentially with a zero-padded 4-digit prefix
(`0001`, `0002`, …). `0000` is the template.

## Index

| RFC | Title | Status | Touches |
|---|---|---|---|
| [0000](./0000-template.md) | Template | — | — |
| [0001](./0001-transparency-registry.md) | Transparency Registry | Draft | §4, §5.2, L4 |
| [0002](./0002-reproducible-build-provenance.md) | Reproducible-Build Provenance | Draft | §5.2, L4 |
| [0003](./0003-formal-policy-verification.md) | Formal Policy Verification | Draft | §6, L4 |
| [0004](./0004-dlp-data-flow-proofs.md) | DLP / Data-Flow Proofs | Draft | §12, §15, L4 |
| [0005](./0005-mcp-apps-interop.md) | MCP Apps Interoperability (`ui://` bridge) | Draft | §16, §22 |
| [0006](./0006-async-task-resumability.md) | Async Task Resumability & Replay | Draft | §10, §21 |
| [0007](./0007-obo-token-exchange.md) | On-Behalf-Of Token Exchange Profile | Draft | §7, §15, §26 |
| [0008](./0008-tee-remote-attestation.md) | TEE Remote-Attestation Tier | Draft | §27, L4 |
| [0009](./0009-command-sandbox-tiers.md) | Command Sandbox Tiers | Draft | §14, §28, L2–L4 |

RFCs 0001–0004 are items deliberately deferred out of the v0.1 normative core (see
[`docs/design`](../docs/design)). RFCs 0005–0009 detail the wire mechanics behind the
`2026-06-13` additions (interface capabilities, async tasks, multi-provider
on-behalf-of delegation, the hardware tier of environment attestation, and command
sandbox tiers). All are open for discussion.
