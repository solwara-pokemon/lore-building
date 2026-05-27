# Map System

## Overview

Solwara uses a Slay the Spire-inspired procedural node map. Each zone (grief stage) has its own seeded map generated at run start. The map is the same every time you enter a zone with the same seed, so reloading mid-run returns you to exactly where you were.

## Grid Structure

**7 columns × 15 floors + 1 gym node (floor 14)**

- Floors are 0-indexed (0–14)
- Floor 14 is always the gym — a single convergence node
- Floors 0–13 are content floors

```
Floor 14:  [           GYM           ]
Floor 13:  [rest] [SETTLEMENT] [rest] [rest] [rest]
  ...
Floor  1:  [─] [─] [─] [─] [─] [─]   (6 branching paths)
Floor  0:  [         WILD            ]
```

## Path Generation (StS-style)

1. Floor 0 is always a **single wild node** centred on the map — all paths originate from it
2. **6 independent seeded paths** walk upward from floor 1 to floor 13
3. Each path steps to one of the 3 nearest columns on the next floor
4. **No-crossing invariant** — paths maintain left-to-right ordering (path N never exceeds path N+1's column)
5. Any node on floors 1–13 unreachable by any path is **pruned**
6. All surviving floor-13 nodes connect to the single gym node

## Floor Guarantees

| Floor | Rule |
|---|---|
| 0 | Single `wild` node, centred horizontally |
| 1–4 | `elite_trainer` not eligible |
| 5+ | `elite_trainer` allowed |
| 8 | One guaranteed `item` node at a seeded column; rest random |
| 7, 9, 10, 11, 12 | Rival floor — one picked per zone (seeded). All nodes on that floor become `rival` |
| 13 | One `settlement` node at a seeded column; all others are `rest`. All connect to gym |
| 14 | Single `gym` node |

## Quality Rules

- **Elite/rest separation** — `elite_trainer` and `rest` cannot be adjacent on any path
- **Branching variety** — a node with 2+ outgoing connections cannot send both to nodes of the same type

## Rival Placement

The rival floor is chosen from `[7, 9, 10, 11, 12]` (0-indexed) at run start using the zone seed. Floor 8 (guaranteed item) is excluded from rival eligibility. Because the entire rival floor is locked to type `rival`, any path the player takes will encounter the rival exactly once per zone.

## Visual Layout

**Isometric stagger** — odd-numbered floors are offset horizontally by `COL_W / 2` (60px), creating a triangular grid. Connection lines are drawn straight (no midpoint kink), letting the stagger provide natural visual separation.

### Layout Constants
| Constant | Value | Notes |
|---|---|---|
| `COLS` | 7 | Columns per floor |
| `COL_W` | 120px | Horizontal spacing |
| `LAYER_H` | 72px | Vertical spacing between floors |
| `MAP_PAD_TOP` | 48px | Padding above floor 0 |
| `CANVAS_W` | 1280px | |

Floor 0 and floor 14 (single nodes) are always rendered at `CANVAS_W / 2 = 640px`.

## Camera / Navigation

- Map starts **centred on the first available node** at zoom **1.6×**
- **Pan** — drag anywhere (pointer down + move)
- **Pinch zoom** — two-finger pinch (`activePointers: 2` in Phaser game config)
- **Scroll wheel** — zoom in/out on desktop
- Min zoom `0.3×`, max `2.0×`

## Node Selection

- Available nodes show a pulsing glow ring
- Tap a node to enter it
- Once a node is entered, it is locked as **in-progress** (`currentNodeId` in save)
- **Node is only marked visited on return** — reloading mid-node sends the player back to it
- Only one node per floor layer can be visited per zone run

## Implementation Files

| File | Responsibility |
|---|---|
| `src/systems/map/MapGenerator.ts` | Pure data — generates the node graph |
| `src/scenes/map/NodeMapScene.ts` | Thin orchestrator — init, routing, state |
| `src/scenes/map/NodeMapRenderer.ts` | All drawing — nodes, lines, grid, legend |
| `src/scenes/map/NodeMapInput.ts` | Pan, zoom, pinch |
| `src/scenes/map/NodeMapGlobals.ts` | Shared module state |
