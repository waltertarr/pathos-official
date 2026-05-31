# Echoes of the Soul — Master Plan

This is the entry point for turning [PATHOS_ECHOES_BRANCH_SPEC.md](PATHOS_ECHOES_BRANCH_SPEC.md)
into a **public GitHub branch of Pathos**. The spec is the *design*; these docs are the
*execution plan* — how to fork, set up, build, implement, and publish.

> The dungeon resets. The soul remembers.

---

## Decisions locked in

| Decision | Choice |
|---|---|
| Scope of first session | Planning documents only (this set) |
| Local build/test | Yes — Pathos desktop will be installed and used to build/playtest |
| GitHub structure | Fork `callanh/pathos-official` to `waltertarr`, work on branch `feature/echoes-of-the-soul` |
| Mode name (internal) | **Echoes of the Soul** |
| Licensing | Non-commercial freeware — preserve all upstream notices, never monetise |

---

## Verified environment snapshot (as of 2026-05-30)

| Check | Result |
|---|---|
| Upstream repo | `https://github.com/callanh/pathos-official` — **exists**, public, C#, ~178★ / 42 forks |
| Upstream license | Non-commercial freeware ("variants must not be commercialised in any way") |
| Build artifact | C# `Codex/` + `Modules/` compiled by **PathosMaker** into a `Pathos.Codex` binary loaded by the installed game |
| `gh` CLI | Authenticated as **waltertarr** (scopes: `repo`, `workflow`, `gist`, `read:org`) |
| .NET SDK | **9.0.306** installed |
| Git | 2.16 (works; old but fine) |
| **Pathos desktop** | **NOT installed** at `C:\Games\Pathos` — required before you can build/run a variant |

---

## The shape of the work

```
callanh/pathos-official  (upstream, public)
        │  fork
        ▼
waltertarr/pathos-official  (your public fork)
        │  branch
        ▼
feature/echoes-of-the-soul  (your public branch — this is "the branch")
        │  build via installed Pathos + PathosMaker
        ▼
playable Echoes variant
```

"A publicly available branch" = a branch on your **public fork**. Forks of a public repo
are public by default, so the branch is visible to anyone the moment you push it. No extra
publishing step is needed beyond pushing.

---

## Planning document set (this folder)

| Document | Purpose |
|---|---|
| **ECHOES_PLAN.md** (this file) | Index, decisions, next steps, open risks |
| [ECHOES_GITHUB_AND_SETUP.md](ECHOES_GITHUB_AND_SETUP.md) | Exact commands: fork, clone, branch, install Pathos, build, run, push, sync upstream, licensing |
| [ECHOES_EXECUTION_CHECKLIST.md](ECHOES_EXECUTION_CHECKLIST.md) | Phase 0→10 check-off tracker with acceptance criteria + commit mapping |
| [PATHOS_ECHOES_BRANCH_SPEC.md](PATHOS_ECHOES_BRANCH_SPEC.md) | The full design spec (source of truth for *what* to build) |

Documents that will live **inside the fork** once we start (per the spec):

| In-repo doc | Created during |
|---|---|
| `docs/ECHOES_OF_THE_SOUL.md` | Phase 1 (copy of the design spec) |
| `docs/ECHOES_REPO_AUDIT.md` | Phase 0 (where modules/gen/save/death hooks live) |
| `docs/ECHOES_PROGRESS.md` | Updated after every phase |

---

## Immediate next steps (the session after you approve this plan)

1. **Install Pathos desktop** to `C:\Games\Pathos` (see setup doc). This unblocks building.
2. **Fork + clone** with `gh repo fork callanh/pathos-official --clone`.
3. **Create the branch** `feature/echoes-of-the-soul`.
4. **Do a build** of the untouched fork to confirm the toolchain works *before* changing anything.
5. **Phase 0 — Repository Audit**: read the real `Codex/`, `Modules/`, `PathosOfficial.cs`,
   save/profile, identification, and death/game-over code; write `docs/ECHOES_REPO_AUDIT.md`.
6. Only then begin Phase 1 (mode skeleton).

> Rule from the spec: **do not start broad implementation before the Phase 0 audit.**

---

## Open risks / unknowns (resolve during Phase 0)

These are assumptions in the spec that the audit must confirm against the *actual* codebase.
The spec's C# models (`SoulProfile`, etc.) are **conceptual** — adapt to real architecture.

1. **Is there a "game mode" concept at all?** The spec assumes a new optional mode can be
   registered. If Pathos has no mode-selection layer, the skeleton needs a different hook.
2. **Save/profile architecture.** Where is persistent state stored, and what serializer?
   Soul persistence must reuse it, not invent a parallel system.
3. **Seeded generation.** Is dungeon generation deterministic from a seed today? Timeline
   reset depends on stable seeded regeneration.
4. **Death/game-over flow.** Single chokepoint to branch on, or scattered? Determines how
   "minimally invasive" the skeleton can be.
5. **Identification system granularity.** Per-item-type identity (potions/scrolls/wands) is
   the cheapest persistence win — confirm it's centralized.
6. **Region vs floor generation.** The biggest feature (region-based gen) may require the
   most upstream surgery. Keep it last; ship MVP without it.
7. **Build/run loop friction.** Confirm the edit→PathosMaker→run cycle time so playtest
   iterations are realistic.

---

## Guardrails (non-negotiable)

- **Classic mode untouched.** Echoes is strictly optional; normal Pathos must behave identically.
- **Non-commercial.** Never monetise; never remove or alter upstream license/attribution.
- **Minimal, reversible, namespaced.** New code labeled `Echoes`/`EchoesOfTheSoul`; comment
  every hook into official systems.
- **Small commits.** Follow the 15-commit plan in the spec; update `ECHOES_PROGRESS.md` per phase.
- **Audit before build.** No broad changes before `docs/ECHOES_REPO_AUDIT.md` exists.
