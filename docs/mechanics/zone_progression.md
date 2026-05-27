# Zone Progression

## Overview

Solwara is divided into 8 zones, each representing a stage of grief. Zones are traversed sequentially. Each zone has its own seeded procedural map, visual theme, spawn pool, and level cap tied to the zone's gym leader.

## Zone Order

| Index | Zone Key | Zone Name | Level Cap | Gym Leader |
|---|---|---|---|---|
| 0 | `denial` | Shock & Denial | 20 | Jon (Normal) |
| 1 | `pain` | Pain & Guilt | 30 | Reese (Poison) |
| 2 | `anger` | Anger & Bargaining | 35 | Mari (Fighting) |
| 3 | `depression` | Depression & Loneliness | 40 | Lupa (Ice) |
| 4 | `upward` | The Upward Turn | 45 | Bobby (Electric) |
| 5 | `reconstruction` | Reconstruction | 50 | Bram (Steel/Rock) |
| 6 | `acceptance` | Acceptance | 55 | Lena (Water) |
| 7 | `hope` | Hope & New Meaning | 60 | Sol (Flying) |

Post-game level caps: E4 = 80, Evan = 100.

## Visual Themes

Each zone has a distinct visual identity applied to the map, node icons, and battle backgrounds.

| Zone | Background | Accent | Mood |
|---|---|---|---|
| denial | Warm amber/brown | Gold | Familiar, almost normal — wrong if you look too closely |
| pain | Deep purple | Violet glow | Heavy, claustrophobic, suffocating |
| anger | Deep red/crimson | Flame orange | Chaotic, hot, aggressive |
| depression | Dark navy | Steel blue | Sparse, cold, vast emptiness |
| upward | Dark forest green | Lime | First hints of life returning |
| reconstruction | Charcoal grey | Slate | Industrial, purposeful, unfinished |
| acceptance | Deep teal | Bioluminescent cyan | Oceanic, still, beautiful |
| hope | Deep midnight blue | Soft gold/white | Open sky, expansive, forward-looking |

## Node Spawn Weights

Each zone biases toward certain node types. Weights below are relative (higher = more common):

| Type | denial | pain | anger | depression | upward | reconstruction | acceptance | hope |
|---|---|---|---|---|---|---|---|---|
| wild | 8 | 6 | 8 | 5 | 6 | 6 | 5 | 6 |
| trainer | 6 | 5 | 8 | 5 | 5 | 6 | 4 | 5 |
| elite | 2 | 2 | 4 | 2 | 3 | 3 | 2 | 3 |
| rest | 3 | 4 | 2 | 5 | 4 | 3 | 5 | 4 |
| lore | 3 | 4 | 2 | 5 | 3 | 3 | 4 | 3 |
| npc | 2 | 3 | 2 | 3 | 3 | 2 | 4 | 3 |
| item | 3 | 2 | 3 | 2 | 4 | 4 | 3 | 3 |
| rescue | 2 | 3 | 1 | 3 | 3 | 2 | 4 | 3 |
| mystery | 2 | 3 | 2 | 3 | 2 | 2 | 2 | 3 |
| evolution | 1 | 1 | 1 | 1 | 1 | 2 | 2 | 2 |

Rival, settlement, rest (floor 13), item (floor 8), wild (floor 0), and gym (floor 14) are placed by floor guarantee rules, not weights.

## Level Cap System

- Pokémon **cannot gain EXP** beyond the zone's level cap
- Bench Pokémon receiving EXP Share **also respect the cap**
- Rare Candy bypasses the current zone's cap (absolute ceiling: level 100)
- Level is shown **bold** on the party screen; turns **soft red** (`#d06060`) when at cap
- Battle HUD level text also turns red when at cap

## The Hope Badge

Sol (Gym 8) is never completed in a normal run — the Unforgotten Party interrupts during Gym 8. The Hope Badge remains greyed out in the badge case throughout the climax, and is only earned by completing the game with all three launch roles (Hero, The Doctor, The Accepted).

## Implementation

```ts
// Zone order constant — single source of truth
export const ZONE_ORDER: ZoneKey[] = [
  'denial', 'pain', 'anger', 'depression',
  'upward', 'reconstruction', 'acceptance', 'hope',
];

// Level caps
export const ZONE_LEVEL_CAPS: Record<ZoneKey | 'e4', number> = {
  denial: 12, pain: 20, anger: 28, depression: 36,
  upward: 42, reconstruction: 48, acceptance: 54, hope: 60, e4: 80,
};
```
