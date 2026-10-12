# l0fck-research — Kilo Skill

> ### ⚠️ Research only — do not use off-target
> **Everything in this repository is for defensive and academic research
> purposes only.** All associated work has been **lab-tested in a secure,
> isolated environment**. **Do not use this research to build custom attack
> chains.** **Do not run the associated tooling in an unsecured environment or
> against a device you do not own and are not authorized to test.**

This repository contains **only** the [`l0fck-research`](#skill) **Kilo
skill** — a structured reference-and-workflow that tells an AI coding agent to
consult the `l0fck` research docs before modifying or adding exploit chains.

The chain/finding documentation itself is **not** stored here. It lives with
the code in the [`l0fck`](https://github.com/MyHouseIsBig/l0fck) repository,
under [`l0fck/RESEARCH/`](https://github.com/MyHouseIsBig/l0fck/tree/main/RESEARCH)
(one doc per implemented chain + one per cross-cutting finding), and this
skill points the agent at those docs.

---

## Skill

**File:** [`SKILL.md`](./SKILL.md)

- **Name:** `l0fck-research`
- **Purpose:** before implementing changes to any `l0fck` exploit chain, or
  adding a new custom chain, the agent reads the relevant docs in
  `l0fck/RESEARCH/` and applies the project's governing rules (zero-fake-success,
  crash ≠ RCE, 2-anchor kbase, valid-26.6.1 offsets only, on-device-execution
  mandatory, do-not-reuse-dead-ends).
- **When it activates:** any task touching `l0fck` chains (bypass-full,
  sovereign, neo-chain, setup-assistant, glyphgrift, restore-chain), kernel R/W,
  kernel offsets, preboard/VUL 23, lockdown/mb2, or activation.

### How it's installed

The skill is loaded from the `l0fck` project workspace. The canonical file is
this repo's `SKILL.md`; the workspace copy at
`.agents/skills/l0fck-research/SKILL.md` is a **symlink** into this repo, so the
repo stays the single source of truth.

```
l0fck-research/
├── README.md    # this file
├── SKILL.md     # the l0fck-research Kilo skill (single source of truth)
└── .gitignore
```

### Where the docs live (not in this repo)

| Reference | Location |
|---|---|
| Chain + findings docs (canonical) | `l0fck/RESEARCH/` |
| Research target | iPhone 16 Pro / A18 Pro / iOS 26.6.1 (23G83), T8140 |
| Tool & self-test | `l0fck/` (`./l0fck --test`, 22/22) |

---

## Research-only notice

This repository is provided **exclusively for defensive and academic research**.

- All associated research was **lab-tested in a secure, isolated environment**.
- The material must **not** be used to create, refine, or deploy custom attack
  chains, or to target any system.
- Do **not** run the associated tooling in an unsecured environment.
- Do **not** use it against any device or person you do not own or have explicit,
  written authorization to test.
- You are responsible for complying with all applicable laws and your
  organizational/ethical obligations. Use of this material is at your own risk.
