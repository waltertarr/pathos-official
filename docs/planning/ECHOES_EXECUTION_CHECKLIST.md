# Echoes of the Soul — Execution Checklist

A check-off tracker derived from the spec's roadmap. Work top to bottom. Each phase lists its
**goal**, **acceptance criteria** (the bar to call it done), and the **commit(s)** it maps to.
Do not skip the audit. Ship the MVP before the advanced region/faction work.

Legend: `[ ]` todo · `[~]` in progress · `[x]` done

---

## Pre-flight (environment) — see ECHOES_GITHUB_AND_SETUP.md

- [ ] Install Pathos desktop → `C:\Games\Pathos`
- [ ] `gh repo fork callanh/pathos-official --clone --remote`
- [ ] `git switch -c feature/echoes-of-the-soul` + push
- [ ] Build the **untouched** fork and launch it once (known-good baseline)

---

## Phase 0 — Repository Audit  ·  commit 2

**Goal:** understand the real codebase before changing anything.
**Deliverable:** `docs/ECHOES_REPO_AUDIT.md` answering:

- [ ] Where are modules defined?
- [ ] Where are dungeon generators defined?
- [ ] Where are classes/archetypes defined?
- [ ] Where are item/spell identification systems defined?
- [ ] Where are save/profile systems defined?
- [ ] Where is death/game-over handled?
- [ ] Where can an optional new mode be registered?
- [ ] What is safe to extend without breaking official modules?
- [ ] Confirmed: canonical build command, license file, seed-determinism of generation

> **Gate:** no broad implementation until this doc exists.

---

## Phase 1 — Mode Skeleton  ·  commits 1, 3

**Goal:** Echoes selectable but minimal.

- [ ] Copy spec → `docs/ECHOES_OF_THE_SOUL.md` (commit 1)
- [ ] Add mode identifier (`EchoesOfTheSoul`)
- [ ] Basic classless start if feasible
- [ ] Reuse existing dungeon generation
- [ ] Death branch that restarts from the **same** timeline seed
- [ ] Loop counter

**Acceptance:**
- [ ] Player can start Echoes mode
- [ ] Death restarts the same seed
- [ ] Loop count increments
- [ ] **Normal Pathos mode unaffected**

---

## Phase 2 — Soul Profile Persistence  ·  commits 4, 5

**Goal:** knowledge + fragments survive death. Adapt `SoulProfile` to the real save system.

- [ ] Add `SoulProfile` (or equivalent) on the existing persistence layer
- [ ] Persist identified items
- [ ] Persist known spells
- [ ] Persist loop count
- [ ] Persist memory fragments
- [ ] Death summary screen

**Acceptance:**
- [ ] Identified-item knowledge survives death
- [ ] Memory fragments awarded (formula in spec)
- [ ] Loop count persists
- [ ] Save/load works

---

## Phase 3 — Memory Tree MVP  ·  commits 6, 7

**Goal:** spend fragments on upgrades. Initial: Potion Memory, Scroll Memory, Monster Scholar, Cartography I.

**Acceptance:**
- [ ] Player can spend memory fragments
- [ ] Upgrade effects visible
- [ ] Upgrades persist across death

---

## Phase 4 — Discipline MVP  ·  commits 8, 9

**Goal:** classless start + learned disciplines (Warrior, Mage, Rogue, Cleric, Alchemist).

- [ ] Track discipline counters
- [ ] Unlock disciplines through play
- [ ] Discipline focus at loop start
- [ ] Basic benefits

**Acceptance:**
- [ ] Player starts classless
- [ ] At least one discipline unlockable
- [ ] Discipline persists after death
- [ ] Focus modifies future starts

---

## Phase 5 — Title MVP  ·  commit 10

**Goal:** titles + simple effects (The Looper, Dwarven Friend, Goblin Bane, Dragon Slayer, Archmage).

**Acceptance:**
- [ ] Titles earned by events/counters
- [ ] Titles persist
- [ ] Titles shown in UI / character sheet
- [ ] ≥1 title changes faction reaction or gameplay

