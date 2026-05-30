# Pathos: Echoes of the Soul — Claude Code Branch Spec

## Purpose

This document is a detailed implementation guide for creating a shareable Pathos variant branch tentatively called **Pathos: Echoes of the Soul**.

The goal is not to make a small balance mod. The goal is to create a new optional gameplay mode that preserves what makes Pathos fun—deep item interactions, roguelike danger, class fantasy, monsters, magic, and discovery—while fixing the parts that become repetitive after level 30: generic dungeon floors, boring ascent gameplay, weak late-game pacing, and the feeling that death deletes too much meaningful progress.

The core idea:

> The dungeon resets. The soul remembers.

This should be implemented as an optional mode, not as a replacement for standard Pathos.

---

## Source Repository Assumptions

The official Pathos content repository is public and intended for variants.

Repository:

```text
https://github.com/callanh/pathos-official
```

Important constraints:

- This should be implemented as a fork or branch of the official content/module repository.
- Do not commercialize this variant.
- Preserve upstream licensing notices.
- Keep all new code, content, and documentation clearly separated from official content when practical.
- Avoid destructive changes to existing modules until the new mode has a working prototype.

Recommended branch name:

```text
feature/echoes-of-the-soul
```

Recommended top-level documentation file:

```text
docs/ECHOES_OF_THE_SOUL.md
```

Recommended implementation strategy:

- First build the mode as a minimally invasive optional module.
- Then add soul persistence.
- Then add region-based dungeon generation.
- Then add factions/titles.
- Then add late-game collapse/escape mechanics.

---

## High-Level Design

### Current Problem

Pathos is rich mechanically, but many dungeon levels feel structurally similar. The standout areas are memorable because they feel like places, not just randomly connected rooms. Examples of memorable content patterns:

- Dwarven areas/mines.
- Medusa-style themed areas.
- Elven areas.
- Faction-like or civilization-like spaces.

The weaker experience is:

- Generic square/box room layouts.
- Repeated corridor-room floors.
- Long late-game progression with less surprise.
- Tedious return-to-surface phase after the major objective.
- Death erasing too much progress, making repeated early-game exploration feel stale.

### New Design Pillars

1. **Knowledge is progression.**
   The player loses physical things on death, but retains internal knowledge.

2. **The soul persists, not the body.**
   Death rewinds the timeline. The dungeon, monsters, items, shops, and rooms reset to their original state, but the player remembers what was learned.

3. **Start classless. Learn disciplines.**
   The player begins weak and classless. Classes become learned disciplines that can be unlocked, trained, evolved, and carried through death.

4. **Generate regions, not isolated floors.**
   The dungeon should be composed of themed regions with identity, factions, bosses, hazards, and local stories.

5. **Titles become reputation.**
   Titles earned by the soul affect factions and unlock options.

6. **Late game becomes a second act.**
   After the final objective, the return should become a dramatic collapse/escape sequence, not tedious stair climbing.

---

## Mode Name

Working names:

- Echoes of the Soul
- Soulbound Mode
- Time Loop Mode
- Chrono Descent
- The Remembered

Use this internally for now:

```text
Echoes of the Soul
```

---

## User-Facing Mode Description

> In Echoes of the Soul, death rewinds the dungeon to the beginning. Your body is lost, but your soul remembers. Maps, discoveries, identified items, learned disciplines, titles, faction impressions, and monster knowledge persist across loops. Start as a classless wanderer and gradually become a legendary soul capable of mastering the dungeon through memory, training, and reputation.

---

## Core Gameplay Loop

1. Player starts as a weak classless character.
2. Dungeon is generated from a fixed seed for the current timeline.
3. Player explores, fights, identifies items, learns spells, discovers regions, meets factions, earns titles.
4. Player dies.
5. Timeline resets to the initial dungeon state.
6. Player loses external/physical progress.
7. Player keeps internal/soul progress.
8. Player restarts in the same timeline with retained knowledge.
9. Player uses knowledge to route better, survive longer, unlock classes, and solve region/faction problems.
10. When the player finally wins, a new timeline/new seed can begin as New Timeline Plus.

---

## Persistence Rules

### Retained After Death

These are internal to the soul and should persist across loops:

#### Knowledge

- Identified item types.
- Potion identities.
- Scroll identities.
- Wand identities.
- Ring/amulet identities.
- Spell knowledge.
- Known monster traits.
- Known boss locations.
- Known shop locations.
- Known region layout summaries.
- Known secret rooms.
- Known trap locations, if discovered.
- Known quest facts.
- Bestiary entries.
- Map memory, depending on memory upgrades.

