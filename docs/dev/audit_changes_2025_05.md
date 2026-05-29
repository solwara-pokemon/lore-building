# Audit-Driven Changes — May 2025

Fixes applied to `solwara-pokemon/solwara` as a direct result of the code audit in [`code_audit_2025_05.md`](./code_audit_2025_05.md). Covers the four issues that were actionable and safe to fix immediately. The remaining items from the audit remain open and are tracked in [`tech_debt.md`](./tech_debt.md).

Commit: `9c06133` on `solwara/main`.

---

## 1. Freeze thaw rate corrected (audit §2.4)

**Files:** `src/systems/battle/StatusEffects.ts`, `src/systems/battle/BattleEngine.ts`

**Problem:** A frozen Pokémon was getting two 20% thaw checks per turn — one in `checkPreMove` (when it tries to move) and a second in `tickStatus` (end-of-turn). The combined probability was approximately 36% per turn instead of the intended 20%.

**Fix:** Removed the `freeze` case entirely from `tickStatus`. Thaw is now handled exclusively in `checkPreMove`, matching main-series behaviour. The `rng` parameter on `tickStatus` was also removed since freeze was its only consumer — the signature is now `tickStatus(mon: BattlePokemon)` and the call site in `BattleEngine.endOfTurn` was updated to match.

**Effect on gameplay:** Freeze is now meaningfully more threatening. A frozen Pokémon has a 20% chance to thaw each turn it tries to act — down from an effective 36% that made freeze feel shorter than it should.

---

## 2. Shell Side Arm no longer re-rolls damage (audit §2.6)

**File:** `src/systems/battle/BattleEngine.ts`

**Problem:** Shell Side Arm's category selection (physical vs. special, whichever hits harder) called `calcDamage` twice to compare, then let the main damage loop call it a third time. Each `calcDamage` call consumes two RNG values (a crit roll and a random damage roll), so Shell Side Arm was consuming 6 RNG values per hit instead of 2 — and the final damage came from the third roll, not either of the comparison rolls.

**Fix:** The winning `DamageResult` from the comparison is now stored in `shellSideArmResult` and fed directly into the damage loop on hit 0, skipping the third `calcDamage` call. Multi-hit Shell Side Arm re-rolls from hit 2 onwards as expected (the stored result is cleared after first use).

**Effect on gameplay:** Shell Side Arm damage is now consistent with what the engine actually calculated when comparing the two categories. This also restores RNG determinism — battles replayed from the same seed will now produce the same Shell Side Arm outcomes.

**Effect on RNG:** Shell Side Arm now consumes 4 RNG values per use (2 per comparison roll × 2 categories) rather than 6. This shifts the RNG sequence for all subsequent rolls in the same battle, meaning seeded replays of battles where Shell Side Arm was used will produce different outcomes than before this fix. This is a one-time correction to a bug, not a balance change.

---

## 3. BattleScene intro timers cancelled on shutdown (audit §1.1)

**File:** `src/scenes/battle/BattleScene.ts`

**Problem:** `create()` fired two `this.time.delayedCall` timers (a 400ms intro delay and a nested 200ms launch delay inside `launchBattle`) but did not store their return values. If the scene was stopped or restarted before either timer fired — e.g. if the user navigated away during the fade-in — the callbacks would execute against a destroyed scene and call into stale globals like `typewriterText` and `playEventQueue`.

**Fix:** Both timers are now stored as private instance fields (`introTimer`, `launchTimer`) and explicitly cancelled in the existing `'shutdown'` event handler alongside the weather/terrain overlay cleanup.

```ts
// Before
this.time.delayedCall(ms(400), () => this.showTrainerIntro(...));

// After
this.introTimer = this.time.delayedCall(ms(400), () => this.showTrainerIntro(...));
// ...and in the shutdown handler:
this.introTimer?.remove();  this.introTimer  = null;
this.launchTimer?.remove(); this.launchTimer = null;
```

---

## 4. BattleText skip-handler race window closed (audit §1.2)

**File:** `src/scenes/battle/BattleText.ts`

**Problem:** The typewriter's skip-while-typing handler was registered with a 50ms delay (to avoid mis-firing on the tap that opened the previous menu). The `clearAllListeners` function could cancel a fully-registered `skipHandler`, but it had no way to cancel the pending 50ms timer that was *about* to register one. If a new message arrived within that 50ms window, `clearAllListeners` ran first (clearing nothing — `skipHandler` was still null), the 50ms timer then fired and registered a fresh handler, and nothing ever cleared it. The stale handler would then fire on the next unrelated tap and incorrectly advance a future message or menu interaction.

