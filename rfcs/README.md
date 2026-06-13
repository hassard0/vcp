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

These four are the items deliberately deferred out of the v0.1 normative core (see
[`docs/design`](../docs/design)) and are open for discussion.