#### Soul Progression

- Memory fragments.
- Discipline/class unlocks.
- Discipline experience/mastery.
- Titles.
- Achievements.
- Faction impressions/reputation.
- Major story discoveries.
- Completed timeline history.

#### Optional Retention by Upgrade

Some things should only persist after specific memory upgrades:

- Full maps.
- Shop inventory knowledge.
- Exact item locations.
- Trap locations.
- Monster patrol knowledge.
- Boss weaknesses.

### Lost After Death

These are external to the current body and should reset:

- Current level.
- Current attributes.
- Current HP/MP.
- Gold.
- Equipment.
- Inventory.
- Consumables.
- Current blessings/curses.
- Physical quest items.
- Temporary buffs.
- Current companion state, unless later redesigned as soulbound companions.

---

## Timeline Reset Rules

The mode should not be a persistent-world corpse-recovery mode. On death:

- The world rewinds.
- Monsters return.
- Items return.
- Shops return.
- Doors reset.
- Secrets are hidden again.
- Faction settlements return to their initial state.
- Bosses return.
- The same seed/layout should be reused until timeline victory.

The player should feel like they are replaying the same timeline with better knowledge.

### Timeline Seed

Each Echoes run should have a stable timeline seed.

Implementation concept:

```text
SoulProfile.CurrentTimelineSeed
```

The seed should remain fixed until the player completes the timeline or manually abandons it.

### New Timeline Plus

After victory:

- Generate a new seed.
- Keep soul progression.
- Reset discovered maps for the new timeline.
- Keep generalized knowledge and titles.
- Optionally add timeline difficulty modifiers.

---

## Soul Profile Data Model

Create a new persistent profile object. Exact implementation depends on existing save architecture.

Suggested conceptual model:

```csharp
public sealed class SoulProfile
{
    public Guid SoulId { get; set; }
    public string SoulName { get; set; }
    public int LoopCount { get; set; }
    public int TimelineCount { get; set; }
    public int CurrentTimelineSeed { get; set; }
    public int MemoryFragments { get; set; }

    public HashSet<string> IdentifiedItems { get; set; }
    public HashSet<string> KnownSpells { get; set; }
    public HashSet<string> KnownMonsters { get; set; }
    public HashSet<string> KnownRegions { get; set; }
    public HashSet<string> KnownSecrets { get; set; }
    public HashSet<string> KnownTraps { get; set; }
    public HashSet<string> KnownShops { get; set; }
    public HashSet<string> KnownBosses { get; set; }

    public Dictionary<string, DisciplineProgress> Disciplines { get; set; }
    public HashSet<string> UnlockedDisciplines { get; set; }
    public HashSet<string> Titles { get; set; }
    public Dictionary<string, int> FactionReputation { get; set; }
    public Dictionary<string, TimelineMemory> TimelineMemories { get; set; }
    public Dictionary<string, int> Counters { get; set; }
}

public sealed class DisciplineProgress
{
    public string DisciplineId { get; set; }
    public int Rank { get; set; }
    public int Experience { get; set; }
    public bool Unlocked { get; set; }
    public bool Mastered { get; set; }
}

public sealed class TimelineMemory
{
    public int TimelineSeed { get; set; }
    public HashSet<string> RevealedMapCells { get; set; }
    public HashSet<string> DiscoveredRegionIds { get; set; }
    public HashSet<string> DiscoveredSecretIds { get; set; }
    public HashSet<string> DiscoveredTrapIds { get; set; }
    public HashSet<string> DiscoveredShopIds { get; set; }
    public HashSet<string> DiscoveredBossIds { get; set; }
}
```

Do not implement the exact model blindly. First inspect the existing save/profile systems and adapt to the repository style.

---

## Death Handling

Find the existing death/game-over flow.

Add an Echoes-specific branch:

```text
if current mode == EchoesOfTheSoul:
    award memory fragments
    update soul knowledge
    update discipline progress
    update titles
    update faction impressions
    reset run state
    regenerate dungeon from current timeline seed
    create new classless body with available discipline training options
else:
    use normal death flow
```

### Memory Fragment Award Formula

Initial simple formula:

```text
MemoryFragmentsGained = floor(MaxDepthReached * 1.5)
                      + UniqueRegionsDiscovered * 5
                      + BossesDefeated * 10
                      + NewItemIdentifications * 1
                      + NewSpellDiscoveries * 2
                      + NewTitlesEarned * 5
```

