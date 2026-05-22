# Node Types

## Overview
The world map is a procedurally generated node graph in the style of Slay the Spire. The player navigates nodes to progress toward each gym leader. The map visual transforms to reflect the current grief stage zone.

## Standard Node Types

### Combat Nodes
- **Wild Encounter** — standard Pokémon battle, can attempt to catch
- **Trainer Battle** — NPC trainer, no catching permitted
- **Elite Trainer** — harder trainer battle, better rewards
- **Gym Leader** — fixed story node, major battle
- **E4 Member** — fixed late game node, any of the four E4 members

### Recovery Nodes
- **Rest** — heal party
- **Settlement** — shop, heal, optional side dialogue with NPCs

### Story Nodes
- **Lore** — optional world building. Diary entries from drowned cities, historical records, pre-flood documents
- **NPC Encounter** — dialogue, relationship building with recurring characters
- **Evan Encounter** — rival appears, variable tone depending on story progress

### Reward Nodes
- **Item Find** — loot node, items including potential Rare Candy or EXP Share
- **Pokémon Rescue** — find a displaced Pokémon, choose to take it or leave it. Free, no battle.
- **Mystery** — unknown until arrival, could be any non-combat node type

### Evolution Nodes
- **Common** — friendship and miscellaneous evolutions
- **Uncommon** — item, trade, and Hisuian evolutions including regional form options
- **Rare** — early/illegal evolution of level-based Pokémon

Evolution nodes only spawn if a party member can use them.

## Map Structure
- Procedurally generated each run using a seeded graph algorithm
- Divided into layers between each gym
- Each layer has 1-3 nodes
- Connections between layers follow rules — no dead ends, enforced variety
- Player sees the path ahead but not everything
- Branching paths converge at gym leaders

## Visual Identity
The map transforms visually to reflect the current grief stage zone. Each zone has a distinct color palette, atmosphere, and emotional tone. See spawn_system.md for zone details.

## E4 Map
The E4 has its own separate node map, like each gym. E4 members are scattered within nodes — no defined order. Any two can appear in any run in any sequence before the interruption occurs.
