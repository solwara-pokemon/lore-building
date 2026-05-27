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
- ✅ Nameplates with type badges, stat stages, status condition (colour-coded)
- ✅ Status display on HUD — updates mid-turn, clears on cure
- ✅ EXP bar + level up events (all 6 growth rate curves)
- ✅ Faint → forced switch menu
- ✅ Enemy auto-swap on faint (enemy_send_out event, sprite rebuild)
- ✅ Fainted pokemon's move skipped (engine-level fix)
- ✅ Catching grants EXP (after caught message)
- ✅ Failed catch → opponent attacks
- ✅ Back sprites for player pokemon
- ✅ Status conditions, abilities, type effectiveness
- ✅ Voluntary switch during battle (PartyPicker 'pick' mode, two-tap confirm via SWITCH overlay)
- ✅ Pokéball throw animation (arc, open, shrink, bounce, shake, stars on catch)
- ✅ Real catch rates from PokéAPI data
- ✅ Move priority system wired to MoveRegistry
- ✅ Sleep Talk — Gen IX unselectable list, Comatose support, actual sub-move execution

### Move System
- ✅ MoveId const object (789 moves, PokéAPI numeric IDs)
- ✅ Move / AttackMove / StatusMove / SelfStatusMove base classes
- ✅ MoveAttr composition system (StatusEffectAttr, StatChangeAttr, RecoilAttr, DrainAttr, HealAttr, FlinchAttr, ConfuseAttr, FixedDamageAttr, LevelDamageAttr, MultiHitAttr)
- ✅ MoveRegistry — registry takes priority over legacy movesDb in engine
- ✅ All 16 type families declared with implementation status flags
- ✅ Multi-hit moves (Gen 5+ distribution for 2-5 hit)
- ✅ Fixed damage moves (Dragon Rage, Sonic Boom)
- ✅ Level-based damage (Night Shade, Seismic Toss)
- ✅ Legacy MoveEffects.ts fallback for unregistered moves

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
- ✅ Node selection, in-progress tracking (node only marked visited on return)
- ✅ Wild encounter node (type-biased, BST ceiling per zone, min level 2)
- ✅ Trainer node (Youngster, seeded team)
- ✅ Rival node (Evan with iris sprite)
- ✅ Rest node (heal / train move) — uses PartyPicker 'pick' mode for mon selection
- ✅ Lore node (5 types: document, gossip, broadcast, relic, rumour)
- ✅ Item node — item card selection fixed (hit zone offset bug resolved)
- ✅ Settlement, Rescue, NPC, Mystery, Evolution nodes
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
- ✅ Full Heal conditional — only appears if team member has status condition
- ✅ Held item pool (15 items — equippable but no battle effect yet)
- ✅ EXP Charm (stackable ×50, +50% EXP each)
- ✅ Item loot generation passes party for conditional filtering

### Save System
- ✅ Account-level: discoveredLore[], unlockedCharms, roles
- ✅ Run slots: team, inventory, timeStep, mapSeed, nodesVisited, balls
- ✅ `currentNodeId` tracks in-progress node (not written to nodesVisited until return)

### Code Quality
- ✅ `toHex()` utility — replaced 21 inline `toString(16).padStart(6,'0')` calls
- ✅ `SceneTypes.ts` — `NodeMapReturnData` shared type across all node scenes
- ✅ `PartyPicker` 'pick' mode — all party selection flows use same UI
- ✅ `ZONE_ORDER` single export from MapGenerator
- ✅ `ms()` wraps all timing values
- ✅ `status_change` events — HUD refreshes mid-turn on any status change

## TODO

### Battle
- [ ] Variable power moves (Gyro Ball, Flail, Reversal, Wring Out, etc.)
- [ ] Two-turn moves (Fly, Dig, Solar Beam, Bounce, etc.)
- [ ] Held item effects in battle (15 held items currently decorative)
- [ ] Fix Rest double-decrement (wakes 1 turn early)
- [ ] Weather system
- [ ] Terrain system
- [ ] Entry hazards
- [ ] Protect / Detect variants
- [ ] OHKO moves
- [ ] Delete MoveEffects.ts / movesDb once registry is complete

### Content
- [ ] Gym battles (GymScene not started)
- [ ] Evan real sprite (currently using iris placeholder)
- [ ] Move Mastery system
- [ ] Meta shop (Solwara Coins flow)
- [ ] More lore entries
- [ ] More trainer classes
- [ ] Elite Four

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
