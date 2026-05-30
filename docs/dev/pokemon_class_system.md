# Pokémon Class System — Design Document

**Status:** Approved, pending implementation  
**Replaces:** `pokemon_class_refactor.md` (superseded)

---

## The Problem

The current system uses `base.json` — a PokéAPI artifact — as the runtime source of truth for all Pokémon data. This creates several classes of bugs and architectural weaknesses:

- Form display names are inferred at runtime via fragile suffix-stripping (`monName.ts`, `stripFormSuffix`)
- Two duplicate `FORM_SUFFIXES` lists exist in the codebase and are already out of sync
- Alternate forms like `giratina-altered`, `aegislash-blade`, `castform-sunny` are not correctly stripped and display with raw slug names
- Regional forms like `marowak-alola` strip to `Marowak` instead of `Alolan Marowak`
- No type safety — species are raw `number` everywhere, `pokemonDb[dex]` can silently return `undefined`
- `BattlePokemon` and `RunPokemon` have no formal relationship — syncing between them is manual and error-prone
- `SaveManager.save()` is called too eagerly, inside battle flows where it should never fire

---

## New File Structure

```
src/data/pokemon/
  SpeciesId.ts                  — enum, all ~1025 species, fully explicit values
  PokemonSpecies.ts             — PokemonSpecies, PokemonForm, SpeciesBuilder, FormBuilder classes
  species/
    gen1.ts                     — registry entries for Gen 1
    gen2.ts
    gen3.ts
    gen4.ts
    gen5.ts
    gen6.ts
    gen7.ts
    gen8.ts
    gen9.ts
    index.ts                    — assembles all gens, exports speciesRegistry
  learnsets/
    level.ts                    — re-keyed from level.json, uses SpeciesId
    tm.ts
    tutor.ts
    egg.ts
    formOverrides.ts            — form-specific learnset overrides (SpeciesId → formKey → LevelMove[])

src/systems/pokemon/
  RuntimePokemon.ts             — NEW: wraps RunPokemon + species reference, adds methods
```

**Deleted once migration is complete:**
- `src/data/pokemon/base.json`
- `src/data/pokemon/learnsets/level.json`
- `src/data/pokemon/learnsets/tm.json`
- `src/data/pokemon/learnsets/tutor.json`
- `src/utils/monName.ts` (replaced by `getDisplayName()`)

---

## The Three-Layer Stack

```
RunPokemon          — plain interface, serialized to localStorage
    ↓ wraps
RuntimePokemon      — in-memory class, adds species methods, used everywhere outside battle
    ↓ wraps
BattlePokemon       — adds ephemeral battle state, thrown away on tab close / battle end
```

Each layer owns distinct concerns:

| Layer | Owns | Persists |
|---|---|---|
| `RunPokemon` | speciesId, formKey, nickname, level, moves, pp, hp, status, heldItem, exp | Yes — localStorage |
| `RuntimePokemon` | species reference, display methods, stat lookups | No — in-memory only |
| `BattlePokemon` | stat stages, confusion, flinch, protect state, charge state, uid | No — ephemeral |

---

## `SpeciesId` Enum

Fully explicit on every entry — no auto-increment, no ambiguity if species are ever reordered.

```ts
// src/data/pokemon/SpeciesId.ts

export enum SpeciesId {
  BULBASAUR   = 1,
  IVYSAUR     = 2,
  VENUSAUR    = 3,
  CHARMANDER  = 4,
  CHARMELEON  = 5,
  CHARIZARD   = 6,
  // ...
  MR_MIME     = 122,
  // ...
  NIDORAN_F   = 29,
  NIDORAN_M   = 32,
  // all ~1025 species, explicit
}
```

All current uses of raw `number` for species (`pokemonDb[dex]`, `RunPokemon.dex`, `BattlePokemon.speciesId`) migrate to `SpeciesId`.

---

## `PokemonSpecies` and `PokemonForm`

```ts
// src/data/pokemon/PokemonSpecies.ts

export interface BaseStats {
  hp: number; atk: number; def: number;
  spAtk: number; spDef: number; spe: number;
}

// Abilities: [slot1, slot2 | null, hidden]
export type AbilitySlots = [string, string | null, string];

export type GrowthRate =
  | 'erratic' | 'fast' | 'medium-fast' | 'medium-slow' | 'slow' | 'fluctuating';

export class PokemonForm {
  readonly speciesId:  SpeciesId;
  readonly formKey:    string;       // "sandy", "alola", "zen" — slug used in saves + sprite lookup
  readonly formName:   string;       // "Sandy Cloak", "Alolan Form", "Zen Mode" — display
  readonly types:      [string, string?];
  readonly stats:      BaseStats;
  readonly abilities:  AbilitySlots;
}

export class PokemonSpecies {
  readonly id:          SpeciesId;
  readonly name:        string;      // "Wormadam", "Mr. Mime" — display-ready, not a slug
  readonly types:       [string, string?];
  readonly stats:       BaseStats;
  readonly abilities:   AbilitySlots;
  readonly catchRate:   number;
  readonly genderRatio: number | null;  // % male; null = genderless
  readonly growthRate:  GrowthRate;
  readonly baseExp:     number;
  readonly legendary:   boolean;
  readonly mythical:    boolean;
  readonly forms:       PokemonForm[];

  // Returns the form matching the given key, or null
  getForm(formKey: string | null): PokemonForm | null

  // Returns the display name, accounting for form
  // null/undefined formKey → species name only ("Wormadam")
  // with formKey → "Sandy Cloak Wormadam", "Alolan Marowak", etc.
  getDisplayName(formKey?: string | null): string

  getTypes(formKey?: string | null): [string, string?]
  getStats(formKey?: string | null): BaseStats
  getAbilities(formKey?: string | null): AbilitySlots
  getBST(formKey?: string | null): number
}
```