Caps may be needed for balance.

### Death Summary Screen

Add a death summary for Echoes mode:

- Loop number.
- Deepest floor reached.
- Regions discovered.
- New items identified.
- New spells learned.
- Titles earned.
- Discipline progress gained.
- Memory fragments earned.
- Faction impression changes.

---

## Classless Start

In Echoes mode, do not start with a normal class selection.

Starting identity:

```text
Soulbound Wanderer
```

Baseline:

- Weak stats.
- No strong class abilities.
- Basic equipment only.
- Can learn any discipline through play.

The player may eventually choose one or more learned disciplines at start, but only after unlocking them.

---

## Discipline System

Classes become learnable disciplines.

### Basic Disciplines

#### Warrior

Unlock conditions:

- Kill 100 enemies with melee weapons, or
- Defeat a regional boss using mostly melee.

Benefits:

- Weapon proficiency training.
- Improved melee accuracy.
- Improved armor use.

#### Mage

Unlock conditions:

- Cast 100 spells, or
- Learn 10 unique spells.

Benefits:

- Improved spell learning.
- Reduced spell failure.
- Mana efficiency.

#### Rogue

Unlock conditions:

- Disarm traps.
- Discover secret doors.
- Complete stealth or theft objectives.

Benefits:

- Trap detection.
- Sneak attacks.
- Lockpicking.

#### Cleric

Unlock conditions:

- Use healing/divine magic.
- Restore allies.
- Complete shrine-related objectives.

Benefits:

- Divine spell affinity.
- Improved healing.
- Undead resistance.

#### Ranger

Unlock conditions:

- Use ranged weapons.
- Tame or fight beasts.
- Survive wilderness-themed regions.

Benefits:

- Ranged accuracy.
- Better tracking.
- Animal affinity.

#### Alchemist

Unlock conditions:

- Identify potions.
- Brew or use potions.
- Discover labs or herbal regions.

Benefits:

- Potion identification.
- Improved potion effects.
- Crafting options.

### Hybrid Disciplines

Unlock when two base disciplines reach sufficient progress.

Examples:

| Hybrid | Requirements | Theme |
|---|---|---|
| Spellblade | Warrior + Mage | Weapon magic |
| Paladin | Warrior + Cleric | Holy knight |
| Shadow Dancer | Rogue + Monk/Rogue equivalent | Mobility/stealth |
| Arcane Scholar | Mage + Alchemist | Identification/crafting |
| Witch Hunter | Warrior + Alchemist/Cleric | Anti-magic/anti-undead |
| Beast Warden | Ranger + Cleric | Companions/nature |

### Discipline Selection at New Loop

At the beginning of each loop, allow the player to choose a training focus:

- No focus: classless wanderer, faster general learning.
- One discipline: stronger starting direction.
- Two disciplines: hybrid path, slower mastery.

Do not allow the player to become overpowered too quickly.

---

## Memory Tree

Memory Fragments purchase upgrades.

### Cartography Branch

#### Cartography I

- Retain remembered map outlines for first 5 floors.

#### Cartography II

- Retain remembered map outlines for first 10 floors.

#### Cartography III

- Retain remembered map outlines for all discovered floors in the current timeline.

#### Secret Sense

- Previously discovered secret rooms appear as faint memory markers.

### Identification Branch

#### Alchemical Memory

- Potion identities persist.

#### Runic Memory

- Scroll identities persist.

#### Wand Memory

- Wand identities persist.

#### Relic Memory

- Ring/amulet identities persist.

### Bestiary Branch

#### Monster Scholar

- Known monster information persists.

#### Weakness Recall

- Previously discovered weaknesses are visible.

#### Boss Memory

- Boss traits and locations persist.

### Faction Branch

#### Aura of Recognition

- Factions react more strongly to titles.

#### Diplomat's Echo

- Positive faction reputation decays less between timelines.

#### Infamous Soul

- Negative faction reputation unlocks intimidation options.

### Discipline Branch

#### Practice Memory

- Discipline experience persists.

#### Faster Relearning

- Reacquire known discipline abilities faster during a loop.

#### Hybrid Insight

- Reveal possible hybrid discipline paths.

---

## Title System

Titles are permanent labels earned by accomplishments.

They are not just achievements. They should affect gameplay.

### Title Data Model

Conceptual model:

