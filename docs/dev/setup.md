# Dev Environment

## Stack
- **Phaser 4.x** (currently 4.1.0) — CRITICAL: use Phaser 4 API, not Phaser 3
- **TypeScript** + **Vite** as bundler/dev server
- **Node 24**, **npm 11**

## Phaser 4 Notes
- Filters: use `gameObject.enableFilters()` then `gameObject.filters.internal.addGlow()`
- NO `postFX`, NO `preFX` — those are Phaser 3 APIs
- Unified Filter system replaces all old FX/Mask systems

## Infrastructure
- Proxmox host: `node.scooom.com:8006`
- Game LXC hostname: `solwara`, private IP `10.8.0.10` (VPN only)
- Public URL: `solwara.scooom.com` (nginx proxying to Vite dev server)
- SSH: `ssh root@10.8.0.10` (over VPN) or `pct enter <CTID>` from Proxmox host

## Repos
- Game code: `solwara-pokemon/solwara` (branch: `main`)
- Lore/design: `solwara-pokemon/lore-building`

## Services
- `sudo systemctl start solwara` — Vite dev server
- `sudo systemctl restart solwara` — restart dev server
- `sudo systemctl start solwara-deploy` — build and deploy to nginx

## Project Structure
```
src/
  main.ts                  ← entry point, loads locale, starts Phaser
  theme.ts                 ← all colors and fonts (maroon #4a0000, Cinzel, Crimson Pro)
  scenes/
    boot/
      BootScene.ts         ← preloads splash image only
      SplashScene.ts       ← avatar splash + asset preloader
    title/
      TitleScene.ts        ← title screen with background art and menu
    settings/
      SettingsScene.ts     ← stub, coming soon
  utils/
    i18n.ts               ← translation system, t() function
    InputManager.ts        ← normalizes mouse/touch/keyboard input
    KeyBindings.ts         ← configurable keybindings (up/down/left/right/confirm/cancel)
  data/
    locales/
      en.json             ← all English strings
```

## Coding Conventions
- All user-facing strings go through `t()` from i18n — except splash screen
- All keyboard input goes through `KeyBindings.getInstance()`
- All pointer/touch input goes through `InputManager.getInstance(this)`
- All colors/fonts reference `Theme` from `src/theme.ts`
- Phaser 4 only — no Phaser 3 APIs

## Next Steps
- Role Select scene (Hero / The Doctor / The Accepted)
- Starter Select scene within role
- Node Map (procedural, grief-stage themed)