Forms are **always fully explicit** — abilities, types, and stats are never inherited from the parent species. More verbose, zero ambiguity.

---

## Builder API

Fluent registration used in the gen files. `.addForm()` returns a `FormBuilder`; calling `.addForm()` again or finishing the chain returns to the `SpeciesBuilder`.

```ts
// Simple species — no forms
addSpecies(SpeciesId.MACHOP, 'Machop')
  .types('fighting')
  .stats({ hp: 70, atk: 80, def: 50, spAtk: 35, spDef: 35, spe: 35 })
  .abilities('guts', 'no-guard', 'steadfast')
  .catchRate(180)
  .genderRatio(75)
  .growthRate('medium-slow')
  .baseExp(61);

// Species with forms
addSpecies(SpeciesId.WORMADAM, 'Wormadam')
  .types('bug', 'grass')
  .stats({ hp: 60, atk: 59, def: 85, spAtk: 79, spDef: 105, spe: 36 })
  .abilities('anticipation', null, 'overcoat')
  .catchRate(45)
  .genderRatio(0)
  .growthRate('medium')
  .baseExp(162)
  .addForm('Plant Cloak', 'plant')
    .types('bug', 'grass')
    .stats({ hp: 60, atk: 59, def: 85, spAtk: 79, spDef: 105, spe: 36 })
    .abilities('anticipation', null, 'overcoat')
  .addForm('Sandy Cloak', 'sandy')
    .types('bug', 'ground')
    .stats({ hp: 60, atk: 79, def: 105, spAtk: 59, spDef: 85, spe: 36 })
    .abilities('anticipation', null, 'overcoat')
  .addForm('Trash Cloak', 'trash')
    .types('bug', 'steel')
    .stats({ hp: 60, atk: 69, def: 95, spAtk: 69, spDef: 95, spe: 36 })
    .abilities('anticipation', null, 'overcoat');

// Regional form — galar rattata gets its own SpeciesId entry, same as now
addSpecies(SpeciesId.MAROWAK_ALOLA, 'Marowak')
  .types('fire', 'ghost')
  .stats({ hp: 60, atk: 80, def: 110, spAtk: 50, spDef: 80, spe: 45 })
  .abilities('cursed-body', 'lightning-rod', 'rock-head')
  .catchRate(75)
  .genderRatio(50)
  .growthRate('medium-fast')
  .baseExp(149)
  .regionalVariant('Alolan');  // drives getDisplayName() → "Alolan Marowak"
```

---

## Registry Functions

```ts
// src/data/pokemon/species/index.ts

export function getSpecies(id: SpeciesId): PokemonSpecies
// Replaces: pokemonDb[dex]
// Throws if id not found — no silent undefined

export function getAllSpecies(): PokemonSpecies[]

export function isValidSpecies(id: number): id is SpeciesId
```

`getForm()` and `getDisplayName()` are methods on `PokemonSpecies`, not standalone functions — call sites look like:

```ts
getSpecies(SpeciesId.WORMADAM).getDisplayName('sandy')  // "Sandy Cloak Wormadam"
getSpecies(SpeciesId.MAROWAK_ALOLA).getDisplayName()    // "Alolan Marowak"
getSpecies(SpeciesId.MACHOP).getDisplayName()           // "Machop"
```

---

## `RunPokemon` Shape Change

Clean break — no migration compatibility needed (dev builds only, no live saves to protect).

```ts
// Before
interface RunPokemon {
  dex:      number;
  // no formKey
}

// After
interface RunPokemon {
  speciesId: SpeciesId;
  formKey:   string | null;   // null = default form
  // ...rest unchanged
}
```

---

## `RuntimePokemon`

Wraps `RunPokemon` and holds a reference to the resolved `PokemonSpecies`. All display and lookup logic lives here — call sites never need to know about `formKey` or `getSpecies()`.

