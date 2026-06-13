# Examples

The canonical worked example — *"Look at Alex's email and schedule the demo for next
week"* — is documented end to end in
[`SPECIFICATION.md` §16](../SPECIFICATION.md#16-mcp-bridge-profile-vcp-bridge),
showing how read-only calls run unattended, how a write goes through plan/apply with
a user-visible dry-run diff, and how a prompt injection hidden in an email is
contained because authority never flows from tainted data.

**Runnable** versions of this and other examples — driven by the reference SDKs and
gateways — live in the implementation repository:
[`vcp-servers/examples`](https://github.com/hassard0/vcp-servers).

This directory holds protocol-level message traces (request/response envelopes) that
are language-agnostic and validate against the schemas in [`../schemas`](../schemas).