```csharp
public sealed class SoulTitle
{
    public string Id { get; set; }
    public string DisplayName { get; set; }
    public string Description { get; set; }
    public string Category { get; set; }
    public Dictionary<string, int> FactionModifiers { get; set; }
    public List<string> Unlocks { get; set; }
    public bool HiddenUntilEarned { get; set; }
}
```

### Example Titles

#### The Looper

Condition:

- Die 10 times in Echoes mode.

Effect:

- Unlocks special time-loop dialogue.

#### The Remembered

Condition:

- Complete one timeline after at least 25 loops.

Effect:

- Factions occasionally sense familiarity.

#### Dwarven Friend

Condition:

- Save or assist a dwarven settlement.

Effect:

- Dwarves start friendlier.
- Dwarven merchants may offer better prices.

#### Goblin Bane

Condition:

- Kill many goblins or destroy a goblin stronghold.

Effect:

- Goblins become more hostile.
- Dwarves may respect the player.

#### Goblin Kingmaker

Condition:

- Help a goblin faction overthrow its ruler.

Effect:

- Goblin factions may negotiate.
- Dwarves may distrust the player.

#### Dragon Slayer

Condition:

- Kill a dragon.

Effect:

- Dragons react with fear, anger, or respect depending on type.

#### Archmage

Condition:

- Master multiple schools of magic.

Effect:

- Mage factions recognize the player.
- Unlocks advanced training.

#### Betrayer of Elves

Condition:

- Harm or betray an elven faction.

Effect:

- Elves distrust the player.
- Dark factions may approve.

### Active vs Passive Titles

Implement in stages:

Stage 1:

- Titles are passive achievements with small stat/reputation modifiers.

Stage 2:

- Allow the player to equip/display one active title.

Stage 3:

- Factions react to both active title and full title history.

---

## Faction System

Add region/faction identity to dungeon generation.

### Core Factions

Start with these:

- Dwarves
- Elves
- Goblins
- Necromancers
- Vampires
- Dragons
- Mages
- Cultists
- Beasts/Spiders

### Faction Reputation

Use a simple integer scale:

```text
-100 hostile
-50 suspicious
0 neutral
50 friendly
100 allied
```

### Faction Reaction Inputs

Faction reaction should consider:

- Current reputation.
- Titles.
- Race/species/class/discipline if available.
- Recent actions in current loop.
- Historical actions from the soul profile.

### Faction Consequences

Examples:

- Merchants change prices.
- Guards allow/deny entry.
- Faction bosses offer quests.
- Patrols become hostile or friendly.
- Region layouts include more/less faction presence.
- Assassins may spawn if the player is infamous.

---

## Region-Based Dungeon Generation

Replace or supplement generic floor generation with multi-floor regions.

### Design Rule

Do not generate 40 disconnected generic floors.

Generate a sequence of regions, where each region has:

- Theme.
- Architecture.
- Monster table.
- Loot table.
- Hazards.
- Faction presence.
- Quest hooks.
- Optional settlement.
- Regional boss.
- Secrets.
- Transition to next region.

### Initial Region List

#### Dwarven Mines

Identity:

- Mining tunnels.
- Forges.
- Stone halls.
- Dwarven settlements.
- Ore veins.

Gameplay:

- Strong crafting opportunities.
- Dwarf/goblin conflict.
- Mine collapses.
- Forge hazards.

Boss ideas:

- Goblin siege chief.
- Corrupted forge guardian.
- Ancient stone golem.

#### Goblin Warrens

Identity:

- Cramped tunnels.
- Traps.
- Ambushes.
- Messy camps.

Gameplay:

- High enemy density.
- Sneak routes.
- Chieftain politics.
- Prisoners to rescue.

Boss ideas:

- Goblin King.
- Hobgoblin warlord.
- Goblin shaman.

#### Elven Enclave

Identity:

- Underground forest/city.
- Bridges.
- Moonlit groves.
- Archers and magic.

Gameplay:

- Social/diplomatic options.
- Nature magic.
- Sacred trees/shrines.
- Elf/dark faction conflict.

Boss ideas:

- Corrupted elven prince.
- Ancient treant.
- Dark elf assassin.

#### Medusa's Garden

Identity:

- Statues.
- Marble ruins.
- Serpents.
- Petrification hazards.

Gameplay:

- Line-of-sight danger.
- Reflection/mirror tools.
- Statue secrets.
- Petrified NPCs.

Boss ideas:

- Medusa queen.
- Gorgon oracle.

#### Ancient Library

Identity:

- Scroll rooms.
- Animated books.
- Constructs.
- Forbidden knowledge.

