# Move System

## Overview

Solwara uses a custom move system with all 789 learnset moves declared as typed class instances with attribute composition. Mechanics follow Gen IX rules. The legacy `MoveEffects.ts` dispatch table has been deleted — all moves route through `MoveRegistry`.

## Architecture

```
src/systems/battle/moves/
  core/
    Move.ts          — base classes: AttackMove, StatusMove, SelfStatusMove
    MoveAttr.ts      — abstract attribute base
    MoveContext.ts   — context passed to every attr.apply()
    MoveId.ts        — const object mapping 789 move names to PokéAPI numeric IDs
    MoveRegistry.ts  — Map<MoveId, Move>, populated at boot via registerAll()
  attrs/
    StatusEffectAttr.ts      — status condition at chance %
    StatChangeAttr.ts        — stat raise/lower on self or target (incl. accuracy/evasion)
    RecoilAttr.ts            — recoil as % of damage dealt
    DrainAttr.ts             — heal attacker as % of damage dealt
    HealAttr.ts              — heal self as % of max HP
    FlinchAttr.ts            — flinch at chance %
    ConfuseAttr.ts           — confusion at chance %
    FixedDamageAttr.ts       — exact fixed damage (Dragon Rage = 40)
    LevelDamageAttr.ts       — damage = user's level (Night Shade, Seismic Toss)
    MultiHitAttr.ts          — 2-5 hits with Gen 5+ weight distribution
    HpRatioDamageAttr.ts     — power scales with user HP (Flail, Water Spout)
    WeightDamageAttr.ts      — power scales with weight (Low Kick, Heavy Slam)
    SpeedRatioDamageAttr.ts  — power scales with speed (Gyro Ball, Electro Ball)
    TargetHpDamageAttr.ts    — power scales with target HP (Wring Out, Crush Grip)
    StageScaledDamageAttr.ts — power scales with positive stat stages (Stored Power)
    TwoTurnMoveAttr.ts       — charge turn + release (Fly, Dig, Solar Beam, etc.)
  moves/
    NormalMoves.ts / NormalMoves2.ts
    FireMoves.ts / ElectricMoves.ts / WaterMoves.ts / GrassMoves.ts
    IceMoves.ts / FightingMoves.ts / PoisonMoves.ts / GroundMoves.ts
    FlyingMoves.ts / PsychicMoves.ts / BugMoves.ts / RockMoves.ts
    GhostMoves.ts / DragonMoves.ts / DarkMoves.ts / SteelMoves.ts / FairyMoves.ts
```

## Declaring a Move

