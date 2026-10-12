# l0fck-research

**Published research reference for [`l0fck`](https://github.com/MyHouseIsBig/l0fck) — the iPhone 16 Pro / A18 Pro / iOS 26.6.1 security research suite.**

This repository holds the **chain-by-chain and finding-by-finding reference**
for every exploit chain implemented in `l0fck` and the cross-cutting findings
that bound them. It is the source an implementer reads **before** changing an
existing chain or adding a new custom one.

> **Relationship to `l0fck/`:** the canonical docs live inside the project at
> `l0fck/RESEARCH/` (kept in sync with the code). This repository is the
> standalone, browsable mirror of that reference plus the surrounding
> research context. When the two disagree, the in-repo `l0fck/RESEARCH/` is
> authoritative.

---

## Research target

| Parameter | Value |
|---|---|
| SoC | Apple A18 Pro (`T8140`) |
| Chip ID (CPID) | `0x8140` |
| Target models | `iPhone17,1` (iPhone 16 Pro), `iPhone17,2` (iPhone 16 Pro Max) |
| OS | iOS 26.6.1 |
| Build | `23G83` |
| Kernel offsets | `kernel-deltas/state/t8140/26.6.1-23G83.txt` (only non-zero values valid) |
| Kernel base slide hint | `0xfffffff007004000` |

**Strategic enabler:** A18 Pro has **zero MTE** (vs M5's full MTE4), so
kernel-heap exploits are unmitigated. A18 Pro is **post-checkm8** (no
pwned-bootrom), so on-device payloads must be delivered by other means (AFC
staging / Wi-Fi).

---

## Document index

### Chains (one per implemented chain)

| Doc | Chain | CLI | Transport | Kernel R/W | Status |
|---|---|---|---|---|---|
| `chain-bypass-full.md` | Bypass-Full (5-stage TOCTOU) | `--bypass-full` / `--bypass-watch` | USB | No | Blocked on SEP nonce crypto (`Error -5`) |
| `chain-sovereign.md` | Sovereign-Chain / Sovereign-Lite | `--sovereign` / `--sovereign-lite` | USB | No | No race stage; injection + notification + respring |
| `chain-neo-chain.md` | Neo-Chain (kernel R/W framework) | `--neo-chain` (+ Stage 6) | USB | Yes (AppleAVE2 primary) | Stages 0–6 implemented; on-device primitive mandatory |
| `chain-setup-assistant.md` | SetupAssistant-Chain (Wi-Fi only) | `--setup-assistant-chain` | **Wi-Fi (no USB)** | Yes (AppleAVE2 over WebSheet) | First genuine kbase leak (`0xfffffff0d2000000`) |
| `chain-glyphgrift.md` | GlyphGrift-Chain (CVE-2026-86950) | `--glyphgrift-chain` | **Wi-Fi (no USB)** | Userspace OOB (not kernel R/W yet) | Trigger fires; corruption not yet observed |
| `chain-restore-chain.md` | Restore-Chain (MB2 restore-staging) | `--restore-chain` / `--restore-probe` | USB (lockdown / CoreDevice RSD) | No | A13-class only on this host; A18 Pro close-out (`ServiceProhibited`) |

### Findings (cross-cutting)

| Doc | Topic |
|---|---|
| `findings-kernel-rw-hardening.md` | Which primitive won (AppleAVE2 CVE-2026-84607) + Stage-6 fixes (KASLR slide, PAC strip, page-skip, cave discovery, allproc validation) + PMAP lab results |
| `findings-kernel-offsets.md` | 26.6.1-23G83 T8140 offset source of truth, recovered 2026-09-02 values, blocked list, 2-anchor cross-check, promotion rules |
| `findings-preboard-vul23.md` | Preboard PoC/triage + the D1 finding that the "USB-first 60%" race keys are invented (dead end) |
| `findings-lockdown-ssl-bypass.md` | Unauthenticated lockdownd SSL bypass & host-trust path traversal (Vuln #03) |
| `findings-activation-ambit.md` | SEP local-nonce `Error -5` wall, AFC Media write limits, mb2 `InvalidService`/`ServiceProhibited` gates, A18 Pro close-out, why DFU is not a substitute |
| `findings-zero-fake-success.md` | The governing project policy: no fake success, crash ≠ RCE, read-back proof; refuted "final reports" |

The full working set (with inline `PLAN/*.md` provenance) is mirrored in
[`l0fck/RESEARCH/`](https://github.com/MyHouseIsBig/l0fck/tree/main/RESEARCH).

---

## Conventions (read before implementing)

1. **Zero-fake-success** — every stage returns honest failure
   (`-1` / `STAGED / PENDING ON-DEVICE BRIDGE`) when its primitive is a stub.
   No `return 0`, no hardcoded value presented as a live read.
2. **Crash ≠ RCE, crash ≠ write** — a crash, a "stage completed" log, or a
   parse-accept is **not** success. A success claim needs a read-back
   (AFC canary, telemetry `krw_ready:true`, a result plist, or a 2-anchor
   kbase cross-check).
3. **Offsets** — only non-zero values from
   `kernel-deltas/state/t8140/26.6.1-23G83.txt` are valid; Apple-Bug-Bounty
   (iOS 26.0.x) offsets are deprecated. A leaked kbase is trusted only with a
   **2-anchor cross-check**.
4. **On-device execution is mandatory** for kernel exploits — the host cannot
   invoke IOKit/Mach VM across USB.
5. **Do not reuse a dead path** — VUL 23 preboard race (invented RPC keys),
   record-based activation (SEP nonce), GlyphGrift "kernel R/W" (it's
   userspace OOB), and DFU (no booted-OS services; cannot clear Find My).

---

## Provenance & supersession

Docs synthesize the `l0fck` `PLAN/` history, `l0fck/src/**`, and live A18 Pro
results (2026-10-12 close-out in `RESEARCH/IMPORTANT TO CHECK/
research-2026-09-16-afc-26.6.1.md`, Session 4 addendum, and
`l0fck/tools/mb2_reachability_*.txt`). Where a `PLAN/` "final report" was
refuted by device logs, the corrective plan wins and the refuted claim is
marked **superseded** — never silently deleted.

## Related repositories

- [`l0fck`](https://github.com/MyHouseIsBig/l0fck) — the tool itself (C,
  libimobiledevice/libusb/OpenSSL/curl; `./l0fck --test` self-test suite).
- `kernel-deltas` — T8140 offset deltas & state files.
- `RESEARCH/` (in `l0fck` workspace) — raw CVE writeups, offsets, logs.

---

*Security research for defensive and academic use. No guarantees of any kind.*
