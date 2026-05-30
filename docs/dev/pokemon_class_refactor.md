# Pokemon Class Refactor — Forms & Display Names

**Branch proposal for:** `solwara-pokemon/solwara`  
**Related doc:** `lore-building/docs/dev/`  
**Status:** Design proposal

---

## Problem Statement

The current system sources Pokémon data entirely from PokéAPI slugs stored in `base.json`. This works for straightforward cases, but creates a class of bugs around **forms** and **display names** that can't be cleanly solved with the current data shape.

### Bug Class 1: Display Name Erasure

`monName.ts` strips form suffixes to produce a display name. The intent is good — `wormadam-plant` → `Wormadam` is correct. But:

- `giratina-altered` → `Giratina-Altered` (suffix `-altered` is **missing** from the strip list)
- `aegislash-blade` → `Aegislash-Blade` (same — `-blade` and `-shield` not in list)
- `castform-sunny` → `Castform-Sunny` (same — `-sunny`, `-rainy`, `-snowy` not in list)
- `squawkabilly-green-plumage` → not stripped (missing variant)
- `tatsugiri-curly` → not stripped (missing variant)
- `frillish-male` / `jellicent-male` → not stripped (gender forms not handled)

The suffix list in `monName.ts` and the one in `PokemonDataLoader.ts` are **already out of sync** — two lists that must be kept manually identical, and aren't.

### Bug Class 2: Display Name Over-Erasure

For forms that ARE in the suffix list, stripping loses meaningful identity:

- `marowak-alola` → `Marowak` (should be `Alolan Marowak`)
- `raichu-alola` → `Raichu` (should be `Alolan Raichu`)
- `meowth-galar` → `Meowth` (should be `Galarian Meowth`)
- `darmanitan-zen` → `Darmanitan` (form matters for stats/type)
- `deoxys-attack` vs `deoxys-speed` → both `Deoxys` (indistinguishable)

A player who catches an Alolan Marowak will see it labelled `Marowak` in the HUD, party picker, and evo scene.

### Bug Class 3: Sprite Fallback Fragility

`pokemonSpritePath(dex)` uses the numeric dex ID. For alternate forms (IDs in the `10000+` range), this works if the sprite exists. But there's no fallback to the base form sprite, no handling for forms that share a sprite, and the sprite path function has no awareness of what the dex ID actually represents.

### Root Cause

The data in `base.json` stores the PokéAPI internal slug (`wormadam-plant`, `giratina-altered`) as `name`, with no separation between:
- The **species slug** (used for lookups, learnsets, evolutions)
- The **form identifier** (which form this is)  
- The **display name** (what the player sees)

---

## Proposed Fix: Enrich `base.json` at Fetch Time

Rather than inferring display names at runtime via fragile string manipulation, compute them **once** in `fetch-data.mjs` and store them in the JSON. The runtime then reads pre-computed values directly.

### New Fields in `PokemonBaseData`

```ts
export interface PokemonBaseData {
  id: number;
  name: string;           // API slug (unchanged) — used for learnset/evo lookups
  displayName: string;    // NEW: human-readable, e.g. "Alolan Marowak", "Wormadam"
  formName: string | null; // NEW: form qualifier only, e.g. "Alola", "Plant Cloak", null
  baseSpeciesId: number;  // NEW: dex ID of the base form (self for base forms)
  // ... rest unchanged
}
```

### Fetch-Time Display Name Logic (in `fetch-data.mjs`)

PokéAPI's `pokemon-species` endpoint includes a `names` array with localized names and a `varieties` array. The `pokemon` endpoint includes a `forms` array. We already call species — we just don't use this data.