Gameplay:

- Spell discovery.
- Lore progression.
- Dangerous reading.
- Knowledge puzzles.

Boss ideas:

- Mad archmage.
- Living spellbook.
- Librarian construct.

#### Flooded Aqueduct

Identity:

- Water channels.
- Bridges.
- Flooded chambers.
- Aquatic monsters.

Gameplay:

- Movement constraints.
- Drowning/flood hazards.
- Water-based secrets.
- Optional valves/drainage objectives.

Boss ideas:

- Kraken.
- Water elemental lord.
- Drowned king.

#### Vampire Court

Identity:

- Gothic halls.
- Nobles.
- Blood rituals.
- Thralls.

Gameplay:

- Diplomacy or combat.
- Day/night-like mechanics if possible.
- Charm/resistance checks.
- Social deception.

Boss ideas:

- Vampire duke.
- Blood queen.

#### Infernal Forge

Identity:

- Lava.
- Demonic machinery.
- Blacksmith demons.
- Cursed artifacts.

Gameplay:

- Heat/lava hazards.
- High-risk crafting.
- Demon faction.
- Weapon upgrades with corruption risk.

Boss ideas:

- Forge demon.
- Hellsmith.

#### Dragon Peaks / Deep Dragon Lair

Identity:

- Vast caverns.
- Treasure hoards.
- Vertical chambers.
- Elemental dragon zones.

Gameplay:

- High risk/high reward.
- Dragon diplomacy.
- Hoard theft.
- Environmental elemental hazards.

Boss ideas:

- Ancient dragon.
- Dragon tyrant.

#### Void Realm

Identity:

- Reality tears.
- Strange geometry.
- Teleportation.
- Time instability.

Gameplay:

- Late-game region.
- Memory/soul mechanics become relevant.
- Timeline corruption.

Boss ideas:

- Chronomancer.
- Void monarch.
- Time-devouring entity.

---

## Region Selection Algorithm

Initial simple version:

```text
Early game regions:
- Dwarven Mines
- Goblin Warrens
- Spider/Fungal Caves

Mid game regions:
- Elven Enclave
- Ancient Library
- Flooded Aqueduct
- Medusa's Garden
- Vampire Court

Late game regions:
- Infernal Forge
- Dragon Lair
- Necromancer Citadel
- Void Realm
```

Generate a timeline as:

```text
Start Area
Early Region A
Early Region B
Mid Region A
Mid Region B
Late Region A
Late Region B
Final Region
Escape Collapse
```

Each region can be 2–5 floors.

### Memory Compatibility

Each generated region should have stable region IDs so the soul can remember discoveries across loops.

Suggested ID pattern:

```text
{TimelineSeed}:{RegionType}:{RegionIndex}:{FloorIndex}:{FeatureId}
```

---

## Dynamic Events

Events should make regions feel alive.

Start small.

### Event Examples

#### Goblin Siege

- Goblins attack a dwarven settlement.
- Player may help dwarves, help goblins, or ignore.
- Grants faction reputation and titles.

#### Undead Plague

- Necromancers corrupt an existing region.
- Dead creatures rise again.
- Cleric/Paladin paths gain opportunities.

#### Dragon Migration

- Dragon presence appears in a region that normally would not have dragons.
- Adds hoards and danger.

#### Mage Experiment Gone Wrong

- Ancient Library or Mage Academy gets unstable magic effects.
- Adds random spell effects and constructs.

#### Vampire Masquerade

- Social/diplomatic event in Vampire Court.
- Rogue/Mage/Cleric choices matter.

---

## Late-Game Escape / Dungeon Collapse

Remove or reduce the boring ascent.

After defeating the final boss or completing the final objective, trigger a second phase:

```text
The timeline destabilizes. Escape before the dungeon collapses.
```

### Collapse Mechanics

Possible effects:

- New exits open.
- Stairs shift.
- Lava/flooding spreads.
- Reality tears spawn monsters.
- Timed pressure.
- Previously safe regions become dangerous.
- Shortcuts discovered earlier matter.

### Design Goal

The escape should be tense and memorable, not a 30-floor slog.

### Implementation Stage 1

Simplify:

- Generate a smaller escape route composed of several region fragments.
- Add increasing monster pressure.
- Add a final surface gate.

### Implementation Stage 2

Use remembered shortcuts, faction allies, and titles to modify escape.

Examples:

- Dwarven Friend: dwarves open a mine elevator.
- Dragon Slayer: dragons block escape route.
- Archmage: magical shortcut appears.
- Goblin Kingmaker: goblins guide player through warrens.