**Fix:** The 50ms `delayedCall` return is now stored in `skipDelayTimer` and cancelled first in `clearAllListeners`:

```ts
// New variable
let skipDelayTimer: Phaser.Time.TimerEvent | null = null;

// clearAllListeners now also does:
if (skipDelayTimer) { skipDelayTimer.remove(); skipDelayTimer = null; }

// The registration clears itself on fire:
skipDelayTimer = scene.time.delayedCall(50, () => {
  skipDelayTimer = null;
  // ... register skipHandler as before
});
```

This closes the race: any call to `clearAllListeners` within the 50ms window will now cancel the pending registration, so no stale handler can escape.

---

---

## Pass 2 — §2.2, §2.5, §2.3 (commit `92547f2`)

### §2.2 — AI hazard context in normal move turns

**File:** `src/systems/battle/BattleEngine.ts` → `determineTurnOrder`

The `chooseMove` call inside `determineTurnOrder` — which selects the opponent's move during regular (non-switch, non-run, non-ball) turns — was not passing `opponentHazards`. The three other call sites (run, ball, switch branches) already passed it correctly. The fix is a one-liner: add `opponentHazards: state.playerHazards` to the `determineTurnOrder` call.

**Effect:** The AI now correctly avoids wasting turns re-setting hazards it already placed during normal combat turns, matching the behaviour that was already correct in the escape/catch/switch branches.

### §2.5 — `toxicCounter` reset when status is cleared

**Files:** `src/systems/battle/StatusEffects.ts`, `src/systems/battle/moves/attrs/WakeUpSlapAttr.ts`

Added a `clearStatus(mon: BattlePokemon)` helper that resets `status`, `statusTurns`, and `toxicCounter` together. All in-battle status-clearing paths now use it:

- `tickStatus` — sleep waking up naturally
- `checkPreMove` — freeze thawing before a move
- `WakeUpSlapAttr` — waking the target on hit

`tryApplyStatus` already reset `toxicCounter = 1` on fresh badly-poison application, so the bug only manifested if a mon accumulated a counter, was cured via one of the paths above, then was re-poisoned by the opponent. The counter would resume from its previous value instead of starting fresh at 1/16. Now any status clear resets it to 0, so a fresh poison always starts clean.

### §2.3 — Save migration system

**File:** `src/systems/saves/SaveManager.ts`

Replaced the old `version === SAVE_VERSION` check + single `discoveredLore` backfill with a proper migration system:

- `migrate(raw)` — accepts any parsed JSON object, returns a fully-valid v1 `SaveData`. Preserves all recoverable data (coins, starters unlocked, runs completed, lore, run slots, team members). Fields absent from the raw save are filled with safe defaults.
- `migrateSlot(s, i)` — shapes one run slot, backfilling every field in the `RunSlot` interface.
- `migratePokemon(p)` — shapes one team member, backfilling `heldItem`, `currentExp`, `pp`, `maxPp`, and all other `RunPokemon` fields.

All load paths now go through `migrate()`:
- `load()` — called on startup from `localStorage`
- `loadFromData()` — called when importing a `.solsav` file

`migrate()` is idempotent for up-to-date saves, so current-version saves are unaffected. Pre-release saves with any missing fields are now fully recovered rather than discarded. `SAVE_VERSION` remains at `1` — the intent is that `1` is the release version and everything before it migrates in cleanly.

---

## Open items (updated)

| Audit ref | Issue | Status |
|-----------|-------|--------|
| §2.2 | AI hazard context on normal turns | ✅ Fixed pass 2 |
| §2.3 | Save backfill incomplete | ✅ Fixed pass 2 (full migration system) |
| §2.5 | `toxicCounter` not reset on cure | ✅ Fixed pass 2 |
| §3.1 | `as any` probe in BattleMenus | Open — TextButton refactor needed first |
| §3.2 | Duplicated team-save block in endBattle | Open — extract when endBattle changes |
| §3.4 | `showDialogLines` duplicated | Open — housekeeping pass |
| §3.5 | Tech debt doc lists implemented items | Open — next doc pass |
| §3.6 | `void moveName` dead variable | Open — fix opportunistically |
| §3.7 | Hardcoded inventory key strings | Open — fix when ItemData gets constants |
| §4.1 | `resolveMove` is 350+ lines | Open — major refactor, own branch |
| §4.2 | Duplicate delayed-damage handling | Open |
| §4.3 | Weather accuracy by string match | Open |
| §4.4–4.5 | Catch logic in menu layer; switch_required queue mutation | Open — design decisions |