```ts
// Damaging move with burn chance
new AttackMove(MoveId.FLAMETHROWER, 'fire', 'special', 90, 100, 15, 10)
  .attr(StatusEffectAttr, 'burn', 10)
  .implemented('full');

// Variable power (Gyro Ball)
new AttackMove(MoveId.GYRO_BALL, 'steel', 'physical', null, 100, 5, -1)
  .attr(SpeedRatioDamageAttr, 'gyro')
  .implemented('full');

// Two-turn move (Fly)
new AttackMove(MoveId.FLY, 'flying', 'physical', 90, 95, 15, -1)
  .attr(TwoTurnMoveAttr, '{name} flew up high!', true)  // true = invisible
  .implemented('full');

// Stat boost
new SelfStatusMove(MoveId.DRAGON_DANCE, 'dragon', 20)
  .attr(StatChangeAttr, 'atk', 1, true)
  .attr(StatChangeAttr, 'spe', 1, true)
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
| `power` | `number \| null` | null for variable-power or status moves |
| `accuracy` | `number \| null` | null = never misses |
| `pp` | `number` | Max PP |
| `chance` | `number` | Secondary effect chance; pass `-1` if unused |
| `priority` | `number` | Default 0 |
| `target` | `MoveTarget` | Default `'opponent'` |

## Implementation Status

Every move is tagged with `.implemented()`:

| Status | Meaning |
|---|---|
| `'full'` | Works correctly including all secondary effects |
| `'partial'` | Damage applies but a mechanic is missing (switch-out, HP-lock, etc.) |
| `'not'` | Declaration exists; engine shows "(not implemented)" message |

## Engine Integration

`BattleEngine.resolveMove()` checks `MoveRegistry.getMove(slot.moveId)` first:

1. **Two-turn charge** — if `TwoTurnMoveAttr` present and not already charged, sets `attacker.chargingMove`, emits charge message, returns. On the next turn the charge is detected and skipped to full resolution
2. **Priority** — read from `move.priority`
3. **Variable power** — checks `FixedDamageAttr`, `LevelDamageAttr`, `HpRatioDamageAttr`, `WeightDamageAttr`, `SpeedRatioDamageAttr`, `TargetHpDamageAttr`, `StageScaledDamageAttr` before calling `calcDamage`
4. **Multi-hit** — `MultiHitAttr` rolls hit count, loops damage N times
5. **Effects** — `move.applyAttrs(ctx)` runs all attrs with damage result
6. **Status events** — `status_change` emitted so HUD refreshes immediately

If no registered move is found (should not happen with 789/789 coverage), shows "(not implemented)" and returns.

## Variable Power Attrs

| Attr | Mode | Formula |
|---|---|---|
| `HpRatioDamageAttr('low')` | Flail, Reversal | Gen IX table: 20→200 as HP% drops |
| `HpRatioDamageAttr('high')` | Water Spout, Eruption, Dragon Energy | `floor(150 × hp/maxHp)`, min 1 |
| `WeightDamageAttr('target')` | Low Kick, Grass Knot | Target kg → 20/40/60/80/100/120 |
| `WeightDamageAttr('ratio')` | Heavy Slam, Heat Crash | User/target ratio → 40/60/80/100/120 |
| `SpeedRatioDamageAttr('gyro')` | Gyro Ball | `floor(25 × target_spe / user_spe)`, max 150 |
| `SpeedRatioDamageAttr('electro')` | Electro Ball | Speed ratio buckets → 40/60/80/120/150 |
| `TargetHpDamageAttr` | Wring Out, Crush Grip | `floor(120 × target_hp/maxHp)`, min 1 |
| `StageScaledDamageAttr` | Stored Power, Power Trip | `base + base × positive_stages` |

## Two-Turn Moves

`TwoTurnMoveAttr(chargeMessage, invisible?)`:
- **Charge turn**: sets `attacker.chargingMove = { moveIndex, invisible }`, emits `chargeMessage`, consumes PP, returns early (no damage)
- **Release turn**: engine detects `attacker.chargingMove.moveIndex === moveIndex`, clears it, falls through to full damage resolution
- **Invisible flag**: `true` for Fly, Dig, Dive, Bounce, Phantom Force, Shadow Force — sets `invisible: true` on `chargingMove`. Note: accuracy bypass for invisible targets is not yet implemented in the engine
- **AI**: `chooseMove` is bypassed when `opponentActive.chargingMove` is set; AI always uses the charged move index
- **Cleared**: on faint (`handleFaint`) and on switch (`resolveTurnToEvents` switch block)

## Two-Turn Move List

| Move | Charge message | Invisible |
|---|---|---|
| Fly | "flew up high!" | ✅ |
| Dig | "burrowed underground!" | ✅ |
| Dive | "hid underwater!" | ✅ |
| Bounce | "sprang up!" | ✅ |
| Phantom Force | "vanished instantly!" | ✅ |
| Shadow Force | "vanished instantly!" | ✅ |
| Sky Attack | "is glowing!" | ❌ |
| Sky Drop | "took the target into the sky!" | ✅ |
| Solar Beam | "absorbed light!" | ❌ |
| Solar Blade | "absorbed light!" | ❌ |
| Electro Shot | "absorbed electricity!" | ❌ |
| Freeze Shock | "became cloaked in a freezing light!" | ❌ |
| Ice Burn | "became cloaked in a freezing light!" | ❌ |
| Skull Bash | "tucked in its head!" | ❌ |
| Razor Wind | "whipped up a whirlwind!" | ❌ |
| Geomancy | "is absorbing power!" | ❌ |

## What Is Not Yet Implemented

- **Weather/terrain** — Rain Dance, Sunny Day, Electric Terrain etc. (`not`)
- **Entry hazards** — Stealth Rock, Spikes, Toxic Spikes, Sticky Web (`not`)
- **Protect variants** — Protect, Detect, Baneful Bunker etc. (`not`)
- **Switch effects** — Volt Switch, U-turn, Parting Shot, Baton Pass (`partial` — damage applies, switch doesn't)
- **OHKO moves** — Horn Drill, Guillotine, Sheer Cold, Fissure (`not`)
- **Delayed damage** — Future Sight, Doom Desire (`not`)
- **Invulnerability** — `chargingMove.invisible` flag exists but accuracy bypass not yet checked
- **Specific move quirks** — Body Slam paralysis (treated as stat drop proxy), Facade double power when statused, Wake-Up Slap double vs sleeping, etc.

## Adding a New Move

1. Find the move's `MoveId` entry (all 789 already listed)
2. Add the declaration to the appropriate type family file
3. Chain `.attr()` calls for secondary effects
4. Set `.implemented()` status honestly
5. Automatically registered at boot via `registerAll()` — no other changes needed
