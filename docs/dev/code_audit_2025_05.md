# Solwara Code Audit — May 2025

Full review of `solwara-pokemon/solwara` at current `main`. Covers memory leaks, logic bugs, quality issues, and refactor opportunities now that weather, terrain, hazards, protect, OHKO, two-turn moves, held items, and the move attr system are all live.

---

## 1. Memory Leaks & Resource Cleanup

### 1.1 Phaser timer leak: delayedCall without cleanup in BattleScene ⚠️
**File:** `src/scenes/battle/BattleScene.ts`

`this.time.delayedCall` is used twice in `create()` — the `launchBattle` call and the `showTrainerIntro` call — but neither is stored or cancelled in the `shutdown` listener. If the scene is stopped mid-intro (e.g. the game reloads or the user force-navigates), these timers fire against a destroyed scene and call into dead globals like `typewriterText`.

**Fix:** Store both `delayedCall` returns and cancel them in the `'shutdown'` handler alongside the overlay cleanup that's already there.

```ts
// Already doing:
this.events.once('shutdown', () => {
  overlay.destroy();
  setWeatherOverlay(null);
  terrainOv.destroy();
  setTerrainOverlay(null);
  // ADD:
  introTimer?.remove();
  launchTimer?.remove();
});
```

### 1.2 Orphaned pointerdown listeners in BattleText on fast-skip ⚠️
**File:** `src/scenes/battle/BattleText.ts` → `typewriterText`

There's a 50ms `delayedCall` before registering the skip handler:
```ts
scene.time.delayedCall(50, () => {
  const handler = () => { skipHandler = null; if (!finished) finish(); };
  skipHandler = handler;
  scene.input.once('pointerdown', handler);
});
```

If `clearAllListeners` is called during this 50ms window (because a new message came in), the 50ms timer has already been queued but `skipHandler` is still null — so the timer fires, registers a handler, and nothing clears it. The handler will then fire spuriously on the next tap and advance a message it shouldn't. This is a race condition, not just theoretical — spamming taps fast enough on mobile triggers it.

**Fix:** Store the 50ms `delayedCall` in a tracked variable and cancel it inside `clearAllListeners`:
```ts
let skipDelayTimer: Phaser.Time.TimerEvent | null = null;

export function clearAllListeners(scene: Phaser.Scene) {
  skipDelayTimer?.remove(); skipDelayTimer = null;
  // ... rest unchanged
}
```

### 1.3 Graphics objects in BattleHUD are never destroyed ℹ️
**File:** `src/scenes/battle/BattleHUD.ts`

`buildHUD` creates several `scene.add.graphics()` objects (panel backgrounds, HP bar backgrounds) that have no `destroy()` path. Phaser scenes clean up their display lists on `shutdown`, but the module-level refs (`playerHpBar`, `enemyHpBar`, etc.) still hold stale references after the scene restarts. On the next `buildHUD` call, new objects are created and the old refs are overwritten without ever being explicitly destroyed.

