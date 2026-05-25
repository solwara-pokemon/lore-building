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
- ✅ Nameplates with type badges, stat stages
- ✅ EXP bar + level up events (all 6 growth rate curves)
- ✅ Faint → forced switch menu
- ✅ Enemy auto-swap on faint (enemy_send_out event, sprite rebuild)
- ✅ Fainted pokemon's move skipped (engine-level fix)
- ✅ Catching grants EXP (after caught message)
- ✅ Failed catch → opponent attacks
- ✅ Back sprites for player pokemon
- ✅ Status conditions, abilities, type effectiveness

### Trainer / Rival System
- ✅ Unified trainer intro: sprite slides in → dialog in battle text box → sprite slides out → pokemon slide in
- ✅ Trainer sprites from PokeRogue (youngster_m, iris for Evan)
- ✅ TrainerDef with preDialog / winDialog / loseDialog
- ✅ Trainer node: picks random class per zone layer, seeded team generation
- ✅ Youngster class: 2 base Normal-type pokemon
- ✅ All post-battle dialog through battle text box (no DialogueOverlay)
- ✅ Win/loss dialog for both trainers and rival

### Map / Progression
- ✅ Node Map (procedural, grief-stage themed)
- ✅ Wild encounter node (type-biased, BST ceiling per zone, min level 2)
- ✅ Trainer node (Youngster, seeded team)
- ✅ Rival node (Evan with iris sprite)
- ✅ Rest node (heal / train move) with watercolor background images per time of day
- ✅ Lore node (5 types: document, gossip, broadcast, relic, rumour)
- ✅ Logbook (account-level, accessible from title + node map)

### Day/Night System
- ✅ 14-step cycle: 1-2=Dawn, 3-7=Day, 8-9=Dusk, 10-14=Night
- ✅ Random phase-boundary start per run
- ✅ Advances on every node completion
- ✅ Persists in save slot
- ✅ Affects lore weighting (3× matching phase, 0.25× non-matching)
- ✅ Rest scene background images vary by time of day

### Items
- ✅ EXP Charm (stackable ×50, +50% EXP each, starts with 1 per run)
- ✅ Item loot pool with tier weights

### Save System
- ✅ Account-level: discoveredLore[], unlockedCharms, roles
- ✅ Run slots: team, inventory, timeStep, mapSeed, nodesVisited, balls

## TODO
- [ ] Spawn system full design — wild encounters need time of day as primary factor
- [ ] Settlement node
- [ ] Shop node
- [ ] Item node
- [ ] Mystery node
- [ ] Evolution node
- [ ] Elite trainer node
- [ ] More trainer classes
- [ ] More lore entries (collaborative)
- [ ] Evan real sprite (currently using iris as placeholder)
- [ ] Battle: voluntary switch during battle
- [ ] Battle: items during battle (potions)
- [ ] Evolution system
- [ ] Move mastery system

## Architecture Notes
- **Phaser 4 only** — `filters.addGlow()`, not `postFX.addGlow()`
- Touch = primary, keyboard = bonus
- All battle text through `playEventQueue` / `showDialogLines`
- All battle dialog (trainer/rival pre/post) through battle text box
- Settings: per-scene button with `returnTo` pattern, `scene.start('SettingsScene', { returnTo, returnData })`
- Game speed: `ms()` wraps all timing values
- Trainer sprites: multiatlas from `public/images/trainers/`, frame `0001.png` = idle
- Battle animations: PokeRogue assets, `public/battle-anims/` and `public/images/battle_anims/`
- Type/category icons: PokeRogue `public/images/types.png` / `categories.png`