---

## Additional Mechanics Worth Improving

### 1. Rumor System

NPCs, taverns, books, inscriptions, and shrines should provide useful clues.

Examples:

- "The third level hides a cracked wall near running water."
- "The dwarves fear a goblin king below the mines."
- "A mirror shield can turn the gaze of stone."

Rumors should be saved as soul knowledge.

### 2. Region Objectives

Each region should have something to do besides finding stairs.

Examples:

- Rescue prisoners.
- Restore a forge.
- Kill a regional boss.
- Negotiate a truce.
- Drain a flooded level.
- Recover a stolen artifact.
- Break a curse.

### 3. Better Secret Content

Secrets should sometimes unlock whole areas, not just small rooms.

Examples:

- Hidden dwarven kingdom.
- Forgotten shrine.
- Secret faction route.
- Ancient vault.

### 4. Rival Adventurers

Add optional NPC adventurers:

- Some compete for loot.
- Some can be allies.
- Some become enemies.
- Some may remember the player only indirectly through title aura.

### 5. Legacy Echoes

Because the mode is soul/time-loop themed, add rare encounters with echoes of past loops.

Examples:

- A ghostly version of the player warns about a boss.
- A memory remnant points to a secret.
- A corrupted echo becomes a miniboss.

### 6. Artifact Memory

Most gear is lost on death, but legendary artifacts can become known to the soul.

Possible design:

- Artifact physical item resets with timeline.
- Soul remembers where/how to find it.
- Repeated use across loops unlocks lore or mastery.

### 7. Companion Improvements

Companions should matter more.

Ideas:

- Named companions.
- Loyalty.
- Personal quests.
- Faction ties.
- Optional soulbound companion path, unlocked late.

### 8. Better Floor Layout Variety

Avoid overusing 3x3 or box-room layouts.

Add generators for:

- Caverns.
- Fortresses.
- Cities.
- Mines.
- Vertical-feeling bridge/chasm maps.
- Flooded maps.
- Gardens.
- Libraries.
- Labyrinths.
- Temples.
- War camps.

### 9. Meaningful Shops and Settlements

Shops should be placed in believable settlements.

Examples:

- Dwarven blacksmith.
- Goblin junk trader.
- Elven apothecary.
- Vampire blood merchant.
- Mage scroll archive.

### 10. Multiple Endings

Possible endings:

- Escape the dungeon.
- Break the time loop.
- Become the dungeon's ruler.
- Become a lich.
- Join the dragons.
- Save a faction civilization.
- Destroy all factions.

---

## Implementation Roadmap

### Phase 0 — Repository Audit

Claude Code should first inspect the repository and answer:

1. Where are modules defined?
2. Where are dungeon generators defined?
3. Where are classes/archetypes defined?
4. Where are item/spell identification systems defined?
5. Where are save/profile systems defined?
6. Where is death/game-over handled?
7. Where can an optional new mode be registered?
8. What is safe to extend without breaking official modules?

Deliverable:

```text
docs/ECHOES_REPO_AUDIT.md
```

Do not start broad implementation before this audit.

### Phase 1 — Mode Skeleton

Goal:

Create Echoes mode as selectable but minimally functional.

Tasks:

- Add mode identifier.
- Add documentation.
- Add basic classless start if possible.
- Reuse existing dungeon generation at first.
- Add death handling branch that restarts from same seed.
- Add loop counter.

Acceptance criteria:

- Player can start Echoes mode.
- Player death restarts the same seed.
- Loop count increments.
- Normal Pathos mode remains unaffected.

### Phase 2 — Soul Profile Persistence

Goal:

Save knowledge and memory fragments between deaths.

Tasks:

- Add SoulProfile or equivalent.
- Persist identified items.
- Persist known spells.
- Persist loop count.
- Persist memory fragments.
- Add death summary.

Acceptance criteria:

- Identified item knowledge survives death.
- Memory fragments are awarded.
- Loop count persists.
- Save/load works.

### Phase 3 — Memory Tree MVP

Goal:

Add basic memory upgrades.

Initial upgrades:

- Potion Memory.
- Scroll Memory.
- Monster Scholar.
- Cartography I.

Acceptance criteria:

- Player can spend memory fragments.
- Upgrade effects are visible.
- Upgrades persist across death.

### Phase 4 — Discipline MVP

Goal:

Implement classless start and learned disciplines.

Initial disciplines:

