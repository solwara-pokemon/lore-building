# Development Progress

## Completed

### Infrastructure
- ✅ Proxmox LXC setup (Debian 13, NAT, VPN access)
- ✅ Nginx serving solwara.scooom.com
- ✅ Phaser 4 + TypeScript + Vite scaffold
- ✅ Deploy service (`systemctl start solwara-deploy`)
- ✅ Pokemon sprites committed directly to repo (no submodule)

### UI / UX
- ✅ Theme system (maroon #4a0000, Cinzel + Crimson Pro fonts)
- ✅ i18n system with English translations
- ✅ InputManager (normalizes mouse/touch)
- ✅ KeyBindings system (configurable, persisted)
- ✅ BootScene → SplashScene → TitleScene flow
- ✅ SettingsScene (accessible from all scenes via per-scene button, returnTo pattern)
- ✅ Role Select, Starter Select, Slot Select scenes
- ✅ Logbook on title screen (browse all discovered lore)
- ✅ BagViewer — scrollable bag overlay (finger/mouse drag, one item per row, tier colour-coded, opens PartyPicker on tap)

### Battle System
- ✅ Typed BattleEvent[] stream — all battle actions are events
- ✅ Sequential event queue (playEventQueue) — one event at a time, tap to advance
- ✅ Move animations (PokeRogue battle-anims system)
- ✅ HP bars animate with move animation (smooth tween, color-coded)
- ✅ Damage/heal numbers
- ✅ Typewriter text — skip on first tap, advance on second
- ✅ Two-tap move selection (first tap = info, second tap = confirm)
- ✅ Move detail panel: type/category icons (PokeRogue assets), PP/Power/Accuracy
- ✅ Stat stage popup (hold nameplate)
- ✅ Nameplates with type badges, stat stages, status condition (colour-coded, updates mid-turn)
- ✅ Status display on HUD — colour-coded pill, refreshes on apply and on cure
- ✅ EXP bar + level up events (all 6 growth rate curves)
- ✅ Faint → forced switch menu (PartyPicker, no close button)
- ✅ Enemy auto-swap on faint (enemy_send_out event, sprite rebuild)
- ✅ Fainted pokemon's move skipped (engine-level fix)
- ✅ Catching grants EXP (after caught message)
- ✅ Failed catch → opponent attacks
- ✅ Back sprites for player pokemon
- ✅ Status conditions, abilities, type effectiveness
- ✅ Voluntary switch during battle (PartyPicker 'pick' mode, SWITCH overlay on first tap, second tap confirms; active mon excluded from picker)
- ✅ Pokéball throw animation (arc, open, shrink, bounce, shake, stars on catch)
- ✅ Real catch rates from PokéAPI data
- ✅ Move priority system wired to MoveRegistry (not movesDb)
- ✅ Sleep Talk — Gen IX unselectable list (including Rest), Comatose support, actual sub-move execution, PP not consumed
- ✅ player_send_out event — sprite + HUD update properly sequenced through event queue on both voluntary and forced switch

### Held Item System
- ✅ HeldItemEffects.ts — 7 hook functions (modifyDamageDealt, modifyDamageReceived, modifyStat, afterDamageDealt, endOfTurn, accuracyModifier, focusSash)
- ✅ Life Orb — 1.3× damage, 1/10 maxHP recoil after each hit
- ✅ Shell Bell — heals 1/8 of damage dealt
- ✅ Choice Band — 1.5× Attack (physical), locks first move used
- ✅ Choice Specs — 1.5× Sp. Attack (special), locks first move used
- ✅ Choice Scarf — 1.5× Speed, locks first move used; locked moves greyed out in move menu
- ✅ Expert Belt — 1.2× damage on super effective hits
- ✅ Assault Vest — special damage ÷1.5; status moves blocked with message
- ✅ Eviolite — 1.5× Def and SpDef if not fully evolved (checks evoChains)
- ✅ Leftovers — +1/16 maxHP at end of turn
- ✅ Black Sludge — +1/16 maxHP for Poison types; -1/8 maxHP for others
- ✅ Rocky Helmet — deals 1/6 attacker's maxHP on contact moves
- ✅ Focus Sash — survive one KO from full HP; item consumed on trigger
- ✅ Kings Rock — 10% flinch chance on all damaging moves
- ✅ Wide Lens — ×1.1 accuracy
- ✅ Zoom Lens — ×1.2 accuracy only when moving second (movingSecond flag threaded through engine)
- ✅ lockedMoveIndex cleared on switch-out

### Move System — Complete
- ✅ MoveId const object (789 moves, PokéAPI numeric IDs)
- ✅ Move / AttackMove / StatusMove / SelfStatusMove base classes with `.attr()` chaining
- ✅ MoveAttr composition system — 16 attr classes:
  - `StatusEffectAttr` (accepts selfTarget param), `StatChangeAttr`, `RecoilAttr`, `DrainAttr`, `HealAttr`
  - `FlinchAttr`, `ConfuseAttr`, `FixedDamageAttr`, `LevelDamageAttr`, `MultiHitAttr`
  - `HpRatioDamageAttr`, `WeightDamageAttr`, `SpeedRatioDamageAttr`, `TargetHpDamageAttr`
  - `StageScaledDamageAttr`, `TwoTurnMoveAttr`
  - `RestAttr` — full heal, clear status, apply 2-turn sleep (Gen IX)
- ✅ MoveRegistry — all 789 moves registered at boot, engine routes through registry exclusively
- ✅ All 16 type families fully declared (789/789 coverage)
- ✅ Variable power moves fully implemented
- ✅ Two-turn moves fully implemented
- ✅ Fixed damage, level-based damage, multi-hit moves
- ✅ Rest properly implemented via RestAttr (was declared but missing attrs)
- ✅ MoveEffects.ts deleted — legacy system fully gone

### Trainer / Rival System
- ✅ Unified trainer intro: sprite slides in → dialog in battle text box → sprite slides out → pokemon slide in
- ✅ Trainer sprites from PokeRogue (youngster_m, iris for Evan)
- ✅ TrainerDef with preDialog / winDialog / loseDialog
- ✅ Trainer node: picks random class per zone layer, seeded team generation
- ✅ Youngster class: 2 base Normal-type pokemon
- ✅ All post-battle dialog through battle text box (no DialogueOverlay)
- ✅ Win/loss dialog for both trainers and rival

### Map / Progression
- ✅ StS-style 7×15 procedural map (6 seeded paths, no-crossing invariant)
- ✅ Floor guarantees — floor 0 wild, floor 8 item, rival floor, floor 13 settlement/rest, floor 14 gym
- ✅ Isometric stagger layout
- ✅ Pan / zoom / pinch-zoom (activePointers: 2)
- ✅ Starts centred on active node at 1.6× zoom
- ✅ Node selection; node only marked visited on completion (markNodeComplete refactor)
- ✅ markNodeComplete — static method on NodeMapScene; every node scene calls it on successful completion before returning to map
- ✅ Mystery node — all 8 resolved types pass nodeId correctly; lore/item no-entry fallback calls markNodeComplete
- ✅ currentZoneIndex persisted to save on every node entry; restored on load (fixes skip-wave bug)
- ✅ All node types: Wild, Trainer, Rival, Rest, Lore, Item, Settlement, Rescue, NPC, Mystery, Evolution
- ✅ Logbook (account-level, accessible from title + node map)
- ✅ Zone themes (8 distinct visual identities)

### Day/Night System
- ✅ 14-step cycle: 1-2=Dawn, 3-7=Day, 8-9=Dusk, 10-14=Night
- ✅ Random phase-boundary start per run
- ✅ Advances on every node completion
- ✅ Persists in save slot
- ✅ Affects lore weighting (3× matching phase, 0.25× non-matching)
- ✅ Rest scene background images vary by time of day

### Items
- ✅ Full consumable item pool with tier weights and floor unlocks
- ✅ Full Heal conditional — only appears if team member has non-volatile status condition
- ✅ All 15 held items fully implemented in battle (see Held Item System above)
- ✅ Ball items — Poké Ball ×5 (Common), Great Ball ×5 (Uncommon), Ultra Ball ×5 (Rare)
- ✅ Ball rewards correctly write to slot.balls (not inventory) via ball: effect handler in RewardHandler
- ✅ EXP Charm (stackable, +50% EXP each)
- ✅ Item loot generation passes party for conditional filtering
- ✅ Antidote, Awakening, Burn Heal removed (redundant given Full Heal)

### Save System
- ✅ Account-level: discoveredLore[], unlockedCharms, roles
- ✅ Run slots: team, inventory, timeStep, mapSeed, nodesVisited, balls, currentZoneIndex
- ✅ nodesVisited only written on node completion (not on entry)

### Lore / NPC System
- ✅ `pickLoreEntry()` — seeded RNG, zone + time-of-day filter, rarity weighting (common 6×, uncommon 3×, rare 1×), skips discovered
- ✅ `pickNpcEntry()` — same as above but filters to entries with `npcName`, no time-of-day restriction
- ✅ Both functions accept `Set<string> | string[]` for discovered IDs

### Code Quality
- ✅ `monName()` / `titleCase()` utility (`src/utils/monName.ts`) — all Pokémon name outputs now title-cased (Yungoos not YUNGOOS)
- ✅ PartyPicker `'select'` mode — single-tap pick, no SWITCH overlay (used in RestScene train flow)
- ✅ Evan sprite updated to new uploaded art


- ✅ `SceneTypes.ts` — `NodeMapReturnData` includes nodeId? for completion tracking
- ✅ `PartyPicker` 'pick' mode — excludePartyIndex option to hide active mon in battle
- ✅ `stagesEqual()` helper in StatStages — replaces JSON.stringify comparisons
- ✅ `ZONE_ORDER` single export from MapGenerator
- ✅ `ms()` wraps all timing values
- ✅ `status_change` events — HUD refreshes mid-turn on any status change

## TODO

### Battle
- ✅ Invulnerability frames for two-turn invisible moves — `chargingMove.invisible` flag checked in engine; bypass moves (Earthquake vs Dig, Surf vs Dive, etc.) deal double damage via `INVULNERABILITY_BYPASSES`
- [ ] Weather system
- [ ] Terrain system
- [ ] Entry hazards (Stealth Rock, Spikes, Toxic Spikes, Sticky Web)
- [ ] Protect / Detect variants
- [ ] OHKO moves
- [ ] Switch-out moves (Volt Switch, U-turn, Flip Turn) — damage applies but switch doesn't trigger
- [ ] Delayed damage (Future Sight, Doom Desire)

### Content
- [ ] Gym battles (GymScene not started)
- [ ] Evan real sprite (currently using iris placeholder)
- [ ] Move Mastery system
- [ ] Meta shop (Solwara Coins flow)
- [ ] More lore entries
- [ ] More trainer classes
- [ ] Elite Four
- [ ] Items usable during battle (in-battle bag UI)

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