This is low risk in practice because Phaser's scene manager wipes the display list on `shutdown` and the scene starts fresh on each battle — but it's worth auditing once scene restart is confirmed to call `shutdown` cleanly. Setting all module-level refs to `null` in a `clearHUD()` export (called from BattleGlobals' shutdown) would make intent clearer and prevent subtle bugs if the scene is ever reused.

### 1.4 uidCounter on BattlePokemon is module-level and never resets ℹ️
**File:** `src/systems/battle/BattlePokemon.ts`

```ts
let uidCounter = 0;
```

This persists across battles. Not a memory leak per se, but after many battles the counter can grow large. No actual impact unless the UIDs are compared numerically (they're strings and used as identity only). Harmless, but worth noting.

---

## 2. Logic Bugs

### 2.1 opponentFaintedThisTurn check doesn't handle ForceSwitchAttr correctly 🐛
**File:** `src/systems/battle/BattleEngine.ts` → `resolveTurnToEvents`

```ts
const prevOpp = state.opponentActive;
resolveMove(events, state, state.playerActive, state.opponentActive, playerAction.moveIndex, false);
if (state.opponentActive !== prevOpp) opponentFaintedThisTurn = true;
```

This flag is used to skip the opponent's move if the opponent fainted. But `state.opponentActive` can also change via `ForceSwitchAttr` (Dragon Tail / Circle Throw) when the opponent is a trainer — a forced switch on a trainer battle swaps the active mon without faint. In that case `opponentFaintedThisTurn` is set to `true`, so the newly sent-out mon skips its move entirely this turn.

This is actually correct _for the force-switch case_ (in Gen V+ the forced-out mon doesn't get to move), but the variable name is misleading and could cause confusion. If the logic around this ever changes, the misnamed flag could cause the new mon to incorrectly skip its turn.

**Fix (low urgency):** Rename to `opponentMoveBlockedThisTurn` or restructure to track the reason explicitly.

### 2.2 Opponent move selection doesn't pass opponent hazards during switch action ⚠️
**File:** `src/systems/battle/BattleEngine.ts` → `resolveTurnToEvents` (switch branch)

```ts
resolveMove(events, state, state.opponentActive, state.playerActive,
  chooseMove({ user: state.opponentActive, target: state.playerActive,
    moves: opMoveData, rng: _rng, turn: state.turn,
    // opponentHazards: NOT PASSED — always undefined here
  }).moveIndex);
```

When the player switches, the AI's `chooseMove` is called without `opponentHazards`, so the AI can't avoid redundantly setting hazards it already set. The `run` and `ball` branches have the same gap:

```ts
const opChoice = chooseMove({ ..., opponentHazards: state.playerHazards }); // ✅ run branch
const opChoice = chooseMove({ ..., opponentHazards: state.playerHazards }); // ✅ ball branch
```

But the switch branch and the normal-turn branch don't pass `opponentHazards` to the inner `chooseMove` call that fires after the player switches.

**Fix:** Add `opponentHazards: state.playerHazards` to all `chooseMove` calls consistently.

### 2.3 Save version migration is incomplete — new fields can be missing 🐛
**File:** `src/systems/saves/SaveManager.ts` → `load()`

```ts
if (parsed.version === SAVE_VERSION) {
  if (!parsed.discoveredLore) parsed.discoveredLore = [];
  return parsed;
}
```

Only `discoveredLore` is backfilled. The following fields added after initial release are **not** backfilled:
- `RunSlot.inventory` (defaults to `{}` in `emptySlot` but `slot.inventory` was not always present)
- `RunSlot.charmsEquipped` (similar gap)
- `RunSlot.currentExp` on `RunPokemon`
- `RunSlot.heldItem` on `RunPokemon`
- `RunSlot.timeStep`

In practice, `saveRun` always writes `Object.assign` over any missing fields when a run starts, and `BattleSetup` uses `r.currentExp ?? 0` and `r.heldItem ?? null` defensively. But the gap between a save loaded from disk and a save that has gone through `startRun` means that some code paths accessing `slot.inventory` directly (without `?? {}`) could throw on an old save.

**Fix:** Extend the backfill block to cover all fields added after v1:
```ts
if (!parsed.runSlots[i].charmsEquipped) parsed.runSlots[i].charmsEquipped = [];
if (!parsed.runSlots[i].inventory)      parsed.runSlots[i].inventory = {};
// etc.
```

### 2.4 Freeze check fires twice per turn (pre-move and end-of-turn) 🐛
**File:** `src/systems/battle/StatusEffects.ts`

In `checkPreMove`:
```ts
if (mon.status === 'freeze') {
  if (rng() < 0.2) {
    mon.status = null;  // thaw check 1: at pre-move
    ...
  }
}
```

In `tickStatus`:
```ts
case 'freeze': {
  if (rng() < 0.2) {
    mon.status = null;  // thaw check 2: at end-of-turn
    ...
  }
}
```

A frozen Pokémon gets two 20% thaw chances per turn: once when trying to move, and again at end-of-turn. This effectively gives ~36% thaw per turn, which is above the intended 20% per turn from the main-series games (which only do the pre-move check).

**Fix:** Remove the freeze case from `tickStatus`. Freeze is already handled in `checkPreMove` and should not tick end-of-turn.

### 2.5 Toxic counter is not reset on status cure ℹ️
**File:** `src/systems/battle/StatusEffects.ts` → `tickStatus`

When a badly-poisoned Pokémon wakes up, thaws, or is healed by a move/item, `toxicCounter` is not reset to 0. If the same `BattlePokemon` instance is re-poisoned later in the same battle, the counter continues from where it left off, causing disproportionate damage.

This is low risk because curing poison usually involves items that fully clear the slot, and re-poisoning in the same battle is uncommon. But for full correctness, any code path that sets `mon.status = null` should also set `mon.toxicCounter = 0` (or reset it in `tryApplyStatus` when poison is applied).

### 2.6 Shell Side Arm calcDamage called twice, RNG consumed twice 🐛
**File:** `src/systems/battle/BattleEngine.ts` → `resolveMove`

```ts
const physResult = calcDamage(attacker, defender, { ...moveData, category: 'physical' }, _rng, state.weather);
const specResult = calcDamage(attacker, defender, { ...moveData, category: 'special'  }, _rng, state.weather);
```

`calcDamage` internally rolls a random damage factor. Running it twice to pick the higher result consumes two RNG values, then the chosen path re-runs damage calculation again in the main loop below. This means three RNG rolls for Shell Side Arm instead of one — the actual damage value won't match either of the two preview rolls since a new roll is made later. The category selection is correct, but the displayed damage is derived from a fresh roll.

**Fix:** After determining the winning category, pass the pre-calculated damage result directly without re-running `calcDamage`:
```ts
const winner = physDmg >= specDmg ? physResult : specResult;
finalDamage = winner.damage; // use the already-rolled result
```

### 2.7 `opponentFaintedThisTurn` tracks reference change, not faint ℹ️
**File:** `src/systems/battle/BattleEngine.ts`

```ts
const prevOpp = state.opponentActive;
resolveMove(...);
if (state.opponentActive !== prevOpp) opponentFaintedThisTurn = true;
```

If `handleFaint` replaces the active mon with the next party member (trainer battle), the reference changes and the flag fires — that's the intended case. But if `ForceSwitchAttr` (Dragon Tail) fires instead, the reference also changes without a faint. These two cases are conflated. Described more fully under 2.1 above.

---

## 3. Code Quality Issues

### 3.1 `as any` bypass in BattleMenus ⚠️
**File:** `src/scenes/battle/BattleMenus.ts:251`

```ts
const enabledBtns = moveBtns.filter(b => (b as any).bg?.input?.enabled !== false);
```

This reaches into the internals of `TextButton` to detect disabled state. The `TextButton` class should expose a typed `isEnabled: boolean` property or a `disabled` flag so this can be checked cleanly. The current pattern is fragile — if `TextButton`'s internal structure changes, this silently breaks.

**Fix:** Add `get isEnabled(): boolean` to `TextButton` that reads from `this.bg.input?.enabled ?? true`.

### 3.2 Duplicated team-save logic in BattleFlow ⚠️
**File:** `src/scenes/battle/BattleFlow.ts` → `endBattle`

The "save team state" block appears twice in `endBattle` — once for the `player_win` path and once for the `player_fled` path:

```ts
const savedTeam = slot.team.map((r, i) => {
  const bp = battleState.playerParty[i];
  if (!bp) return r;
  return { ...r, level: bp.level, hp: bp.currentHp, ... };
});
```

These blocks are byte-for-byte identical. Extract to a `syncTeamToSave()` helper and call it from both branches (and also from `player_lose` before `clearSlot`, if HP/PP recovery on loss matters for post-run stats).

### 3.3 BattleGlobals is a global mutable singleton ℹ️
**File:** `src/scenes/battle/BattleGlobals.ts`

The module-level `let` exports are set once in `BattleScene.create()` and then read by every other battle subsystem. This pattern works fine for a single-scene game but has no protection against concurrent or re-entrant access. If the battle scene is ever restarted before cleanup completes (e.g. fast fade-out + new battle), the globals would be in a mixed state.

This is the same pattern as PokeRogue's `globalScene` and is a known tradeoff. Not a bug now, but worth keeping in mind as more scenes are added. A class-based `BattleContext` passed through function arguments would be safer at scale.

### 3.4 `showDialogLines` is duplicated between BattleScene and BattleFlow ℹ️
**File:** `src/scenes/battle/BattleScene.ts` and `src/scenes/battle/BattleFlow.ts`

Both files define an identical `showDialogLines(lines, onDone)` local function. Extract to `BattleText.ts` as an exported helper.

### 3.5 Tech debt doc is out of date ℹ️
**File:** `docs/dev/tech_debt.md`

Several "Unimplemented Mechanics" listed in the tech debt doc are now fully implemented:
- Weather system ✅ (WeatherOverlay, weatherTurns, end-of-turn damage, stat boosts)
- Terrain system ✅ (TerrainOverlay, terrainTurns, power modifiers, terrain immunity)
- Entry hazards ✅ (HazardState, applyHazards, EntryHazardAttr)
- Protect variants ✅ (ProtectAttr, protectedThisTurn, consecutiveProtects, all variants)
- OHKO moves ✅ (OhkoAttr, Sheer Cold ice-type penalty)
- Switch-out effects ✅ (SwitchOutAttr, ForceSwitchAttr, U-turn/Volt Switch/Dragon Tail)
- Delayed damage ✅ (DelayedDamageAttr, DelayedMove, tickDelayed)
- Two-turn move invulnerability ✅ (INVULNERABILITY_BYPASSES map, invisible flag)

The tech debt doc also lists the two-turn move invulnerability as a bug, but it was fixed. Archive resolved items so the doc reflects current reality.

### 3.6 `void moveName` suppresses linter warning, not intent ℹ️
**File:** `src/systems/battle/BattleEngine.ts` → `resolveStatusMove`

```ts
const moveName = registeredMove.type === 'psychic' ? 'Future Sight' : 'Doom Desire';
msg(events, `${attacker.displayName} foresaw an attack!`);
void moveName;  // ← declared but only used in this void
```

The `moveName` variable is computed but never used. Delete the variable and the `void` suppression, or use it in the message string.

### 3.7 Hardcoded `exp-charm` and `exp-all` inventory keys in BattleSetup ℹ️
**File:** `src/systems/battle/BattleSetup.ts`

```ts
const expCharms   = Math.min(50, inventory['exp-charm'] ?? 0);
const expAllCount = Math.min(5,  inventory['exp-all']   ?? 0);
```

These string keys should be constants from `ItemData.ts`. If the item keys ever change, this would silently give all players 0 exp bonuses.

### 3.8 `SaveCrypto` secret is hardcoded plaintext ℹ️
**File:** `src/utils/SaveCrypto.ts`

```ts
const APP_SECRET = '5mMcd6Z2tDb14Q5575512WoB05i';
```

This is fine for a client-side game where the encryption is tamper-detection rather than confidentiality — the GCM auth tag prevents undetected modification. It's worth adding a comment making this intent explicit, so future devs don't assume it's a private key that needs protecting.

---

## 4. Refactor Opportunities

### 4.1 BattleEngine `resolveMove` is 350+ lines — split into phases ℹ️
**File:** `src/systems/battle/BattleEngine.ts`

`resolveMove` handles 10+ distinct phases sequentially (pre-checks, priority moves, recharge, status, accuracy, damage, multi-hit, secondary effects, switch-out, force-switch) and is hard to follow. Each phase is already well-commented, but the function is too large to modify safely.

Suggested split:
- `runPreMoveChecks(events, state, attacker, defender, moveData)` → returns `canContinue` 
- `runDamageLoop(events, state, attacker, defender, moveData, hits, ctx)` → returns `totalDmg`
- Keep `resolveMove` as a thin orchestrator

### 4.2 Dual delayed-damage handling (attack move and status move branches) ℹ️
**File:** `src/systems/battle/BattleEngine.ts`

`DelayedDamageAttr` is handled twice: once in the attack-move branch (lines ~195-213) and once in `resolveStatusMove`. Both blocks compute the same `baseDmg` formula with the same logic. The attack-move branch appears to be a legacy path — `DelayedDamageAttr` should only fire for status moves (Future Sight/Doom Desire are status category). Verify and remove the dead attack-move intercept if confirmed.

### 4.3 Weather accuracy override duplicates data from move registry ℹ️
**File:** `src/systems/battle/BattleEngine.ts`

```ts
const moveLower = moveData.name.toLowerCase().replace(/ /g, '-');
if (state.weather === 'rain' && (moveLower === 'thunder' || moveLower === 'hurricane')) {
  weatherAccOverride = Infinity;
}
```

This hardcodes move names as strings. Thunder and Hurricane are in the move registry — a `WeatherAccuracyAttr` or flag on the registered move would be cleaner and eliminate the string matching.

### 4.4 MapGenerator zone generation uses `Math.random()` directly ℹ️
**File:** `src/systems/map/MapGenerator.ts` (inferred from `MapGenerator` usage in NodeMapScene)

`generateFullRun(slot.mapSeed)` seeds a `SeededRNG` but if the generator ever falls through to `Math.random()` internally (e.g. for sub-steps), maps won't be reproducible across devices. Verify all random calls inside `MapGenerator` route through the seeded RNG. 

### 4.5 `BattleMenus.ts` mixes rendering and battle logic ℹ️
**File:** `src/scenes/battle/BattleMenus.ts`

The bag/catch flow in `showBagMenu` calls `calcCatch`, `resolveTurnToEvents`, and `gainExp` directly — core battle-engine functions called from a Phaser UI file. This creates a tight coupling between the menu layer and the engine. Catch resolution should be modelled as a `PlayerAction` type (`{ type: 'ball', ballKey }`) and handled by `resolveTurnToEvents` like other actions. The scene would then just pass the action and play the resulting events.

### 4.6 `switch_required` event handling in BattleFlow clears the remaining queue ⚠️
**File:** `src/scenes/battle/BattleFlow.ts`

```ts
case 'switch_required': {
  const resumeEvents = [...remaining].filter(e => !(e.type === 'move_use' && e.moverIsPlayer));
  remaining.length = 0;
  showForcedSwitchMenu(playEventQueue, () => playEventQueue(resumeEvents, onDone));
  break;
}
```

The `remaining.length = 0` mutates the live queue array that the outer `next()` loop is iterating. While this works because the event handler returns immediately after showing the menu, it's fragile — any refactor that calls `next()` after the menu appears would process an empty queue and lose events. A safer pattern is to not mutate `remaining` at all and instead break the `playEventQueue` call chain by creating a fresh queue for the resume path.

---

## 5. System Interaction Notes

### 5.1 Level cap stat recalculation in gainExp ✅ correct but incomplete
**File:** `src/systems/battle/BattleEngine.ts` → `gainExp`

When a Pokémon levels up, stats are recalculated without natures or EVs (consistent with the rest of the engine). The formula correctly handles the Shedinja special case (HP = 1 always) indirectly — Shedinja's base HP of 1 gives `maxHp = 1` at any level. Worth a comment confirming this is intentional.

### 5.2 Exp share interacts correctly with level cap
`calcExpBase` computes base EXP independently of the winner's level cap, so benched Pokémon still receive EXP share even when the active mon is already at cap. This is correct and well-commented.

### 5.3 AI hazard tracking missing from the switch/normal branches (see 2.2)
The AI's hazard-avoidance score correctly penalises resetting already-set hazards, but only when `opponentHazards` is passed. Two call sites omit it. See bug 2.2 for the fix.

### 5.4 SeededRNG and `setBattleRng` — battle is deterministic per seed ✅
`BattleSetup.buildBattleState` seeds a `SeededRNG` from `slot.mapSeed ^ (fightNum * 0x1337)` and calls `setBattleRng`. All RNG in the engine flows through `_rng`, so replaying the same battle from the same seed should produce the same outcome. This is good for debugging.

---

## 6. Priority Summary

| Priority | Issue | File |
|----------|-------|------|
| 🔴 Bug | Freeze checks twice per turn (+16% extra thaw) | `StatusEffects.ts` |
| 🔴 Bug | Shell Side Arm re-rolls damage after choosing category | `BattleEngine.ts` |
| 🟡 Bug | Switch action doesn't pass hazard context to AI | `BattleEngine.ts` |
| 🟡 Bug | Save backfill only covers `discoveredLore` | `SaveManager.ts` |
| 🟡 Leak | `delayedCall` timers not cancelled on shutdown | `BattleScene.ts` |
| 🟡 Leak | Skip-handler 50ms race in typewriter | `BattleText.ts` |
| 🟡 Quality | `as any` probe inside TextButton | `BattleMenus.ts` |
| 🟡 Quality | Duplicated `showDialogLines` function | `BattleScene.ts` / `BattleFlow.ts` |
| 🟡 Quality | Duplicated team-save block in endBattle | `BattleFlow.ts` |
| 🟠 Quality | `void moveName` dead variable | `BattleEngine.ts` |
| 🟠 Quality | `switch_required` mutates live queue array | `BattleFlow.ts` |
| 🟠 Refactor | `resolveMove` is 350+ lines | `BattleEngine.ts` |
| 🟠 Refactor | Catch/bag logic in menu layer | `BattleMenus.ts` |
| 🟠 Debt | Tech debt doc lists already-fixed items | `tech_debt.md` |
| ⚪ Minor | `toxicCounter` not reset on cure | `StatusEffects.ts` |
| ⚪ Minor | Hardcoded inventory key strings | `BattleSetup.ts` |
| ⚪ Minor | Duplicate delayed-damage handling branches | `BattleEngine.ts` |
| ⚪ Minor | Weather accuracy by string match | `BattleEngine.ts` |
| ⚪ Info | `uidCounter` never resets across battles | `BattlePokemon.ts` |

---

*Audit performed on `solwara` at HEAD of `main` and `lore-building` at HEAD of `main`, May 2025.*
