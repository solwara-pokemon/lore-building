# Move System

## Overview

Solwara uses a custom move system built on top of PokeAPI data. Moves are declared as class instances with typed attribute composition, matching Gen IX mechanics.

## Architecture

```
src/systems/battle/moves/
  core/
    Move.ts          — base classes: AttackMove, StatusMove, SelfStatusMove
    MoveAttr.ts      — abstract attribute base
    MoveContext.ts   — context passed to every attr.apply()
    MoveId.ts        — const object mapping move names to PokéAPI numeric IDs
    MoveRegistry.ts  — Map<MoveId, Move>, populated at boot
  attrs/
    StatusEffectAttr.ts   — applies a status condition at a chance %
    StatChangeAttr.ts     — raises/lowers stats on self or target
    RecoilAttr.ts         — recoil as % of damage dealt
    DrainAttr.ts          — heals attacker as % of damage dealt
    HealAttr.ts           — heals self as % of max HP
    FlinchAttr.ts         — causes flinch at chance %
    ConfuseAttr.ts        — inflicts confusion at chance %
    FixedDamageAttr.ts    — exact fixed damage (Dragon Rage = 40)
    LevelDamageAttr.ts    — damage = user's level (Night Shade, Seismic Toss)
    MultiHitAttr.ts       — 2-5 hits with Gen 5+ weight distribution
  moves/
    NormalMoves.ts / NormalMoves2.ts
    FireMoves.ts
    ElectricMoves.ts
    WaterMoves.ts
    GrassMoves.ts
    IceMoves.ts
    FightingMoves.ts
    PoisonMoves.ts
    GroundMoves.ts
    FlyingMoves.ts
    PsychicMoves.ts
    BugMoves.ts
    RockMoves.ts
    GhostMoves.ts
    DragonMoves.ts
    DarkMoves.ts
    SteelMoves.ts
    FairyMoves.ts
```

## Declaring a Move

```ts
// Simple damaging move with secondary effect
export const Thunderbolt = new AttackMove(MoveId.THUNDERBOLT, 'electric', 'special', 90, 100, 15, 10)
  .attr(StatusEffectAttr, 'paralysis', 10)
  .implemented('full');

// Status move
export const ThunderWave = new StatusMove(MoveId.THUNDER_WAVE, 'electric', 90, 20)
  .attr(StatusEffectAttr, 'paralysis', 100)
  .implemented('full');

// Self-targeting stat boost
export const DragonDance = new SelfStatusMove(MoveId.DRAGON_DANCE, 'dragon', 20)
  .attr(StatChangeAttr, 'atk', 1, true)
  .attr(StatChangeAttr, 'spe', 1, true)
  .implemented('full');

// Multi-hit
export const BulletSeed = new AttackMove(MoveId.BULLET_SEED, 'grass', 'physical', 25, 100, 30, -1)
  .attr(MultiHitAttr, 2, 5)
  .implemented('full');
```

## AttackMove Constructor

```ts
new AttackMove(id, type, category, power, accuracy, pp, chance, priority?, target?)
```

| Param | Type | Notes |
|---|---|---|
| `id` | `MoveId` | PokéAPI numeric ID |
| `type` | `Type` | e.g. `'fire'`, `'electric'` |
| `category` | `'physical' \| 'special'` | |
| `power` | `number \| null` | null for variable-power moves |
| `accuracy` | `number \| null` | null = never misses |
| `pp` | `number` | Max PP |
| `chance` | `number` | Secondary effect chance (pass `-1` if unused) |
| `priority` | `number` | Default 0 |
| `target` | `MoveTarget` | Default `'opponent'` |

## Implementation Status

Every move is tagged with `.implemented()`:

| Status | Meaning |
|---|---|
| `'full'` | Move works correctly including all secondary effects |
| `'partial'` | Damage applies but a secondary mechanic is missing (e.g. two-turn charge, switch-out after, HP-scaled power) |
| `'not'` | Move declaration exists but the engine ignores all effects |

The engine checks `move.implStatus === 'not'` and skips attrs entirely — damage still applies for attack moves, but no secondary effects run.

## Engine Integration

`BattleEngine.resolveMove()` checks `MoveRegistry.getMove(slot.moveId)` first. If a registered move is found:

1. **Priority** — read from `move.priority` (not `movesDb`)
2. **Fixed damage** — checks for `FixedDamageAttr` before calling `calcDamage`
3. **Level damage** — checks for `LevelDamageAttr` before calling `calcDamage`
4. **Multi-hit** — checks for `MultiHitAttr`, loops damage N times
5. **Effects** — calls `move.applyAttrs(ctx)` with damage result
6. **Status updates** — emits `status_change` event so HUD refreshes

If no registered move is found, falls back to the legacy `MoveEffects.ts` dispatch table.

## Adding a New Move

1. Find the PokéAPI ID (check `MoveId.ts` — all 789 learnset moves are already listed)
2. Add the declaration to the appropriate type family file
3. Chain `.attr()` calls for secondary effects
4. Set `.implemented()` status honestly
5. The move is automatically registered at boot via `registerAll()`

## What Is Not Yet Implemented

- **Variable power** — Gyro Ball, Heavy Slam, Flail, Reversal, Stored Power, Wring Out, etc. (need new attr classes)
- **Two-turn moves** — Fly, Dig, Bounce, Solar Beam, Skull Bash etc. (`partial`)
- **Weather/terrain** — Rain Dance, Sunny Day, Electric Terrain etc. (`not`)
- **Entry hazards** — Stealth Rock, Spikes, Toxic Spikes, Sticky Web (`not`)
- **Protect variants** — Protect, Detect, Baneful Bunker etc. (`not`)
- **Switch effects** — Volt Switch, U-turn, Parting Shot, Baton Pass (`partial`)
- **OHKO moves** — Horn Drill, Guillotine, Sheer Cold, Fissure (`not`)
- **Delayed damage** — Future Sight, Doom Desire (`not`)

## MoveId

`MoveId` is a `const` object mapping screaming-snake-case names to PokéAPI numeric IDs:

```ts
export const MoveId = {
  POUND: 1,
  KARATE_CHOP: 2,
  FLAMETHROWER: 53,
  THUNDERBOLT: 85,
  // ... 789 total
} as const;

export type MoveId = typeof MoveId[keyof typeof MoveId];
```

`MOVE_NAME_TO_ID` maps kebab-case strings (as stored in learnset data) to `MoveId` values.
