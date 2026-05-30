# Changelog

---

## 0.1.4 — 2026-05-30

Alpha prep session — UI polish, trainer case, meta currency wiring.

### Features
- **Trainer Case overlay** — accessible from map and battle via "Trainer" button (left of Settings); shows trainer ID (5-digit from mapSeed), role, in-run money, and 8 badge slots filling in as gyms are beaten; Hope Badge always greyed out
- **Solwara Coins panel** — framed gold display on title screen; hidden at zero so new players don't see a confusing ✦ 0
- **Solwara Coins awarded on loss** — 10% of inRunMoney converted to persistent coins before clearSlot(); was designed but never implemented
- **In-battle Pokéball select via BagViewer** — `'battle'` mode shows balls only, sorted best-first; keyboard navigable (UP/DOWN, ENTER/ESC); teal focus highlight scrolls with list
- **Battle action menu redesign** — FIGHT is primary (tall, teal border, always glowing); BAG/POKÉMON/RUN are a secondary row below; all buttons use Graphics for carved-stone look with inset depth; gradient panel background
- **Wave/rock edge image** — procedurally generated bioluminescent rock silhouette separating the field from the UI panel; shared between action menu and move menu
- **Move menu visual parity** — same gradient background and wave edge as action menu; detail panel upgraded to navy with teal separator

### Bug Fixes
- **Back button bleed-through** — pressing BACK from move menu immediately pressed the action menu button underneath; fixed by waiting for pointer up before rebuilding
- **BagViewer row taps swallowed** — invisible drag zone was sitting above all rows in Phaser's hit-test order; fixed by moving drag tracking onto each rowBg directly
- **BagViewer battle mode fires onClose after ball selected** — hide()'s onComplete called onBack which clobbered throwBall's flow; fixed with ballSelected flag
- **Settings import label orphaned** — HTML label element not removed on battle scene shutdown; fixed by calling settingsOverlay.hide() in shutdown handler
- **Action menu hitzones fired on pointerup** — caused bleed-through since back() waited for pointerup; changed to pointerdown to match InputManager

---

## 0.1.3 — May 2026

Full code audit pass + bug fixes. See [`audit_changes_2025_05.md`](./audit_changes_2025_05.md) for the detailed audit record.

### Bug Fixes
- **Freeze thaw rate** — was ~36%/turn due to double-checking; now correctly 20%/turn (pre-move only)
- **Shell Side Arm** — re-rolled damage a third time after category selection; now reuses the comparison result
- **BattleScene timer leak** — intro/launch `delayedCall` timers not cancelled on shutdown; fixed
- **BattleText skip-handler race** — 50ms deferred skip registration could escape `clearAllListeners`; fixed
- **AI hazard context** — `chooseMove` in `determineTurnOrder` wasn't receiving `opponentHazards`; fixed
- **`toxicCounter` not reset on cure** — re-poisoning mid-battle inherited stale counter; fixed via `clearStatus()` helper
- **Save migration** — load only backfilled `discoveredLore`; replaced with full `migrate()` system covering all fields
- **Player moveset ignored in battle** — `buildBattleState` was rebuilding moves from the level-up learnset every battle, ignoring the saved moveset entirely; trained and forgotten moves had no effect
- **Rest node move training** — `pp` and `maxPp` arrays not updated when teaching a new move (add or replace path)
- **Ball break-free** — failed catch routed to node map instead of returning to action menu
- **Caught Pokémon faint message** — successful catch incorrectly displayed `{mon} fainted!`
- **Battle crash on exit** — `kbCleanup` called `setSelected` on destroyed scene text objects

### Refactors (audit pass 3)
- `TextButton.isEnabled` getter — removes `as any` probe in `BattleMenus`
- `syncTeamToSave()` — deduplicates team-save block in `endBattle`
- `showDialogLines` — consolidated from `BattleScene` + `BattleFlow` into `BattleText`
- `void moveName` dead variable removed; Future Sight/Doom Desire now announce by name
- `ITEM_KEY_EXP_CHARM` / `ITEM_KEY_EXP_ALL` constants exported from `ItemData`
- `resolveMove` split — `resolveAttackMove` extracted (~250 lines); `resolveMove` is now a ~100 line orchestrator
- Thunder/Hurricane weather accuracy via `rainAccuracy()` flag on `Move` — no more string matching
- Catch/ball logic moved into engine — `ball_throw` and `catch_success` events added to `BattleEvents`
- `switch_required` queue mutation fixed — events drained via `shift()` into snapshot instead of `remaining.length = 0`

---

## 0.1.1

Initial playable build. Battle system, map system, node types, save/load.