```ts
// src/systems/pokemon/RuntimePokemon.ts

export class RuntimePokemon {
  private readonly data:    RunPokemon;
  private readonly species: PokemonSpecies;

  constructor(data: RunPokemon) {
    this.data    = data;
    this.species = getSpecies(data.speciesId);
  }

  // Identity
  getDisplayName(): string   // nickname if set, else species.getDisplayName(formKey)
  getSpeciesName(): string   // always the species name, ignores nickname
  getFormKey(): string | null

  // Species data — form-aware
  getTypes(): [string, string?]
  getStats(): BaseStats
  getAbilities(): AbilitySlots
  getBST(): number
  getForm(): PokemonForm | null

  // Save data passthrough — read only
  get level(): number
  get hp(): number
  get maxHp(): number
  get moves(): string[]
  get pp(): number[]
  get maxPp(): number[]
  get status(): string | null
  get heldItem(): string | null
  get currentExp(): number
  get nickname(): string | null

  // Returns the underlying save data — used when persisting
  toRunPokemon(): RunPokemon
}
```

`BattleSetup`, `PartyPicker`, `EvoScene`, and all other consumers that currently call `pokemonDb[mon.dex]` will instead hold a `RuntimePokemon` and call its methods directly.

---

## Learnsets

Stay as separate files — not moved into the species class. Learnset data is balance data, species data is biological data; they change for different reasons.

Re-keyed from raw dex numbers to `SpeciesId`. Structure otherwise unchanged.

Form-specific learnsets handled via `formOverrides.ts`:

```ts
// formOverrides.ts
// Keyed SpeciesId → formKey → LevelMove[]
// Falls back to species learnset when no override exists

export const formLevelMoveOverrides: Partial<Record<SpeciesId, Record<string, LevelMove[]>>> = {
  [SpeciesId.DARMANITAN_GALAR]: {
    'zen': [ /* zen mode specific moves */ ],
  },
};
```

---

## Save Timing

`SaveManager.save()` must **never** be called from inside `systems/battle/` or `scenes/battle/`. The save fires only when a node is marked complete.

This is intentional design — reloading the tab rewinds to before node entry. Players can retry battles or pick different nodes by reloading. This is a supported play pattern, not a bug.

Any PR that introduces a `save()` call inside the battle system should be rejected at review.

---

## Dev Mode

`__DEV_MODE__` is defined in `vite.config.ts` as a compile-time constant:

```ts
define: {
  __DEV_MODE__: process.env.DEV_MODE === 'true' || command === 'serve',
}
```

| Command | `__DEV_MODE__` |
|---|---|
| `pnpm dev` | `true` |
| `pnpm build` | `false` |
| `pnpm build:dev` | `true` |

`DevOverrides` already gates on this. The settings page gets a **"Clear All Save Data"** button visible only when `__DEV_MODE__` is `true`.

---

## Call Site Migration Map

| Current | Replacement |
|---|---|
| `pokemonDb[dex]` | `getSpecies(speciesId)` |
| `pokemonDb[dex]?.name` | `runtimeMon.getDisplayName()` |
| `monName(nickname, data?.name, dex)` | `runtimeMon.getDisplayName()` |
| `stripFormSuffix(name)` | deleted |
| `FORM_SUFFIXES` (both copies) | deleted |
| `mon.dex` on `RunPokemon` | `mon.speciesId` |
| raw `pokemonDb` lookup in scenes | construct `RuntimePokemon`, call methods |

19 files, 44 `pokemonDb` call sites — all touched in the migration phase.

---

## What's Deferred

- **In-battle form changes** (Aegislash, Castform, Morpeko, etc.) — the data model supports them (`forms` array exists on every species), trigger logic not implemented yet
- **Shiny / gender sprite variants** — `getSpriteKey()` stub exists on `RuntimePokemon`, implementation deferred
- **Full learnset authoring** — learnset files will be re-keyed from existing JSON initially; hand-auditing per species is ongoing

---

## Phased Implementation Plan

**Phase 1 — Core scaffolding**  
`SpeciesId.ts`, `PokemonSpecies.ts` (classes + builder + registry). No species data yet. Verified compiling and importable.

**Phase 2 — Gen 1 data + learnsets**  
Author Gen 1 in `gen1.ts`. Re-key learnset files to `SpeciesId`. Verify `getSpecies()`, `getDisplayName()`, form lookups work against real data.

**Phase 3 — `RuntimePokemon`**  
Implement `RuntimePokemon.ts`. Migrate `BattleSetup` and one scene (`EvoScene`) as a proof of concept. Verify the three-layer stack works end to end.

**Phase 4 — Remaining gens**  
Gen 2–9, one file at a time. Can be parallelised or done incrementally.

**Phase 5 — Full migration**  
Replace all `pokemonDb` call sites. Delete `base.json` and old learnset JSON. Delete `monName.ts`. Update `RunPokemon` shape. Audit and remove any `save()` calls inside battle.

**Phase 6 — Dev tooling**  
Clear save data button in settings, gated on `__DEV_MODE__`.
