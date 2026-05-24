# Battle System

## Format
- Turn-based, singles only
- 4 moves per Pokémon
- Standard PP system
- Standard type effectiveness chart
- Standard status conditions (burn, freeze, paralysis, poison, sleep, confusion)
- Items usable in battle
- Switch on your turn
- No Mega Evolutions, no Gigantamax

## Experience System
- Only the battling Pokémon gains full EXP
- EXP Share: 20% of battler EXP given flat to every non-participating party member
- First EXP Share given free after beating Gym 1 (Jon)
- Up to 4 more findable via loot and events
- Maximum 5 EXP Shares = every non-battler gets 100% of battler EXP flat

## Level Cap System
Hard caps per gym. Cannot gain EXP beyond cap regardless of battles.
Rare Candy bypasses the current gym cap. Absolute ceiling is level 100, no exceptions.

| Gym | Leader | Hard Cap |
|---|---|---|
| 1 | Jon | 20 |
| 2 | Reese | 30 |
| 3 | Mari | 35 |
| 4 | Lupa | 40 |
| 5 | Bobby | 45 |
| 6 | Bram | 50 |
| 7 | Lena | 55 |
| 8 | Sol | 60 |
| E4 | — | 80 |
| Evan | — | 100 |

## Input
- Keyboard supported on desktop
- Touch supported on mobile (touch = mouse click where applicable)
- Designed for both from day one

---

## Implementation Architecture

### Event Queue System
The battle runs on a typed event stream. `resolveTurnToEvents()` in `BattleEngine.ts` resolves the entire turn as pure logic — no Phaser, no timing — and returns a flat `BattleEvent[]`. `BattleScene` then plays these events sequentially via `playEventQueue()`.

Each `BattleEvent` is one atomic thing:

| Event | What it does |
|---|---|
| `move_use` | Typewriter text + move animation in parallel. Grabs any trailing `hp_change` events and fires bars/numbers when both text and anim finish. Waits for tap. |
| `hp_change` | Tweens the HP bar smoothly, spawns a damage/heal number. Fires instantly, no tap needed. |
| `message` | Typewriter text, wait for tap. |
| `faint` | Drop animation, then continues. |
| `battle_end` | Triggers end screen. |

The engine mutates `BattleState` HP values as it runs, so every `hp_change` event already carries the correct `newHp` — no reconciliation or snapshot diffing.

### Move Animations
Move animations mirror PokeRogue's `battle-anims.ts` system exactly:

- **`target=0` (USER)** — clone of the user's pokemon sprite, positioned by focus math
- **`target=1` (TARGET)** — clone of the target's pokemon sprite (claw slashes, impact reactions)
- **`target=2` (GRAPHIC)** — sprite from the move's graphic spritesheet

Animations are sourced from `pagefaultgames/pokerogue-assets` battle-anims JSONs and sprite sheets, served from the LXC filesystem. The `AnimFrameTarget` enum is `USER=0, TARGET=1, GRAPHIC=2`.

Self-targeting moves (target=`"user"` in PokeAPI) pass `mover === moveTarget` so all focus math lands on the correct pokemon. Damaging moves blink the target; status moves do not.

### Text Display
All text goes through `typewriterText(text, onDone)`. First tap skips to full text; second tap calls `onDone`. There is no separate message queue — `showMsg` is gone. Everything is a `message` event in the queue.
