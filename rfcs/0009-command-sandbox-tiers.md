# RFC 0009: Command Sandbox Tiers

- **RFC:** 0009
- **Title:** Sandbox Tiers for Command/CLI Capabilities
- **Author:** hassard0
- **Status:** Draft
- **Created:** 2026-06-13
- **Discussion:** (open a Discussion thread and link it here)
- **Affected sections / schemas:** §14 (local sandbox), §28 (command capabilities), conformance L2–L4

## Summary

Specify the sandbox *tiers* a Gateway uses to execute `command` capabilities (§28),
from a lightweight in-kernel confinement suitable for L2 to a microVM/gVisor/WASM
boundary required at L4, and how a manifest names the tier it needs.

## Motivation

§28 mandates that commands run sandboxed with filesystem/network/env confinement, and
notes that shared-kernel isolation is not sufficient for execution that untrusted
input can influence. But it does not pin *how*. Implementations need a portable way to
say "this command needs at least a microVM" and Gateways need to know what each tier
guarantees. The 2026 practitioner consensus is explicit that Docker/runc shared-kernel
isolation is the wrong default for untrusted, model-influenced code execution.

## Detailed design

Three tiers, named in `sandbox.tier` (default `os`):

- **`os`** (L2 default) — in-OS confinement: Linux **Landlock + seccomp-bpf + user
  namespaces** (syscall filtering blocking `ptrace`, `mount`, `unshare`, `bpf`, …);
  macOS **Seatbelt** profile; Windows AppContainer/job objects. Filesystem and network
  allowlists per §28.2. Suitable when the command itself is trusted but its *inputs*
  are not.
- **`microvm`** (L3) — a hardware-virtualized microVM (Firecracker/Cloud Hypervisor)
  or **gVisor** user-space kernel. Required when the command or its dependencies are
  themselves untrusted (e.g. running model-generated code, installing arbitrary
  packages).
- **`wasm`** (L3/L4, where applicable) — a WASM sandbox for commands compiled to or
  expressible as WASM, giving capability-based, deny-by-default host access.

Rules:

- A manifest **MAY** declare a minimum tier; the Gateway **MUST** run at that tier or
  higher, or refuse (`SANDBOX_VIOLATION`).
- **L4** deployments **MUST NOT** execute a `command` whose inputs are influenced by
  `untrusted_*` data at tier `os`; they **MUST** use `microvm` or stronger.
- The sandbox profile id (and tier) is recorded in the audit event (§28.6) so the
  isolation actually used is auditable.

## Normative changes

- Add `sandbox.tier` to the manifest schema with the enum `{os, microvm, wasm}`.
- Add §28.2 detail mapping each tier to its guarantees and the L-level that requires it.
- Add an L4 conformance scenario: a command with untrusted-influenced input is refused
  at tier `os`.

## Backward compatibility

Additive. `sandbox.tier` defaults to `os`, matching current §28.2 behavior; existing
command capabilities are unaffected.

## Security considerations

Tiering makes the isolation guarantee explicit and auditable rather than implicit.
The residual risk is misdeclaration — a manifest claiming `os` for genuinely untrusted
code; policy SHOULD raise the required tier based on the data-flow labels reaching the
command, not trust the manifest alone.

## Alternatives considered

- **Single mandatory microVM for all commands**: too heavy for the common trusted-CLI
  case (`git status`), which `os` handles well.
- **No tiers, leave to implementations**: loses portability and auditability of the
  isolation guarantee.

## Open questions

- Should the required tier be *derived* from the data-flow labels (§12) automatically
  rather than declared?
- A portable profile format for `os`-tier (Landlock/seccomp/Seatbelt) so a manifest
  can ship one profile that maps across kernels?
