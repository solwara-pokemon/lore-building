# Tech Debt & Known Issues

Active list of known issues, deferred work, and architectural debt. Updated as items are resolved or discovered.

---

## Active Bugs

### Two-turn move invulnerability not enforced
**Location:** `BattleEngine.ts` → `resolveMove` accuracy check
**Cause:** `chargingMove.invisible` flag is set on `BattlePokemon` during the charge turn for moves like Fly, Dig, Dive, Bounce, Phantom Force, Shadow Force — but the engine doesn't check it when the opponent selects a target. Attacks that should miss an airborne/underground Pokémon currently hit normally.
**Fix:** In `resolveMove`, before the accuracy check, test if `defender.chargingMove?.invisible === true` and skip the move (show "avoided the attack" message) unless the attacking move has a specific bypass flag (e.g. Earthquake hits Dig, Gust hits Fly).
**Priority:** Medium

### `String Shot` declared in both GrassMoves and BugMoves
**Location:** `src/systems/battle/moves/moves/GrassMoves.ts` line ~40
**Cause:** Copy-paste error during family file authoring.
**Fix:** Remove the entry from `GrassMoves.ts`. String Shot is a Bug-type move.
**Priority:** Low

---

## Unimplemented Mechanics

### Held items do nothing in battle
All 15 held items can be equipped via the bag but `BattlePokemon.heldItem` is never read by the engine. None of the following effects are implemented:
- Shell Bell (heal 1/8 of damage dealt)
- Life Orb (1.3× damage, 10% recoil)
- Choice Band/Specs/Scarf (1.5× stat, lock to first move)
- Focus Sash (survive one KO hit at full HP)
- Rocky Helmet (deal 1/6 HP to contact attackers)
- Assault Vest (1.5× SpDef, block status moves)
- Leftovers / Black Sludge (end-of-turn heal/damage)
- Expert Belt (1.2× on super effective hits)
- Eviolite (1.5× Def/SpDef if not fully evolved)
- Wide Lens / Zoom Lens / Kings Rock (accuracy/flinch modifiers)

### Weather system
Rain Dance, Sunny Day, Sandstorm, Hail, Snowscape — all declared as `not` in their move families. No weather state on `BattleState`, no weather-dependent move power modifiers, no end-of-turn weather damage.

### Terrain system
Electric Terrain, Grassy Terrain, Misty Terrain, Psychic Terrain — declared `not`. No terrain state on `BattleState`.

### Entry hazards
Stealth Rock, Spikes, Toxic Spikes, Sticky Web — declared `not`. No hazard state on `BattleState`, no switch-in damage.

### Protect variants
Protect, Detect, Baneful Bunker, Spiky Shield, King's Shield, Obstruct, Silk Trap, Quick Guard, Wide Guard — all declared `not`. No protection state on `BattlePokemon`.

### OHKO moves
Horn Drill, Guillotine, Sheer Cold, Fissure — declared `not`. Accuracy formula and type immunity (Ice-types immune to Sheer Cold) not implemented.

### Switch-out effects
Volt Switch, U-turn, Flip Turn, Parting Shot, Baton Pass, Circle Throw, Dragon Tail — declared `partial`. Damage applies but the switch-out does not trigger.

### Delayed damage
Future Sight, Doom Desire — declared `not`. Requires a pending-damage queue on `BattleState`.

### Gym battles
Gym node shows a placeholder overlay. `GymScene` not started. Gym leaders fully designed in `story/gym_leaders.md`.

### Move Mastery system
Defined in `mechanics/move_mastery.md`. Not started. Requires meta shop first.

### Meta shop
Defined in `mechanics/meta_progression.md`. Not started. Requires Solwara Coins flow and a between-run shop scene.

### Evan's real sprite
Evan uses the Iris sprite as a placeholder. `public/images/trainers/iris.json` and `iris.png`.

### Rival team scaling across zones
Rival fights use `RivalTeamGen.ts` with predefined fight numbers. The scaling across all 8 zones has not been playtested.

---

## Architecture Debt

### `movesDb` still used for display
`BattleMenus.ts`, `RestScene.ts`, and `RescueScene.ts` still import `movesDb` from `PokemonDataLoader` for move name and PP display. These should eventually read from `MoveRegistry` directly so `moves.json` / `moves-index.json` can be deleted. Low urgency — the data files are small and accurate.

### `CatchCalc` hardcoded catch rate fallback
`CatchCalc.ts` uses `pokemonDb[target.speciesId]?.catchRate ?? 100`. The `?? 100` fallback is safe but masks missing data silently.

### `BattlePokemon.moves[i].moveName` is display only
Move names are stored as strings for display but the engine uses numeric `moveId` for logic. Low risk, but worth auditing if move data changes.

---

## Resolved (for reference)

- ✅ `ZONE_ORDER` duplicated in `RivalTeamGen` — fixed, now imports from `MapGenerator`
- ✅ `ms()` missing from `LoreScene` / `RestScene` fadeOut calls — fixed
- ✅ `caughtMon` typed as `any` in `BattleSceneData` — fixed, field added to interface
- ✅ `console.log` noise in `UIScene` — removed
- ✅ Node marked visited before completion — fixed, `currentNodeId` tracks in-progress, `nodesVisited` only written on return
- ✅ Exp share gave 0 when active mon at level cap — fixed via `calcExpBase`
- ✅ Exp bar animated for bench mons — fixed via `isActivePlayer` flag on `exp_gain` events
- ✅ RestScene custom party picker — replaced with `PartyPicker` `'pick'` mode
- ✅ Item node cards unresponsive — fixed, hit zone was offset due to container origin
- ✅ Ball throw animation froze after recoil — fixed, `RecoilAttr` now emits `hp_change` not custom `damage` event
- ✅ Rest wakes up early — fixed, sleep turns only decremented in `tickStatus` (end of turn), not `checkPreMove`
- ✅ Variable power moves did nothing — fixed via `HpRatioDamageAttr`, `WeightDamageAttr`, `SpeedRatioDamageAttr`, `TargetHpDamageAttr`, `StageScaledDamageAttr`
- ✅ Two-turn moves did nothing — fixed via `TwoTurnMoveAttr`, `chargingMove` state on `BattlePokemon`
- ✅ `MoveEffects.ts` legacy fallback — deleted, all 789 moves now route through `MoveRegistry`
- ✅ Status not showing on HUD after Rest/paralysis — fixed via `status_change` events emitted by engine at every status transition
