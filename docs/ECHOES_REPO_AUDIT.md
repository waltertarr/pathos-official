# Echoes of the Soul — Repository Audit (Phase 0)

Audit of `callanh/pathos-official` to determine where Echoes hooks in, and — more
importantly — **what is and isn't possible** given this repo's boundaries. Read this before
writing any implementation code.

Date: 2026-05-30 (engine inspection resolved 2026-05-31) · Branch: `feature/echoes-of-the-soul`

---

## ⚠️ RESOLVED VERDICT (2026-05-31): the time-loop conceit is NOT buildable in this fork

The engine DLLs are now installed and have been decompiled and inspected (ILSpy). The
make-or-break question from section 5 is answered, and the answer is the **worst case**:

- **`Module` has one hook only:** `public abstract void Execute(Generator)`, invoked once when
  a new adventure is generated. There is **no** death, run-end, victory, turn, or save callback.
- **The engine does not expose its internals to the content assembly.**
  `PathosEngine` declares `InternalsVisibleTo` for `Pathos`, `PathosGame`, `PathosMaker`, etc.
  — **but not `PathosOfficial`.** Modules can call only the engine's *public* API.
- **The seed is engine-controlled.** `Adventure.Seed` has a public getter but an `internal`
  setter (`SetSeed`), so a module can read the seed but cannot pin/reset it. (Players can type a
  seed at new-game time, but that's a manual UI action, not module logic.)
- **There is no module-writable store that survives death.** `World` is a per-adventure content
  container; `Conduct` is a per-run scorecard with `internal` setters; there is no bones/legacy
  system; the only cross-adventure persistent object is `Profile`, which is engine-owned
  *settings* (volume, zoom, custom heroes), not a soul/meta-progression store.

**Therefore:** "die → same seed regenerates → soul knowledge / memory fragments / disciplines /
titles persist and reload into the next run" **cannot be implemented by forking this content
repo.** It would require *engine* changes — a Module lifecycle with death/start hooks, a
persistent per-soul meta-store, and module-controllable seed pinning — and the engine is
closed-source (we have only the compiled DLLs). Only upstream (`callanh`) can add those hooks.

**What remains fully buildable here (Tier A):** a new selectable `EchoesModule` — a themed,
region-structured campaign (distinct Sites with their own monster/loot/atmosphere tables,
regional bosses, settlements, dialogue, a classless-ish starting Hero, faction-flavoured
content). That is a real, shareable Pathos variant; it just is not the soul/time-loop game.
Players who want a "replay the same layout" feel can opt into a **seeded run** manually.

The rest of this document is the original pre-inspection audit, retained for context. See
section 5 and the Status box for how this verdict maps onto them.

---

## TL;DR — the one thing that changes the plan

**This repository is content + module-generation only. The game engine is closed and
precompiled.** The repo's `PathosOfficial.csproj` references `PathosEngine.dll`,
`PathosLibrary.dll`, and `Inv.Library.dll` as binaries from the installed game at
`C:\Games\Pathos`. The build output is a `Pathos.Codex` data file the installed engine loads.

Consequence for the spec:

| Spec pillar | Where it lives | Feasible in this repo? |
|---|---|---|
| Optional new **mode** | `Module` subclass + `AddModule(...)` | **Yes — natural fit** |
| **Region**-based generation | `World.AddSite` / `AddMap` / `AddLevel` in `Module.Execute` | **Yes — this is literally how the base game is built** |
| Themed monster/loot/boss tables | `Generator` + `Codex` content | **Yes** |
| Classless-ish start | new `Class` + `Hero` content | **Mostly** |
| Titles / factions (as content + flavour) | new `Codex` content, dialogue, `Standing`-like | **Partly** |
| **Soul persistence across death** (the core conceit) | **engine save/profile system** | **Unknown — needs DLL inspection; likely not moddable here** |
| Death-flow branch, custom death summary, meta-progression UI | **engine** | **Unknown — likely not moddable here** |
| Same-seed timeline reset on death | **engine campaign/seed control** | **Unknown — likely not moddable here** |
| Memory-fragment meta-currency persisted between runs | **engine save** | **Unknown — likely not moddable here** |

**Bottom line:** the *world-building half* of Echoes (a themed, region-structured optional
campaign with bosses, factions-as-flavour, and a classless start) is fully achievable here and
would already be a genuinely shareable variant. The *time-loop persistence half* — the thing
that makes it "Echoes" — depends on engine APIs not present in this repo. We cannot confirm it
until the Pathos desktop DLLs are installed and their public surface inspected. **That
inspection is the remaining Phase 0 task and the single biggest risk to the spec as written.**

---

## 1. Repository architecture

```
PathosOfficial.sln / .csproj      net10.0-windows class library -> PathosOfficial.dll
PathosOfficial.cs                 OfficialCampaign : Campaign   (top-level entry)
PathosResources.cs
Codex/   (60 files)               content definitions (items, classes, spells, monsters, ...)
Modules/ (Nethack, Opus, SPD,     world-generation algorithms = the playable "modes"
          Dhak, Sandbox)
Atlases/ Albums/ Assets/          tilesets, sounds, translations, music
Resources/ Reports/ ChangeLogs/
license.txt                       Creative Commons Attribution-NonCommercial 4.0
```

### Entry point — `PathosOfficial.cs`

```csharp
public sealed class OfficialCampaign : Campaign
{
  this.Codex = new Codex(new Manifest("Pathos"));   // builds all content (MASTER_CODEX)
  SetManifest(Codex.Manifest);
  AddModule(new NethackModule(Codex));              // <-- modes are registered here
  AddModule(new OpusModule(Codex));
  AddModule(new SPDModule(Codex));
  AddModule(new DhakModule(Codex));
  AddModule(new SandboxModule(Codex));
}
```

`Campaign`, `Codex`, `Manifest`, `Module`, `Generator` all come from the engine DLLs.
`OfficialCampaign` and the `Codex*` / `*Module` classes are what we own and can extend.

### Build pipeline

- `MASTER_CODEX` is defined only in **Debug**. In Debug the codex is built from C# (~1.3s);
  in Release it's loaded from the serialized `Pathos.Codex`. So content-declaration code is
  wrapped in `#if MASTER_CODEX`.
- A `PostBuild` target runs `PathosMaker.exe` from the install to process atlases/albums and
  emit the codex binary. **The installed game is a hard build dependency.**
- `[assembly: InternalsVisibleTo("Pathos")]` etc. — the engine reaches into this assembly's
  internals; modules are declared `internal`.

---

## 2. Build & toolchain reality (gaps to close)

| Item | Status | Action |
|---|---|---|
| Target framework | `net10.0-windows` | Installed SDK is **.NET 9.0.306** — **install the .NET 10 SDK** (or the VS 2026 / Build Tools .NET desktop workload, which brings it) |
| Pathos desktop | **not installed** at `C:\Games\Pathos` | Install it (download from pathos.azurewebsites.net) — required for the DLL refs and `PathosMaker` |
| Build command | VS Code `Ctrl+Shift+B` / VS run `PathosOfficial` | `dotnet build` may work once .NET 10 + DLLs present, but the canonical path is the IDE build that triggers `PathosMaker` |
| License | **CC BY-NC 4.0** (`license.txt`) | Keep intact; never commercialise; credit upstream |

> Until Pathos + .NET 10 are installed, the project **cannot compile** (missing DLL
> references). All implementation work is blocked on this; doc/design work is not.

---

## 3. Answers to the Phase 0 questions (from the spec)

**1. Where are modules defined?**
`Modules/*.cs`, each a `Module` subclass with `Execute(Generator)`. Registered in
`PathosOfficial.cs`. A new mode = a new `EchoesModule : Module` added there.

**2. Where are dungeon generators defined?**
Inside each module's `Execute`. `NethackModule` (6191 lines) is the reference: it builds a
`Dungeon` site of stacked levels plus themed sub-sites (Mines, Minetown, Lost Chambers,
Labyrinth, Sokoban, Fort Ludios, Elf Kingdom). `SPD*` files are a Shattered-Pixel-Dungeon-style
procedural level generator library used by `SPDModule`. The `Generator` API (engine) exposes
`PlaceRoom`, `PlaceShop`, `PlaceCharacter`, `PlaceTrap`, `StartSquare`, dialogues, etc.

**3. Where are classes/archetypes defined?**
`Codex/PathosClasses.cs` (1439 lines) — the 13 NetHack classes. `Codex/PathosHeroes.cs`
binds a starting `Hero` = race `Entity` + `Class` + pet. New disciplines/classless start =
new content here.

**4. Where are item/spell identification systems defined?**
Content for items/spells: `Codex/PathosItems.cs`, `Codex/PathosSpells.cs`,
`Codex/PathosSchools.cs`. **The identification *state machine* (which items are unidentified
per run, how identity is learned) is engine-side** — not in this repo. Confirm exposure via
DLL inspection.

**5. Where are save/profile systems defined?**
**Not in this repo.** Save/serialization of a playthrough is engine-side. `CodexGovernor`
only serializes the *content codex*, not savegames. This is the crux for soul persistence.

**6. Where is death/game-over handled?**
**Not in this repo.** Death, game-over, and run-end are engine-side. No content/module code
references a death hook. Confirm whether `Module`/`Campaign`/`Adventure` expose any death or
run-lifecycle callback via DLL inspection.

**7. Where can an optional new mode be registered?**
`PathosOfficial.cs` → `AddModule(new EchoesModule(Codex))`. `Module`'s constructor takes
`Handle`, `Name`, `Description`, `Colour`, `Author`, `RequiresMasterMode`. Setting
`RequiresMasterMode: false` makes it appear in the normal module list. **This cleanly
satisfies "optional mode, classic untouched."**

**8. What is safe to extend without breaking official modules?**
- Adding a **new module** — fully isolated; other modules untouched.
- Adding **new codex content** (new classes, items, monsters, standings, dialogue) — additive.
- Risk: editing shared `Codex/*` entries used by existing modules. Prefer *adding* new named
  content over *mutating* existing entries.

---

## 4. Spec concept → Pathos concept mapping

| Spec term | Native Pathos equivalent | Notes |
|---|---|---|
| Mode (Echoes) | `Module` | `RequiresMasterMode: false` to be selectable |
| Region | `Site` (a set of `Level`s sharing a theme) | Mines/Elf Kingdom already work this way |
| Floor | `Level` + `Map` | `SetDifficulty`, `SetAtmosphere` |
| Region boss / settlement | placed characters + dialogue + shops | see `NethackModule` Minetown |
| Faction reputation | nearest existing: `Standing` (damned→exalted) | but `Standing` is global alignment, not per-faction; real factions = new content |
| Class / discipline | `Class` + `Hero` | classless start = a deliberately weak custom class |
| Titles | new content + dialogue flags | gameplay effect needs engine support to be more than flavour |
| Identified items persist | engine identification + engine save | **gated on engine APIs** |
| Timeline seed / reset on death | engine campaign/seed | **gated on engine APIs** |
| Memory fragments (meta-currency) | none | needs persisted profile = **gated on engine APIs** |

Useful discovery: `Standing` already carries a `SpawnModifier` that scales encounter
difficulty by reputation tier — a possible lever for faction-flavoured spawn changes *within*
a run, even without cross-run persistence.

---

## 5. The engine boundary & the persistence question (make-or-break)

The spec's emotional core — "the dungeon resets, the soul remembers" — requires **state that
outlives a character's death and is reloaded into the next run**. In this repo, a module's
`Execute(Generator)` runs **once, at world creation, for a single adventure**. There is no
content-layer hook for death, run-end, or cross-run storage.

So everything hinges on what the **engine** exposes to a `Module`/`Campaign`. Three outcomes:

- **Best case:** the engine exposes a per-profile or per-campaign persistent store and a
  run-lifecycle/death callback a module can subscribe to. Then most of the spec is buildable.
- **Middle case:** the engine has a generic save blob but no module hooks. We might piggyback
  via existing mechanisms (e.g. a persistent "bones"/legacy file, achievements, or a save the
  module can read at generation time). Partial Echoes.
- **Worst case:** no module-accessible persistence at all. Then true cross-death soul
  progression is **not implementable without upstream engine changes**, and Echoes must be
  rescoped to a within-run themed campaign (still shippable, but not the time-loop fantasy).

**We cannot tell which case we're in from this repo.** Resolving it is the top priority once
the DLLs are installed.

---

## 6. Revised, feasibility-tiered strategy

Re-order the spec's roadmap around the engine boundary so we ship value early and de-risk the
unknown before investing in it.

### Tier A — buildable in this repo today (no engine unknowns)
1. **`EchoesModule` skeleton** — selectable mode, custom intro/conclusion/track, a small
   generated world. (Spec Phase 1, minus same-seed-reset.)
2. **Region-structured generation** — 2–3 themed Sites (e.g. Dwarven Mines, Goblin Warrens,
   Ancient Library) with distinct monster/loot/atmosphere and one regional boss each.
   (Spec Phase 6 — promoted earlier because it's the most certain win.)
3. **Classless "Soulbound Wanderer" start** — a weak custom `Class`/`Hero`.
   (Spec Phase 4, content portion.)
4. **Flavour factions** — themed settlements/dialogue; optionally drive spawn via `Standing`.

> Tier A alone = a real, shareable Pathos variant: "a new themed, region-based campaign with a
> classless start." It satisfies the spec's "Shareable Branch Goal" minus the loop mechanic.

### Tier B — gated on the DLL inspection (do step 0 first)
0. **Inspect the engine DLLs** (section 7). Decide best/middle/worst case.
5. **Soul persistence** (SoulProfile, identified-item/loop-count/memory-fragment retention).
6. **Death → same-seed reset + loop counter + death summary.**
7. **Memory tree, discipline unlocks, titles-with-effects, dynamic events, collapse escape.**

Each Tier B item should be re-scoped to the persistence mechanism the engine actually offers.

---

## 7. Remaining Phase 0 task — inspect the installed engine DLLs

Once Pathos is installed, inspect the public/InternalsVisible surface of these types (via the
DLLs in `C:\Games\Pathos`, an object browser, or a decompiler such as ILSpy):

- `Campaign` — lifecycle methods beyond `Make`? Any per-run or per-profile state?
- `Module` — any callbacks besides `Execute`? Death/victory/turn hooks? Seed access?
- `Generator` / `Adventure` / `World` — is the seed exposed/settable? `Adventure.Abolition`
  hints at run-flags; what else is on `Adventure`?
- Any type named like `Profile`, `Save`, `Persistence`, `Legacy`, `Bones`, `MetaProgress`,
  `Identification`, `Discovery`.
- How identification state is stored and whether a module can read/seed it.

Record findings by updating this file, then unblock Tier B planning.

---

## 8. License confirmation

`license.txt` = **Creative Commons Attribution-NonCommercial 4.0 International (CC BY-NC 4.0)**.
Obligations for this variant: attribute upstream Pathos, keep `license.txt` intact, do not
commercialise in any form. `docs/ECHOES_OF_THE_SOUL.md` should credit
`https://github.com/callanh/pathos-official`.

---

## Status

- [x] Repo structure, entry point, module/codex model understood
- [x] Build pipeline + toolchain gaps identified (.NET 10 + Pathos install required)
- [x] License confirmed (CC BY-NC 4.0)
- [x] Spec→Pathos concept mapping done
- [x] Engine boundary identified; persistence flagged as make-or-break
- [x] **Engine DLL inspection** — DONE (see RESOLVED VERDICT at top). Tier B (soul persistence,
      death reset, memory fragments) is **not implementable in this fork** — engine-gated.
- [x] First clean build of untouched fork — **succeeds** (.NET 10 + installed Pathos DLLs;
      `dotnet build -c Debug`, 0 errors, PathosMaker runs the asset pipeline)
- [ ] **Decision needed:** rescope Echoes to Tier A (themed region campaign) and/or propose
      engine hooks to upstream for the soul-persistence layer
