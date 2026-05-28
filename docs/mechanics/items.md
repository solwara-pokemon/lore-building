# Items

## Overview

Items are found at Item nodes, Settlement shops, and as battle rewards. They are divided into consumables (single-use, used from the bag) and held items (equipped to a Pokémon, passive effect in battle).

## Item Tiers

| Tier | Name | Weight |
|---|---|---|
| 1 | Common | High |
| 2 | Uncommon | Medium |
| 3 | Rare | Low |
| 4 | Ultra Rare | Very low |

Items can upgrade tier at generation time. Each item has a base tier and a per-item upgrade chance. Upgrades cascade — a Common can upgrade to Uncommon, then potentially to Rare in a single roll.

## Consumables

All consumables are fully implemented unless noted.

### HP Restoration
| Item | Effect | Tier | Min Floor |
|---|---|---|---|
| Potion | Restore 20 HP | 1 | 1 |
| Super Potion | Restore 60 HP | 2 | 2 |
| Hyper Potion | Restore 120 HP | 3 | 3 |
| Max Potion | Restore full HP | 3 | 4 |

### Status Cure
| Item | Effect | Tier | Min Floor | Notes |
|---|---|---|---|---|
| Full Heal | Cure any status | 2 | 2 | Only appears if a party member has a non-volatile status |

### PP Restoration
| Item | Effect | Tier | Min Floor |
|---|---|---|---|
| Ether | Restore 10 PP to one move | 2 | 2 |
| Elixir | Restore 10 PP to all moves | 3 | 3 |

### Revival
| Item | Effect | Tier | Min Floor |
|---|---|---|---|
| Revive | Revive fainted mon to 50% HP | 3 | 3 |
| Max Revive | Revive fainted mon to full HP | 4 | 5 |

### EXP
| Item | Effect | Tier | Min Floor | Notes |
|---|---|---|---|---|
| Rare Candy | Gain 1 level (bypasses zone cap) | 3 | 1 | |
| EXP Charm | Stackable, +50% EXP per charm | — | — | First given free at run start; up to 5 total from meta shop |

### EXP All
The Exp All item grants EXP share to the entire party. Up to 4 can be found in a run. Combined with 5 EXP Charms, bench Pokémon can receive 100%+ of the battler's EXP.

### Poké Balls
| Item | Effect | Tier | Min Floor |
|---|---|---|---|
| Poké Ball ×5 | Adds 5 Poké Balls to bag | 1 | 1 |
| Great Ball ×5 | Adds 5 Great Balls to bag | 2 | 1 |
| Ultra Ball ×5 | Adds 5 Ultra Balls to bag | 3 | 2 |

Ball items write directly to `slot.balls` (not inventory) via the `ball:<key>:<count>` effect handler in `RewardHandler`.

---

## Held Items

All held items are fully implemented in battle via `HeldItemEffects.ts`.

| Item | Effect | Tier | Min Floor |
|---|---|---|---|
| Wide Lens | ×1.1 move accuracy | 1 | 1 |
| Shell Bell | Heal 1/8 of damage dealt | 1 | 2 |
| Kings Rock | +10% flinch chance on all damaging moves | 1 | 2 |
| Expert Belt | ×1.2 damage on super effective hits | 2 | 2 |
| Zoom Lens | ×1.2 accuracy when moving second | 2 | 3 |
| Eviolite | ×1.5 Def and SpDef if not fully evolved | 2 | 3 |
| Leftovers | Restore 1/16 max HP per turn | 2 | 3 |
| Black Sludge | Poison types: +1/16 HP/turn; others: −1/8 HP/turn | 2 | 3 |
| Focus Sash | Survive one KO hit at full HP; consumed on trigger | 3 | 4 |
| Rocky Helmet | Deal 1/6 HP to contact attackers | 3 | 4 |
| Assault Vest | Special damage ÷1.5; blocks status moves | 3 | 4 |
| Choice Band | ×1.5 Attack (physical); locks first move used | 3 | 4 |
| Choice Specs | ×1.5 Sp. Attack (special); locks first move used | 3 | 4 |
| Choice Scarf | ×1.5 Speed; locks first move used | 3 | 4 |
| Life Orb | ×1.3 damage; 10% max HP recoil per hit | 3 | 4 |

### Choice Item Locking
When a Pokémon holds a Choice item and uses a move, `lockedMoveIndex` is set on `BattlePokemon`. All other moves are greyed out in the move menu. The lock clears on switch-out. The engine blocks non-locked moves with a "can't use that move!" message.

### Zoom Lens Condition
Zoom Lens only applies its accuracy bonus when the bearer moves after the opponent. The `movingSecond` flag is derived from turn order in `resolveTurnToEvents` and threaded into `resolveMove` → `accuracyCheck`.

---

## Loot Generation

`generateLootChoices(floor, rng, count, luck, party)` builds the item pool:

1. Filter by `item.minFloor <= floor`
2. Filter conditionals — `full-heal` is excluded if no party member has a status condition
3. Roll tier for each candidate (base tier + upgrade chance cascade)
4. Weight candidates by tier, return `count` choices

The `party` parameter (array of `RunPokemon`) is passed from both `ItemScene` and `RewardHandler` so conditional filtering is always team-aware.

---

## Bag UI

The BagViewer component (`src/ui/components/BagViewer.ts`) provides a scrollable overlay:
- One item per row (56px height)
- Sorted by tier descending (rarest first)
- Left accent bar colour-coded by tier
- Shows name, description, count, and HELD/USE badge
- Finger drag scrolling with geometry mask clipping
- Mouse wheel support
- Tapping a row opens PartyPicker in equip-item or use-item mode as appropriate

---

## Removed Items

The following items were removed from the loot pool as they are made redundant by Full Heal:

- ~~Antidote~~ (cured poison)
- ~~Awakening~~ (cured sleep)
- ~~Burn Heal~~ (cured burn)
