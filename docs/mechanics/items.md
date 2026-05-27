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

---

## Held Items

Held items can be equipped to a Pokémon from the bag. They currently **have no effect in battle** — this is a known tech debt item. All held items are in the loot pool and can be found, equipped, and displayed, but their passive effects are not implemented.

| Item | Intended Effect | Tier | Min Floor |
|---|---|---|---|
| Wide Lens | +10% move accuracy | 1 | 1 |
| Shell Bell | Heal 1/8 of damage dealt | 1 | 2 |
| Kings Rock | +10% flinch chance | 1 | 2 |
| Expert Belt | 1.2× damage on super effective hits | 2 | 2 |
| Zoom Lens | +20% accuracy if moving second | 2 | 3 |
| Eviolite | 1.5× Def and SpDef if not fully evolved | 2 | 3 |
| Leftovers | Restore 1/16 max HP per turn | 2 | 3 |
| Focus Sash | Survive one KO hit at full HP | 3 | 4 |
| Rocky Helmet | Deal 1/6 HP to contact attackers | 3 | 4 |
| Assault Vest | 1.5× SpDef; blocks status moves | 3 | 4 |
| Black Sludge | Poison types: +1/16 HP/turn; others: -1/8 HP/turn | 2 | 3 |
| Choice Band | 1.5× Attack; locked to first move | 3 | 4 |
| Choice Specs | 1.5× Sp. Attack; locked to first move | 3 | 4 |
| Choice Scarf | 1.5× Speed; locked to first move | 3 | 4 |
| Life Orb | 1.3× damage; 10% recoil on use | 3 | 4 |

---

## Loot Generation

`generateLootChoices(floor, rng, count, luck, party)` builds the item pool:

1. Filter by `item.minFloor <= floor`
2. Filter conditionals — `full-heal` is excluded if no party member has a status condition
3. Roll tier for each candidate (base tier + upgrade chance cascade)
4. Weight candidates by tier, return `count` choices

The `party` parameter (array of `RunPokemon`) is passed from both `ItemScene` and `RewardHandler` so conditional filtering is always team-aware.

---

## Removed Items

The following items were removed from the loot pool as they are made redundant by Full Heal:

- ~~Antidote~~ (cured poison)
- ~~Awakening~~ (cured sleep)
- ~~Burn Heal~~ (cured burn)