---

## Phase 6 — Region Generator MVP  ·  commit 12

**Goal:** less generic dungeon. Initial regions: Dwarven Mines, Goblin Warrens, Elven Enclave, Medusa's Garden, Ancient Library.

> Largest upstream surgery — keep it after MVP-critical pieces.

- [ ] Region-selection layer above floor generation
- [ ] Floors assigned to region types
- [ ] Per-region monster/loot/theme tables
- [ ] ≥1 regional boss
- [ ] Stable region IDs for memory (`{Seed}:{Type}:{RegionIdx}:{FloorIdx}:{FeatureId}`)

**Acceptance:**
- [ ] Timeline includes named regions
- [ ] Distinct enemy/loot/layout patterns
- [ ] Regions discoverable + remembered

---

## Phase 7 — Faction MVP  ·  commit 11

**Goal:** titles/actions affect groups (Dwarves, Goblins, Elves, Mages).

- [ ] Reputation values (−100..100 scale)
- [ ] Title modifiers
- [ ] Simple faction reactions
- [ ] Merchant/guard behavior if feasible

**Acceptance:**
- [ ] Helping dwarves affects dwarf rep
- [ ] Killing goblins affects goblin rep
- [ ] Titles affect reaction
- [ ] Reactions persist as soul impressions

---

## Phase 8 — Dynamic Events MVP  ·  commit 13

**Goal:** region events. Initial: Goblin Siege of Dwarven Mine.

**Acceptance:**
- [ ] Event appears in some timelines
- [ ] Player can affect outcome
- [ ] Outcome grants title/reputation
- [ ] Replayable after reset; soul remembers discovery

---

## Phase 9 — Collapse Escape MVP  ·  commit 14

**Goal:** replace the boring ascent with a tense finale.

- [ ] Detect final-objective completion
- [ ] Trigger collapse state
- [ ] Shortened escape path
- [ ] Escalating hazards
- [ ] Faction/title-based shortcuts if feasible

**Acceptance:**
- [ ] Endgame escape shorter + more intense
- [ ] No walking up 30 unchanged floors
- [ ] Prior knowledge helps escape

---

## Phase 10 — Balance & Polish  ·  commit 15

- [ ] Tune fragment rewards
- [ ] Tune discipline unlock rates
- [ ] Tune region depth/difficulty
- [ ] Reduce overpowered stacking
- [ ] UI text
- [ ] Docs
- [ ] Debug commands if useful

---

## MVP definition (ship when these are true — do NOT wait for Phases 6–10)

- [ ] Echoes mode selectable
- [ ] Same dungeon seed after death
- [ ] Loop counter
- [ ] Memory fragments after death
- [ ] Item-identification persistence (at minimum)
- [ ] Classless start
- [ ] ≥1 learned discipline
- [ ] ≥1 title
- [ ] ≥1 region name/theme modification
- [ ] Normal mode unaffected

---

## Suggested commit sequence (from spec)

```
1  docs: add Echoes of the Soul design spec
2  docs: add repository audit for Echoes implementation hooks
3  feat(echoes): add optional mode skeleton and timeline seed
4  feat(echoes): add soul profile persistence and loop counter
5  feat(echoes): preserve basic knowledge across death
6  feat(echoes): add memory fragments and death summary
7  feat(echoes): add memory tree MVP
8  feat(echoes): add classless wanderer start
9  feat(echoes): add discipline unlock/progress MVP
10 feat(echoes): add title system MVP
11 feat(echoes): add faction reputation MVP
12 feat(echoes): add region generation layer MVP
13 feat(echoes): add Dwarven Mines/Goblin Warrens event MVP
14 feat(echoes): add collapse escape MVP
15 docs: add balance notes and playtest checklist
```

> Update `docs/ECHOES_PROGRESS.md` after each phase: completed work, files changed,
> current limitations, manual test steps. Run the matching test scenario from the spec's
> Testing Plan before checking a phase's acceptance boxes.
