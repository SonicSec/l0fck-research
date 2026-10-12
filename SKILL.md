---
name: l0fck-research
description: "Use the l0fck RESEARCH/ reference docs as the source of truth before implementing changes to any l0fck exploit chain or adding a new custom chain. Activates on any task touching l0fck chains (bypass-full, sovereign, neo-chain, setup-assistant, glyphgrift, restore-chain), kernel R/W, kernel offsets, preboard/VUL 23, lockdown/mb2, or activation."
---

# l0fck Research Reference Skill

Before you **implement changes to an existing `l0fck` chain** or **add a new
custom chain**, you MUST read the relevant docs in
`l0fck/RESEARCH/`. They are the canonical, synthesized, log-verified
reference for every chain and cross-cutting finding. Do not start editing
`l0fck/src/**` until you have read at least the chain doc and the
`findings-zero-fake-success.md` policy.

## Target (all chains)

iPhone 16 Pro / Pro Max (`iPhone17,1`/`iPhone17,2`, **A18 Pro / T8140 /
CPID `0x8140`**), **iOS 26.6.1 (build 23G83)**, in Setup Assistant. Kernel
base slide hint `0xfffffff007004000`. A18 Pro has **zero MTE** (vs M5 MTE4) —
the strategic enabler for kernel-heap exploits.

## Step 1 — Pick the right docs

Open the index first: `l0fck/RESEARCH/README.md`. Then read **all** that
apply to your task:

| If you're touching… | Read (at minimum) |
|---|---|
| Any existing chain | that chain's `chain-*.md` + `findings-zero-fake-success.md` |
| A **new** custom chain | **all** `chain-*.md` (to avoid reusing a dead path) + `findings-zero-fake-success.md` + `findings-kernel-rw-hardening.md` + `findings-kernel-offsets.md` + `findings-activation-ambit.md` |
| Kernel R/W / Stage 6 | `chain-neo-chain.md`, `findings-kernel-rw-hardening.md`, `findings-kernel-offsets.md` |
| AppleAVE2 / WebCodecs / Wi-Fi | `chain-setup-assistant.md`, `findings-kernel-rw-hardening.md` |
| CoreGraphics PDF / OOB | `chain-glyphgrift.md` (note: userspace OOB, NOT kernel R/W yet) |
| Activation / SEP / Find My / mb2 / DFU | `findings-activation-ambit.md`, `chain-restore-chain.md` |
| Unauthenticated lockdown / AFC / SSL | `findings-lockdown-ssl-bypass.md` |
| Offsets (any) | `findings-kernel-offsets.md` |
| Preboard / VUL 23 | `findings-preboard-vul23.md` (the race is a **dead end** — D1) |

## Step 2 — Apply the governing rules (non-negotiable)

From `findings-zero-fake-success.md` and the `RESEARCH/README.md`
conventions:

1. **Zero-fake-success.** A stub returns honest `-1` /
   `STAGED / PENDING ON-DEVICE BRIDGE` — never `return 0`, never a
   hardcoded value presented as a live read. (The original
   `appleave2_exploit.c` kread returned `0xdeadbeef1337c0de`; that is the
   anti-pattern.)
2. **Crash ≠ RCE, crash ≠ write.** A crash, a "stage completed" log, or a
   parse-accept is **not** success. A success claim needs a **read-back**
   (canary over AFC, telemetry `krw_ready:true`, a result plist, or a
   2-anchor kbase cross-check).
3. **Offsets** — only non-zero values from
   `kernel-deltas/state/t8140/26.6.1-23G83.txt` are valid; Apple-Bug-Bounty
   (iOS 26.0.x) offsets are **deprecated**. A leaked kbase is trusted only
   with a **2-anchor cross-check**. Regenerate JS anchor tables from
   `OFFSETS/t8140-26.6.1-23G83/`.
4. **On-device execution is mandatory** for kernel exploits — the host
   cannot invoke IOKit/Mach VM over USB; A18 Pro is not checkm8-compatible.
   A new kernel chain needs a delivered on-device payload (AFC-staged `.app`).
5. **Do not reuse a dead path.** Known dead ends: the VUL 23 preboard race
   (invented RPC keys, D1), record-based activation (SEP local-nonce
   `Error -5`), GlyphGrift "kernel R/W" (it's userspace OOB), and DFU
   (no booted-OS services; cannot clear Find My). If your new chain's
   premise overlaps one of these, stop and reconcile first.

## Step 3 — Implement, then prove it

- Wire any new primitive to honest failure until an on-device payload /
  read-back exists.
- Add a **self-test** that exercises the **failure path** (not just the
  mocked success path) and keep `./l0fck --test` green (currently 22/22; a
  new chain gets the next number and a row in the README Self-Test table).
- Add the new chain to: `start.sh` (a menu entry + `print_doc_*` /
  `print_art_*` + confirmation screen, matching the existing 7-section
  grouped style), `src/main.c` (a `--<flag>` long-option + dispatch), and a
  new `l0fck/RESEARCH/chain-<name>.md`.
- Update `l0fck/README.md` (same format): the chain in the Bypass Chain
  Comparison table, the Interactive Menu Section-6 table, the stage flow,
  and a **Recent Changes** entry.
- Cite the **telemetry artifact** (JSON/plist/log path) for every success
  claim you record in the new RESEARCH doc.

## Provenance & supersession

Docs synthesize `PLAN/*.md`, `l0fck/src/**`, and live A18 Pro results
(2026-10-12 close-out in `RESEARCH/IMPORTANT TO CHECK/
research-2026-09-16-afc-26.6.1.md`, Session 4 addendum, and
`l0fck/tools/mb2_reachability_*.txt`). When a `PLAN/` "final report"
conflicts with a fix plan or logs, **the fix plan and logs win** — mark the
report **superseded** in the RESEARCH doc rather than deleting it.

## Quick map

```
l0fck/RESEARCH/
├── README.md                     # index + conventions
├── chain-bypass-full.md          # 5-stage TOCTOU (blocked: SEP Error -5)
├── chain-sovereign.md            # sovereign / sovereign-lite
├── chain-neo-chain.md            # kernel R/W framework + Stage 6
├── chain-setup-assistant.md      # Wi-Fi only, first real kbase
├── chain-glyphgrift.md           # CVE-2026-86950, userspace OOB
├── chain-restore-chain.md        # MB2 restore-staging + A18 close-out
├── findings-kernel-rw-hardening.md
├── findings-kernel-offsets.md
├── findings-preboard-vul23.md    # D1 dead end
├── findings-lockdown-ssl-bypass.md
├── findings-activation-ambit.md  # SEP wall / AFC / mb2 / DFU
└── findings-zero-fake-success.md # governing policy
```