- Warrior.
- Mage.
- Rogue.
- Cleric.
- Alchemist.

Tasks:

- Track discipline-relevant counters.
- Unlock disciplines through play.
- Allow discipline focus at loop start.
- Add basic benefits.

Acceptance criteria:

- Player starts classless.
- Player can unlock at least one discipline.
- Discipline persists after death.
- Discipline focus modifies future starts.

### Phase 5 — Title MVP

Goal:

Add titles and simple effects.

Initial titles:

- The Looper.
- Dwarven Friend.
- Goblin Bane.
- Dragon Slayer.
- Archmage.

Acceptance criteria:

- Titles are earned by events/counters.
- Titles persist.
- Titles appear in UI or character sheet.
- At least one title changes faction reaction or gameplay.

### Phase 6 — Region Generator MVP

Goal:

Make the dungeon feel less generic.

Initial regions:

- Dwarven Mines.
- Goblin Warrens.
- Elven Enclave.
- Medusa's Garden.
- Ancient Library.

Tasks:

- Add region selection layer above floor generation.
- Assign floors to region types.
- Modify monster/loot/theme tables per region.
- Add at least one regional boss.
- Add region IDs for memory.

Acceptance criteria:

- A timeline includes named regions.
- Regions have distinct enemy/loot/layout patterns.
- Player can discover and remember regions.

### Phase 7 — Faction MVP

Goal:

Make titles and actions affect groups.

Initial factions:

- Dwarves.
- Goblins.
- Elves.
- Mages.

Tasks:

- Add reputation values.
- Add title modifiers.
- Add simple faction reactions.
- Add merchant/guard behavior if possible.

Acceptance criteria:

- Helping dwarves affects dwarf reputation.
- Killing goblins affects goblin reputation.
- Titles affect reaction.
- Reactions persist as soul impressions.

### Phase 8 — Dynamic Events MVP

Goal:

Add region events.

Initial event:

- Goblin Siege of Dwarven Mine.

Acceptance criteria:

- Event appears in some timelines.
- Player can affect outcome.
- Outcome grants title/reputation.
- Event is replayable after death because the timeline resets.
- Soul remembers event discovery.

### Phase 9 — Collapse Escape MVP

Goal:

Replace boring return climb with an exciting finale.

Tasks:

- Detect final objective completion.
- Trigger collapse state.
- Create shortened escape path.
- Add escalating hazards.
- Add faction/title-based shortcuts if feasible.

Acceptance criteria:

- Endgame escape is shorter and more intense.
- Player does not simply walk up 30 unchanged floors.
- Prior knowledge helps escape.

### Phase 10 — Balance and Polish

Tasks:

- Tune memory fragment rewards.
- Tune discipline unlock rates.
- Tune region depth/difficulty.
- Reduce overpowered stacking.
- Add UI text.
- Add documentation.
- Add debug commands if useful.

---

## Testing Plan

### Unit-Level Tests / Debug Checks

If the repo supports tests, add tests for:

- SoulProfile serialization.
- Death reset preserving soul data.
- Timeline seed stability.
- Memory fragment calculation.
- Discipline unlock counters.
- Title unlock conditions.
- Faction modifier calculations.

### Manual Test Scenarios

#### Test 1 — Timeline Reset

1. Start Echoes mode.
2. Record first floor layout.
3. Die.
4. Restart.
5. Confirm first floor layout is the same.
6. Confirm items/monsters reset.
7. Confirm loop count increased.

#### Test 2 — Item Identification Persistence

1. Identify a potion.
2. Die.
3. Restart.
4. Confirm potion identity is remembered if appropriate upgrade/system is active.

#### Test 3 — Classless Start

1. Start Echoes mode.
2. Confirm no standard class is selected.
3. Perform actions toward Mage or Warrior.
4. Die.
5. Confirm discipline progress persists.

#### Test 4 — Title Unlock

1. Meet title condition.
2. Confirm title appears.
3. Die.
4. Confirm title persists.

#### Test 5 — Region Identity

1. Generate a timeline.
2. Confirm named regions appear.
3. Confirm region-specific monsters/loot/hazards.
4. Die.
5. Confirm region layout is discoverable again but remembered by the soul.

#### Test 6 — Faction Reaction

1. Earn Dwarven Friend.
2. Die.
3. Restart.
4. Encounter dwarves.
5. Confirm altered reaction.

#### Test 7 — Collapse Escape

1. Complete final objective.
2. Confirm collapse state triggers.
3. Confirm escape route/hazards appear.
4. Confirm victory condition works.