```js
// In fetchPokemon(), after fetching species:
const speciesData = await fetchCached(`pokemon-species/${p.speciesName}`);

// Get the canonical English species name (e.g. "Marowak", "Wormadam")
const englishName = speciesData.names.find(n => n.language.name === 'en')?.name 
  ?? titleCase(p.speciesName);

// Get form-specific name from the form endpoint
const formData = p.forms?.length ? await fetchCached(`pokemon-form/${p.name}`) : null;
const formEnglishName = formData?.form_names?.find(n => n.language.name === 'en')?.name ?? null;

// Determine the base species dex ID
const baseVariety = speciesData.varieties.find(v => v.is_default);
const baseFormName = baseVariety?.pokemon?.name;
const baseFormData = baseFormName ? pokemonByName[baseFormName] : null;
const baseSpeciesId = baseFormData?.id ?? p.id;

// Build display name
let displayName: string;
if (formEnglishName) {
  // PokéAPI has a proper form name, e.g. "Alolan Form" → use it
  // Strip " Form" suffix since we have the species name separately
  const qualifier = formEnglishName.replace(/ Form$/, '').replace(/ Forme$/, '');
  displayName = `${qualifier} ${englishName}`;
  // But for things like "Plant Cloak Wormadam" → just "Wormadam" if qualifier=species
  // Use formEnglishName directly if it contains the full name already
  if (formEnglishName.includes(englishName)) displayName = formEnglishName;
} else if (p.id === baseSpeciesId) {
  // This is the base form
  displayName = englishName;
} else {
  // Alternate form with no PokéAPI form name → fallback to species name
  displayName = englishName;
}

pokemon[p.id].displayName = displayName;
pokemon[p.id].formName = formEnglishName; // raw, for reference
pokemon[p.id].baseSpeciesId = baseSpeciesId;
```

### What This Buys

| Before | After |
|--------|-------|
| `marowak-alola` → `Marowak` (wrong) | `marowak-alola` → `Alolan Marowak` ✓ |
| `giratina-altered` → `Giratina-Altered` (wrong) | `giratina-altered` → `Giratina` ✓ |
| `aegislash-blade` → `Aegislash-Blade` (wrong) | `aegislash-blade` → `Aegislash` ✓ |
| `darmanitan-zen` → `Darmanitan` | `darmanitan-zen` → `Zen Mode Darmanitan` ✓ |
| `deoxys-attack` → `Deoxys` | `deoxys-attack` → `Attack Forme Deoxys` ✓ |
| Two lists (PokemonDataLoader + monName) must sync | One source of truth in JSON ✓ |

---

## Runtime Changes

### `PokemonDataLoader.ts`

Add `displayName`, `formName`, `baseSpeciesId` to `PokemonBaseData` interface. No logic changes — just reads the new fields.

### `monName.ts`

Simplify dramatically. `stripFormSuffix` and the duplicated `FORM_SUFFIXES` list are **deleted**. `monName` becomes:

```ts
import { pokemonDb } from '../systems/pokemon/PokemonDataLoader';

export function monName(
  nickname: string | undefined | null,
  speciesId: number,
): string {
  if (nickname) return nickname; // already title-cased when set
  return pokemonDb[speciesId]?.displayName ?? `#${speciesId}`;
}
```

All call sites change from `monName(nickname, data?.name, dex)` to `monName(nickname, speciesId)`. This is strictly simpler and removes the 3-argument ambiguity.

### `sprites.ts`

Add a form-aware sprite helper:

```ts
export function pokemonSpritePathSafe(speciesId: number): string {
  // Try form sprite first, fall back to base form
  const base = pokemonDb[speciesId]?.baseSpeciesId ?? speciesId;
  // Prefer exact ID, fall back to base species ID for shared sprites
  return speciesId >= 10000
    ? `sprites/sprites/pokemon/${speciesId}.png`  // alternate form
    : `sprites/sprites/pokemon/${speciesId}.png`;  // standard
}

