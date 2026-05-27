# Tech Debt & Known Issues

Active list of known issues, deferred work, and architectural debt. Updated as items are resolved or discovered.

---

## Active Bugs

### Rest wakes up early
**Location:** `BattleEngine.ts` → `checkPreMove` + `endOfTurn` → `tickStatus`
**Cause:** Sleep turns are decremented twice per turn — once in `checkPreMove` (before the move) and once in `tickStatus` (end of turn). Gen IX specifies sleep is decremented once per turn.
**Fix:** Remove the decrement from `checkPreMove`; let `tickStatus` handle it exclusively.
**Priority:** Medium

---

## Move System Debt

### Variable power moves not implemented
Moves with `power: null` that scale with game state need new attr classes:

| Attr class needed | Moves |
|---|---|
| `WeightDamageAttr` | Low Kick, Heavy Slam |
| `SpeedRatioDamageAttr` | Gyro Ball, Electro Ball |
| `HpRatioDamageAttr` | Flail, Reversal, Dragon Energy, Water Spout |
| `TargetHpDamageAttr` | Wring Out, Crush Grip |
| `StageScaledDamageAttr` | Stored Power, Power Trip |

All currently marked `partial` — damage does not apply because `power: null` with no special attr results in the move being skipped by the engine.

### Two-turn moves not implemented
Fly, Dig, Bounce, Solar Beam, Skull Bash, Sky Attack, Shadow Force, Phantom Force, Freeze Shock, Ice Burn, Geomancy, Razor Wind.
**Needs:** `chargingMove` field on `BattlePokemon`, pre-move check in engine that completes the charge on the next turn.

### `MoveEffects.ts` legacy system still active
The old dispatch table is still the fallback for moves not in the registry. Once variable power attrs exist and the registry is complete, `MoveEffects.ts` and `movesDb` can be deleted.

### `String Shot` duplicated
`String Shot` (Bug type) is declared in both `GrassMoves.ts` and `BugMoves.ts`. The `GrassMoves.ts` entry should be removed.

---

## Architecture Debt

### `NodeMapScene` is a TODO for split
`NodeMapScene.ts` was split into `NodeMapGlobals`, `NodeMapRenderer`, `NodeMapInput`, and `NodeMapScene`. The scene is now ~310 lines (down from 1135). If it grows again a further split is warranted.
**Comment in file:** Yes

### `CatchCalc` hardcoded catch rate fallback
`CatchCalc.ts` uses `pokemonDb[target.speciesId]?.catchRate ?? 100`. The data is populated correctly; the `?? 100` fallback is safe but masks missing data silently.

### `BattlePokemon.moves[i].moveName` is display only
Move names are stored as strings for display but the battle engine uses numeric `moveId` for logic. If a move name ever diverges from the ID mapping this could cause UI/logic mismatch. Low risk, but worth auditing if move data changes.

---

## Unimplemented Mechanics

### Move effects
See `move_system.md` for the full list of unimplemented move categories.

### Held items do nothing in battle
All 15 held items can be equipped via the bag but `BattlePokemon.heldItem` is never read by the engine. None of the following effects are implemented:
- Shell Bell (heal on hit)
- Life Orb (1.3× damage, recoil)
- Choice Band/Specs/Scarf (1.5× + lock)
- Focus Sash (survive at 1 HP)
- Rocky Helmet (damage on contact)
- Assault Vest (block status moves, 1.5× SpDef)
- Leftovers / Black Sludge (end-of-turn heal/damage)
- Expert Belt (super effective boost)
- Eviolite (1.5× def/spdef if not fully evolved)
- Wide Lens / Zoom Lens / Kings Rock (accuracy/flinch modifiers)

### Voluntary switch does not cost a turn correctly
Voluntary switch is implemented but the opponent always gets a free attack after. The engine handles `{ type: 'switch' }` correctly — this is working as intended — but needs testing to confirm the opponent's free move resolves properly.

### Gym battles not implemented
Gym node shows a placeholder overlay. Gym leaders defined in story docs but no `GymScene` exists.

### Move Mastery system
Defined in `mechanics/move_mastery.md`. Not started. Requires meta shop first.

### Meta shop
Defined in `mechanics/meta_progression.md`. Not started. Requires Solwara Coins flow and a between-run shop scene.

### Evan's real sprite
Evan uses the Iris sprite as a placeholder. `public/images/trainers/iris.json` and `iris.png`.

### Rival team scaling across zones
Rival fights use `RivalTeamGen.ts` with predefined fight numbers. The scaling across all 8 zones has not been playtested.

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
