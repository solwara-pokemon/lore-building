# Development Progress

## Completed

### Infrastructure
- ✅ Proxmox LXC setup (Debian 13, NAT, VPN access)
- ✅ Nginx serving solwara.scooom.com
- ✅ Phaser 4 + TypeScript + Vite scaffold
- ✅ Deploy service (`systemctl start solwara-deploy`)
- ✅ PokeAPI sprites submodule (sparse checkout, ~30MB)

### UI / UX
- ✅ Theme system (maroon #4a0000, Cinzel + Crimson Pro fonts)
- ✅ i18n system with English translations
- ✅ InputManager (normalizes mouse/touch)
- ✅ KeyBindings system (configurable, persisted to localStorage)
- ✅ BootScene (preloads splash image)
- ✅ SplashScene (avatar + progress bar + preloader)
- ✅ TitleScene (background art, menu, keyboard + touch nav)
- ✅ SettingsScene (game speed, keybindings)
- ✅ Role Select scene
- ✅ Starter Select scene

### Battle System
- ✅ BattleEngine — pure logic, no Phaser dependency
- ✅ Typed `BattleEvent[]` stream (move_use, hp_change, message, faint, battle_end)
- ✅ Sequential event queue (`playEventQueue`) — one event at a time, tap to advance
- ✅ Move animations mirroring PokeRogue's battle-anims system exactly
  - USER clone (target=0), TARGET clone (target=1), GRAPHIC sheet sprite (target=2)
  - FOCUS_USER, FOCUS_TARGET, FOCUS_USER_TARGET coordinate math
  - Per-frame background events (Surf, Rain Dance, etc.)
  - isOppAnim support (enemy-perspective anim variants)
- ✅ HP bars animate in real time as move animation plays (smooth tween, speed-aware)
- ✅ Damage/heal numbers spawn immediately with HP bar
- ✅ Typewriter text system (skip on first tap, advance on second)
- ✅ Self-targeting moves animate on correct pokemon
- ✅ Status moves do not trigger blink
- ✅ Pokemon sprite clones inherit flipX from original
- ✅ Catch system (calcCatch, ball types)
- ✅ Run system
- ✅ Faint animation
- ✅ EXP system (basic)
- ✅ Level cap enforcement
- ✅ Status conditions (burn, paralysis, sleep, freeze, poison, confusion)
- ✅ Ability effects (pre-move, damage modifier, end-of-turn, switch-in)
- ✅ Type effectiveness chart

### Map / Progression
- ✅ Node Map (procedural, grief-stage themed visuals)
- ✅ Meta progression / Solwara Coins (stub)

## In Progress
- 🟡 Battle polish (move animation edge cases)
- 🟡 Rival battle dialogue (Evan lines)

## Up Next
- Battle: switch during battle
- Battle: items during battle (potions etc.)
- Battle: faint → send next pokemon
- Move mastery system
- Evolution system
- Full node map event types (shop, rest, lore, elite)
- Save/load robustness

## Architecture Notes
- **Phaser 4 only** — never use Phaser 3 APIs. Phaser 4 uses `filters.addGlow()`, not `postFX.addGlow()`.
- Touch = primary input target, keyboard = bonus
- Landscape only, portrait shows rotate warning
- All strings through `t()`, all colors through `Theme`
- All battle text through `playEventQueue` — no direct `battleText.setText()` outside event handlers
- Battle animations sourced from `pagefaultgames/pokerogue-assets`, served from LXC filesystem
- Game speed (`ms()` utility) must wrap all timing values — tweens, delays, typewriter intervals
