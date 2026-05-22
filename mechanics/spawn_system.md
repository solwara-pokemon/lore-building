# Spawn System

## Philosophy
All Pokémon are available in Solwara. Every type must be represented in at least two grief stage zones. No Pokémon is locked to a single zone — everything can appear anywhere, just with different weights.

## Zone Weight System
Each Pokémon has a spawn weight per grief zone. Higher weight = more common in that zone.

- **Primary zones** — appears frequently, feels thematically at home
- **Secondary zones** — appears occasionally, makes thematic sense
- **Tertiary zones** — very rare, surprising to find here
- **Absent** — does not appear (should be rare — most Pokémon have at least two zones)

Spawn weights are stored in the master Pokémon JSON file and can be tuned without touching code.

## Zone Thematic Flavor
Each grief zone has a visual identity and emotional tone that informs which Pokémon feel at home there:

| Zone | Tone | Natural Pokémon Flavor |
|---|---|---|
| Denial | Aggressively normal, warm | Normal types, familiar, non-threatening |
| Pain & Guilt | Muted, heavy, somber | Ghost, Poison, lingering types |
| Anger & Bargaining | Red, stormy, jagged | Fighting, Fire, aggressive types |
| Depression & Loneliness | Cold, vast, empty | Ice, alone-natured, slow types |
| Upward Turn | Warming, sparking | Electric, emerging types |
| Reconstruction | Ordered, geometric | Rock, Steel, structured types |
| Acceptance | Bioluminescent, gorgeous | Water, diverse, at peace |
| Hope | Open sky, light | Flying, bright, soaring types |

## Rescue Node Spawns
See catch_system.md for full rescue node rules.
Rescue nodes draw from a global pool weighted by BST — not zone specific.

## JSON Structure
\`\`\`json
"spawnData": {
  "nodeTypes": ["wild", "rescue"],
  "zoneWeights": {
    "denial": 2,
    "pain": 15,
    "anger": 3,
    "depression": 12,
    "upward": 2,
    "reconstruction": 1,
    "acceptance": 4,
    "hope": 2
  }
}
\`\`\`