---

## Claude Code Instructions

Use this section as the initial instruction to Claude Code.

```text
You are working in a fork of the official Pathos content/module repository. Build an optional variant mode called "Echoes of the Soul". Do not break normal Pathos gameplay. Do not commercialize or remove license notices.

First, audit the repository and create docs/ECHOES_REPO_AUDIT.md explaining where modules, dungeon generation, classes, save/profile logic, identification, and death/game-over flows are implemented.

Then implement the feature in small commits. Do not attempt everything at once. Start with a mode skeleton that allows a stable timeline seed and loop counter. Then add SoulProfile persistence. Then add memory fragments. Then add classless start and learned disciplines. Then add titles/factions. Then add region-based dungeon generation. Then add collapse escape.

Prefer minimal, reversible changes. Keep new code namespaced or clearly labeled as Echoes/EchoesOfTheSoul. Add comments where the implementation hooks into official systems. Preserve normal mode behavior.

After each phase, update docs/ECHOES_PROGRESS.md with completed work, files changed, current limitations, and manual test steps.
```

---

## Suggested Commit Plan

```text
commit 1: docs: add Echoes of the Soul design spec
commit 2: docs: add repository audit for Echoes implementation hooks
commit 3: feat(echoes): add optional mode skeleton and timeline seed
commit 4: feat(echoes): add soul profile persistence and loop counter
commit 5: feat(echoes): preserve basic knowledge across death
commit 6: feat(echoes): add memory fragments and death summary
commit 7: feat(echoes): add memory tree MVP
commit 8: feat(echoes): add classless wanderer start
commit 9: feat(echoes): add discipline unlock/progress MVP
commit 10: feat(echoes): add title system MVP
commit 11: feat(echoes): add faction reputation MVP
commit 12: feat(echoes): add region generation layer MVP
commit 13: feat(echoes): add Dwarven Mines/Goblin Warrens event MVP
commit 14: feat(echoes): add collapse escape MVP
commit 15: docs: add balance notes and playtest checklist
```

---

## Balance Warnings

### Do Not Make Soul Progress Too Stat-Heavy

Avoid permanent raw stat boosts early.

Bad:

- +10 strength permanently.
- Start with legendary gear.
- Permanent HP stacking.

Good:

- Remember potion identities.
- Unlock training options.
- Know maps.
- Recognize monsters.
- Gain faction options.

### Keep Death Meaningful

Death should hurt because the current body and inventory are lost. But it should not feel like wasted time because knowledge persists.

### Avoid Too Much Certainty

If the player knows literally everything, the game becomes a rote speedrun. Add limited uncertainty later:

- Minor timeline drift.
- Optional corruption.
- Events that vary slightly.
- Faction reactions that depend on current-loop choices.

### Keep Classic Mode Untouched

This is critical. Echoes mode should be optional.

---

## Long-Term Expansion Ideas

### Timeline Drift

After many deaths, 5% of the dungeon mutates.

### Soul Scars

Repeated deaths to the same enemy type create both penalties and insights.

### Nemesis Echoes

Enemies that repeatedly kill the player gain special status within the timeline.

### Faction Endings

Win by aligning with a faction rather than simply escaping.

### Soulbound Companion

A rare companion remembers the loops too.

### Challenge Timelines

Special seeds with unusual rules:

- No shops.
- All factions hostile.
- Dragon-dominated world.
- Undead plague world.
- Collapsing timeline from turn one.

---

## Minimum Viable Product Definition

The MVP should include:

1. Echoes mode selectable.
2. Same dungeon seed after death.
3. Loop counter.
4. Memory fragments after death.
5. At least item identification persistence.
6. Classless start.
7. At least one learned discipline.
8. At least one title.
9. At least one region name/theme modification.
10. Normal mode unaffected.

Do not wait for all advanced region/faction features before shipping the first playable branch.

---

## Shareable Branch Goal

The branch should be shareable when another developer can:

1. Clone/fork the repository.
2. Build the project using the documented Pathos content workflow.
3. Select Echoes mode.
4. Die and restart the same timeline.
5. See retained knowledge/soul progress.
6. Understand where future region/faction/title systems will be added.

---

## Final Product Feel

The final experience should feel like this:

> You are not a new adventurer each run. You are the same soul, trapped in a repeating dungeon, slowly learning every secret, mastering every discipline, earning titles that echo through reality, impressing or terrifying factions, and eventually breaking the loop through knowledge rather than brute force.

