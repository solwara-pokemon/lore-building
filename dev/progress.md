# Development Progress

## Completed
- ✅ Proxmox LXC setup (Debian 13, NAT, VPN access)
- ✅ Nginx serving solwara.scooom.com
- ✅ Phaser 4 + TypeScript + Vite scaffold
- ✅ Deploy service (systemctl start solwara-deploy)
- ✅ PokeAPI sprites submodule (sparse checkout, ~30MB)
- ✅ Theme system (maroon #4a0000, Cinzel + Crimson Pro fonts)
- ✅ i18n system with English translations
- ✅ InputManager (normalizes mouse/touch)
- ✅ KeyBindings system (configurable, persisted to localStorage)
- ✅ BootScene (preloads splash image)
- ✅ SplashScene (avatar + progress bar + preloader)
- ✅ TitleScene (background art, menu, keyboard + touch nav)
- ✅ SettingsScene (stub with working back button)

## In Progress
- 🟡 Role Select scene

## Up Next
- Role Select scene
- Starter Select scene
- Node Map (procedural, grief-stage themed visuals)
- Battle system
- Catch system
- Meta progression / Solwara Coins

## Known Issues / Notes
- Phaser 4 only — never use Phaser 3 APIs
- Touch = primary input target, keyboard = bonus
- Landscape only, portrait shows rotate warning
- All strings through t(), all keys through KeyBindings, all colors through Theme
- i18n excludes splash screen intentionally
