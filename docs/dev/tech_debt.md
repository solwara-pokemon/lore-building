# Tech Debt & Known Issues

Active list of known issues, deferred work, and architectural debt. Updated as items are resolved or discovered.

---

## Active Bugs

*No known active bugs as of May 2025 audit.*

---

## Unimplemented Mechanics

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

### In-battle bag UI
The BAG button in battle currently shows a basic item list. A proper in-battle bag (filter to usable battle items, use on party mon) is not yet implemented.

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

- ✅ Held items had no battle effect — fixed, full `HeldItemEffects.ts` attr system implemented (all 15 items)
- ✅ Ball items gave 1× instead of 5× and wrote to inventory — fixed, `ball:` effect handler in `RewardHandler` writes to `slot.balls`; fallback item key corrected
- ✅ Active mon appeared in voluntary/forced switch picker — fixed, `excludePartyIndex` option on `PartyPicker`
- ✅ Close button on forced switch picker caused softlock — fixed, button hidden when no `onCancel` provided
- ✅ Skip-wave on load — fixed, `currentZoneIndex` now persisted to save on every node entry and restored on continue
- ✅ Node marked visited before completion — fixed, `markNodeComplete` refactored to static method called by each node scene on success only
- ✅ Mystery node trainer/npc/rescue/settlement branches missing `nodeId` — fixed
- ✅ Lore/item no-entry fallback didn't call `markNodeComplete` — fixed
- ✅ `JSON.stringify` used for stat stage comparison — replaced with `stagesEqual()` helper
- ✅ `freshSlot` renamed from `freshSlot2` (naming leftover from refactor)
- ✅ Dead `nodeId` param on `markNodeEntered` — removed
- ✅ `String Shot` declared in both GrassMoves and BugMoves — removed from GrassMoves
- ✅ `ZONE_ORDER` duplicated in `RivalTeamGen` — fixed, now imports from `MapGenerator`
- ✅ `ms()` missing from `LoreScene` / `RestScene` fadeOut calls — fixed
- ✅ `caughtMon` typed as `any` in `BattleSceneData` — fixed, field added to interface
- ✅ `console.log` noise in `UIScene` — removed
- ✅ Exp share gave 0 when active mon at level cap — fixed via `calcExpBase`
- ✅ Exp bar animated for bench mons — fixed via `isActivePlayer` flag on `exp_gain` events
- ✅ RestScene custom party picker — replaced with `PartyPicker` `'pick'` mode
- ✅ Item node cards unresponsive — fixed, hit zone was offset due to container origin
- ✅ Ball throw animation froze after recoil — fixed, `RecoilAttr` now emits `hp_change` not custom `damage` event
- ✅ Rest wakes up early — fixed, sleep turns only decremented in `tickStatus` (end of turn), not `checkPreMove`
- ✅ Variable power moves did nothing — fixed via attr system
- ✅ Two-turn moves did nothing — fixed via `TwoTurnMoveAttr`
- ✅ `MoveEffects.ts` legacy fallback — deleted
- ✅ Status not showing on HUD after Rest/paralysis — fixed via `status_change` events
- ✅ Rest had no attrs despite `.implemented('full')` — fixed via `RestAttr`
- ✅ player_send_out event missing — sprite/HUD update now properly sequenced through event queue
- ✅ Weather system — fully implemented (WeatherOverlay, weatherTurns, end-of-turn damage, stat boosts)
- ✅ Terrain system — fully implemented (TerrainOverlay, terrainTurns, power modifiers, terrain immunity)
- ✅ Entry hazards — fully implemented (HazardState, applyHazards, EntryHazardAttr, all four hazard types)
- ✅ Protect variants — fully implemented (ProtectAttr, protectedThisTurn, consecutiveProtects, all variants)
- ✅ OHKO moves — fully implemented (OhkoAttr, Sheer Cold ice-type penalty)
- ✅ Switch-out effects — fully implemented (SwitchOutAttr, ForceSwitchAttr, U-turn/Volt Switch/Dragon Tail)
- ✅ Delayed damage — fully implemented (DelayedDamageAttr, DelayedMove, tickDelayed)
- ✅ Two-turn move invulnerability — fully implemented (INVULNERABILITY_BYPASSES map, invisible flag)
- ✅ Freeze thaw rate double-counted — fixed, thaw check now only in `checkPreMove` (20%/turn)
- ✅ Shell Side Arm re-rolled damage — fixed, winning DamageResult reused on hit 0
- ✅ BattleScene intro timers leaked on shutdown — fixed, stored and cancelled in shutdown handler
- ✅ BattleText skip-handler 50ms race — fixed, skipDelayTimer tracked and cancelled in clearAllListeners
- ✅ AI hazard context missing on normal turns — fixed, `opponentHazards` passed in determineTurnOrder
- ✅ `toxicCounter` not reset on status cure — fixed via `clearStatus()` helper
- ✅ Save backfill incomplete — fixed, full migration system with `migrate()` / `migrateSlot()` / `migratePokemon()`
- ✅ Thunder/Hurricane accuracy by string match — fixed, `rainAccuracy()` flag on Move, set on Thunder + Hurricane
- ✅ `as any` probe in BattleMenus — fixed, `TextButton.isEnabled` getter added
- ✅ `showDialogLines` duplicated — consolidated into BattleText export
- ✅ Team-save block duplicated in endBattle — extracted to `syncTeamToSave()`
- ✅ `void moveName` dead variable — removed; message now uses the variable
- ✅ Hardcoded inventory key strings — `ITEM_KEY_EXP_CHARM` / `ITEM_KEY_EXP_ALL` constants in ItemData
- ✅ Dead `DelayedDamageAttr` block in resolveStatusMove — marked as unreachable with explanation
- ✅ `resolveMove` 350+ lines — split; `resolveAttackMove` extracted as a separate phase function
- ✅ Catch/ball logic in menu layer — engine now emits `ball_throw` + `catch_success` events; BattleFlow handles save
- ✅ `switch_required` mutated live queue array — fixed, events drained via shift() into snapshot array

