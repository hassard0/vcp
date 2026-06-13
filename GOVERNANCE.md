# VCP Governance

This document describes how the **Verifiable Capability Protocol (VCP)** project is
governed: its goal, its roles, how decisions are made, and how the specification
evolves through the RFC process.

This is a process document. The RFC 2119 keywords (MUST, SHOULD, MAY) are reserved
for the normative protocol text in [`SPECIFICATION.md`](./SPECIFICATION.md); this
document uses ordinary prose.

## Project goal

VCP exists to provide an **open, vendor-neutral, security-first capability protocol**
for AI agents — a stricter, zero-trust alternative to designs where the model is
trusted with authority. The specification is the source of truth; reference
implementations live in a separate repository and pin the spec by hash.

The project is governed in the open. Design discussion happens in GitHub Discussions,
defects and ambiguities are filed as Issues, and any change to normative requirements
or schemas goes through the RFC process described below.

## Roles

- **Maintainers** — hold commit and merge rights on this repository, steward the RFC
  process, cut dated protocol revisions, and have final say on what is accepted. The
  current maintainer is [@hassard0](https://github.com/hassard0). Maintainers act by
  lazy consensus among themselves; disagreement is resolved by discussion, and a
  maintainer may ask for more time before a change merges.
- **Contributors** — anyone who opens an issue, comments on a Discussion, submits a
  pull request, contributes conformance vectors, or authors an RFC. No formal status
  is required to contribute.
- **RFC authors** — contributors who shepherd a specific proposed change through the
  RFC lifecycle. An RFC author is responsible for keeping their RFC and its Discussion
  thread current, incorporating feedback, and driving it toward Last Call.

## Decision-making

Decisions are scaled to their blast radius:

- **Lazy consensus for small changes.** Editorial fixes, clarifications that do not
  change meaning, typos, broken links, examples, non-normative prose, and additions to
  tooling can be merged by a maintainer once they have been open long enough for
  objection (typically a few days) and no maintainer objects. Silence is assent.
- **RFC process for normative changes.** Anything that adds, removes, or changes a
  normative **MUST**/**MUST NOT**/**SHOULD**/**SHOULD NOT**/**REQUIRED** requirement,
  changes an actor's responsibilities, alters the wire shape of an envelope, or
  changes a schema in [`schemas/`](./schemas/) **requires an accepted RFC**. If a pull
  request would change normative behavior without an RFC, a maintainer will ask for
  one first.

When in doubt about which path a change needs, open a Discussion and ask.

## RFC process

RFCs are the unit of deliberate, normative change. They live in
[`rfcs/`](./rfcs/), are written from [`rfcs/0000-template.md`](./rfcs/0000-template.md),
and are numbered with a zero-padded, sequential 4-digit number (`0001`, `0002`, …).
The number is assigned when the RFC's pull request is opened; if two PRs collide on a
number, the later one renames.

### Lifecycle

```
Draft  →  Discussion  →  Last Call  →  Accepted / Rejected  →  Final
```

- **Draft** — the RFC exists as a PR adding `rfcs/NNNN-title.md`. It need not be
  complete, but it should fill in the template's Summary, Motivation, and a sketch of
  the Detailed design.
- **Discussion** — the proposal is debated in its linked GitHub Discussion thread and
  on the PR. The author revises in response to feedback. This is where most of the
  work happens.
- **Last Call** — a maintainer judges the proposal mature, freezes substantive
  changes, and announces a final review window (typically 7–14 days) for any
  last objections.
- **Accepted / Rejected** — at the end of Last Call, maintainers decide by lazy
  consensus. An **Accepted** RFC merges with `Status: Accepted` and its normative
  changes are scheduled into a protocol revision. A **Rejected** RFC merges (or is
  closed) with `Status: Rejected` and a recorded rationale, so the decision is
  discoverable later.
- **Final** — once an Accepted RFC's normative changes have shipped in a dated
  protocol revision and the relevant schemas are updated, the RFC is marked `Final`.

### How to propose an RFC

1. Open a GitHub **Discussion** describing the problem (use the RFC discussion form).
2. Copy [`rfcs/0000-template.md`](./rfcs/0000-template.md) to
   `rfcs/NNNN-short-title.md`, fill in the header (number, title, author,
   `Status: Draft`, created date, discussion link, affected sections/schemas) and the
   body sections.
3. Open a **pull request** adding that file and linking the Discussion thread.
4. Iterate through the lifecycle above.

See [`rfcs/README.md`](./rfcs/README.md) for the RFC-focused walkthrough and the
current index.

## Versioning policy

- The protocol is identified by a **dated revision** (`YYYY-MM-DD`), not a semantic
  version. Revisions are recorded in [`CHANGELOG.md`](./CHANGELOG.md).
- While the protocol's status is **Draft**, dated revisions **may change
  incompatibly**. Backward-incompatible changes are allowed until the protocol is
  declared **Stable**; after that, incompatible changes require a new RFC and a
  clearly announced migration path.
- **Schemas are versioned with the prose.** A change to a schema in
  [`schemas/`](./schemas/) ships in the same dated revision as the prose change that
  motivates it, and the two must stay in sync. The schemas are normative.

## Becoming a maintainer

Maintainership is earned through sustained, high-quality contribution: shepherding
RFCs, reviewing pull requests, contributing conformance vectors, and demonstrating
good judgment about the project's security-first goals. An existing maintainer may
nominate a contributor; the nomination is accepted by lazy consensus of the current
maintainers. Maintainers who become inactive may move to emeritus status by their own
request or by consensus of the active maintainers.

## Code of conduct

All participation is governed by the project
[Code of Conduct](./CODE_OF_CONDUCT.md). Maintainers are responsible for its
enforcement.