// Adds a fallback: if form sprite missing, use base species sprite
export function pokemonSpriteKeyWithFallback(
  scene: Phaser.Scene, 
  speciesId: number
): string {
  const key = `pkmn-${speciesId}`;
  const baseKey = `pkmn-${pokemonDb[speciesId]?.baseSpeciesId ?? speciesId}`;
  return scene.textures.exists(key) ? key : baseKey;
}
```

---

## Migration Plan

### Phase 1: Fetch-time enrichment (no runtime changes yet)
1. Update `fetch-data.mjs` to compute and store `displayName`, `formName`, `baseSpeciesId`
2. Re-run `node scripts/fetch-data.mjs --pokemon` to regenerate `base.json`
3. Update `PokemonBaseData` interface to include new fields
4. **Verify** — spot-check 20 entries across base forms, regional forms, megas, gender forms

### Phase 2: Runtime simplification
5. Rewrite `monName()` to use `speciesId` directly
6. Delete `stripFormSuffix()` and the `FORM_SUFFIXES` constant from `monName.ts`
7. Remove the duplicate `FORM_SUFFIXES` from `PokemonDataLoader.ts` — the `isAlternateForm()` logic that drives `baseFormIds` can use `baseSpeciesId !== id` instead
8. Update all `monName()` call sites (BattleHUD, EvoScene, StarterSelectScene, PartyPicker, etc.)

### Phase 3: Sprite safety net (low priority)
9. Add `pokemonSpriteKeyWithFallback()` to `sprites.ts`
10. Use it in BattleField where enemy/player sprites are loaded

---

## Call Site Inventory

Files that call `monName()` or do their own name formatting:

| File | Current usage | Change needed |
|------|---------------|---------------|
| `BattleHUD.ts` | `monName(null, pokemonDb[pl.speciesId]?.name ?? pl.displayName)` | → `monName(null, pl.speciesId)` |
| `EvoScene.ts` | `monName(mon.nickname, pokemonDb[mon.dex]?.name, mon.dex)` | → `monName(mon.nickname, mon.dex)` |
| `EvoScene.ts` | `monName(null, pokemonDb[evo.evolvesInto]?.name, evo.evolvesInto)` | → `monName(null, evo.evolvesInto)` |
| `EvoScene.ts` (applyEvolution) | inline `toData?.name ?? '#${}'` | → `monName(null, evo.evolvesInto)` |
| `PartyPicker.ts` | (check) | likely similar |
| `TrainerCaseOverlay.ts` | (check) | likely similar |
| `BattleMenus.ts` | (check) | likely similar |
| `RescueScene.ts` | (check) | likely similar |
| `StarterSelectScene.ts` | (check) | likely similar |

---

## Risks & Mitigations

| Risk | Mitigation |
|------|------------|
| PokéAPI form names are inconsistent across generations | Spot-check after fetch; manual overrides table in fetch script as escape hatch |
| Re-fetching species data is slow (already cached) | Cache already exists — incremental fetch only hits new entries |
| `base.json` grows slightly | ~20 bytes/entry × 1350 entries = ~27KB. Negligible. |
| Call site misses | TypeScript: changing `monName` signature will cause compile errors at all old call sites, making misses impossible |

---

## What We're NOT Doing

- **No hardcoded Pokémon classes.** The concern in the thread title about "hard writing pokemon classes" is addressed by enriching the JSON, not by creating per-species TypeScript classes. That would be hundreds of files and a maintenance nightmare.
- **No pokemondb dependency.** All data continues to come from PokéAPI.
- **No changes to save format.** `RunPokemon.dex` stays as the numeric ID. The display layer changes; the storage layer doesn't.

---

## Estimated Effort

| Phase | Effort |
|-------|--------|
| Phase 1 (fetch-time enrichment) | ~2–3 hours (fetch script + verification) |
| Phase 2 (runtime simplification) | ~1–2 hours (monName rewrite + call sites) |
| Phase 3 (sprite safety net) | ~30 min |

**Total: ~4–5 hours. Low risk, high correctness payoff.**
