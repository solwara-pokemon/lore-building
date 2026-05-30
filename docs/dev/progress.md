# Development Progress

## Completed

> Items are dated _(YYYY-MM-DD)_ with the date the feature was merged/committed.

### Infrastructure
- ✅ Proxmox LXC setup (Debian 13, NAT, VPN access) _(2026-05-22)_
- ✅ Nginx serving solwara.scooom.com _(2026-05-22)_
- ✅ Phaser 4 + TypeScript + Vite scaffold _(2026-05-22)_
- ✅ Deploy service (`systemctl start solwara-deploy`) _(2026-05-22)_
- ✅ Pokemon sprites committed directly to repo (no submodule) _(2026-05-24)_

### UI / UX
- ✅ Theme system (maroon #4a0000, Cinzel + Crimson Pro fonts) _(2026-05-23)_
- ✅ i18n system with English translations _(2026-05-23)_
- ✅ InputManager (normalizes mouse/touch) _(2026-05-23)_
- ✅ KeyBindings system (configurable, persisted) _(2026-05-23)_
- ✅ BootScene → SplashScene → TitleScene flow _(2026-05-23)_
- ✅ SettingsScene (accessible from all scenes via per-scene button, returnTo pattern) _(2026-05-23)_
- ✅ Role Select, Starter Select, Slot Select scenes _(2026-05-23)_
- ✅ Logbook on title screen (browse all discovered lore) _(2026-05-24)_
- ✅ BagViewer — scrollable bag overlay (finger/mouse drag, one item per row, tier colour-coded, opens PartyPicker on tap) _(2026-05-28)_

### Battle System
- ✅ Typed BattleEvent[] stream — all battle actions are events _(2026-05-24)_
- ✅ Sequential event queue (playEventQueue) — one event at a time, tap to advance _(2026-05-24)_
- ✅ Move animations (PokeRogue battle-anims system) _(2026-05-23)_
- ✅ HP bars animate with move animation (smooth tween, color-coded) _(2026-05-23)_
- ✅ Damage/heal numbers _(2026-05-23)_
- ✅ Typewriter text — skip on first tap, advance on second _(2026-05-24)_
- ✅ Two-tap move selection (first tap = info, second tap = confirm) _(2026-05-24)_
- ✅ Move detail panel: type/category icons (PokeRogue assets), PP/Power/Accuracy _(2026-05-24)_
- ✅ Stat stage popup (hold nameplate) _(2026-05-24)_
- ✅ Nameplates with type badges, stat stages, status condition (colour-coded, updates mid-turn) _(2026-05-24)_
- ✅ Status display on HUD — colour-coded pill, refreshes on apply and on cure _(2026-05-26)_
- ✅ EXP bar + level up events (all 6 growth rate curves) _(2026-05-24)_
- ✅ Faint → forced switch menu (PartyPicker, no close button) _(2026-05-24)_
- ✅ Enemy auto-swap on faint (enemy_send_out event, sprite rebuild) _(2026-05-24)_
- ✅ Fainted pokemon's move skipped (engine-level fix) _(2026-05-24)_
- ✅ Catching grants EXP (after caught message) _(2026-05-24)_
- ✅ Failed catch → opponent attacks _(2026-05-24)_
- ✅ Back sprites for player pokemon _(2026-05-24)_
- ✅ Status conditions, abilities, type effectiveness _(2026-05-23)_
- ✅ Voluntary switch during battle (PartyPicker 'pick' mode, SWITCH overlay on first tap, second tap confirms; active mon excluded from picker) _(2026-05-26)_
- ✅ Pokéball throw animation (arc, open, shrink, bounce, shake, stars on catch) _(2026-05-26)_
- ✅ Real catch rates from PokéAPI data _(2026-05-26)_
- ✅ Move priority system wired to MoveRegistry (not movesDb) _(2026-05-27)_
- ✅ Sleep Talk — Gen IX unselectable list (including Rest), Comatose support, actual sub-move execution, PP not consumed _(2026-05-26)_
- ✅ player_send_out event — sprite + HUD update properly sequenced through event queue on both voluntary and forced switch _(2026-05-28)_

### Held Item System
- ✅ HeldItemEffects.ts — 7 hook functions (modifyDamageDealt, modifyDamageReceived, modifyStat, afterDamageDealt, endOfTurn, accuracyModifier, focusSash) _(2026-05-28)_
- ✅ Life Orb — 1.3× damage, 1/10 maxHP recoil after each hit _(2026-05-28)_
- ✅ Shell Bell — heals 1/8 of damage dealt _(2026-05-28)_
- ✅ Choice Band — 1.5× Attack (physical), locks first move used _(2026-05-28)_
- ✅ Choice Specs — 1.5× Sp. Attack (special), locks first move used _(2026-05-28)_
- ✅ Choice Scarf — 1.5× Speed, locks first move used; locked moves greyed out in move menu _(2026-05-28)_
- ✅ Expert Belt — 1.2× damage on super effective hits _(2026-05-28)_
- ✅ Assault Vest — special damage ÷1.5; status moves blocked with message _(2026-05-28)_
- ✅ Eviolite — 1.5× Def and SpDef if not fully evolved (checks evoChains) _(2026-05-28)_
- ✅ Leftovers — +1/16 maxHP at end of turn _(2026-05-28)_
- ✅ Black Sludge — +1/16 maxHP for Poison types; -1/8 maxHP for others _(2026-05-28)_
- ✅ Rocky Helmet — deals 1/6 attacker's maxHP on contact moves _(2026-05-28)_
- ✅ Focus Sash — survive one KO from full HP; item consumed on trigger _(2026-05-28)_
- ✅ Kings Rock — 10% flinch chance on all damaging moves _(2026-05-28)_
- ✅ Wide Lens — ×1.1 accuracy _(2026-05-28)_
- ✅ Zoom Lens — ×1.2 accuracy only when moving second (movingSecond flag threaded through engine) _(2026-05-28)_
- ✅ lockedMoveIndex cleared on switch-out _(2026-05-28)_

### Move System — Complete
- ✅ MoveId const object (789 moves, PokéAPI numeric IDs) _(2026-05-26)_
- ✅ Move / AttackMove / StatusMove / SelfStatusMove base classes with `.attr()` chaining _(2026-05-26)_
- ✅ MoveAttr composition system — 16 attr classes: _(2026-05-26)_
  - `StatusEffectAttr` (accepts selfTarget param), `StatChangeAttr`, `RecoilAttr`, `DrainAttr`, `HealAttr`
  - `FlinchAttr`, `ConfuseAttr`, `FixedDamageAttr`, `LevelDamageAttr`, `MultiHitAttr`
  - `HpRatioDamageAttr`, `WeightDamageAttr`, `SpeedRatioDamageAttr`, `TargetHpDamageAttr`
  - `StageScaledDamageAttr`, `TwoTurnMoveAttr`
  - `RestAttr` — full heal, clear status, apply 2-turn sleep (Gen IX)
- ✅ MoveRegistry — all 789 moves registered at boot, engine routes through registry exclusively _(2026-05-27)_
- ✅ All 16 type families fully declared (789/789 coverage) _(2026-05-27)_
- ✅ Variable power moves fully implemented _(2026-05-27)_
- ✅ Two-turn moves fully implemented _(2026-05-28)_
- ✅ Fixed damage, level-based damage, multi-hit moves _(2026-05-26)_
- ✅ Rest properly implemented via RestAttr (was declared but missing attrs) _(2026-05-28)_
- ✅ MoveEffects.ts deleted — legacy system fully gone _(2026-05-28)_

### Trainer / Rival System
- ✅ Unified trainer intro: sprite slides in → dialog in battle text box → sprite slides out → pokemon slide in _(2026-05-24)_
- ✅ Trainer sprites from PokeRogue (youngster_m, iris for Evan) _(2026-05-24)_
- ✅ TrainerDef with preDialog / winDialog / loseDialog _(2026-05-24)_
- ✅ Trainer node: picks random class per zone layer, seeded team generation _(2026-05-24)_
- ✅ Youngster class: 2 base Normal-type pokemon _(2026-05-24)_
- ✅ All post-battle dialog through battle text box (no DialogueOverlay) _(2026-05-24)_
- ✅ Win/loss dialog for both trainers and rival _(2026-05-24)_

### Map / Progression
- ✅ StS-style 7×15 procedural map (6 seeded paths, no-crossing invariant) _(2026-05-26)_
- ✅ Floor guarantees — floor 0 wild, floor 8 item, rival floor, floor 13 settlement/rest, floor 14 gym _(2026-05-26)_
- ✅ Isometric stagger layout _(2026-05-26)_
- ✅ Pan / zoom / pinch-zoom (activePointers: 2) _(2026-05-25)_
- ✅ Starts centred on active node at 1.6× zoom _(2026-05-26)_
- ✅ Node selection; node only marked visited on completion (markNodeComplete refactor) _(2026-05-28)_
- ✅ markNodeComplete — static method on NodeMapScene; every node scene calls it on successful completion before returning to map _(2026-05-28)_
- ✅ Mystery node — all 8 resolved types pass nodeId correctly; lore/item no-entry fallback calls markNodeComplete _(2026-05-28)_
- ✅ currentZoneIndex persisted to save on every node entry; restored on load (fixes skip-wave bug) _(2026-05-28)_
- ✅ All node types: Wild, Trainer, Rival, Rest, Lore, Item, Settlement, Rescue, NPC, Mystery, Evolution _(2026-05-25)_
- ✅ Logbook (account-level, accessible from title + node map) _(2026-05-24)_
- ✅ Zone themes (8 distinct visual identities) _(2026-05-23)_

### Day/Night System
- ✅ 14-step cycle: 1-2=Dawn, 3-7=Day, 8-9=Dusk, 10-14=Night _(2026-05-24)_
- ✅ Random phase-boundary start per run _(2026-05-24)_
- ✅ Advances on every node completion _(2026-05-24)_
- ✅ Persists in save slot _(2026-05-24)_
- ✅ Affects lore weighting (3× matching phase, 0.25× non-matching) _(2026-05-24)_
- ✅ Rest scene background images vary by time of day _(2026-05-24)_

### Items
- ✅ Full consumable item pool with tier weights and floor unlocks _(2026-05-25)_
- ✅ Full Heal conditional — only appears if team member has non-volatile status condition _(2026-05-26)_
- ✅ All 15 held items fully implemented in battle (see Held Item System above) _(2026-05-28)_
- ✅ Ball items — Poké Ball ×5 (Common), Great Ball ×5 (Uncommon), Ultra Ball ×5 (Rare) _(2026-05-28)_
- ✅ Ball rewards correctly write to slot.balls (not inventory) via ball: effect handler in RewardHandler _(2026-05-28)_
- ✅ EXP Charm (stackable, +50% EXP each) _(2026-05-24)_
- ✅ Item loot generation passes party for conditional filtering _(2026-05-26)_
- ✅ Antidote, Awakening, Burn Heal removed (redundant given Full Heal) _(2026-05-26)_

### Save System
- ✅ Account-level: discoveredLore[], unlockedCharms, roles _(2026-05-23)_
- ✅ Run slots: team, inventory, timeStep, mapSeed, nodesVisited, balls, currentZoneIndex _(2026-05-23)_
- ✅ nodesVisited only written on node completion (not on entry) _(2026-05-26)_

### Lore / NPC System
- ✅ `pickLoreEntry()` — seeded RNG, zone + time-of-day filter, rarity weighting (common 6×, uncommon 3×, rare 1×), skips discovered _(2026-05-28)_
- ✅ `pickNpcEntry()` — same as above but filters to entries with `npcName`, no time-of-day restriction _(2026-05-28)_
- ✅ Both functions accept `Set<string> | string[]` for discovered IDs _(2026-05-28)_

### Code Quality
- ✅ `monName()` / `titleCase()` utility (`src/utils/monName.ts`) — all Pokémon name outputs now title-cased (Yungoos not YUNGOOS) _(2026-05-28)_
- ✅ PartyPicker `'select'` mode — single-tap pick, no SWITCH overlay (used in RestScene train flow) _(2026-05-28)_
- ✅ Evan sprite updated to new uploaded art _(2026-05-29)_
- ✅ `SegmentedControl` component — reusable horizontal option picker (highlighted active segment, generic typed API) _(2026-05-29)_
- ✅ TypeScript zero-error baseline — fixed all pre-existing errors (missing imports, undefined→null, BagViewer wheel handler) _(2026-05-29)_

### Settings UI
- ✅ Full-screen layout — labelled rows with subtitles, bottom bar (Back left / Export↓ Import↑ right) _(2026-05-29)_
- ✅ Game Speed — `SegmentedControl` (Slow / Normal / Fast), replaces tap-to-cycle _(2026-05-29)_
- ✅ Move Animations toggle — ON/OFF, gates `animPlayer.play()` in BattleFlow _(2026-05-29)_
- ✅ Weather Animations toggle — ON/OFF, gates `WeatherOverlay.setWeather()` visuals _(2026-05-29)_
- ✅ Damage Numbers toggle (existing, now in full-screen row layout) _(2026-05-29)_

### Weather System
- ✅ Rain, Sun, Sandstorm, Snow (Hail skipped — Gen 9 replaced by Snow) _(2026-05-29)_
- ✅ Damage modifiers: Water/Fire ×1.5/×0.5 in Rain; Fire/Water ×1.5/×0.5 in Sun _(2026-05-29)_
- ✅ End-of-turn: Sandstorm 1/16 damage (Rock/Ground/Steel immune); Snow no damage (Ice +50% Def only) _(2026-05-29)_
- ✅ Accuracy overrides: Thunder/Hurricane always hit in Rain, 50% in Sun _(2026-05-29)_
- ✅ Speed abilities: Swift Swim, Chlorophyll, Sand Rush, Slush Rush (×2 in matching weather) _(2026-05-29)_
- ✅ Weather-setting abilities on switch-in: Drought, Drizzle, Sand Stream, Snow Warning _(2026-05-29)_
- ✅ Ability end-of-turn: Rain Dish, Dry Skin, Solar Power, Ice Body _(2026-05-29)_
- ✅ WeatherOverlay — particle emitters + screen tint per weather type, respects ms()/getSpeedFactor() _(2026-05-29)_
- ✅ `weather_change` battle event drives visual transitions; clears on expiry or overwrite _(2026-05-29)_

### Terrain System
- ✅ TerrainAttr — mirrors WeatherAttr, engine reads to set terrain state _(2026-05-29)_
- ✅ BattleState terrain + terrainTurns fields; terrain_change event type _(2026-05-29)_
- ✅ MoveContext terrain field threaded to all attrs _(2026-05-29)_
- ✅ Terrain power modifiers: +30% Electric/Grass/Psychic (grounded attacker); −50% Dragon (Misty, grounded defender) _(2026-05-29)_
- ✅ Grassy Terrain halves Earthquake, Bulldoze, Magnitude _(2026-05-29)_
- ✅ Grassy Terrain end-of-turn +1/16 HP heal for grounded Pokémon _(2026-05-29)_
- ✅ 5-turn expiry with messages; terrain_change event on expiry _(2026-05-29)_
- ✅ Misty Terrain blocks all non-volatile status + confusion (grounded only) _(2026-05-29)_
- ✅ Electric Terrain blocks sleep (grounded only) _(2026-05-29)_
- ✅ Psychic Terrain blocks priority moves hitting grounded Pokémon _(2026-05-29)_
- ✅ Surge abilities (electric-surge, grassy-surge, misty-surge, psychic-surge) in onSwitchIn _(2026-05-29)_
- ✅ All 4 terrain moves registered: Electric Terrain, Grassy Terrain, Misty Terrain, Psychic Terrain _(2026-05-29)_
- ✅ TerrainOverlay — bottom-edge gradient glow + subtle field tint, colour per terrain (yellow/green/purple/pink), gentle alpha pulse; reuses showWeatherAnimations toggle; depth 52 _(2026-05-29)_

### Battle System — Additional
- ✅ Parting Shot (Dark status move) — lowers Atk/SpAtk then triggers switch UI _(2026-05-29)_
- ✅ Recharge moves — Hyper Beam, Giga Impact, Rock Wrecker, Roar of Time, Eternabeam, Meteor Assault, Prismatic Laser all skip next turn via RechargeAttr _(2026-05-29)_
- ✅ Foul Play — uses target's Attack stat (FoulPlayAttr, statAtkMult in damage path) _(2026-05-29)_
- ✅ Body Press — uses user's Defense stat instead of Attack (BodyPressAttr) _(2026-05-29)_
- ✅ Knock Off — 1.5× damage if target holds item, then removes the item (KnockOffAttr) _(2026-05-29)_
- ✅ Assurance — doubles power if target was already damaged this turn (AssuranceAttr + damagedThisTurn flag) _(2026-05-29)_
- ✅ Payback — doubles power if user moved second (PaybackAttr + movedSecond in MoveContext) _(2026-05-29)_
- ✅ Revenge — doubles power if user was damaged this turn (RevengeAttr) _(2026-05-29)_
- ✅ Wake-Up Slap — doubles power vs sleeping target, cures sleep (WakeUpSlapAttr) _(2026-05-29)_
- ✅ Sucker Punch — fails if target is not using a damaging move (SuckerPunchAttr + pendingPlayerMoveIndex) _(2026-05-29)_
- ✅ Dragon Tail / Circle Throw — force opponent to switch to random party member; ends wild battle (ForceSwitchAttr) _(2026-05-29)_
- ✅ Battle messages auto-advance after pause (BASE_MSG_PAUSE); tap skips wait; trainer dialog still tap-to-continue _(2026-05-29)_
- ✅ Stale pointerdown listeners tracked and explicitly removed — fixes pokéball rethrow bug _(2026-05-29)_
- ✅ Fast-tap race condition fixed — autoAdvance handler removes itself before nulling ref; no orphaned .on() listeners; cancelTypewriter clears all listeners _(2026-05-29)_
- ✅ setPromptText() — static "What will X do?" label, registers zero input listeners _(2026-05-29)_
- ✅ Run (flee) now saves full team state (HP/PP/status) and calls markNodeComplete before returning to map _(2026-05-29)_
- ✅ Damage numbers: larger (26px) and linger longer (1400ms) _(2026-05-29)_
- ✅ Active mon greyed out in PartyPicker (dark blue tint, "IN BATTLE" label, untappable) _(2026-05-29)_
- ✅ AI improvements: OHKO level check, Protect scoring by HP ratio, hazard awareness (skips already-set hazards), pivot moves boosted at low HP _(2026-05-29)_
- ✅ Correct `implemented('partial')` flags — 19 moves demoted from false 'full' status _(2026-05-29)_

### Contact System
- ✅ `AttackMove.contact` on base `Move` class (physical default true, special default false) _(2026-05-29)_
- ✅ `.noContact()` — 40 physical non-contact moves marked (Earthquake, Rock Slide, Explosion, etc.) _(2026-05-29)_
- ✅ `.makeContact()` — 5 special contact moves (Grass Knot, Draining Kiss, Infestation, Oblivion Wing, Sparkling Aria) _(2026-05-29)_
- ✅ Shell Side Arm — `ShellSideArmAttr`; probes calcDamage twice, uses higher, sets contact flag accordingly _(2026-05-29)_

### Protect Secondary Effects
- ✅ `protectVariant` stored on BattlePokemon when protect activates _(2026-05-28)_
- ✅ King's Shield → Attack −2 on contact; Spiky Shield → 1/8 HP damage; Baneful Bunker → poison; Obstruct → Def −2; Silk Trap → Speed −1 _(2026-05-28)_

### Version
- ✅ `src/version.ts` — VERSION = '0.1.1' _(2026-05-29)_
- ✅ Title screen shows *The Flooded Region* v0.1.1 (italic subtitle + plain version tag) _(2026-05-29)_


- ✅ `SceneTypes.ts` — `NodeMapReturnData` includes nodeId? for completion tracking _(2026-05-25)_
- ✅ `PartyPicker` 'pick' mode — excludePartyIndex option to hide active mon in battle _(2026-05-28)_
- ✅ `stagesEqual()` helper in StatStages — replaces JSON.stringify comparisons _(2026-05-28)_
- ✅ `ZONE_ORDER` single export from MapGenerator _(2026-05-25)_
- ✅ `ms()` wraps all timing values _(2026-05-23)_
- ✅ `status_change` events — HUD refreshes mid-turn on any status change _(2026-05-26)_

## TODO

### Battle UI
- [ ] Move menu keyboard navigation polish (currently functional but uses TextButton grid, not custom carved buttons like action menu)

### Content
- [ ] More trainer classes (currently only Youngster — all trainer battles feel samey)
- [ ] More lore entries (68 entries across 8 zones — passable for alpha, more needed for full release)
- [ ] Elite Four

### Meta Progression
- [ ] Meta shop (Solwara Coins can be spent — coins earn and display correctly, shop scene not started)
- [ ] Move Mastery system (save schema ready with `masteredMoves[]`, no UI or battle integration yet)

### Known Issues
- [ ] Settings overlay: opening during battle freeze can orphan the HTML import label (partially mitigated — shutdown cleanup added, edge case may remain)

## Recently Completed (this session, 2026-05-30)

- ✅ In-battle Pokéball use via BagViewer (`'battle'` mode — scrollable, keyboard navigable, fires throwBall on select)
- ✅ Battle action menu redesign — asymmetric layout (FIGHT primary left, BAG/POKÉMON/RUN secondary row), carved-stone Graphics buttons, gradient panel, wave/rock edge image
- ✅ Move menu visual parity — same gradient background and wave edge as action menu
- ✅ Back button bleed-through fixed — waits for pointer up before rebuilding action menu
- ✅ Settings import label orphan fix — BattleScene shutdown now calls settingsOverlay.hide()
- ✅ Trainer Case overlay — accessible from map and battle via "Trainer" button left of Settings; shows trainer ID, role, money, 8 badge slots (filled as earned; Hope Badge always greyed out)
- ✅ Solwara Coins panel on title screen — framed display, gold text, hidden at zero
- ✅ Solwara Coins actually awarded on run loss — 10% of inRunMoney converted via battleSave.addCoins() before clearSlot()
- ✅ Gym battles fully wired (was listed as TODO but was already complete — GymLeaderData routes through BattleScene with battleType: 'gym', badges awarded in BattleRewardScene)
- ✅ Evan sprite updated (was listed as TODO but already done)

## Architecture Notes
- **Phaser 4 only** — `filters.addGlow()`, not `postFX.addGlow()`
- Touch = primary, keyboard = bonus
- All battle text through `playEventQueue` / typewriter events
- All battle dialog (trainer/rival pre/post) through battle text box
- Settings: per-scene button with `returnTo` pattern
- Game speed: `ms()` wraps all timing values — ball animations use additional `0.7×` multiplier
- Trainer sprites: multiatlas from `public/images/trainers/`, frame `0001.png` = idle
- Battle animations: PokeRogue assets, `public/battle-anims/` and `public/images/battle_anims/`
- Type/category icons: PokeRogue `public/images/types.png` / `categories.png`
- Pokéball sprites: PokeRogue `public/images/pb.json` / `pb.png` (multiatlas, frames: `pb`, `pb_opening`, `pb_open` per ball type)
- Move registry populated at boot via `registerAll()` in SplashScene
- `movesDb` / `moves.json` still used by `BattleMenus`, `RestScene`, `RescueScene` for display (names, PP) — not battle logic
- Held item hooks: 7 functions in `HeldItemEffects.ts`, called from `BattleEngine` at exact points; `BattlePokemon` carries `heldItem` + `lockedMoveIndex`
